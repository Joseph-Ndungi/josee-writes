---
title: "An Afternoon Spent Arguing With a Dropdown"
date: 2026-08-11
categories:
  - Frontend
  - Angular
tags:
  - Angular
  - DevExtreme
  - Plotly
  - Data Visualization
  - UI UX
description: "What was meant to be a light evening task, showing fitted distribution results for a security, turned into a full rebuild covering a security selector, reusable helpers, Plotly charts, and expansion panels."
image: https://images.unsplash.com/photo-1666875753105-c63a6f3bdc86?q=80&w=1173&auto=format&fit=crop&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
---

# An Afternoon Spent Arguing With a Dropdown

Spent the morning knee deep in subscription flow work, so by evening I was looking for something light to close the day with. Show fitted distribution results for a security. Grid on one side, raw text output on the other. Looked like a perfect evening task.

Some days start small and stay small. This was not one of those days.

Then someone pointed out that the analysis actually returns results for multiple securities, not just one, and the code was quietly grabbing `res[0]` and throwing the rest away like an intern shredding documents they did not understand. So here is the full walk through of how a "quick UI task" turned into a proper afternoon, step by step, mistakes included.

## Step 1, notice the crime scene

The original subscribe callback looked innocent enough.

```typescript
const fitted = res[0]?.Result?.FittedDistributions ?? [];
```

That `res[0]` is the whole problem in five characters. The backend was dutifully returning an array, one entry per security, each with its own `FittedDistributions`, and the frontend was politely nodding, taking the first one, and pretending the other five did not exist. Functionally correct for a single security demo. Structurally a landmine the moment anyone asked "can I see Limuru Tea as well."

Someone asked.

## Step 2, stop throwing data away

First real fix, keep the entire response around instead of destructuring it down to nothing on arrival.

```typescript
allDistributionResults: any[] = [];
securities: string[] = [];
selectedSecurity: string = '';

next: (res: any[]) => {
  if (!res?.length) {
    return;
  }
  this.hasFitDistributions = true;
  this.allDistributionResults = res;
  this.securities = res.map((r) => r.Security);
  this.selectedSecurity = this.securities[0];
  this.applySelectedSecurity();
}
```

Simple change, but it flips the whole mental model. Instead of the API response being consumed once and discarded, it becomes a small local dataset the UI can query on demand. Everything downstream now depends on `selectedSecurity` rather than an array index that happened to work for the demo data.

## Step 3, give the user a way to pick

Added a dropdown, went with DevExtreme's `dx select box` since the rest of the grid work was already in that ecosystem and mixing component libraries for no reason is how you end up with three different focus ring colors on one screen.

```html
<div class="security-selector">
  <span class="selector-label">Security</span>
  <dx-select-box
    [dataSource]="securities"
    [value]="selectedSecurity"
    (onValueChanged)="onSecurityChange($event.value)"
    placeholder="Select security"
  >
  </dx-select-box>
</div>
```

```typescript
onSecurityChange(security: string): void {
  this.selectedSecurity = security;
  this.applySelectedSecurity();
}
```

Nothing clever here. Pick a security, rebuild the view for that security. The interesting part is what "rebuild the view" actually means once you break it apart.

## Step 4, build helpers instead of one giant blob

The original code built the grid rows and the raw text output inline inside the subscribe callback, which is fine when there is one code path and gets ugly fast the moment you need to rerun that logic every time the dropdown changes. So it got split into three small, boring, single purpose helpers.

```typescript
private applySelectedSecurity(): void {
  const selectedResult = this.allDistributionResults.find(
    (r) => r.Security === this.selectedSecurity
  );
  const fitted = selectedResult?.Result?.FittedDistributions ?? [];

  this.distributionGrid = this.buildDistributionGrid(fitted, this.selectedSecurity);
  this.distributionRawOutput = this.buildRawOutput(fitted);
  this.distributionVisuals = this.buildVisualsData(fitted);
}
```

`buildDistributionGrid` maps fitted distributions into grid rows, same shape as before, just now security aware rather than hardcoded to whatever `res[0]` happened to be. `buildRawOutput` reassembles the human readable text block, distribution name, parameters, AIC, BIC, log likelihood, KS statistic, P value, same format as the original but rerunnable per selection. `buildVisualsData` is the new one, and it is the reason the whole afternoon got more interesting than originally planned.

## Step 5, numbers alone were not telling the story

A grid full of AIC and BIC values is useful if you already know what a good fit looks like. It is significantly less useful if you are trying to explain to someone why StudentT won over Normal without pulling out a statistics textbook. So charts went in, built off two arrays the backend was already sending and nobody was using, `PdfValues` for the fitted curve and `HistogramBins` for the actual observed data.

```typescript
private buildVisualsData(fitted: any[]): any[] {
  return fitted.map((d) => {
    const histogramBins = d.HistogramBins ?? [];
    const pdfValues = d.PdfValues ?? [];

    const histogramTrace = {
      x: histogramBins.map((b) => b.MidPoint),
      y: histogramBins.map((b) => b.Density),
      type: 'bar',
      name: 'Histogram',
      marker: { color: 'rgba(100, 149, 237, 0.6)' },
    };

    const pdfTrace = {
      x: pdfValues.map((p) => p.X),
      y: pdfValues.map((p) => p.Y),
      type: 'scatter',
      mode: 'lines',
      name: 'Fitted PDF',
      line: { color: 'rgb(220, 20, 60)', width: 2 },
    };

    return {
      distribution: d.Distribution,
      data: [histogramTrace, pdfTrace],
      layout: {
        title: d.Distribution,
        xaxis: { title: 'Value' },
        yaxis: { title: 'Density' },
        margin: { t: 40, l: 50, r: 20, b: 40 },
        height: 300,
      },
    };
  });
}
```

Bar chart for the observed histogram density, line for the fitted curve, both on the same axis so they overlay cleanly. Suddenly you do not need to interpret a P value in isolation, you can just look at whether the red line sits reasonably close to the blue bars or whether it is off having its own adventure somewhere in the tails.

Rendered with Plotly, which I had already used elsewhere so no new dependency to justify to anyone.

```html
<div class="distribution-visuals">
  <div *ngFor="let v of distributionVisuals" class="visual-block">
    <h4>{{ v.distribution }}</h4>
    <plotly-plot
      [data]="v.data"
      [layout]="v.layout"
      [config]="{ displayModeBar: false, responsive: true }"
    >
    </plotly-plot>
  </div>
</div>
```

## Step 6, three things fighting for the same screen

Grid, raw output, and now charts, all wanting attention at once. First attempt tried squeezing all three into a three column layout, grid left, raw output middle, charts right. Looked fine in theory. In practice it looked like a cockpit dashboard, and not the good kind.

Fixed it by keeping grid and raw output side by side, since those two are naturally compact and comparative, and moving the charts to their own full width section below, since PDF curves need horizontal room to actually be readable rather than squashed into a sidebar.

## Step 7, hide what is not needed yet

Even with the layout sorted, the page still opened with everything visible at once, selector, grid, raw output, and charts, before the user had picked anything meaningful. So the whole output section got wrapped into two Angular Material expansion panels, one for selections and one for results.

```typescript
selectionsExpanded = true;
outputExpanded = false;
```

And once results actually come back, the handoff happens automatically.

```typescript
next: (res: any[]) => {
  // ...build everything...

  this.selectionsExpanded = false;
  this.outputExpanded = true;
}
```

Small detail, disproportionately satisfying to watch work. Pick a security, hit run, selections panel politely closes itself, results panel opens up like it has been waiting all day for its turn.

## On performance, since it came up

Nothing dramatic to report here, which is the best kind of report. Switching between securities does not refetch anything, the full response is already sitting in memory from the initial call, so `applySelectedSecurity` just filters and remaps existing data. No loading spinners, no network round trips, no reason to write three paragraphs justifying a caching strategy nobody asked for. The helpers are small, pure, and only run when the selection actually changes, so there is no unnecessary recomputation happening in the background either.

## Where it landed

What started as "show the first security's results" is now a proper multi security view. Pick any security from a dropdown, get a comparison grid of fitted distributions, a readable text breakdown, and charts that actually let you see the fit rather than just read numbers about it, all tucked behind two expansion panels that only show you what is relevant at each stage.

Took considerably longer than the "before dinner" estimate I had given myself. Worth it though, and the dropdown and I are on speaking terms again.

Happy Life.
---
title: "Building Plots in Angular for Data Visualization"
date: 2026-23-03
categories: [Plotly, Angular, Data Visualization]
tags: [Angular, Data Visualization, Plotly]
canonical_url: "https://dev.to/josephndungi/building-plots-in-angular-for-data-visualization-487j"
image: https://plus.unsplash.com/premium_photo-1661493966437-48e33f1df64a?q=80&w=1332&auto=format&fit=crop&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
description: "A short guide on building and managing data visualizations in Angular with Plotly, including handling multiple datasets and improving UI with popup charts."
---


# Building Plots in Angular for Data Visualization

Data is easier to understand when you can see it. Tables are useful, but charts help you quickly spot trends, patterns, and outliers. In this post, I will share how I built plots in Angular for a linear regression tool and some lessons I learned along the way.

---

## Why plots matter

When working with models like linear regression, numbers alone are not enough. You need visuals to answer questions like:

* Are predictions close to actual values
* Are errors randomly distributed
* Are there outliers affecting the model

Plots such as residual charts and QQ plots make this very clear.

---

## Choosing a plotting library

In Angular, one of the easiest ways to create charts is by using Plotly.

Plotly works well because:

* It supports many chart types
* It is interactive out of the box
* It works nicely with Angular components

To use it, install the Angular wrapper:

```bash
npm install plotly.js angular-plotly.js
```

Then register it in your module:

```ts
import { PlotlyModule } from 'angular-plotly.js';

@NgModule({
  imports: [PlotlyModule]
})
export class AppModule {}
```

---

## Preparing your data

Most APIs return data as arrays of objects. For example:

```ts
[
  { Fitted: 1.2, Residual: 0.3 },
  { Fitted: 1.5, Residual: -0.2 }
]
```

Before plotting, you need to map this into arrays:

```ts
const x = data.map(d => d.Fitted);
const y = data.map(d => d.Residual);
```

This is a very common step when working with charts.

---

## Creating a simple plot

Here is an example of a residual plot:

```ts
createResidualPlot(data: any[]) {
  const trace = {
    x: data.map(d => d.Fitted),
    y: data.map(d => d.Residual),
    mode: 'markers',
    type: 'scatter',
    name: 'Residuals'
  };

  const layout = {
      title: 'Residuals vs Fitted',
      xaxis: {
        title: {
          text: 'Fitted Values',
        }
      },
      yaxis: {
        title: {
          text: 'Residuals',
        }
      },
    };

  this.residualPlot = {
    data: [trace],
    layout
  };
}
```

And in your HTML:

```html
<plotly-plot
  [data]="residualPlot.data"
  [layout]="residualPlot.layout"
  style="width: 100%; height: 400px">
</plotly-plot>
```

---

## Supporting multiple plots

As your app grows, you may need several plots:

* Residuals vs fitted
* Actual vs fitted
* QQ plot
* Cook’s distance

A good approach is to create one function per plot and store each result:

```ts
this.residualPlot = {...};
this.qqPlot = {...};
this.actualVsFittedPlot = {...};
```

This keeps your code clean and easy to manage.

---

## Handling multiple datasets

One challenge I faced was supporting multiple securities. Each one had its own set of plots.

Instead of overwriting data, I stored them in a map:

```ts
visualizationMap[security] = visualization;
```

Then when the user selects a security, I load its plots.

---

## Improving user experience

At first, all plots were shown in the main screen. This worked for one dataset but became messy with many.

A better solution was:

* Add a graph button in the table
* Open plots inside a popup
* Show charts only when needed

This keeps the interface clean and focused.

---

## Common issues to watch out for

Here are a few things that can go wrong:

### Undefined data

If you try to access `data.Fitted` instead of mapping an array, your plot will not render.

### Missing module

If Angular does not recognize `plotly-plot`, make sure you imported `PlotlyModule`.

## Final thoughts

Adding plots to an Angular app can greatly improve how users understand data. With a library like Plotly, you can build powerful and interactive visuals with little effort.

Start simple. Focus on one plot. Then grow your solution step by step.

In my case, moving charts into a popup and supporting multiple datasets made a big difference in usability.

---

Happy Coding!

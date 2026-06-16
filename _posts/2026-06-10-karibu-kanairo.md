---
layout: post
title: "I Built a Full Weather App for a Fake Job. Here Is the Whole Story."
date: 2026-06-10
author: Joseph Ndungi
tags: [career, scam, javascript, angular, nodejs, life]
categories: [Blog Post, Career]
canonical_url: "https://blogs.innova.co.ke/progress-is-hard-to-see/"
image:https://plus.unsplash.com/premium_photo-1664201889896-6a42c19e953a?q=80&w=1236&auto=format&fit=crop&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
description: I was welcomed in the city as they say
---

# I Built a Full Weather App for a Fake Job. Here Is the Whole Story.

*By Joseph Ndungi*

---

It was a regular Tuesday. I was on leave, casually scrolling through my phone, enjoying a quiet break from work when something caught my attention. My resume is on job sites. Passive job search, they call it. Plant seeds. See what grows.

What grew was a scam. A very well dressed, impressively engineered scam. I can only stan.


---

## Chapter 1: The Email That Made Me Feel Like a Main Character

The subject line said "Software Engineer Opportunity." I get those. I almost scrolled past it.

Then I read it.

Full stack developer. Remote. KES 80K to 180K per month. Angular. TypeScript. React. Node.js. Firebase. The stack read like someone had looked at my resume and built a job description around it. Which, in hindsight, they had. Because that is exactly what scammers do.

The company was called weather-ai.co. They had a website. They had API documentation. They had branding. They had, and I cannot stress this enough, a developer platform with actual working endpoints.

I applied. Obviously.

---

## Chapter 2: The Technical Challenge (Which Was Real and I Enjoyed It)

Here is the part that still gets me. They sent back a proper technical challenge.

> *"Your task is to build a simple application that integrates any of our APIs from our developer platform. We want to see how you consume our data and translate it into a clean, functional project."*

Forty eight hours. Public GitHub repo. Live deployment. The works.

Now, I could have built a simple fetch and display thing. A div with a temperature number. Call it a day. But no. I am a software engineer. I have standards. Also, I had decided this was a real job and I wanted it.

So I built **[Skyline](https://joseweather.netlify.app/)**.

Angular 17 on the frontend. Node.js and Express on the backend as a clean proxy so the API key never touches the browser. Seven background themes that shift based on weather conditions, clear day in blue, storms in near black, fog in grey. A scrollable hourly forecast strip. A seven day forecast with temperature range bars that scale relative to the week's global min and max, not absolute values, which is the detail that Apple Weather does that most people never notice but your brain does.

I spent time on the typography. I used font weight 200 for the main temperature number because that is what gives it that light, airy Apple Weather feeling. I named the app. I wrote a README. I set up environment files for dev and production. I added RxJS debouncing to the city search so it would not hammer the geocoding API on every keystroke.

For a fake job.

I submitted it. A live deployment link and everything.

The response came back within hours.

> *"Congratulations! Out of a massive pool of applicants, your engineering experience stood out."*

I would be lying if I said I did not feel a small surge of pride. My engineering experience stood out. From a massive pool. They said so.

---

## Chapter 3: The Offer Letter

A few days later, an offer letter arrived. Conditional, but an offer. It had my name on it. It had a salary figure. It had terms and conditions and a signature page and a deadline of two business days.

The salary was not what I wanted. The email mentioned this was the probation period rate and it would be reviewed. I asked about the post probation number. They were vague. I decided I was okay with it for now and signed.

I sent back the signed page.

They thanked me. They said the next step was a background check.

---

## Chapter 4: The Background Check (This Is Where It Goes Wrong)

The email contained a link. Verika.org. A background check service I had never heard of, which in itself is not unusual because there are many background check services. They also said, and read this carefully:

> *"Any extra costs during the screening will be reimbursed by the company when you attend the meeting in the office."*

And:

> *"You also have the option to use any background check website of your choice, and the company will reimburse you upon presentation of receipts."*

Reader, I knew.

Somewhere between reading those two sentences and clicking the link, a part of my brain that I clearly did not listen to said: this is the part. This is the part where it stops being real.

I paid anyway.

I do not fully understand my own psychology in that moment. Maybe it was the sunk cost of the technical challenge. Maybe it was the very convincing paper trail. Maybe it was that I genuinely wanted it to be real because I had built something I was proud of and I wanted that work to mean something.

It meant nothing. To them, anyway.

The certificate never came. The meeting at Nairobi Garage never happened. The reimbursement, predictably, did not materialise. Weather-ai.co, as a legitimate employer, does not exist.

---

## Chapter 5: What Actually Happened and Who These People Are

Let me be specific, because I promised specifics.

The company presenting itself as a legitimate employer is **weather-ai.co**. The background check link they sent was **https://verika.org/screen?ref=VRK-VDX2MS**. This link was described as unique to me and expiring in two days, a classic urgency tactic.

The weather API itself appears to be real, or at least functional enough to fool a developer for forty eight hours. Whether they built it as a prop specifically for this scam or whether it is a legitimate product being used as a front, I cannot say. What I can say is that no legitimate employer asks you to pay for a background check out of pocket with a promise of reimbursement later.

The in person meeting they proposed was at Nairobi Garage, Pinetree Plaza, Ngong Road. I did not attend.
---

## Chapter 6: The Lessons, Served With Minimal Moralising

I am not going to stand here and tell you I should have seen it coming, because I did see it coming and I ignored myself. That is a different problem entirely and one I am taking up with myself privately.

What I will tell you is the specific signals, not generic "be careful" advice, the actual things in this specific email chain that were wrong:

The background check fee. Legitimate companies pay for background checks. Full stop. Sterling, Checkr, HireRight, Truscreen, whichever vendor they use, they pay the vendor directly. They do not ask you to pay and promise to reimburse. Ever.

The "any website of your choice" option. No employer does this. Background checks need to meet specific legal and compliance standards. An employer who says "use whatever website you want" either does not understand compliance or does not intend to actually employ you.

Reimbursement contingent on physical attendance. You pay first, you get it back when you show up. This means you are financially and physically motivated to show up to a meeting with strangers. That is a trap with two possible outcomes and neither of them is good.

Vagueness about post probation salary. This one is on me. Ask the number before you sign. "Will be reviewed" is not a number.

---

## Chapter 7: The Part Where I Refuse to Be Fully Sad About This

Here is what is also true.

I built Skyline. It is a real, working, well architected application. The Angular components are clean. The backend is readable. The weather themes are genuinely nice. I learned the WeatherAI API inside out, found a mismatch between what their documentation described and what their endpoints actually returned, and debugged it in real time. I did a live deployment to Netlify and Render. I wrote a README that a junior developer could follow without my help.

None of that work disappeared because the job was fake. It is on my GitHub. It is in my portfolio. It is a thing I made.

The scammers got some money. I got a project. I paid to do my own work. Puta!

I am calling that a draw.

---

## What To Do If This Happens To You

Call your bank or mobile money provider immediately. Forwarded the message to Saf who shared PayStack contact but I know there will be zero help.

Report the job listing on whatever platform you found it. Take it down so the next person does not go through the same thing.

Attend any in-person meeting they arrange and beat them down. If you're delulu enough, you might actually meet them.

Search the company name plus the word scam before you sign anything. I searched after. Search before.

---

## Final Thought

These people are good at what they do. They built a functional API. They wrote convincing job descriptions. They ran a multi stage interview process. They produced an offer letter with the right formatting and the right language. They are putting serious effort into this.

The only thing they asked for at the end was a background check fee.

That is the tell. That will always be the tell.

Stay sharp out there.

---

*Joseph Ndungi is a software engineer based in Nairobi, Kenya. He builds things for financial institutions across Africa and occasionally for fake companies. You can find Skyline, the weather app he built for nobody, on his GitHub.*

👉 [Skyline](https://joseweather.netlify.app/)

👉 [Github Repo](https://github.com/Joseph-Ndungi/my-weatherapp)
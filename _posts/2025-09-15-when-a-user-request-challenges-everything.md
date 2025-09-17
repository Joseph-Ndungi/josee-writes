---
title: "When a User Request Challenges Everything"
date: 2025-09-15
categories: [Software Design]
tags: [Software Design]
canonical_url: ""
image: https://pbs.twimg.com/media/EwS1IBqW8AQMpnG?format=jpg&name=small
description: "We're talking about a recent challenge: a user request to limit editing of unapproved entries to the creator only. See how we navigated this and what we learned about balancing user needs with system security."
---


Hello there, fellow developers and QA pros! Ever get a request that makes you scratch your head and wonder, "Wait, is this really necessary?" We recently faced one of those, and it led to some deep dives into software design, critical thinking, and finding the right balance between user needs and system integrity.

Let's break down a recent user request and how we handled it.

### The Request: "Restrict Editing to the Creator"

The issue was simple on the surface: Users wanted to restrict editing of unapproved entries to their creator. In other words, if I create a record and it's pending approval, no one else should be able to edit it—only me.

As a techie, you hear a lot of wild stuff. The first reaction is often to reject it. After all, we have perfectly good systems in place, right? Our system is already role-based, with well-defined roles like creator, maker, and approver. We thought these roles provided all the sanity checks we needed. So, why did we even consider this?

#### Why a User Might Want This

Before rejecting any request, it's crucial to put on your critical thinking hat and understand the "why." Why would a user ask for this?

* Data Integrity and Ownership: A user might feel that if they created the entry, they have a certain ownership over it. They might be afraid someone else will make a change that they don't agree with, leading to inaccurate data.
* Avoiding Confusion: In a fast-paced environment, multiple people editing the same pending record could lead to confusion and errors. This request could be a way to avoid that chaos.

These are valid concerns. So, even if the request seems to go against our established system, we have to acknowledge the user's perspective.

### A Quick Look at our Existing System

Our current setup is pretty robust, with two key security settings:

1. Creator Cannot Complete: This is a classic separation of duties. The person who creates a transaction cannot be the one to approve or complete it. This is a crucial fraud prevention measure, especially in finance and other sensitive areas. It ensures two pairs of eyes are on every transaction, reducing the risk of a single person making a mistake or malicious change.

2. Maker-Checker (Creator can create and check transactions): This is often used in smaller teams. User creates the transaction, and approves it. It can be efficient for small teams where trust is high.

We initially felt these settings were more than enough. Adding a new, custom rule felt like an over-engineering.

### Why We Almost Rejected It

Our first instinct was to simply say "no." Here’s why:

* Role-Based System is King: Our whole security model is built on roles. A user’s permissions are defined by their role, not by who created a specific entry. Adding an exception for "creator-only editing" would add complexity to our codebase and could potentially create bugs.
* Potential for Bottlenecks: What if the creator is on vacation or leaves the company? The pending entry would be stuck, unable to be edited or approved. This could halt a critical workflow. This is a classic single point of failure problem.

We had already implemented this feature in a development environment and quickly saw these drawbacks firsthand. It was a good reminder that just because you can build something doesn't mean you should.

### The Middle Ground

Instead of a flat rejection, we decided to be more flexible. We introduced a new setting: "Restrict Editing of Unapproved Entries."

The key here is that it's a toggle. By default, it's disabled. This means that if a team wants this functionality, they can turn it on. For everyone else, our standard, robust role-based system remains the default.

This solution gave us the best of both worlds:

* We accommodated the user's request and showed that we were listening.
* We maintained the integrity of our core system.
* We avoided creating a new mandatory rule that could cause problems for other users.

This whole process was a great lesson in software design and critical thinking. It’s not about accepting or rejecting every request. It's about understanding the underlying need, evaluating the pros and cons, and finding a solution that is both flexible and robust. Sometimes, the best solution isn't to build a new feature from scratch, but to build a toggle that empowers users to customize their experience without compromising the system's core principles.

Happy Coding!

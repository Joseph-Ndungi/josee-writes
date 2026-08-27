---
title: "When Sandbox Testing Starts Feeling Too Real: Integrating PesaPal"
date: 2026-08-25
Categories: [Backend, Payments, Integration]
Tags: [PesaPal, Payments, Angular, .NET, API Integration, Payment Gateway, Technical Debt]
description: "Integrating PesaPal into a payment service, embedding the checkout in Angular, handling IPN callbacks and payment status polling, and discovering that sandbox testing can sometimes feel a little too real."
image: https://images.unsplash.com/photo-1423666639041-f56000c27a9a?q=80&w=1174&auto=format&fit=crop&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
---

Absolutely. I’d frame it as a short engineering/blog update: what you built, what worked, the iframe/callback challenge, the remaining technical debt, and the funny sandbox-money discovery.

# Integrating PesaPal: When Sandbox Testing Starts Feeling Too Real

Been integrating **PesaPal** into our payment service and have spent quite some time testing with the sandbox.

The integration started fairly straightforward: create a billing session, send the selected subscription plan and customer details to the payment service, create the PesaPal order, and return the payment URL to the frontend.

From there, I wanted the payment experience to stay within the application rather than sending users away to a completely separate page. So I worked on embedding the PesaPal checkout inside an iframe, while keeping the backend responsible for confirming the actual payment status.

The final flow looks roughly like this:

**Application → Payment Service → PesaPal → IPN/Callback → Payment Status → Application**

On the frontend, the application starts polling the payment service after the transaction is initialized. Once the backend confirms that the payment was successful, the application refreshes the user's session and activates the subscription.

There were a few interesting challenges along the way.

The first was getting the PesaPal URL safely into an Angular iframe. Angular doesn't allow arbitrary external resource URLs without explicitly trusting them, so the payment URL needed to be handled through `DomSanitizer`.

Then came the callback/IPN confusion.

The PesaPal callback and IPN serve different purposes, and initially the backend IPN endpoint was also being used as the callback URL. That meant that once payment completed, the iframe could end up navigating to the backend endpoint and exposing the development tunnel page instead of returning cleanly to the application.

The solution is to separate the responsibilities:

* **IPN** → backend notification and payment processing.
* **Callback URL** → customer/browser return location.
* **Payment polling** → application-side confirmation of the actual payment status.

There was also a small but classic bug hiding in the payment polling logic.

The backend was returning:

```text
Successful
```

while the frontend was checking for:

```text
success
```

After converting the response to lowercase, the frontend was receiving:

```text
successful
```

but checking against:

```text
success
```

So the payment was successfully processed, the backend knew about it, but the frontend never entered the success handler.

A good reminder that sometimes the hardest bugs are two strings that simply don't agree. 😂

### What's working

At this point, the core payment flow is working:

* Subscription plan selected.
* Billing session initialized.
* PesaPal payment created.
* Payment displayed in the application.
* Payment status tracked by the backend.
* Successful payment detected by the frontend.
* OIDC session refreshed.
* User redirected to the dashboard.

There are still a few rough edges and some technical debt that I'll clean up early tomorrow morning.

The main one is making the callback/iframe flow cleaner and more resilient across local development, tunnels, and production. I'll also tighten up the payment state handling, cancellation flow, timeout handling, and some of the duplicated frontend logic.

For now, though, the important part is that the end-to-end payment flow is working.

## And then there's the sandbox...

The documentation says the sandbox uses simulated funds and that no real money is involved.

But in my experience, actual money has been moving during testing.

At this point, the test transactions are adding up, so I am definitely becoming more interested in the refund process. 😂

According to the docs, refunds require merchant approval and are then processed by their finance team. So unless I add code to trigger the refund process after every successful transaction, testing this repeatedly is starting to become an expensive way of saying:

> "The integration works."

😅

Still testing, but definitely keeping a closer eye on those transactions now.

## Final thoughts

Payment integrations are one of those things that look deceptively simple from the outside.

Create an order. Get a payment URL. Let the customer pay. Confirm the transaction.

Then you discover callbacks, IPNs, redirects, iframes, authentication sessions, polling, browser security policies, development tunnels, and a surprising number of opportunities to accidentally make two strings that should match... not match.

Still, it's been a useful integration exercise, and the core flow is now in place.

**Technical debt:** clean up the callback architecture, improve payment state management, handle the remaining edge cases, and make the development/production configuration cleaner.

That can be tomorrow morning's problem. 😂

Happy Coding!
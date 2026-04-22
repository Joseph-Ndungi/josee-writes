---
title: "Building a Live Data Dashboard with FastAPI and Angular"
date: 2026-04-20
categories: [FastAPI, Angular, Real-Time Apps]
tags: [FastAPI, Angular, Server-Sent Events, Real-Time Data, Data Visualization]
canonical_url: "https://dev.to/josephndungi/building-a-live-data-dashboard-with-fastapi-and-angular"
image: https://plus.unsplash.com/premium_photo-1681487769650-a0c3fbaed85a?q=80&w=1332&auto=format&fit=crop&ixlib=rb-4.0.3
description: "A mini project demonstrating how to stream live sensor or financial data using FastAPI and Server-Sent Events, and visualize it in an Angular dashboard with dynamic charts."
---


# 📝 Pulse Live Dashboard

### Building Real Time Systems with FastAPI and Angular

---

## 1.Scenario

Imagine you are watching a dashboard that shows asset prices. The market is moving fast. A price spike happens, and you want to react immediately.

But your dashboard is not updating in real time.

You refresh the page. Nothing changes. You refresh again. Now the data finally updates but the opportunity is already gone.

This is not just a poor user experience, it is a real business problem.

To solve this, many systems today still use **polling**, where the client repeatedly asks the server for updates at fixed intervals. Polling means the system keeps checking again and again: “Has anything changed yet?” even when nothing has.

This approach introduces delay and unnecessary requests. In fast-moving systems, that delay can be costly.

This is where a better approach like **Server-Sent Events (SSE)** comes in.

---

## 2.Polling

What usually happens in most dashboards?

The frontend sends a request every few seconds:

"Do you have new data?"

The server responds:

"No, not yet."

A few seconds later:

"Do you have new data?"

At small scale, it works fine. But at large scale, it becomes a problem.


### Why Polling Becomes Expensive

Imagine this:

* 10,000 users
* Each user sends a request every 2 seconds

That is **5,000 requests per second** hitting your backend.

Most of these requests return nothing new.

So what happens?

* CPU usage increases
* Database load increases
* Network traffic increases
* Latency increases

And ironically, users still do not get real time data.


Think about a user creating a new Google account and typing an email address during the signup process. What is the usual system behavior while the user is typing, and how is the input typically handled in real time within a well-designed system?

Does every keystroke triggers a validation request?

---

## 3. Rethinking the Model

Instead of repeatedly asking “Is there new data?”, the client keeps an open connection and simply waits while the server pushes updates whenever something changes.
 In other words, the server says, “I will tell you when there is new data,” and actively sends updates over time without being prompted each time. 
 This approach is called **Server-Sent Events (SSEs)**—a web standard that allows a server to stream real-time updates to a client over a single, long-lived HTTP connection. It removes the need for constant polling and makes real-time communication more efficient and lightweight.

---

## 4. What Are Server Sent Events

Server Sent Events allow a server to send data to the client continuously over a single HTTP connection.

The connection stays open.

The server pushes updates whenever they happen.

The client listens.


### How It Works

* The client opens a connection to the server
* The server keeps the connection open
* The server sends updates as they occur
* The browser automatically reconnects if the connection drops

There is no repeated request cycle.


### Why This Is Powerful

* No unnecessary requests
* Lower latency
* Better performance
* Simpler than WebSockets for one way data

---

## 5. Choosing the Right Tool

When designing real time systems, we usually consider three options.


### REST APIs

* Simple and widely used
* Request response model
* Not suitable for live updates


### WebSockets

* Full duplex communication
* Very powerful
* More complex to implement and maintain


### Server Sent Events

* One way communication from server to client
* Lightweight
* Built on standard HTTP
* Perfect for dashboards and monitoring


### Our Choice

For this mini-project, we only need data to flow from server to client.

So Server Sent Events are the perfect fit.

---

## 6.  Live Dashboard

Built a mini system that simulates a real time trading dashboard.

The system has two parts:

* **Backend using FastAPI**: A Python-based API server that generates and streams synthetic market data.
* **Frontend using Angular**: A TypeScript-based SPA that consumes the stream and displays live updates.

The tech stack includes:
- **FastAPI** (Python): For the backend API and SSE streaming.
- **Angular** (TypeScript): For the frontend UI and reactive data handling.
- **RxJS** (JavaScript): For managing asynchronous data streams in Angular.
- **Tailwind CSS**: For styling the dashboard.
- **Server-Sent Events (SSE)**: The core real-time communication protocol.
- **EventSource API**: Browser-native API for consuming SSE.

---

## 7. Backend Deep Dive

We chose FastAPI for a reason.

FastAPI is a modern, high-performance web framework for building APIs with Python. It's built on top of Starlette (an ASGI framework) and Pydantic (for data validation), making it ideal for asynchronous operations and real-time features like streaming. Unlike older frameworks like Flask or Django, FastAPI leverages Python's async/await syntax natively, which is crucial for handling concurrent connections without blocking threads. This makes it perfect for Server-Sent Events, where the server needs to maintain long-lived connections and push data continuously.

In our implementation, FastAPI handles:
- CORS middleware for cross-origin requests (essential since the frontend runs on a different port)
- A single `/stream` endpoint that returns a `StreamingResponse` with `media_type="text/event-stream"`
- Asynchronous event generation using a Python generator function

The core streaming logic uses an async generator that yields SSE-formatted data every second, simulating live market data.


It continuously sends data like this:

* Asset name
* Price
* Percentage change
* Timestamp


### How the Stream Works

We use a generator function.

This function runs in a loop.

```py
ASSETS = ["AAPL", "TSLA", "BTC", "ETH", "GOOGL"]

async def event_generator():
    while True:
        await asyncio.sleep(2)

        asset = random.choice(ASSETS)
        price = round(random.uniform(100, 1500), 2)
        change = round(random.uniform(-5, 5), 2)

        data = {
            "asset": asset,
            "price": price,
            "change": change,
            "timestamp": datetime.utcnow().isoformat()
        }

        yield f"data: {json.dumps(data)}\n\n"
```

Every second:

* It creates new synthetic data
* It formats the data
* It sends it to the client

The response type is:

```
text/event-stream
```

This tells the browser to treat the response as a stream.


### Why This Matters

* The connection stays open
* Data flows continuously
* No repeated requests

This is the foundation of real time systems.

---

## 8. Frontend Deep Dive

On the frontend, we take a different approach.

We do not use HTTP calls.

We use the browser built in **EventSource API**.

```ts
 this.source = new EventSource('http://localhost:8000/stream');

    this.source.onmessage = (event) => {
      const data = JSON.parse(event.data);

      this.zone.run(() => {
        this.assetSubject.next(data);
      });
    };
```


### What EventSource Does

EventSource is a native JavaScript API that establishes a persistent HTTP connection to a server endpoint. It's specifically designed for Server-Sent Events and handles the connection lifecycle automatically—including reconnection on failures. When the server sends data, EventSource fires a `message` event, allowing the client to process updates in real-time without polling.


### Angular Integration

Angular is a TypeScript-based framework for building scalable single-page applications (SPAs). It uses a component-based architecture with dependency injection, making it great for modular, maintainable code. In our project, we integrate EventSource within an Angular service that wraps it in RxJS (Reactive Extensions for JavaScript), a library for handling asynchronous data streams.

The service:
- Opens the EventSource connection to `http://localhost:8000/stream`
- Parses incoming JSON data from the SSE stream
- Uses `NgZone` to ensure Angular's change detection runs properly (since EventSource callbacks are outside Angular's zone)
- Emits data via an RxJS `Subject`, which components can subscribe to reactively

Components subscribe to this observable stream, updating the UI automatically as new data arrives. This reactive pattern keeps the architecture clean and efficient.


### What the User Sees

* Live asset updates
* Smooth transitions
* Color coded gains and losses
* No refresh required

The interface feels alive.

---

## 9. Performance and Scalability

This approach is not just about visuals.

Below are some engineering benefits.


### Reduced Server Load

Instead of thousands of repeated requests:

* One connection per client
* Data sent only when needed


### Lower Latency

Data arrives instantly.

No waiting for the next polling cycle.


### Better Resource Usage

* Less CPU usage
* Less network traffic
* Efficient memory usage

---

## 10. Real World Applications

This pattern is useful in many systems.


### Financial Systems

* Live price feeds
* Trading dashboards
* Risk monitoring


### Sensor Systems

* IoT devices
* Environmental monitoring
* Smart infrastructure


### Internal Tools

* System health dashboards
* Logs and alerts
* Operational monitoring


## 11. Limitations to Consider

Server Sent Events have some limitations.

---

### One Way Communication

Data flows only from server to client.

If you need two way communication, WebSockets are better.

### Browser Limits

Some browsers limit the number of open connections.

### Not Ideal for Binary Data

SSE is text based.

## 12. Lessons Learned

From this project, we learn a few key ideas.

Do not always ask the server for data. Let the server push data when ready.

Do not jump to complex solutions. SSE solves many problems with less effort.

Users notice responsiveness. Real time updates improve trust and usability.

---

## 14. Final Thoughts

This project may be small in scope, but the underlying idea is powerful and widely applicable. We have taken a clear step away from traditional systems that rely on waiting, refreshing, or repeatedly asking for updates, and moved toward systems that respond the moment data changes. This shift from a reactive pull model to a real time push model represents a fundamental change in how modern applications are designed. It improves performance, reduces unnecessary load, and most importantly, creates a more immediate and intuitive user experience. As systems continue to evolve toward real time interaction, this approach becomes not just an optimization, but a standard expectation for modern software.

---

## 16. How to Run the Project

👉[GitHub Repository](https://github.com/Joseph-Ndungi/pulse-dashboard.git)

To see the Pulse Live Dashboard in action:

### Backend Setup
1. Navigate to the `backend/` folder.
2. Create a virtual environment: `python -m venv venv` (or use your preferred method).
3. Activate it: `venv\Scripts\activate` (Windows) or `source venv/bin/activate` (Linux/Mac).
4. Install dependencies: `pip install -r requirements.txt`.
5. Run the server: `uvicorn app.main:app --reload --host 0.0.0.0 --port 8000`.

### Frontend Setup
1. Navigate to the `pulse-dashboard/` folder.
2. Install dependencies: `npm install`.
3. Run the development server: `ng serve`.
4. Open `http://localhost:4200` in your browser.

The dashboard will connect to the backend and start displaying live asset updates every second.

---

Happy Coding!

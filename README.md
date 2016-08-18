# Wampum
An asynchronous task library in C++, drawing inspiration from Node.js single threaded async call-back model. Trying to find a middle ground between thread based processing and event driven approach.

## Threads vs Events

The debate between threads and events is more than a decade old. Yet it shows no signs of dying down. 

A thread based system dedicates one thread to process a request. This offers a relatively simple sequential processing model that is often easier for programmers to compose and understand.  On the other hand, such systems must maintain large amount of threads to achieve high throughput, which often result in significant cost in memory, OS thread scheduling and context switches.

On the other hand, in an event-driven system, handlers of events (e.g. arrival of requests, completion of disk read) are processed in one or more event queues. The number of event queues in a system is often tied to number of CPU cores, relatively a small cost comparing to thread based systems.

Unfortunately, programming an event-driven system is difficult, as the programming model is no longer sequential. The logic for processing a single request must be divided into multiple event handlers, often in a non-intuitive way, resulting code harder to understand and maintain.

> 1995: Why Threads are a Bad Idea (for most purposes), John Ousterhout, (UC Berkeley, Sun Labs)

> 2001: SEDA: An Architecture for Well-Conditioned, Scalable Internet Services, Staged, Event-driven Architecture, M. Welsh, D. Culler, and Eric Brewer (UC Berkeley)

> 2003: Why Events are a Bad Idea (for high-concurrency servers), R. van Behren, J. Condit, Eric Brewer (UC Berkeley)

## Async Callbacks and “Single Threaded” Processing

Parallel programming is difficult. So Node.js runs your code in one thread. This thread does not wait for I/O though, as all long running I/O operations are async and handled by a separate worker thread pool. 
This seems to be a nice compromise between even-driven and threaded approach. Business logic for processing a request is executed in one thread, and long running I/O operations are handled asynchronously, in an event-based approach. The closure in Javascript allows you to put the logic before and after an I/O operation together, improving readability. The following code read file `/etc/hosts` and dump the content to console.
```
fs = require('fs')
fs.readFile('/etc/hosts', 'utf8', function (err,data) {
  if (err) {
    return console.log(err);
  }
  console.log(data);
});
```

## Wampum: Asynchronous Sequential Processing

One observation we make here is that we can avoid the headache of parallel programming as long as the business logic execute sequentially. It does not have to be in the same thread. Here in Wampum we define a concept as an “Activity”, which you can think of as a virtual thread. 

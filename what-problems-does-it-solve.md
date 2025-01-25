# What Problems Does It Solve?

### Poor UX for Async - Cancelability, Concurrency Control, Progress Notifications

Tools like Promises have a hard time delivering top-notch UX experiences around async processes. Users will feel less confused, and more in control, if operations are canceled when no longer needed, race conditions are prevented declaratively, and progress can be reflected incrementally vs all-at-the-end. Promises, as a data-type, lack the fundamental properties of cancelability, or incremental delivery, so we need additional developer tools that embrace all the features of async, without having to put custom glue code together each time an edge case like these arise.

### Framework-dependence

In the early web, tools like JQuery and Backbone worked in an event-oriented way, just as the DOM itself did. Now with Angular, React, and other  'modern' frameworks, the paradigm became one of 'reactive state'. But each framework has their own kind of reactive state, and they are not compatible with each other. Even though we all use Javascript to alter the DOM, we don't speak the same language!

𝗥𝘅𝑓𝑥 breaks down silos by emphasizing _"Events in the Core, State at the Edge"_. The good news: event bus skills and tooling cannot be obsoleted by a framework change! This is how developers can gain portable skills - and say - port a React app to an Angular or Vue one in 30 minutes or less.

### Race Conditions

Timing problems in async arise all the time. A typeahead component may display an old result. Two AJAX requests may reach the server in the opposite order they were sent. Since each Web framework has a different set of tools for handling these problems (none of them complete!), these issues are often not spotted until code has reached production. When they are addressed, it's with error-prone, architecturally significant changes.

𝗥𝘅𝑓𝑥 uses the power of RxJS—a mature, performant library used more than any single web framework—to allow for _declarative_ strategies for stopping race conditions before they are allowed to happen. You can change to a queueing strategy, if that solves your problem, without declaring a new variable to hold the queue or making any changes other than plugging in the strategy!\
Read more in Concurrency Modes.

### Resource Leaks

If you add an Event Listener, you should remove it. If you start an HTTP request, or open a web socket that is no longer needed, you should close it. Todays Promise-based code does not handle cancelation very well. While it suffices for many apps to always let AJAX requests run to completion, tuning performance means plugging these resource leaks.

𝗥𝘅𝑓𝑥 lets you return an Observable where you would have returned a Promise, and cancelation is handled for you automatically! Components, when they unmount, can transparently cancel any effects they have in progress. This makes resource leaks the exception, not the norm.

### Request-Response Limitations

Because the Web evolved from a document-linking platform, and async functions evolved from synchronous functions, the assumption we make in Web Development is that one request always begets one response ( 1 request < - > 1 response ). By choosing this subset of real behaviors to support in our tools, we have hurt our ability to model everyday constructs like:

* Streaming stock tickers, or recurring things like subscriptions or salaries
* Full-duplex communication like WebSockets
* Response-less (daemon-like) or batched-response endpoints

𝗥𝘅𝑓𝑥 can speak to any type of service just as readily as a REST server, and switch between types (eg REST to WebSocket) with _no impact on architecture_. By simply responding to events, in whatever timing they arrive, an 𝗥𝘅𝑓𝑥 listener seamlessly upgrades from REST to real-time; from JIRA to Trello. Read on to see how an Event Bus is the perfect way to decouple request events from response events.

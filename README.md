---
description: >-
  RxFx is async JavaScript - done better. A way to execute and react to
  asynchronous effects that can be used Web, Server or Mobile. Built with RxJS,
  usable by anyone.
---

# RxFx

RxFx uses a few principles to enable _simpler_ async coding than in _any_ modern front-end framework. When you can do work in raw "Vanilla JS", the amount of each front-end framework you need can shrink. This will enable better testing, easier porting to new frameworks, and more reliable code with easy cancelation and race condition prevention.

RxFx' goals are achievable when you follow these core principles:

- Decouple event publishers and subscribers (triggerers and listeners).
- Prefer triggering events to invoking functions.
- _"Return The Work"_ from event handlers—as Observables when possible.
- Use declarative Concurrency Modes to eliminate race conditions.

Read on for examples, or view the API docs/READMEs of the sub-libraries to understand how this is done.

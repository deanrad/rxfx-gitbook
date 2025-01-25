---
description: >-
  As a frontend developer, you'll probably want to get started by creating a
  service from the @rxfx/service library, then setting your framework to make
  requests of it, and react to it.
---

# How do I get started?

From its [README](https://github.com/deanrad/rxfx):

A service is both an effect manager, and a reducer of state from the lifecycle actions of the effect. It is also a bus listener, thus amenable to being logged, or stopped, from the bus level, not just its own individual API.

```
import { createService } from '@rxfx/service';
export const uploader = createService<File,void>(
  'uploader',                              // a name, for logging
  (file) => fetch({ method: POST, file }) // an effect to be run on .request()
);
```

As an Effect manager, it provides an API `service.request()` to request an execution of an event handler (subject to the concurrency mode).

```
const file:File;
uploader.request(file);   // or uploader(file);
```

To monitor the activity of the service, without creating your own custom state fields - in React you would use `useService(service)` and you'd have a state field you could drive the UI from. In plain JS, `service.isActive.subscribe(displayStatus)` could call a function that displays the current status as it changes.

```
import { uploader } from './services/uploader';
import { useService } from '@rxfx/react';

function UploadStatus() {
  const { isActive } = useService(uploader);
  return <span>{ isActive ? "⏳" : ""}</span>
}
```

The handler returns either a Promise or an Observable, and so can be an `async` function. (It may also return an Iterable, if its result is determinable synchronously - a less common but supported situation). It provides an API `service.cancelCurrent()` - which if the handler returned an Observable instead of a Promise, will cancel and run the handler's teardown function. Promise-based functions that can be canceled on a AbortSignal can be made into cancelable handlers via `makeAbortableHandler` from `@rxfx/ajax.` In either case, Promise, or Observable, no further `next` or `complete` events from a handling will be raised after a call to `cancelCurrent()`

If the service was created as a queueing one via `createQueueingService()`, `cancelCurrentAndQueued()` will empty the queue as well as cancel any existing handling, otherwise `cancelCurrent()` would just progress to the next handling in the queue.

Any concurrency mode - Immediate, Queueing, Switching, Blocking, and Toggling can be applied merely by changing which service factory function is called - e.g. `createSwitchingService()`

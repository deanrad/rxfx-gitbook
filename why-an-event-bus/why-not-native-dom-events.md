# Why Not Native DOM Events?

In short—portability. To have an Event Bus that works just as well in the DOM as on React Native, in NodeJS or Deno makes it usable across even more teams, breaking down more silos.

Also, DOM Native event listeners don't have built-in the notion of async handlers and Concurrency Strategies, since they don't _"Return The Work"_.

Lastly, the bubble/capture model, and `stopPropogation` and `preventDefault` methods on events are not really needed once the source of events is abstracted out of the DOM, so you get a simpler API with the Event Bus abstraction.

Just `trigger`, `filter` and `listen` your way to framework-independent effect and state management!

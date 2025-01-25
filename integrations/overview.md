# Overview

{% hint style="info" %}
Integrating with any Web Library or Framework simply involves sending events into 𝗥𝘅𝑓𝑥 and subscribing to the event bus for when to update the UI.
{% endhint %}

𝗥𝘅𝑓𝑥 is a container for async side effects that is framework independent. That is to say - it only depends upon RxJS - a more stable library than any major version of a modern framework. But if you are in a React codebase, for example, you ultimately need to hand React what it needs to update the UI when a listener responds.

𝗥𝘅𝑓𝑥 has a demo app - an alarm clock - that has been integrated with:

* React ([CodeSandbox](https://codesandbox.io/s/react-omnibus-alarm-clock-h2op8l))
* Angular ([CodeSandbox](https://codesandbox.io/s/angular-omnibus-alarm-clock-xz7276))
* Vue ([CodeSandbox](https://codesandbox.io/s/vue-omnibus-alarm-clock-ukb1kh))
* Svelte ([CodeSandbox](https://codesandbox.io/s/svelte-omnibus-alarm-clock-8kkuz9))

This demo app uses the higher-level `createService` interface, where events are sent using `service(request)`, and state updates recieved via `service.state.subscribe()`. But if you use the lower level `bus.trigger(req)` and `bus.listen()` interfaces, you will still be able to send updates into your framework.

Read on for the framework-specific APIs you can use to start leveraging 𝗥𝘅𝑓𝑥 and RxJS inside of your specific UI framework.

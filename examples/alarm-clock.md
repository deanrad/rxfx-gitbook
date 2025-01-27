# Alarm Clock

{% hint style="info" %}
**Objective:** Build an alarm clock which has a 'modal' interaction style, allowing time changes only while a certain button is pressed.\
**Concepts:** `after`, modeling, Observables. blocking, Services.
{% endhint %}

For this demo, we'll show how you may use RxFx to make an alarm clock. This time, we'll start from the VueJS framework, then we'll make the service, starting with a mocked Observable of user behavior, to make sure it works. Then we'll plug in the real user behavior and be done!\
(This web-based alarm clock works like many real-life alarm clocks. The H and M keys increment the time in hours and minutes, but only while the 'Set Time' button is pressed, watch [a video of it being built](https://youtu.be/JAlct1jZ4h8), or read on!\
Also, the core time-setting functionality is framework independent, so once you have it built, you can easily port it to React, Angular, Vue or Svelte!

### The API to an RxFx Service

Instead of modeling a 'setting time' state, or thinking about an `isSetting` boolean, we'll use an RxFx Service called the `timeSetter`. The Service will encapsulate the time and listen for keypresses to change it. And the service will notify us of changes so we can pass those to the UI. We'll invoke the `timeSetter` service when "Set Time" button is pressed, and cancel it upon release. Here's the Vue code that does this.

```javascript
<button @mousedown="allowAdjustment" @mouseup="exitAdjustment">
  set <b>T</b>ime
</button>
import { timeSetter } from "../services/timeSetter";
export default {
  methods: {
    allowAdjustment() {
      bus.trigger(timeSetter.actions.request());
    },
    exitAdjustment() {
      bus.trigger(timeSetter.actions.cancel());
    },
  },
  mounted() {
    timeSetter.state.subscribe((state) => {
      const { hour, min } = state;
      liveState.value.hour = hour;
      liveState.value.min = min;
    });
  }
};

```

That's all we need to do to install our service into a Vue component! And it would be only slightly different in React. Now we'll model time itself, and what events can change it.

\
We need an initial time, and a way to indicate that an `H` keypress advances the time an hour, and an `M` by a minute. Let's build a pure reducer, which will apply events to the time in a framework-independent way.

```javascript
export const initialState = {
  hour: new Date().getHours(),
  min: new Date().getMinutes(),
};

const reducer = (s = initialState, inc) => {
  switch (inc?.payload?.field) {
    case "h":
      return { hour: (s.hour + 1) % 24, min: s.min };
    case "m":
      return { hour: s.hour, min: (s.min + 1) % 60 };
    default:
      return s;
  }
};
```

We've coded the reducer to expect actions with a `payload.field` property that is either `H`, or `M.` The output of this reducer - the new time - will be pushed out to VueJS via the `timeService.state` Observable.

To see the service in action, let's first create the service with the `increment` namespace, a mock of actions, and our reducer:

```javascript
function handleTimeSetMock() {
  return concat(
    after(0, { field: "m" }),
    after(1000, { field: "m" }),
    after(1000, { field: "m" })
    // after(1000, interval(100).pipe(mapTo({ field: "m" })))
  );
}
export const timeSetter = createBlockingService(
  "increment",
  bus,
  // An Observable of payloads of actions, with the specified timing
  handleTimeSetMock,
  () => reducer
);
```

The `concat`-enation of payloads delayed via `after` will simulate multiple keypresses over time. Now, we hold in the Set Time button, and we see the minutes incrementing - cool! Check it out in the sandbox, or see the relevant trace here:

```
increment/request
increment/started // 10:00 PM
increment/next  {field: "m"} // 10:01 PM
increment/next  {field: "m"} // 10:02 PM
increment/next  {field: "m"} // 10:03 PM
increment/cancel
increment/complete
```

Getting real keypresses is just a finishing detail now - we've already seen our system works :) Try it out here, and see in the comments some alternate implementations of real listening for `H` and `M` keypresses. As an exercise, try and change the key repeat rate the alarm clock accepts!

{% embed url="https://codesandbox.io/embed/vue-omnibus-alarm-clock-ukb1kh?fontsize=14&hidenavigation=1&theme=dark" %}

The service handler that uses real events is shown below. Because of RxJS, it is automatically leak-proof and cancelable, and because our service is defined with `createBlockingService` there will never be more than one listener or time setter at a time!

```javascript
function handleTimeSetFromDOMEvents() {
  return fromEvent(document, "keydown").pipe(
    filter(({ key }) => key === "h" || key === "m"),
    map(({ key }) => ({ field: key }))
  );
}
```

### Done! It's About Time :)

In this demo we've seen how Observables triggered by a Service can provide events to reducers to produce a live-updating state. And we saw how to tie this into the VueJS framework, using no particulars from Vue, except how to get the state from the Service into a Vue `liveRef`. We've also seen that this event-production style with RxFx stays very close to the language of the requirements, with few extraneous details.

> Hour and minute buttons (H and M keys here) increment the time, only while the 'Set Time' button is held down.

Also, the clean separation from the UI framework is nice. See how little changes when its ported to React, Angular, Vue or Svelte!

Now, let's take a step up in complexity and see what challenges the TodoMVC demo app has in store for us, and let's even try a new UI framework this time!

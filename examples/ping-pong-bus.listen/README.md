# Ping Pong (bus.listen)

{% hint style="info" %}
**Objective:** Set up 2 listeners that PING and PONG each other

**Concepts:** effects, listen, trigger, spy, after, Observables, unsubscribe
{% endhint %}

The simplest use of an event bus would trigger one event to a single listener. But the bus really excels when multiple parts of your app are communicating over it, enjoying decoupling, error isolation, cancelation and all those other bus benefits. So our Hello World will set two listeners against each other in a game! We'll start with ping-pong in a console, and end up with soccer in a browser looking like this:

{% embed url="https://codesandbox.io/embed/omnibus-ping-pong-6lzvdv?fontsize=14&hidenavigation=1&theme=dark" %}

As the console shows, the heart of this app can be seen in its output where, after a first PING, each return follows 2 seconds apart.

```
PING
PONG
PING
PONG
PING
Player 1: bye!
```

We'll build this up in NodeJS, then hook it up to the DOM. Read on, or watch it be done in realtime:\
\
\- NodeJS Video Walkthrough ([Youtube](https://www.youtube.com/watch?v=9C7NuIgToQ8), 4:15)

### Actor Model

While not a literal implementation of the Actor Model, RxFx supports listeners sending messages to each other, so we can consider our ping pong match as two actor-listeners with the following specs:

```
Describe: Player1 ("PING" listener)
  It: triggers "PONG" 1 second after it hears "PING"
Describe: Player2 ("PONG" listener)
  It: triggers "PING" 1 second after it hears "PONG"
```

First let's use a bus that supports strings, and add a spy to it to log everything to the console.

```javascript
const bus = new Bus();
bus.spy((e) => {
  console.log("Bus: " + e);
});
```

See Spies, filters and guards later for more information on these functions, but let's keep going.

#### Player 1 - Explicitly Triggering

If we trigger a PING now, the spy will log it to the console. But—we don't have a game yet until a listener can return the ball with a PONG. This listener will use an async utility to wait 1000 ms, then explicitly trigger a new action (which is already logged).

```javascript
const player1 = bus.listen(
  (e) => e === "PING",
  () => delay(1000, () => bus.trigger("PONG"))
);
bus.trigger("PING");

function delay(ms, fn) {
  return new Promise((resolve) => setTimeout(() => resolve(fn()), ms));
}
// Output:
// PING
// PONG
```

We're underway now! The criteria for the listener to run is that a bus item arrives that equals PING. The handler function to be run on those events triggers a PONG after 1 second. Easy!

\
You may have noticed the handler function returns the result of calling `delay`. Because `delay` returns a Promise that runs immediately - we didn't have to. But in RxFx, generally handlers "return the work" rather than explicitly starting it. This will make more sense when we return an Observable from a handler or apply Concurrency Modes.

#### Player 2 - Implicitly Triggering

To keep the loop going we'll use another player that puts a PING on the bus. This handler will be different - it won't call `bus.trigger`. Instead, it will simply resolve to PING in 1 second, and another argument to the listener will capture the PING and trigger it. Like this:

```javascript
const player2 = bus.listen(
  (e) => e === "PONG",
  () => delay(1000, () => "PONG"),
  bus.observeAll() // our handler results should be triggered :)
);
```

Keeping handler functions (the 2nd argument to `listen`) free of RxFx details makes them generic, and easy to stub out in tests. But how do their values get back onto the bus? `bus.observeAll()` returns an Observer object. Read more about how this works in Observers and Retriggering.

At this point, with two listeners, we have a program that will run forever! While that's cool for real ping-pong, that's not what we're after here.

#### End the match: Observable-style

How do we end the match? Simple - `player1`'s listener stops listening! To do this we call `player1.unsubscribe()`. Instead of the Promise-based `delay` , we'll use a function called `after` to demystify using an Observable. All together we have:

```javascript
after(5000, () => {
  player1.unsubscribe();
  console.log("Player 1: bye!");
}).subscribe();
```

The price of using an Observable is that—in general—we need to call `subscribe` on it. But it does indeed tell player1 to stop. However, our program now does the following:

```
PING
PONG
PING
PONG
PING
Player 1: bye!
PONG
PING
```

Oh no, what happened?! We told player1 to stop - **but** the Promise they started (to trigger PONG) didn't know this. So player2 got the ball back and returned it. But fear not! With one small change, we can bind the lifetime of the handler (the delayed PONG) to the listener, so it is cleanly shut down. Just like this:

```diff
const player1 = bus.listen(
  e => e==="PING",
- () => delay(1000, () => bus.trigger('PONG')) // Promise - uncancelable
+ () => after(1000, () => bus.trigger('PONG')) // Observable - cancelable!
);
```

We simply changed the return value of the handler from an un-cancelable Promise to a cancelable Observable. And now we have the clean result we wanted :)

```
PING
PONG
PING
PONG
PING
Player 1: bye!
```

### Pure NodeJS

This StackBlitz shows you the working tutorial up to this point.\
[https://stackblitz.com/edit/node-znwrdt?file=index.js](https://stackblitz.com/edit/node-znwrdt?file=index.js)

### VanillaJS - It Comes To Life!

The real fun of RxFx is building the behavior outside of any UI Framework, knowing its correct, then gluing it up to a browser to get the visuals you desire. See this CodeSandbox to view this tutorial brought to life in a browser using only Vanilla JS. You may need to click Refresh 🔄 in the lower left to restart it.

{% embed url="https://codesandbox.io/embed/omnibus-ping-pong-6lzvdv?fontsize=14&hidenavigation=1&theme=dark&view=preview" %}

### Now get building!

Imagine how productive your teams could be if async JS experts worked on the core logic _in parallel_ with designers working on the styles! This is part of the benefit a decoupling event bus brings to your apps and your teams.

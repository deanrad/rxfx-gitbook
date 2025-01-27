# How does RxFx simplify working with Observables?

For many developers, Observables are not easily used. They take the simplicity of a Promise: just calling `await` to capture its value, and make you call `subscribe` to start them running. The challenge with `subscribe()` - is mainly - Where do you put your `Subscription` objects returned by it, which are needed for cancelation?&#x20;

RxFx has you covered - by basically using a layer that calls `subscribe()` for you, and manages subscriptions. Let's build up to writing the `createEffect` function from `@rxfx/effect.` A first-pass draft of it is below:

```javascript

function createEffect(fn) {
  return (...args) => {
    const maybeObservableResult = fn(...args);
    let subscription;

    if (maybeObservableResult.subscribe) {
        subscription = maybeObservableResult.subscribe()
    }
 }
}

const effect1 = createEffect((arg) => {
 console.log(`sync effect: ${arg}`)
);

const effect2 = createEffect((arg) => {
 Promise.resolve().then(() => {
    console.log(`promise effect: ${arg}`)
 });
);

const effect3 = createEffect((arg) => {
 // "return the work"!
 return new Observable(notify => {
    console.log(`Observable effect: ${arg}`)
    notify.complete();
 });
);

effect1(1);
effect2(2);
effect3(3);
// Runs in order: sync, observable, then Promise
```

Now we have a wrapper - `createEffect` - which is a higher-order function that returns an enhanced function. This enhanced one can take the return value of a regular function, and call `subscribe` on it, if it looks like an Observable! This way - whether you return an Observable, a Promise, or nothing - your code inside the Effect will run. But Observables are cancelable - so how can we preserve the ability to cancel it, without a reference to its `Subscription`? Well, we can extend the `createEffect` function to include a `cancelCurrent` method like this.

```javascript
// Cancelation like this:
effect3 = createEffect(/* some fn */);
effect3(1);
effect3.cancelCurrent();

// Could be obtained if we enhanced the returned function.
function createEffect(fn) {
  let subscription;

  // Define the function that calls subscribe on the return from `fn`
  const returnFn = (...args) => {
    const maybeObservableResult = fn(...args);
    if (maybeObservableResult.subscribe) {
      subscription = maybeObservableResult.subscribe();
    }
  };

  // Enhance it and return
  returnFn.cancelCurrent = () => {
    subscription?.unsubscribe();
  };
  return returnFn;
}
```

This way, the variable `subscription` can have its `.unsubscribe()` method invoked when the caller calls `.cancelCurrent` on the enhanced Effect function.

But - you astutlely ask - if a Promise is returned, can it be cancelable? In short, the answer is no. However, it will work to cancel an Observable-returning Effect, so if you need to upgrade from a non-cancelable Promise to a canceling Observable, you can just replace it - _without needing to touch another line of code in your app_!

So this example shows how you can create a higher-order function with `createEffect` that will work with an Observable-returning function just as easily as a Promise-returning one. And we know one reason to work with Observables is that they allow cancelation. But cancelation is not just useful for saving resources (which is important of course). The dreaded source of bugs - the race condition - can often be solved by canceling the current effect. For example, if the input in a search box is 'a', and an effect for its AJAX search is running, then the request for 'aa' arrives - if you don't cancel the first effect, you could end up with 'abacus' appearing even after the user input the search 'aa'.&#x20;

## Cancelation enables Concurrency Control

Because we hate race conditions, we still have to ask - Will we ever run more than one Effect at the same time? What Concurrency Mode options do we have with RxFx? Surely when there's a conflict between an existing effect and a new request, we have more options than just calling `.subscribe()` right away, right?

Thankfully we have those that RxJS has provided, and can write our own custom ones as well. RxFx provides all the power of the concurrency operators of RxJS (), but with friendlier names like 'queueing' and 'blocking' for `concatMap` and `exhaustMap` for example.

So our goal is that the creator of an effect is able to specify what concurrency mode that effect will run in. The default will be calling `.subscribe()` right away like our example did - a mode RxJS calls `mergeMap` and which RxFx calls 'Immediate'. \
\
Here's what we'll implement:

```javascript
effectFn = createEffect(fn); // immediate aka mergeMap
effectFn = createQueueingEffect(fn); // queued aka concatMap
effectFn = createSwitchingEffect(fn); // switching aka switchMap
effectFn = createBlockingEffect(fn); // blocking akaa exhaustMap
```

---

## How to write \`createEffect\` with RxJS

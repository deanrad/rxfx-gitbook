# Animation 60FPS

{% hint style="info" %}
The 𝗥𝘅𝑓𝑥 principles of “return the work” and “fake it till you make it” are shown in this example of using Javascript rather than CSS animation.

**Concepts**: Stubbing handlers, requestAnimationFrame, unrolling recursion. .
{% endhint %}

{% embed url="https://codesandbox.io/embed/animation-recipe-yd7nlr?fontsize=14&hidenavigation=1&theme=dark" %}

In this example of a carnival game, a player on the left and a player on the right can press (or tap) the targets to advance their unicorn across the screen. We'll build this using 𝗥𝘅𝑓𝑥, but first let's answer why we'd choose to animate with JS, not just CSS properties.

In CSS-based animation, a change of the `left` or `transform` style property is accompanied with a `transition` directive like `transition: left 0.25s;`. This suffers the age-old issue of 'events coming in too fast'—Javascript has no idea that an animation is going on. It thinks that a change of the `left` property is instantaneous. So a player could just mash that target 10 times quickly and the unicorn would be all the way across the board. The gameplay could be improved if animation could finish before the unicorn moves again. This is just 𝗥𝘅𝑓𝑥' `listenBlocking` strategy, so let's begin!

### A setup, in React

Our setup: We're in React, and we have an event handler to trigger events of type `player1/touch/start` to the bus. And we have a state setter for the X position of player1. A portion of this might be:

```jsx
const Game = () => {
  const [p1Pos, setP1Pos] = useState(0);
  return (
    <div className="world">
      <div className="player p1" /* shows the unicorn */
        style={{
          left: `${p1Pos}px`
        }}
      />
      <div
        onTouchStart={() => {
          bus.trigger({ type: "player1/touch/start" });
        }}
      >🎯</div>
```

### Moving (approximately)

The movement listener will start out very rough, but we will enhance it until we have full 60fps motion. We'll start with a crude 2-step animation in `listenBlocking` mode, to throttle the player's movement and make it more interesting:

```javascript
const MOVE_AMOUNT = WIDTH / 10;
const DURATION = 250;
bus.listenBlocking(
   ({type}) => type==="player1/touch/start",
   () => concat(
      after(DURATION / 2, { deltaX: MOVE_AMOUNT / 2 }),
      after(DURATION, { deltaX: MOVE_AMOUNT })
   ),
   { 
     next({ deltaX }){ setP1Pos(p => p + deltaX) }   
   }
);
```

The first, familiar argument to the listener is the event we listen for. The final argument is an Observer saying what we do each time the handler produces an event, which is calling our state setter.

\
But why is the handler in the middle returning only two 'frames' of motion? Solely to illustrate the _"fake it till you make it"_ principle. With 𝗥𝘅𝑓𝑥 we can substitute one movement process for another _with no change to the surrounding architecture!_ So until we have smooth animation figured out, 2 steps of animation will do. We have our gameplay right—we just need to improve the display.

### Smooth animationFrames

To upgrade to animationFrames, the smoothest kind of animation, used to require that we jump through hoops to write some recursive function calls. But RxJS has a better idea built in: an Observable that hides that recursion, and gives us frames we can observe, just as we observed our 2-frame animation! In our 2-frame animation we knew the amount of time between each frame, but in general with animation frames we do not. The details of how we do this are in the `reduceToDeltas` function (not shown) but what we see is how we map each time `delta` to an amount to move by in the X direction:

```javascript
import { animationFrames } from "rxjs";
export function moveFrames(distance, duration) {
  return animationFrames().pipe(
    takeWhile(({ elapsed }) => elapsed < duration),
    scan(..reduceToDeltas),
    map(({ delta }) => ({
        deltaX: (delta / duration) * distance
    }))
  );
}

```

Basically: if we have 10 frames, we move about 1/10th of the distance each frame! Now, getting smooth animation is just a matter of swapping this Observable of X deltas for our original stub!

```diff
bus.listenBlocking(
   ({type}) => type==="player1/touch/start",
-   () => concat(
-      after(DURATION / 2, { deltaX: MOVE_AMOUNT / 2 }),
-      after(DURATION, { deltaX: MOVE_AMOUNT })
-  ),
+  () => moveFrames(MOVE_AMOUNT, DURATION),
   { 
     next({ deltaX }){ setP1Pos(p => p + deltaX) }   
   }
);
```

Boom! We have the gameplay we want, and all the smoothness the browser can achieve. We can eventually get even better performance by applying the X position to the DOM through a `transform2d` or similar. But that too would not change our architecture - the `animationFrames` Observable would automatically adjust and just serve shorter deltas!

### Summary

What we have shown is how to stub one process with another (_"fake it till you make it"_), how to do animation outside of any library or framework, and a way to replace recursive code with evented code.

See the full game here: [https://codesandbox.io/s/animation-recipe-yd7nlr](https://codesandbox.io/s/animation-recipe-yd7nlr), and try it with some kids on a mobile device if you get a chance - in my user-testing it is well received!

_\*Note: Some details have been left out of this article, and it is no substitute for learning the true principles of animation._

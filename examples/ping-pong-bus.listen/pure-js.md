# Pure JS

We just saw a ping pong app that can run in NodeJS, or Deno. How much effort is it to bring it into a browser? What will we have to change?

Amazingly - we can do this solely by adding code - not changing a single line we've written! We'll even omit using a framework (though you certainly could tie it into one if you wanted).

### HTML and CSS

We'll set up a ball in HTML, then listeners for PING and PONG events will move it from one side of the screen to the other. CSS transitions will add a realistic motion. Ok, let's do this!

Here is our markup containing a ball (a soccer ball, since a ping pong ball wouldn't be as readily identifiable).

```
<div class="court">
  <div id="ball" class="ani-ball">⚽️</div>
  <div id="turf"/>
</div>
```

CSS transitions are in place so the ball will animate smoothly. Now we can listen for event bus events, and mutate the DOM directly when they occur:

### JavaScript Listeners

```
function player1Kick() {
  ballElem.style.left = "88vw";
  ballElem.style.transform = "rotate(-320deg)";
}
function player2Kick() {
  ballElem.style.left = "2vw";
  ballElem.style.transform = "rotate(0deg)";
}

bus.listen((e): e => e === "PING", player1Kick);
bus.listen((e): e => e === "PONG", player2Kick);
```

Now, we copy the code that ran in NodeJS into this file, and boom, our DOM gets mutated on each of those events! If you don't want to mutate the DOM directly, it's easy enough to call your frameworks' state-setting functions in the handlers. Or bundle up those style changes into named classes. But we're showing that you can sometimes bypass 'state management' if you just orchestrate changes with effects, which we do in a readable fashion here.

Simplicity, For The Win!

{% embed url="https://codesandbox.io/embed/omnibus-ping-pong-6lzvdv?fontsize=14&hidenavigation=1&theme=dark" %}

Now that we're done, would you like to explore Non-CSS Animation for more complex animations, Error Handling for fault tolerance, or TypeScript Integration for code-cleanliness? Or just read on to see how an event bus improves testability.

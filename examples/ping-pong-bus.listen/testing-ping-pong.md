# Testing Ping Pong

Now that we have seen an asynchronous ping-pong match that can run outside of any framework, how do we test it?\
\
Here are the behaviors we have specified and implemented:

```
Describe: Player1 ("PING" listener)
  It: triggers "PONG" 1 second after it hears "PING"
Describe: Player2 ("PONG" listener)
  It: triggers "PING" 1 second after it hears "PONG"
```

All that's required to test these player implementations, is to use a bus in our test suite, trigger events, and await responses!

It is useful to be able to record triggered events, so let's set up with a spy that updates an array:

```
const seen: string[] = [];
bus.spy((e) => seen.push(e));
```

This could be done in a beforeEach, or inline in each test, whatever isolation you require.\
Now, bring in the player that creates the listener that responds:

```
import { player1 } from './players'
```

And send a message the player responds to. Next, wait the specified amount of time, then assert that a new message was seen by the spy!

```
    it('triggers "PONG" when it hears "PING"', async () => {
      bus.trigger("PING");
      await after(2000);
      expect(seen).toEqual(["PING", "PONG"]);
    });
```

And you have tested! It's a good idea to use Virtual Time Testing when possible, but that's a bonus for later. Exactly how you bind to your test suite will be up to you, but these concepts will allow you to test any listener, without even mocking.\
\
**Lessons:** Components are much simpler to test when they are not hard-coded to their effects, but only coupled via a bus. Mocks are not necessary.

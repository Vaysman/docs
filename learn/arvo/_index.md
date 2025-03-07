+++
title = "Arvo"
weight = 1
sort_by = "weight"
template = "sections/docs/chapters.html"
+++
Arvo is Urbit's functional operating system, written in [Hoon](@/docs/learn/hoon/_index.md). It's composed of modules called _vanes_:

## [Ames](@/docs/learn/arvo/ames.md)

Ames is the name of our network and the vane that communicates over it. It's an encrypted P2P network composed of instances of the Arvo operating system.

## [Behn](@/docs/learn/arvo/behn.md)

Behn is our timer. It allows vanes and applications to set and timer events, which are managed in a simple priority queue.

## [Clay](@/docs/learn/arvo/clay.md)

Clay is our filesystem and revision-control system.

## [Dill](@/docs/learn/arvo/dill.md)

Dill is our terminal driver. Unix sends keyboard events to dill from the terminal, and dill produces terminal output.

## [Eyre](@/docs/learn/arvo/eyre.md)

Eyre is our HTTP server. Unix sends HTTP messages to `%eyre`, and `%eyre` produces HTTP messages in response.

## [Ford](@/docs/learn/arvo/ford.md)

Ford is our build system. It handles resources and publishing.

## [Gall](@/docs/learn/arvo/gall.md)

Gall is the vane for controlling userspace apps.

### [Hall](@/docs/learn/arvo/hall.md)

Hall isn't a vane, but an important Gall app that functions as the Urbit messaging protocol.

## Principles

At a high level `%arvo` takes a mess of Unix I/O events and turns them into something clean and structured for the programmer.

`%arvo` is designed to avoid the usual state of complex event networks: event spaghetti. We keep track of every event's cause so that we have a clear causal chain for every computation. At the bottom of every chain is a Unix I/O event, such as a network request, terminal input, file sync, or timer event. We push every step in the path the request takes onto the chain until we get to the terminal cause of the computation. Then we use this causal stack to route results back to the caller.

### Ducts

The `%arvo` causal stack is called a `++duct`. This is represented simply as a list of paths, where each path represents a step in the causal chain. The first element in the path is the first letter of whichever vane handled that step in the computation, or the empty span for Unix.

Here's a duct that was recently observed in the wild:

```
~[
  /g/a/~zod/4_shell_terminal/u/time
  /g/a/~zod/shell_terminal/u/child/4/main
  /g/a/~zod/terminal/u/txt
  /d/term-mess
  //term/1
]
```

This is the duct the timer vane receives when "timer" sample app asks the timer vane to set a timer. This is also the duct over which the response is produced at the specified time. Unix sent a terminal keystroke event (enter), and Arvo routed it to `%dill` (our terminal), which passed it on to the `%gall` app terminal, which sent it to shell, its child, which created a new child (with process id 4), which on startup asked the timer vane to set a timer.

The timer vane saves this duct, so that when the specified time arrives and Unix sends a wakeup event to the timer vane, it can produce the response on the same duct. This response is routed to the place we popped off the top of the duct, i.e. the time app. This app produces the text "ding", which falls down to the shell, which drops it through to the terminal. Terminal drops this down to dill, which converts it into an effect that Unix will recognize as a request to print "ding" to the screen. When dill produces this, the last path in the duct has an
initial element of the empty span, so this is routed to Unix, which applies the effects.

This is a call stack, with a crucial feature: the stack is a first-class citizen. You can respond over a duct zero, one, or many times. You can save ducts for later use. There are definitely parallels to Scheme-style continuations, but simpler and with more structure.

### Making Moves

If ducts are a call stack, then how do we make calls and produce results? Arvo processes "moves" which are a combination of message data and metadata. There are two types of moves. A `%pass` move is analogous to a call:

```
[duct %pass return-path=path vane-name=@tD data=card]
```

Arvo pushes the return path (preceded by the first letter of the vane name) onto the duct and sends the given data, a card, to the vane we specified. Any response will come along the same duct with the path `return-path`.

A `%give` move is analogous to a return:

```
[duct %give data=card]
```

Arvo pops the top path off the duct and sends the given card back to the caller.

### Vanes

As shown above, we use Arvo proper to route and control the flow of moves. However, Arvo proper is rarely directly responsible for processing the event data that directly causes the desired outcome of a move. This event data is contained within a card, which is simply a `(pair term noun)`. Instead, Arvo proper passes the card off to one of its vanes, which each present an interface to clients for a particular well-defined, stable, and general-purpose piece of functionality.

As of this writing, we have seven vanes, which each provide the following services:

- `%ames`: the name of both our network and the vane that communicates over it.
- `%behn`: a simple timer.
- `%clay`: our version-controlled, referentially- transparent, and global filesystem.
- `%dill`: a terminal driver. Unix sends keyboard events to `%dill` from either the console or telnet, and `%dill` produces terminal output.
- `%eyre`: an http server. Unix sends http messages to `%eyre`, and `%eyre` produces http messages in response.
- `%ford`: handles resources and publishing.
- `%gall`: manages our userspace applications. `%gall` keeps state and manages subscribers.

### Cards

Cards are the vane-specific portion of a move. Each vane defines a protocol for interacting with other vanes (via Arvo) by defining four types of cards: tasks, gifts, notes, and signs.

When one vane is `%pass`ed a card in its `++task` (defined in zuse), Arvo activates the `++call` gate with the card as its argument. To produce a result, the vane `%give`s one of the cards defined in its `++gift`. If the vane needs to request something of another vane, it `%pass`es it a `++note` card. When that other vane returns a result, Arvo activates the `++take` gate of the initial vane with one of the cards defined in its `++sign`.

In other words, there are only four ways of seeing a move: (1) as a request seen by the caller, which is a ++note. (2) that same request as seen by the callee, a `++task`. (3) the response to that first request as seen by the callee, a `++gift`. (4) the response to the first request as seen by the caller, a `++sign`.

When a `++task` card is passed to a vane, Arvo calls its `++call` gate, passing it both the card and its duct. This gate must be defined in every vane. It produces two things in the following order: a list of moves and a possibly modified copy of its context. The moves are used to interact with other vanes, while the new context allows the vane to save its state. The next time Arvo activates the vane it will have this context as its subject.

This overview has detailed how to pass a card to a particular vane. To see the cards each vane can be `%pass`ed as a `++task` or return as a `++gift` (as well as the semantics tied to them), each vane's public interface is explained in detail in its respective overview.

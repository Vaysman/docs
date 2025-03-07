+++
title = "Arvo Glossary"
weight = 3
template = "doc.html"
+++

Urbit is renowned for its exotic terminology. Here's a simple overview
from the strange words within.

As Dijkstra put it: "The purpose of abstraction is not to be vague, but
to create a new semantic level in which one can be absolutely precise."

This glossary only covers terms the Arvo side of the project. You can find the
Azimuth glossary [here](../azimuth-glossary)

### application

Also known as an “app,” an _application_ is a Hoon program that can hold state.

### arm

An _arm_ is a named, functionally-computed attribute of a [core](#core).

The [hoon](#a-hoon) of each arm is compiled to a Nock formula, with the
enclosing [core](#core) itself as the subject.

There are two kinds of arms: _dry_ and _wet_. Most arms are
dry.

#### dry

  normal, `++`, polymorphic by means of _variance_

  For a dry arm, we ask: is the new payload compatible with the old
  payload (against which the core was compiled)?

#### wet

  unusual, `+-`, polymorphic by means of _genericity_

  A wet arm uses the hoon as a macro. We create a new type analysis
  path, which works as if we expanded the callee with the caller's
  context.

_See [advanced types](/docs/reference/hoon-expressions/advanced/)_.



### Arvo

The Urbit operating system and kernel. Arvo's state is a pure function of its
event log, and it serves as the Urbit event manager. It contains vanes, which are
kernel modules that perform essential system operations. Arvo itself can only
deal with one file at a time. For more, it relies on Ford, the build-system
vane.

Arvo being purely functional means that the it is a function that is:

`(state, event) → (state, effects)`

Arvo coordinates and reloads vanes. It can be thought of as a traffic-director.
Any vane needs to go through Arvo before reaching another vane. Events and
their effects happen like so:

`Unix event → vere (the virtual machine that runs Urbit) → Arvo → vane → Arvo`

Arvo is located in `/home/sys/arvo.hoon` within your urbit.

More in-depth information on Arvo can be found
[`here`](@/docs/learn/arvo/_index.md).

Below are Arvo modules, which are called "vanes".

#### Ames

  The networking vane. Ames handles all things UDP. Here live the protocols for
  discovering and interacting with other urbits. It’s also the name for the Urbit
  network itself. All communication is signed and encrypted.

  Ames is located in `/home/sys/vane/ames.hoon` within your urbit.

  More in-depth information on Ames can be found
  [`here`](@/docs/learn/arvo/ames.md).

#### Behn

  The timing vane. Behn allows for applications to schedule events, which are
  managed in a simple priority queue. For example, Clay, the Urbit filesystem,
  uses Behn to keep track of time-specific file requests. Eyre, the Urbit web
  vane, uses Behn for timing-out HTTP sessions.

  Behn is located in `/home/sys/vane/behn.hoon` within your urbit.

#### Clay

  The filesystem and typed revision-control vane. Think of Clay as a
  continuously synced git. It handles file-change events and maps them from Urbit
  to Unix and vice versa.

  A common way to use it is to create a pier, a directory that exists in and is
  visible in Unix.

  Clay is located at `/home/sys/vane/clay.hoon` within your urbit.

  Basic information on using Clay can be found
  [`here`](@/docs/using/filesystem.md). An in-depth explanation
  of the architecture can be found
  [`here`](@/docs/learn/arvo/clay.md).

#### Dill

  The terminal-driver vane. You run your urbit in your Unix terminal, and
  Unix sends every event -- such as a keystroke or a change in the dimensions of
  the terminal window -- to be handled by Dill. Dill acts as an intermediary for
  anything that uses keyboard events. This is why there’s some input lag in
  Urbit’s command-line interface. (Optimization will come later!)

  A keyboard event's journey from Unix to Dojo, the Urbit shell, can be imagined
  as diagrammed below:
  ```
  Keystroke in Unix -> Vere (virtual machine) -> Arvo -> Dill -> the Dojo
  ```

  Dill is located at `/home/sys/vane/clay.hoon` within your urbit.

  More in-depth information on Dill can be found
  [`here`](@/docs/learn/arvo/dill.md).

#### Eyre

  The web-server vane. Anything HTTP-related lives here. Unix sends HTTP messages
  though to Eyre, and Eyre produces HTTP messages in response.

  In general, apps and vanes do not call Eyre; rather, Eyre calls apps and vanes.
  Eyre uses Ford and Gall to functionally publish pages and facilitate
  communication with apps.

  Eyre is located at `/home/sys/vane/eyre.hoon` within your urbit.

  More in-depth information on Eyre can be found
  [`here`](@/docs/learn/arvo/eyre.md).

#### Ford

  The build-system vane. Ford is capable of sequencing generic asynchronous
  computations. Its uses include linking source files into programs, assembling
  live-updating websites, and performing file-type conversions.

  Ford neither accepts Unix events nor produces Unix effects. It exists
  entirely for the benefit of Urbit applications and other vanes, in particular
  Gall.

  Ford is located at `/home/sys/vane/ford.hoon` within your urbit.

  More in-depth information on Ford can be found
  [`here`](@/docs/learn/arvo/ford.md).

#### Gall

  The application-management vane. Userspace apps -- daemons, really -- are
  started, stopped, and sandboxed by Gall. Gall provides developers with a
  consistent interface for hooking their app up to the operating system. It allows
  applications and other vanes to send messages to applications and subscribe to
  data streams. Messages coming into Gall are routed to the intended application,
  and the response comes back along the same route. If the intended target is on
  another ship, Gall will route it, behind the scenes, through Ames to the other
  ship.

  Gall is located at `/home/sys/vane/gall.hoon` within your urbit.

  More in-depth information on Gall can be found
  [`here`](@/docs/learn/arvo/gall.md).

### atom

An atom is any natural number, including zero. The atom is the most basic data
type in [Nock](#nock) and [Hoon](#hoon). A Hoon atom type describes a Nock
atom with two additional pieces of metadata: an [_aura_](#aura), and an
optional constant.

An atom type is _warm_ or _cold_ based on whether the constant exists.


#### warm

  If the constant is `~` (null), any atom is in the type.

#### cold

  If the constant is `[~ atom]`, its only legal value is `atom`.

_See [basic types](@/docs/reference/hoon-expressions/basic.md)_

### aura

An aura is a soft atom type. They appear as strings beginning with `@`. Auras
represent the structure of an atom, print format, or other semantics. Its
constraints on the value of an atom aren't enforced in any way.

You can cast something to an aura, like a decimal, and get back its hexademical.
For example:

```
    > =a `@`29
    > ^-  @ux  a
    0x1d
```

Auras are an advisory type system. You can go up or down, but not sideways in
the hierarchy.  `@t` and `@` can each be cast to the other, but @t can’t be cast
to `@u`, for example, without first being cast to the empty aura `@`.

For example

```
    > ^-  @  'c'                       ::  'c' is of the aura @t
    99

    > ^-  @u  'c'
    ! exit
```

Here is a non-exhaustive list of auras:

```
Aura         Meaning                        Example Literal Syntax
-------------------------------------------------------------------------
@d           date
  @da        absolute date                  ~2018.5.14..22.31.46..1435
@n           nil                            ~
@p           phonemic string (ship name)    ~sorreg-namtyv
@r           IEEE floating-point
  @rd        double precision  (64 bits)    .~6.02214085774e23
  @rh        half precision (16 bits)       .~~3.14
@s           signed integer, sign-bit low
  @sb        signed binary                  --0b11.1000
  @sd        signed decimal                 --1.000.056
  @sx        signed hexadecimal             -0x5f5.e138
@t           UTF-8 text (cord)              'howdy'
  @ta        ASCII text (knot)              ~.howdy
    @tas     ASCII text symbol (term)       %howdy
@u              unsigned integer
  @ub           unsigned binary             0b11.1000
  @ud           unsigned decimal            1.000.056
  @ux           unsigned hexadecimal        0x5f5.e138
```

You can find the list of all auras [here](@/docs/reference/hoon-expressions/rune/constants.md).

### blocksize

Blocksize is the power-of-two bitwidth of an atom. So, an atom with a blocksize
of `3` has a bitwidth of `2^3`, which is 8. A bitwidth of 8, in turn, can
represent `2^8` values, or 256.

### cast

To cast is to put a Hoon expression through the compiler’s type-checker to test
whether the expression is guaranteed to evaluate to a product that falls within
the desired type. If the expression is not guaranteed to behave in this way,
then the compiler will halt with a nest-fail crash. Proper Hoon code will
include a cast for the product type of each gate.

The cast rune is `^-` (pronounced "ket-hep"). So, to cast to a list of atoms,
we write `^-  (list @)`. The irregular form of that expression is ``(list @)``.

### cell

In Nock and Hoon, a _cell_ is an ordered pair of [nouns](#noun).

Syntax: `[a b]`.

Cells orient to the right; for example, the noun [2 6 14 15] is short for
[2 [6 [14 15]]].


### clam

A clam is the use of a structure as a gate. Usually used to verify marks from
other urbits.

```
    > ((list @t) [97 98 ~])
    <|a b|>
```

### cord

In Hoon, `cord` is an aura for representing atoms as strings. UTF-8,
least-significant bit first. Represented as the aura `@t`.

```
    > `@t`478.560.413.032
    'hello'
```

### core

A _core_ is a [cell](#nock-cell) of `[code data]`, where we call the
code head the _battery_ and the data tail the _payload_. All code-data
structures in normal languages (functions, objects, modules, etc.)
translate to cores in Hoon.

#### battery

  The code-containing head of a core.

#### payload

  The data-containing tail of a core.

_See [basic types](@/docs/reference/hoon-expressions/basic.md)._

### Dojo

The Dojo is the Urbit shell; the command line where you run programs,
expressions, and system commands. It’s a CLI that is encountered right when
an urbit is started for the first time. It is somewhat analogous to Bash, the
Unix terminal, and more analogous to a Hoon REPL.

You’ll notice that many things can’t be written here; this is normal. The Dojo
enforces syntactical correctness right at the command line. Strings that can’t
be continued into a valid expression are automatically trimmed back to something
that can be. Some syntactically invalid expressions can be completed, but the
Dojo will  prevent them from being entered as a command. You can play around
with this functionality learn what syntax is kosher.


### duct

A duct is an Arvo-level call stack. At the beginning of every duct is a Unix
event, such as a keystroke, network packet, timer event, or file change.

A duct is a `list` of `wire`s. A wire is itself a `list` of `knot`s. A `knot`
is, in turn, a knot.

### face

A _face_ is the Hoon analog to what is a variable name in most other
programming languages. Faces are metadata labels for legs, allowing us to
address parts. Because there is no metadata in Nock, faces cannot exist in Nock.

```
    > a=2
    a=2
```

In the above example, `a` is the face and `2` is the value.

Hoon has no scope or symbol-table; there is only the subject. To "declare" a
"variable" is to construct a new subject: `[name=value old-subject]`.

_See [advanced types](@/docs/reference/hoon-expressions/advanced.md)_.


### flag

A `flag` is Hoon's version of a boolean. `0` (`%.y`) is _yes_, `1` (`%.n`) is
_no_.

### gate

A _gate_ is a [core](#core) with one [arm](#arm) -- Hoon's closest
analog to a function. To call a gate on an argument, replace the sample
(at [tree address](@/docs/reference/hoon-expressions/limb/limb.md) `+6`
in the core) with the argument, and then compute the arm.

The payload of a gate has a shape of `[sample context]`.

#### sample

  The argument tuple.

#### context

  The subject in which the gate was defined.

_See [basic types](@/docs/reference/hoon-expressions/basic.md),
[`%-` ("cenhep")](@/docs/reference/hoon-expressions/rune/cen.md#cenhep) (the
[rune](#rune) for calling a `gate`)._

### generator

A _generator_ is like a limited Hoon script: it’s a shell command that produces
a value before disappearing. Generators take an input [noun](#noun) (a value)
and produce an  output `noun` by performing calculations on its arguments,
prompting for input, or making HTTP requests.

You need a `bar` rune like `|=` in your generator, because a generator is
essentially a function that you’re calling.

Generators, unlike [apps](#application), hold no state. They are pure,
stateless functions, and they cannot send [moves](#move). Because of this,
generators are have a more narrow use than apps. Among the non-trivial uses of
a a generator is moving data into apps -- they’re the backbone of Urbit’s
information pipeline.

Generators are run from the `/gen` directory of your current desk. For example,
to run a generator named `whatever.hoon`, you would type `> +whatever` in the
[Dojo](@/docs/using/shell.md).

### glyph

A _glyph_ is a single, non-alphanumeric ASCII character. Each glyph has its own
monosyllabic name, which allows the efficient communication of Hoon-related
information in speech. [`Runes`](#rune) are composed of two glyphs.
Below is a complete list of glyphs with their associated pronunciations.

```
ace [1 space]   gal <               pal (
bar |           gap [>1 space, nl]  par )
bas \           gar >               sel [
buc $           hax #               mic ;
cab _           hep -               ser ]
cen %           kel {               sig ~
col :           ker }               soq '
com ,           ket ^               tar *
doq "           lus +               tic `
dot .           pam &               tis =
fas /           pat @               wut ?
zap !
```

### Hall

_Hall_ is the Urbit communication back-end. It’s a messaging bus that can work
with clients of all sorts. [`:talk`](#talk), the built-in chat client, is one
such app that uses Hall.

_More information on Hall can be found [here](@/docs/learn/arvo/hall.md)._

#### circle

  A circle is the “audience” of your messages -- who can receive them. It can be
  thought of as a chat channel, but this is a little too narrow. More
  specifically, it’s a named collection of messages created by and hosted on a
  ship’s Hall, usually represented as `~ship-name/circle-name`. On Talk,
  Urbit’s built-in chat front-end, the first circle that you will likely
  encounter is `~dopzod/urbit-help`.

### Hood

_Hood_ orchestrates many of the Urbit initialization systems necessary
for boot.

### Hoon

Hoon is a strict, higher-order typed functional language that compiles itself
to Nock.

The Hoon source file is located in `/home/sys/hoon.hoon` within your urbit.

_More information can be found in the [hoon](@/docs/learn/hoon/_index.md) section._

#### Mint

  _Mint_ is the Hoon compiler function. Mint takes the subject type and the
  expression source (a [hoon](#a-hoon), for example) and produces the product
  type and nock formula. So [type hoon] is mapped to [type nock].

### hoon

A _hoon_ (lowercase) is the result of parsing a Hoon source expression into an
AST node. These AST nodes are nouns, like all other Hoon data. Because every
Hoon program is, in its entirety, a single expression of Hoon, the result of
parsing the whole thing into an AST is a single `hoon`.

A hoon is a tagged union of the form `[%tag data]`, where the tag
is a constant such as `%brts` (from the source [rune](#rune) `|=`,
i.e. "bartis"), and is matched up with the appropriate type of data
(often more `hoon`s, from source subexpressions). For example, the
expression `:-(2 17)`, once parsed into an AST, becomes the following:

```
[%clhp p=[%sand p=%ud q=2] q=[%sand p=%ud q=17]]
```

The `%clhp` is produced from the rune `:-` (i.e. "colhep"). The 2 and
17 have each been parsed as `%sand`-tagged hoons, which represent
atoms (in this case each with an aura of `%ud`, i.e. unsigned
decimal).

To parse Hoon source into a hoon AST, use `ream` on a `cord`
containing Hoon source. Try the following in [Dojo](@/docs/using/shell.md):

```
(ream ':+(12 7 %a)')
```

The result should be:

```
[%clls p=[%sand p=%ud q=12] q=[%sand p=%ud q=7] r=[%rock p=%tas q=97]]
```

See [expression reference](@/docs/reference/hoon-expressions/rune/_index.md) for the full list of hoons.

### irregular form

Irregular forms are alternative ways of writing Hoon expressions, alternatives
to the digraphic runes such as `:~`. Irregular forms are usually more compact
than regular forms.=

_See [irregular forms](@/docs/reference/hoon-expressions/irregular.md)._

### jet

A _jet_ is a piece of C code that have equivalent products to a piece of Hoon or
Nock code. They are used for performance reasons; jets are meant to be faster
than the code that they are substituting for.

Jets are necessary for mathematics, performance-wise, because Nock handles it
so inefficiently. Nock, for example, can’t decrement (reduce a number by one)
except by starting building a loop that starts from zero and increments to a
number just below.

_See [API overview](@/docs/learn/vere/api.md)._

### lark

A lark is an expression that returns a [limb](#limb) at the specified
address in a cell. Using `-` by itself returns the head of the subject, and
using `+` by itself returns the tail:

```
> -:[[4 5] [6 [14 15]]]
[4 5]

> +:[[4 5] [6 [14 15]]]
[6 14 15]
```

You can combine `+` or `-` with either of `>` or `<` to get a more specific limb
of the subject.  `-<` returns the head of the head; `->` returns the tail of the
head; `+<` returns the head of the tail; and `+>` returns the tail of the tail.
By alternating the `+`/`-` symbols with `<`/`>` symbols, you can go as deep as
you desire.

```
> +>-:[[4 5] [6 [14 15]]]
14

> +>+:[[4 5] [6 [14 15]]]
15
```

### limb

A _limb_ is an attribute or variable reference. A limb is an [arm](#arm) or a
[leg](#leg).

To resolve a _limb_ named "foo", the subject is searched depth-first,
head-first for either a face named "foo" or a core with an arm of
"foo". If a face is found, the result is a leg, if a core is
found, the result is the product of the arm.

#### leg

  a subexpression, or subtree of the
  [subject](#nock-subject).

#### wing

  a search path into the subject, composed of
  limbs. Search occurs from right to left (`a.b` means `b` within `a`).

_See [Limbs and wings](@/docs/reference/hoon-expressions/limb/_index.md)_

### move

A move is the [Arvo](#arvo) equivalent of a syscall.

### noun

In [Nock](#nock) and [Hoon](#hoon), a _noun_ is an atom or a cell.

### Nock

Nock is a Turing-complete, non-lambda combinator interpreter. It's
Urbit's low-level programming language. Nock is functional and typeless.

#### noun

  an _atom_ or a _cell_.

#### atom

  any natural number, including zero.

#### cell

  any ordered pair of nouns.


#### subject

  a noun - the data against which a _formula_ is evaluated.

#### formula

  a noun - a function at the nock level.

#### product

  a noun - the result of evaluating a formula against a subject.

_See the [Nock definition](@/docs/learn/nock/definition.md)._

### mark

A _mark_ is Urbit's version of a MIME type, if a MIME type was an
executable specification. The mark is just a label that's used as a path
to a local source file in the Arvo filesystem.  This source file defines
a core that can mold untrusted data, diff and patch, convert to other
marks, etc.

If this sounds like magic, it isn't quite magic.  There's no way
for different urbits to make sure they mean the same thing by the
same mark. However, when incompatibility happens, marks ensure
that we at least handle the situation in a predictable way.

#### metal

Every core has a _metal_ which defines its variance model (ie, the
properties of the type of a compatible core). The default is `gold`
(invariant).

- `gold`: _invariant_
- `lead`: _bivariant_
- `zinc`: _covariant_
- `iron`: _contravariant_

_See [advanced types](@/docs/reference/hoon-expressions/advanced.md)_.

### ship

An Urbit _ship_ is: a cryptographic title on a _will_ signed by private key; a
human-memorable name; and a packet-routing address. Ships are classed by the
number of bits in their address:

```
    Size   Name    Parent  Object      Example
    -----  ------  ------  ------      -------
    2^8    galaxy  ~       supernode   ~zod
    2^16   star    galaxy  supernode   ~dozbud
    2^32   planet  star    user        ~tasfyn-partyv
    2^64   moon    planet  device      ~sigsam-nimbot-tasfyn-partyv
    2^128  comet   ~       bot         ~racmus-mollen-fallyt-linpex--watres-sibbur-modlux-rinmex
```

Any ship can be called an _urbit_. The lowercase "u" distinguishes it from Urbit
the software stack and Urbit the network.

_You can find a longer-form summary [here.](https://urbit.org/posts/essays/the-urbit-address-space/)._

An Urbit identity is a string like `~firbyr-napbes`. It means nothing,
but it's easy to remember and say out loud. `~firbyr-napbes` is actually
just a 32-bit number (`3.237.967.392`, to be exact), like an IP address,
that we turn into a human-memorable string. The full name of this string
can be viewed by typing `our` in the Dojo, Urbit's shell. This is useful
when running a ship with a longer name, such as a _moon_ or a _comet_.

Technically, an urbit is a secure digital identity that you own and
control with a cryptographic key, like a Bitcoin wallet. As in Bitcoin,
the supply of urbits is mathematically limited. This keeps the network
friendly, by making spam and abuse expensive.

An urbit name is just a number; smaller numbers make shorter names.
Shorter names are easier to remember, so they're more valuable. So
urbits are classified by the number of bits in their name. (A ship name
is just a scrambled base-256 representation of the number.)

A 32-bit urbit (like `~firbyr-napbes`) is called a _planet._ A 16-bit
urbit (like `~pollev)` is a _star._ An 8-bit ship (like `~mun`) is a
_galaxy._ A planet is an identity for an independent, adult human. Stars
and galaxies are network infrastructure.

Each planet or star is launched by its _parent,_ the star or galaxy
whose number is its bottom half. So the planet `~firbyr-napbes`,
`0xdead.beef` or `3.735.928.559`, is the child of `~pollev`, `0xbeef` or
`48.879`, whose parent is `~mun`, `0xef`, `239`. The galaxy at the address of
`0` is `~zod`.

#### pier

  A ship's _pier_ is its Unix directory.  For planets, the name of the pier is
  usually the planet name.

### structure

A _structure_ is an idempotent [gate](#gate) (function) that constructs and
validates types in Hoon.

An example: Arvo [marks](#mark) use Hoon's type system via structures to
validate untrusted network data.

Here's some common structure terminology:

#### bunt

  A bunt is the default value of a [structure](#structure). It's also a verb;
  to _bunt_ a structure is to produce the bunt of that structure. For example,
  bunting `@` results in `0`, and bunting `@t` results  in `''`. Bunting is
  performed with `*` followed by the structure in question:

  ```
      > *@
      0
      > *@t
      ''
  ```

#### icon: The type of the mold's range

_See [mold hoons](/docs/reference/hoon-expressions/rune/buc/)._

### rune

A Hoon _rune_ is a pair of ASCII symbols used to begin a
[Hoon expression](#a-hoon).

For example, the rune [`?:`](@/docs/reference/hoon-expressions/rune/wut.md#wutcol) is
Hoon's most common conditional, a branch on a boolean test. The first
symbol in a rune represents a family of related runes. For example, the
[`?` family](@/docs/reference/hoon-expressions/rune/wut.md) are all conditionals.

The result of parsing a Hoon source expression&mdash;the rune, followed
by its respective children&mdash;into an AST node is simply called
a [`hoon`](#a-hoon).

Runes have two syntactic forms, _tall_ and _flat_:

#### tall

  multiple lines, no parentheses, two or more spaces between tokens

  Example:

  ```
  > %+  add  2  2
  4
  ```

#### flat

  one line, parentheses, one space between tokens

  Example:
  ```
  > %+(add 2 2)                              ::  regular flat form
  4

  > (add 2 2)                                ::  irregular flat form
  4
  ```

Tall hoons can contain flat hoons, but not vice versa. All irregular
forms are flat.

_See [hoon concept](#a-hoon),
[expressions](@/docs/reference/hoon-expressions/rune/_index.md).

### Sail

_Sail_ is the Hoon markup language for XML.

### scry

To _scry_ means to request data from the Arvo namespace and bring it
to userspace.

_See [.^ (dot-ket)](@/docs/reference/hoon-expressions/rune/dot.md#dotket)._

### slot

`slot` is the name for Hoon's tree-addressing scheme.

Every cell has a head and a tail, each of which may be either an atom or
a cell. Therefore every noun is a binary tree. `+1` or `.` resolves to
the whole cell (technically this would work against an atom as well).
The head of `+n` is `+2n`, the tail is `+(2n+1)`.

`+7` is a special address for gates, because the position is defined
(by convention) as the context of a gate and of all cores.

### talk

_Talk_ is Urbit's built-in chat app. It’s one example of an app that can be
built on top of Hall, the Urbit back-end messaging system.

_See [Messaging](@/docs/using/messaging.md). for more information._

### type

A Hoon _type_ defines a set (finite or infinite) of nouns and ascribes
some semantics to it.  There is no direct syntax for defining
types; they are always defined by inference (i.e., by
[`mint`](#mint)), usually using a constructor ([structure](#structure)).

All types are assembled out of base types defined in `++type`.
(Look up `++  type` in hoon.hoon for examples.) When the compiler
does type-inference on a program, it assembles complex types out
of the simpler built-in types.

#### nest

  `nest` is an internal Hoon function on two types which performs a type
  compatibility test. `nest` produces yes if the
  set of nouns in the second type is provably a subset of the first.
  If `nest` produces no, the Hoon programmer will receive a `nest-fail`
  error. This is one of the most commons errors in Hoon programming.

  _See [advanced types](@/docs/reference/hoon-expressions/advanced.md)_ and [troubleshooting](@/docs/reference/troubleshooting.md).


### Unit

Hoon's equivalent of a nullable pointer or a Haskell Maybe. If `q` is `~`,
null, the type is _warm_; any atom is in the type. If `q` is `[~ x]`, where
`x` is any atom, the type is _cold_; its only legal value is the constant `x`.

### Vane

A _vane_ is an [Arvo](#arvo) module. Each Vane is a single file because Arvo
cannot, on its own, deal with programs that are split between multiple files.
Some vanes can do this, however.

### Vere (pronounced “vair”)

The runtime system, written in C, that implements [Nock](#nock). All
It’s written in C. Vere-level errors are prepended with as `bail:`.

Vere is a virtual machine: it pretends to be hardware, and is the single point
of contact between your Urbit and the rest of your system. But more than that,
Vere handles the Urbit event log and handles jets. It’s also contains the i/o
drivers for Urbit’s vanes, responsible for generating events from Unix and
applying effects to Unix.

### Zuse

_Zuse_ is a defined interface for vanes. It contains the part of the Hoon
standard library that is specific to Arvo functionality.

Arvo is located in `/home/sys/zuse.hoon` within your urbit.

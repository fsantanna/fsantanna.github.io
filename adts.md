# Algebraic Data Types with Subtyping and Inheritance

<img src="twitter.png" style="vertical-align:middle">
[@_fsantanna](https://twitter.com/_fsantanna)

Algebraic Data Types (ADTs) combine tuples and tagged unions to form complex
types.

The upcoming version of [Ceu][1] supports anonymous tuples and unions as its
basic ADT mechanisms as follows:

[1]: https://github.com/fsantanna/ceu

```
var pt: [Int,Int] = [10,20]  -- a pair tuple
output std pt.1     --> 10

var ui: <(),Int> = <.2 20>   -- a pair union (either Unit or Int)
output std ui?2     --> 1 (yes, ui holds the second Int variant)
output std ui!2     --> 20
```

Tuple types and constructors use square brackets and are pretty standard.
Union types use angle brackets and can be anonymous just like tuples.
Union constructors also use angle brackets and receive the variant index and
its argument.
Union predicates and discriminators use question and exclamation marks,
respectively, followed by a variant index.

Ceu also supports user-defined types, which can combine tuples and unions to
form classic ADTs.
The next snippet is extracted from an event library with keyboard and mouse
events:

```
type Point = [Int,Int]

type Event = <<Int,Int>, <[Point,Int],Point>>
           -- Key Dn/Up | Mouse Click,Motion

var e: Event = <.2 <.1 [[10,20],1]>> -- a mouse click with but=1 at pos=[10,20]
output e?1?1    --> 0 (no, this is not a key down event)
output e!2!1.2  --> 1 (clicked mouse button number)
```

Instead of indexes, we can also use tuple fields and variant names as aliases:

```
type Point = [x:Int,y:Int]

type Event = <
    Key = <
        Down = Int,
        Up = Int
    >,
    Mouse = <
        Click = [pos:Point, but:Int],
        Motion = Point
    >
>

var e = Event.Mouse.Click [[10,20], 1]
output e?Key?Down        --> 0 (no, this is not a key down event)
output e!Mouse!Click.but --> 1 (clicked mouse button number)
```

Note how nested anonymous unions can induce type hierarchies with arbitrary
levels:
An `Event.Key.Up` could be a subtype of `Event.Key`, and both could be subtypes
of `Event`.
We are therefore experimenting with ADT subtyping in Ceu:

```
var clk: Event.Mouse.Click = Event.Mouse.Click [[10,20], 1]
var evt: Event = clk                                    -- upcast always ok
var cst: Event.Mouse.Click = clk :: Event.Mouse.Click   -- downcast success
var err: Event.Key = clk :: Event.Key                   -- downcast error
```

One peculiarity of this design is that the hierarchy chain must always be
explicit: `Click` is not a type by itself without the full path
`Event.Mouse.Click`.

The other experimental ADT feature in Ceu is inheritance.
Note that `Event.Mouse.Click` and `Event.Mouse.Motion` both hold the mouse
position, while `Event.Key.Down` and `Event.Key.Up` both hold the key code.
In Ceu, subtypes can share and reuse fields attached to their supertypes.
Hence, we can rewrite the `Event` type hierarchy as follows:

```
type Event = <
    Key = [key:Int] + <  -- [key:Int] is attached to subtypes:
        Down = (),       --     Event.Key.Down [key:Int]
        Up = ()
    >,
    Mouse = [pos:Point] + <
        Click = [but:Int],
        Motion = ()
    >
>

var mot = Event.Mouse.Motion [[10,20]]
output std mot!Mouse!Motion.pos.x   --> 10

var key = Event.Key.Up [65]
output std key!Key!Up.key           --> 65
```

These are early experiments and there is still a lot to do about generics,
variance, recursion, and memory management though...

Do you know about other languages or papers presenting similar functionalities?

- Anonymous tagged unions.
- Subtyping
- Data inheritance

I'm interested in learning more about them.

Comment on <img src="twitter.png" style="vertical-align:middle"> [@_fsantanna](https://twitter.com/_fsantanna/status/1498993572783271949).

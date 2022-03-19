# Algebraic Data Types with Inheritance and Subtyping

Algebraic Data Types (ADTs) combine tuples and tagged unions to form complex
types.
The upcoming version of Ceu supports anonymous tuples and unions as follows:

```
var pt: [Int,Int] = [10,20]
output std pt.1  --> 10

var ui: <(),Int> = <.2 20>   -- either Unit or Int
output std ui?2   --> 1 (yes, ui holds the second Int variant)
output std ui!2   --> 20
```

Tuple types and constructors use square brackets and are pretty standard.
Union types use angle brackets and can be anonymous as shown in the example.
Union constructors also use angle brackets and receive the variant index and
its argument.

ADTs can combine tuples and unions in user-defined types:

```
type Point = [Int,Int]

type Event = <Int, <[Point,Int],Point>, <Int,Int>>
         -- Frame | Mouse Click,Motion | Key Dn/Up

var e: Event = <.2 <.1 [[10,20],1]>> -- a mouse click at pos=[10,20] with but=1
output e?3?1    --> 0 (no, this is not a key down event)
output e!2!1.2  --> 1 (clicked mouse button number)
```

Instead of indexes, we can also use tuple fields and variant names as aliases:

```
type Point = [x:Int,y:Int]

type Event = <
    Frame = Int,
    Mouse = <
        Click = [pos:Point, but:Int],
        Motion = Point
    >,
    Key = <
        Down = Int,
        Up = Int
    >
>

var e = Event.Mouse.Click [[10,20], 1]
output e?Key?Down        --> 0 (no, this is not a key down event)
output e!Mouse!Click.but --> 1 (clicked mouse button number)
```

Note how nested anonymous unions induce type hierarchies with arbitrary levels:
An `Event.Key.Up` could be a subtype of `Event.Key` and both could be subtypes
of `Event`.
We are therefore experimenting with ADTs subtyping in Ceu:

```
var clk: Event.Mouse.Click = Event.Mouse.Click [[10,20], 1]
var evt: Event = clk                                    -- upcast always ok
var cst: Event.Mouse.Click = clk :: Event.Mouse.Click   -- downcast success
var err: Event.Frame = clk :: Event.Frame               -- downcast error
```

The other experimental feature is data inheritance.
Note that `Event.Mouse.Click` and `Event.Mouse.Motion` both hold the mouse
position, while `Event.Key.Down` and `Event.Key.Up` share the key code.
We can rewrite the type as follows:

```
type Point = [x:Int,y:Int]

type Event = <
    Frame = Int,
    Mouse = [pos:Point] + <
        Click = [but:Int],
        Motion = ()
    >,
    Key = [key:Int] + <
        Down = (),
        Up = ()
    >
>

var mot = Event.Mouse.Motion [[10,20]]
output std mot!Mouse!Motion.pos.x   --> 10

var key = Event.Key.Up [65]
output std key!Key!Up.key   --> 65
```

<!--
- AND / OR

- implementation
    - invert tag
        - composition same as inheritance

- anonymous
- inheritance
    - reuse fields
- subtyping
    - pass to function

- still closed
- exhaustive match

- parametric?
- variance?


what other languages do you know with
- anonymous tagged unions
- reuse inheritance
- subtytping
-->

# Algebraic Data Types with Subtyping and Inheritance

<img src="twitter.png" style="vertical-align:middle">
[@\_fsantanna](https://twitter.com/_fsantanna)

Algebraic Data Types (ADTs) combine tuples and tagged unions to form complex
types.

The upcoming version of [Ceu][1] supports anonymous tuples and unions as its
basic ADT mechanisms as follows:

[1]: https://github.com/fsantanna/ceu

<pre>
var</b> pt: [Int,Int] = [10,20]  -- a pair tuple
output</b> std pt.1     --> 10

var ui: <(),Int> = <.2 20>   -- a pair union (either Unit or Int)
output</b> std ui?2     --> 1 (yes, ui holds the second Int variant)
output</b> std ui!2     --> 20
</pre>

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

<pre>
<b>type</b> Point = [Int,Int]

<b>type</b> Event = <<Int,Int>, <[Point,Int],Point>>
           -- Key Dn/Up | Mouse Click,Motion

<b>var</b> e: Event = <.2 <.1 [[10,20],1]>> -- a mouse click with but=1 at pos=[10,20]
<b>output</b> e?1?1    --> 0 (no, this is not a key down event)
<b>output</b> e!2!1.2  --> 1 (clicked mouse button number)
</pre>

Instead of indexes, we can also use tuple fields and variant names as aliases:

<pre>
<b>type</b> Point = [x:Int,y:Int]

<b>type</b> Event = <
    Key = <
        Down = Int,
        Up = Int
    >,
    Mouse = <
        Click = [pos:Point, but:Int],
        Motion = Point
    >
>

<b>var</b> e = Event.Mouse.Click [[10,20], 1]
<b>output</b> e?Key?Down        --> 0 (no, this is not a key down event)
<b>output</b> e!Mouse!Click.but --> 1 (clicked mouse button number)
</pre>

Note how nested anonymous unions can induce type hierarchies with arbitrary
levels:
An `Event.Key.Up` could be a subtype of `Event.Key`, and both could be subtypes
of `Event`.
We are therefore experimenting with ADT subtyping in Ceu:

<pre>
<b>var</b> clk: Event.Mouse.Click = Event.Mouse.Click [[10,20], 1]
<b>var</b> evt: Event = clk                                    -- upcast always ok
<b>var</b> cst: Event.Mouse.Click = clk :: Event.Mouse.Click   -- downcast success
<b>var</b> err: Event.Key = clk :: Event.Key                   -- downcast error
</pre>
</pre>

One peculiarity of this design is that the hierarchy chain must always be
explicit: `Click` is not a type by itself without the full path
`Event.Mouse.Click`.

The other experimental ADT feature in Ceu is inheritance.
Note that `Event.Mouse.Click` and `Event.Mouse.Motion` both hold the mouse
position, while `Event.Key.Down` and `Event.Key.Up` both hold the key code.
In Ceu, subtypes can share and reuse fields attached to their supertypes.
Hence, we can rewrite the `Event` type hierarchy as follows:

<pre>
<b>type</b> Event = <
    Key = [key:Int] + <  -- [key:Int] is attached to subtypes:
        Down = (),       --     Event.Key.Down [key:Int]
        Up = ()
    >,
    Mouse = [pos:Point] + <
        Click = [but:Int],
        Motion = ()
    >
>

<b>var</b> mot = Event.Mouse.Motion [[10,20]]
<b>output</b> std mot!Mouse!Motion.pos.x   --> 10

<b>var</b> key = Event.Key.Up [65]
<b>output</b> std key!Key!Up.key           --> 65
</pre>

These are early experiments and there is still a lot to do about generics,
variance, recursion, and memory management though...

Do you know about other languages or papers presenting similar functionalities?

- Anonymous tagged unions.
- Subtyping
- Data inheritance

I'm interested in learning more about them.

Comment on <img src="twitter.png" style="vertical-align:middle"> [@\_fsantanna](https://twitter.com/_fsantanna/status/1505195728502722566).

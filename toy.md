# A Toy Problem: Drag, Click, or Cancel

<img src="twitter.png" style="vertical-align:middle">
[@\_fsantanna](https://twitter.com/_fsantanna)

Last week Patrick Dubroy ([@dubroy][0]) proposed a [toy problem][1] to handle
user input:

> The goal is to implement a square that you can either drag and drop, or
> click. The code should distinguish between the two gestures: a click
> shouldn’t just be treated as a drop with no drag. Finally, when you’re
> dragging, pressing escape should abort the drag and reset the object back to
> its original position.

He proposed solutions using three different implementation techniques:
    Event listeners, Polling, and Process-oriented.
For the process-oriented approach he uses his Esterel-inspired [Abro.js][2] and
argues that it provides "a clear, explicit sequencing between the different
states".

Since he mentions [Céu][3] and since I'm currently working on its upcoming
version, I felt motivated to also post a solution to the problem.
The solution in Céu is similar to his, and uses the `paror` and `watching`
constructs to safely abort behaviors that did not complete.
A small difference worth mentioning is relying on the deterministic scheduling
semantics of Céu to eliminate a state variable (`didDrag`).

Currently, Céu lacks primitive types (e.g., strings and integers), and named
fields, so some parts of the code might look strange.
Anyways, here's the complete solution with [an accompanying video][4]:

<pre>
^"int.ceu"
^"pico.ceu"

-- rectangle to control
<b>var</b> rect: Rect = [[_10,_10],[_5,_5]]

-- tasks to behaviors: cancel, drag and drop, click
<b>spawn</b> {
    -- outer loop to restart after each behavior is detected
    <b>loop</b> {
        -- first detects the first click on the rectangle
        <b>var</b> dxy: Size   -- holds the offset to its center
        {
            <b>var</b> mouse: Point
            <b>await</b> evt?Mouse?Button?Down <b>until</b> isPointInsideRect [mouse,rect]
                <b>where</b> {
                    <b>set</b> mouse = evt!Mouse!Button!Down.pos
                }
            <b>set</b> dxy = [sub [rect.pos.x,mouse.x], sub [rect.pos.y,mouse.y]]
        }

        -- then either cancel, drag/drop, or click
        <b>paror</b> {
            -- cancel behavior: restores the original position on any key
            <b>var</b> orig = rect
            <b>await</b> evt?Key?Down <b>until</b> eq [evt!Key!Down.key,_SDLK_ESCAPE]
            <b>set</b> rect = orig
            <b>output</b> std _("Cancelled!"):_(char*)
        } <b>with</b> {
            -- drag/drop behavior: must be before click (see below)
            <b>await</b> evt?Mouse?Motion
            <b>output</b> std _("Dragging..."):_(char*)
            <b>var</b> mouse = evt!Mouse!Motion
            <b>watching</b> evt?Mouse?Button?Up {   -- terminates on mouse up
                -- tracks mouse motion to move the rectangle
                <b>loop</b> {
                    <b>set</b> rect = [pt, rect.size]
                        <b>where</b> {
                            <b>var</b> pt: Point = [add [mouse.pos.x,dxy.w], add [mouse.pos.y,dxy.h]]
                        }
                    <b>await</b> evt?Mouse?Motion
                    <b>set</b> mouse = evt!Mouse!Motion
                }
            }
            <b>output</b> std _("Dropped!"):_(char*)
        } <b>with</b> {
            -- behavior: must be the last
            -- otherwise conflicts w/ motion termination
            <b>await</b> evt?Mouse?Button?Up
            <b>output</b> std _("Clicked!"):_(char*)
        }
    }
}

-- task for redrawing
<b>spawn</b> {
    <b>loop</b> {
        <b>await</b> _1
        <b>output</b> pico Pico.Output.Clear
        <b>output</b> pico Pico.Output.Draw.Rect rect
        <b>output</b> pico Pico.Output.Present
    }
}

<b>call</b> pico_loop ()
</pre>

Comment on <img src="twitter.png" style="vertical-align:middle"> [@\_fsantanna](https://twitter.com/_fsantanna/status/1495115884637134852).

[0]: https://twitter.com/dubroy
[1]: https://dubroy.com/blog/three-ways-of-handling-user-input/
[2]: https://github.com/pdubroy/abro
[3]: http://www.ceu-lang.org/
[4]: https://youtu.be/eC1d5MevRbg

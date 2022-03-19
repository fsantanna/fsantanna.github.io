# A Toy Problem: Drag, Click, or Cancel

<img src="twitter.png" style="vertical-align:middle">
[@_fsantanna](https://twitter.com/_fsantanna)

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

```
^"int.ceu"
^"pico.ceu"

-- rectangle to control
var rect: Rect = [[_10,_10],[_5,_5]]

-- tasks to behaviors: cancel, drag and drop, click
spawn {
    -- outer loop to restart after each behavior is detected
    loop {
        -- first detects the first click on the rectangle
        var dxy: Size   -- holds the offset to its center
        {
            var mouse: Point
            await evt?Mouse?Button?Down until isPointInsideRect [mouse,rect]
                where {
                    set mouse = evt!Mouse!Button!Down.pos
                }
            set dxy = [sub [rect.pos.x,mouse.x], sub [rect.pos.y,mouse.y]]
        }

        -- then either cancel, drag/drop, or click
        paror {
            -- cancel behavior: restores the original position on any key
            var orig = rect
            await evt?Key?Down until eq [evt!Key!Down.key,_SDLK_ESCAPE]
            set rect = orig
            output std _("Cancelled!"):_(char*)
        } with {
            -- drag/drop behavior: must be before click (see below)
            await evt?Mouse?Motion
            output std _("Dragging..."):_(char*)
            var mouse = evt!Mouse!Motion
            watching evt?Mouse?Button?Up {   -- terminates on mouse up
                -- tracks mouse motion to move the rectangle
                loop {
                    set rect = [pt, rect.size]
                        where {
                            var pt: Point = [add [mouse.pos.x,dxy.w], add [mouse.pos.y,dxy.h]]
                        }
                    await evt?Mouse?Motion
                    set mouse = evt!Mouse!Motion
                }
            }
            output std _("Dropped!"):_(char*)
        } with {
            -- behavior: must be the last
            -- otherwise conflicts w/ motion termination
            await evt?Mouse?Button?Up
            output std _("Clicked!"):_(char*)
        }
    }
}

-- task for redrawing
spawn {
    loop {
        await _1
        output pico Pico.Output.Clear
        output pico Pico.Output.Draw.Rect rect
        output pico Pico.Output.Present
    }
}

call pico_loop ()
```

Comment on <img src="twitter.png" style="vertical-align:middle"> [@_fsantanna](https://twitter.com/_fsantanna/status/1495115884637134852).

[0]: https://twitter.com/dubroy
[1]: https://dubroy.com/blog/three-ways-of-handling-user-input/
[2]: https://github.com/pdubroy/abro
[3]: http://www.ceu-lang.org/
[4]: https://youtu.be/eC1d5MevRbg

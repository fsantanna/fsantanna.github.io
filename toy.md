# A Toy Problem: Drag, Click, or Cancel

<img src="bsky.svg" style="vertical-align:middle" height="25">
[@fsantanna](https://bsky.app/profile/fsantanna.bsky.social)

- `[fev/22]`: first posted
- `[jul/24]`: updated to `ceu-v0.4`
- `[ago/25]`: updated to `atmos-v0.2`

Patrick Dubroy ([@dubroy][dubroy]) proposed a [toy problem][toy] to handle user
input:

> The goal is to implement a square that you can either drag and drop, or
> click. The code should distinguish between the two gestures: a click
> shouldn’t just be treated as a drop with no drag. Finally, when you’re
> dragging, pressing escape should abort the drag and reset the object back to
> its original position.

He proposed solutions using three different implementation techniques:
    Event listeners, Polling, and Process-oriented.
For the process-oriented approach he uses his Esterel-inspired [Abro.js][abro]
and argues that it provides "a clear, explicit sequencing between the different
states".

Since he mentions [Céu][ceu] and since I'm currently working on its upcoming
version [Atmos][atmos], I felt motivated to also post a solution to the
problem.
The solution in Atmos is similar to his, and uses the `par-or` and `watching`
constructs to safely abort behaviors that did not complete.
A small difference worth mentioning is relying on the deterministic scheduling
semantics of Atmos to eliminate a state variable (`didDrag`).
Here's [the solution][code] with [an accompanying video][video]:

<pre>
;; skip some SDL initialization

;; rectangle to control and text to display

var rect = @{ x=108,y=108, w=40,h=40 }
var text = " "

;; task to redraw the rectangle in the current position
<b>spawn</b> {
    <b>every</b> :sdl.draw {
        REN::setDrawColor(0x000000)
        REN::clear()
        REN::setDrawColor(0xFFFFFF)
        REN::fillRect(rect)
        sdl.write(FNT, text, @{x=256/2, y=200})
        REN::present()
    }
}

;; outer loop restarts after each behavior is detected
<b>loop</b> {
    ;; 1. detects first click on the rectangle
    <b>val</b> click = <b>await</b>(SDL.event.MouseButtonDown, \{point_vs_rect(it, rect)})
    <b>val</b> orig = @{x=rect.x, y=rect.y, w=rect.w, h=rect.h}
    <b>set</b> text = "... clicking ..."

    ;; 2. either cancel, drag/drop, or click
    <b>par_or</b> {
        ;; cancel: restores the original position on key :Escape
        <b>await</b>(SDL.event.KeyDown, :Escape)
        <b>set</b> rect = orig
        <b>set</b> text = "!!! CANCELLED !!!"
    } with {
        <b>par_or</b> {
            ;; drag/drop task: must be before click (see below)
            <b>await</b>(SDL.event.MouseMotion)
            <b>set</b> text = "... dragging ..."
            <b>await</b>(SDL.event.MouseButtonUp)
            <b>set</b> text = "!!! DRAGGED !!!"
        } <b>with</b> {
            ;; tracks mouse motion to move the rectangle
            <b>every</b> SDL.event.MouseMotion \{
                <b>set</b> rect.x = orig.x + (it.x - click.x)
                <b>set</b> rect.y = orig.y + (it.y - click.y)
            }
        }
    } <b>with</b> {
        ;; click task: must be the last
        ;; otherwise conflicts with motion termination
        <b>await</b>(SDL.event.MouseButtonUp)
        <b>set</b> text = "!!! CLICKED !!!"
    }
}
</pre>

Comment on <img src="bsky.svg" style="vertical-align:middle" height="25">
[@fsantanna](https://bsky.app/profile/fsantanna.bsky.social/post/3kxsdzxbk5k2h).


[dubroy]: https://twitter.com/dubroy
[toy]:    https://dubroy.com/blog/three-ways-of-handling-user-input/
[abro]:   https://github.com/pdubroy/abro
[ceu]:    http://www.ceu-lang.org/
[atmos]:  https://github.com/atmos-lang/atmos
[code]:   https://github.com/atmos-lang/atmos/blob/main/exs/click-drag-cancel.atm
[video]:  https://youtu.be/eC1d5MevRbg

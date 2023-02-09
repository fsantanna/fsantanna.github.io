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

Since he mentions [Ceu][3] and since I'm currently working on its upcoming
version, I felt motivated to also post a solution to the problem.
The solution in Ceu is similar to his, and uses the `par-or` and `awaiting`
constructs to safely abort behaviors that did not complete.
A small difference worth mentioning is relying on the deterministic scheduling
semantics of Ceu to eliminate a state variable (`didDrag`).
Here's the complete solution with [an accompanying video][4]:

<pre>
^["@/pico/pico.ceu"]

pico-state-set-title("pico-Ceu: Click, Drag, or Cancel")

;; outer task with nested tasks to redraw, cancel, drag and drop, and click
<b>spawn</b> {
    ;; rectangle to control
    <b>var</b> rect :Rect = [[10,10],[5,5]]

    ;; task to redraw the rectangle in the current position
    <b>spawn</b> {
        <b>every</b> :Pico.Draw {
            pico-state-set-color-clear([0,0,0,255])
            pico-output-draw-rect(rect)
        }
    }

    ;; outer loop restarts after each behavior is detected
    <b>loop</b> {
        ;; 1. detects first click on the rectangle
        <b>await</b> :Pico.Mouse.Button.Dn, pico-point-vs-rect?(<b>evt</b>.pos,rect)
        println("> clicking...")
        <b>val</b> orig  :Rect = copy(rect)
        <b>val</b> click :XY   = copy(<b>evt</b>.pos)

        ;; 2. either cancel, drag/drop, or click
        <b>par-or</b> {
            ;; cancel task: restores the original position on key ESC
            <b>await</b> :Pico.Key.Dn, (<b>evt</b>.key == :Key-Escape)
            <b>set</b> rect = copy(orig)
            println("<<< Cancelled!")
        } <b>with</b> {
            ;; drag/drop task: must be before click (see below)
            <b>await</b> :Pico.Mouse.Motion
            println("> dragging...")
            <b>awaiting</b> :Pico.Mouse.Button.Up { ;; terminates on mouse up
                ;; tracks mouse motion to move the rectangle
                <b>loop</b> {
                    <b>set</b> rect.pos.x = orig.pos.x + (<b>evt</b>.pos.x - click.x)
                    <b>set</b> rect.pos.y = orig.pos.y + (<b>evt</b>.pos.y - click.y)
                    <b>await</b> :Pico.Mouse.Motion
                }
            }
            println("<<< Dragged!")
        } <b>with</b> {
            ;; click task: must be the last
            ;; otherwise conflicts with motion termination
            <b>await</b> :Pico.Mouse.Button.Up
            println("<<< Clicked!")
        }
    }
}

pico-loop()
</pre>

Comment on <img src="twitter.png" style="vertical-align:middle">
[@\_fsantanna](https://twitter.com/_fsantanna/status/1495115884637134852).

[0]: https://twitter.com/dubroy
[1]: https://dubroy.com/blog/three-ways-of-handling-user-input/
[2]: https://github.com/pdubroy/abro
[3]: https://github.com/fsantanna/dceu
[4]: https://youtu.be/eC1d5MevRbg

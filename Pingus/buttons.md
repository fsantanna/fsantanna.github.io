# Menu buttons as tasks

<img src="twitter.png" style="vertical-align:middle">
[@\_fsantanna](https://twitter.com/_fsantanna)

<img src="menu.gif" align="right" width="350">

- On rewriting [Pingus][pingus.md] from C++ to Ceu
    - A structured [main menu](menu.md)
    - **Menu buttons as tasks**

In the [previous post](menu.md), we discussed the outermost code to alternate
between the main menu and game screen, leaving out the implementations of the
`main_menu` and `menu_button` tasks:

<pre>
-- enumeration with the possible menu choices
<b>type</b> Menu = <Story=(), Editor=(), ...>

-- task signatures for the menu and buttons
<b>task</b> main_menu: () -> Menu
    -- returns the chosen screen to navigate
<b>task</b> menu_button: [pos:Point, lbl:String] -> ()
    -- receives a position and label to show

-- spawns the game code
<b>spawn</b> {
    -- the outer loop                         1️⃣
    <b>loop</b> {
        -- main menu
        <b>var</b> opt = <b>await</b> <b>spawn</b> main_menu ()    2️⃣

        -- chosen screen
        <b>var</b> lbl = <b>ifs</b> {
            opt ? Story  { "Story"  }
            opt ? Editor { "Editor" }
            ... -- other options
        }
        <b>await</b> <b>spawn</b> menu_button [[0,0], lbl]  2️⃣

        -- loops back to menu after screen terminates
    }
}

-- enters the SDL engine loop
<b>call</b> pico_loop ()
</pre>

---

The `main_menu` task spawns a set of `menu_button` tasks in parallel, each one
corresponding to a menu option, and returns the clicked option as an
enumeration:

<pre>
<b>task</b> main_menu: () -> Menu {
    <b>par</b> {
        <b>await</b> <b>spawn</b> menu_button [[-125,35], "Story"]  1️⃣
        <b>return</b> Menu.Story
    } <b>with</b> {
        <b>await</b> <b>spawn</b> menu_button [[ 125,35], "Editor"] 2️⃣
        <b>return</b> Menu.Editor
    } <b>with</b> {
        ...     -- other options
    }
}
</pre>

The `main_menu` expects that a `menu_button` terminates when it is clicked (2️⃣)
in order to return the corresponding enumeration to the outermost code (2️⃣).
Like the outermost code, we use `spawn-await`'s direct style to nest arbitrary
task.
Here, we also use the `par` construct which is a rich structured mechanism
whose branches expand to anonymous tasks that capture its enclosing lexical
context.
For instance, note that the `return` statements inside the `par` branches
terminate the enclosing `main_menu` as a whole.
I believe this kind of construct is not possible in programming languages that
provide [structured concurrency mechanisms](../sc.md) as libraries.

Finally, the most relevant structured mechanism in the menu is how Ceu handles
the lifespan of tasks.
A task in Ceu is like a local variable, i.e., its lifespan is attached to the
block enclosing it.
In the `main_menu`, each `par` branch spawns an anonymous task, and each
anonymous task spawns a `menu_button` task.
Hence, the `main_menu` handles at least 2x tasks for each menu option, which
are all active at the same time.
When a `return` is reached, it escapes all blocks in the parent task, aborting
all nested tasks automatically.
This hierarchy of tasks is one of the control-flow pattern identified in the
[previous post](pingus.md):

- **Lifespan Hierarchies:** Entities typically form a lifespan hierarchy in
   which a terminating container entity automatically destroys its managed
   children.
    - Examples: UI containers, particle systems.

---

<!--
3. **Dispatching Hierarchies:** Entities typically form a dispatching hierarchy
   in which a container that receives a stimulus automatically forwards it to
   its managed children.
    - Examples: redraw & update callbacks.

```
task menu_button: [pos:Point,tit:_(char*)] -> () {
    var size: Size
    output pico Pico.Output.Get.Size.Image [/size, _("data/images/menuitem.png")]

    spawn {
        every evt?Draw {
            output pico Pico.Output.Draw.Image [arg.pos, _("data/images/menuitem.png")]
            output pico Pico.Output.Set.Font [_("data/fonts/film-cryptic/Filmcryptic.ttf"),_45]
            output pico Pico.Output.Draw.Text [arg.pos, arg.tit]
        }
    }

    await evt?Mouse?Button?Down until isPointVsRect [pos,[arg.pos,size]]
        where {
            var pos = evt!Mouse!Button!Down.pos
        }
}
```
-->

---

- Do you know mechanisms for anonymous tasks in other languages?

Comment on <img src="twitter.png" style="vertical-align:middle"> [@\_fsantanna](https://twitter.com/_fsantanna/status/TODO).

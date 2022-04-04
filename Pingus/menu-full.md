# A structured main menu: full code

<img src="../twitter.png" style="vertical-align:middle">
[@\_fsantanna](https://twitter.com/_fsantanna)

<img src="menu.gif" align="right" width="250">

- On rewriting [Pingus](pingus.md) from C++ to Ceu
    - A structured [main menu](menu.md)
    - Menu [buttons](buttons.md) as local tasks
    - A self-reacting [button](button.md)
    - **A structured main menu: full code** 👈 (this post)

This is the full commented code for the structured main menu:

<pre>
-- The language still misses very basic functionality like primitive integers
-- and strings. Hence, the code relies on native C symbols, which are
-- prefixed with an underscore (e.g., `_255` and `_("Story")`).
-- `Pico` is a simple graphical library based on SDL.

-- Includes auxiliary files
^"prelude.ceu"
^"int.ceu"
^"float.ceu"
^"pico.ceu"

-- Sets the initial window properties
<b>output</b> Pico.Set.Title _("Pingus")
<b>output</b> Pico.Set.Size [_641,_481]
<b>output</b> Pico.Set.Zoom [_100,_100]
<b>output</b> Pico.Set.Grid _0
<b>output</b> Pico.Set.Color.Clear [_0,_0,_0,_255]
<b>output</b> Pico.Clear
<b>output</b> Pico.Set.Auto _0

-- Menu enumeration
<b>type</b> Menu = <Story=(),Editor=(),Levelsets=(),Options=(),Exit=()>

-- Task to show a single button:
--  - receives point and label to show
--  - terminates on a mouse click
<b>task</b> menu_button: [pos:Point,lbl:_(char*)] -> () -> () {
    <b>var</b> size: Size  -- get dimensions to detect click
    <b>output</b> Pico.Get.Size.Image [/size, _("data/images/menuitem.png")]

    -- <b>task</b> to show itself on every Draw event
    <b>spawn</b> {
        every evt?Draw {
            -- draw image with label on top
            <b>output</b> Pico.Draw.Image [arg.pos, _("data/images/menuitem.png")]
            <b>output</b> Pico.Set.Font   [_("data/fonts/film-cryptic/Filmcryptic.ttf"),_45]
            <b>output</b> Pico.Draw.Text  [arg.pos, arg.lbl]
        }
    }

    -- awaits a mouse click inside its region to terminate
    <b>await</b> evt?Mouse?Button?Down <b>until</b> isPointVsRect [pos,[arg.pos,size]]
        where {
            <b>var</b> pos = evt!Mouse!Button!Down.pos
        }
}

-- Task to show the main menu:
--  - spawns a `menu_button` task for each button in the menu
--  - terminates when any button is clicked
--  - returns the `Menu` enumeration associated with the click
<b>task</b> main_menu: () -> () -> Menu {
    -- spawns multiple tasks in parallel to wait for the buttons
    <b>par</b> {
        -- spawns and awaits each button
        <b>await</b> <b>spawn</b> menu_button [[_(-125),_(  35)], _("Story")]
        -- returns the associated enumeration
        <b>return</b> Menu.Story
    } <b>with</b> {
        <b>await</b> <b>spawn</b> menu_button [[_( 125),_(  35)], _("Editor")]
        <b>return</b> Menu.Editor
    } <b>with</b> {
        <b>await</b> <b>spawn</b> menu_button [[_(-125),_( -35)], _("Levelsets")]
        <b>return</b> Menu.Levelsets
    } <b>with</b> {
        <b>await</b> <b>spawn</b> menu_button [[_( 125),_( -35)], _("Options")]
        <b>return</b> Menu.Options
    } <b>with</b> {
        <b>await</b> <b>spawn</b> menu_button [[_(   0),_(-105)], _("Exit")]
        <b>return</b> Menu.Exit
    }
}

-- Main application:
--  - a loop that alternates between the menu and the chosen button
<b>spawn</b> {
    <b>loop</b> {
        -- spawns the main menu and awaits its termination
        <b>var</b> opt = <b>await</b> <b>spawn</b> main_menu ()

        -- maps the chosen button to a label to show next
        <b>var</b> lbl: _(char*) = <b>ifs</b> {
            opt ? Story     { _("Story")     }
            opt ? Editor    { _("Editor")    }
            opt ? Levelsets { _("Levelsets") }
            opt ? Options   { _("Options")   }
            opt ? Exit      { _("Exit")      }
        }

        -- shows and awaits the chosen button in the center of the screen
        <b>await</b> <b>spawn</b> menu_button [[_0,_0], lbl]
    }
}

-- Enters the engine main loop.
-- Now the application is entirely reactive.
-- In this code, only the individual buttons (`menu_button` task) react to
-- events (evt?Draw and evt?Mouse).
<b>call</b> pico_loop ()
</pre>

Comment on <img src="../twitter.png" style="vertical-align:middle">
[@\_fsantanna](https://twitter.com/_fsantanna/status/TODO).

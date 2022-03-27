# A Structured Main Menu

<img src="twitter.png" style="vertical-align:middle">
[\_@fsantanna](https://twitter.com/_fsantanna)

<img src="menu.gif" align="right" width="350">

A main menu typically displays a set of buttons that allows players to navigate
the game.
Selecting a button transfers the game to another screen, such as an options
screen or the gameplay itself.
Eventually, after the chosen screen terminates, the game transits back to the
main menu.

In the figure demonstrating our implementation, we symbolize the chosen screens
as clickable buttons associated with the user choices.
Clicking the button again terminates the screen and returns to the main menu.
Our goal is to apply [structured reactive techniques](pingus.md) in the
implementation.
Let's discuss it in a top-down approach, starting with the main application:

```
output Set.Title "Pingus"
... -- other trivial initializations

-- enumeration with the possible main menu choices
enum Menu = <Story, Editor, Levelsets, Options, Exit>

-- menu button implementation
task menu_button: [pos:Point, lbl:String] -> () {
    ... -- discussed further
}

-- main menu implementation
task main_menu: () -> Menu {
    ... -- discussed further
}

-- spawns the game code
spawn {
    -- the outer loop
    loop {
        -- main menu
        var opt = await spawn main_menu ()

        -- chosen screen
        var lbl = ifs {
            opt ? Story     { "Story"     }
            opt ? Editor    { "Editor"    }
            opt ? Levelsets { "Levelsets" }
            opt ? Options   { "Options"   }
            opt ? Exit      { "Exit"      }
        }
        await spawn menu_button [[0,0], lbl]

        -- loops back to main menu after chosen screen terminates
    }
}

-- enters the SDL engine loop
call pico_loop ()
```

The important structured mechanism is the outer loop that alternates between
the main menu and the chosen screen.
These screens can be arbitrarily complex and are handled with the `spawn-await`
combination: spawn the task in the background and await its termination.
The language keeps the loop context alive (i.e., locals and program counter),
while the current screen executes in a separate task, possibly reacting to
events and spawning auxiliary tasks.

- what about C++ Pingus
- push_screen
- pop_screen
    - explicit stack management, imagine a language w/o locals
    - what a language w/o stack would do



<!--
- top down

2. **Continuation Passing:** The completion of a long-lasting activity may
   carry a continuation, i.e., some action to execute next.
    - Examples: interactive dialogs, menu transitions.
3. **Dispatching Hierarchies:** Entities typically form a dispatching hierarchy
   in which a container that receives a stimulus automatically forwards it to
   its managed children.
    - Examples: redraw & update callbacks.
4. **Lifespan Hierarchies:** Entities typically form a lifespan hierarchy in
   which a terminating container entity automatically destroys its managed
   children.
    - Examples: UI containers, particle systems.

```
task menu_button: [pos:Point,tit:_(char*)] -> () {
    ...
}

task menu: () -> Menu {
    par {
        await spawn menu_button [[_(-125),_(  35)], _("Story")]
        return Menu.Story
    } with {
        await spawn menu_button [[_( 125),_(  35)], _("Editor")]
        return Menu.Editor
    } with {
        await spawn menu_button [[_(-125),_( -35)], _("Levelsets")]
        return Menu.Levelsets
    } with {
        await spawn menu_button [[_( 125),_( -35)], _("Options")]
        return Menu.Options
    } with {
        await spawn menu_button [[_(   0),_(-105)], _("Exit")]
        return Menu.Exit
    }
}
```

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

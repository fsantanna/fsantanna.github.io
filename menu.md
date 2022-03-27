# A Basic Main Menu

<img src="twitter.png" style="vertical-align:middle">
[\_@fsantanna](https://twitter.com/_fsantanna)

A game main menu typically displays a set of buttons that allows players to
navigate in the game.
Selecting a button transfers the game to another screen, such as a
configuration screen or the gameplay itself.
Eventually, after the chosen screen completes, the game transits back to the
main menu.
In the figure demonstrating our implementation, we symbolize the chosen screens
as clickable buttons representing the user choice.
Clicking the button again completes the screen and returns to the main menu.

<img src="menu.gif" align="right" width="350">

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
output pico Pico.Output.Set.Title _("Pingus")
...

enum Menu = <Story, Editor, Levelsets, Options, Exit>   :one:

task menu: () -> Menu {
    ...
}

spawn {
    loop {
        var opt = await spawn menu ()
        var str: _(char*) = ifs {
            opt ? Story     { _("Story")     }
            opt ? Editor    { _("Editor")    }
            opt ? Levelsets { _("Levelsets") }
            opt ? Options   { _("Options")   }
            opt ? Exit      { _("Exit")      }
        }
        await spawn menu_button [[_0,_0], str]
    }
}

call pico_loop ()
```

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

# A structured main menu

<img src="../twitter.png" style="vertical-align:middle">
[@\_fsantanna](https://twitter.com/_fsantanna)

<img src="menu.gif" align="right" width="350">

- On rewriting [Pingus](pingus.md) from C++ to Ceu
    - **A structured main menu**
<!--
    - Menu [buttons](buttons.md) as local tasks
-->

A main menu typically displays a set of buttons that allows players to navigate
the game.
Selecting a button transfers the game to another screen, such as an options
screen or the gameplay itself.
Eventually, after the chosen screen terminates, the game transits back to the
main menu.

In the demo figure, we symbolize the chosen screens as clickable buttons
associated with the user choices.
Clicking the button again terminates the screen and returns to the main menu.
Our goal is to apply [structured reactive techniques](pingus.md) in our
implementation in Ceu.
Let's discuss it in a top-down approach, starting with the main application:

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

The relevant structured mechanism in the code is the outer loop (1️⃣), which
alternates between the main menu and the chosen screen.
These screens can be arbitrarily complex and are handled with the `spawn-await`
combination (2️⃣), which resembles conventional function calls: spawn the task
and await its termination.
We will discuss the `main_menu` and `menu_button` implementations in future
posts.
While the main task waits in the loop, the current screen executes in a
separate task, possibly reacting to events and spawning auxiliary tasks.
In the meantime, the language keeps the loop context alive (i.e., locals and
program counter) similarly to coroutines.
This [direct style][1] with `spawn-await` contrasts with the arguably more
intricate *continuation passing style (CPS)*, which is one of the control-flow
pattern identified in the [previous post](pingus.md):

- **Continuation Passing:** The completion of a long-lasting activity may
   carry a continuation, i.e., some action to execute next.
    - Examples: interactive dialogs, menu transitions.

The original implementation in C++ uses CPS and pushes the screen navigation to
occur inside the [button click callback][2], thus textually far away from the
main game function:

```cpp
MainMenu::MainMenu () {
    ...
    story_but = gui->create<MenuButton>(...)
    ...  // other menu buttons
}

void MainMenu::on_click (MenuButton* button) {
    if (button == story_but) {
        ...
        Screen::push_screen(worldmap);  // screen navigation
        ...
    } else ... {    // other buttons actions
        ...
    }
}
```

The implementation in C++ also relies on an [explicit stack][3] to alternate
between the main menu and the chosen screen: it pushes a new screen on top of
the main menu, which must be explicitly popped when terminating:

```cpp
void Worldmap::update (...) {
    ...
    if (exit_worldmap) {
        Screen::pop_screen();
    }
    ...
}
void WorldmapCloseButton::on_click() {
    Screen::pop_screen();
}
```

Not only this approach requires the called screen to be aware of its parent
navigation track, but also relies on a data structure to simulate a
control-flow mechanism.

---

[1]: https://handwiki.org/wiki/Direct_style
[2]: https://github.com/Pingus/pingus/blob/master/src/pingus/screens/pingus_menu.cpp#L178
[3]: https://github.com/Pingus/pingus/blob/master/src/pingus/worldmap/worldmap_screen.cpp#L179

- What other typical game behaviors rely on CPS?
- How do you feel about the direct style implementation?
- Is the top-down code understandable (even without the menu and button details)?

Comment on <img src="../twitter.png" style="vertical-align:middle">
[@\_fsantanna](https://twitter.com/_fsantanna/status/TODO).

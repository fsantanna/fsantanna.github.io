# A self-reacting button

<img src="../twitter.png" style="vertical-align:middle">
[@\_fsantanna](https://twitter.com/_fsantanna)

<img src="menu.gif" align="right" width="350">

- On rewriting [Pingus](pingus.md) from C++ to Ceu
    - A structured [main menu](menu.md)
    - Menu [buttons](buttons.md) as local tasks
    - **A self-reacting button**          👈 (this post)

In the previous posts, we discussed the [outermost code](menu.md) to alternate
between screens and the [main menu buttons](buttons.md):

<pre>
-- enumeration with the possible menu choices
<b>type</b> Menu = <Story=(), Editor=(), ...>

<b>task</b> menu_button: [pos:Point, lbl:String] -> ()  👈 (this post)
    -- receives a position and label to show

<b>task</b> main_menu: () -> Menu {
    <b>par</b> {
        <b>await</b> <b>spawn</b> menu_button [[-125,35], "Story"]  3️⃣
        <b>return</b> Menu.Story
    } <b>with</b> {
        <b>await</b> <b>spawn</b> menu_button [[ 125,35], "Editor"] 3️⃣
        <b>return</b> Menu.Editor
    } <b>with</b> {
        ... -- other options
    }
}

-- spawns the game code
<b>spawn</b> {                                     1️⃣
    -- the outer loop
    <b>loop</b> {
        -- main menu
        <b>var</b> opt = <b>await</b> <b>spawn</b> main_menu ()  2️⃣

        -- chosen screen
        <b>var</b> lbl = <b>ifs</b> {
            opt ? Story  { "Story"  }
            opt ? Editor { "Editor" }
            ... -- other options
        }
        <b>await</b> <b>spawn</b> menu_button [[0,0], lbl]

        -- loops back to menu after screen terminates
    }
}

-- enters the SDL engine loop
<b>call</b> pico_loop ()
</pre>

In this post, we complete the structured main menu with the `menu_button`
implementation.
The outermost code (1️⃣) spawns the `main_menu` task (2️⃣), which spawns several
`menu_button` tasks in parallel (3️⃣).
Each button receives a position and label to show, and is responsible for
redrawing (1️⃣) and terminating itself on a mouse click (2️⃣) as follows:

<pre>
<b>task</b> menu_button: [pos:Point, tit:String] -> () {
    <b>var</b> size: Size
    <b>output</b> Get.Size.Image [/size, "data/images/menuitem.png"]

    <b>spawn</b> {
        <b>every</b> evt?Draw {    1️⃣
            <b>output</b> Draw.Image [arg.pos, "data/images/menuitem.png"]
            <b>output</b> Set.Font   ["data/fonts/film-cryptic/Filmcryptic.ttf",45]
            <b>output</b> Draw.Text  [arg.pos, arg.tit]
        }
    }

    <b>await</b> and [             2️⃣
        evt?Mouse?Button?Down
        isPointVsRect [evt!Mouse!Button!Down.pos, [arg.pos,size]]
    ]
}
</pre>

The `menu_button` task spawns a dedicated task to redraw itself on each
occurrence of `evt?Draw` (1️⃣), which is an engine event to signal that the
screen must be updated.
While the redrawing task executes in the background, the `menu_button` also
waits for an `evt?Mouse?Button?Down` event (2️⃣).
If the click occurs inside the button, the `menu_button` task awakes and
terminates.
Its termination awakes the `main_menu` task, which in turn returns the chosen
screen to the outermost code.

The relevant structured mechanism in this code is how a deep nested task, such
as `menu_button`, can react directly to engine events directly (2️⃣ and 2️⃣),
bypassing the task hierarchy entirely.
This self-dispatching mechanism is one of the control-flow patterns in the
[previous post](pingus.md):

- **Dispatching Hierarchies:** Entities typically form a dispatching hierarchy
   in which a container that receives a stimulus automatically forwards it to
   its managed children.
    - Examples: redraw & update callbacks.

---

The original implementation in C++ needs to dispatch the events explicitly
through the containers hierarchy, which is split in multiple files.
Starting at the engine, the `draw` method is dispatched through the path
`ScreenManager` -> `GUIScreen` -> `GroupComponent` -> `SurfaceButton`:

```cpp
// screen_manager.cpp:
// https://github.com/Pingus/pingus/blob/master/src/engine/screen/screen_manager.cpp#L200
void ScreenManager::update (...) {
    ...
    get_current_screen()->draw(...);
    ...
}

// gui_screen.cpp
// https://github.com/Pingus/pingus/blob/master/src/engine/screen/gui_screen.cpp#L40
void GUIScreen::draw (...) {
    ...
    gui_manager->draw(...);
    ...
}

// group_component.cpp
// https://github.com/Pingus/pingus/blob/master/src/engine/gui/group_component.cpp#L47
void GroupComponent::draw (...) {
    ...
    for (Components::iterator i...) {
        ...
        (*i)->draw(...);
    }
    ...
}

// surface_button.cpp
// https://github.com/Pingus/pingus/blob/master/src/engine/gui/surface_button.cpp#L52
void SurfaceButton::draw (...) {
    ...
    gc.draw(...);
    ...
}
```

Understanding the method dispatch requires examining at least 4 files, not
including the class hierarchy.
For instance, we started examining the `PingusMenu` implementing the main menu,
which extends the `GUIScreen` in the dispatching path.
This is only for redrawing, as the `update` dispatch goes through a similar
hierarchy.

---

An important detail is that self dispatching in Ceu relies on two properties of
its [synchronous execution model](../sc.md):

- Synchronous hypothesis: the button tasks must react to the events with zero
    delay, otherwise the game would freeze.
    This is a common assumption for C++ methods and callbacks as well.
- Lexical determinism: tasks react in sequence in the order they appear in the
    code. It's the programmer responsibility to arrange them in a meaninful
    way. For instance, a background image should be spawned before the buttons.
    This is also a common assumption in C++ dispatching call, although they are
    more explicit.

---

- How long can dispatching and class hierarchies be in C++?
- How unclear can lexical order dispatching become?

Comment on <img src="../twitter.png" style="vertical-align:middle">
[@\_fsantanna](https://twitter.com/_fsantanna/status/TODO).

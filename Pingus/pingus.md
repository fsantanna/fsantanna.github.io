# On rewriting Pingus from C++ to Ceu

<img src="../twitter.png" style="vertical-align:middle">
[@\_fsantanna](https://twitter.com/_fsantanna)

- **On rewriting Pingus from C++ to Ceu** 👈 (this post)
    - A structured [main menu](menu.md)
    - Menu [buttons](buttons.md) as local tasks
    - A self-reacting [button](button.md)
    - A structured main menu: [full code](menu-full.md)

<img src="pingus.png" align="right" width="350">

A few years ago, I wrote a [blog post][3] (and a simplified [paper][4]) on
rewriting Pingus from C++ to Ceu.
[Pingus][1] is an open-source clone of [Lemmings][2], a puzzle-platformer video
game.
We rewrote 50% of the codebase, or 20k lines of code, which constitutes the
entire game logic.
The study advocates *"Structured Synchronous Reactive Programming (SSRP)"* as a
more productive alternative for game logic development:

> Ceu supports reactive control-flow primitives that eliminate callbacks and
> let programmers write code in direct and sequential style.
> Structured reactivity helps describing complex control-flow relationships in
> the game logic more concisely.

SSRP can be considered a flavor of *"Structured Concurrency"* (discussed in a
previous [blog post](../sc.md)) with a focus on real-time interactive
applications, such as video games.

The bulk of the study is a qualitative analysis of the programming techniques
we applied during the rewriting process.
For typical game behaviors, we could reduce up to 20% of the source code (e.g.,
event detection, interactive animations, menu transitions, etc).
During the process, we also identified 5 control-flow patterns that likely
apply to other games.
In this context, a control-flow pattern is a recurring technique to describe
execution dependency and/or explicit ordering between statements.
They are:

1. **Finite State Machines:** State machines describe the behavior of entities
   by mapping event occurrences to transitions between states that trigger
   appropriate actions.
    - Examples: double-click detection, character animations.
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
5. **Signaling Mechanisms:** Entities often need to communicate explicitly
   through signaling mechanisms, especially if there is no hierarchy
   relationship between them.
    - Examples: key shortcuts, screen pausing.

The reason I'm revisiting this study is because I'm currently rewriting Pingus
to the upcoming version of Ceu.
The goal is twofold:

- Evolving the language implementation to a launching state.
- Discussing these control-flow patterns in future blog posts.

[1]: http://pingus.seul.org/
[2]: https://en.wikipedia.org/wiki/Lemmings_(video_game)
[3]: https://fsantanna.github.io/pingus/
[4]: http://ceu-lang.org/chico/ceu_sbgames18.pdf

---

- Do you identify other general control-flow patterns in games?

Comment on <img src="../twitter.png" style="vertical-align:middle">
[@\_fsantanna](https://twitter.com/_fsantanna/status/1508091964390092810).

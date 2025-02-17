# Gameplay

The player character is able to move and push boxes on the field. Many (but not all!) games have some kind of objective
for the player to achieve.  The objective for Sokoban-style games is typically to push boxes onto a goal spot.  There's
nothing stopping the player from doing this now, but the game also isn't checking for success.  The player might achieve
the objective without realizing it!  Let's update the game to check for the success state.

Let's think about what we'll need to add to this game to check for the success condition and to notify the user
when they've beaten the level:

- A `resource` for tracking the game state
  - Is the game in progress or completed?
  - How many move has the player made?
- A `system` for checking if the user has completed their objective
- A `system` for updating the number of moves made
- UI for reporting game state

## Gameplay Resource

We're choosing to use a `resource` to track game state because the game state is
not associated with a specific entity. Let's start by defining a `Gameplay` resource.

```rust
// resources.rs
{{#include ../../../code/rust-sokoban-c02-05/src/components.rs:gameplay_state}}
```

`Gameplay` has two fields: `state` and `moves_count`. These are used to track the
current state of the game (is the game still in play, or has the player won?) and
the number of moves made.  `state` is described by an `enum`.

The eagle-eyed reader will note that we used a macro to derive the `Default` trait
for `Gameplay`, and a `#[default]` annotation for the  `GameplayState` enum. All this annotation does is tell the compiler that if we ever call `GameplayState::default()` we should get back `GameplayState::Playing`, which makes sense.

Now, when the game is started, the `Gameplay` resource will look like this:

```rust
Gameplay {
    state: GameplayState::Playing,
    moves_count: 0
}
```

## Step Counter System

We can increment `Gameplay`'s `moves_count` field to track the number of turns taken.
We already have a system dealing with user input in the input system, so let's adapt that for this purpose.

```rust
// systems/input.rs
{{#include ../../../code/rust-sokoban-c02-05/src/systems/input.rs:run_input_begin}}
    // Movement code omitted for clarity
    // .....
    // .....
    
{{#include ../../../code/rust-sokoban-c02-05/src/systems/input.rs:run_input_update_moves}}
```

Since we've already done the work to check if a player character will move in
response to a keypress, we can use that to determine when to increment the step
counter.

## Gameplay System

Next, let's integrate this resource with a new gameplay state system.  This
system will continuously check to see if all the boxes have the same
position as all the box spots. Once all the boxes are on all the box spots,
the game has been won!

Aside from `Gameplay`, this system only needs read-only access to the
`Position`, `Box`, and `BoxSpot` components.

If all box spots have a corresponding box at the same position, the game is over and the player has won.
Otherwise, the game is still in play.

```rust
// systems/gameplay.rs
{{#include ../../../code/rust-sokoban-c02-05/src/systems/gameplay.rs}}
```

Finally, let's run the gameplay system in our main update loop.

```rust
// main.rs
{{#include ../../../code/rust-sokoban-c02-05/src/main.rs}}
```

## Gameplay UI

The last step is to provide feedback to the user letting them know what the
state of the game is.  This requires a resource to track the state and a
system to update the state. We can adapt the `GameplayState` resource and
rendering system for this.

First, we'll implement `Display` for `GameplayState` so we can render the
state of the game as text. We'll use a match expression to allow `GameplayState`
to render "Playing" or "Won".

```rust
// resources.rs
{{#include ../../../code/rust-sokoban-c02-05/src/components.rs:gameplay_state_impl_display}}
```

Next, we'll add a `draw_text` method to rendering system, so it can print
`GameplayState` to the screen...

```rust
// rendering_systems.rs
{{#include ../../../code/rust-sokoban-c02-05/src/systems/rendering.rs:draw_text}}
```

...and then we'll add the `Gameplay` resource to `RenderingSystem` so we can
call `draw_text`, and use it all to render the state and number of moves.

```rust
// rendering.rs
{{#include ../../../code/rust-sokoban-c02-05/src/systems/rendering.rs:draw_gameplay_state}}
```

At this point, the game will provide basic feedback to the user:

- Counts the number of steps
- Tells the player when they have won

Here's how it looks.

![Sokoban play](./images/moves.gif)

There are plenty of other enhancements that can be made!

> **_CODELINK:_**  You can see the full code in this example [here](https://github.com/iolivia/rust-sokoban/tree/master/code/rust-sokoban-c02-05).

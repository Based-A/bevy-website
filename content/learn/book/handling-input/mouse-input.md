+++
title = "Mouse Input"
insert_anchor_links = "right"
[extra]
weight = 4
+++

Unlike keyboards, mice are multi-faceted: they can provide both button input _and_ motion.
When we take in input from a mouse, we're also taking in more data.

When the `mouse` feature is enabled in your game, Bevy will automatically:

- Add [`MouseButtonInput`], [`MouseMotion`], and [`MouseWheel`] messages to the `World`.
  - These messages record the actual input data from mouse button clicks, mouse motion, and scroll wheel actions.
- Initialize [`ButtonInput<MouseButton>`], [`AccumulatedMouseMotion`], and [`AccumulatedMouseScroll`] and resources.
  - These resources are updated with the input data that is captured in their respective message.
- Insert the [`mouse_button_input_system`], [`accumulate_mouse_motion_system`], and [`accumulate_mouse_scroll_system`] to the [`PreUpdate`] schedule.
  - These systems update their respective resources with all input actions and data that occur.

```rust
fn mouse_inputs(
    button_input: Res<ButtonInput<MouseButton>>,
    movement_input: Res<AccumulatedMouseMotion>,
    scroll_input: Res<AccumulatedMouseScroll>,
) {
    // Read a mouse button press.
    if button_input.pressed(MouseButton::LeftMouseButton) {
        println!("Left Mouse Button has been pressed.");
    }
    // See if the mouse is moving.
    if movement_input.delta == Vec2{x: 0.0, y: 0.0} {
        println!("Mouse isn't moving!");
    } else {
        println!("The mouse is moving!");
    }
    // Read the scroll wheel value.
    if scroll_input.delta.y > 10.0 {
        println!("We're scrolling too fast!");
    }
}
```

[`MouseButtonInput`]: https://docs.rs/bevy/latest/bevy/input/mouse/struct.MouseButtonInput.html
[`MouseMotion`]: https://docs.rs/bevy/latest/bevy/input/mouse/struct.MouseMotion.html
[`MouseWheel`]: https://docs.rs/bevy/latest/bevy/input/mouse/struct.MouseWheel.html
[`ButtonInput<MouseButton>`]: https://docs.rs/bevy/latest/bevy/input/mouse/enum.MouseButton.html
[`AccumulatedMouseMotion`]: https://docs.rs/bevy/latest/bevy/input/mouse/struct.AccumulatedMouseMotion.html
[`AccumulatedMouseScroll`]: https://docs.rs/bevy/latest/bevy/input/mouse/struct.AccumulatedMouseScroll.html
[`mouse_button_input_system`]: https://docs.rs/bevy/latest/bevy/input/mouse/fn.mouse_button_input_system.html
[`accumulate_mouse_motion_system`]: https://docs.rs/bevy/latest/bevy/input/mouse/fn.accumulate_mouse_motion_system.html
[`accumulate_mouse_scroll_system`]: https://docs.rs/bevy/latest/bevy/input/mouse/fn.accumulate_mouse_scroll_system.html

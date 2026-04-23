+++
title = "Touch Input"
insert_anchor_links = "right"
[extra]
weight = 5
+++

Last, but certainly not least, is touch input.
While keyboards, mice, and gamepads all have unique data to process, touch input is relatively simple in Bevy.
A [`Touch`] action is used to store the position and force of a touch input.
You can see all of the data a `Touch` action stores by visiting the docs page.

When the `touch` feature is enabled in your project, Bevy will automatically:

- Add a [`TouchInput`] message to the `World`.
  - This message records the input action.
- Initialize the [`Touches`] resource.
  - This resource stores the value provided by `TouchInput`.
- Insert the [`touch_screen_input_system`] into the `PreUpdate` schedule.
  - This system updates the `Touches` resource with the values of every `TouchInput` message received.

```rust
fn touch_system(touches: Res<Touches>) {
    for touch in touches.iter_just_pressed() {
        info!(
            "just pressed touch with id: {}, at: {}",
            touch.id(),
            touch.position()
        );
    }

    for touch in touches.iter_just_released() {
        info!(
            "just released touch with id: {}, at: {}",
            touch.id(),
            touch.position()
        );
    }

    for touch in touches.iter_just_canceled() {
        info!("canceled touch with id: {}", touch.id());
    }

    // You can also iterate all current touches and retrieve their state like this:
    for touch in touches.iter() {
        info!("active touch: {touch:?}");
        info!("  just_pressed: {}", touches.just_pressed(touch.id()));
    }
}
```

[`Touch`]: https://docs.rs/bevy/latest/bevy/input/touch/struct.Touch.html
[`TouchInput`]: https://docs.rs/bevy/latest/bevy/prelude/struct.TouchInput.html
[`Touches`]: https://docs.rs/bevy/latest/bevy/input/touch/struct.Touches.html
[`touch_screen_input_system`]: https://docs.rs/bevy/latest/bevy/input/touch/fn.touch_screen_input_system.html

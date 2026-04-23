+++
title = "Gamepad Input"
insert_anchor_links = "right"
[extra]
weight = 5
+++

In Bevy, gamepads are devices which rely on buttons, joysticks, triggers, and directional pads (d-pads).
They include everything from the controllers that come with gaming consoles to dance platforms in arcades to flight-sim and racing-sim accessories.
As long as their input can be translated to one of the many [`GamepadButton`] variants, Bevy can use it as a gamepad.

## Gamepad Input Data Types

Since gamepads can receive multiple types of data (button presses, joystick directions, trigger pulls, etc.), we can't just use a blanket approach like we do with keyboards.
Each type of input data is formatted uniquely

### Button Presses

### Joystick Input

### Trigger

## Using Multiple Gamepads

While we are still receiving button input, a gamepad can also supply axis input data from any thumbsticks it might have.
It might seem like we could treat the axis input data like we would motion on a mouse, but axis values aren't recording actual motion.
Instead we're receiving a 2D directional value, which results in the axis value input providing additional data and allowing for different configurations.

When the `gamepad` feature is enabled in your project, Bevy will automatically setup a number of [message events] along with two systems: [`gamepad_connection_system`] and [`gamepad_event_processing_system`], both placed in `PreUpdate`.

You'll notice that we aren't setting up any resources to directly access.
This is because each Bevy uses various `Settings` structs within a larger [`GamepadSettings`] to determine similar conditions and thresholds to those in `ButtonInput`.
Since multiple gamepads can be connected at the same time, placed these settings within a `Resource` wouldn't make sense.
Instead, each gamepad can have it's own `GamepadSettings`, allowing you to adjust each gamepad based on any conditions or situations you might want.

```rust
fn gamepad_system(gamepads: Query<(Entity, &Gamepad)>) {
    for (entity, gamepad) in &gamepads {
        if gamepad.just_pressed(GamepadButton::South) {
            info!("{} just pressed South", entity);
        } else if gamepad.just_released(GamepadButton::South) {
            info!("{} just released South", entity);
        }

        let right_trigger = gamepad.get(GamepadButton::RightTrigger2).unwrap();
        if right_trigger.abs() > 0.01 {
            info!("{} RightTrigger2 value is {}", entity, right_trigger);
        }

        let left_stick_x = gamepad.get(GamepadAxis::LeftStickX).unwrap();
        if left_stick_x.abs() > 0.01 {
            info!("{} LeftStickX value is {}", entity, left_stick_x);
        }
    }
}
```

[message events]: https://docs.rs/bevy/latest/bevy/input/gamepad/index.html?search=gamepad%20event
[`gamepad_connection_system`]: https://docs.rs/bevy/latest/bevy/input/gamepad/fn.gamepad_connection_system.html
[`gamepad_event_processing_system`]: https://docs.rs/bevy/latest/bevy/input/gamepad/fn.gamepad_event_processing_system.html
[`GamepadSettings`]: https://docs.rs/bevy/latest/bevy/prelude/struct.GamepadSettings.html

+++
title = "Gamepad Devices"
insert_anchor_links = "right"
[extra]
weight = 5
+++

In Bevy, gamepads are devices which rely on buttons, joysticks, triggers, and directional pads (d-pads) to provide input data.
This includes everything from the controllers that come with gaming consoles to dance platforms in arcades and flight-sim and racing-sim accessories.
As long as a device's input data can be translated into one of the many [`GamepadButton`] or [`GamepadAxis`] variants, Bevy can use it as a gamepad.

[`GamepadButton`]: https://docs.rs/bevy/latest/bevy/prelude/enum.GamepadButton.html
[`GamepadAxis`]: https://docs.rs/bevy/latest/bevy/prelude/enum.GamepadAxis.html

## Gamepad Input
// Put In Section on Multiple Gamepads

Input data from gamepads is stored in a [`Gamepad`] struct, and is accessed through a [`Query`] rather than a resource.
We aren't able to use a resource since multiple gamepads can be connected at once (like for party games or for split-screen experiences), which negates the ability for a gamepad to be "unique".
However, accessing a gamepad through a `Query` can still give us a similar level of control as a [`ButtonInput`] resource can.
Methods like [`.just_pressed`] and [`.just_released`] provide the same information and context as their `ButtonInput` counterparts do.

```rust
fn gamepad_usage_system(gamepads: Query<(&Name, &Gamepad)>) {
    for (name, gamepad) in &gamepads {
        println!("{name}");

        if gamepad.just_pressed(GamepadButton::North) {
            println!("{} just pressed North", name)
        }
    }
}
```

Additionally, since gamepads can have multiple input forms (button presses, joystick directions, trigger pulls, etc.), we also have access to similar methods for the joysticks, d-pads, and triggers alongside buttons.
The `Gamepad` struct itself is really just a container for two input types: the [`GamepadButton`] enum and the [`GamepadAxis`] enum.
Both of these enums hold variants for all of the input data that Bevy can read from a gamepad.

[`Gamepad`]: https://docs.rs/bevy/latest/bevy/prelude/struct.Gamepad.html#method.product_id
[`Query`]: https://docs.rs/bevy/latest/bevy/ecs/prelude/struct.Query.html
[`ButtonInput`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html
[`.just_pressed`]: https://docs.rs/bevy/latest/bevy/input/gamepad/struct.Gamepad.html#method.just_pressed
[`.just_released`]: https://docs.rs/bevy/latest/bevy/input/gamepad/struct.Gamepad.html#method.just_released

### Gamepad Buttons

If you've read the [Using Input page], you'll know that Bevy classifies every "press-able" input type as being "button-like".
While this was brought up in the context of what input types a [`ButtonInput`] resource stores, we can use the same concept to figure out which gamepad input data is stored in `GamepadButton`.
As such, buttons, triggers, d-pads, and any other input that is "pressed" will be stored in `GamepadButton`.

```rust
fn gamepad_button_system(gamepads_input: Query<&Gamepad>) {
    for gamepad in &gamepads_input {
        // Jump when the bottom action button of a gamepad is pressed.
        // E.g. "Cross" on a PlayStation Controller, "A" on an Xbox Controller.
        if gamepad.just_pressed(GamepadButton::South) {
            println!("Jump");
        }
        // Unfocus the weapon when LeftTrigger is released. 
        if gamepad.just_released(GamepadButton::LeftTrigger) {
            println!("Weapon Unfocused");
        }
        // Activate a Minimap when Up/North on a D-Pad is pressed.
        if gamepad.pressed(Gamepad::DPadNorth) {
            println!("Minimap is Active");
        }
    }
}
```

We also have the ability to get the exact value amount that a `GamepadButton` is being pressed using the [`.get`] method.
The `.get` method can be handy for creating thresholds that need to be met before an action actually takes place.
Implementing a threshold can prevent unintentional actions from affecting the game and can also address faulty buttons on gamepads that might not
behave as they should.

```rust
fn gamepad_usage_system(gamepads_input: Query<&Gamepad>) {
    for gamepad in &gamepads_input {
        let right_trigger = gamepad.get(GamepadButton::RightTrigger).unwrap();
        if right_trigger.abs() > 0.01 {
            println!("RightTrigger is being pressed.");
        }
    }
}
```

[Using Input page]: /learn/book/handling-input/using-input

[`.get`]: https://docs.rs/bevy/latest/bevy/input/gamepad/struct.Gamepad.html#method.get

### Gamepad Joysticks

Similar to `GamepadButton`, [`GamepadAxis`] gives us information about the joystick input that a `Gamepad` is providing.
The data of this input type is mapped into a -1.0 to 1.0 range for two directions, and is stored as a `Vec2` type.
Accessing this data can be done through either the [`.left_stick`] or [`.right_stick`] methods:

```rust
fn gamepad_axis_system(gamepads_input: Query<&Gamepad>) {
    for gamepad in gamepads_input {
        if gamepad.left_stick() != Vec2{x: 0.0, y: 0.0} {
            println!("LeftStick is moving");
        }
        if gamepad.right_stick() != Vec2{x: 0.0, y: 0.0} {
            println!("LeftStick is moving");
        }
    }
}
```

Alternatively, we can use `.get` to read the data of a single `GamepadAxis` rather than both axes as a `Vec2`.

```rust
fn gamepad_axis_system(gamepads_input: Query<&Gamepad>) {
    for gamepad in &gamepads_input {
        let right_stick_x = gamepad.get(GamepadAxis::RightStickX).unwrap();
        
        if right_stick_x > 0.01 {
            println!("The RightStick is moving in the X axis.");
        }
    }
}
```

[`.left_stick`]: https://docs.rs/bevy/latest/bevy/input/gamepad/struct.Gamepad.html#method.left_stick
[`.right_stick`]: https://docs.rs/bevy/latest/bevy/input/gamepad/struct.Gamepad.html#method.right_stick

## Creating Custom Gamepad Settings

Sometimes you might find that Bevy's default way of handling gamepad input isn't a good fit for your own games.
Maybe you need to immediately tell when a button is being pressed, or you only want to read joystick data when the joystick is fully pressed in a direction.
Both of these scenarios (and many more) can be remedied by building your own [`GamepadSettings`] struct and connecting it with a `Gamepad`.

Every `Gamepad` has an associated `GamepadSettings` struct that is created when the `Gamepad` is connected.
Like the `Gamepad` struct, the `GamepadSettings` struct is really just a container for three values: [`ButtonSettings`], [`ButtonAxisSettings`], and [`AxisSettings`].
The first two values help define the thresholds that determine when a button is "pressed" and "released", while the last value

### Rumble/Haptic Feedback 

## Using Multiple Gamepads


When the `gamepad` feature is enabled in your project, Bevy will automatically setup a number of [message events] along with two systems: [`gamepad_connection_system`] and [`gamepad_event_processing_system`], both placed in `PreUpdate`.

You'll notice that we aren't setting up any resources to directly access.
This is because each Bevy uses various `Settings` structs within a larger [`GamepadSettings`] to determine similar conditions and thresholds to those in `ButtonInput`.
Since multiple gamepads can be connected at the same time, placed these settings within a `Resource` wouldn't make sense.
Instead, each gamepad can have it's own `GamepadSettings`, allowing you to adjust each gamepad based on any conditions or situations you might want.


[message events]: https://docs.rs/bevy/latest/bevy/input/gamepad/index.html?search=gamepad%20event
[`gamepad_connection_system`]: https://docs.rs/bevy/latest/bevy/input/gamepad/fn.gamepad_connection_system.html
[`gamepad_event_processing_system`]: https://docs.rs/bevy/latest/bevy/input/gamepad/fn.gamepad_event_processing_system.html
[`GamepadSettings`]: https://docs.rs/bevy/latest/bevy/prelude/struct.GamepadSettings.html

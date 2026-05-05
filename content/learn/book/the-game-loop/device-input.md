+++
title = "Device Input"
insert_anchor_links = "right"
[extra]
weight = 6
+++

Most developers want their game to be interactive.
If a player provides some kind of input, the game should react or change in some way.
To make this happen, we first need to receive the input data from the player: typically through an input device.

Additionally, we might also want to know the context that the input was created under.
Is a button being actively held down?
Was a button just released?
Is an input being repeatedly sent?
Fortunately, Bevy's [`bevy_input`] subcrate provides all of the tools to automatically setup, process, and make available any input data your game or library requires.

{% callout(type="warning") %}
## Creating Player Controls 

While `bevy_input` provides a plethora of handy tools for interacting with input data, it _is not recommended_ to directly use it for creating control schemes for your game.
These tools are made for parsing input data directly from devices through [Winit], an external crate used to capture input data.
They might suffice for simple games or for testing and debugging, however these tools can be cumbersome (if not overtly detrimental) if you do not plan for how they're used in your game.

Instead, it's more likely that you'll want to use an **input manager** for establishing the control schemes in your games.
An input manager allows you to assign actions to specific inputs without needing to directly handle the input data itself.
It provides a great deal of flexiblility when it comes to creating control schemes, such as allowing for different keybindings, accounting for different contexts, or integrating UI elements into a control scheme.

Bevy currently does not have a built-in solution for this, however we can point you towards several community crates that provide these features:

- [`bevy_enhanced_input`] aims to emulate the functionality of Unreal Engine's Enhanced Input system, utilizing observers to help setup bindings for different in-game actions.
- [`leafwing_input_manager`] is a straightforward but robust input-action manager that collects inputs from various sources and makes them conveniently accessible in your game logic.

Additionally, if you're finding that there isn't an existing solution that fits your project, you can also make your own input manager!
However that is well beyond the scope of what we'll cover here.
Instead, the rest of this page will cover how Bevy reads input data and makes it available for an input manager to use.

[Winit]: https://crates.io/crates/winit
[`bevy_enhanced_input`]: https://github.com/simgine/bevy_enhanced_input
[`leafwing_input_manager`]: https://github.com/Leafwing-Studios/leafwing-input-manager

{% end %}

[`bevy_input`]: https://docs.rs/bevy/latest/bevy/input/index.html

## Reading Input Events

Getting input data starts with the device itself, with an input event being generated and then sent to the computer which the device is connected to.
Bevy uses [Winit] (via [`bevy_winit`], Bevy's conversion layer for Winit) to read the event and process the input data.
The processed input data is then turned into a [`Message`] and made available to us for the first time.
Each type of input has a unique `Messages<M>` resource created and registered when the game is started, [`KeyboardInput`] for keyboard button presses, [`MouseMotion`] for mouse movements, etc.

Accessing input data through messages gives us all of the regular functionality that messages normally provide, including [`MessageReader`], [`MessageWriter`], and [`MessageMutator`].

```rust
// This system reads and prints out all `KeyboardInput` messages.
fn keyboard_message_reader(keyboard_inputs: MessageReader<KeyboardInput>) {
    for keyboard_input in keyboard_inputs.read() {
        info!("{keyboard_input:?}");
    }
}
// This system changes the right Ctrl, Alt, and Shift keys to their left versions.
fn keyboard_message_mutator(mut keyboard_inputs: MessageMutator<KeyboardInput>) {
    for message in keyboard_inputs.par_read() {
        if message.key_code == KeyCode::CtrlRight {
            message.key_code = KeyCode::CtrlLeft
        }
        if message.key_code == KeyCode::AltRight {
            message.key_code == KeyCode::AltLeft
        }
        if message.key_code == KeyCode::ShiftRight {
            message.key_code == KeyCode::ShiftLeft
        }
    }
}
```

Input messages are only sent when an input is initially activated (and can even be re-sent periodically for `KeyboardInput` messages if a keyboard key is being held down).
Since these messages aren't being received consistently, it's not recommended to use them for logic that needs to be continuously updated.
Instead, input messages are better suited for testing and logging input events, tracking text input, and activating systems or logic that don't rely on consistently repeated input.

[Winit]: https://crates.io/crates/winit
[`bevy_winit`]: https://docs.rs/bevy/latest/bevy/winit/index.html

[`Message`]: https://docs.rs/bevy/latest/bevy/ecs/message/trait.Message.html
[`KeyboardInput`]: https://docs.rs/bevy/latest/bevy/input/keyboard/struct.KeyboardInput.html
[`MouseMotion`]: https://docs.rs/bevy/latest/bevy/input/mouse/struct.MouseMotion.html
[`MessageReader`]: https://docs.rs/bevy/latest/bevy/prelude/struct.MessageReader.html
[`MessageWriter`]: https://docs.rs/bevy/latest/bevy/prelude/struct.MessageWriter.html
[`MessageMutator`]: https://docs.rs/bevy/latest/bevy/ecs/message/struct.MessageMutator.html

## Input System Parameters 

Input messages can be beneficial for some scenarios, however we have a more convenient way of accessing input data.
Instead of needing to read each message, Bevy allows us to access system parameters and a variety of methods to quickly see what input is being sent along with how the input is being sent.

Mouse and keyboard inputs are accessed through `Resources`, specifically a [`ButtonInput<KeyCode>`] or [`ButtonInput<Key>`] for keyboard button presses, and [`AccumulatedMouseMotion`], [`AccumulatedMouseScroll`], and [`ButtonInput<MouseButton>`] for mouse movement, scroll, and button presses respectively.

```rust
fn keyboard_mouse_system(
    keyboard_input: Res<ButtonInput<KeyCode>>,
    mouse_input: Res<AccumulatedMouseMotion>,
) {
    if keyboard_input.just_pressed(KeyCode::KeyA) {
        info!("'A' was just pressed");
    }
    if mouse_input.delta != Vec2::ZERO {
        let delta = mouse_motion.delta;
        info!("mouse moved ({}, {})", delta.x, delta.y);
    }
}
```

Meanwhile, gamepad input can be accessed through a `Query` that targets the [`Gamepad`] component.
We aren't able to use a `Resource` for gamepads as there could be multiple gamepads connected at once (like with party games or split-screen gamemodes).

Each gamepad is functionally identical in Bevy.
A `Gamepad` contains some metadata about the gamepad (like its name or device ID) and two components: [`GamepadButton`] and [`GamepadAxis`].
`GamepadButton` is an enum containing a list of buttons that Bevy recognizes for gamepads (like DPads, triggers, start buttons, etc.), while `GamepadAxis` is an enum with all potential joytstick inputs for a gamepad.

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

{% callout(type="info") %}

#### What Makes An Input "Button-like"?

You might have noticed that some input data is accessed through a `ButtonInput` struct (or a `GamepadButton` for a gamepad), while others aren't.
This may lead you to wonder what makes an input "button-like"?

It's actually fairly straightforward.
To be considered "button-like" in Bevy, the input has to be "press-able".
This means that Bevy can register the state of the input as either `pressed` or `released`.
Both of these values are explicitly stored in a [`ButtonState`] enum, which is recorded as a part of every "button-like" input event.

Something like a joystick can be pressed if the joystick can be "clicked".
If it can be clicked, then the joytstick button data will be accessible in a `ButtonInput` resource.
However, the direction you move the joystick in is not "press-able", and therefore is not "button-like" and will not be accessible in a `ButtonInput` resource.

[`ButtonState`]: https://docs.rs/bevy/latest/bevy/input/enum.ButtonState.html
{% end %}

[`ButtonInput<KeyCode>`]: https://docs.rs/bevy/latest/bevy/input/keyboard/enum.KeyCode.html
[`ButtonInput<Key>`]: https://docs.rs/bevy/latest/bevy/input/keyboard/enum.Key.html
[`AccumulatedMouseMotion`]: https://docs.rs/bevy/latest/bevy/input/mouse/struct.AccumulatedMouseMotion.html
[`AccumulatedMouseScroll`]: https://docs.rs/bevy/latest/bevy/input/mouse/struct.AccumulatedMouseScroll.html
[`ButtonInput<MouseButton>`]: https://docs.rs/bevy/latest/bevy/input/mouse/enum.MouseButton.html
[`Gamepad`]: https://docs.rs/bevy/latest/bevy/input/gamepad/struct.Gamepad.html
[`GamepadButton`]: https://docs.rs/bevy/latest/bevy/prelude/enum.GamepadButton.html
[`GamepadAxis`]: https://docs.rs/bevy/latest/bevy/prelude/enum.GamepadAxis.html

### Pressed Versus Just Pressed

Each [`ButtonInput`] resource and [`Gamepad`] component provide us with methods that let us access information about the state of a button.
For example, there are different methods which will return a `bool` based on if the button has just been pressed ([`just_pressed`]), is currently being pressed ([`pressed`]), or if its just been released ([`just_released`]).

Although it might appear like the difference between [`pressed`] and [`just_pressed`] is negligible, the two are quite distinct.
While both signal a `ButtonInput` button being activated, `pressed` is continuously `true` until the input is released.
`just_pressed` will only be `true` for _a single frame_ after the input is activated.
The same is true for [`just_released`], which will only be `true` for a single frame after the input is deactivated.

```rust
fn keyboard_input_system(
    keyboard_input: Res<ButtonInput<KeyCode>>,
) {
    // Returns `true` while A is currently being pressed.
    if keyboard_input.pressed(KeyCode::KeyA) {
        info!("'A' is currently being pressed");
    }
    // Returns `true` for one frame after A is pressed.
    if keyboard_input.just_pressed(KeyCode::KeyA) {
        info!("'A' was just pressed");
    }
    // Returns `true` for one frame after A is released.
    if keyboard_input.just_released(KeyCode::KeyA) {
        info!("'A' was just released");
    }
}
```

[`ButtonInput`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html
[`pressed`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.pressed
[`just_pressed`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.just_pressed
[`just_released`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.just_released

### Button Combinations

Button combinations can be made by accessing multiple buttons within a `ButtonInput` or `Gamepad`.
Using the [`any_pressed`], [`any_just_pressed`], or [`any_just_released`] methods allow us to establish AND logic for handling combinations.
Alternatively, we could use the [`all_pressed`], [`all_just_pressed`], and [`all_just_released`] methods and supply a list of button inputs instead.

```rust
// This system prints when `Ctrl + Shift + A` is pressed.
fn and_keyboard_combo(input: Res<ButtonInput<KeyCode>>) {
    let shift = input.any_pressed([KeyCode::ShiftLeft, KeyCode::ShiftRight]);
    let ctrl = input.any_pressed([KeyCode::ControlLeft, KeyCode::ControlRight]);

    if ctrl && shift && input.just_pressed(KeyCode::KeyA) {
        info!("Just pressed Ctrl + Shift + A!");
    }
}
// This system requires that `AltLeft` and `F4` are pressed.
fn strict_keyboard_combo(input: Res<ButtonInput<KeyCode>>) {
    if input.all_just_pressed([KeyCode::AltLeft, KeyCode::F4]) {
        info!("The game will shut down now!");
    }
}
```

We can also access a collection of every button that is being interacted with on a given frame.
Using [`get_pressed`], [`get_just_pressed`], or [`get_just_released`] will return an [`Iterator`] (specifically an [`ExactSizeIterator`]) of all currently pressed, just pressed, or just released buttons.
Any of these options can be preferable when processing a lot of input data at once since you can iterate over all relevant buttons.
As an example, if you want the `Escape` key to always be evaluated first (but not evaluated separately), we can use `get_pressed` to see if its being pressed.

```rust
fn get_all_pressed_buttons(input: Res<ButtonInput<KeyCode>>) {
    if input.get_pressed().iter().any(|key| key == KeyCode::Esc) {
        // Pause the game, open the menu.
    }
}
```

[`any_pressed`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.any_pressed
[`any_just_pressed`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.any_just_pressed
[`any_just_released`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.any_just_released
[`all_pressed`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.all_pressed
[`all_just_pressed`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.all_just_pressed
[`all_just_released`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.all_just_released
[`get_pressed`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.get_pressed
[`get_just_pressed`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.get_just_pressed
[`get_just_released`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.get_just_released
[`Iterator`]: https://doc.rust-lang.org/nightly/core/iter/trait.Iterator.html
[`ExactSizeIterator`]: https://doc.rust-lang.org/nightly/core/iter/trait.ExactSizeIterator.html

### Resetting Button Inputs

Since `ButtonInput` is accessed as a `Resource`, we also have the ability to alter its data through a `ResMut` system parameter.
This can be especially helpful if we want to clear all input from a specific key or button, or even reset all input from the entire device.
The [`clear`], [`clear_just_pressed`], and [`clear_just_released`] methods allow us to remove the current state of a button input.
For example, if we use `clear_just_pressed` on a button input, we won't receive a `true` value from calling `just_pressed` on that button input until a new button press occurs.

```rust
// Clear the `just_pressed` state of Left MouseButton.
fn clear_a_mouse_click(mut mouse_clicks: ResMut<ButtonInput<MouseButton>>) {
    if mouse_clicks.just_pressed(MouseButton::Left) {
        mouse_click.clear_just_pressed(MouseButton::Left);
    }
}
```

If we want to go one step further, we also have the [`reset`] and [`reset_all`] methods which will completely reset the state of either a single button or all buttons.

```rust
// Reset all MouseButton states.
fn clear_all_mouse_clicks(mut mouse_clicks: ResMut<ButtonInput<MouseButton>>) {
    if mouse_clicks.get_pressed().len() != 0 {
        mouse_clicks.reset_all();
    }
}
```

Finally, we also have the [`release`] and [`release_all`] methods which will register a release event.
`release` allows you to generate a release event for a specific button, while `release_all` will create release events for every button on the device.
These methods can be helpful in situations where the player continuously holding a button press would cause issues for your game.

```rust
// We only want this event to be triggered once.
#[deriver(Event)]
struct SingleTriggerEvent;

fn only_trigger_once(mut input: ResMut<ButtonInput<MouseButton>>, mut commands: Commands) {
    if input.just_pressed(MouseButton::Left) {
        // As soon as Left MouseButton is pressed, release it.
        input.release(MouseButton::Left);
        commands.trigger(SingleTriggerEvent);
    }
}
```

[`clear`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.clear
[`clear_just_pressed`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.clear_just_pressed
[`clear_just_released`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.clear_just_released
[`reset`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.reset
[`reset_all`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.reset_all

[`release`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.release
[`release_all`]: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html#method.release_all

### Axis Inputs

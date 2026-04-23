+++
title = "Keyboard Input"
insert_anchor_links = "right"
[extra]
weight = 3
+++

Using input from a keyboard is relatively straightforward.
Each key is read as input through a [`KeyboardInput`] message, which is then placed into the [`ButtonInput`] resource during the `PreUpdate` schedule.
This resource allows you access different conditions, thresholds, and key combinations that you can use to trigger some functionality in response.

When the `keyboard` feature is enabled in your game, Bevy will automatically:

- Add the [`KeyboardInput`] and [`KeyboardFocusLost`] messages to the `World`.
  - `KeyboardInput` records the actual keyboard input data, while `KeyboardFocusLost` indicates if a keyboard is sending input to the application.
- Initialize the `ButtonInput<Key>` and `ButtonInput<KeyCode>` resources.
  - This gives us access to the specific [`KeyCode`] buttons and generic [`Key`] value to read from.
  - This resource is updated with the data that is captured in the `KeyboardInput` message.
- Insert the [`keyboard_input_system`] into the [`PreUpdate`] schedule.
  - This system updates both `ButtonInput` resources with the input received in the `KeyboardInput` message.

```rust
fn keyboard_messages(
    keyboard_messages: MessageReader<KeyboardInput>,
) {
    // Read the KeyboardInput message events.
    for key_press in keyboard_messages.read() {
        if key_press.key == Key::F11 {
            // Take a screenshot.
        }
    }
}
fn keyboard_connection(
    keyboard_connection: MessageReader<KeyboardFocusLost>,
) {
    // See if the keyboard is still providing input to the game.
    if keyboard_connection.len() > 0 {
        println!("No longer receiving keyboard input!");
    }
}
fn keyboard_access(
    keyboard_resource: Res<ButtonInput<KeyCode>>,
) {
    // Check if Alt + F4 is being pressed.
    if keyboard_resource.all_pressed([KeyCode::AltLeft, KeyCode::F4]) {
        println!("The Game Will Shutdown!");
    }
}
```

[`KeyboardInput`]: https://docs.rs/bevy/latest/bevy/input/keyboard/struct.KeyboardInput.html
[`KeyboardFocusLost`]: https://docs.rs/bevy/latest/bevy/input/keyboard/struct.KeyboardFocusLost.html
[`KeyCode`]: https://docs.rs/bevy/latest/bevy/input/keyboard/enum.KeyCode.html
[`Key`]: https://docs.rs/bevy/latest/bevy/input/keyboard/enum.Key.html
[`keyboard_input_system`]: https://docs.rs/bevy/latest/bevy/input/keyboard/fn.keyboard_input_system.html
[`PreUpdate`]: https://docs.rs/bevy/latest/bevy/app/struct.PreUpdate.html

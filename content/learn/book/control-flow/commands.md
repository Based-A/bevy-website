+++
title = "Commands"
insert_anchor_links = "right"
[extra]
weight = 5
status = 'hidden'
+++

**Commands** represent instructions for manipulations to perform on the world, the next time that we have access to all of it once.

Many operations in the ECS can only be done via exclusive world access, such as:

- Spawning and Despawning Entities

```rust
// Spawn a new Entity with `bundle` Components.
commands.spawn(bundle);

// Despawn the `entity` Entity.
commands.entity(entity).despawn(); 
```

- Adding and Removing Components

```rust
// Insert `bundle` Components into `entity` Entity.
commands.entity(entity).insert(bundle);

// Remove `bundle` Components from `entity` Entity.
commands.entity(entity).remove(bundle); 
```

- Inserting and Removing Resources

```rust
// Add `resource` to the World.
commands.init_resource::<resource>();

// Remove `resource` from the World.
commands.remove_resource::<resource>(); 
```

- Running One-Shot Systems and Schedules

```rust
// Run a System matching the ID in `system_id`.
commands.run_system(system_id); 

// Run all Systems that are in the `schedule` Schedule.
commands.run_schedule(schedule);
```

- Triggering Observers

```rust
// Trigger an `event` Event for all Observers watching `event`.
commands.trigger(event); 
```

Commands allow users to queue up these changes as part of systems, deferring the work until later to avoid disrupting both system parallelism and data-oriented access of components.

When applying changes to a single entity, the [`Commands`] type is transformed into [`EntityCommands`] via [`Commands::entity`]. While the [`Command`] trait can have arbitrary effect on the [`World`], the [`EntityCommand`] trait is designed to modify a single entity.

```rust
// Marker Component for a Player.
#[derive(Component)]
struct Player;

// Health Component for a Player.
#[derive(Component)]
struct Health(pub usize);

// Create a new Entity with the Player Component and save the Entity to the variable `player`.
let player = commands.spawn((Player));

// This accesses the EntityCommands for the Entity in `player`.
let player_commands = commands.entity(player);

// This adds a Health Component to the `player` Entity if it doesn't already exist.
player_commands.insert_if_new((Health(10)));
```

If you want to send commands from within a parallel context (such as via [`Query::par_iter_mut`]), [`ParallelCommands`] can be used.

```rust
// This System runs across multiple threads, and ensures that each Query entry is only iterated over once.
fn parallel_command_system(
    mut query: Query<(Entity, &Velocity)>,
    par_commands: ParallelCommands
) {
    query.par_iter_mut().for_each(|(entity, velocity)| {
        if velocity.magnitude() > 10.0 {
            // Emergency Stop!
            velocity = 0.0;
            println!("{} stopped very quickly!", entity);
        }
    });
}
```

Even more broadly, custom [`Commands`]-like [`SystemParam`] can be constructed with the use of the generic [`Deferred`] system parameter.

[`Commands`]: https://docs.rs/bevy/latest/bevy/ecs/prelude/struct.Commands.html
[`EntityCommands`]: https://docs.rs/bevy/latest/bevy/prelude/struct.EntityCommands.html
[`Commands::entity`]: https://docs.rs/bevy/latest/bevy/ecs/prelude/struct.Commands.html#method.entity
[`Command`]: https://docs.rs/bevy/latest/bevy/prelude/trait.Command.html
[`World`]: https://docs.rs/bevy/latest/bevy/prelude/struct.World.html
[`EntityCommand`]: https://docs.rs/bevy/latest/bevy/prelude/trait.EntityCommand.html
[`Query::par_iter_mut`]: https://docs.rs/bevy/latest/bevy/ecs/prelude/struct.Query.html#method.par_iter_mut
[`ParallelCommands`]: https://docs.rs/bevy/latest/bevy/prelude/struct.ParallelCommands.html
[`SystemParam`]: https://docs.rs/bevy/latest/bevy/ecs/system/trait.SystemParam.html
[`Deferred`]: https://docs.rs/bevy/latest/bevy/prelude/struct.Deferred.html

## When Do Commands Take Effect?

Commands are applied whenever a [`Schedule`] is completed. Ordinarily, this will occur multiple times during and after each frame. As a result, systems will always see the effects of commands queued by systems in other schedules.

```rust
// An Event we want to trigger.
#[derive(Event)]
struct UpdateEvent;

// Add our Systems to their Schedules.
app.add_systems(Startup, insert_observer_system);
app.add_system(Update, update_system);

// This System will run in the `Startup` schedule, meaning that when our application launches this Observer is added.
fn insert_observer_system(mut commands: Commands) {
    commands.add_observer(|update: On<UpdateEvent>| {
        println!("This runs every Update!");
    });
}

// Meanwhile this System runs in the `Update` schedule, so once our application is loaded 
// and begins to run, this System will run everytime the `Update` Schedule runs.
fn update_system(mut commands: Commands) {
    commands.trigger(UpdateEvent);
}
```

In addition, if a system with [`Commands`] is ordered before another system, that system will always see the effects of the commands in the first system. Bevy ensures this occurs by dynamically inserting synchronization points, during which all commands are applied. Each system can hold their own copy of [`Commands`] in their local system state. When commands are applied, these queues are evaluated as in the same order that the systems were run. Within each system, the commands are applied in a first-in-first-out order.

```rust
// Add our Systems in the same Schedule, but placed in a specific order.
app.add_systems(
    Update, (
        add_the_component, 
        remove_the_component.after(add_the_component)
        // `after` specifies that the `remove_the_component` System 
        // will only run after `add_the_component` had completed.
    )
);

// This System will add the TargetComponent to the `player` Entity.
fn add_the_component(mut commands: Commands, mut player: Single<Entity, Without<TargetComponent>) {
    commands.entity(player).insert_if_new((TargetComponent));
    println!("TargetComponent added!");
}

// This System will remove the TargetComponent from the `player` Entity.
fn remove_the_component(mut commands: Commands, mut player: Single<Entity, With<TargetComponent>) {
    commands.entity(player).remove((TargetComponent));
    println!("TargetComponent removed!");
}

```

[`Schedule`]: https://docs.rs/bevy/latest/bevy/prelude/struct.Schedule.html

## Custom Commands

Because of their flexible nature, custom commands are a powerful tool for implementing game-specific operations. While they may not be as fast or transparent as working with events or observers, the arbitrary flexibility can be great for quickly evolving game logic and performing operations atomically.

Writing custom commands is quite simple: create a struct and implement [`Command`] for it. If you want to pass in data, add fields to your struct. To send a custom command, simply call `commands.queue(CustomCommandStruct { my_data })`.

You can make this pattern even more ergonomic by writing an [extension trait] for the [`Commands`] type, allowing you to call new methods as long as the extension trait is imported. `commands.custom_command(my_data)` is shorter and plays nicer with auto-complete, this approach has no functional benefit or cost: it's simply a matter of style.

One-off commands can also be sent, due to the [blanket implementation of `Command` for all functions with a `&mut World`] argument. This is convenient, but leads to more duplicated code and can be less clear.

These same strategies can be applied for the [`EntityCommand`] trait and the [`EntityCommands`] struct.

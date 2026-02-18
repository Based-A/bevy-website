+++
title = "Commands"
insert_anchor_links = "right"
[extra]
weight = 5
status = 'hidden'
+++

Commands represent instructions for manipulations to perform on the world, the next time that we have access to all of it once.

Many operations in the ECS can only be done via exclusive world access, such as:

- Spawning and despawning entities
- Adding and removing components
- Inserting and removing resources
- Running one-shot systems and schedules
- Triggering observers

Commands allow users to queue up these changes as part of systems, deferring the work until later to avoid disrupting both system parallelism and data-oriented access of components.

When applying changes to a single entity, the [`Commands`] type is transformed into [`EntityCommands`] via [`Commands::entity`]. While the [`Command`] trait can have arbitrary effect on the [`World`], the [`EntityCommand`] trait is designed to modify a single entity.

If you want to send commands from within a parallel context (such as via [`Query::par_iter_mut`]), [`ParallelCommands`] can be used.

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

Commands are applied whenever a [schedule] is completed. Ordinarily, this will occur multiple times during and after each frame. As a result, systems will always see the effects of commands queued by systems in other schedules.

In addition, if a system with [`Commands`] is [ordered] before another system, that system will always see the effects of the commands in the first system. Bevy ensures this occurs by dynamically inserting synchronization points, during which all commands are applied.

## What Order Do Commands Get Applied In?

Each system can hold their own copy of [`Commands`] in their local system state. When commands are applied, these queues are evaluated as in the same order that the systems were run. Within each system, the commands are applied in a first-in-first-out order.

## Custom Commands

Because of their flexible nature, custom commands are a powerful tool for implementing game-specific operations. While they may not be as fast or transparent as working with events or observers, the arbitrary flexibility can be great for quickly evolving game logic and performing operations atomically.

Writing custom commands is quite simple: create a struct and implement [`Command`] for it. If you want to pass in data, add fields to your struct. To send a custom command, simply call `commands.queue(CustomCommandStruct { my_data })`.

You can make this pattern even more ergonomic by writing an [extension trait] for the [`Commands`] type, allowing you to call new methods as long as the extension trait is imported. `commands.custom_command(my_data)` is shorter and plays nicer with auto-complete, this approach has no functional benefit or cost: it's simply a matter of style.

One-off commands can also be sent, due to the [blanket implementation of `Command` for all functions with a `&mut World`] argument. This is convenient, but leads to more duplicated code and can be less clear.

These same strategies can be applied for the [`EntityCommand`] trait and the [`EntityCommands`] struct.

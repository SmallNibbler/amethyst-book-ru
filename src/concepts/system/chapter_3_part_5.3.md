# Реализация типажа SystemDesc

Если производная `SystemDesc` не может сгенерировать реализацию типажа `SystemDesc` для инициализации системы, типаж `SystemDesc` можно реализовать вручную:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
#
use amethyst::{
    audio::output::Output,
    core::SystemDesc,
    ecs::{System, SystemData, World},
};

# /// Syncs 3D transform data with the audio engine to provide 3D audio.
# #[derive(Debug, Default)]
# pub struct AudioSystem(Output);
# impl<'a> System<'a> for AudioSystem {
#     type SystemData = ();
#     fn run(&mut self, _: Self::SystemData) {}
# }
#
/// Собирает `AudioSystem`.
#[derive(Default, Debug)]
pub struct AudioSystemDesc {
    /// Аудио `Output`.
    pub output: Output,
}

impl<'a, 'b> SystemDesc<'a, 'b, AudioSystem> for AudioSystemDesc {
    fn build(self, world: &mut World) -> AudioSystem {
        <AudioSystem as System<'_>>::SystemData::setup(world);

        world.insert(self.output.clone());

        AudioSystem(self.output)
    }
}

// in `main.rs`:
// let game_data = GameDataBuilder::default()
//     .with_system_desc(AudioSystemDesc::default(), "", &[]);
```

## Шаблоны

```rust,ignore
use amethyst_core::SystemDesc;

/// Собирает `SystemName`.
#[derive(Default, Debug)]
pub struct SystemNameDesc;

impl<'a, 'b> SystemDesc<'a, 'b, SystemName> for SystemNameDesc {
    fn build(self, world: &mut World) -> SystemName {
        <SystemName as System<'_>>::SystemData::setup(world);

        let arg = unimplemented!("Replace code here");

        SystemName::new(arg)
    }
}
```

С параметрами типа:

```rust,ignore
use std::marker::PhantomData;

use derivative::Derivative;

use amethyst_core::ecs::SystemData;
use amethyst_core::SystemDesc;

/// Собирает `SystemName`.
#[derive(Derivative, Debug)]
#[derivative(Default(bound = ""))]
pub struct SystemNameDesc<T> {
    marker: PhantomData<T>,
}

impl<'a, 'b, T> SystemDesc<'a, 'b, SystemName<T>>
    for SystemNameDesc<T>
where
    T: unimplemented!("Replace me."),
{
    fn build(self, world: &mut World) -> SystemName<T> {
        <SystemName<T> as System<'_>>::SystemData::setup(world);

        let arg = unimplemented!("Replace code here");

        SystemName::new(arg)
    }
}
```

---

Документация была переведена специально для сообщества "**rust_lang_ru**".

Подпишись на нас в **[Telegram][telegram]** и **[YouTube][youtube]**!

[telegram]: http://tlinks.run/rust_lang_ru
[youtube]: https://www.youtube.com/channel/UCu413rnSfuSSOR3OsIThlZA

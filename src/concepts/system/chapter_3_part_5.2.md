# Производная SystemDesc

Производная `SystemDesc` поддерживает следующие случаи при создании реализации типажа `SystemDesc`:

* Параметры для передачи в системный конструктор.
* Поля для пропуска по умолчанию используемые системным конструктором.
* Регистрация `ReaderId` для `EventChannel<\_>` в `World`.
* Регистрация `ReaderId` в `FlaggedStorage` компонента.
* Вставка ресурса в `World`.

Если ваш сценарий использования для инициализации системы не описан, см. страницу [«Реализация типажа `SystemDesc`»][impl-systemdesc-trait].

[impl-systemdesc-trait]: ./chapter_3_part_5.3.html

## Передача параметров в системный конструктор

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
#
# use amethyst::{
#     derive::SystemDesc,
#     ecs::{System, SystemData},
# };
#
#[derive(SystemDesc)]
#[system_desc(name(SystemNameDesc))]
pub struct SystemName {
    field_0: u32,
    field_1: String,
}

impl SystemName {
    fn new(field_0: u32, field_1: String) -> Self {
        SystemName { field_0, field_1 }
    }
}
#
# impl<'a> System<'a> for SystemName {
#     type SystemData = ();
#     fn run(&mut self, data: Self::SystemData) {}
# }
```

<details>
<summary>Generated code</summary>

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
#
# use amethyst::{
#     derive::SystemDesc,
#     ecs::{System, SystemData},
# };
#
# pub struct SystemName {
#     field_0: u32,
#     field_1: String,
# }
#
# impl SystemName {
#     fn new(field_0: u32, field_1: String) -> Self {
#         SystemName { field_0, field_1 }
#     }
# }
#
# impl<'a> System<'a> for SystemName {
#     type SystemData = ();
#     fn run(&mut self, data: Self::SystemData) {}
# }
#
/// Собирает `SystemName`.
#[derive(Default, Debug)]
pub struct SystemNameDesc {
    field_0: u32,
    field_1: String,
}

impl SystemNameDesc {
    fn new(field_0: u32, field_1: String) -> Self {
        SystemNameDesc { field_0, field_1 }
    }
}

impl<'a, 'b> ::amethyst::core::SystemDesc<'a, 'b, SystemName> for SystemNameDesc {
    fn build(self, world: &mut ::amethyst::ecs::World) -> SystemName {
        <SystemName as ::amethyst::ecs::System<'_>>::SystemData::setup(world);

        SystemName::new(self.field_0, self.field_1)
    }
}
```

</details>

## Поля для пропуска по умолчанию  используемые конструктором системы

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
#
# use amethyst::{
#     derive::SystemDesc,
#     ecs::{System, SystemData},
# };
#
#[derive(SystemDesc)]
#[system_desc(name(SystemNameDesc))]
pub struct SystemName {
    #[system_desc(skip)]
    field_0: u32,
    field_1: String,
}

impl SystemName {
    fn new(field_1: String) -> Self {
        SystemName { field_0: 123, field_1 }
    }
}
#
# impl<'a> System<'a> for SystemName {
#     type SystemData = ();
#     fn run(&mut self, data: Self::SystemData) {}
# }
```

<details>
<summary>Generated code</summary>

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
#
# use amethyst::{
#     derive::SystemDesc,
#     ecs::{System, SystemData},
# };
#
# pub struct SystemName {
#     field_0: u32,
#     field_1: String,
# }
#
# impl SystemName {
#     fn new(field_1: String) -> Self {
#         SystemName { field_0: 123, field_1 }
#     }
# }
#
# impl<'a> System<'a> for SystemName {
#     type SystemData = ();
#     fn run(&mut self, data: Self::SystemData) {}
# }
#
/// Собирает `SystemName`.
#[derive(Default, Debug)]
pub struct SystemNameDesc {
    field_1: String,
}

impl SystemNameDesc {
    fn new(field_1: String) -> Self {
        SystemNameDesc { field_1 }
    }
}

impl<'a, 'b> ::amethyst::core::SystemDesc<'a, 'b, SystemName> for SystemNameDesc {
    fn build(self, world: &mut ::amethyst::ecs::World) -> SystemName {
        <SystemName as ::amethyst::ecs::System<'_>>::SystemData::setup(world);

        SystemName::new(self.field_1)
    }
}
```

</details>

**Примечание.** Если параметры поля отсутствуют, реализация `SystemDesc` вызовет `SystemName::default()`:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
#
# use amethyst::{
#     derive::SystemDesc,
#     ecs::{System, SystemData},
# };
#
#[derive(Default, SystemDesc)]
#[system_desc(name(SystemNameDesc))]
pub struct SystemName {
    #[system_desc(skip)]
    field_0: u32,
}
#
# impl<'a> System<'a> for SystemName {
#     type SystemData = ();
#     fn run(&mut self, data: Self::SystemData) {}
# }
```

<details>
<summary>Generated code</summary>

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
#
# use amethyst::{
#     derive::SystemDesc,
#     ecs::{System, SystemData},
# };
#
# #[derive(Default)]
# pub struct SystemName {
#     field_0: u32,
# }
#
# impl<'a> System<'a> for SystemName {
#     type SystemData = ();
#     fn run(&mut self, data: Self::SystemData) {}
# }
#
/// Собирает `SystemName`.
#[derive(Debug)]
pub struct SystemNameDesc {}

impl Default for SystemNameDesc {
    fn default() -> Self {
        SystemNameDesc {}
    }
}

impl<'a, 'b> ::amethyst::core::SystemDesc<'a, 'b, SystemName> for SystemNameDesc {
    fn build(self, world: &mut ::amethyst::ecs::World) -> SystemName {
        <SystemName as ::amethyst::ecs::System<'_>>::SystemData::setup(world);

        SystemName::default()
    }
}
```

</details>

## Регистрация `ReaderId` для `EventChannel<\_>` в `World`.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
#
# use amethyst::{
#     derive::SystemDesc,
#     ecs::{System, SystemData},
#     shrev::{EventChannel, ReaderId},
#     ui::UiEvent,
# };
#
#[derive(SystemDesc)]
#[system_desc(name(SystemNameDesc))]
pub struct SystemName {
    #[system_desc(event_channel_reader)]
    reader_id: ReaderId<UiEvent>,
}

impl SystemName {
    fn new(reader_id: ReaderId<UiEvent>) -> Self {
        SystemName { reader_id }
    }
}
#
# impl<'a> System<'a> for SystemName {
#     type SystemData = ();
#     fn run(&mut self, data: Self::SystemData) {}
# }
```

<details>
<summary>Generated code</summary>

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
#
# use amethyst::{
#     derive::SystemDesc,
#     ecs::{System, SystemData},
#     shrev::{EventChannel, ReaderId},
#     ui::UiEvent,
# };
#
# pub struct SystemName {
#     reader_id: ReaderId<UiEvent>,
# }
#
# impl SystemName {
#     fn new(reader_id: ReaderId<UiEvent>) -> Self {
#         SystemName { reader_id }
#     }
# }
#
# impl<'a> System<'a> for SystemName {
#     type SystemData = ();
#     fn run(&mut self, data: Self::SystemData) {}
# }
#
/// Собирает `SystemName`.
#[derive(Debug)]
pub struct SystemNameDesc;

impl Default for SystemNameDesc {
    fn default() -> Self {
        SystemNameDesc {}
    }
}

impl<'a, 'b> ::amethyst::core::SystemDesc<'a, 'b, SystemName> for SystemNameDesc {
    fn build(self, world: &mut ::amethyst::ecs::World) -> SystemName {
        <SystemName as ::amethyst::ecs::System<'_>>::SystemData::setup(world);

        let reader_id = world
            .fetch_mut::<EventChannel<UiEvent>>()
            .register_reader();

        SystemName::new(reader_id)
    }
}
```

</details>

## Регистрация `ReaderId` в `FlaggedStorage` компонента.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
#
# use amethyst::{
#     derive::SystemDesc,
#     ecs::{storage::ComponentEvent, System, SystemData, WriteStorage},
#     shrev::{EventChannel, ReaderId},
#     ui::UiResize,
# };
#
#[derive(SystemDesc)]
#[system_desc(name(SystemNameDesc))]
pub struct SystemName {
    #[system_desc(flagged_storage_reader(UiResize))]
    resize_events_id: ReaderId<ComponentEvent>,
}

impl SystemName {
    fn new(resize_events_id: ReaderId<ComponentEvent>) -> Self {
        SystemName { resize_events_id }
    }
}
#
# impl<'a> System<'a> for SystemName {
#     type SystemData = ();
#     fn run(&mut self, data: Self::SystemData) {}
# }
```

<details>
<summary>Generated code</summary>

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
#
# use amethyst::{
#     derive::SystemDesc,
#     ecs::{storage::ComponentEvent, System, SystemData, WriteStorage},
#     shrev::{EventChannel, ReaderId},
#     ui::UiResize,
# };
#
# pub struct SystemName {
#     resize_events_id: ReaderId<ComponentEvent>,
# }
#
# impl SystemName {
#     fn new(resize_events_id: ReaderId<ComponentEvent>) -> Self {
#         SystemName { resize_events_id }
#     }
# }
#
# impl<'a> System<'a> for SystemName {
#     type SystemData = ();
#     fn run(&mut self, data: Self::SystemData) {}
# }
#
/// Собирает `SystemName`.
#[derive(Debug)]
pub struct SystemNameDesc;

impl Default for SystemNameDesc {
    fn default() -> Self {
        SystemNameDesc {}
    }
}

impl<'a, 'b> ::amethyst::core::SystemDesc<'a, 'b, SystemName> for SystemNameDesc {
    fn build(self, world: &mut ::amethyst::ecs::World) -> SystemName {
        <SystemName as ::amethyst::ecs::System<'_>>::SystemData::setup(world);

        let resize_events_id = WriteStorage::<UiResize>::fetch(&world)
                            .register_reader();

        SystemName::new(resize_events_id)
    }
}
```

</details>

## Вставка ресурса в `World`.

**Примечание.** Если ресурс, который вы хотите вставить, является результатом выражения, такого как вызов функции, вы должны заключить это выражение в кавычки, например, `#[system_desc(insert("MyResource::default()"))]`.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
#
# use amethyst::{
#     derive::SystemDesc,
#     ecs::{ReadExpect, System, SystemData},
# };
#
pub struct NonDefault;

#[derive(Default, SystemDesc)]
#[system_desc(insert(NonDefault))]
pub struct SystemName;

impl<'a> System<'a> for SystemName {
    type SystemData = ReadExpect<'a, NonDefault>;
    fn run(&mut self, data: Self::SystemData) {}
}
```

<details>
<summary>Generated code</summary>

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
#
# use amethyst::{
#     derive::SystemDesc,
#     ecs::{ReadExpect, System, SystemData},
# };
#
# pub struct NonDefault;
#
# #[derive(Default)]
# pub struct SystemName;
#
# impl<'a> System<'a> for SystemName {
#     type SystemData = ReadExpect<'a, NonDefault>;
#     fn run(&mut self, data: Self::SystemData) {}
# }
#
/// Собирает `SystemName`.
#[derive(Debug)]
pub struct SystemNameDesc;

impl Default for SystemNameDesc {
    fn default() -> Self {
        SystemNameDesc {}
    }
}

impl<'a, 'b> ::amethyst::core::SystemDesc<'a, 'b, SystemName> for SystemNameDesc {
    fn build(self, world: &mut ::amethyst::ecs::World) -> SystemName {
        <SystemName as ::amethyst::ecs::System<'_>>::SystemData::setup(world);

        world.insert(NonDefault);

        SystemName::default()
    }
}
```

---

Документация была переведена специально для сообщества "**rust_lang_ru**".

Подпишись на нас в **[Telegram][telegram]** и **[YouTube][youtube]**!

[telegram]: http://tlinks.run/rust_lang_ru
[youtube]: https://www.youtube.com/channel/UCu413rnSfuSSOR3OsIThlZA

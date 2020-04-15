# Канал событий

Что такое канал событий?

`EventChannel` - это широковещательная очередь событий. События могут быть любого типа, который реализует `Send` + `Sync` + `'static`.

Как правило, `EventChannels` вставляются как ресурсы в `World`.

## Примеры

### Создание канала событий

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::shrev::EventChannel;
// В следующих примерах `MyEvent` - это тип события канала.
#[derive(Debug)]
pub enum MyEvent {
    A,
    B,
}

let mut channel = EventChannel::<MyEvent>::new();
```

Запись событий в канал событий

Запись одного события:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# #[derive(Debug)]
# pub enum MyEvent {
#   A,
#   B,
# }
# fn main() {
#   let mut channel = amethyst::shrev::EventChannel::<MyEvent>::new();
    channel.single_write(MyEvent::A);
# }
```

Запись нескольких событий:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# #[derive(Debug)]
# pub enum MyEvent {
#   A,
#   B,
# }
# fn main() {
#   let mut channel = amethyst::shrev::EventChannel::<MyEvent>::new();
    channel.iter_write(vec![MyEvent::A, MyEvent::A, MyEvent::B].into_iter());
# }
```

### Чтение событий

`EventChannels` гарантируют отправку событий каждому читателю. Чтобы подписаться на события, зарегистрируйте читателя в `EventChannel`, чтобы получить `ReaderId`:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# #[derive(Debug)]
# pub enum MyEvent {
#   A,
#   B,
# }
# fn main() {
#   let mut channel = amethyst::shrev::EventChannel::<MyEvent>::new();
let mut reader_id = channel.register_reader();
# }
```

При чтении событий передайте `ReaderId` в:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# #[derive(Debug)]
# pub enum MyEvent {
#   A,
#   B,
# }
# fn main() {
#   let mut channel = amethyst::shrev::EventChannel::<MyEvent>::new();
#   let mut reader_id = channel.register_reader();
for event in channel.read(&mut reader_id) {
    // Тип события выводится из универсального типа,
    // который мы назначили для `EventChannel <MyEvent>` ранее при его создании.
    println!("Received event value of: {:?}", event);
}
# }
```

Обратите внимание, что вам нужно иметь доступ только для чтения канала при чтении событий. Это `ReaderId`, который должен быть изменяемым, чтобы отслеживать, где было ваше последнее чтение.

> **ВАЖНО:** Канал событий автоматически увеличивается по мере добавления к нему событий и уменьшается в размере только после того, как все читатели прочитают старые события.
>
> Это означает, что если вы создаете `ReaderId`, но не читаете его в каждом кадре, канал событий начнет потреблять все больше и больше памяти.

Шаблоны

При использовании канала событий мы обычно снова и снова используем один и тот же шаблон, чтобы максимизировать параллелизм. Последовательность такая:

Создаём канал событий и добавляем его в мир во время создания `State`:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::{ecs::{World, WorldExt}, shrev::EventChannel};
# #[derive(Debug)]
# pub enum MyEvent {
#   A,
#   B,
# }
# fn main() {
#   let mut world = World::new();
world.insert(EventChannel::<MyEvent>::new());
# }
```

**Примечание.** Вы также можете установить производную (`derive`) от `Default`, таким образом вам не нужно вручную создавать свой ресурс и добавлять его. Ресурсы, реализующие `Default`, автоматически добавляются в `Resources`, когда `System` использует их (`Read` или `Write` в `SystemData`).

В системе, **создающей события**, получите изменяемую ссылку на ваш ресурс:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::Write;
# use amethyst::shrev::EventChannel;
# #[derive(Debug)]
# pub enum MyEvent {
#   A,
#   B,
# }
# struct MySystem;
# impl<'a> amethyst::ecs::System<'a> for MySystem {
type SystemData = Write<'a, EventChannel<MyEvent>>;
#   fn run(&mut self, _: Self::SystemData) { }
# }
```

В системах, **получающих события**, вам нужно где-то хранить `ReaderId`.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::shrev::ReaderId;
# #[derive(Debug)]
# pub enum MyEvent {
#   A,
#   B,
# }
struct ReceiverSystem {
    // Тип внутри `ReaderId` должен быть типом события, которое вы используете.
    reader: Option<ReaderId<MyEvent>>,
}
```

И вам также нужно получить доступ для чтения:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::Read;
# use amethyst::shrev::EventChannel;
# #[derive(Debug)]
# pub enum MyEvent {
#   A,
#   B,
# }
# struct MySystem;
# impl<'a> amethyst::ecs::System<'a> for MySystem {
    type SystemData = Read<'a, EventChannel<MyEvent>>;
#   fn run(&mut self, _: Self::SystemData) { }
# }
```

Затем в методе системы `new`:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::shrev::{EventChannel, ReaderId};
# use amethyst::ecs::{System, SystemData, World};
# #[derive(Debug)]
# pub enum MyEvent {
#   A,
#   B,
# }
# struct MySystem { reader_id: ReaderId<MyEvent>, }
#
impl MySystem {
    pub fn new(world: &mut World) -> Self {
        <Self as System<'_>>::SystemData::setup(world);
        let reader_id = world.fetch_mut::<EventChannel<MyEvent>>().register_reader();
        Self { reader_id }
    }
}
#
# impl<'a> amethyst::ecs::System<'a> for MySystem {
#   type SystemData = ();
#   fn run(&mut self, _: Self::SystemData) { }
# }
```

Наконец, вы можете читать события из вашей системы.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::Read;
# use amethyst::shrev::EventChannel;
# #[derive(Debug)]
# pub enum MyEvent {
#   A,
#   B,
# }
# struct MySystem {
#   reader_id: amethyst::shrev::ReaderId<MyEvent>,
# }
impl<'a> amethyst::ecs::System<'a> for MySystem {
    type SystemData = Read<'a, EventChannel<MyEvent>>;
    fn run(&mut self, my_event_channel: Self::SystemData) {
        for event in my_event_channel.read(&mut self.reader_id) {
            println!("Received an event: {:?}", event);
        }
    }
}
```

---

Документация была переведена специально для сообщества "**rust_lang_ru**".

Подпишись на нас в **[Telegram][telegram]** и **[YouTube][youtube]**!

[telegram]: http://tlinks.run/rust_lang_ru
[youtube]: https://www.youtube.com/channel/UCu413rnSfuSSOR3OsIThlZA

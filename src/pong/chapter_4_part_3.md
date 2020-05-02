# Перемещение ракеток

В предыдущей главе мы узнали об отношениях между сущностями и компонентами, а также о том, как они представляют «вещи» в наших играх. Эта глава знакомит с системами (Systems) - S в «ECS». Системы - это объекты, которые представляют операции над сущностями или, более конкретно, над комбинациями компонентов. Давайте добавим систему, которая перемещает ракетки на основе пользовательского ввода.

Система - это не что иное, как функция, которая запускается один раз в каждом кадре и потенциально вносит некоторые изменения в компоненты. Если вы использовали другие игровые движки, это, вероятно, звучит знакомо: движок Unity называет эти объекты `MonoBehaviours`, а движок Unreal называет их `Actors`, но все они представляют одну и ту же базовую идею.

Системы в Specs/Amethyst немного отличаются. Вместо описания поведения одного экземпляра (например, одного врага в вашей игре), они описывают поведение всех компонентов определенного типа (всех врагов). Это делает ваш код более модульным, его легче тестировать и он работает быстрее.

Давайте начнем.

## Захват пользовательского ввода

Чтобы захватить ввод пользователя, нам нужно добавить еще несколько файлов в нашу игру. Начнем с создания файла конфигурации в каталоге `config` нашего проекта с именем `bindings.ron`, который будет содержать RON-представление структуры [`amethyst_input::Bindings`][bindings-struct]:

[bindings-struct]: https://docs.amethyst.rs/stable/amethyst_input/struct.Bindings.html

```ron,ignore
(
  axes: {
    "left_paddle": Emulated(pos: Key(W), neg: Key(S)),
    "right_paddle": Emulated(pos: Key(Up), neg: Key(Down)),
  },
  actions: {},
)
```

В Amethyst вводом могут быть либо оси (`axes`) (диапазон, который представляет собой аналоговый джойстик или связывает две кнопки как противоположные концы диапазона), либо действия (`actions`) (также известные как скалярный ввод - кнопка, которая либо нажата, либо нет). В этом файле мы создаем входные данные для перемещения каждой ракетки вверх (`pos:`) или вниз (`neg:`) по вертикальной оси: **W** и **S** для левой ракетки и клавиши со стрелками **вверх** и **вниз** для правой ракетки. Мы называем их `«left_paddle»` и `«right_paddle»`, что позволит нам ссылаться на них по имени в коде, когда нам потребуется прочитать их соответствующие значения для обновления позиций.

Далее мы добавим `InputBundle` к игровому объекту `Application`, который содержит систему `InputHandler`, которая захватывает входные данные и сопоставляет их с осями, которые мы определили. Давайте внесем следующие изменения в `main.rs`.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::prelude::*;
# use amethyst::core::transform::TransformBundle;
# use amethyst::utils::application_root_dir;
# use amethyst::window::DisplayConfig;
# macro_rules! env { ($x:expr) => ("") }
# fn main() -> amethyst::Result<()> {
use amethyst::input::{InputBundle, StringBindings};

# let app_root = application_root_dir()?;
let binding_path = app_root.join("config").join("bindings.ron");

let input_bundle = InputBundle::<StringBindings>::new()
    .with_bindings_from_file(binding_path)?;

# let path = "./config/display.ron";
# let config = DisplayConfig::load(&path)?;
# let assets_dir = "assets";
# struct Pong;
# impl SimpleState for Pong { }
let game_data = GameDataBuilder::default()
    .with_bundle(TransformBundle::new())?
    .with_bundle(input_bundle)?
    // ..
    ;
let mut game = Application::new(assets_dir, Pong, game_data)?;
game.run();
# Ok(())
# }
```

Для `InputBundle<StringBindings>` тип параметра определяет способ идентификации `axes` и `actions` в файле `bindings.ron` (в этом примере используются строки (`String`); например, `«left_paddle»`).

На этом этапе мы готовы написать систему, которая считывает ввод с `InputHandler` и соответственно перемещает ракетки. Во-первых, мы создадим каталог с именем `systems` в `src` для хранения всех наших систем. Мы будем использовать модуль для сбора и экспорта каждой из наших систем в остальную часть приложения. Вот наш `mod.rs` для `src/systems`:

```rust,ignore
pub use self::paddle::PaddleSystem;

mod paddle;
```

Наконец, мы готовы реализовать `PaddleSystem` в `systems/paddle.rs`:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
#
# mod pong {
#     use amethyst::ecs::prelude::*;
#
#     pub enum Side {
#       Left,
#       Right,
#     }
#     pub struct Paddle {
#       pub side: Side,
#     }
#     impl Component for Paddle {
#       type Storage = VecStorage<Self>;
#     }
#
#     pub const ARENA_HEIGHT: f32 = 100.0;
#     pub const PADDLE_HEIGHT: f32 = 16.0;
# }
#
use amethyst::core::{Transform, SystemDesc};
use amethyst::derive::SystemDesc;
use amethyst::ecs::{Join, Read, ReadStorage, System, SystemData, World, WriteStorage};
use amethyst::input::{InputHandler, StringBindings};

// Вам нужно будет пометить PADDLE_HEIGHT как общедоступную (pub) в pong.rs
use crate::pong::{Paddle, Side, ARENA_HEIGHT, PADDLE_HEIGHT};

#[derive(SystemDesc)]
pub struct PaddleSystem;

impl<'s> System<'s> for PaddleSystem {
    type SystemData = (
        WriteStorage<'s, Transform>,
        ReadStorage<'s, Paddle>,
        Read<'s, InputHandler<StringBindings>>,
    );

    fn run(&mut self, (mut transforms, paddles, input): Self::SystemData) {
        for (paddle, transform) in (&paddles, &mut transforms).join() {
            let movement = match paddle.side {
                Side::Left => input.axis_value("left_paddle"),
                Side::Right => input.axis_value("right_paddle"),
            };
            if let Some(mv_amount) = movement {
                if mv_amount != 0.0 {
                    let side_name = match paddle.side {
                        Side::Left => "left",
                        Side::Right => "right",
                    };
                    println!("Side {:?} moving {}", side_name, mv_amount);
                }
            }
        }
    }
}
#
# fn main() {}
```

Хорошо, здесь происходит довольно много всего!

Мы создаем структуру `PaddleSystem`, и помечаем её как производную от `SystemDesc`. Это сокращение от **System Descriptor** (Системный дескриптор). В Amethyst системам может потребоваться доступ к ресурсам из `World`, чтобы их можно было создать. Для каждой системы должна быть предусмотрена реализация типажа `SystemDesc`, чтобы указать логику для создания экземпляра системы. Для систем, которые не требуют специальной логики создания экземпляров, производная от `SystemDesc` автоматически реализует признак `SystemDesc` для типа системы.

Затем мы реализуем для нее типаж `System` с временем жизни компонентов, на которых она работает. Внутри реализации мы определяем данные, над которыми работает система, в кортеже `SystemData`: `WriteStorage`, `ReadStorage` и `Read`. В частности, обобщенные типы, которые мы использовали здесь, говорят нам, что `PaddleSystem` изменяет компоненты `Transform`, `WriteStorage <'s, Transform>`, читает компоненты `Paddle`, `ReadStorage <'s, Paddle>`, а также обращается к ресурсу `InputHandler<StringBindings>`, который мы создали ранее, используя структуру `Read`.

> Для `InputHandler<StringBindings>` убедитесь, что тип параметра совпадает с тем, который использовался для создания `InputBundle` ранее.

Теперь, когда у нас есть доступ к хранилищам компонентов, которые нам необходимы, мы можем перебирать их. Мы выполняем операцию соединения с хранилищами `Transform` и `Paddle`. Это будет перебирать все сущности, к которым прикреплены `Paddle` и `Transform`, и даст нам доступ к фактическим компонентам, без возможности изменения для `Paddle` и с возможностью изменения для `Transform`.

> Есть много других способов использования хранилищ. Например, вы можете использовать их, чтобы получить ссылку на компонент определенного типа, содержащийся в объекте, или просто выполнить итерацию по ним без объединения. Однако на практике ваше наиболее распространенное использование - объединение нескольких хранилищ, поскольку система редко влияет на один конкретный компонент.

> Также обратите внимание, что можно объединить хранилища, используя несколько потоков, используя `par_join` вместо `join`, но здесь вводимые издержки не стоят того, что дает параллелизм.

Давайте добавим эту систему в наш `GameDataBuilder` в `main.rs`:

```rust,edition2018,no_run,noplaypen
mod systems; // Импортируем модуль
// --вырезано--

# extern crate amethyst;
# use amethyst::prelude::*;
# use amethyst::core::transform::TransformBundle;
# use amethyst::input::StringBindings;
# use amethyst::window::DisplayConfig;
fn main() -> amethyst::Result<()> {
// --вырезано--

# let path = "./config/display.ron";
# let config = DisplayConfig::load(&path)?;
# mod systems {
#
# use amethyst::core::ecs::{System, SystemData, World};
# use amethyst::core::SystemDesc;
# use amethyst::derive::SystemDesc;
#
# use amethyst;
# #[derive(SystemDesc)]
# pub struct PaddleSystem;
# impl<'a> amethyst::ecs::System<'a> for PaddleSystem {
# type SystemData = ();
# fn run(&mut self, _: Self::SystemData) { }
# }
# }
# let input_bundle = amethyst::input::InputBundle::<StringBindings>::new();
let game_data = GameDataBuilder::default()
    // ...
    .with_bundle(TransformBundle::new())?
    .with_bundle(input_bundle)?
    .with(systems::PaddleSystem, "paddle_system", &["input_system"]) // Добавим эту линию
    // ...
#   ;
# let assets_dir = "/";
# struct Pong;
# impl SimpleState for Pong { }
# let mut game = Application::new(assets_dir, Pong, game_data)?;
# Ok(())
}
```

Посмотрите на вызов метода `with`. Здесь мы не добавляем комплект, мы добавляем только систему. Мы предоставляем экземпляр системы, строку, представляющую её имя и список зависимостей. Зависимости - это имена систем, которые должны быть запущены до нашей новой системы. Здесь мы требуем, чтобы `input_system` была уже запущена, поскольку мы будем использовать пользовательский ввод для перемещения ракетки, поэтому нам нужно подготовить эти данные. Имя системы `input_system` определено в стандартном `InputBundle`.

## Изменение позиции

Если мы запустим игру сейчас, мы увидим, как консоль печатает наши нажатия клавиш. Давайте сделаем так, чтобы она обновляла положение ракетки. Для этого мы изменим позицию по оси **y**.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::core::Transform;
# use amethyst::core::SystemDesc;
# use amethyst::derive::SystemDesc;
# use amethyst::ecs::{Join, Read, ReadStorage, System, SystemData, World, WriteStorage};
# use amethyst::input::{InputHandler, StringBindings};
# enum Side {
#   Left,
#   Right,
# }
# pub struct Paddle {
#   side: Side,
# }
# impl amethyst::ecs::Component for Paddle {
#   type Storage = amethyst::ecs::VecStorage<Paddle>;
# }
# #[derive(SystemDesc)]
# pub struct PaddleSystem;
# impl<'s> System<'s> for PaddleSystem {
#  type SystemData = (
#    WriteStorage<'s, Transform>,
#    ReadStorage<'s, Paddle>,
#    Read<'s, InputHandler<StringBindings>>,
#  );
fn run(&mut self, (mut transforms, paddles, input): Self::SystemData) {
    for (paddle, transform) in (&paddles, &mut transforms).join() {
        let movement = match paddle.side {
            Side::Left => input.axis_value("left_paddle"),
            Side::Right => input.axis_value("right_paddle"),
        };
        if let Some(mv_amount) = movement {
            let scaled_amount = 1.2 * mv_amount as f32;
            transform.prepend_translation_y(scaled_amount);
        }
    }
}
# }
```

Это наша первая попытка перемещения ракеток: мы берем движение и масштабируем его по некоторому коэффициенту, чтобы движение казалось плавным. В реальной игре мы использовали бы время прошедшее между кадрами, для определения того, как далеко нужно перемещать ракетку. Это необходимо чтобы поведение игры не было привязано к частоте кадров игры. Amethyst предоставляет вам  `amethyst::core::timing::Time` для этой цели, но на данный момент достаточно подходящего. Если вы запустите игру сейчас, вы заметите, что ракетки могут уходить за край игровой зоны.

Чтобы это исправить, нам нужно ограничить движение ракетки до границы арены минимальным и максимальным значением. Но поскольку точка привязки ракетки находится в середине спрайта, нам также необходимо сместить этот предел на половину высоты спрайта, чтобы ракетки не выходили на половину спрайта за экран. Поэтому мы ограничим значение **y** преобразования от `ARENA_HEIGHT - PADDLE_HEIGHT * 0,5` (верх арены минус смещение) до `PADDLE_HEIGHT * 0,5` (низ арены плюс смещение).

Наша функция `run` должна выглядеть примерно так:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::core::Transform;
# use amethyst::core::SystemDesc;
# use amethyst::derive::SystemDesc;
# use amethyst::ecs::{Join, Read, ReadStorage, System, SystemData, World, WriteStorage};
# use amethyst::input::{InputHandler, StringBindings};
# const PADDLE_HEIGHT: f32 = 16.0;
# const PADDLE_WIDTH: f32 = 4.0;
# const ARENA_HEIGHT: f32 = 100.0;
# const ARENA_WIDTH: f32 = 100.0;
# enum Side {
#   Left,
#   Right,
# }
# pub struct Paddle {
#   side: Side,
# }
# impl amethyst::ecs::Component for Paddle {
#   type Storage = amethyst::ecs::VecStorage<Paddle>;
# }
# #[derive(SystemDesc)]
# pub struct PaddleSystem;
# impl<'s> System<'s> for PaddleSystem {
#  type SystemData = (
#    WriteStorage<'s, Transform>,
#    ReadStorage<'s, Paddle>,
#    Read<'s, InputHandler<StringBindings>>,
#  );
fn run(&mut self, (mut transforms, paddles, input): Self::SystemData) {
    for (paddle, transform) in (&paddles, &mut transforms).join() {
        let movement = match paddle.side {
            Side::Left => input.axis_value("left_paddle"),
            Side::Right => input.axis_value("right_paddle"),
        };
        if let Some(mv_amount) = movement {
            let scaled_amount = 1.2 * mv_amount as f32;
            let paddle_y = transform.translation().y;
            transform.set_translation_y(
                (paddle_y + scaled_amount)
                    .min(ARENA_HEIGHT - PADDLE_HEIGHT * 0.5)
                    .max(PADDLE_HEIGHT * 0.5),
            );
        }
    }
}
# }
```

## Автоматическая настройка ресурсов системой

Возможно, вы помните, что у нас были проблемы, потому что Amethyst требует от нас зарегистрировать (`register`) хранилище для `Paddle`, прежде чем мы сможем его использовать. Теперь, когда у нас есть система, использующая компонент `Paddle`, нам больше не нужно вручную регистрировать его используя `world`: система позаботится об этом за нас, а также настроит хранилище.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::assets::Handle;
# use amethyst::ecs::World;
# use amethyst::prelude::*;
# use amethyst::renderer::SpriteSheet;
# struct Paddle;
# impl amethyst::ecs::Component for Paddle {
#   type Storage = amethyst::ecs::VecStorage<Paddle>;
# }
# fn initialise_paddles(world: &mut World, spritesheet: Handle<SpriteSheet>) { }
# fn initialise_camera(world: &mut World) { }
# fn load_sprite_sheet(world: &mut World) -> Handle<SpriteSheet> { unimplemented!() }
# struct MyState;
# impl SimpleState for MyState {
fn on_start(&mut self, data: StateData<'_, GameData<'_, '_>>) {
    let world = data.world;

    // Загрузка спрайт-листа необходима для визуализации графики.
    let sprite_sheet_handle = load_sprite_sheet(world);

    world.register::<Paddle>(); // <<-- Более не требуется

    initialise_paddles(world, sprite_sheet_handle);
    initialise_camera(world);
}
# }
```

## Подведём итог

В этой главе мы добавили обработчик ввода в нашу игру, чтобы мы могли фиксировать нажатия клавиш. Затем мы создали систему, которая будет интерпретировать эти нажатия клавиш и соответственно перемещать ракетки нашей игры. В следующей главе мы рассмотрим еще одну ключевую концепцию в играх реального времени: время. Мы проинформируем нашу игру о времени и добавим шар, который будет отталкиваться от.

---

Документация была переведена специально для сообщества "**rust_lang_ru**".

Подпишись на нас в **[Telegram][telegram]** и **[YouTube][youtube]**!

[telegram]: http://tlinks.run/rust_lang_ru
[youtube]: https://www.youtube.com/channel/UCu413rnSfuSSOR3OsIThlZA

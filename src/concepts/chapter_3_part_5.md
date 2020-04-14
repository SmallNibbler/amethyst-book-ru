# Система

## Что такое система?

Система (`System`) - это место, где выполняется логика игры. На практике система состоит из структуры, реализующей функцию, выполняемую на каждой итерации игрового цикла, и принимающей в качестве аргумента данные об игре. Системы можно рассматривать как маленькую единицу логики. Все системы запускаются движком вместе (даже параллельно, когда это возможно) и выполняют специализированную операцию с одной или группой сущностей.

Структура

Структура системы - это структура, реализующая типаж amethyst::ecs::System. Вот очень простой пример реализации:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::System;
struct MyFirstSystem;

impl<'a> System<'a> for MyFirstSystem {
    type SystemData = ();

    fn run(&mut self, data: Self::SystemData) {
        println!("Hello!");
    }
}
```

Эта система будет на каждой итерации игрового цикла выводить «Hello!» в консоли. Это довольно скучная система, поскольку она вообще не взаимодействует с игрой. Давайте немного оживим это.

Доступ к контексту игры

В определении системы этот типаж требует от вас определения типа `SystemData`. Этот тип определяет, какие данные будут предоставляться системой при каждом вызове метода `run`. `SystemData` предназначен только для переноса информации, доступной для нескольких систем. Данные, локальные для системы, обычно хранятся в самой структуре системы.

Движок Amethyst предоставляет полезные системные типы данных для доступа к контексту игры. Вот некоторые из наиболее важных:

* `Read<'a, Resource>` (или `Write<'a, Resource>` соответственно) позволяет вам получить неизменяемую (или соответственно изменяемую) ссылку на ресурс указанного вами типа. Это гарантированно не завершится сбоем, так как если ресурс недоступен, он даст вам `Default::default()` вашего ресурса.
* `ReadExpect<'a, Resource>` (или `WriteExpect<'a, Resource>` соответственно) является отказоустойчивой альтернативой предыдущему типу, поэтому вы можете использовать ресурсы, которые не реализуют типаж `Default`.
* `ReadStorage<'a, Component>` (или `WriteStorage<'a, Component>` соответственно) позволяет получить неизменяемую (или соответственно изменяемую) ссылку на всё хранилище определенного типа `Component`.
* `Entities<'a>` позволяет вам создавать или уничтожать сущности в контексте системы.

Затем вы можете использовать один или несколько из них с помощью кортежа.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{System, Read};
# use amethyst::core::timing::Time;
struct MyFirstSystem;

impl<'a> System<'a> for MyFirstSystem {
    type SystemData = Read<'a, Time>;

    fn run(&mut self, data: Self::SystemData) {
        println!("{}", data.delta_seconds());
    }
}
```

Здесь мы получаем ресурс `amethyst::core::timer::Time` для печати в консоль времени, прошедшего между двумя кадрами. Хорошо! Но это все еще немного скучно.

## Манипуляции с хранилищами

Получив доступ к хранилищу, вы можете использовать их по-разному.

### Получение компонента конкретной сущности

Иногда бывает полезно получить компонент конкретной сущности из хранилища. Это легко сделать с помощью метода `get` или, для изменяемых хранилищ, метода `get_mut`.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{Entity, System, WriteStorage};
# use amethyst::core::Transform;
struct WalkPlayerUp {
    player: Entity,
}

impl<'a> System<'a> for WalkPlayerUp {
    type SystemData = WriteStorage<'a, Transform>;

    fn run(&mut self, mut transforms: Self::SystemData) {
        transforms.get_mut(self.player).unwrap().prepend_translation_y(0.1);
    }
}
```
Эта система заставляет игрока подниматься на 0,1 единицы за каждую итерацию игрового цикла! Чтобы определить, что это за игрок, мы заранее сохранили его в структуре системы. Затем мы получаем его `Transform` из хранилища `Transform` и перемещаем его вдоль оси Y на 0,1.

> `Transform` - очень распространенная структура в разработке игр. Она представляет положение, вращение и масштаб объекта в игровом мире. Вы будете использовать их очень часто, так как они - то, что вам нужно изменить, когда вы хотите что-то изменить в своей игре.

Однако такой подход довольно редок, потому что большую часть времени вы не знаете, какой сущностью вы хотите манипулировать, и чаще вы будете применить свои изменения к нескольким сущностям.

### Получение всех сущностей с конкретными компонентами

В большинстве случаев вы захотите выполнить логику для всех сущностей с определенным компонентом или даже для всех сущностей с набором компонентов.

Это возможно с помощью метода `join`. Возможно, вы знакомы с операциями объединения, если вы когда-либо работали с базами данных. Метод `join` принимает несколько хранилищ и выполняет итерацию по всем сущностям, которые имеют компонент в каждом из этих хранилищ. Это работает как "И" ворота. Он вернет итератор, содержащий кортеж всех запрошенных компонентов, если они **все** на одной и той же сущности.

Если вы используете 'join' c компонентами A, B и C, будут рассматриваться только те объекты, которые имеют ВСЕ эти компоненты.

Само собой разумеется, что вы можете использовать его только с одним хранилищем для перебора всех объектов с определенным компонентом.

Помните, что метод `join` доступен только при импорте `amethyst::ecs::Join`.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{System, ReadStorage, WriteStorage};
# use amethyst::core::Transform;
# struct FallingObject;
# impl amethyst::ecs::Component for FallingObject {
#   type Storage = amethyst::ecs::DenseVecStorage<FallingObject>;
# }
use amethyst::ecs::Join;

struct MakeObjectsFall;

impl<'a> System<'a> for MakeObjectsFall {
    type SystemData = (
        WriteStorage<'a, Transform>,
        ReadStorage<'a, FallingObject>,
    );

    fn run(&mut self, (mut transforms, falling): Self::SystemData) {
        for (mut transform, _) in (&mut transforms, &falling).join() {
            if transform.translation().y > 0.0 {
                transform.prepend_translation_y(-0.1);
            }
        }
    }
}
```

Эта система найдёт все сущности которым принадлежат оба компонента. Компонент `Transform` с положительной координатой `y` и компонент тег `FallingObject`. И уменьшит их координату `y` на 0,1 единицу за итерацию игрового цикла. Обратите внимание, что поскольку `FallingObject` присутствует здесь только в качестве тега для ограничения операции объединения (`join`), мы немедленно отбрасываем его, используя синтаксис `\_`.

Здорово! Теперь это похоже на то, что мы действительно будем делать в наших играх!

### Получение сущностей, которые имеют некоторые компоненты, но не другие

В спецификациях есть специальный тип хранилища, называемый `AntiStorage`. Оператор *not* (`!`) превращает хранилище в его аналог `AntiStorage`, что позволяет перебирать объекты, которые *не* имеют этот компонент.

Используется так:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{System, ReadStorage, WriteStorage};
# use amethyst::core::Transform;
# struct FallingObject;
# impl amethyst::ecs::Component for FallingObject {
#   type Storage = amethyst::ecs::DenseVecStorage<FallingObject>;
# }
use amethyst::ecs::Join;

struct NotFallingObjects;

impl<'a> System<'a> for NotFallingObjects {
    type SystemData = (
        WriteStorage<'a, Transform>,
        ReadStorage<'a, FallingObject>,
    );

    fn run(&mut self, (mut transforms, falling): Self::SystemData) {
        for (mut transform, _) in (&mut transforms, !&falling).join() {
            // Если они не падают, почему бы не заставить их подняться!
            transform.prepend_translation_y(0.1);
        }
    }
}
```

## Управление структурой сущностей

Иногда может быть полезно манипулировать структурой сущностей в системе, например, создавать новые или модифицировать компоновку существующих компонентов. Этот процесс выполняется с использованием системных данных `Entities <'a>`.

> Запрос `Entities<'a>` не влияет на производительность, поскольку он содержит только неизменяемые ресурсы и, следовательно, [не блокирует диспетчеризацию][dispatcher].

[dispatcher]: https://book-src.amethyst.rs/stable/concepts/dispatcher.html

### Создание новых сущностей в системе

Создание сущности в контексте системы очень похоже на способ создания сущности с использованием структуры `World`. Единственное отличие состоит в том, что необходимо обеспечить изменяемые хранилища всех компонентов, которые вы планируете добавить в сущность.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{System, WriteStorage, Entities};
# use amethyst::core::Transform;
# struct Enemy;
# impl amethyst::ecs::Component for Enemy {
#   type Storage = amethyst::ecs::VecStorage<Enemy>;
# }
struct SpawnEnemies {
    counter: u32,
}

impl<'a> System<'a> for SpawnEnemies {
    type SystemData = (
        WriteStorage<'a, Transform>,
        WriteStorage<'a, Enemy>,
        Entities<'a>,
    );

    fn run(&mut self, (mut transforms, mut enemies, entities): Self::SystemData) {
        self.counter += 1;
        if self.counter > 200 {
            entities.build_entity()
                .with(Transform::default(), &mut transforms)
                .with(Enemy, &mut enemies)
                .build();
            self.counter = 0;
        }
    }
}
```

Эта система будет порождать нового врага каждые 200 итераций игрового цикла.

### Удаление сущности

Удалить сущность очень легко, используя `Entities <'a>`.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{System, Entities, Entity};
# struct MySystem { entity: Entity }
# impl<'a> System<'a> for MySystem {
#   type SystemData = Entities<'a>;
#   fn run(&mut self, entities: Self::SystemData) {
#       let entity = self.entity;
entities.delete(entity);
#   }
# }
```

### Перебор компонентов со связанной сущностью

Иногда, когда вы перебираете компоненты, вы также можете узнать, с какой сущностью вы работаете. Для этого вы можете использовать операцию `join` с `Entities <'a>`.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{Join, System, Entities, WriteStorage, ReadStorage};
# use amethyst::core::Transform;
# struct FallingObject;
# impl amethyst::ecs::Component for FallingObject {
#   type Storage = amethyst::ecs::VecStorage<FallingObject>;
# }
struct MakeObjectsFall;

impl<'a> System<'a> for MakeObjectsFall {
    type SystemData = (
        Entities<'a>,
        WriteStorage<'a, Transform>,
        ReadStorage<'a, FallingObject>,
    );

    fn run(&mut self, (entities, mut transforms, falling): Self::SystemData) {
        for (e, mut transform, _) in (&*entities, &mut transforms, &falling).join() {
            if transform.translation().y > 0.0 {
                transform.prepend_translation_y(-0.1);
            } else {
                entities.delete(e);
            }
        }
    }
}
```

Эта система делает то же самое, что и предыдущая `MakeObjectsFall`, но также убирает падающие объекты, которые достигли земли.

### Добавление или удаление компонентов

Вы также можете вставить или удалить компоненты из определенной сущности. Для этого вам нужно получить изменяемое хранилище компонента, который вы хотите изменить, и просто выполните:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{System, Entities, Entity, WriteStorage};
# struct MyComponent;
# impl amethyst::ecs::Component for MyComponent {
#   type Storage = amethyst::ecs::VecStorage<MyComponent>;
# }
# struct MySystem { entity: Entity }
# impl<'a> System<'a> for MySystem {
#   type SystemData = WriteStorage<'a, MyComponent>;
#   fn run(&mut self, mut write_storage: Self::SystemData) {
#       let entity = self.entity;
// Add the component
write_storage.insert(entity, MyComponent);

// Remove the component
write_storage.remove(entity);
#   }
# }
```

Имейте в виду, что вставка компонента в сущность, которая уже имеет компонент того же типа, *перезапишет предыдущий*.

## Изменение состояния через ресурсы

В одном из предыдущих разделов мы говорили о состояниях (`States`) и о том, как они используются для организации вашей игры в разные логические разделы. Иногда мы хотим инициировать переход состояния из системы. Например, если игрок умирает, мы можем захотеть удалить его сущность и дать сигнал конечному автомату выдвинуть состояние, которое показывает экран «Вы умерли».

Итак, как мы можем влиять на состояния из систем? Есть несколько способов, но в этом разделе мы рассмотрим самый простой: использование ресурса (`Resource`).

Перед этим давайте просто быстро напомним себе, что такое ресурс:

> `Resource` - это любой тип, в котором хранятся данные, которые могут вам понадобиться для вашей игры, и которые не относятся к конкретной сущности.

Данные в ресурсе доступны как для систем, так и для состояний. Мы можем использовать это в наших интересах!

Допустим, у вас есть два следующих состояния:

* `GameplayState`: состояние, в котором запущена игра.
* `GameMenuState`: состояние, в котором игра приостановлена, и мы взаимодействуем с игровым меню.

В следующем примере показано, как отслеживать в каком состоянии мы находимся в данный момент. Это позволяет нам сделать немного условной логики в наших системах, чтобы определить, что делать в зависимости от того, какое состояние в данный момент активно, и манипулировать состояниями, отслеживая действия пользователя.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
use amethyst::prelude::*;

#[derive(Clone, Copy, Debug, PartialEq, Eq)]
enum CurrentState {
    MainMenu,
    Gameplay,
}

#[derive(Clone, Copy, Debug, PartialEq, Eq)]
enum UserAction {
    OpenMenu,
    ResumeGame,
    Quit,
}

impl Default for CurrentState {
    fn default() -> Self {
        CurrentState::Gameplay
    }
}

struct Game {
    user_action: Option<UserAction>,
    current_state: CurrentState,
}

impl Default for Game {
    fn default() -> Self {
        Game {
            user_action: None,
            current_state: CurrentState::default(),
        }
    }
}

struct GameplayState;

impl SimpleState for GameplayState {
    fn update(&mut self, data: &mut StateData<'_, GameData<'_, '_>>) -> SimpleTrans {
        // Если ресурс `Game` был настроен для возврата в меню, нажмите
        // состояние меню, чтобы мы вернулись.

        let mut game = data.world.write_resource::<Game>();

        if let Some(UserAction::OpenMenu) = game.user_action.take() {
            return Trans::Push(Box::new(GameMenuState));
        }

        Trans::None
    }

    fn on_resume(&mut self, mut data: StateData<'_, GameData<'_, '_>>) {
        // Отметьте, что текущее состояние является состоянием геймплея.
        data.world.write_resource::<Game>().current_state = CurrentState::Gameplay;
    }
}

struct GameMenuState;

impl SimpleState for GameMenuState {
    fn update(&mut self, data: &mut StateData<'_, GameData<'_, '_>>) -> SimpleTrans {
        let mut game = data.world.write_resource::<Game>();

        match game.user_action.take() {
            Some(UserAction::ResumeGame) => Trans::Pop,
            Some(UserAction::Quit) => {
                // Примечание: убирать не нужно :)
                Trans::Quit
            },
            _ => Trans::None,
        }
    }

    fn on_resume(&mut self, mut data: StateData<'_, GameData<'_, '_>>) {
        // Отметьте, что текущее состояние является состоянием главного меню.
        data.world.write_resource::<Game>().current_state = CurrentState::MainMenu;
    }
}
```

Допустим, мы хотим, чтобы игрок мог нажать escape, чтобы войти в меню. Мы модифицируем наш обработчик ввода, чтобы отобразить действие `open_menu` на `Esc`, и мы пишем следующую систему:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
#
# #[derive(Clone, Copy, Debug, PartialEq, Eq)]
# enum CurrentState {
#     MainMenu,
#     Gameplay,
# }
#
# impl Default for CurrentState { fn default() -> Self { CurrentState::Gameplay } }
#
# #[derive(Clone, Copy, Debug, PartialEq, Eq)]
# enum UserAction {
#     OpenMenu,
#     ResumeGame,
#     Quit,
# }
#
# struct Game {
#     user_action: Option<UserAction>,
#     current_state: CurrentState,
# }
#
# impl Default for Game {
#     fn default() -> Self {
#         Game {
#             user_action: None,
#             current_state: CurrentState::default(),
#         }
#     }
# }
#
use amethyst::{
    prelude::*,
    ecs::{System, prelude::*},
    input::{InputHandler, StringBindings},
};

struct MyGameplaySystem;

impl<'s> System<'s> for MyGameplaySystem {
    type SystemData = (
        Read<'s, InputHandler<StringBindings>>,
        Write<'s, Game>,
    );

    fn run(&mut self, (input, mut game): Self::SystemData) {
        match game.current_state {
            CurrentState::Gameplay => {
                let open_menu = input
                    .action_is_down("open_menu")
                    .unwrap_or(false);

                // Переключите переменную `open_menu`, чтобы указать состояние
                // перехода.
                if open_menu {
                    game.user_action = Some(UserAction::OpenMenu);
                }
            }
            // Ничего не делать для других состояний.
            _ => {}
        }
    }
}
```

Теперь, когда вы играете в игру и нажимаете кнопку, связанную с действием `open_menu`, `GameMenuState` возобновляется, а `GameplayState` приостанавливается.

## Типаж SystemData

Хоть это и редко может быть полезно, но возможно создание пользовательских типов `SystemData`.

`Dispatcher` заполняет `SystemData` при каждом вызове метода `run`. Чтобы сделать это, ваш тип `SystemData` должен реализовать типаж `amethyst::ecs::SystemData` для того, чтобы он был действительным.

Это довольно сложная черта для реализации, к счастью, Amethyst предоставляет для нее производный макрос, который может реализовать черту для любой структуры, если все ее поля - `SystemData`. Однако в большинстве случаев вам даже не нужно будет реализовывать это вообще, так как вы будете использовать структуры `SystemData`, предоставляемые движком.

Обратите внимание, что кортежи структур, реализующих `SystemData`, сами являются `SystemData`. Это очень полезно, когда вам нужно быстро запросить несколько `SystemData` одновременно.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# extern crate shred;
# #[macro_use] extern crate shred_derive;
#
# use amethyst::{
#     ecs::{Component, Join, ReadStorage, System, SystemData, VecStorage, World, WriteStorage},
#     shred::ResourceId,
# };
#
# struct FooComponent {
#   stuff: f32,
# }
# impl Component for FooComponent {
#   type Storage = VecStorage<FooComponent>;
# }
#
# struct BarComponent {
#   stuff: f32,
# }
# impl Component for BarComponent {
#   type Storage = VecStorage<BarComponent>;
# }
#
# #[derive(SystemData)]
# struct BazSystemData<'a> {
#  field: ReadStorage<'a, FooComponent>,
# }
#
# impl<'a> BazSystemData<'a> {
#   fn should_process(&self) -> bool {
#       true
#   }
# }
#
#[derive(SystemData)]
struct MySystemData<'a> {
    foo: ReadStorage<'a, FooComponent>,
    bar: WriteStorage<'a, BarComponent>,
    baz: BazSystemData<'a>,
}

struct MyFirstSystem;

impl<'a> System<'a> for MyFirstSystem {
    type SystemData = MySystemData<'a>;

    fn run(&mut self, mut data: Self::SystemData) {
        if data.baz.should_process() {
            for (foo, mut bar) in (&data.foo, &mut data.bar).join() {
                bar.stuff += foo.stuff;
            }
        }
    }
}
```

---

Документация была переведена специально для сообщества "**rust_lang_ru**".

Подпишись на нас в **[Telegram][telegram]** и **[YouTube][youtube]**!

[telegram]: http://tlinks.run/rust_lang_ru
[youtube]: https://www.youtube.com/channel/UCu413rnSfuSSOR3OsIThlZA

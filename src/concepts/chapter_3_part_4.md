# Мир

## Что такое мир?

Мир (`World`) - это контейнер для ресурсов с некоторыми вспомогательными функциями, которые облегчают вашу жизнь. Эта глава продемонстрирует эти функции и их использование.

## Добавление ресурса

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
use amethyst::ecs::{World, WorldExt};

// Простая структура без данных.
struct MyResource;

fn main() {
    // Мы создали новый экземпляр `World`.
    let mut world = World::new();

    // Мы создали наш ресурс.
    let my = MyResource;

    // Мы добавили ресурс в наш мир.
    world.insert(my);
}
```

## Извлечение ресурса

Вот как можно получить ресурс только для чтения. Помните, что этот метод вызывает панику, если ресурс не присутствует в `Resources`.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{World, WorldExt};
# struct MyResource;
# fn main() {
#   let mut world = World::new();
    let my = world.read_resource::<MyResource>();
# }
```

Если вы не уверены, что ресурс присутствует, используйте методы, доступные в `Resources`, как показано в главе, посвященной ресурсам.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{World, WorldExt};
# struct MyResource;
# fn main() {
#   let mut world = World::new();
    let my = world.entry::<MyResource>().or_insert_with(|| MyResource);
# }
```

## Модификация ресурса

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{World, WorldExt};
# struct MyResource;
# fn main() {
#   let mut world = World::new();
    let mut my = world.write_resource::<MyResource>();
# }
```

## Создание сущностей

Сначала вы создаете построитель сущностей. Затем вы можете добавить компоненты к вашей сущности. Наконец, вы вызываете метод `build()` для построителя сущностей, чтобы получить данную сущность.

**Обратите внимание.** Для использования этого синтаксиса вам необходимо импортировать типаж `amethyst::prelude::Builder`.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{World, WorldExt};
# struct MyComponent;
# impl amethyst::ecs::Component for MyComponent {
#   type Storage = amethyst::ecs::VecStorage<MyComponent>;
# }
# fn main() {
#   let mut world = World::new();
    world.register::<MyComponent>();
    use amethyst::prelude::Builder;

    let mut entity_builder = world.create_entity();
    entity_builder = entity_builder.with(MyComponent);
    let my_entity = entity_builder.build();
# }
```

Короткая версия:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{World, WorldExt};
# struct MyComponent;
# impl amethyst::ecs::Component for MyComponent {
#   type Storage = amethyst::ecs::VecStorage<MyComponent>;
# }
# fn main() {
#   let mut world = World::new();
    use amethyst::prelude::Builder;

    let my_entity = world
       .create_entity()
       .with(MyComponent)
       .build();
# }
```

Внутренне `World` взаимодействует с `EntitiesRes`, который является ресурсом, содержащим сущности внутри `Resources`.

## Доступ к `Component`

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{Builder, World, WorldExt};
# struct MyComponent;
# impl amethyst::ecs::Component for MyComponent {
#   type Storage = amethyst::ecs::VecStorage<MyComponent>;
# }
# fn main() {
#   let mut world = World::new();
    // Создаём `Entity` с `MyComponent`.
    // `World` будет неявно записывать в хранилище компонента в `Resources`.
    let my_entity = world.create_entity().with(MyComponent).build();

    // Получаем ReadStorage<MyComponent>
    let storage = world.read_storage::<MyComponent>();

    // Получаем наш компонент из хранилища.
    let my = storage.get(my_entity).expect("Failed to get component for entity");
# }
```

## Изменение `Component`

Это почти то же самое, что и доступ к компоненту:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{Builder, World, WorldExt};
# struct MyComponent;
# impl amethyst::ecs::Component for MyComponent {
#   type Storage = amethyst::ecs::VecStorage<MyComponent>;
# }
# fn main() {
#   let mut world = World::new();
    let my_entity = world.create_entity().with(MyComponent).build();
    let mut storage = world.write_storage::<MyComponent>();
    let mut my = storage.get_mut(my_entity).expect("Failed to get component for entity");
# }
```

## Получение всех сущностей

Это довольно редко используется, но может быть полезно в некоторых случаях.

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{World, WorldExt};
# fn main() {
#   let mut world = World::new();
    // Возвращает `EntitiesRes`.
    let entities = world.entities();
# }
```

## Удалить сущность

Одину:
```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{Builder, World, WorldExt};
# fn main() {
#   let mut world = World::new();
#   let my_entity = world.create_entity().build();
    world.delete_entity(my_entity).expect("Failed to delete entity. Was it already removed?");
# }
```

Несколько:
```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{Builder, World, WorldExt};
# fn main() {
#   let mut world = World::new();
#   let entity_vec: Vec<amethyst::ecs::Entity> = vec![world.create_entity().build()];
    world.delete_entities(entity_vec.as_slice()).expect("Failed to delete entities from specified list.");
# }
```

Все:
```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{World, WorldExt};
# fn main() {
#   let mut world = World::new();
    world.delete_all();
# }
```

**Примечание.** Объекты удаляются лениво, что означает, что удаление происходит только в конце кадра, а не сразу при вызове метода удаления.

## Проверка, была ли сущность удалена

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{Builder, World, WorldExt};
# fn main() {
#   let mut world = World::new();
#   let my_entity = world.create_entity().build();
    // Возвращает true если сущность НЕ удалена.
    let is_alive = world.is_alive(my_entity);
# }
```

## Exec

Эта часть просто чтобы показать, что эта функция существует. Это нормально, не понимать, что она делает, пока вы не прочитаете главу "Система". Иногда вам может понадобиться создать объект, для которого вам нужно получить ресурсы, чтобы создать для него правильные компоненты. Есть функция, которая действует как сокращение для этого:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::{ReadExpect, World, WorldExt};
# struct Dummy;
# type SomeSystemData<'a> = ReadExpect<'a, Dummy>;
# trait DoSomething {
#   fn do_something(&mut self);
# }
# impl<'a> DoSomething for SomeSystemData<'a> {
#   fn do_something(&mut self) { }
# }
# fn main() {
#   let mut world = World::new();
    world.exec(|mut data: SomeSystemData| {
        data.do_something();
    });
# }
```

Мы поговорим о том, что такое `SystemData`, в главе "Система".

---

Документация была переведена специально для сообщества "**rust_lang_ru**".

Подпишись на нас в **[Telegram][telegram]** и **[YouTube][youtube]**!

[telegram]: http://tlinks.run/rust_lang_ru
[youtube]: https://www.youtube.com/channel/UCu413rnSfuSSOR3OsIThlZA

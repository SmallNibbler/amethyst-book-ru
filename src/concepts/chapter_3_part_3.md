# Ресурс

## Что такое ресурс?

Ресурс - это любой тип, в котором хранятся данные, которые могут вам понадобиться для вашей игры, и которые не относятся к конкретной сущности. Например, счет игры в понг является глобальным для всей игры и не принадлежит ни одной из сущностей (ракетка, мяч и даже текст количества очков в пользовательском интерфейсе).

## Создание ресурса

Ресурсы хранятся в контейнере `World`. Добавление ресурса в экземпляр `World` производится следующим образом:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
use amethyst::ecs::World;

struct MyResource {
    pub game_score: i32,
}

fn main() {
    let mut world = World::empty();

    let my = MyResource {
        game_score: 0,
    };

    world.insert(my);
}
```

## Получение ресурса (из `World`)

Извлечение ресурса может быть сделано так:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::World;
# #[derive(Debug, PartialEq)]
# struct MyResource {
#   pub game_score: i32,
# }
# fn main() {
#   let mut world = World::empty();
#   let my = MyResource{
#     game_score: 0,
#   };
#   world.insert(my);
  // try_fetch возвращает тип Option<Fetch<MyResource>>.
  let fetched = world.try_fetch::<MyResource>();
  if let Some(fetched_resource) = fetched {
      // Разыменование Fetch<MyResource> для доступа к данным.
      assert_eq!(*fetched_resource, MyResource{ game_score: 0, });
  } else {
      println!("No MyResource present in `World`");
  }
# }
```

Если вы хотите получить ресурс или создать его, если он не существует:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::World;
# struct MyResource;
# fn main() {
#   let mut world = World::empty();
    let my = MyResource;
  // Если ресурс не внутри `World`,
  // он вставит экземпляр, который мы создали.
let fetched = world.entry::<MyResource>().or_insert_with(|| my);
# }
```

Если вы хотите изменить ресурс, который уже находится внутри `World`:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
# use amethyst::ecs::World;
# struct MyResource {
#   pub game_score: i32,
# }
# fn main() {
#   let mut world = World::empty();
#   let my = MyResource{
#     game_score: 0,
#   };
#   world.insert(my);
  // try_fetch_mut возвращает Option<FetchMut<MyResource>>.
  let fetched = world.try_fetch_mut::<MyResource>();
  if let Some(mut fetched_resource) = fetched {
    assert_eq!(fetched_resource.game_score, 0);
    fetched_resource.game_score = 10;
    assert_eq!(fetched_resource.game_score, 10);
  } else {
    println!("No MyResource present in `World`");
  }
# }
```

Другие способы получения ресурса будут рассмотрены в разделе книги "Система".

## Удаление ресурса

Нет способа правильно «удалить» ресурс, добавленный в мир. Обычный метод для достижения чего-то похожего - это добавить `Option<MyResource>` и установить его в `None`, когда вы хотите удалить его.

## Хранилища, часть 2

`Storage` компонента - это ресурс. Компоненты «привязаны» к объектам, но, как было сказано ранее, они не «принадлежат» объектам на уровне реализации. Храня их в хранилищах и размещая `Storage` внутри `World`, он обеспечивает глобальный доступ ко всем компонентам во время выполнения с минимальными усилиями.

Фактический доступ к компонентам внутри хранилищ будет описан в разделах книги "Мир" и "Система".

**ВНИМАНИЕ:** Если вы попытаетесь получить компонент напрямую, вы не получите хранилище. Вы получите экземпляр `Default::default()` этого компонента. Чтобы получить ресурс `Storage`, в котором хранятся все экземпляры `MyComponent`, необходимо извлечь `ReadStorage<MyComponent>`.

---

Документация была переведена специально для сообщества "**rust_lang_ru**".

Подпишись на нас в **[Telegram][telegram]** и **[YouTube][youtube]**!

[telegram]: http://tlinks.run/rust_lang_ru
[youtube]: https://www.youtube.com/channel/UCu413rnSfuSSOR3OsIThlZA

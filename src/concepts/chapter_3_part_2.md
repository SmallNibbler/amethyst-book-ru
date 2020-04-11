# Сущность и компонент

## Что такое `Entity` и `Component`?

Сущность (`Entity`) представляет один объект в вашем мире. Компонент (`Component`) представляет один аспект объекта. Например, бутылка воды имеет форму, объем, цвет и изготовлена ​​из материала (обычно из пластика). В этом примере бутылка - это сущность, а свойства - это компоненты.

## Сущность и компонент в Amethyst

В дизайне наследования, сущность обычно содержит компоненты. Все данные и методы, связанные с сущностью, хранятся внутри. Однако в проекте ECS сущность является просто объектом общего назначения. На самом деле реализация `Entity` в Amethyst - это просто:

```rust,ignore
struct Entity(u32, Generation);
```

Где `u32` - идентификатор объекта, а `Generation` используется для проверки была ли удалена сущность.

`Entity` хранятся в специальном контейнере `EntitiesRes`. Принимая во внимание, что данные, связанные с объектами, группируются в компоненты и хранятся в назначенных хранилищах.

Рассмотрим пример, где у вас есть три объекта: две бутылки и человек.

|  object  |   x   |   y   |   shape  |  color  |   name  |
|:--------:|:-----:|:-----:|:--------:|:-------:|:-------:|
| Bottle A | 150.0 | 202.1 |  "round" |  "red"  |         |
| Bottle B | 570.0 | 122.0 | "square" | "white" |         |
| Person C | 100.5 | 300.8 |          |         | "Peter" |

Мы можем разделить свойства бутылки на `PositionComponent` и `BottleComponent`, а свойства человека на `PositionComponent` и `PersonComponent`. Вот иллюстрация того, как будут храниться три объекта.

![alt text](..\images\chapter_3_part_2.png)

Как видно из графика, сущности не хранят данные. Они также не знают никакой информации о своих компонентах. Они служат для идентификации объекта и отслеживания существования объекта. Хранилище компонентов хранит все данные и их связь с сущностями.

Если вы знакомы с реляционными базами данных, эта организация выглядит очень похоже на таблицы в базе данных, где идентификатор сущности служит ключом в каждой таблице. На самом деле, вы даже можете объединять компоненты или сущности, также как объединяете таблицы. Например, чтобы обновить положение всех людей, вам нужно соединить `PersonComponent` и `PositionComponent`.

## EntitiesRes

Хотя структура сущности довольно проста, манипуляции с сущностями очень сложны и имеют решающее значение для производительности игры. Вот почему сущности обрабатываются исключительно структурой `EntitiesRes`. `EntitiesRes` предоставляет два способа создания/удаления объектов:

* Немедленное создание/удаление, используется для настройки игры или очистки.
* Ленивое создание/удаление, используется в игровом состоянии. Он обновляет сущности в пакете в конце каждого игрового цикла. Это также называется атомарным созданием/удалением.

Вы увидите, как эти методы используются в следующих главах.

## Объявление компонента

Чтобы объявить компонент, вы сначала объявляете соответствующие базовые данные:

```rust,edition2018,no_run,noplaypen
/// Этот `Component` описывает форму `Entity`
enum Shape {
    Sphere { radius: f32 },
    Cuboid { height: f32, width: f32, depth: f32 },
}

/// Этот `Component` описывает преобразование `Entity`
pub struct Transform {
    /// Значение сдвиг + вращение
    iso: Isometry3<f32>,
    /// Вектор масштаба
    scale: Vector3<f32>,
}
```

А затем реализуете для них типаж `Component`:

```rust,edition2018,no_run,noplaypen
use amethyst::ecs::{Component, DenseVecStorage, FlaggedStorage};

impl Component for Shape {
    type Storage = DenseVecStorage<Self>;
}

impl Component for Transform {
    type Storage = FlaggedStorage<Self, DenseVecStorage<Self>>;
}
```

Тип хранилища будет определять способ хранения компонента, но не инициализировать хранилище. Хранилище инициализируется при регистрации компонента в `World` или при использовании этого компонента в `System`.

## Хранилища

Существует несколько стратегий хранения для различных сценариев использования. Наиболее часто используемые типы - `DenseVecStorage`, `VecStorage` и `FlaggedStorage`.

* `DenseVecStorage`: Элементы хранятся в непрерывном векторе. Между компонентами не остается пустого пространства, что позволяет уменьшить использование памяти для больших компонентов.
* `VecStorage`: Элементы хранятся в разреженном массиве. Идентификатор объекта совпадает с индексом компонента. Если ваш компонент небольшой (<= 16 байт) или поддерживается большинством объектов, это предпочтительнее, чем `DenseVecStorage`.
* `FlaggedStorage`: Используется для отслеживания изменений компонента. Полезно для кэширования.

<!-- DenseVec Storage Diagram Table -->
<div style="width: 100%">
    <h4 style="text-align: center; font-weight: bold">DenseVecStorage ( <em>entity_id</em> сопоставляет с <em>data_id</em> )</h4>
    <div style="display: flex">
        <div style="margin-right: 3em">
            <table style="text-align: center;">
                <tr><td style="background-color: #D8E5FD; color: black;">data</td></tr>
                <tr><td style="background-color: #D8E5FD; color: black;">data_id</td></tr>
                <tr><td style="background-color: #D8E5FD; color: black;">entity_id</td></tr>
            </table>
        </div>
        <div style="flex-grow: 1; text-align: center">
            <table style="width: 100%">
                <tr>
                    <td>data</td>
                    <td>data</td>
                    <td>data</td>
                    <td>data</td>
                    <td>...</td>
                </tr>
                <tr>
                    <td>0</td>
                    <td>2</td>
                    <td>3</td>
                    <td>1</td>
                    <td>...</td>
                </tr>
                <tr>
                    <td>0</td>
                    <td>1</td>
                    <td>5</td>
                    <td>9</td>
                    <td>...</td>
                </tr>
            </table>
        </div>
</div>

<!-- VecStorage Diagram Table -->
<div style="width: 100%">
    <h4 style="text-align: center; font-weight: bold">VecStorage ( <em>entity_id</em> = индекс данных, может быть пустым )</h4>
    <div style="display: flex">
        <div style="margin-right: 3em">
            <table><tr><td style="background-color: #D8E5FD; color: black;">data</td></tr></table>
        </div>
        <div style="flex-grow: 1; text-align: center">
            <table style="width: 100%">
                <tr>
                    <td>data</td>
                    <td>data</td>
                    <td style="background-color: #E22C2C99; color: black;">empty</td>
                    <td>data</td>
                    <td>...</td>
                </tr>
            </table>
        </div>
</div>

Для получения дополнительной информации см. [specs::storage][specs-storage] и [раздел «Хранилища»][storages] книги specs.

[specs-storage]: https://docs.rs/specs/0.16.1/specs/storage/index.html
[storages]: https://specs.amethyst.rs/docs/tutorials/05_storages.html

Существует множество других хранилищ, и решение о том, какое из них лучше, не тривиально и должно быть сделано на основе тщательного бенчмаркинга. Общее правило: если ваш компонент используется более чем в 30% объектов, используйте `VecStorage`. Если вы не знаете, какой из них следует использовать, `DenseVecStorage` будет хорошим вариантом по умолчанию. Ему потребуется больше памяти, чем `VecStorage` для указателей размера компонентов, но он будет работать хорошо для большинства сценариев.

## Теги

Компоненты также могут быть использованы для «маркировки» объектов. Обычный способ сделать это - создать пустую структуру и реализовать `Component`, используя `NullStorage` в качестве типа хранилища для него. Нулевое хранилище означает, что для хранения этих компонентов не потребуется пространство памяти.

Вы узнаете, как использовать эти компоненты тегов в главе «Система».

---

Документация была переведена специально для сообщества "**rust_lang_ru**".

Подпишись на нас в **[Telegram][telegram]** и **[YouTube][youtube]**!

[telegram]: http://tlinks.run/rust_lang_ru
[youtube]: https://www.youtube.com/channel/UCu413rnSfuSSOR3OsIThlZA

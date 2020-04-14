# Инициализация системы

Системы могут нуждаться в доступе к ресурсам из `World`, чтобы быть реализованными. Например, получение `ReaderId` для `EventChannel`, который существует в `World`. Когда в `World` существует канал событий, `System` должна зарегистрироваться как читатель этого канала, а не заменить его, поскольку это делает недействительными всех других читателей.

В Amethyst `World`, с которого начинается приложение, заполняется несколькими ресурсами по умолчанию - каналами событий, пулом потоков, ограничителем кадров и так далее.

Поскольку ресурсы по умолчанию начинаются со специальных ограничений, нам нужен способ передать логику инициализации `System` в приложение, включая параметры в конструктор `System`. Это информация, которую захватывает типаж `SystemDesc`.

Для каждой `System` реализация признака `SystemDesc` определяет логику для создания экземпляра `System`. Для систем, которые не требуют специальной логики инициализации, `#[derive(SystemDesc)]` автоматически реализует типаж `SystemDesc` для типа системы:

```rust,edition2018,no_run,noplaypen
# extern crate amethyst;
use amethyst::{
    core::SystemDesc,
    derive::SystemDesc,
    ecs::{System, SystemData, World},
};

#[derive(SystemDesc)]
struct SystemName;

impl<'a> System<'a> for SystemName {
    type SystemData = ();

    fn run(&mut self, data: Self::SystemData) {
        println!("Hello!");
    }
}
```

[Страница производная `SystemDesc`][systemdesc-derive] демонстрирует варианты использования, поддерживаемые производной `SystemDesc`. Для более сложных случаев страница [«Реализация типажа `SystemDesc`»][impl-systemdesc-trait] объясняет, как реализовать типаж `SystemDesc`.

[systemdesc-derive]: ./chapter_3_part_5.2.html
[impl-systemdesc-trait]: ./chapter_3_part_5.3.html

---

Документация была переведена специально для сообщества "**rust_lang_ru**".

Подпишись на нас в **[Telegram][telegram]** и **[YouTube][youtube]**!

[telegram]: http://tlinks.run/rust_lang_ru
[youtube]: https://www.youtube.com/channel/UCu413rnSfuSSOR3OsIThlZA

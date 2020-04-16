# Игра Понг

Чтобы лучше понять, как работает Amethyst, мы создадим клон игры Понг. Вы можете найти [полный пример][full-example] этой игры (наша конечная цель) в [папке примеров][examples-folder] Amethyst. Это руководство разбивает этот проект на отдельные шаги, чтобы было легче понять, что каждая отдельная часть кода делает.

[full-example]: https://github.com/amethyst/amethyst/tree/master/examples/pong
[examples-folder]: https://github.com/amethyst/amethyst/tree/master/examples

## Предварительные требования

Обязательно ознакомьтесь с главой [«Начало работы»][getting-started], перед началом урока или запуском примеров.

[getting-started]: ./chapter_1.html

## Выполнение кода после прочтения главы

Если вы клонировали репозиторий Amethyst, вы можете запустить любой из примеров следующим образом:

>\> `cargo run --example pong_tutorial_01 --features "vulkan"`

Пример с именем `pong_tutorial_xy` содержит код, который вы должны иметь после выполнения всех руководств, от 1 до xy.

> **Примечание.** В macOS вы можете использовать `"metal"` вместо `"vulkan"`.

Основное различие между реальным игровым кодом и кодом примера заключается в расположении папок `config` и `assets`.

Например, в примере pong_tutorial_01 мы имеем:

```rust,ignore
let display_config_path =
    app_root.join("examples/pong_tutorial_01/config/display.ron");

let assets_dir = app_root.join("examples/assets/");
```

Но в вашем проекте, вероятно, будет что-то вроде этого:

```rust,ignore
let display_config_path = app_root.join("config/display.ron");

let assets_dir = app_root.join("assets/");
```

---

Документация была переведена специально для сообщества "**rust_lang_ru**".

Подпишись на нас в **[Telegram][telegram]** и **[YouTube][youtube]**!

[telegram]: http://tlinks.run/rust_lang_ru
[youtube]: https://www.youtube.com/channel/UCu413rnSfuSSOR3OsIThlZA

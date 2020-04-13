# Начало работы

## Настройка Rust

Мы рекомендуем использовать [rustup][rustup] для простой установки последней стабильной версии Rust. Инструкции должны быть на экране, как только rustup загрузится.

[rustup]: https://rustup.rs/

**Обновление Rust:** Если у вас уже установлен Rust, убедитесь, что вы используете последнюю версию, запустив команду:

> \> `rustup update`

Мы рекомендуем использовать стабильную версию Rust, так как Rust Nightly обычно ломаются довольно часто.

**Использование стабильной версии:** Rustup можно настроить по умолчанию на стабильную версию, запустив команду:

> \> `rustup default stable`

## Требуемые зависимости

Пожалуйста, проверьте раздел зависимостей [README.md][readme] для получения подробной информации о том, какие зависимости необходимы для компиляции Amethyst.

[readme]: https://github.com/amethyst/amethyst/blob/master/README.md#dependencies

Обратите внимание, что вам необходимо установить функциональный графический драйвер. Если вы получаете ошибку из-за того, что средство рендеринга не может создать контекст рендеринга при попытке запустить пример, возможно проблема в неправильной установке драйвера.

## Настройка Amethyst

Вы можете использовать [Amethyst CLI][amethyst-cli] или `cargo` для настройки вашего проекта.

[amethyst-cli]: https://github.com/amethyst/tools

### Amethyst CLI

Если вы хотите использовать инструмент Amethyst CLI, вы можете установить его командой:

> \> `cargo install amethyst_tools`

А затем запустить командой:

> \> `amethyst new name`

Вы должны получить папку с файлами *Cargo.toml*, *src/main.rs* и *config/display.ron*.

### Стартовый проект

Если вы хотите начать экспериментировать с Amethyst как можно быстрее, вы также можете использовать стартовые проекты. Они специально созданы для определенных типов игр и предоставят вам основу, необходимую для немедленного начала.

Файл *README.md* содержит все, что вам нужно знать для запуска начального проекта.

> **Примечание:** прямо сейчас, единственный доступный стартовый проект для 2D-игр. Со временем появиться больше вариантов для разных типов игр.

* [2D Starter][2d-starter]

[2d-starter]: https://github.com/amethyst/amethyst-starter-2d

### Ручная настройка

Если вы используете `cargo`, вот что вам нужно сделать:

* Добавьте Amethyst в качестве зависимости в вашем *Cargo.toml*.
* Создайте папку *config* и поместите в нее *display.ron*.
* (Необязательно) Скопируйте код из одного из примеров аметиста.

## Важное замечание о версии

Amethyst делится на две основные версии:

* Выпущенная версия crates.io, которая является последней версией, доступной на crates.io.
* Git (master) версия, которая является текущей неизданной версией разработки Amethyst, доступной на [Github][github].

[github]: https://github.com/amethyst/amethyst

> **Примечание.** Вы можете узнать, какую версию книги вы просматриваете в настоящий момент, проверив URL в своем браузере. Книга/документация для мастер версии содержит «master» в адресе, версия crates.io называется «стабильной».

В зависимости от версии книги, которую вы выбрали для чтения, убедитесь, что версия Amethyst в вашем *Cargo.toml* соответствует этому.

Для выпущенной версии crates.io у вас должно быть что-то вроде этого:

```rust,ignore
[dependencies]
amethyst = "LATEST_CRATES.IO_VERSION"
```

Последнюю версию crates.io можно найти [здесь][crates-io-amethyst].

[crates-io-amethyst]: https://crates.io/crates/amethyst

Если вы хотите использовать последние неизданные изменения, ваш файл Cargo.toml должен выглядеть следующим образом:

```rust,ignore
[dependencies]
amethyst = { git = "https://github.com/amethyst/amethyst", rev = "COMMIT_HASH" }
```

Часть `COMMIT_HASH` является необязательной. Он указывает, какой конкретный коммит использует ваш проект, чтобы предотвратить непредвиденный сбой при внесении изменений в версию git.

---

Документация была переведена специально для сообщества "**rust_lang_ru**".

Подпишись на нас в **[Telegram][telegram]** и **[YouTube][youtube]**!

[telegram]: http://tlinks.run/rust_lang_ru
[youtube]: https://www.youtube.com/channel/UCu413rnSfuSSOR3OsIThlZA
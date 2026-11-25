# Використання агента та prompt-команд

## Передумови

- Відкрити репозиторій у VS Code із GitHub Copilot.
- Переконатися, що у списку доступних агентів є **Build Engineer** з `.github/agents/build-engineer.agent.md`.
- Для генерації майбутнього застосунку потрібен Node.js 26 з npm; перевірки SEA залежать від актуальної підтримки Node.js та ОС.

## Повна ініціалізація

Якщо ваша версія VS Code підтримує prompt-файли як команди, запустіть `init` і перевірте, що агент виконує всі п'ять кроків у визначеному порядку. Підтримка prompt-файлів як slash-команд залежить від версії VS Code та Copilot harness.

Якщо команда не з'явилася або prompt-файли не підтримуються як slash-команди, попросіть агента явно прочитати та виконати `.github/prompts/init.prompt.md`. Приклад запиту:

> Використай агента Build Engineer. Прочитай повний файл `.github/prompts/init.prompt.md` і виконуй його інструкції. Для кожного кроку окремо прочитай відповідний prompt-файл, дочекайся успіху й зупинись при першій помилці. Не змінюй гілку та не створюй commit або push.

Оркестратор сам послідовно читає:

1. `.github/prompts/git-init.prompt.md`
2. `.github/prompts/create-project.prompt.md`
3. `.github/prompts/create-build.prompt.md`
4. `.github/prompts/create-actions.prompt.md`
5. `.github/prompts/check.prompt.md`

Не запускайте кроки паралельно: наступний етап використовує результати попереднього. Повторний запуск має зберігати наявний коректний вміст і зупинятись на неоднозначному конфлікті.

## Окремі команди

Prompt-файли можна використати як самостійні інструкції, якщо потрібно виконати тільки один етап:

- `git-init.prompt.md` — перевірити/підготувати репозиторій без commit і push.
- `create-project.prompt.md` — створити вихідний модуль, Jest-тест і npm metadata.
- `create-build.prompt.md` — налаштувати npm scripts, SEA build і `scripts/ci.js`.
- `create-actions.prompt.md` — створити GitHub Actions workflow.
- `check.prompt.md` — виконати фінальні перевірки без змін відстежуваних файлів.

Попросіть агента явно прочитати потрібний файл із `.github/prompts/` і виконати його інструкції. Запуск `check` не є командою для виправлення: він лише звітує про помилки.

## Очікуваний результат наступного етапу

Агент створить `src/hello.js`, Jest-тест `BasicAddition`, npm manifest і lock-файл, Node.js 26 SEA executable та спільний pipeline у `scripts/ci.js`. Workflow матиме одну matrix job для Ubuntu, Windows і macOS та завантажуватиме executable як artifact. Деталі контрактів наведені в [architecture.md](architecture.md).

## Персональні дані

Заповніть ПІБ, групу й посилання на GitHub-профіль у README самостійно. Не передавайте агенту паролі, токени або приватні ключі.

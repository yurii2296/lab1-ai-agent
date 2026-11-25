# Архітектура рішення

## Призначення

Цей репозиторій готує інструкції агента для подальшої генерації мінімального JavaScript-застосунку з автоматизованим build/test pipeline та CI/CD. Поточний етап містить лише агентські інструкції й документацію; він навмисно не містить вихідного застосунку, тестів, npm manifest чи workflow.

## Цільова архітектура майбутнього проєкту

```text
src/hello.js
  ├─ вивід Hello World
  └─ експорт add(a, b)
       └─ Jest: BasicAddition

package.json + package-lock.json
  ├─ npm start
  ├─ npm run build ── scripts/build.js ── Node.js 26 SEA
  ├─ npm test ── Jest
  └─ npm run ci ── scripts/ci.js
                  ├─ npm ci
                  ├─ build
                  ├─ test
                  └─ smoke test executable

.github/workflows/ci.yml
  └─ одна job, matrix:
       ├─ ubuntu-latest ── build/hello
       ├─ windows-latest ── build/hello.exe
       └─ macos-latest ── build/hello (ad-hoc codesign)
```

## Відповідальність компонентів

- **Build Engineer agent** — застосовує вимоги, безпекові обмеження та порядок роботи.
- **`project-scaffold` skill** — створює тільки вихідний модуль, Jest-тест і npm metadata.
- **`build-and-test` skill** — володіє npm scripts, SEA збіркою та кросплатформною перевіркою в `scripts/ci.js`.
- **`github-actions` skill** — створює мінімальний workflow, що делегує всю CI логіку Node.js скрипту.
- **Prompt-команди** — забезпечують окремі кроки; `init` координує їх послідовно й зупиняється на помилці.

## Контракти та межі

- Runtime: Node.js 26; package manager/build orchestration: npm scripts; unit tests: Jest.
- Виконуваний артефакт має бути справжнім Node.js Single Executable Application, зібраним через `node --build-sea`, а не оболонкою навколо JavaScript-файлу. Якщо bundling необхідний, він обмежується одним entry point.
- `scripts/ci.js` — єдине місце послідовності `npm ci`, build, tests і executable smoke test. Workflow не дублює цю логіку.
- Кожна CI ОС збирає executable нативно на своєму runner; workflow завантажує тільки відповідний артефакт.
- Відстежуваними залишаються вихідний код, тести, manifest, lock-файл і конфігурація. `build/`, `node_modules/`, `coverage/`, кеші та секрети не додаються до Git.
- Етап `check` — read-only щодо відстежуваних Git-файлів.

## Невизначеності середовища

Доступність і точні вимоги до Node.js 26 SEA інструментів треба перевірити під час виконання. Агент має орієнтуватись на фактичні `node --help`/документацію Node.js, не вигадувати прапорці та явно звітувати, якщо локально потрібний runner чи інструмент відсутній.

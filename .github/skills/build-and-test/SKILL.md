---
name: build-and-test
description: Налаштовує npm build/test/CI scripts і Node.js 26 SEA executable, а також надійний кросплатформний scripts/ci.js.
---

# Призначення

Додати до застосунку варіанта 3 відтворюване складання, Jest-тестування, Node.js 26 Single Executable Application і єдиний CI entry point, що коректно повертає коди завершення.

# Вхідні дані

- Готові `src/hello.js`, тест і `package.json`/`package-lock.json`.
- Node.js 26 та npm; кожна цільова ОС збирає власний executable на своєму runner.
- Інструкція агента та наявні скрипти/конфігурації, які не можна перезаписувати без аналізу.

# Вихідні файли

- `scripts/build.js` або інша мінімальна реалізація build, що створює справжній SEA executable у `build/`.
- `scripts/ci.js`: кросплатформний послідовний запуск `npm ci`, build, tests і smoke test executable.
- Узгоджені npm scripts у `package.json`: `start`, `build`, `test`, `ci`.
- Узгоджені залежності та `package-lock.json`, якщо збірці потрібен esbuild або інший перевірений інструмент.
- Тимчасові SEA config/blob і згенеровані executable — лише в build output, який ігнорується Git.

# Алгоритм

1. Перевір вихідний код, package scripts, lock-файл, `.gitignore`, Node/npm версії та чистоту робочого дерева. Не перезаписуй наявний build/CI-код до розуміння його призначення.
2. З'ясуй доступний Node.js SEA flow у Node.js 26 через `node --help` та офіційну документацію. Використовуй саме `node --build-sea`; не вигадуй параметри, формат config або спосіб вбудовування SEA blob. Якщо потрібно, об'єднай entry point через esbuild, щоб SEA не залежав від непідтримуваного файлового модуля.
3. Створи executable, який при запуску виводить `Hello World`. Ім'я має відповідати ОС: `build/hello.exe` у Windows, `build/hello` у Linux/macOS. Не підміняй executable копією JS-файла чи скриптом-обгорткою.
4. Забезпеч platform-specific обробку: у macOS виконай ad-hoc signing (`codesign` із ad-hoc identity) після вбудовування; у Windows збережи `.exe`; у Linux/macOS забезпеч executable permission. Не запускай непотрібні інструменти на іншій ОС.
5. Переконайся, що `npm run build` делегує складання одному скрипту; Jest і build логіку не дублюй у YAML чи в кількох незалежних скриптах.
6. Реалізуй `scripts/ci.js` стандартними Node.js API. Виконуй `npm ci`, потім build, Jest і smoke test executable по черзі. Для кожної дочірньої команди передавай аргументи без shell-склеювання, використовуй відповідний executable для ОС, перевір `status`/`error` і заверши CI ненульовим кодом на першій помилці. Не допускай успіху при відсутньому executable або неправильному виводі.
7. Налаштуй `npm start`, `npm run build`, `npm test`, `npm run ci` без циклічного виклику. Наприклад, `npm run ci` запускає `node scripts/ci.js`, а сам `ci.js` послідовно запускає інші команди та `npm ci`.
8. Додай у `.gitignore` потрібні тимчасові/вихідні каталоги, не ігноруючи lock-файл або вихідний код. Перевір `git status` і `git check-ignore`.
9. Запусти доступні перевірки локально; якщо інша ОС недоступна, зазнач, що її поведінку перевірить GitHub Actions matrix, а не видавай її за локально протестовану.

# Перевірки

- `npm ci`, `npm run build`, `npm test` і `npm run ci` завершуються з кодом 0 на успішному сценарії.
- `npm run build` створює справжній виконуваний файл під очікуваним платформним ім'ям, що друкує `Hello World`.
- Jest містить і проходить тест `BasicAddition`.
- `scripts/ci.js` завершується з помилкою, якщо npm-команда впала, executable відсутній, його запуск неуспішний або вивід не відповідає контракту.
- Немає shell-команд із платформно-специфічними separators/path assumptions у логіці `scripts/ci.js`.
- `build/`, `node_modules/`, `coverage/`, кеші та секрети ігноруються Git.

# Приклад використання

> Прочитай `.github/skills/build-and-test/SKILL.md`. Перевір Node.js 26 SEA підтримку, налаштуй npm build/test/ci та scripts/ci.js. Створи executable, запусти перевірки й повідом, які платформи реально були доступні.

# Сценарій помилки

Якщо `node --build-sea` недоступний або SEA blob не вбудовується за підтримуваним у Node.js 26 способом, зупини збірку з точним виводом версії Node.js і команди. Не створюй JS-обгортку замість executable і не додавай неперевірений інструмент лише для маскування помилки. Якщо підпроцес повернув помилку, передай ненульовий код завершення CI та коротке повідомлення; не запускай наступні кроки.

---
name: github-actions
description: Створює мінімальний GitHub Actions workflow для Node.js 26 з однією кросплатформною matrix job та artifact executable.
---

# Призначення

Налаштувати CI GitHub Actions для майбутнього JavaScript-застосунку на Windows, Linux і macOS. Workflow оркеструє запуск спільного `scripts/ci.js`, а не повторює реалізацію build/test.

# Вхідні дані

- `package.json`, `package-lock.json` і `scripts/ci.js`, створені попередніми кроками.
- Вимоги до triggers, matrix та executable для кожної ОС.
- Поточний стан `.github/workflows/` і Git; збережи наявні workflows.

# Вихідні файли

- `.github/workflows/ci.yml`: один workflow, одна job із matrix `ubuntu-latest`, `windows-latest`, `macos-latest`.
- Мінімальні permissions на рівні workflow/job, достатні для checkout і завантаження artifact.

# Алгоритм

1. Перевір наявні workflows і `scripts/ci.js`. Якщо цільовий `ci.yml` вже існує, не перезаписуй його автоматично.
2. Налаштуй `push` для гілок `feature/**`, `develop`, `master`; YAML-патерн `feature/**` за потреби процитуй. Налаштуй `pull_request` до `develop` і `master`.
3. Створи одну job із `strategy.matrix.os`: `ubuntu-latest`, `windows-latest`, `macos-latest`.
4. Job послідовно виконує checkout, setup-node із версією 26, `node scripts/ci.js`, а після успіху завантажує executable з `build/` як artifact.
5. Artifact має містити потрібне платформне ім'я (`hello.exe` у Windows, `hello` у Linux/macOS). Можна використати безпечний glob `build/hello*` з `if-no-files-found: error`, якщо він не завантажує сторонні файли.
6. Не додавай окремих `npm ci`, `npm test` чи build-команд у YAML: єдине джерело цієї логіки — `scripts/ci.js`.
7. Не додавай secrets, токени або зайві permissions. Перевір YAML структуру, trigger-и, кількість jobs та відсутність дубльованої build/test логіки.

# Перевірки

- `ci.yml` містить рівно одну job і matrix з усіма трьома runner labels.
- Є обидва trigger-и з потрібними branch filters.
- Checkout і Node.js 26 setup передують `node scripts/ci.js`.
- Artifact upload виконується тільки після успішного CI та завершується помилкою, якщо файл відсутній.
- Workflow не містить окремих кроків `npm ci`, build чи test і не вимагає секретів.
- YAML синтаксис перевірений доступним локальним засобом або результат буде підтверджено GitHub Actions після push користувачем.

# Приклад використання

> Прочитай `.github/skills/github-actions/SKILL.md` і створи відсутній `.github/workflows/ci.yml` для варіанта 3. Збережи існуючі workflows, перевір тригери, matrix і artifact upload; не дублюй логіку `scripts/ci.js`.

# Сценарій помилки

Якщо `scripts/ci.js` або інший обов'язковий вхід відсутній, не створюй workflow, який виглядає успішним, але не може виконатися. Повідом шлях до відсутнього файлу й заверши крок помилкою, щоб orchestrator зупинився. Якщо `ci.yml` уже існує, наведи конфлікт і збережи його без змін.

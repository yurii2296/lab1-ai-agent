---
description: Налаштовує npm build/test/ci та Node.js 26 SEA executable зі спільним кросплатформним CI runner.
agent: Build Engineer
---

Прочитай і виконай `.github/skills/build-and-test/SKILL.md`. Перевір доступний Node.js 26 `node --build-sea`, створи справжні platform-specific executable та кросплатформний `scripts/ci.js`, що послідовно виконує `npm ci`, build, тести і запуск executable з перевіркою кодів завершення та виводу. Узгодь scripts `npm start`, `npm run build`, `npm test`, `npm run ci` і lock-файл. Не створюй GitHub Actions workflow. Не підміняй SEA executable JS-файлом. На помилці зупинись і повідом блокер.

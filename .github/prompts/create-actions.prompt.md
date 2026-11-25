---
description: Створює workflow GitHub Actions для Node.js 26 на Windows, Linux і macOS.
agent: Build Engineer
---

Прочитай і виконай `.github/skills/github-actions/SKILL.md`. Створи `.github/workflows/ci.yml` з однією job та matrix `ubuntu-latest`, `windows-latest`, `macos-latest`; triggers — `push` для `feature/**`, `develop`, `master` і `pull_request` до `develop`, `master`. Використовуй checkout, setup-node для Node.js 26, один виклик `node scripts/ci.js` і upload artifact executable. Не дублюй build/test/`npm ci` в YAML. Не перезаписуй наявний workflow; на конфлікті або помилці зупинись.

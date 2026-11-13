# Задача 19: План релиза студенческого пет-проекта

**Студент:** Ленец Е.

**Постановка.** `CHANGELOG` / `ROADMAP`. Ветка `release-1.1` описывает один набор изменений; ветка `hotfix-notes` правит те же разделы под срочный фикс. Создайте merge-конфликт заметок о релизе и разрешите его.

Ссылка на репозиторий: https://github.com/qqZeR002/lab5-git-conflicts

Pull Request с конфликтом: https://github.com/qqZeR002/lab5-git-conflicts/pull/1

## Ход работы

### 1. Структура репозитория

Пет-проект «Конспектик» — веб-приложение для конспектов лекций. Кода самого приложения здесь нет, в репозитории только план релизов.

| Файл | Что в нём |
|------|-----------|
| `CHANGELOG.md` | заметки о релизах |
| `ROADMAP.md` | план: ближайший релиз и что потом |
| `version.txt` | текущая версия одной строкой |
| `README.md` | этот отчёт |

### 2. Ветки

| Ветка | Зачем |
|-------|-------|
| `main` | основная |
| `release-1.1` | заметки о плановом релизе 1.1.0 (поиск, PDF, тёмная тема) |
| `hotfix-notes` | заметки о срочном фиксе 1.0.1 (пропадал конспект без названия) |

### 3. Локальный репозиторий и начальные коммиты

```bash
git init -b main
git add README.md
git commit -m "Начальный README проекта"
git add CHANGELOG.md
git commit -m "Добавил CHANGELOG с релизом 1.0.0"
git add ROADMAP.md version.txt
git commit -m "Добавил ROADMAP и файл версии"
```

В `CHANGELOG.md` на этом шаге был пустой раздел `[Unreleased]`, а в `ROADMAP.md` в разделе «Ближайший релиз» стояло «уточняется». Именно эти строки потом меняют обе ветки.

### 4. Удалённый репозиторий

Репозиторий на GitHub создал через GitHub CLI, он же привязал `origin` и сделал первый push:

```bash
gh repo create lab5-git-conflicts --public --source=. --push
git remote -v
```

То же самое без `gh`: создать пустой репозиторий на сайте, потом `git remote add origin <адрес>` и `git push -u origin main`.

### 5. Работа в ветках (сценарий конфликта)

Ветка `release-1.1`:

```bash
git switch -c release-1.1
# правка CHANGELOG.md и ROADMAP.md
git commit -am "Заметки о релизе 1.1.0 в CHANGELOG и ROADMAP"
# в version.txt: 1.1.0
git commit -am "Версия 1.1.0"
```

Ветка `hotfix-notes` (создана от `main`, а не от `release-1.1`):

```bash
git switch main
git switch -c hotfix-notes
# правка тех же строк в CHANGELOG.md и ROADMAP.md
git commit -am "Срочный фикс 1.0.1: заметки в CHANGELOG и ROADMAP"
# в version.txt: 1.0.1
git commit -am "Версия 1.0.1"
```

Что меняла каждая ветка:

| Место | `release-1.1` | `hotfix-notes` |
|-------|---------------|----------------|
| `CHANGELOG.md`, раздел `[Unreleased]` | заменён на `[1.1.0] - 2026-12-01` с тремя фичами | заменён на `[1.0.1] - 2026-11-16` с одним исправлением |
| `ROADMAP.md`, «Ближайший релиз» | версия 1.1.0, 1 декабря | версия 1.0.1 (hotfix), 16 ноября |
| `version.txt` | `1.1.0` | `1.0.1` |

Срочный фикс важнее, поэтому он попал в `main` первым:

```bash
git switch main
git merge --no-ff hotfix-notes -m "Merge branch 'hotfix-notes': срочный фикс 1.0.1"
git push origin main hotfix-notes release-1.1
```

`--no-ff` нужен, чтобы в истории остался merge-коммит и было видно, что это была отдельная ветка.

### 6. Конфликт

Открыл Pull Request `release-1.1` → `main`:

```bash
gh pr create --base main --head release-1.1 --title "Релиз 1.1.0: заметки о релизе"
```

GitHub показал, что слить нельзя: `mergeable: CONFLICTING`. Конфликт разрешал локально через **merge** (не rebase): подтянул `main` в ветку релиза.

```bash
git switch release-1.1
git merge main
```

Вывод git:

```
Auto-merging CHANGELOG.md
CONFLICT (content): Merge conflict in CHANGELOG.md
Auto-merging ROADMAP.md
CONFLICT (content): Merge conflict in ROADMAP.md
Auto-merging version.txt
CONFLICT (content): Merge conflict in version.txt
Automatic merge failed; fix conflicts and then commit the result.
```

`git status` показал три файла с пометкой `UU` (both modified). Так выглядел `CHANGELOG.md` с маркерами:

```
<<<<<<< HEAD
## [1.1.0] - 2026-12-01

### Добавлено
- поиск по тексту конспектов
- экспорт конспекта в PDF
- тёмная тема

### Исправлено
- медленная загрузка списка при 100+ конспектах
=======
## [1.0.1] - 2026-11-16

### Добавлено
- ничего, релиз только с исправлением

### Исправлено
- СРОЧНО: конспект пропадал, если сохранить его без названия
>>>>>>> main
```

`ROADMAP.md`:

```
<<<<<<< HEAD
- Версия: 1.1.0
- Дата: 1 декабря 2026
- Что войдёт: поиск, экспорт в PDF, тёмная тема
=======
- Версия: 1.0.1 (hotfix)
- Дата: 16 ноября 2026
- Что войдёт: только исправление потери конспекта без названия
>>>>>>> main
```

Между `<<<<<<< HEAD` и `=======` лежит вариант текущей ветки (`release-1.1`), между `=======` и `>>>>>>> main` — вариант из `main`.

### 7. Как разрешил

Просто выбрать одну из сторон здесь нельзя, потому что обе правки нужны:

- **`CHANGELOG.md`** — оставил **оба** раздела. Сверху `[1.1.0]`, под ним `[1.0.1]`, потому что в changelog новые версии пишут выше старых. У 1.0.1 убрал подраздел «Добавлено» со строкой «ничего», он пустой по смыслу.
- **`ROADMAP.md`** — ближайшим релизом остался 1.1.0, а фикс 1.0.1 перенёс в новый раздел «Уже выпущено». К моменту релиза 1.1 он уже выйдет.
- **`version.txt`** — взял `1.1.0`, так как это ветка релиза 1.1, и версия не может идти назад.

Маркеры `<<<<<<<`, `=======`, `>>>>>>>` удалил вручную и проверил, что их не осталось:

```bash
grep -n -E '^(<<<<<<<|=======|>>>>>>>)' CHANGELOG.md ROADMAP.md version.txt
git add CHANGELOG.md ROADMAP.md version.txt
git commit -m "Merge branch 'main' into release-1.1: разрешил конфликт заметок 1.1.0 и hotfix 1.0.1"
```

### 8. Синхронизация с remote

```bash
git push origin release-1.1      # после этого PR стал без конфликтов
gh pr merge 1 --merge            # слил PR на GitHub обычным merge-коммитом
git switch main
git pull                         # забрал merge-коммит с GitHub к себе
```

`git pull` скачал слияние, сделанное на GitHub:

```
 CHANGELOG.md | 11 +++++++++--
 ROADMAP.md   | 10 +++++++---
 version.txt  |  2 +-
 3 files changed, 17 insertions(+), 6 deletions(-)
```

### 9. Итоговая история

`git log --oneline --graph --all`:

```
*   Merge pull request #1: релиз 1.1.0
|\
| *   Merge branch 'main' into release-1.1: разрешил конфликт заметок 1.1.0 и hotfix 1.0.1
| |\
| |/
|/|
* |   Merge branch 'hotfix-notes': срочный фикс 1.0.1
|\ \
| * | Версия 1.0.1
| * | Срочный фикс 1.0.1: заметки в CHANGELOG и ROADMAP
|/ /
| * Версия 1.1.0
| * Заметки о релизе 1.1.0 в CHANGELOG и ROADMAP
|/
* Добавил ROADMAP и файл версии
* Добавил CHANGELOG с релизом 1.0.0
* Начальный README проекта
```

Историю не переписывал (без `rebase` и `push --force`), поэтому видно всё: начальные коммиты, две ветки, слияние hotfix, конфликтный merge и слияние через Pull Request. Ветки `release-1.1` и `hotfix-notes` на GitHub оставил, чтобы их можно было посмотреть.

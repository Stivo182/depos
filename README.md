# depos

[![Release](https://img.shields.io/github/release/Stivo182/depos.svg)](https://github.com/Stivo182/depos/releases)
[![Тестирование](https://github.com/Stivo182/depos/actions/workflows/test.yml/badge.svg)](https://github.com/Stivo182/depos/actions/workflows/test.yml)
[![Статус порога качества](https://sonar.openbsl.ru/api/project_badges/measure?project=depos&metric=alert_status&token=sqb_eeb76aef6de0478eafe0a1a68037f51d59bf787f)](https://sonar.openbsl.ru/dashboard?id=depos)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**depos** - инструмент для анализа и обновления версий зависимостей в манифесте пакета OneScript.

- Читает манифесты `packagedef` и `opm-metadata.xml`.
- Обновляет версии зависимостей в `packagedef` до _последней_, _минорной_ или _патч_-версии.
- Фильтрует пакеты по имени, шаблону или регулярному выражению.
- Выявляет отсутствующие, неиспользуемые и неустановленные зависимости.
- Обнаруживает конфликты версий установленных пакетов.

> [!IMPORTANT]
> **depos** не является пакетным менеджером и не устанавливает пакеты. Для установки используйте `opm install` после обновления манифеста.

## Оглавление

- [Установка](#installation)
- [Быстрый старт](#quick-start)
- [Команды](#check)
  - [check](#check)
  - [upgrade](#upgrade)
  - [doctor](#doctor)
- [GitHub Action](#github-action)

## Установка <a name="installation"></a>

```bsl
opm install depos
```

## Быстрый старт <a name="quick-start"></a>

```bash
# Перейти в каталог проекта
cd path/to/project

# Проверить зависимости на наличие проблем
depos doctor

# Проверить доступные обновления
depos check

# Обновить версии зависимостей в packagedef
depos upgrade

# Установить обновленные пакеты
opm install
```

## check 

Проверяет наличие обновлений для зависимостей без изменения манифеста Выводит список пакетов с указанием текущих и доступных версий.

**Синтаксис:**
```bash
depos check [-m|--manifest ] [-p|--package ] [-d|--deprecated] 
            [-f|--filter ] [-t|--target ] [-o|--output ]
```

| Опция | Описание |
| --- | --- |
| -m, --manifest <путь> | Путь к файлу манифеста или его каталогу. (_по умолчанию_: текущая директория) |
| -p, --package <имя> | Имя установленного пакета. |
| -d, --deprecated | Показывать только устаревшие пакеты. |
| -f, --filter <фильтр> | Фильтр пакетов по именам (через запятую или пробел), шаблону (`*`, `?`) или регулярному выражению. |
| -t, --target <тип> | Тип целевой версии (_по умолчанию_: `latest`): <br>- `latest` - последняя доступная версия пакета.<br>- `minor` - последняя минорная или патч-версия в пределах основной версии.<br>- `patch` - последняя патч-версия в пределах минорной версии. |
| -o, --output <файл> | Экспорт результатов проверки в JSON-файл. |

**Примеры использования:**

```bash
# Проверка в текущем каталоге
depos check

# Проверка конкретного манифеста
depos check --manifest /path/to/packagedef

# Проверка конкретного пакета
depos check --package autumn

# Только минорные обновления
depos check --target minor

# Только устаревшие пакеты
depos check --deprecated

# Фильтр по именам
depos check -f autumn,oint
depos check -f 'autumn oint'

# Фильтр по шаблону
depos check -f 'autumn-*'

# Фильтр по регулярному выражению
depos check -f '/^autumn-.*$/'
```

**Пример вывода:**
```bash
Проверка зависимостей: path/to/packagedef

 ↑ 1connector    2.3.1 → 2.3.3
 ↑ autumn        3.3.0 → 4.3.11
 ↑ autumn-cli    1.1.0 → 1.2.0
   semver        1.0.0 → 1.0.0

Найдено обновлений: 3
```

## upgrade 

Обновляет версии зависимостей в файле `packagedef` до указанных целевых версий. После обновления требуется установка новых версий пакетов с помощью `opm install`.

> [!CAUTION]
> Убедитесь, что файл `packagedef` находится в системе контроля версий, и все изменения зафиксированы. Это действие перезапишет ваш файл.

**Синтаксис:**
```bash
depos upgrade [-m|--manifest ] [--backup] [-f|--filter ] 
              [-t|--target ] [-o|--output ]
```

| Опция | Описание |
| --- | --- |
| -m, --manifest <путь> | Путь к файлу `packagedef` или его каталогу. (_по умолчанию_: текущая директория) |
| --backup | Создать резервную копию файла `packagedef` перед изменением. |
| -f, --filter <фильтр> | Фильтр пакетов по именам (через запятую или пробел), шаблону (`*`, `?`) или регулярному выражению. |
| -t, --target <тип> | Тип целевой версии (_по умолчанию_: `latest`): <br>- `latest` - последняя доступная версия пакета.<br>- `minor` - последняя минорная или патч-версия в пределах основной версии.<br>- `patch` - последняя патч-версия в пределах минорной версии. |
| -o, --output <файл> | Экспорт отчета об обновлениях в JSON-файл. |

**Примеры использования:**

```bash
# Обновление всех пакетов в манифесте
depos upgrade

# С резервной копией
depos upgrade --backup

# Конкретный манифест
depos upgrade --manifest /path/to/project/packagedef

# Только патч-версии
depos upgrade --target patch

# Фильтр по именам
depos upgrade -f 'autumn,1connector'

# Фильтр по шаблону
depos upgrade -f 'autumn-*'

# Фильтр по регулярному выражению
depos upgrade -f '/^autumn-.*$/'
```

**Пример вывода:**
```bash
Обновление зависимостей: path/to/packagedef

 ✓ 1connector    2.3.1 → 2.3.3
 ✓ autumn        3.3.0 → 4.3.11
 ✓ autumn-cli    1.1.0 → 1.2.0
 
Выполните opm install для установки новых версий пакетов.
```

## doctor

Выполняет комплексную диагностику проекта и выявляет проблемы:
- Отсутствующие в манифесте пакеты, используемые в коде
- Неиспользуемые пакеты, объявленные в манифесте
- Неустановленные пакеты
- Несоответствие версий установленных пакетов

**Синтаксис:**
```bash
depos doctor [-m|--manifest ] [-p|--package ] 
             [--src-dirs ] [--dev-dirs ] 
             [--ignore-dev] [--strict]
```

| Опция | Описание |
| --- | --- |
| -m, --manifest <путь> | Путь к файлу манифеста или его каталогу. (_по умолчанию_: текущая директория) |
| -p, --package <имя> | Имя установленного пакета. |
| --src-dirs <каталоги> | Каталоги с исходным кодом для анализа (через запятую). (_по умолчанию_: `src`) |
| --dev-dirs <каталоги> | Каталоги с вспомогательным кодом для анализа (через запятую). (_по умолчанию_: `tests` и `tasks`) |
| --ignore-dev | Игнорировать зависимости для разработки при анализе. Всегда `Истина` при указании опции `-p`, `--package` |
| --strict | Завершать выполнение с ненулевым кодом при обнаружении проблем. |

**Примеры использования:**
```bash
# Диагностика проекта
depos doctor

# Конкретный манифест
depos doctor --manifest /path/to/project/packagedef

# Конкретный пакет
depos doctor --package autumn

# Кастомные каталоги исходников
depos doctor --src-dirs src,lib,modules

# Кастомные dev-каталоги
depos doctor --dev-dirs tests,tasks,benchmark

# Без dev-зависимостей
depos doctor --ignore-dev

# Строгий режим (ненулевой код возврата при проблемах)
depos doctor --strict
```

**Пример вывода:**
```bash
Диагностика зависимостей: path/to/packagedef

ОТСУТСТВУЮЩИЕ В МАНИФЕСТЕ

   jason
     ├─ path/to/src/МойКласс1.os
     └─ path/to/src/МойКласс2.os
   asserts (dev)
     └─ path/to/tests/Тесты.os

НЕИСПОЛЬЗУЕМЫЕ В КОДЕ

   cli
   1bdd (dev)

НЕ УСТАНОВЛЕНЫ

  1connector

НЕСООТВЕТСТВИЕ ВЕРСИЙ

 Пакет         Установленная  Требуемая  Источник
----------------------------------------------------------------
 autumn        3.3.0          >=4.3.0    path/to/lib/autumn
 asserts  dev  1.5.0          >=1.6.0    path/to/additional_lib/asserts
 
Найдено проблем: 7
```

## Github Action

Автоматизация обновления зависимостей с созданием Pull Request:

```yaml
name: Обновление зависимостей

on:
  schedule:
    - cron: '0 0 * * 1' # Каждый понедельник
  workflow_dispatch:

jobs:
  update-dependencies:
    # if: github.repository_owner == '<your-username>'
    runs-on: ubuntu-latest
    steps:
      - name: Обновление зависимостей
        uses: Stivo182/depos-action@v1
        with:
          filter: autumn-* # Обновлять только пакеты autumn
          target: minor    # До минорных версий
          message-prefix: build(deps)
          token: ${{ secrets.PAT }}
```

Подробная документация: [depos-action](https://github.com/Stivo182/depos-action).

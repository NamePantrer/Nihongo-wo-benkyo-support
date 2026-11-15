# Nihongo-wo-benkyo-support

Вспомогательное дерево локального тьютора японского **日本語学習アシスタント** и справочника **日本語便覧**. В репозитории лежат окно, тесты, скрипты упаковки словарей, дампы JMdict и KanjiVG, сценарии запуска и сборки под Windows, а также уже закоммиченные локальные базы и следы сборки PyInstaller.

Правила продукта — в `AGENTS.md`. Статическая разметка `web/index.html` показывает три рельса: Темы, Zoom, Настройки. Для справочника `AGENTS.md` задаёт рельсы Темы, 擬音, Словарь и отдельный каталог данных `data-benran`. Имена готовых файлов в `AGENTS.md` и в `tests/test_pack.py`: `dist\Nihongo.exe` и `dist\Benran.exe`. Копирование второго файла делает `pack/win_pack.py` (имена берёт из `proba.brand`).

## Связь с nihongo-tutor

Каталога `proba/` в этом дереве нет. Его импортируют:

- `run.ps1` — модуль `proba.launch` (ключ `-Atlas` добавляет `--atlas`);
- `pack/proba.spec` — точка входа `proba/launch.py`, имя сборки `Nihongo`;
- `pack/win_pack.py` и `pack/smoke_exe.py`;
- тесты в `tests/` (`from proba import ...`).

Модули и JSON, которые эти файлы ожидают (`jlpt_kanji.json`, `compounds.json`, `giongo.json`, `giongo_en_ru.json`, `kanjivg_paths.json`, `kanjivg_parts.json`), лежат в репозитории [nihongo-tutor](https://github.com/NamePantrer/nihongo-tutor). Побайтно совпадают `run.ps1`, `build.ps1`, `pack/make_icon.py`, `pack/win_pack.py`, `pack/smoke_exe.py`, иконки в `assets/`, `web/romaji.js`, `web/sky.js`, `web/strokes.js` и часть тестов. `web/app.js`, `web/styles.css`, `pack/proba.spec` и скрипты `tools/` в двух репозиториях различаются.

Пока каталога `proba/` нет рядом с этими файлами, `.\run.ps1`, `.\build.ps1` и тесты с `import proba` не запускаются.

## Структура

| Путь | Содержимое |
| --- | --- |
| `web/` | Окно: `index.html`, `app.js`, `styles.css`, `romaji.js`, `sky.js`, `strokes.js` |
| `tests/` | Проверки пакета `proba` и скриптов упаковки |
| `tools/` | Сборка JSON из JMdict и KanjiVG |
| `pack/` | Спека PyInstaller, иконка, копирование exe справочника, дымовой запуск |
| `assets/` | `jmdict-eng.json.zip`, `jmdict-rus.json.zip`, `kanjivg-20250816.xml.gz`, иконки |
| `build/` | След PyInstaller (`*.toc`, `*.pyc`, `base_library.zip`) и куски `giongo_en_*.json` |
| `data/` | База тьютора `proba.db`, `toast_log.jsonl`, wav в `data/capture/` |
| `data-benran/` | База справочника |
| `dist/` | Базы и `proba-debug.log` в каталоге, куда сборка кладёт exe |
| `run.ps1`, `build.ps1` | Запуск из исходников и сборка |
| `requirements.txt` | Зависимости Python |
| `AGENTS.md` | Правила продукта |

В хабе «Настройки» (`web/app.js`) пункты: Срез, Текст или PDF, Занятие вручную, Очередь, Словарь, Черновики, К учителю, После паузы. Адрес `#/import` открывает `#/settings?s=text`.

## Запуск и сборка

Скрипты написаны на PowerShell под Windows. `run.ps1` ищет `%LOCALAPPDATA%\Programs\Python\Python313\python.exe`, иначе вызывает `py`, создаёт `.venv`, ставит `requirements.txt` и запускает модуль.

```powershell
.\run.ps1
.\run.ps1 -Atlas
```

`build.ps1` останавливает уже запущенный exe, ставит зависимости и PyInstaller, вызывает `pack/make_icon.py` и собирает `pack/proba.spec`. Спека пишет `dist\Nihongo.exe`. Шаг `pack/win_pack.py copy` копирует его во второй exe справочника и удаляет прочие exe и ярлыки с рабочего стола и из меню Пуск. Список `excludes` в спеке выкидывает из exe `faster_whisper` и соседние пакеты распознавания. В `requirements.txt` пакет `faster-whisper` указан: им пользуется запуск из исходников, когда пакет `proba` на месте.

Дымовая проверка собранного файла — `pack/smoke_exe.py`. Ей нужны `proba.brand` и файл `dist\Nihongo.exe`.

Тесты, когда интерпретатор видит пакет `proba`:

```powershell
.\.venv\Scripts\python.exe -m unittest discover -s tests -q
```

## PDF и метка TeX

Файлов `.tex` в дереве нет, и репозиторий не собирает PDF.

Долю языка TeX на GitHub дают два файла PyInstaller: `build/proba/Analysis-00.toc` и `build/proba/EXE-00.toc`. Это списки модулей сборки.

Чужой PDF окно принимает на экране «Текст или PDF»: поле файла с `accept="application/pdf"`, запрос `POST /api/import/pdf`. В зависимостях за чтение PDF отвечает `pypdf`. Обработчик этого адреса находится в пакете `proba`.

## Упаковка справочных JSON

Скрипты читают список кандзи из `proba/jlpt_kanji.json` и пишут результат в `proba/`. Эти пути появляются вместе с пакетом из nihongo-tutor. Архивы словарей уже лежат в `assets/`. Если файла нет или он короче порога в скрипте, скрипт скачивает релиз [jmdict-simplified](https://github.com/scriptin/jmdict-simplified) `3.6.2+20260622163854`.

```powershell
.\.venv\Scripts\python.exe tools\pack_compounds.py
.\.venv\Scripts\python.exe tools\pack_giongo.py
.\.venv\Scripts\python.exe tools\pack_giongo.py rewrite
.\.venv\Scripts\python.exe tools\pack_kanjivg.py
```

- `pack_compounds.py` читает `assets/jmdict-rus.json.zip` и пишет `proba/compounds.json` (`source`: `jmdict`).
- `pack_giongo.py` читает русскую и английскую выгрузки JMdict, отбирает пометы `on-mim` и подставляет русский текст из `proba/giongo_en_ru.json` в `proba/giongo.json`. Аргумент `rewrite` заново прогоняет уже записанный `giongo.json` через очистку глосс.
- `pack_kanjivg.py` читает `assets/kanjivg-20250816.xml.gz` и пишет `proba/kanjivg_paths.json` и `proba/kanjivg_parts.json`. В `AGENTS.md` начертания KanjiVG указаны как CC BY-SA.

Файлы `build/giongo_en_chunk_*.json`, `build/giongo_en_ru_*.json` и `build/giongo_en_unique.json` скрипты `tools/` не открывают.

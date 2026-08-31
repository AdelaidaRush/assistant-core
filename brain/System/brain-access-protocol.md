---
name: brain-access-protocol
description: How Jarvis code reads and writes the Brain — markdown-as-RAG contract. Stable paths, frontmatter, retrieval flow.
type: reference
updated: 2026-06-28
---

# 🔌 Brain access protocol (для кода Jarvis)

Контракт между кодом Jarvis и мозгом. Цель — чтобы ретривер был простым и надёжным.

## Где
- Корень: `brain` (стабильный ASCII-путь, не iCloud → нет «выгрузки» файлов).
- Индекс: `_INDEX.md`. MOC: `*/_*.md`. Атомарные заметки: `*/*.md` с frontmatter.

## Чтение (retrieval flow)
1. Всегда сначала парсить `_INDEX.md` — это карта (дёшево).
2. Для запроса:
 - **MVP-ретривер:** матч по `description:` из frontmatter всех заметок (одна строка на файл) → берём top-k файлов целиком.
 - **v2:** эмбеддинги тела заметок (chunk = заметка, она уже атомарна) → косинус top-k. Frontmatter `description` — хороший короткий вектор для предфильтра.
3. В контекст модели подаём **только top-k заметок**, не весь мозг.

## Парсинг заметки
- Frontmatter YAML: `name` (= id для ``ссылок``), `description`, `type`, `updated`.
- Тело: markdown, связи ``name`` → строить граф (как в Obsidian).
- `type`: `index | moc | reference | brand | project | tool`. Фильтровать по типу при поиске.

## Запись (когда Jarvis узнал новое)
1. Grep по `name:`/`description:` — нет ли дубля.
2. Новый файл по формату из [[memory-protocol]] в нужной папке (`System/Playbooks/Preferences`).
3. Добавить строку в `_INDEX.md` и в соответствующий MOC.
4. Никаких секретов в теле — только указатель на защищённый файл.

## Технологии (рекомендация, не догма)
- Парсер frontmatter: `python-frontmatter` / `gray-matter`.
- Эмбеддинги: локальные (`sentence-transformers`) или API — решим на этапе 2.
- Хранилище векторов: на старте in-memory/`faiss`; vault остаётся источником правды, индекс — производное.

## Инварианты
- Vault — **единственный источник правды**. Любой векторный индекс перестраивается из него.
- Пути стабильны. Имена файлов = `name` из frontmatter (kebab-case).
- Markdown остаётся человекочитаемым (владелец правит в Obsidian — Jarvis это видит).

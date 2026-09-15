# INTEGRATION.md — как перенести прототипы в SpinLid

> Документ для ИИ-ассистента (Claude Code / Cursor / ZCode), которому поручили перенести один из шести дизайнов из этого репозитория в рабочее приложение SpinLid. Прочитайте целиком перед началом работы.

---

## 1. Контекст

В этом репозитории — **автономные HTML-прототипы** трёх ключевых страниц продукта SpinLid (B2B-лидогенерация по отзывам с карт), выполненные в шести разных дизайн-системах. Они не подключены к API и наполнены демо-данными. Их цель — выбрать дизайн-направление и служить пиксельной/logic-спецификацией при переносе.

Живой просмотр: **https://the4per.github.io/spinlid-ui-proto/**

Основной продукт: Next.js 14 (App Router) + TypeScript + Tailwind 3.4, backend FastAPI. Рабочий репозиторий: `colaba2402` (монорепо, см. его `AGENTS.md` — правила обязательны).

## 2. Карта прототипов

```
index.html                          ← сравнение 6 систем, начинать отсюда
01-poisk-lidov.html                 ← BENTO: вход поиска
02-rezultaty-poiska.html            ← BENTO: результаты
03-poisk-po-boli.html               ← BENTO: поиск по боли
designs/shadcn/01..03               ← SHADCN (нейтральный, data-таблица)
designs/neobrutalism/01..03         ← NEOBRUTALISM (рамки 2px, тени 4px)
designs/editorial/01..03            ← EDITORIAL (Playfair, газета)
designs/enterprise/01..03           ← ENTERPRISE (тёмный дашборд)
designs/premium/01..03              ← PREMIUM (Apple-стиль)
preview/*.png                       ← скриншоты
INTEGRATION.md                      ← этот файл
```

Каждая страница = один самодостаточный HTML: все стили в `<style>` (дизайн-токены — CSS-переменные в `:root` в начале файла), вся интерактивность — маленький `<script>` в конце, демо-данные — массивы в этом же скрипте. Шрифты — Google Fonts (в продукте заменить на локальные/`@fontsource`, см. §6.4).

## 3. Семантика страниц (одинакова во всех системах)

**01 — Вход поиска.** Ниша + город (datalist-подсказки) → источники (2GIS/Я.Карты/Google) → CTA. Быстрый старт (4 пресета), «Мои пресеты», свёрнутая «Тонкая настройка»: режим город/радиус+адрес+слайдер, слова в отзывах (содержит/не содержит), доп. ниши/города (массовый прогон). Подсказка: фильтры — на странице результатов.

**02 — Результаты.** Шапка: запрос, счётчики «найдено/под фильтры», бейдж «из кэша», экспорт + «Сформировать КП». Прогресс сбора (SSE в реальном приложении). Строка применённых фильтров-чипов. Сайдбар: 7 готовых сценариев (с живыми счётчиками), рейтинг от/до, отзывов от, негативных от, ответы владельца, сайт, ЛПР, hh.ru, оборот/возраст/тип юрлица, слова в отзывах, боли клиентов (мультиселект чипов), источник, сортировка. Список компаний (таблица в shadcn/enterprise, строки/карточки в остальных) + плавающая bulk-панель при выборе: в список / КП / найти ЛПР / экспорт.

**03 — Поиск по боли.** Ранжированный список болей с барами частот (ширина = count, моно-цифра справа) — первичный ввод; «показать ещё»; свой текст → матчинг тега. Город/ниша/источник. Результаты: карточки с pain-тегами, цитатой, «готовностью купить», контактом; выбор всех + Excel/написать/в список.

Во всех системах информационная архитектура идентична — отличается только кожа. Это позволяет вести перенос системы X постранично, не меняя логику.

## 4. Дизайн-токены по системам

Точные значения — в `:root` каждой страницы. Сводно:

| Система | Фон | Акцент | Текст | Шрифты (прототип) | Радиусы/тени |
|---|---|---|---|---|---|
| Bento | `#F4EBDA` бумага, ячейки `#FFFBF2` | персик `#FAD4C0`→`#F0B590`, текст-акцент `#C2410C` | `#1C1917` | Golos Text + JetBrains Mono | ячейки 20px, мягкая тёплая тень |
| Shadcn | `#FFFFFF`, секции `#FAFAFA` | чёрный `#18181B` (primary-кнопка) | `#09090B` / `#71717A` | Inter Tight + Fira Code | 8px, теней нет, ring-фокус |
| Neobrutalism | `#FBFBF9` + точечный паттерн | жёлтый `#FDC800`, фиолетовый `#432DD7` | `#1C293C` | Montserrat + JetBrains Mono | рамки `2px solid`, тень `4px 4px 0`, кнопки-пилюли |
| Editorial | `#FFFFFF` | IBM-синий `#0F62FE` | `#111111` / `#444` | Playfair Display + IBM Plex Sans + Ubuntu Mono | 0–4px, волосяные линейки 1px |
| Enterprise | `#09090B`, панели `#101014` | синий `#0C5CAB`, текст-ссылки `#4D9FFF` | `#FAFAFA` / `#A1A1AA` | IBM Plex Sans + IBM Plex Mono | 8–12px, стеклянные панели, мягкие тени |
| Premium | `#FFFFFF`, секции `#F5F5F7` | один синий `#3B82F6` | `#1D1D1F` / `#6E6E73` | системный стек (SF) + JetBrains Mono | кнопки-пилюли, карточки 16–20px, тень `0 8px 24px rgba(0,0,0,.06)` |

Семантические цвета во всех: success `#16A34A`, warning `#D97706`, danger `#DC2626` (enterprise: `#10B981/#F59E0B/#EF4444`).

## 5. Куда это встраивать в рабочем проекте

Соответствие «прототип → файлы продукта» (пути от корня `colaba2402/frontend`):

| Прототип | Реальная страница | Главные компоненты |
|---|---|---|
| 01 вход поиска | `app/app/leads/page.tsx` (таб «По картам») | `components/maps/MapsSearchPanel.tsx`, `MapsSearchForm.tsx` (1091 строк) |
| 02 результаты | тот же роут, состояние `results` | `components/maps/MapsSearchResults.tsx` (2617), `MapsFiltersPanel.tsx` (1202), `MapsCompanyCard.tsx` (680), `MapsCompanyDetailDrawer.tsx`, `useSearchStream.ts` (SSE) |
| 03 по боли | `app/app/pains/page.tsx` (1261, самодостаточная) | `components/pains/DraftEmailPopover.tsx`, общий `MapsCompanyDetailDrawer` |

Токены продукта живут в `app/globals.css` («SpinLid Design Tokens v2» + legacy HSL) и `tailwind.config.js`. Шрифты подключены через `@fontsource` (`Manrope`, `Unbounded`). **shadcn/radix НЕ установлены** — `components/ui/*` самописные lookalike'ы.

## 6. Пошаговый план интеграции

### Шаг 0. Выбор системы
Человек выбирает одну систему (смотреть страницу «Результаты» каждой — она самая плотная). Дальше работаем только с её тремя файлами.

### Шаг 1. Токены
1. Из `:root` выбранного прототипа перенести переменные в `app/globals.css` (заменить/дополнить «SpinLid Design Tokens v2»; legacy-переменные `--bg --surface --border ...` переопределить, их используют старые экраны — не удалять).
2. Отразить палитру в `tailwind.config.js` (расширение `colors`, `borderRadius`, `boxShadow`, `fontFamily`).
3. Шрифты прототипа добавить через `@fontsource/*` (Golos Text, IBM Plex Sans/Mono, Playfair Display, Montserrat, Inter Tight, Fira Code, Ubuntu Mono — у всех есть кириллица), подключить в `app/layout.tsx`. Для Premium оставить системный стек — вебшрифт не нужен.

### Шаг 2. Общие компоненты
Собрать из прототипа базовые кирпичи как React-компоненты рядом с существующими `components/ui/*` (не переписывая старые сразу): кнопка (primary/ghost, у neo — с жёсткой тенью и :active-сдвигом), инпут с focus-состоянием, чип/токен, светофор-флаги (bad/warn/good/mut), микро-лейбл uppercase, прогресс-бар, bulk-панель. Иконки — только инлайн-SVG 1.5–2px stroke (в прототипах уже есть готовые пути — брать оттуда).

### Шаг 3. Страница входа (01)
Перенести разметку `MapsSearchForm.tsx` в новую структуру: hero + форма + быстрый старт + пресеты + `<details>` тонкой настройки. Существующие контролы (datalist из `lib/cities.ts`, `builtinPresets.ts`, `SaveFilterPresetModal`) сохранить и обернуть новой кожей. CTA — существующий обработчик создания поиска (`POST /maps/search`).

### Шаг 4. Результаты (02)
- Шапка/прогресс: существующий SSE-поток `useSearchStream.ts` (`/maps/search/{id}/stream`) → прогресс-бар и счётчики прототипа.
- Сайдбар фильтров: `MapsFiltersPanel.tsx` — перенести ВСЕ поля (инвентарь в §3) в сгруппированный вид прототипа; значения → query-параметры `listMapCompanies` (см. §7). Дебаунс 500 мс на числах уже есть — сохранить.
- Список: карточки/строки по прототипу (в shadcn/enterprise — таблица `<table>`, см. их 02). Данные — те же, что питают `MapsCompanyCard`.
- Bulk-панель: закрепить существующие обработчики «В список» (`AddToListModal`), «Сформировать КП» (localStorage `lib/kp-bulk-pending.ts` + `/app/leads/kp-jobs/new`), «Найти ЛПР» (`POST /maps/companies/enrich-team`), экспорт (`/maps/search/{id}/export`, website-leads XLSX).

### Шаг 5. По боли (03)
`app/app/pains/page.tsx`: тайлы/бары частот ← `listPainTags` (`GET /maps/pain-tags?niche=&city=&source=&sentiment=`), счётчик — `occurrences_count`. Выбор тегов → `pain_tag_ids[]`. Выдача ← `listCompaniesByPain`. Свой текст → матчинг по лейблам тегов (логика уже есть на странице). Batch-действия: экспорт `/maps/pains/companies/export`, письмо — `DraftEmailPopover`, список — `AddToListModal`.

### Шаг 6. Проверки
```
cd frontend && npm run lint && npm run type-check
```
Затем вручную: вход → поиск по пресету → фильтры → выбор 3 компаний → bulk → КП-флоу; страница болей → выбор 2 болей → выдача → экспорт.

## 7. API-шпаргалка (полный клиент — `src/services/api/maps.ts`)

- `POST /maps/search` — `{niche, city, sources[], filters, mode: city|radius, address, radius_meters}`; режим радиуса — только 2GIS.
- `GET /maps/search/{id}/companies` — `min_rating, max_rating, min_reviews, min_negative, has_owner_replies, has_website, has_lpr, hiring_marketing, pain_tag_ids[], min_pain_mentions, review_text_contains(+_any[]), review_text_excludes(+_any[]), source_filter, opf_in[], sort_by, limit, offset`.
- `GET /maps/search/{id}/stream` — SSE: события company/progress/done, 3 реконнекта.
- `GET /maps/pain-tags?niche=&city=&source=&sentiment=` — источник баров на 03.
- `GET /maps/pains/companies?pain_key=&pain_tag_ids[]&city=&niche=&limit=&offset=`.
- Сортировки: `rating_asc/desc, reviews_desc, negative_desc, pain_desc, temperature_desc, website_score_desc` (`ai_score_*` — только клиентские).

## 8. Чего НЕ ломать

1. **URL-параметры**: `?map_search_id=` восстанавливает выдачу (обёртка в Suspense на месте); `?niche=&city=` на болях — drill-through из админки.
2. **SSE-стрим** и реконнекты; live-прогресс во время парсинга.
3. **Аутентификацию**: `middleware.ts` пускает по куке `access_token`; JS-детект — `auth_present`. Не трогать.
4. **localStorage-контракты**: `lib/kp-bulk-pending.ts` (массовое КП), пресеты пользователя (`src/services/api/user-presets.ts`).
5. **Мобильную версию**: сейчас фильтры открываются через `BottomSheet` — сохранить паттерн, адаптив прототипов (медиа-запросы в их CSS) как ориентир.
6. **Правила репозитория** `AGENTS.md`: никаких push без явного разрешения человека; conventional commits; файлы только в `frontend/`.

## 9. Правила стиля (почему прототипы выглядят «не как ИИ»)

Сохранять при переносе: **один акцентный цвет** на систему; никаких эмодзи-иконок (только SVG-stroke); никаких фиолетово-синих градиентов и glow-теней; текст-«мьют» всегда тонирован в цвет системы (не чисто-серый); цифры/телефоны/счётчики — моно-шрифт с tabular-nums; focus-visible кольца обязательны (WCAG 2.2 AA); touch-таргеты ≥ 40px; `prefers-reduced-motion`; заголовки без «Title Case»-крика, микро-лейблы uppercase с трекингом.

## 10. Как обновлять сами прототипы

Прототипы живут в этом репозитории; GitHub Pages пересобирается автоматически при push в `main` (~1–2 мин). Правки демо-данных/копирайта — правьте HTML и пушьте; основное приложение от этого репозитория не зависит.

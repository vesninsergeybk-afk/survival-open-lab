# A3 — UX, подача, техника: разрушительный аудит K9–K12

Дата: 2026-11-07 · Ветка: `main` · Метод: только чтение кода/ассетов, инвентаризация файлов, grep по именам.
Базовая линия тестов на момент аудита: `node --test "arcade/tests/*.test.mjs"` → **1234 pass / 0 fail**.
Ранее проверено и принято как факт: `docs/AUDIT-MENU-2026-11-06.md`, `docs/AUDIT-MENU-SELFREVIEW-2026-11-06.md`, `qa/MENU-REAUDIT-2026-11-06.md`.
Канон цитируется по: `.agents/skills/game-ui-and-hud/SKILL.md` (HUD), `ui-ux-game/SKILL.md` (UX), `game-ui-frontend/SKILL.md` (фронтенд), `game-playtest/SKILL.md` (проверка).

## 1. Таблица контура

| Контур | Канон | Факт (файл:строка) | Разрыв | Уверенность |
|---|---|---|---|---|
| K9 · служебные данные в HUD | «On HUD только то, что читают mid-action» (game-ui-and-hud) | `arcade/index.html:118` `telemetry-foot`: `60 FPS · 0 ЦЕЛЕЙ`; пишется каждый кадр `hud.js:94`; виден на desktop (`hud.css:70`, `hud-art.css:113`), скрывается только при `pointer:coarse` (`hud-art.css:242`) | FPS — отладочная метрика, не игровая величина; игрок видит «служебное» | высокая |
| K9 · дубли индикаторов | «Less is more; state is always visible — но один раз» (ui-ux-game) | Рывок: `index.html:60` `leftBurstText` + `index.html:82` `bursttext`, оба пишутся `hud.js:168`. Дроны: `index.html:104-110` (`hudDrone0/1`+`wingStatus`) + `index.html:115` `wing`. Коды: `index.html:77` слот + `index.html:116` `hack`. Удар: `index.html:101-103` + `index.html:86` слот | Одна и та же величина показана 2–3 раза в одном кадре | высокая |
| K9 · две параллельные строки статуса | «Одна иерархия информации» (game-ui-and-hud) | `stateText` — 12 ветвей (`hud.js:192-198`, `210`); `flightMode` — ещё 12 ветвей (`hud.js:212-220`, `230`) с теми же смыслами (добыча, ремонт, форсаж, доктрина) | Два конкурирующих канала об одном и том же; на desktop оба длинные | высокая |
| K9 · добыча в трёх местах | «один сигнал — одно место» (game-ui-and-hud) | `miningStatus` (`index.html:90`, `hud.js:139-157`) + `flightMode` «ДОБЫЧА» (`hud.js:214`) + `contractObjective` (`index.html:123`) | Тройное кодирование одного события | высокая |
| K9 · честность приборов | «не показывать пустой датчик» (комментарий-канон `hud.js:232-233`) | Термодатчик скрыт без модели: `hud.js:235-237`; `hidden` правило `hud.css:11` | Разрыва нет — сделано верно, сохранить | высокая |
| K9 · доступность радара | «colour-blind safety» (game-ui-and-hud) | Враги и союзники — одинаковые круги, различие только цветом: `hud.js:404-405` (`#ff7b83` / `#6ce5ff`) | Кодирование только цветом | высокая |
| K9 · объявления для скринридера | «screen reader support» (ui-ux-game) | `aria-live="off"` у цели (`index.html:123`) и добычи (`index.html:90`); `interactionCue` без live (`index.html:122`) | Смена цели/угрозы не озвучивается | средняя |
| K10 · слои меню | «Protect the playfield; один визуальный язык» (game-ui-frontend) | 13 `<link>` CSS (`index.html:8-20`); меню рисуют 5 слоёв: `menu-system.css` (1230 стр.), `menu-rebuild.css` (372), `all-menus.css` (149), `station-art.css` (233), `station-terminal-art.css` (102); **23 селектора определены в двух слоях** (замер) | Каскадный долг: правка требует знать порядок 5 файлов | высокая |
| K10 · монолит панели станции | «Build clear hierarchy» (game-ui-frontend) | `base-panel.js` — 1004 строки, ~20 панелей в одном модуле (`firstContractRewardPanel:196`, `contractPanel:224`, `corridorPanel:268`, `probePanel:300` …) | Нет модульных границ: любая правка экрана = правка 1000-строчного файла | высокая |
| K10 · экраны и потоки | «feedback instants; teach by doing» (ui-ux-game) | `menu-screens.js` — 5 экранов с `aria-modal`, экранированием (`:4`) и `<details>`-пояснениями (`:23`, `:34`); системное меню: roving tabs (`game-menu.js:39-55`), восстановление фокуса (`:3-12`) | Разрыва не найдено (потоки уже отсмотрены в AUDIT-MENU-2026-11-06) | средняя |
| K11 · музыка | «CC BY 4.0 attribution» (`music-playlist.js:3-4`) | 12 треков (`music-playlist.js:6-19`), 5 пулов (`:21-27`), ротация с сохранением (`:32-58`), все 12 файлов на диске (44,3 МБ `assets/audio`, `CREDITS.md`) | Разрыва нет | высокая |
| K11 · адаптивная музыка | 5 сцен (audio-director.js) | `MUSIC_SCENES` 5 сцен (`:5-11`), гистерезис 5.5 с и recovery 9 с (`:53-62`), «окно входа» 24 с (`:41-42`) | Разрыва нет | высокая |
| **K11 · звук оружия — тишина** | «Feedback is instant: визуальный + звуковой отклик» (ui-ux-game) | `audio-lab/sound-map.js:78-95` подменяет 46 сэмплов на `./assets/sfx/kenney/<name>.ogg`; в `audio-lab/assets/sfx/kenney/` лежат **только `manifest.json` и `SOURCES.md` (0 .ogg)**; `audio-mixer.js:7-8`: «no oscillator/noise fallback… Missing files stay silent». Нет файлов: `impactMetal_medium_*`, `impactSoft_medium_*`, `laserLarge_*`, `laserRetro_*`, `forceField_*`, `explosionCrunch_*`, `thrusterFire_000`, `spaceEngineLow_000`, `doorClose_*`, `computerNoise_*`, `impactMining_*` | Плазма/рельса/ион/картечь/ракета, взрывы, урон, смерть, level-up, босс — **беззвучны**; тесты проверяют манифест, а не наличие файлов | **очень высокая** |
| K11 · 21 studio-сэмпл | «только проверенные файлы» (`sound-map.js:76-77`) | `WEAPON_SOUNDS` ссылается на `./assets/sfx/studio/*.ogg` (`sound-map.js:12-24`); в `audio-lab/assets/sfx/studio/` — только `SOURCES.md` | 21 мёртвая ссылка (перекрыта `replacements`, но манифест врёт) | высокая |
| K11 · базовые сэмплы | «recorded CC0 station recordings» | 29 файлов `audio-lab/assets/sfx/base/*.ogg` реально есть и звучат (`heritage.js:143-155`: `base_repair_pulse`, `unlock_tech_tier`, `base_raid_warning` …) | Работает только станционный слой | высокая |
| K11 · VFX | «readable at a glance; reduced motion» | Ограничения: очередь 64 (`weapon-fx.js:13`), «no shadows/gradients» (`:7`), reducedMotion (`:78,173,202`); fps-гейты `combat-feedback.js:65,73,99`, `tactical-vfx.js:189`, `visuals.js:232` | Разрыва нет — VFX дисциплинированы | высокая |
| K12 · переназначение клавиш | «remappable controls» (ui-ux-game) | `control-layout.js:32-51` валидация и `RESERVED`; подписи в меню переписываются `game-menu.js:24-25` | Разрыва нет | высокая |
| K12 · reduced motion | «reduced motion» (game-ui-and-hud) | `hud.js:63`, `332-336`; `visuals.js:184-185`; 5 блоков `prefers-reduced-motion` в `hud-art.css` (313/410/496/554/640) | Разрыва нет | высокая |
| K12 · touch-таргеты и скринридер | «44px, screen reader support» | `mobile-hud.css` — 273 строки адаптива, но размеры кнопок не замерены; `aria-live` выключен (см. выше) | Не доказано ни в одну сторону | низкая |
| K12 · тесты как защита | «screenshots обязательны для canvas» (game-playtest) | 1234 теста зелёные, при этом 46 звуковых файлов отсутствуют — тестов на существование файлов нет | Зелёный CI маскирует тишину в бою | высокая |

## 2. Инвентарь графики и звука

**Типы (318 файлов в `assets/` + `audio/` + `audio-lab/`):** 101 `.png`, 83 `.glb`, 60 `.webp`, 46 `.ogg`, 5 `.gltf`, 11 `.md`, 4 `.json`, 2 `.js`, 2 `.txt`, 1 `.html`, 1 `.mjs`, 1 `.svg`, 1 `.jpg`.

**Объём `assets/` = 113,5 МБ:** `audio` 44,3 · `space` 37,4 · `prologue` 18,6 · `3d` 12,4 · `fx` 0,5 · `disruptor-ships` 0,1 · `ninja` 0,1 · `kenney` 0 · `upgrade` ≈0.

| Класс | Количество | Используется | Мёртвый груз |
|---|---|---|---|
| Музыка `.ogg` | 12 (44,3 МБ) | все 12 в `music-playlist.js:6-19` | нет |
| Станционные SFX | 29 (`audio-lab/assets/sfx/base/`) | да, `heritage.js:143-155` | нет |
| Прочие SFX | 5 (`hit`,`armor_hit`,`pickup`,`explosion_small`,`starter_pulse`) | частично | `starter_pulse` — 0 ссылок в аркаде |
| Kenney SFX (манифест) | 46 имён в `sound-map.js:81-94` | **0 файлов на диске** | весь боевой слой — дырка, не груз |
| Studio SFX (манифест) | 21 ссылка | 0 файлов | мёртвые ссылки в манифесте |
| 3D (`assets/3d`) | 91 файл / 88 меш-ов, 12,4 МБ | только `hybrid-3d.js:125-126`, `prologue-robot-3d.js:48` | в аркаде 0 прямых ссылок |
| `assets/prologue` | 40 файлов, 18,6 МБ | 11 упоминаний (пролог/архивный режим) | вне аркады — проверить принадлежность |
| `assets/space` | 73 файла, 37,4 МБ | 62 ссылки | нет |
| `assets/upgrade` | 32 файла, ≈0 МБ | 13 ссылок | нет |
| `assets/ninja` | 8 файлов | 0 ссылок | **мёртвый груз** |
| `assets/kenney` (визуал) | 4 файла | 0 ссылок | **мёртвый груз** |
| `assets/disruptor-ships` | 2 файла | 2 ссылки (вне аркады) | проверить |
| Картинки `assets` | 161 (`png`+`webp`) | 88 упомянуты по имени | **73 не упомянуты (≈45 %)** |
| UI-киты `arcade/studio` | `hud-kit` 25, `menu-kit` 25, `assets` 15 | 13 / 6 / 14 ссылок | проверить поштучно |

## 3. Где HUD перегружен и показывает служебное

1. `index.html:118` — `FPS · ЦЕЛЕЙ` в живом HUD на desktop (скрыто лишь на тач-устройствах, `hud-art.css:242`). Кандидат №1 на удаление или перенос в панель разработчика.
2. `hud.js:94` — `fps`/`enemies` переписываются каждый кадр, то есть строка живёт постоянно, а не как диагностика по запросу.
3. Правый «боевой контур» (`index.html:92-119`) содержит 9 индикаторов, из них 4 дублируют друг друга (дроны ×2, коды ×2, удар ×2, рывок ×2) — при 9 элементах в колонке ~240 px.
4. `index.html:60` + `:82` — два отсчёта рывка одновременно (`hud.js:168` пишет один текст в два места).
5. `hud.js:192-198` и `:212-220` — две строки состояния, до 12 ветвей каждая; на desktop в `flightMode` попадают длинные формулы вида `АВТООГОНЬ · ВЫКЛЮЧЕН`, `ДОК · РЕМОНТ +60 %`.
6. `index.html:114-116` — «БАЗЫ / ВЕДОМЫЕ / КОДЫ» служебно-счётчикового вида (в `hud-art.css:112` уже скрыты, но в разметке и `hud.js:91-93` продолжают обновляться каждый кадр).
7. `hud.css:70` — `telemetry-foot` 10 px: мелкий текст, контраст не измерен.

## 4. Не проверено

- Визуальные скриншоты (game-playtest требует их для canvas): HUD-вес и перекрытие игрового поля **не измерены**, только по разметке.
- Контраст текста (WCAG 4.5:1) и размеры touch-таргетов 44 px — не замерялись.
- Полный прогон 4 skills-руководств: прочитаны только шапки/принципы, не все разделы.
- `assets/prologue`, `assets/disruptor-ships`, `arcade/studio/{assets,menu-kit}` — поштучная сверка использования не сделана.
- Судьба `docs/AUDIT-MENU*` выводов после последних правок не перепроверялась (принято как факт).
- Реальное звучание после исправления ассетов (громкости/баланс) — не проверялось, только наличие файлов.

## 5. План (приоритет → эффект → критерий приёмки → файлы)

1. **P0 — вернуть боевой звук.** Положить в `audio-lab/assets/sfx/kenney/` 46 файлов ровно с именами из `sound-map.js:81-94` (Kenney Sci-Fi/Impact, CC0) и удалить 21 мёртвую studio-ссылку из `sound-map.js:12-24`. Эффект: плазма, рельса, ион, картечь, ракета, взрывы, урон, смерть, level-up, босс снова звучат. Приёмка: новый тест `arcade/tests/sound-assets.test.mjs` проверяет существование каждого `sampleUrl`/`sampleUrls`/`sampleLayerUrls` (0 отсутствующих), `#fail 0`. Файлы: `audio-lab/sound-map.js`, `audio-lab/assets/sfx/kenney/*`, `audio-lab/assets/sfx/studio/SOURCES.md`, новый тест.
2. **P0 — убрать FPS из живого HUD.** `index.html:118` → перенос в панель разработчика/`development-panel.js`, счётчик целей оставить как игровую величину. Эффект: минус строка служебного текста на desktop. Приёмка: в DOM живого HUD нет `#fps`, `hud.js` не пишет `fps`; тест `hud.test.mjs` проверяет отсутствие узла. Файлы: `index.html`, `hud.js`, `hud.css`/`hud-art.css`, `development-panel.js`.
3. **P1 — снять дубли приборов.** Оставить по одному представлению: рывок — `leftBurstText` (убрать `bursttext`), дроны — иконки + `0/2` (убрать `ВЕДОМЫЕ`), коды/удар — только слоты (убрать `hack`, `hudStrikeCount` или наоборот). Эффект: правый контур с 9 до 6 элементов, меньше мигания при обновлении. Приёмка: `hud.js` пишет каждую величину в один узел (grep-тест на дубли id), скриншот правого контура до/после. Файлы: `index.html`, `hud.js`, `hud-art.css`, `mobile-hud.css`.
4. **P1 — слить `stateText` и `flightMode` в один приоритетный канал.** Один список приоритетов вместо двух машин состояний; длинные пояснения — в `aria-label`/tooltip. Эффект: игрок читает одно сообщение. Приёмка: одна функция `systemStatus(game)` и тест на 12 сценариев (авария, ремонт, добыча, форсаж, база, overcharge, scan, доктрина). Файлы: `hud.js`, `hud.css:94` (critical-alert), `arcade/tests/hud.test.mjs`.
5. **P1 — доступность HUD.** (а) форма контактов радара различается не только цветом (враг — треугольник, союзник — круг, станция — ромб), `hud.js:396-415`; (б) `aria-live="polite"` для смены цели/угрозы (`index.html:123`) с троттлингом; (в) минимальный кегль 11-12 px для `telemetry-foot`-подобных строк. Приёмка: тест на форму контактов + прогон скринридера, `#fail 0`. Файлы: `hud.js`, `index.html`, `hud.css`.
6. **P2 — сжать каскад меню.** Пять слоёв → три: базовый `menu-system.css` + один тематический `station-art.css`, `menu-rebuild.css` влить (устранить 23 продублированных селектора). Эффект: предсказуемая правка, минус ~300-400 строк мёртвого override. Приёмка: скрипт-проверка «нет селектора, объявленного в двух слоях» = 0; визуальный прогон 5 экранов станции. Файлы: `menu-system.css`, `menu-rebuild.css`, `all-menus.css`, `station-art.css`, `station-terminal-art.css`, `index.html:13-19`.
7. **P2 — разложить `base-panel.js`.** Вынести `contractPanel`, `corridorPanel`, `probePanel`, `firstContractRewardPanel` в `base-panels/*.js` с чистыми входами. Эффект: правка одного экрана не трогает 1000 строк. Приёмка: `base-panel.js` ≤ 400 строк, все существующие тесты зелёные, экспорт API не меняется. Файлы: `arcade/base-panel.js`, новые модули, `arcade/tests/*panel*.test.mjs`.
8. **P2 — вычистить мёртвые ассеты.** `assets/ninja` (8 файлов), `assets/kenney` (4), 73 неиспользуемых png/webp, `starter_pulse.ogg` без ссылок; `assets/3d` (12,4 МБ) и `assets/prologue` (18,6 МБ) — проверить принадлежность прологу и вынести из публикации аркады при подтверждении. Эффект: минус ~30 МБ трафика и минус ложные надежды при поиске ассетов. Приёмка: отчёт-скрипт «файл без ссылок = 0» для аркадных путей, сборка Pages без 404. Файлы: `assets/*`, `.github/workflows/publish-pages.yml`, `arcade/ASSET-REUSE-MAP.md`.
9. **P3 — регрессия подачи в CI.** Скриншот-прогон HUD и меню (game-playtest: screenshot обязателен для canvas), проверка «нет диагностики в HUD», проверка существования всех аудио-URL. Эффект: тишина и служебный текст больше не проходят CI. Приёмка: job в Actions падает на отсутствующем ogg. Файлы: `arcade/tools/*`, `.github/workflows/publish-pages.yml`.

### CC0/лицензионные библиотеки — что уместно и под какой лицензией

| Библиотека | Что брать | Лицензия | Как применять здесь |
|---|---|---|---|
| **Kenney** (kenney.nl) | Sci-Fi Sounds, Impact Sounds, UI Audio, Space Kit | **CC0 1.0** — атрибуция не требуется | Основной путь для P0: имена в `sound-map.js:81-94` (`impactMetal_medium_000`, `laserLarge_000`, `forceField_000`, `explosionCrunch_003`, `spaceEngineLow_000`) — это уже номенклатура Kenney; положить файлы, лицензию — в `audio-lab/assets/sfx/kenney/SOURCES.md` |
| **OpenGameArt** | только записи с явной меткой CC0 (часть звуков и тайлов) | смешанные лицензии: CC0 / CC BY 3.0-4.0 / GPL — брать **только CC0 или CC BY с атрибуцией** | Точечно, если Kenney не покрывает нужный «тяжёлый» удар; каждый файл — строка в `CREDITS.md` |
| **Free Music Archive** | фоновая музыка | в основном **CC BY / CC BY-SA / CC BY-NC** — CC0 там редкость | Только CC BY (с атрибуцией) и никогда NC/SA в коммерческой сборке; текущий каталог (Scott Buckley, **CC BY 4.0**) уже соответствует — расширять в том же формате |
| **freesound.org** | фоли-записи ударов/двигателей | CC0 (фильтр) / CC BY | Допустимо с фильтром CC0; предпочтительнее Kenney как единый стиль |

Правило приёмки для любых новых ассетов: файл + лицензия + источник в `CREDITS.md`/`SOURCES.md` и в панели Credits (`game.js:767-775`, `creditsDialog`).

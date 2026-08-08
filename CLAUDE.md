# Инструкции для всех репозиториев

> Kimi → `AGENTS.md`, Hermes → Aisystant MCP `get_instructions`; доставка ядра из авторского IWE в этот шаблон — внутренний авторский конвейер (не входит в шаблон, недоступен пользователю). Slim-ядро: триггеры + правила hot; детали → `memory/`, `.claude/rules-lazy/`, `.claude/skills/`.

## 1. Архитектура репозиториев

| Тип | Что содержит | Первоисточник |
|-----|-------------|---------------|
| **Base** (Принципы + Форматы) | ZP, FPF, SPF, FMT-* | Да (платформа) |
| **Pack** | Паспорт предметной области | Да (пользователь) |
| **DS** (instrument/governance/surface) | Код, планы, курсы | Нет (производное от Pack) |

**Fallback Chain:** DS → Pack → Base (SPF → FPF → ZP)
**Pack = source-of-truth для доменного знания. DS меняется вслед за Pack.**
Детали типов, именование, измерения: → `memory/repo-type-rules.md`

**Pack Creation Gate:** хочешь создать Pack → `/pack-new`. Структура Pack = `SPF/pack-template/`. Процесс = `SPF/process/01-11`. Имя = существительное-домен (не тема, не инструмент). Если `FPF/` или `SPF/` отсутствуют в рабочей директории — `/pack-new` клонирует их автоматически.

## 2. ОРЗ-фрактал (Открытие → Работа → Закрытие)

> Пропуск Открытия = незапланированная работа. Пропуск Закрытия = незафиксированный результат.
> **Сессия:** `memory/protocol-open.md § Сессия` → `memory/protocol-work.md` → `/run-protocol close` · **День:** `/day-open` («открывай») → `/run-protocol day-close` · **Неделя:** `/run-protocol week-close` · **Месяц:** `/month-close` (первый Пн, до Strategy Session).

| Масштаб | Открытие | Работа | Закрытие |
|---------|----------|--------|----------|
| **Сессия** | `protocol-open.md § Сессия` (любое задание) | `protocol-work.md` | `/run-protocol close` |
| **День** | `/day-open` («открывай») | Между Day Open и Day Close | `/run-protocol day-close` |
| **Неделя** | — | — | `/run-protocol week-close` |
| **Месяц** | — | Между Month Close предыдущего и текущего | `/month-close` (первый Пн месяца, до Strategy Session) |

> **SoT (WP-272 Ф1):** `PACK-agent-rules/rules/AR.NNN.md` (реестр `.claude/rules-registry.yaml`) — авторский источник, не шипится в шаблон (генератор `.claude/scripts/generate-rules-registry.py` требует `PACK-agent-rules`, которого у пользователя нет). На пользовательской установке — шипящаяся выжимка тех же 10 правил → `.claude/rules-lazy/blocking-rules-full.md`. **Приоритет = нумерация:** структурное (1-5) перевешивает поведенческое (6-10).

> **Source-of-truth (WP-272 Ф1, 26 апр):** правила формализованы в `PACK-agent-rules/rules/AR.NNN.md` с frontmatter (id, type, priority, triggers, tests, hook). Реестр генерируется в `.claude/rules-registry.yaml` через `python3 .claude/scripts/generate-rules-registry.py`. Диспатчер: `.claude/hooks/rule-engine.sh`. Ниже — горячая выжимка top правил для агента; полный текст — в Pack.
>
> **Иерархия при конфликте:** правила нумерованы по приоритету (= AR-priority в Pack).
> Структурное (1-5) ВСЕГДА перевешивает поведенческое (6-10). Структурное = «без них работы нет». Поведенческое = «как себя вести внутри уже согласованной работы».
> Пример (C-001 в conflicts.md): WP Gate priority=1, Автономность priority=6. При конфликте AR.001 выигрывает.

1. **WP Gate:** ЛЮБОЕ задание → протокол Открытия → ДО начала работы.
   **Ритуал согласования (горячий, не lazy):** при создании нового РП (нет в плане недели) Claude обязан:
   - Шаг 1. Объявить: Роль пользователя · Роль Claude · Работа · РП (артефакт) · Режим ТВС (текущее/важное/срочное — конвейерная модель; «срочное» только при угрозе остановки конвейера, не по дедлайну/«горит») · Класс верификации (trivial/closed-loop/open-loop/problem-framing) · Метод · Оценка ~Xh · Модель.
   - Шаг 2. **Дождаться согласования.** Без явного «да»/«делаем»/«открывай» от пользователя — НЕ регистрировать РП в 4 местах (REGISTRY/WeekPlan/context/Linear). Это исключение из Правила 7 (Автономность).
   - Шаг 3-4. См. `memory/protocol-open.md` (детали).
2. **Push:** «заливай» / «запуши» / «закрывай» → commit + push без доп. вопросов. Push ДО отчёта Закрытия. **При любом Close-протоколе (Quick/Day/Week):** `git status --short` по ВСЕМ репо сессии — незафиксированные изменения → commit + push ДО перехода к следующему шагу протокола.
3. **Close:** Триггер Закрытия → протокол Закрытия → выполнить.
4. **Pull-on-Touch:** PreToolUse-хук автоматически делает best-effort `git pull --rebase --autostash` один раз на репо. Не выполняй ручной `cd && git pull`; при сбое хук сообщает `potentially stale`, процедура → `memory/reference/agent-core.md`.
5. **Чеклист-верификация:** Quick/Day Close — sub-agent Haiku R23 сверяет с чеклистом. Исключения: ≤15 мин или без изменений файлов.
6. **Hooks/Scripts Bypass Gate (S-33):** без явного разрешения не менять `.claude/hooks|scripts/`, `.iwe-runtime/`, `FMT-exocortex-template/`, не обходить хуки; блок хука → bug-файл + пилоту + ждать. → `.claude/rules-lazy/hooks-bypass-gate.md`.
7. **Автономность:** не спрашивать подтверждения — выполни → отчитайся. Исключения: необратимо-разрушительное; WP Gate Ритуал; choice-question. Полный текст → `.claude/rules-lazy/blocking-rules-full.md` п.7.
8. **Напоминания (S-44):** «напомни через X» → `send_telegram_message(schedule_at)` + ScheduleWakeup резерв; резерв сработал → сначала Telegram, потом чат.
9. **Финиш > отлог (S-46):** доп. задача → дефолт «делаю сейчас»; «сейчас или потом?» = анти-паттерн. Исключения и приоритет WP Gate → `.claude/rules-lazy/blocking-rules-full.md` п.9.

### Протокол Работы (полный → `memory/protocol-work.md`)

**Capture-to-Pack** — на каждом рубеже: есть ли знание для записи? Анонсировать: *«Capture: [что] → [куда]»*. Маршрутизация: правило (1-3 строки) → CLAUDE.md, доменное → Pack, реализационное → DS docs/, урок → memory/. Capture-to-Pack — shortcut внутри маршрутизации знаний, не замена Routing Gate. При создании нового артефакта Routing Gate (DP.KR.001 §5) проверяется первым.
**Self-correction:** расхождение → немедленно предложить фикс (файл, строка, что изменить). Применяется только внутри scope текущего хода: файлы/директории из agenda хода (проверяется `git diff HEAD`). За пределами scope — Drift Reporting (Agent Core SYNC-CORE): отчитаться пилоту, не фиксить.

### Pre-action Gates

| Момент | Проверка |
|--------|---------|
| Начало работы | Какие сервисы (MAP.002) затронуты? |
| Пользовательский сценарий | **SC Gate:** какое обещание (08-service-clauses/) затронуто? |
| Первое содержательное действие в репо (Read файла, Edit, ответ о структуре, commit) | **Repo-Touch Gate:** прочитать `<repo>/CLAUDE.md`. Если содержит блок «обязательно загружай» — загрузить указанные файлы ДО ответа. |
| Архитектурное решение | **АрхГейт** → `/archgate` |
| РП затрагивает PII (email, telegram_id, ЦД, tokens, user_events) | **Security Gate (B7.3):** ответить на §Б чеклист ArchGate ДО реализации. Логирование PII = блокер. |
| РП ≥3h | **Priority Gate:** к какому R{N} ведёт? |
| Новый инструмент/агент/система | **IntegrationGate (БЛОКИРУЮЩЕЕ):** проектирование ТОЛЬКО в последовательности — (1) обещание → (2) сценарии → (3) роль → (4) реализация. См. ниже явный чеклист. Прыжок сразу в реализацию = P10 (DP.FM.010). |
| Замена legacy-компонента (миграция из LMS/внешней системы) | **LegacyPortGate (БЛОКИРУЮЩЕЕ):** сначала 15-мин субагент: «как это работает сейчас?» (cron/API/merchant/токены). Решение портирование vs новый дизайн — ТОЛЬКО после ответа. Прыжок в «новый дизайн» без проверки = DP.FM.014 (Legacy Port Jump). См. `memory/feedback_behaviour.md` Правило 10. |

### IntegrationGate — явный чеклист (БЛОКИРУЮЩЕЕ)


- Начало работы → какие сервисы (MAP.002) затронуты?
- Нетривиальное действие/РП → **State-Transition Gate (WP-457):** `{тип состояния, из→в}`, только `gate_ready: true` → Agent Core ниже.
- Пользовательский сценарий → **SC Gate:** какое обещание (08-service-clauses/) затронуто?
- Создание/размещение артефакта → **Routing Gate:** карта DP.KR.001 §5; «по аналогии с соседним» запрещено.
- Первое содержательное действие в репо → **Repo-Touch Gate:** прочитать `<repo>/CLAUDE.md`; блок «обязательно загружай» → загрузить ДО ответа.
- Архитектурное решение → **АрхГейт** `/archgate`.
- РП затрагивает PII → **Security Gate (B7.3):** §Б чеклист ArchGate ДО реализации; логирование PII = блокер.
- РП ≥3h → **Priority Gate:** к какому R{N} ведёт?
- Новый инструмент/агент/система → **IntegrationGate (БЛОКИРУЮЩЕЕ):** только (1) обещание → (2) сценарии → (3) роль → (4) реализация → `.claude/rules-lazy/integration-gate.md`.
- Замена legacy-компонента → **LegacyPortGate (БЛОКИРУЮЩЕЕ):** сначала 15-мин субагент «как это работает сейчас?» → `.claude/rules-lazy/blocking-rules-full.md`.

## 3. Описания методов (PROCESSES.md)

≤15 мин — не нужен. Внутри системы — `<repo>/PROCESSES.md`. Новая система — сценарий + процессы + данные.

## 4. Memory (Слой 3)

| Ситуация | Читай |
|----------|-------|
| Файлы/репо | `memory/navigation.md` |
| Pack-репо | `memory/repo-type-rules.md` |
| Терминология | `memory/hard-distinctions.md` |
| FPF/SOTA/Роли | `memory/fpf-reference.md`, `memory/sota-reference.md`, `memory/roles.md` |
| Документ/чеклист | `memory/checklists.md` |

Политика: ≤11 файлов; построчно проверяется только distinctions.md (≤150), остальное — суммарным M1/M2-бюджетом (WP-7 NR1.2); lazy-reference без лимита. Горизонты/frontmatter → `memory/memory-lifecycle-spec.md`; temporal metadata → `memory/protocol-work.md §2`.
Рабочая директория: `{{HOME_DIR}}/IWE/`; `memory/` = симлинк на auto-memory.

## 5. АрхГейт — ОБЯЗАТЕЛЬНАЯ оценка

> **БЛОКИРУЮЩЕЕ.** Архитектурное решение → `/archgate`: принципы DP.ARCH.001 §7 → профиль ЭМОГССБ (✅/⚠️/❌) → conjunctive screening; чеклист современности (SOTA.002/001/011 + CGUS/PUA) — внутри `.claude/skills/archgate/SKILL.md`. Профиль без агрегатного балла — так и есть, это осознанный выбор (conjunctive screening, не средневзвешенное).

## 6. Форматирование → `.claude/rules/formatting.md` · Различения → `.claude/rules/distinctions.md`

## 6. Форматирование → `.claude/rules/formatting.md`

Hot-каркас ≤20K токенов (M1), строгая цель ≤12K (M2). Изменил файл из `hot-files.list` (оба CLAUDE.md, rules/*.md) → перед коммитом `{{IWE_TEMPLATE}}/scripts/verify-context-budget.sh`.

## 7. Обновление этого файла

> **3 слоя:** L1 (§1-§7) = платформа (`update.sh`). L2 (§8) = staging. L3 (§9) = авторское.

- Протоколы → `memory/protocol-*.md`
- Различение (1-3 строки) → `.claude/rules/distinctions.md`
- Форматирование → `.claude/rules/formatting.md`
- Стабильные знания → `memory/*.md`
- Свои правила → §8 (staging) или §9 (авторское)

<!-- PLATFORM-END -->

---

## Agent Core (SYNC-CORE → AGENTS.md)

> **WP-394 Ф4.2.** Единое ядро для всех агентов (Claude, Kimi, Codex, Hermes). `AGENTS.md` генерируется отсюда скриптом `scripts/sync-agent-instructions.sh` — **не редактировать `AGENTS.md` вручную**. Элаборация → `memory/reference/agent-core.md`.

<!-- SYNC-CORE-START -->

## WP Gate — CRITICAL

**ЛЮБОЕ задание → протокол Открытия → ДО начала работы.** При создании нового РП: объявить роль, работу, РП, класс верификации, метод, оценку, модель. Дождаться согласования пилота.

## Git Staging — CRITICAL

**Перед любым нетривиальным действием или РП назвать целевой переход состояния пользователя** `{тип состояния, из→в}` (WP-457) — **применимо, если в `{{GOVERNANCE_REPO}}/docs/state-axes-registry.yaml` описаны оси состояний** (авторский артефакт, не шипится в шаблон по умолчанию). Если файл есть — типы только из него, допустимы только `gate_ready: true`; ссылка на declared FSM-owner обязательна, свободный текст не принимается; нет ссылки или тип не `gate_ready` → действие = inventory → СТОП/отложить. **Файла нет (типовая установка)** → гейт неактивен, действовать по остальным Pre-action Gates без остановки. Модель осей (авторский пример) → `archive/wp-contexts/WP-457/CONCEPT-user-states.md §5`; cross-axis → `memory/reference/agent-core.md`.

These commands pick up staged/unstaged changes from OTHER agents (Claude Code works in the same repo simultaneously). Wrong attribution and accidental commits of other agents' work result.

**Always stage only specific files you edited:**
```bash
# Correct
git add path/to/specific-file.md

# FORBIDDEN — captures other agents' work
git add -u
git add .
git add -A
```

**Before every commit: verify staged scope.**
Run `git diff --cached --name-only` and confirm that all staged files belong to the current session's WP/context.
If unexpected files appear — `git restore --staged <file>` before committing.

## Artifact Naming

**Do not invent artifact names.** Names for sections, documents, RPs, and deliverables must come from the plan/task you received. If the task is silent on a name — report "need clarification on name" instead of making one up.

## Drift Reporting

If you discover a discrepancy (file doesn't match plan, stale content, inconsistency):
- **Report to pilot, do not silently fix.**
- Format: "Found drift: [what is inconsistent] in [file]. Should I fix it?"
- Only fix if explicitly instructed.

## Working Directory

`{{WORKSPACE_DIR}}/`

## Status Reporting — Agent Status Registry (РП-395)

**Primary (обязательно):** в начале задачи `agent_status_update(agent=<claude-code|kimi|codex|hermes>, status=working, task=<кратко>, files=[...])`; по завершении — `status=idle`. Статусы: `idle|working|peer-session|blocked`; пилот видит всех через `agent_status_list`. Командный режим (`repo=`) и fail-safe скрипт → `memory/reference/agent-core.md`.

## Long Operation Protocol — 180 s Silence Threshold

**Не молчи больше 180 секунд.** Операция >180с → ДО запуска сообщить: что запускается, длительность, шаг X из Y, id фоновой задачи. >180с тишины → микро-отчёт «Всё ещё работаю. Текущий шаг: [X из Y]. Следующий: [Z].» Касается всего, где пилот видит пустое «Thinking» (bash, subagent, фоновые задачи, Close-протоколы).
## WP-REGISTRY Naming — CRITICAL

**Колонка «Название» в WP-REGISTRY содержит ТОЛЬКО имя артефакта ≤80 символов.**

Запрещено в колонке «Название»: даты закрытия, ссылки на peer-сессии, метрики фаз, SHA коммитов, результаты проверок, количество тестов, и любые другие служебные данные.

- ✅ `~~Алгоритм диагностики~~`
- ❌ `~~Алгоритм диагностики~~ — closed 30 мая (PHASE1=5, MANDATORY=5...)`

**Куда писать:**
- Итог закрытия РП → раздел `## Закрытие` в `archive/wp-contexts/WP-NNN-*.md`
- Текущие фазы и прогресс → frontmatter поля `phases`/`progress` в `inbox/WP-NNN.md`

**При начале работы с РП:** прочитать `inbox/WP-NNN.md`. При изменении статуса фаз → обновить frontmatter карточки, НЕ имя реестра.

## WP Context Scope — Umbrella РП

Для зонтичных (umbrella) РП с `agent_scope: open-only` в frontmatter:
- Читать **только** фазы со статусом `pending` / `in_progress` / `blocked`
- Архивные (`done`, `closed`, `defer`) — **не читать** без явного запроса пользователя
- Исключение: если пользователь даёт задание с указанием конкретной архивной фазы

Применяется к: WP-5, WP-7.

## Calendar Events — CRITICAL

**All platform reminders and calendar events created by the agent must be scheduled BEFORE 09:00 AM.**

This includes: task reminders, follow-up events, template migration tasks, any agent-generated calendar entries.

**Never** schedule agent-created events at or after 09:00 without explicit pilot approval.

If an event is created after 09:00 by mistake:
1. Delete the incorrect event immediately
2. Recreate it before 09:00 on the same day, or on the next available pre-09:00 slot
3. Report the error to the pilot

## Language

Respond in Russian unless the user writes in English.

## Response Style — Pilot-Facing

Агент должен применять правила понятного ответа пилоту (полный текст — `memory/feedback_response_clarity_for_pilot.md`, HOT) в ответах чата, синтезе отчётов и пост-отчётах после действий.

**Channel detector:** технический стиль — для стенограмм ходов peer-сессий, commit-сообщений, PR; режим «на пальцах» — для чата с пилотом (если пилот сам не пишет `grep`/`git`/пути/SHA) и для §1-§4 синтеза report.md.

**Eleven rules (A1-A11), short:** A1 путь файла не подлежащее (только в скобках после русского глагола); A2 английский термин только после русского описания в скобках; A3 первое упоминание колонки/функции — расшифровка одним словом; A4 pre-flight: примет ли пилот решение по этой фразе; A5 ЧТО до КАК; A6 одна стрелка-следствие на предложение; A7 «сделал → эффект», `<details>` — только при наличии нужных пилоту деталей или по его явному запросу; A7.1 журнал (SHA, коммиты, дефекты) — только в файл отчёта, не в чат; A8 журнал процесса по умолчанию не писать; A9 channel detector; A10 английские маркеры статуса (exit/PASS/SHA) → русские слова; A11 активный залог на ошибках и находках.

## Code Style — Engineering (DP.SC.172)


**P-правила, short:** P0 перед коммитом — форматтер+линтер репо (механику закрывает инструмент); P1 тест без проверки наблюдаемого результата запрещён (`assert True` — запах); P2 третье повторение → функция, не `locals()[str]`; P3 мёртвую ветку/enum удалять, не «для совместимости»; P4 `except: pass` без логирования запрещён; P5 длинную функцию со смешанными обязанностями / булевы флаги-режимы — разбить. Граница: жёсткие запреты (`git add -A`, секреты) — в PACK-agent-rules (AR.*), не здесь. У Claude правила приходят хуком (`inject-code-style.sh`); детектор-страховка `code-style-hook.sh` пишет P1/P2/P4 в единый лог стиля.

<!-- SYNC-CORE-END -->

---

## 8. Staging (обкатка → шаблон)

> Правила на обкатке. Работают → переносятся в шаблон (L1).
> **Перенесено в L1 (20 мар):** SC Gate, межсистемные процессы, чеклист-верификация.
> **Промотировано в FMT (20 апр):** S-13 (именование РП = существительное-артефакт), S-14 (синхронизация REGISTRY→производные).

### Staging-канал (my IWE → FMT-exocortex-template)

- **S-45 Agent Inbox** (WP-324, 17 мая, расширено session 6) — `inbox/agent/` структура + 5 templates + SPEC + DP.SC.135 + DP.ROLE.045 + `iwe-agent-dispatcher.py` (headless `claude -p`, обход CCR v1→v2 bug). Промотировано в FMT `extensions/agent-inbox/` + `pack-templates/digital-platform/`. Status: testing (полная end-to-end automation smoke на расписании — defer-ред: требует Nix systemd unit или cron).

**Правило добавления:** новое поведение в §9 (авторское) → ОДНОВРЕМЕННО строка в STAGING.md (`status: testing`).

**Промоция (при Week Close):**
1. Просмотреть STAGING.md → есть `validated`?
2. Убрать авторские константы → заменить на `{{PLACEHOLDER}}`
3. Перенести в `FMT-exocortex-template` + commit `feat: promote S-NN from staging`
4. Обновить STAGING.md: статус → `promoted`

**Отклонение:** специфичное для авторского окружения → статус `rejected` (остаётся навсегда в §9, не промотируется). Не удалять из таблицы — это решение.

---

## 9. Авторское (только мой IWE)

> Элаборации всех пунктов → `memory/reference/agent-core.md`.

- **Без Obsidian (DS-strategy):** просмотр через VS Code.
- **Комментарии кода — только EN (решение Андрея, 14.06.2026):** весь `{{WORKSPACE_DIR}}/**`; исключение — user-facing строки по языку интерфейса.
- **Различения (авторские):** `memory/distinctions-warm.md` и `extensions/`; `.claude/rules/distinctions.md` — поставляемый платформенный hot-слой, пользовательские правки допустимы только внутри явного `USER-SPACE` блока.
- **Именование:** governance-репо называется `DS-strategy` по умолчанию; рабочая директория `{{HOME_DIR}}/IWE/`.
- **Extensions Gate (БЛОКИРУЮЩЕЕ):** пользователи кастомизируют ТОЛЬКО `extensions/*.md` + `params.yaml` (правка `.claude/skills/` или `memory/protocol-*.md` = ошибка); автор (`author_mode: true`) редактирует L1 напрямую — авторский IWE = SoT, доставка в FMT через `template-sync.sh`.
- **README.md (FMT-exocortex-template):** изменение структуры — по согласованию с владельцем.
- **WP Entry Filter (S-47, БЛОКИРУЮЩЕЕ):** новый РП — только при явной связи с R1-R6 месяца или внешнем заказчике; иначе → `inbox/backlog-with-triggers.md`. Исключения: spin-off закрытого РП; прямое поручение пилота.
- **Именование РП:** существительное-артефакт, только русский (Pack-ID допустим); реестр ≤80 символов → SYNC-CORE; переименование — синхронно REGISTRY+MEMORY.md+WeekPlan+DayPlan+WP-context.
- **Память (S-35):** новые `memory/*.md` — обязательный frontmatter; шаблон и горизонты → `memory/memory-lifecycle-spec.md` (единственный источник).
- **Security Audit Cadence (WP-212, S-36):** per-ArchGate (§Б B7.1 + STRIDE) · Week Close (`security-posture.md §3`) · Daily (tsekh-1) · Month Close (VR.R.002).
- **WeekPlan/WeekReport split (WP-297):** WeekPlan = только интенты, WeekReport = только факты; при создании WeekPlan итоги уезжают в WeekReport.
- **Режим «на пальцах» (S-37):** триггеры «объясни», «на пальцах», «что сделали», «простыми словами» → Response Style + `memory/feedback_response_clarity_for_pilot.md`.
- **Календарный конвейер (WP-357):** SoT — `DS-strategy/calendar/process-catalog.yaml` (+ derived `date-ledger.yaml`, не редактировать); новый процесс = каталог + plist; спецификация → `docs/calendar-pipeline.md`.

---

*Последнее обновление: 2026-05-26*

# pipeline

Движок бизнес-инкубатора: исполнимые бизнес-процессы с AI.
Домен → обследование (1+6 / IDEF0) → план-граф → исполнение с контролем.

> Статус: ИССЛЕДОВАНИЕ (research). Автономный форк ai-agent-порта Luck.
> Нативные проекты (luck-репо Python, luck-repo Rust, ai-agent) НЕ модифицируются.

## Возможности

- Компилятор .luck → план (валидатор: циклы, ghost-узлы, VERIFY-предикаты, BRANCH-покрытие)
- Планировщик: топо-порядок, ветвления (CLASSIFY→BRANCH), VERIFY, MERGE, REJECT как состояние
- VERIFY-предикаты (11): grep/contains/file_exists/not_empty + бизнес: stock_level,
  order_match, cash_ok, shelf_life_ok, temp_log_ok, credit_ok (JSON-значения)
- Маппер IDEF0 → Luck (агент-BPWin): функциональные блоки ICOM → исполнимый план
- Рантаймы: OpenRouter (nemotron free) → Ollama (GPU) с фоллбэком
- CLI: validate (валидация), run (живой прогон), web (веб-интерфейс)

## Быстрый старт

```bash
cargo test                                    # 41 тест
cargo run --bin validate -- examples/horeca-daily-cycle.luck

# Живой прогон (Ollama на десктопе, GPU)
OLLAMA_HOST=http://100.64.0.1:11434 OLLAMA_MODEL=hermes3:8b \
OLLAMA_ONLY=1 cargo run --bin run -- examples/horeca-daily-cycle.luck

# Веб-интерфейс (http://localhost:8080)
OLLAMA_HOST=http://100.64.0.1:11434 OLLAMA_MODEL=hermes3:8b \
OLLAMA_ONLY=1 cargo run --bin web -- 8080
```

## Структура

```
├── src/
│   ├── luck_plan.rs       # типы плана + валидатор
│   ├── luck_compile.rs    # парсер .luck → Plan
│   ├── luck_scheduler.rs  # PlanExecutor: VERIFY/BRANCH/MERGE/REJECT + предикаты
│   ├── idef0.rs           # маппер IDEF0 (ICOM-блоки) → Luck-план
│   ├── openrouter.rs      # рантаймы: OpenRouter → Ollama (фоллбэк)
│   └── bin/               # validate, run, web
├── examples/              # сценарии HoReCa (.luck) + IDEF0-модель (.json)
├── tests/e2e_horeca.rs    # E2E: исполнение сценариев (include_str!)
└── docs/                  # TECH.md, AGENT-BPWIN.md (мета-технология)
```

## Примеры (HoReCa)

Дневной цикл, возвраты, инвентаризация, деньги — исполняются вживую на GPU.
Описание домена и результаты: https://github.com/tester-bcs/horeca-pilot (пилот).

## Конвенции

- Синтаксис .luck и питфоллы: скилл luck-language
- VERIFY subject — внутри слота: `VERIFY not_empty INTO picked_order`
- Ветвление — отдельный узел `fork: BRANCH` (CLASSIFY сам ветки не выбирает)
- JSON (для VERIFY) и метка (для BRANCH) — разные узлы

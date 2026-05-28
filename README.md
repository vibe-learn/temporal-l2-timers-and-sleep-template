        # temporal — Таймеры и Sleep

        Homework-шаблон для урока **l2_timers_and_sleep** (Таймеры и Sleep) на платформе Vibe Learn.

        ## Что делать

        На go.temporal.io/sdk реализуй ApprovalWorkflow: ждёт сигнал "approval" не дольше дедлайна
(workflow.NewTimer + workflow.NewSelector); по таймеру — авто-reject, по сигналу — решение
из payload. Затем реализуй PollLoopWorkflow с циклом активности Poll + workflow.Sleep и
обрезкой истории через workflow.NewContinueAsNewError по достижении порога итераций. Тесты
на TestWorkflowEnvironment: промотка виртуального времени проверяет срабатывание таймера и
авто-reject; отправка сигнала через SignalWorkflow до дедлайна даёт решение; цикл вызывает
ContinueAsNew с перенесённым состоянием.

## Контекст (из transfer-задачи урока)

Тебе нужен workflow-«сторож» подписки, который живёт всё время её существования (могут быть
годы): раз в день проверяет статус оплаты (активность CheckPayment), а ещё умеет в любой момент
принять сигнал "cancel" и завершиться досрочно. Коллега написал так: бесконечный
`for { time.Sleep(24*time.Hour); CheckPayment() }` прямо в workflow, без обработки сигнала.

**Вопрос:** разбери и спроектируй правильно. Опиши:
(a) что не так с time.Sleep и бесконечным циклом без обрезки истории;
(b) как одновременно ждать «сутки ИЛИ сигнал cancel» (какие конструкции SDK);
(c) как не дать истории разрастись за годы жизни workflow.

## Recap из урока

- **workflow.Sleep — durable-таймер, а не реальный sleep**: workflow task отдаётся, воркер свободен, через дни/месяцы TimerFired продолжает исполнение через replay.
- Миллион спящих workflow стоят дёшево — это записи-таймеры на сервере, а не заблокированные потоки. «Напомни через 3 дня», «продли через год» — нативные сценарии.
- **NewTimer + Selector** — гонка «таймер ИЛИ сигнал/активность, что раньше»: идиома human-in-the-loop с дедлайном, переживает рестарт.
- Event history растёт и имеет лимиты → циклические/долгие workflow обязаны вызывать **ContinueAsNew** (новый прогон, чистая история, состояние через аргументы, тот же Workflow ID).
- Периодический запуск — **Schedules** (гибкий first-class объект) или **CronSchedule** (легаси, проще). Для нового кода предпочитай Schedules.

        ## Как работать

        1. Платформа Vibe Learn создаёт копию этого репо в твоём GitHub-аккаунте по клику «Начать домашку» на странице урока (через GitHub `/generate`, codecrafters-pattern).
        2. Склонируй копию локально, реализуй TODO в `main.go` (workflow + активности), прогони тесты, запушь.
        3. CI (`.github/workflows/ci.yml`) запускает `go vet` + `go test ./...` на каждый push. Платформа слушает результат через webhook от GitHub Actions и обновляет статус домашки на странице урока.

        ## Локальное окружение

        - Go 1.22+
        - SDK: `go.temporal.io/sdk`
        - Docker + docker-compose — `docker compose up` поднимает Temporal dev server на `:7233` + Web UI на `:8233`. Адрес переопределяется через env `TEMPORAL_ADDRESS` (дефолт `localhost:7233`).
        - Юнит-тесты на `testsuite.TestWorkflowEnvironment` (активности замоканы) бегут в CI БЕЗ сервера; интеграционный тест включается через `TEMPORAL_INTEGRATION=1`.

        ## Запуск

        ```bash
        # Поднять локальный Temporal dev server + UI
        docker compose up -d
        # Web UI: http://localhost:8233

        # Прогнать тесты (юнит на TestWorkflowEnvironment — без сервера;
        # интеграционный включается через TEMPORAL_INTEGRATION=1)
        go test ./...
        TEMPORAL_INTEGRATION=1 go test ./...

        # Запустить воркер (регистрирует workflow + активности, слушает task queue)
        go run .
        ```

        ## Заметка автора

        Это baseline-шаблон, сгенерированный платформой. Бизнес-сущность задачи (что конкретно реализовать в `main.go`, какие тесты сделать строгими) расширяется по ходу итераций — параллельно с углублением теории урока.

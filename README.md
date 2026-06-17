# Курсы и уроки

Репозиторий для материалов курса о том, как превратить ChatGPT в пульт управления внешними сервисами через Custom GPT, Actions, Railway, MCP и n8n.

## Уроки

### ChatGPT как пульт-управления

Начинать отсюда:

1. [`ChatGPT как пульт-управления/STEP_BY_STEP_SETUP.md`](./ChatGPT%20как%20пульт-управления/STEP_BY_STEP_SETUP.md) — пошаговая инструкция для человека: что он делает руками, а что после этого может сделать GPT.
2. [`ChatGPT как пульт-управления/BOOTSTRAP_PROMPT.md`](./ChatGPT%20как%20пульт-управления/BOOTSTRAP_PROMPT.md) — prompt, который надо вставить в Instructions нового Custom GPT.
3. [`ChatGPT как пульт-управления/README.md`](./ChatGPT%20как%20пульт-управления/README.md) — архитектура, ключи, схема и операционная логика.
4. [`ChatGPT как пульт-управления/CHECKLIST.md`](./ChatGPT%20как%20пульт-управления/CHECKLIST.md) — чеклист настройки.
5. [`ChatGPT как пульт-управления/NEEDS_DISCOVERY.md`](./ChatGPT%20как%20пульт-управления/NEEDS_DISCOVERY.md) — что ещё нужно подтвердить в реальной внутрянке Railway/n8n.

## Главная идея

Человек сначала создаёт Custom GPT и добавляет Railway Action с Railway token. После этого GPT получает возможность помогать с Railway: создавать/проверять сервисы, поднимать `n8n-mcp` и `mcpo`, смотреть health/logs и готовить OpenAPI schema для следующего Action.

Важно: `N8N_API_KEY` не вставляется в GPT Actions. Он хранится только server-side в Railway variables backend-сервиса.

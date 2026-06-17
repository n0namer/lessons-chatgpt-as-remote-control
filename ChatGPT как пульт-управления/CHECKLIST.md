# Checklist настройки

## Перед началом

- [ ] Есть аккаунт ChatGPT с доступом к Custom GPT / Actions.
- [ ] Есть Railway account.
- [ ] Есть n8n instance.
- [ ] Есть право создавать n8n API key.
- [ ] Есть понимание, где хранить секреты: только Railway variables / GPT Action auth, не в репозитории.

## GPT / Actions

- [ ] Создан Custom GPT.
- [ ] Добавлена инструкция: read-only first, write only after confirmation.
- [ ] Добавлен Railway Action.
- [ ] Railway Action использует Bearer Railway API token.
- [ ] Добавлен n8n bridge Action.
- [ ] n8n bridge Action использует bridge token, не `N8N_API_KEY`.

## Railway

- [ ] Создан проект.
- [ ] Поднят `n8n-mcp` или `n8n-control-mcp`.
- [ ] Поднят `mcpo` / OpenAPI фасад.
- [ ] Настроен `N8N_API_URL`.
- [ ] Настроен `N8N_API_KEY`.
- [ ] Настроен `MCP_MODE=http`.
- [ ] Настроен `MCP_AUTH_TOKEN` / `AUTH_TOKEN`.
- [ ] Health check показывает, что bridge жив.
- [ ] Target n8n API connection показывает connected.

## n8n

- [ ] Создан n8n API key.
- [ ] API key не вставлен в GPT Action.
- [ ] API key хранится только server-side в Railway env.
- [ ] Read-only smoke test возвращает список workflows.

## Безопасность

- [ ] Секреты не записаны в markdown.
- [ ] Секреты не попали в screenshots.
- [ ] Production workflow не редактируется без подтверждения.
- [ ] Перед patch есть rollback или backup.

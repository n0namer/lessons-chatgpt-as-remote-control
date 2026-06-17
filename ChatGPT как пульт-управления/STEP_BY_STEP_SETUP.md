# Пошаговая настройка: ChatGPT как пульт управления n8n

Цель этого документа — провести человека от нуля до рабочей связки:

```text
Custom GPT → GPT Actions → Railway → mcpo → n8n-mcp → n8n
```

Главная идея: человек вручную делает только те шаги, где нужны аккаунты, UI и секреты. После этого ChatGPT получает минимальный доступ к Railway и уже может помогать развернуть bridge, проверить логи, поправить конфигурацию и собрать OpenAPI-схему для следующих Actions.

---

## 0. Что в итоге должно получиться

После настройки пользователь сможет писать в Custom GPT примерно так:

```text
Покажи список n8n workflows.
Провалидируй последний изменённый workflow.
Объясни предупреждения валидатора.
Предложи безопасную первую волну исправлений.
Сделай patch только после моего подтверждения.
```

GPT будет действовать через защищённую прокладку, а не напрямую через n8n API.

---

## 1. Важная лестница доступа

Настройка идёт не одним большим прыжком, а по этапам.

```text
Этап A. Человек создаёт Custom GPT.
Этап B. Человек добавляет Railway Action и Railway token.
Этап C. GPT получает возможность управлять Railway.
Этап D. GPT помогает поднять n8n-mcp и mcpo на Railway.
Этап E. Человек создаёт n8n API key и кладёт его в Railway env.
Этап F. GPT проверяет bridge и генерирует/уточняет OpenAPI schema.
Этап G. Человек добавляет n8n bridge Action в GPT.
Этап H. GPT может читать, валидировать и после подтверждения редактировать n8n workflows.
```

Именно поэтому сначала нужен Railway Action: без него GPT не сможет “сам” поднять MCP-сервисы.

---

## 2. Что человек должен подготовить вручную

### Аккаунты

- ChatGPT с возможностью создавать Custom GPT и Actions.
- Railway account.
- n8n instance.
- GitHub account — желательно, если код bridge будет храниться в repo.

### Ключи

Нужны три разные группы ключей:

| Ключ | Где создаётся | Куда вставляется | Для чего |
|---|---|---|---|
| Railway API token | Railway → Account Settings → Tokens | GPT Action для Railway | Чтобы GPT мог читать/создавать Railway projects/services |
| Bridge token | Генерируется пользователем или GPT предлагает значение | Railway env + GPT Action bridge auth | Чтобы публичный bridge не вызывали посторонние |
| n8n API key | n8n → Settings → n8n API | Только Railway env как `N8N_API_KEY` | Чтобы backend `n8n-mcp` ходил в n8n API |

Жёсткое правило:

```text
N8N_API_KEY не вставлять в GPT Actions.
Он должен жить только в Railway variables backend-сервиса.
```

---

## 3. Этап A — создать Custom GPT

Человек делает руками:

1. Открыть ChatGPT.
2. Перейти в Explore GPTs / My GPTs.
3. Нажать Create.
4. Назвать GPT, например: `n8n MCP оператор`.
5. В Instructions вставить короткий bootstrap prompt из [`BOOTSTRAP_PROMPT.md`](./BOOTSTRAP_PROMPT.md).
6. Сохранить GPT как draft/private.

На этом этапе GPT ещё ничего не умеет вызывать. Он только знает, что нужно делать.

---

## 4. Этап B — добавить Railway Action

Человек делает руками:

1. В настройках Custom GPT открыть `Actions`.
2. Нажать `Create new action`.
3. Вставить OpenAPI schema для Railway GraphQL Action.
4. В Authentication выбрать API Key / Bearer.
5. Вставить Railway API token.
6. Сохранить Action.
7. В чате с GPT попросить:

```text
Проверь read-only доступ к Railway. Не создавай и не меняй сервисы. Только покажи, какие проекты видишь, и подтверди, что токен работает.
```

Почему это важно: Railway API использует token для доступа к Public API/GraphQL; Railway docs показывают авторизацию через `Authorization: Bearer ACCESS_TOKEN`.

---

## 5. Этап C — GPT проверяет Railway read-only

GPT должен сделать только read-only действия:

- получить текущего пользователя;
- получить список projects;
- получить список services;
- не читать значения секретов;
- не менять env;
- не деплоить.

Если проверка проходит, у GPT появляется “рука” для Railway.

Если проверка не проходит:

- проверить, что токен вставлен без пробелов;
- проверить, что Action использует Bearer;
- создать совый Railway token;
- повторить read-only проверку.

---

## 6. Этап D — GPT помогает поднять MCP stack на Railway

Пользователь даёт GPT команду:

```text
Прочитай инструкцию в этом репозитории. 
Нужно создать на Railway bridge для n8n:
- приватный n8n-mcp backend;
- публичный mcpo/OpenAPI фасад;
- bridge token для внешнего входа;
- N8N_API_KEY хранить только server-side.
Сначала дай план, потом делай только read-only/propose, а apply — после моего подтверждения.
```

GPT должен предложить план:

1. Найти актуальные репозитории/образы:
   - `n8n-mcp`;
   - `mcpo`.
2. Создать Railway project или использовать существующий.
3. Создать service для `n8n-mcp`.
4. Создать service для `mcpo`.
5. Настроить env names.
6. Настроить start command.
7. Проверить logs/health.
8. Сформировать OpenAPI schema для GPT Action.

---

## 7. Этап E — человек создаёт n8n API key

Человек делает руками в n8n:

1. Открыть n8n.
2. Перейти в Settings.
3. Открыть n8n API.
4. Создать API key.
5. Скопировать key один раз.
6. Передать его не в чат как текст, а вставить в Railway variables:

```env
N8N_API_KEY=<masked>
N8N_API_URL=https://<your-n8n-domain>
```

Если GPT имеет Railway Action с правами на env, безопаснее всё равно делать это через UI Railway, чтобы секрет не попадал в чат.

---

## 8. Этап F — настроить bridge token

Bridge token нужен для защиты публичного `mcpo`/bridge endpoint.

В Railway variables bridge-сервиса должны быть переменные вроде:

```env
MCP_MODE=http
MCP_AUTH_TOKEN=<masked>
AUTH_TOKEN=<masked>
NODE_ENV=production
```

В GPT Action для bridge вставляется только этот bridge token, обычно как:

```text
Authorization: Bearer <bridge_token>
```

---

## 9. Этап G — GPT проверяет health

После деплоя GPT должен проверить:

```text
1. Railway service running.
2. Logs без crash loop.
3. HTTP health endpoint отвечает.
4. authTokenConfigured=true.
5. n8n API connected=true.
6. read-only list workflows работает.
```

Ожидаемый безопасный smoke test:

```text
Покажи список n8n workflows. Ничего не редактируй.
```

Если `list workflows` работает, значит цепочка жива:

```text
GPT Action → bridge → n8n-mcp → n8n API
```

---

## 10. Этап H — добавить n8n bridge Action в GPT

Человек делает руками:

1. В Custom GPT открыть Actions.
2. Создать новое Action.
3. Вставить OpenAPI schema, которую дал GPT после проверки `mcpo`.
4. В Authentication выбрать API Key / Bearer.
5. Вставить bridge token.
6. Сохранить.
7. Проверить endpoint list/read-only.

Важно:

```text
В этот Action нельзя вставлять N8N_API_KEY.
Только bridge token.
```

---

## 11. После этого GPT уже может делать больше

Когда Railway Action и n8n bridge Action настроены, GPT может помогать:

### Read-only

- показать workflows;
- найти последний изменённый workflow;
- получить workflow structure;
- валидировать workflow;
- объяснять ошибки и warnings;
- предлагать план исправлений.

### Propose patch

- предложить волну исправлений;
- показать diff/patch;
- указать rollback;
- объяснить риски.

### Apply only after approval

- применить patch;
- проверить workflow;
- сделать readback;
- прогнать smoke test;
- отчитаться evidence.

---

## 12. Минимальный правильный режим работы

Пользователь пишет:

```text
Провалидируй последний изменённый workflow. 
Ничего не меняй, только отчёт.
```

Потом:

```text
Предложи первую безопасную волну исправлений. 
Покажи, что именно поменяешь.
```

Потом:

```text
Подтверждаю применение первой волны.
Сделай patch, затем validate, readback и smoke test.
```

---

## 13. Что запрещено делать автоматически

GPT не должен без отдельного подтверждения:

- менять production workflows;
- менять credentials;
- менять Railway env secrets;
- делать restart/redeploy;
- удалять workflows/services;
- вставлять секреты в markdown;
- публиковать n8n API key в GPT Actions;
- писать `DONE` без evidence.

---

## 14. Как объяснять это ученику простыми словами

```text
Сначала мы даём GPT доступ к Railway.
После этого GPT может сам помочь поднять техническую прокладку.
Но секрет n8n мы не отдаём GPT Action.
Мы кладём его внутрь Railway, как переменную окружения.
Снаружи ChatGPT видит только защищённый bridge endpoint.
```

---

## 15. Контрольная проверка результата

Финальная проверка считается успешной, если:

- GPT видит Railway project/service.
- Bridge service отвечает health.
- Bridge требует Bearer token.
- n8n-mcp видит n8n API.
- GPT может получить список workflows.
- GPT может валидировать workflow.
- GPT не знает и не показывает `N8N_API_KEY`.


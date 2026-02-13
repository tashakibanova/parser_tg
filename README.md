# parser_tg

## Как подключить GPT API ключ и Telegram-канал в n8n

Ниже шаги для workflow `n8n_education_legal_news_agent.json`.

## Важно: если у вас n8n Cloud

В облачной версии обычно **нет доступа к переменным окружения контейнера** (`$env...`), поэтому нужно задать значения прямо в UI n8n или через Credentials.

### OpenAI в n8n Cloud (как на вашем скриншоте)

1. Откройте узел **Rewrite with OpenAI**.
2. В `Header Parameters` у поля `Authorization` замените:
   - было: `Bearer {{$env.OPENAI_API_KEY}}`
   - станет: `Bearer sk-...ваш_ключ...`
3. `Content-Type` оставьте `application/json`.
4. Нажмите **Execute step** и проверьте, что приходит `choices[0].message.content`.

> Более безопасный вариант: вынести ключ в Credential (HTTP Header Auth) и выбрать его в узле, чтобы ключ не хранить в явном виде в ноде.

### Telegram в n8n Cloud

1. Создайте бота в `@BotFather` и получите токен.
2. Добавьте бота админом в канал с правом публикации.
3. В n8n: **Credentials → New → Telegram API** → вставьте токен.
4. В узле `Telegram` выберите созданные credentials.
5. В поле `Chat ID` укажите:
   - `@username_канала` (для публичного канала), или
   - `-100xxxxxxxxxx` (числовой ID).
6. Если сейчас стоит `{{$env.TELEGRAM_CHANNEL_ID}}`, замените на ваш `@username` или `-100...`.

---

## 1) Как вставить API ключ GPT (OpenAI) в self-hosted n8n

В workflow ключ берется из переменной окружения:

- `OPENAI_API_KEY`

Варианты, как задать:

- Через docker-compose для n8n:

```yaml
services:
  n8n:
    environment:
      - OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxx
      - TELEGRAM_CHANNEL_ID=@your_channel_username
```

- Через запуск контейнера:

```bash
docker run -e OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxx -e TELEGRAM_CHANNEL_ID=@your_channel_username n8nio/n8n
```

После этого в узле `Rewrite with OpenAI` будет работать заголовок:

- `Authorization: Bearer {{$env.OPENAI_API_KEY}}`

---

## 2) Как подключить Telegram-канал

### Шаг A. Создайте бота

1. Откройте `@BotFather` в Telegram.
2. Выполните `/newbot`.
3. Сохраните токен бота (формат вроде `123456:ABC...`).

### Шаг B. Добавьте бота в канал

1. Откройте ваш канал.
2. Добавьте бота как администратора.
3. Дайте права на публикацию сообщений.

### Шаг C. Укажите канал

- Для self-hosted: через `TELEGRAM_CHANNEL_ID`.
- Для n8n Cloud: напрямую в поле `Chat ID` узла `Telegram`.

Можно указать:

- `@username_канала` (если есть публичный username), или
- числовой ID канала вида `-100xxxxxxxxxx`.

### Шаг D. Подключите credentials в n8n

1. В n8n: **Credentials → New → Telegram API**.
2. Вставьте токен от BotFather.
3. Откройте узел `Telegram` в workflow.
4. Выберите созданные credentials (вместо placeholder `__FILL_TELEGRAM_CREDENTIAL_ID__`).

---

## 3) Быстрая проверка после настройки

1. Импортируйте `n8n_education_legal_news_agent.json`.
2. Проверьте OpenAI-узел `Rewrite with OpenAI` (есть валидный `Authorization`).
3. Проверьте узел `Telegram` (credentials выбраны, `Chat ID` заполнен).
4. Нажмите **Execute workflow** и проверьте публикацию в канале.

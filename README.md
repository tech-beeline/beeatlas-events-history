# events-history

Сервис **events-history** (Spring Boot 2.7): REST, JPA + PostgreSQL, Flyway, RabbitMQ, Actuator (health, metrics, Prometheus), OpenTelemetry (при включении).

## Требования

- JDK 17, Maven 3.8+
- PostgreSQL, RabbitMQ (локально или через Docker Compose)

## Сборка

```bash
mvn clean package
```

Артефакт: `target/events-history-<версия>.jar` (`pom.xml`, `finalName`).

## Docker

```bash
docker build -t events-history:local .
```

## Docker Compose

```bash
docker compose up --build
```

Сервисы: **postgres** (`events-history-postgres`), **rabbitmq** (`events-history-rabbitmq`), **events-history** (`events-history-app`), сеть `events-history-network`.

При старте RabbitMQ в `command` объявляются очереди: `notification`, `someQueue`, `TECH_PRODUCT_RELATION`, `package_queue`, `business_capability_queue`, `product_queue`.

Приложение: `http://localhost:${APP_PORT:-8080}`.

### RabbitMQ и `app.ambassador-auth`

В `RabbitConfig` выбор способа подключения к брокеру:

- **`app.ambassador-auth=false`** (значение по умолчанию в `application.properties` и явно в compose: **`APP_AMBASSADOR_AUTH=false`**) — подключение по **логину и паролю** из `spring.rabbitmq.username` / `spring.rabbitmq.password`.
- **`app.ambassador-auth=true`** — подключение через **токен** (`AuthSSOClient`, `integration.authsso-server-url`); переменная окружения **`INTEGRATION_AUTHSSO_SERVER_URL`** в compose.

При запуске в Kubernetes/другом окружении для обычного Rabbit с логином/паролем задайте **`APP_AMBASSADOR_AUTH=false`** (или эквивалент `app.ambassador-auth=false` в конфиге).

### Переменные окружения (compose / деплой)

| Переменная | Назначение | Пример / по умолчанию в compose |
|------------|------------|----------------------------------|
| `APP_AMBASSADOR_AUTH` | Режим Rabbit: `false` — логин/пароль; `true` — SSO-токен | `false` |
| `RABBITMQ_HOST` | Хост брокера для приложения | `events-history-rabbitmq` |
| `RABBITMQ_PORT` | Порт AMQP | `5672` |
| `RABBITMQ_USER` / `RABBITMQ_PASSWORD` | Учётная запись RabbitMQ | `guest` / `guest` |
| `RABBITMQ_VHOST` | Виртуальный хост | `/` |
| `RABBITMQ_EXCHANGE` / `RABBITMQ_ROUTING_KEY` | Direct exchange и routing key для `RabbitTemplate` | `events.history.exchange` / `events.history` |
| `POSTGRES_HOST` | Хост БД (имя контейнера Postgres в compose) | `events-history-postgres` |
| `POSTGRES_DB` | Имя БД | `events_history` |
| `POSTGRES_USER` / `POSTGRES_PASSWORD` | Учётные данные PostgreSQL | `postgres` / `postgres` |
| `INTEGRATION_AUTHSSO_SERVER_URL` | URL Auth SSO (нужен при `app.ambassador-auth=true`) | `http://auth-service:8080` |
| `APP_PORT` | Порт HTTP на хосте | `8080` |

Свойства **`queue.events.name`** и **`queue.notification.name`** задаются в `application.properties` (по умолчанию `someQueue` и `notification`); при необходимости переопределите их в своём окружении.

Схема БД: **`entity_events`** (Flyway, JPA).

## Конфигурация

Файл по умолчанию: `src/main/resources/application.properties`. URL БД вне compose: `SPRING_DATASOURCE_URL`, учётные данные — `SPRING_DATASOURCE_USERNAME` / `SPRING_DATASOURCE_PASSWORD`.

## Репозиторий

```bash
git remote add origin https://git.vimpelcom.ru/products/eafdmmart/events-history.git
git branch -M main
git push -uf origin main
```

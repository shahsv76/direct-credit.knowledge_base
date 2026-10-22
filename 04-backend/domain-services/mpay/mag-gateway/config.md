---
type: page
title: "Config (MAG)"
tags: [mag, mpay, config, env]
owner: "Команда ЛК"
updated: "2026-10-22"
---

# Config (MAG)

Конфигурация микросервиса `mpay-api-gateway` задается через переменные окружения (Environment Variables) и парсится при старте приложения с помощью библиотеки `caarlos0/env`.

## Основные настройки

| Переменная (Env)          | Тип      | Значение по умолчанию           | Описание                                 |
| ------------------------- | -------- | ------------------------------- | ---------------------------------------- |
| `APP`                     | String   | `test`                          | Окружение приложения                     |
| `HOSTNAME`                | String   | `localhost`                     | Имя хоста                                |
| `LOG_LEVEL`               | String   | `debug`                         | Уровень логирования (debug, info, error) |
| `APP_PORT`                | Int      | `8888`                          | Порт, на котором запускается сервер      |
| `CONSUMER_ID`             | String   | `mf-gateway`                    | Идентификатор потребителя                |
| `AWAITING_TIME_EVENT`     | Int      | `180`                           | Время ожидания события (в секундах)      |
| `CALLBACK_URL`            | String   | `https://gateway.direct-pay.ru` | URL для callback-запросов                |
| `CALLBACK_TOKEN_LIFETIME` | Duration | `5m`                            | Время жизни токена для callback          |

## Внешние сервисы (URL)

| Переменная (Env)        | Значение по умолчанию                    | Описание                    |
| ----------------------- | ---------------------------------------- | --------------------------- |
| `AUTHENTICATION_URL`    | `http://localhost:8001`                  | Сервис аутентификации       |
| `AUTHORIZATION_URL`     | `http://dcpay-mcia-authorization...`     | Сервис авторизации          |
| `OTP_URL`               | `http://dcpay-mcia-otp...`               | Сервис OTP                  |
| `OFFER_HTTP_URL`        | `http://test-agr.direct-pay.ru`          | Сервис офферов              |
| `ACCOUNTING_URL`        | `http://dcpay-mcia-accounting-reader...` | Сервис бухгалтерии (чтение) |
| `ACCOUNTING_WRITER_URL` | `http://dcpay-mcia-accounting-writer...` | Сервис бухгалтерии (запись) |
| `API_SCORING_URL`       | `http://mfo-cia-scoring-reader...`       | Сервис скоринга             |

## Инфраструктура

### Redis
| Переменная (Env)    | Значение по умолчанию       | Описание                                   |
| ------------------- | --------------------------- | ------------------------------------------ |
| `REDIS_DSN`         | `redis://@127.0.0.1:6379/0` | Строка подключения к Redis                 |
| `RETRIES`           | `5`                         | Количество попыток переподключения         |
| `MAX_RETRY_BACKOFF` | `100`                       | Максимальная задержка между попытками (мс) |
| `EXPIRE_M`          | `10m`                       | Время жизни кэша (Duration)                |

### Metrics (Prometheus и отправка метрик)
| Переменная (Env)             | Значение по умолчанию               | Описание                                     |
| ---------------------------- | ----------------------------------- | -------------------------------------------- |
| `PROMETHEUS_METRICS_ENABLED` | `true`                              | Включить сбор метрик Prometheus              |
| `METRIC_PRODUCER_URL`        | `http://dcpay-mcia-back-metrics...` | URL сервиса сбора метрик                     |
| `METRICS_QUEUE_SIZE`         | `10000`                             | Размер очереди для отправки метрик           |
| `METRICS_WORKER_COUNT`       | `4`                                 | Количество воркеров для отправки метрик      |
| `METRICS_SEND_TIMEOUT`       | `5s`                                | Таймаут отправки метрики                     |
| `METRICS_DROP_ON_FULL`       | `false`                             | Дропать ли метрики, если очередь переполнена |

### WebSocket
| Переменная (Env)     | Значение по умолчанию | Описание                     |
| -------------------- | --------------------- | ---------------------------- |
| `WS_DEFAULT_TIMEOUT` | `300s`                | Таймаут WebSocket-соединения |
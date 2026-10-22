---
type: page
title: "Метрики (MAG)"
tags: [mag, mpay, metrics, observability]
owner: "Команда ЛК"
updated: "2026-10-22"
---

# Метрики (MAG)

Сервис `mpay-api-gateway` собирает и отправляет метрики для мониторинга и аналитики. Используются два механизма:
1. **Prometheus** — технические метрики.
2. **Внешний сервис-приемник** (библиотека `common-libs/metrics-client`) — бизнес-метрики.

## Механизм отправки

`mpay-api-gateway` использует `metrics.Sender` из библиотеки `gitlab.tech.mvideo.ru/mvideoru/fip/common-libs/metrics-client`.

* Метрики отправляются асинхронно через очередь (`chan sendTask`).
* Воркеры (по умолчанию 4) разбирают очередь и делают POST на `(MetricProducerURL)/api/v1/metrics/write`.
* Параметры очереди настраиваются через env (см. [Config (MAG)](./config.md)):

| Параметр                   | Env                          | По умолчанию |
| :------------------------- | :--------------------------- | :----------- |
| Размер очереди             | `METRICS_QUEUE_SIZE`         | 10000        |
| Кол-во воркеров            | `METRICS_WORKER_COUNT`       | 4            |
| Таймаут отправки           | `METRICS_SEND_TIMEOUT`       | 5s           |
| Дропать при полной очереди | `METRICS_DROP_ON_FULL`       | false        |
| Включить Prometheus        | `PROMETHEUS_METRICS_ENABLED` | true         |


## Формат события

Тело запроса к метрик-продюсеру:

```json
{
  "topic": "auth",
  "metric_name": "Login User",
  "metric_data": {
    "phoneHash": "5f4dcc3b5aa765d61d8327deb882cf99",
    "clientID": "client-123",
    "partnerID": "dcpay-mobile",
    "type": "Developer",
    "message": "the final auth status",
    "vendor": "mpay-api-gateway (handler.Authentication.PhaseOTPShort()) 32",
    "statusCode": "201",
    "userAgent": "MobileApp/1.0"
  }
}
```

## Состав metric_data

| Поле         | Описание                                              |
| ------------ | ----------------------------------------------------- |
| `phoneHash`  | MD5-хеш телефона клиента                              |
| `clientID`   | Идентификатор клиента                                 |
| `partnerID`  | Идентификатор партнёра                                |
| `type`       | Тип метрики (обычно `Developer` или `Cyber Security`) |
| `message`    | Текстовое описание события (например, текст ошибки)   |
| `vendor`     | Источник события (см. ниже)                           |
| `statusCode` | HTTP-код ответа                                       |
| `userAgent`  | (Опционально) User-Agent клиента                      |

## Формат vendor (источник)
mpay-api-gateway <источник>.<метод>() <номер>

| Часть              | Описание                                 | Пример                                   |
| :----------------- | :--------------------------------------- | :--------------------------------------- |
| `mpay-api-gateway` | Имя сервиса                              | `mpay-api-gateway`                       |
| `<источник>`       | Handler или service и метод              | `handler.Authentication.PhaseOTPShort()` |
| `<номер>`          | Условный маркер точки в коде (не всегда) | `32`                                     |


*(Если вы хотите, чтобы это выглядело как маркированный список, а не как блок кода, можно оформить так):*

**Примеры из кода:**

* `mpay-api-gateway (handler.Authentication.CheckTokenShort() 2`
* `mpay-api-gateway (handler.Authentication.PhaseOneShort() 5`



## Топики

Каждая метрика мапится на топик через `metricTopicMap`. Если метрика не найдена — используется `topicDefault` (пустая строка).

| Топик          | Назначение                          |
| :------------- | :---------------------------------- |
| `auth`         | Авторизация, аутентификация, вход   |
| `sopd`         | Согласия на обработку ПД            |
| `get_product`  | Скоринг, лимиты, заявки, OTP, карты |
| `payments`     | Платежи, погашения, карты           |
| `promo`        | Промо, онбординг, bottom sheet      |
| `insurance`    | Страховые продукты                  |
| `""` (default) | Метрика без явного топика           |

## Имена метрик

### Топик auth

| Имя метрики               | Описание                   |
| :------------------------ | :------------------------- |
| Login User                | События входа/авторизации  |
| Last Login                | Последний вход             |
| Cyber Security Login User | События безопасности входа |

### Топик get_product

| Имя метрики                  | Описание                  |
| :--------------------------- | :------------------------ |
| MFO Cash scans               | Распознавание документов  |
| MFO Cash step 1              | Шаг 1 скоринга            |
| MFO Cash step 2              | Шаг 2 скоринга            |
| MFO Cash Scoring Response    | Ответ скоринга            |
| MFO Cash Loan Request        | Заявка на займ            |
| MFO Cash Loan Response       | Ответ по заявке на займ   |
| MFO Scoring Photo Request    | Запрос фото-скоринга      |
| MFO Scoring Photo Response   | Ответ фото-скоринга       |
| MFO Cash OTP Check Request   | Проверка OTP              |
| MFO Cash OTP Check Response  | Ответ проверки OTP        |
| MFO Cash OTP Resend Request  | Переотправка OTP          |
| MFO Cash OTP Recheck Request | Повторная проверка OTP    |
| MFO Cash OTP SOPD Send       | Отправка OTP для СОПД     |
| MFO Cash OTP SOPD Check      | Проверка OTP для СОПД     |
| MFO Cash OTP SOPD Resend     | Переотправка OTP для СОПД |
| MFO Cash Approve             | Одобрение                 |
| MFO Cash offers              | Офферы                    |
| MFO Cash Credit              | Кредит                    |
| MFO Choosen Card Request     | Запрос выбора карты       |
| MFO Choosen Card Response    | Ответ выбора карты        |
| Get_cards_v2                 | Получение карт v2         |
| Web-site                     | Веб-сайт                  |
| MFO_sendToPartners           | Отправка партнёрам        |

### Топик payments

| Имя метрики                | Описание           |
| :------------------------- | :----------------- |
| Payment MR                 | Платёж MR          |
| Payment Request MR         | Запрос платежа MR  |
| Payment Request POS        | Запрос платежа POS |
| Payment Request MFO        | Запрос платежа MFO |
| Get_payb2p                 | Оплата B2P         |
| Delete_card                | Удаление карты     |
| Delete_card_v2             | Удаление карты v2  |
| Payments Request           | Запросы платежей   |
| Repayment Payments Request | Запросы погашений  |
| Bank Card Application      | Заявка на карту    |
| receipt                    | Квитанция          |

### Топик sopd

| Имя метрики | Описание        |
| :---------- | :-------------- |
| SOPD Sign   | Подписание СОПД |


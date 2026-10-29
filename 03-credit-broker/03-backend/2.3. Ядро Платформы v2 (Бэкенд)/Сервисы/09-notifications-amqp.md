---
type: service-domain
title: "Сервисы: Уведомления, очереди и интеграция сообщений"
tags: [backend, services, notifications, sms, email, amqp, rabbitmq, bitrix, dc-module]
owner: "Команда ЛК"
updated: "2026-10-29"
---

# Уведомления, очереди и интеграция сообщений

## История изменений
| Дата       | Задача                                                 | Автор                                                                            | Изменение                        |
| ---------- | ------------------------------------------------------ | -------------------------------------------------------------------------------- | -------------------------------- |
| 29.10.2026 | [CASH-4817a](https://jira.mvideo.ru/browse/CASH-4817a) | [Еременко Светлана Александровна(ДК)](https://wiki.mvideo.ru/display/~SEremenko) | Создание документации (через ИИ) |

## Назначение домена

Домен описывает отправку сообщений клиенту и партнёрам: SMS и email, уведомления о статусах заявки, транспорт AMQP (публикация сообщений и обработка dead-letter), распределение нотификаций партнёрам.

## Сервисы

### SMS и email

| Класс                                                                                                                            | Роль                                                                   |
| -------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| [`SmsService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Utils/Notification/Sms/SmsService.php)               | Точка отправки SMS-сообщений.                                          |
| [`Sender`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Utils/Notification/Sms/Sender.php)                       | Транспорт отправки SMS через провайдера.                               |
| [`DefaultSmsService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Utils/Notification/Sms/DefaultSmsService.php) | Дефолтная реализация сервиса SMS (шаблоны, подстановки).               |
| [`DevSmsChecker`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Utils/Notification/Sms/DevSmsChecker.php)         | Проверка dev-режима отправки SMS (отправка только на тестовые номера). |
| [`Sender`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Utils/Notification/Email/Sender.php) (Email)             | Постановка письма в очередь AMQP вместо синхронной отправки.           |
| `Sms\Factory\BuilderFactory`, `Email\Factory\BuilderFactory`                                                                     | Сборка сообщения (SMS/email) из шаблона и данных.                      |

### Уведомления о статусах и партнёрам

| Класс                                                                                                                                         | Роль                                                                    |
| --------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| [`NotificationService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/NotificationService/NotificationService.php) | Отправка уведомлений о смене статуса заявки, логирование запросов.      |
| [`SendStatusSender`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/NotificationService/SendStatusSender.php)       | Транспорт отправки статусных уведомлений.                               |
| [`Manager`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/PartnerNotifier/Manager.php)                                         | Менеджер нотификаций партнёрам: запуск и завершение потока уведомлений. |
| [`ManagerTaskRouter`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/PartnerNotifier/ManagerTaskRouter.php)                     | Роутинг задач нотификации между «smart» и «normal» каналами.            |

### AMQP / очереди

| Класс                                                                                                                                                                                                                | Роль                                                                    |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| [`AmqpService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/AMQP/AmqpService.php)                                                                                                       | Сервисный фасад работы с AMQP: получение данных сообщения/конфигурации. |
| [`Sender`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/AMQP/Sender.php)                                                                                                                 | Отправщик сообщений в AMQP (connect / send / disconnect).               |
| [`Service`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/AMQP/Service.php), [`DlxService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/AMQP/DlxService.php) | Низкоуровневая работа с очередями и dead-letter-очередью.               |
| [`AmqpConfigFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/AMQP/Config/AmqpConfigFactory.php)                                                                                    | Фабрика конфигурации AMQP (очереди, обменники, демоны).                 |
| [`DelayedMailWorkerFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/AMQP/Worker/DelayedMail/DelayedMailWorkerFactory.php)                                                          | Фабрика воркера отложенных писем.                                       |
| [`CurlEnvelopFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/AMQP/Envelope/CurlNotifier/CurlEnvelopFactory.php)                                                                   | Формирование конверта запроса для curl-нотификатора.                    |

## Связь с API

| Action                                       | Что используют сервисы домена                                                     |
| -------------------------------------------- | --------------------------------------------------------------------------------- |
| `sms/send`                                   | `SmsService`, `Sender`, `DevSmsChecker` — отправка кода подтверждения             |
| `sign/code-send`                             | `SmsService`, `NotificationService` — SMS-подписание                              |
| `passport-upload/sms-send`, `photo/send-sms` | `SmsService`, `SendStatusSender`                                                  |
| `status/get`                                 | `NotificationService`, `SendStatusSender` — статус и связанные уведомления        |
| `api-log/send-error`                         | `AmqpService`, `Sender` — публикация ошибок фронта в очередь                      |
| `feedback/call`, `feedback/form`             | `NotificationService`, `PartnerNotifier\Manager` — уведомление партнёру/оператору |

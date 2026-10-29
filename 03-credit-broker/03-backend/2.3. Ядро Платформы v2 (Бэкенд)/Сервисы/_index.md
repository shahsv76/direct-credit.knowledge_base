---
type: service-domain
title: "Сервисы (Ядро Платформы v2) — карта доменов"
tags: [backend, services, monolith, bitrix, dc-module, architecture]
owner: "Команда ЛК"
updated: "2026-10-29"
---

# Сервисы (Ядро Платформы v2)

## История изменений
| Дата       | Задача                                                 | Автор                                                                            | Изменение                        |
| ---------- | ------------------------------------------------------ | -------------------------------------------------------------------------------- | -------------------------------- |
| 29.10.2026 | [CASH-4817a](https://jira.mvideo.ru/browse/CASH-4817a) | [Еременко Светлана Александровна(ДК)](https://wiki.mvideo.ru/display/~SEremenko) | Создание документации (через ИИ) |

## Назначение раздела

Раздел описывает **карту сервисов бизнес-логики** монолита `newprod.direct-credit.ru` (1С-Битрикс, модуль `dc`): какие классы выполняют роль сервисов, менеджеров и фабрик, за какой бизнес-домен они отвечают и через какие методы BFF-API они вызываются.

Документ построен по исходному коду `local/modules/dc/**` (папки `lib/`, `services_api/`, `bankapi/`, `online/`). Ссылки в таблицах ведут на файлы-источники.

## Как отбирались сервисы

Всего в модуле `dc` — **4 326** PHP-файлов, из них **1 205** содержат в имени ключевые слова `Manager`, `Service`, `Factory`, `Sender`, `Checker`, `Provider`, `Resolver`, `Processor`, `Handler`, `Gateway`, `Repository`.

В карту вошли классы, реализующие бизнес-логику. **Не вошли:**

| Что исключено                                                                                                       | Причина                                                                                      |
| ------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| `Bx*Manager`, `Hb*Manager`, `Srv*Manager`, `T*Manager`, `*Table`                                                    | ORM-обёртки и слой доступа к данным, а не бизнес-логика                                      |
| наследники `BaseProfile`, `BaseManager`                                                                             | простые сущности/базовые классы                                                              |
| `bankapi\*\Input\*Factory`, `bankapi\*\Output\*Factory`, `*\Dto\`, `*DtoFactory`                                    | DTO и фабрики DTO                                                                            |
| `*Interface`                                                                                                        | контракты; упоминаются в описаниях соответствующих сервисов                                  |
| `local/modules/dc/online/services/service.php`, `online/api/json_cache/service.php`, `online/support/*_factory.php` | легаси JSONP/JSON-cache API                                                                  |
| хелперы строк/массивов, валидаторы полей, логгеры                                                                   | утилитарный слой (кроме тех, что приведены в [`12-infrastructure.md`](12-infrastructure.md)) |

## Карта доменов

| №   | Домен                       | Документ                                                                     | Ключевые классы                                                                           |
| --- | --------------------------- | ---------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| 1   | Заказы и бизнес-процесс     | [`01-orders.md`](01-orders.md)                                               | `BaseOrderService`, `ApplicationStateService`, `OrderParamsService`, `ServicesUpdater`    |
| 2   | Сплит заказов               | [`02-order-split.md`](02-order-split.md)                                     | `OrderSplitStateService`, `OrderSplitRelationManager`, `SuperOrderFactory`                |
| 3   | Скоринг и SAMA              | [`03-scoring-sama.md`](03-scoring-sama.md)                                   | `ScoreStateService`, `ScoreSender`, `SamaTypeChecker`, `ShortProfileCheckService`         |
| 4   | Отмена и аннулирование      | [`04-cancellation.md`](04-cancellation.md)                                   | `CancelOrderService`, `CancelResolver`, `ReChangeResolver`, `Runner\Factory`              |
| 5   | Документы и подписание      | [`05-documents-signing.md`](05-documents-signing.md)                         | `AgreementDocumentService`, `ArchiveService`, `SignStepChecker`, `ServiceQrGenerator`     |
| 6   | Платежи и переводы          | [`06-payments.md`](06-payments.md)                                           | `FundsTransferService`, `PaymentB2p`, `PaymentMethodsService`, `MpayLimitGateService`     |
| 7   | Страхование и доп. продукты | [`07-insurance-additional-products.md`](07-insurance-additional-products.md) | `ServicePriceCalculator`, `ProductProviderFactory`, `CalculationsAddProductsService`      |
| 8   | Авторизация и безопасность  | [`08-authorization-security.md`](08-authorization-security.md)               | `PartnerAuthService`, `CiaAuthService`, `OrderSecurityService`, `CheckoutSecurityManager` |
| 9   | Уведомления и AMQP          | [`09-notifications-amqp.md`](09-notifications-amqp.md)                       | `SmsService`, `Email\Sender`, `AmqpService`, `PartnerNotifier\Manager`                    |
| 10  | Интеграции с банками        | [`10-bank-integrations.md`](10-bank-integrations.md)                         | `BankLogicFactory`, VTB-process-сервисы, `StandardEventHandler`                           |
| 11  | Логистика и выдача          | [`11-logistics.md`](11-logistics.md)                                         | `DeliveryApiFactory`, `GoodsGivenResolver`                                                |
| 12  | Инфраструктура платформы    | [`12-infrastructure.md`](12-infrastructure.md)                               | `CodesService`, `OmService`, `EventManager`, `Domain\*Repository`                         |

## Точка входа API

Все перечисленные сервисы вызываются из BFF-слоя виджета: маршруты объявлены в [`RouterProvider.php`](../../../../../newprod.direct-сredit.ru/bitrix/components/dc/online.widget.new/lib/RouterProvider.php) (компонент `dc:online.widget.new`), адрес — `/api/widget.php?action=<action>`.

Сводная таблица соответствия «action → домен»:

| Префикс action                                                                                      | Домен                  | Примеры                                                                                        |
| --------------------------------------------------------------------------------------------------- | ---------------------- | ---------------------------------------------------------------------------------------------- |
| `start/*`, `get-current-step`, `step1/*`, `step2/*`, `step3/*`, `calc/*`, `status/get`, `final/get` | Заказы                 | [`01-orders.md`](01-orders.md)                                                                 |
| `bank/get`, `bank/set`, `bank/get-app-status`, `bank/simplified-*`, `ibank/get`                     | Сплит, банки           | [`02-order-split.md`](02-order-split.md), [`10-bank-integrations.md`](10-bank-integrations.md) |
| `auth/form`, `auth/check`, `score/*`                                                                | Скоринг                | [`03-scoring-sama.md`](03-scoring-sama.md)                                                     |
| `cancel/*`, `lk/refuse`, `lk/simplified-refuse`                                                     | Отмена                 | [`04-cancellation.md`](04-cancellation.md)                                                     |
| `agreements-document/*`, `passport-upload/*`, `verify/*`, `photo/*`, `sign/*`, `sms/send`           | Документы и подписание | [`05-documents-signing.md`](05-documents-signing.md)                                           |
| `payment/create`, `payment/check-client-funded`, `yandex-split/*`                                   | Платежи                | [`06-payments.md`](06-payments.md)                                                             |
| `settings/get-services-texts`                                                                       | Доп. продукты          | [`07-insurance-additional-products.md`](07-insurance-additional-products.md)                   |
| `partner/set-url`, `partner/simplified-set-url`                                                     | Авторизация/партнёры   | [`08-authorization-security.md`](08-authorization-security.md)                                 |
| `api-log/send-error`, `feedback/*`                                                                  | Уведомления/логи       | [`09-notifications-amqp.md`](09-notifications-amqp.md)                                         |
| `logistic/*`                                                                                        | Логистика              | [`11-logistics.md`](11-logistics.md)                                                           |
| `config` (widget), `settings/*`, `get-current-step`                                                 | Инфраструктура         | [`12-infrastructure.md`](12-infrastructure.md)                                                 |

## Связанные разделы

* [`../База данных (Схема)/_index.md`](../База%20данных%20(Схема)/_index.md) — таблицы, ИБ и HL-блоки, которыми оперируют сервисы;
* [`../../../../04-backend/domain-services/mpay/bff-front-api/_index.md`](../../../../04-backend/domain-services/mpay/bff-front-api/_index.md) — спецификации BFF-методов;
* [`../../../../05-integrations/banks/_index.md`](../../../../05-integrations/banks/_index.md) — интеграции с банками.

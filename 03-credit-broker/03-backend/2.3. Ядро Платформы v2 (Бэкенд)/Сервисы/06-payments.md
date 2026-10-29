---
type: service-domain
title: "Сервисы: Платежи, переводы средств и фискализация"
tags: [backend, services, payments, best2pay, funds-transfer, fiscal, yandex-split, bitrix, dc-module]
owner: "Команда ЛК"
updated: "2026-10-29"
---

# Платежи, переводы средств и фискализация

## История изменений
| Дата       | Задача                                                 | Автор                                                                            | Изменение                        |
| ---------- | ------------------------------------------------------ | -------------------------------------------------------------------------------- | -------------------------------- |
| 29.10.2026 | [CASH-4817a](https://jira.mvideo.ru/browse/CASH-4817a) | [Еременко Светлана Александровна(ДК)](https://wiki.mvideo.ru/display/~SEremenko) | Создание документации (через ИИ) |

## Назначение домена

Домен отвечает за денежную часть: доступные способы оплаты, выбор клиента, перевод средств по заявке, платёж через Best2Pay, лимиты MPAY, Яндекс.Сплит и фискализацию (чеки).

## Сервисы

### Способы оплаты и выбор клиента

| Класс                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | Роль                                                                                      |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| [`PaymentMethodsService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/PaymentMethodsService/PaymentMethodsService.php)                                                                                                                                                                                                                                                                                                                                                           | Возвращает доступные способы оплаты для заказа с учётом ограничений программ и банков.    |
| [`PaymentMethodDictionaryService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/PaymentMethodsService/PaymentMethodDictionaryService.php)                                                                                                                                                                                                                                                                                                                                         | Справочник способов оплаты для UI (опции выбора).                                         |
| [`AllowedPaymentMethodsFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/PaymentMethodsService/Factory/AllowedPaymentMethodsFactory.php), [`PaymentMethodFilterFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/PaymentMethodsService/Factory/PaymentMethodFilterFactory.php), [`ProgramsFilterFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/PaymentMethodsService/Factory/ProgramsFilterFactory.php) | Фабрики фильтров: какие способы оплаты разрешены и для каких программ.                    |
| [`PaymentMethodCustomerChoiceService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/PaymentMethodCustomerChoiceService/PaymentMethodCustomerChoiceService.php)                                                                                                                                                                                                                                                                                                                    | Сохранение и чтение выбора клиента по способу оплаты, валидация выбора, промо-исключения. |
| [`MyFinancesCustomerChoiceService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/MyFinancesCustomerChoiceService/MyFinancesCustomerChoiceService.php)                                                                                                                                                                                                                                                                                                                             | Выбор клиента в сценарии «Мои финансы» и причины отмены.                                  |

### Движение денег

| Класс                                                                                                                                                                                                                                           | Роль                                                                                                 |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| [`FundsTransferService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/FundsTransferService/FundsTransferService.php)                                                                                                | Создание перевода денежных средств по заказу (зачисление клиенту).                                   |
| [`PaymentB2p`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/PaymentServiceB2p/PaymentB2p.php), [`PaymentB2pClient`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/PaymentServiceB2p/Client/PaymentB2pClient.php) | Платёж через Best2Pay: создание платежа и его финализация.                                           |
| [`MpayLimitGateService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/MpayLimitGateService/MpayLimitGateService.php)                                                                                                | Получение лимита из платёжного шлюза MPAY (настройки тест/прод, сертификаты, таймауты).              |
| [`YandexSplitService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/YandexSplitService/YandexSplitService.php)                                                                                                      | Логика Яндекс.Сплита: когда допустим только сплит, проверка статусов Fast Split, установка атрибута. |
| [`NotifyProxySender`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Rest/FundsTransfer/NotifyProxySender.php)                                                                                                        | Уведомления о событиях перевода средств во внешний сервис.                                           |
| [`NotifyProxySender`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Rest/DcPay/Notify/NotifyProxySender.php)                                                                                                         | Уведомления в DCPay: заявка, клиент, отказ, завершение скоринга.                                     |

### Фискализация

| Класс                                                                                                                                                              | Роль                                                                    |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------- |
| [`CheckSettingsTableManager`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Check/CheckSettingsTableManager.php)                        | Настройки чеков: поиск по ID и по доп. продукту, обновление метаданных. |
| `DictCashRegisterTableManager`, `DictCheckGroupTableManager`, `DictFfdTableManager`, `DictPaymentTypeTableManager`                                                 | Справочники: кассы, группы чеков, версии ФФД, типы оплат.               |
| [`OrangeDataCheckPrinterFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/CheckPrinter/Factory/OrangeDataCheckPrinterFactory.php) | Фабрика фискального принтера OrangeData.                                |
| [`FromBitrixConfigFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/CheckPrinter/Config/Factory/FromBitrixConfigFactory.php)      | Конфигурация принтера чеков из настроек Битрикса.                       |

## Связь с API

| Action                                                                                          | Что используют сервисы домена                                                                          |
| ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| `payment/create`                                                                                | `PaymentB2p`, `FundsTransferService`, `PaymentMethodsService`, `YandexSplitService` — создание платежа |
| `payment/check-client-funded`                                                                   | `FundsTransferService`, `PaymentB2p`, `NotifyProxySender` — проверка зачисления                        |
| `yandex-split/approved`, `yandex-split/refused`, `yandex-split/get-merchant` (+ `simplified-*`) | `YandexSplitService`, `PaymentMethodsService`                                                          |
| `step3/form`, `step3/save`                                                                      | `PaymentMethodsService`, `PaymentMethodCustomerChoiceService`, `MyFinancesCustomerChoiceService`       |
| `final/get`                                                                                     | `FundsTransferService`, `CheckSettingsTableManager` — финальный статус и чек                           |
| `calc/programs`                                                                                 | `PaymentMethodsService` — доступные способы оплаты в калькуляторе                                      |

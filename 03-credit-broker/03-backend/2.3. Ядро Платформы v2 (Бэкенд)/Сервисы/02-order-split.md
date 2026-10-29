---
type: service-domain
title: "Сервисы: Сплит заказов (OrderSplit)"
tags: [backend, services, orders, order-split, sub-orders, bitrix, dc-module]
owner: "Команда ЛК"
updated: "2026-10-29"
---

# Сплит заказов (OrderSplit)

## История изменений
| Дата       | Задача                                                 | Автор                                                                            | Изменение                        |
| ---------- | ------------------------------------------------------ | -------------------------------------------------------------------------------- | -------------------------------- |
| 29.10.2026 | [CASH-4817a](https://jira.mvideo.ru/browse/CASH-4817a) | [Еременко Светлана Александровна(ДК)](https://wiki.mvideo.ru/display/~SEremenko) | Создание документации (через ИИ) |

## Назначение домена

Домен отвечает за разделение одного заказа клиента на несколько заявок в разные банки («суперзаказ» → несколько «сабзаказов»): включение сплита, допустимые варианты разделения, связи между заказами и статусы частей.

## Сервисы

| Класс                                                                                                                                                                                                                                                                                | Роль                                                                                           |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------- |
| [`OrderSplitStateService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/OrderSplit/Service/OrderSplitStateService.php)                                                                                                                                   | Состояние сплита: завершённые и выданные заявки, финальные статусы, список сабзаказов.         |
| [`OrderSplitSettingsManager`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/OrderSplit/Settings/OrderSplitSettingsManager.php)                                                                                                                            | Проверяет, включён ли сплит для магазина/точки и какие варианты разделения доступны.           |
| [`OrderSplitRelationManager`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/OrderSplit/Relations/OrderSplitRelationManager.php)                                                                                                                           | Связи «суперзаказ ↔ сабзаказ»: поиск по заказу/подзаказу, поиск всех частей, сохранение связи. |
| [`OrderSplitRelationCachedManager`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/OrderSplit/Relations/OrderSplitRelationCachedManager.php)                                                                                                               | Кэширующая обёртка над менеджером связей сплита.                                               |
| [`OrderSplitMethodManager`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/OrderSplit/SplitMethod/OrderSplitMethodManager.php)                                                                                                                             | Справочник методов разделения (поиск по коду метода).                                          |
| [`OrderSplitPropertiesManager`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/OrderSplit/Properties/OrderSplitPropertiesManager.php)                                                                                                                      | Свойства сплита, которые сохраняются на заказе.                                                |
| [`SuperOrderFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/OrderSplit/Factory/SuperOrderFactory.php)                                                                                                                                             | Создание суперзаказа, объединяющего части сплита.                                              |
| [`SplitVariantFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/OrderSplit/SplitVariant/Factory/SplitVariantFactory.php)                                                                                                                            | Формирование вариантов разделения заказа для выбора клиентом.                                  |
| [`SubOrderChecker`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/OrderSplit/SubOrderChecker.php)                                                                                                                                                         | Проверка допустимости операций над сабзаказом.                                                 |
| [`AppChecker`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/OrderSplit/AppChecker/AppChecker.php), [`PreConfirmedAppChecker`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/OrderSplit/AppChecker/PreConfirmedAppChecker.php) | Проверки статусов заявок внутри сплита (в т.ч. предподтверждённых).                            |
| [`FinisherModalContentFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/OrderSplit/Finisher/Factory/FinisherModalContentFactory.php)                                                                                                                | Контент финального модального окна после прохождения сплита.                                   |

## Связь с API

| Action                                       | Что используют сервисы домена                                                                                       |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `bank/get`, `bank/set`                       | `OrderSplitSettingsManager`, `SplitVariantFactory`, `OrderSplitRelationManager` — выбор банка и варианта разделения |
| `bank/get-app-status`                        | `OrderSplitStateService`, `AppChecker` — статусы частей сплита                                                      |
| `bank/simplified-get`, `bank/simplified-set` | `OrderSplitSettingsManager`, `SplitVariantFactory` — упрощённый сценарий                                            |
| `score/banks-data`                           | `OrderSplitStateService` — данные банков по заявкам сплита                                                          |
| `final/get`                                  | `OrderSplitStateService`, `FinisherModalContentFactory` — финальное состояние                                       |
| `get-current-step`                           | `OrderSplitStateService`, `SubOrderChecker` — продолжение оформления по части сплита                                |

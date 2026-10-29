---
type: service-domain
title: "Сервисы: Логистика и выдача товара"
tags: [backend, services, logistics, delivery, goods-given, bitrix, dc-module]
owner: "Команда ЛК"
updated: "2026-10-29"
---

# Логистика и выдача товара

## История изменений
| Дата       | Задача                                                 | Автор                                                                            | Изменение                        |
| ---------- | ------------------------------------------------------ | -------------------------------------------------------------------------------- | -------------------------------- |
| 29.10.2026 | [CASH-4817a](https://jira.mvideo.ru/browse/CASH-4817a) | [Еременко Светлана Александровна(ДК)](https://wiki.mvideo.ru/display/~SEremenko) | Создание документации (через ИИ) |

## Назначение домена

Домен отвечает за доставку и выдачу товара по заказу: получение API доставки для конкретного перевозчика, обработку событий доставки документов/товара и определение факта выдачи товара (влияет на доступность возврата).

## Сервисы

| Класс                                                                                                                                                          | Роль                                                                        |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| [`DeliveryApiFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/logistics/api/DeliveryApiFactory.php)                                      | Возвращает реализацию API доставки по коду или ID перевозчика.              |
| [`StandardEventHandler`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/logistics/api/EventHandler/StandardEventHandler.php)                     | Базовая обработка событий логистики.                                        |
| [`DocDeliveryEventHandler`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/logistics/api/EventHandler/DocDeliveryEventHandler.php)               | Обработка события доставки документов.                                      |
| [`DocDeliveryDelayedEventHandler`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/logistics/api/EventHandler/DocDeliveryDelayedEventHandler.php) | Обработка задержки доставки документов.                                     |
| [`LogisticEventHandler`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/AMQP/Delayed/EventHandler/LogisticEventHandler.php)          | Отложенная обработка логистических событий через AMQP.                      |
| [`GoodsGivenResolver`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/GoodsGiven/GoodsGivenResolver.php)                             | Определяет, выдан ли товар по заказу (используется при возвратах и отмене). |
| [`ReturnAvailabilityValidator`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Validators/ReturnAvailabilityValidator.php)                       | Валидатор доступности возврата товара с учётом логистики и статусов.        |

## Связь с API

| Action                   | Что используют сервисы домена                                                     |
| ------------------------ | --------------------------------------------------------------------------------- |
| `logistic/form`          | `DeliveryApiFactory`, `ReturnAvailabilityValidator` — доступные способы доставки  |
| `logistic/save`          | `DeliveryApiFactory`, `DocDeliveryEventHandler` — сохранение доставки             |
| `logistic/check-address` | `DeliveryApiFactory` — проверка адреса доставки                                   |
| `final/get`              | `GoodsGivenResolver`, `DocDeliveryDelayedEventHandler` — финальный статус выдачи  |
| `cancel/form`            | `GoodsGivenResolver`, `ReturnAvailabilityValidator` — доступность возврата/отмены |

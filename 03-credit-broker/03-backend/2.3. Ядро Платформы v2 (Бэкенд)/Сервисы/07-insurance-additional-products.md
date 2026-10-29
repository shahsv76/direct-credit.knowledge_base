---
type: service-domain
title: "Сервисы: Страхование и дополнительные продукты"
tags: [backend, services, insurance, additional-product, vsk, rgs, pricing, bitrix, dc-module]
owner: "Команда ЛК"
updated: "2026-10-29"
---

# Страхование и дополнительные продукты

## История изменений
| Дата       | Задача                                                 | Автор                                                                            | Изменение                        |
| ---------- | ------------------------------------------------------ | -------------------------------------------------------------------------------- | -------------------------------- |
| 29.10.2026 | [CASH-4817a](https://jira.mvideo.ru/browse/CASH-4817a) | [Еременко Светлана Александровна(ДК)](https://wiki.mvideo.ru/display/~SEremenko) | Создание документации (через ИИ) |

## Назначение домена

Домен отвечает за небанковские услуги и доп. продукты: расчёт и сохранение цен, проверки допустимости (лимиты, правила связей), провайдеров страховых продуктов (ВСК, РГС), статусы и параметры доп. продуктов, автоматическую замену услуг.

## Сервисы

### Страхование (банковское и небанковское)

| Класс                                                                                                                                                                                                                                                                                                                        | Роль                                                                    |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| [`InsurancePriceManager`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Insurance/InsurancePriceManager.php)                                                                                                                                                                                      | Сохранение рассчитанных цен страховок по заявке.                        |
| [`InsuranceResultChecker`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Insurance/InsuranceResultChecker.php)                                                                                                                                                                                    | Определяет, разрешена ли страховка по результату расчёта.               |
| [`InsuranceLimitPriceChecker`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Insurance/InsurancePriceChecker/InsuranceLimitPriceChecker.php)                                                                                                                                                      | Проверяет, проходят ли страховки по лимиту стоимости.                   |
| [`ServiceStateCache`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Insurance/ServiceStateCache.php)                                                                                                                                                                                              | Кэш состояний услуг заказа (что включено/выключено клиентом).           |
| [`ServicePriceCalculator`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Insurance/Utils/ServicePriceCalculator.php)                                                                                                                                                                                          | Расчёт стоимости небанковских услуг, включая актуарный расчёт и премии. |
| [`InsuranceNotBankFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/InsuranceNotBank/InsuranceNotBankFactory.php)                                                                                                                                                                           | Фабрика объектов небанковских страховок под магазин/программу.          |
| [`AdditionalProductSaleResolver`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/InsuranceNotBank/AdditionalProductSaleResolver.php)                                                                                                                                                               | Определяет группы, где продажа доп. продукта запрещена.                 |
| [`CustomServiceDataFabric`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/InsuranceNotBank/CustomData/CustomServiceDataFabric.php)                                                                                                                                                                | Фабрика кастомных данных услуги (поля, специфичные для партнёра).       |
| [`AddProductRelRuleFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/InsuranceNotBank/Relation/AddProductRelRuleFactory.php), [`AddProductRelRuleChecker`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/InsuranceNotBank/Relation/AddProductRelRuleChecker.php) | Правила связей «услуга ↔ доп. продукт» и их проверка.                   |
| `VskShop\ShopManager`, `VskShop\VskShopManager`, `VskShop\OutletManager`                                                                                                                                                                                                                                                     | Настройки магазинов, точек и точек продаж ВСК.                          |

### Доп. продукты и их расчёты

| Класс                                                                                                                                                                                                                                                                                                  | Роль                                                            |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------- |
| [`AdditionalProductFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/AdditionalProduct/Factory/AdditionalProductFactory.php)                                                                                                                                          | Создание сущности доп. продукта.                                |
| [`ProductProviderFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/AdditionalProduct/Policy/ProductProvider/ProductProviderFactory.php)                                                                                                                               | Фабрика провайдера страхового продукта по коду.                 |
| [`RgsProvider`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/AdditionalProduct/Policy/ProductProvider/RgsProvider.php), [`VskProvider`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/AdditionalProduct/Policy/ProductProvider/VskProvider.php) | Реализации провайдеров РГС и ВСК.                               |
| [`CardsManager`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/AdditionalProduct/Service/CardsManager.php)                                                                                                                                                                  | Отмена карты доп. продукта.                                     |
| [`InfoInsuranceAdditionalProductService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/AdditionalProduct/Service/InfoInsuranceAdditionalProductService.php)                                                                                                                | Данные страховки по доп. продукту (инфо для клиента).           |
| [`AdditionalProductParamsService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/AdditionalProduct/Store/Params/AdditionalProductParamsService.php)                                                                                                                         | Параметры доп. продукта (чтение/запись).                        |
| [`AdditionalProductStatusManager`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/AdditionalProduct/AdditionalProductStatusManager.php)                                                                                                                                      | Статусы доп. продуктов.                                         |
| [`AdditionalProductServiceLinker`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/AdditionalProduct/Utils/AdditionalProductServiceLinker.php)                                                                                                                                | Связывание доп. продукта с услугой заявки.                      |
| [`CalculationsAddProductsService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/CalculationsAddProductsService/CalculationsAddProductsService.php)                                                                                                                         | Расчёты по доп. продуктам с кэшированием результата.            |
| [`ReplacerServicesFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/AutoReplacingServices/ReplacerServicesFactory.php) + `StandardReplacerServices`, `CashReplacerServices`                                                                                           | Автозамена услуг: стандартный сценарий и сценарий для наличных. |

## Связь с API

| Action                        | Что используют сервисы домена                                                                                     |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `calc/programs`, `calc/save`  | `ServicePriceCalculator`, `CalculationsAddProductsService`, `InsuranceLimitPriceChecker` — расчёт стоимости услуг |
| `step2/form`, `step2/save`    | `InsuranceNotBankFactory`, `AdditionalProductFactory`, `ServiceStateCache`, `AddProductRelRuleChecker`            |
| `step3/form`, `step3/save`    | `AdditionalProductParamsService`, `AdditionalProductStatusManager`, `AdditionalProductServiceLinker`              |
| `settings/get-services-texts` | `CustomServiceDataFabric`, `AdditionalProductParamsService` — тексты и описания услуг                             |
| `final/get`                   | `AdditionalProductStatusManager`, `InfoInsuranceAdditionalProductService` — итоговые данные по продуктам          |

---
type: service-domain
title: "Сервисы: Инфраструктура платформы"
tags: [backend, services, infrastructure, routing, cache, logging, settings, bitrix, dc-module]
owner: "Команда ЛК"
updated: "2026-10-29"
---

# Инфраструктура платформы

## История изменений
| Дата       | Задача                                                 | Автор                                                                            | Изменение                        |
| ---------- | ------------------------------------------------------ | -------------------------------------------------------------------------------- | -------------------------------- |
| 29.10.2026 | [CASH-4817a](https://jira.mvideo.ru/browse/CASH-4817a) | [Еременко Светлана Александровна(ДК)](https://wiki.mvideo.ru/display/~SEremenko) | Создание документации (через ИИ) |

## Назначение домена

Платформенные сервисы, которыми пользуются все остальные домены: маршрутизация и коды шагов, получение объект-менеджеров, репозитории домена, планировщик событий, логирование, кэш/настройки, интеграции нормализации данных (DaData) и аналитики (Elastic), проверки видимости полей.

## Сервисы

### Маршрутизация и шаги

| Класс                                                                                                                                                                                                                                                        | Роль                                                               |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------ |
| [`CodesRouteProvider`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/RouteProvider/CodesRouteProvider.php)                                                                                                                                    | Маршруты по коду шага заявки.                                      |
| [`RouteProvider`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Routing/RouteProvider.php), [`RouteProviderAbstract`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Routing/RouteProviderAbstract.php) | Базовый провайдер маршрутов: параметры маршрута, текущий экшен.    |
| [`CodesService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Utils/Codes/CodesService.php)                                                                                                                                                  | Генерация кодов шагов/QR-HTML и флагов отображения.                |
| [`ContinueGateService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Menu/ContinueGateService.php)                                                                                                                               | Определение текущего шага и ссылок навигации (см. домен «Заказы»). |

### Данные, кэш и настройки

| Класс                                                                                                                                                                                                                                                                                                                                                                                                                                          | Роль                                                                    |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| [`OmService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/ObjectManager/OmService.php)                                                                                                                                                                                                                                                                                                                            | Получение объект-менеджера (синхронного или MQ) для работы с сущностью. |
| [`BxObjectManager`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/ObjectManager/BxObjectManager.php), [`MqBxObjectManager`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/ObjectManager/MqBxObjectManager.php)                                                                                                                                                                           | Синхронная и MQ-реализации объект-менеджера.                            |
| [`OrderRepository`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Domain/OrderRepository.php), [`AppRepository`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Domain/AppRepository.php), [`FileRepository`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Domain/FileRepository.php), [`ScanRepository`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Domain/ScanRepository.php) | Репозитории домена: доступ к заказам, заявкам, файлам и сканам.         |
| [`CommonSettingsService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/CommonSettings/Service/CommonSettingsService.php)                                                                                                                                                                                                                                                                                           | Общие настройки: включённость фич, отображение суммы со скидкой.        |
| [`JsonProfileService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/JsonProfile/JsonProfileService.php)                                                                                                                                                                                                                                                                                                            | Заполнение и чтение профиля в JSON-виде.                                |
| [`DifferService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Differ/DifferService.php)                                                                                                                                                                                                                                                                                                                           | Расчёт различий (диффов) сущностей.                                     |
| [`LimitsAdditionalDiffService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/LimitsAdditionalDiff/LimitsAdditionalDiffService.php)                                                                                                                                                                                                                                                                                 | Диффы лимитов по доп. продуктам.                                        |
| [`ProfileLengthService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/ProfileLength/ProfileLengthService.php)                                                                                                                                                                                                                                                                                                                  | Расчёт «длины» анкеты (набора заполненных полей).                       |
| [`ColorSchemeService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Support/ColorSchemeService.php), [`ColorSchemeFileManager`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Support/ColorSchemeFileManager.php)                                                                                                                                                                                               | Цветовые схемы и файлы логотипов партнёров.                             |
| [`BankUIService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BankUI/BankUIService.php)                                                                                                                                                                                                                                                                                                                                       | Настройки UI банков (иконки, тексты, оформление).                       |

### События и логирование

| Класс                                                                                                                                                                                                                  | Роль                                                                |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| [`EventManager`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/EventScheduler/EventManager.php)                                                                                             | Планировщик отложенных событий: поиск по параметрам, откат события. |
| [`EventTagManager`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/EventScheduler/EventTagManager.php)                                                                                       | Теги событий планировщика.                                          |
| [`LogService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Logger/LogService.php), [`EventLoggerFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Logger/EventLoggerFactory.php) | Логирование и фабрика логгеров по типу события/сущности.            |
| [`RequestIdManager`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Utils/RequestId/RequestIdManager.php)                                                                                                | Сквозной Request-ID для трассировки запросов.                       |
| [`SuccessResolver`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Utils/Curl/SuccessResolver/SuccessResolver.php) + `HttpOkSuccessResolver`, `HttpRevoSuccessResolver`                      | Разбор успешности HTTP-ответов внешних API.                         |

### Внешние данные и проверки

| Класс                                                                                                                                     | Роль                                                                       |
| ----------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| [`ElasticSearchConnector`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Elastic/ElasticSearchConnector.php)               | Аналитика из Elastic: ошибки банков, среднее количество заказов за недели. |
| [`DadataCleanAddress`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Api/Dadata/DadataCleanAddress.php), `DadataCleanName` | Нормализация адресов и ФИО через DaData.                                   |
| `Form\Visible\*Checker` (`FormVisibleChecker`, `ByBankChecker`, `EqualChecker`, `LogicalAndChecker` и др.)                                | Проверки видимости полей формы по значениям и банку.                       |
| [`Validator`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/Validators/Validator.php) + набор валидаторов настроек         | Валидация настроек магазина, точки, цветовых схем и т.п.                   |
| `EntityTest\*Checker` (`CommonTestChecker`, `ValueTestChecker`, `DecimalDownPaymentChecker`)                                              | Тестовые проверки сущностей (тестовые группы/фиктивные заказы).            |

## Связь с API

| Action                            | Что используют сервисы домена                                                                      |
| --------------------------------- | -------------------------------------------------------------------------------------------------- |
| `config` (виджет)                 | `ColorSchemeService`, `BankUIService`, `CommonSettingsService` — брендирование и параметры виджета |
| `start/start`, `get-current-step` | `CodesRouteProvider`, `ContinueGateService`, `CodesService`                                        |
| `settings/get-services-texts`     | `CommonSettingsService`, `JsonProfileService`                                                      |
| `status/get`                      | `OrderRepository`, `AppRepository`, `EventManager`                                                 |
| `api-log/send-error`              | `LogService`, `RequestIdManager`                                                                   |
| Все action                        | `OmService`, `LogService`, `RequestIdManager` — сквозные платформенные зависимости                 |

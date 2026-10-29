---
type: service-domain
title: "Сервисы: Авторизация, доступы и безопасность"
tags: [backend, services, auth, security, passport, dc-id, roles, bitrix, dc-module]
owner: "Команда ЛК"
updated: "2026-10-29"
---

# Авторизация, доступы и безопасность

## История изменений
| Дата       | Задача                                                 | Автор                                                                            | Изменение                        |
| ---------- | ------------------------------------------------------ | -------------------------------------------------------------------------------- | -------------------------------- |
| 29.10.2026 | [CASH-4817a](https://jira.mvideo.ru/browse/CASH-4817a) | [Еременко Светлана Александровна(ДК)](https://wiki.mvideo.ru/display/~SEremenko) | Создание документации (через ИИ) |

## Назначение домена

Домен объединяет всё, что связано с доступом: токены для внешних API (партнёры, CIA), проверка безопасности клиента и паспортных данных, права на заказ в разных каналах, регистрация клиента, роли/группы и доступные типы подписания.

## Сервисы

### Токены и авторизация внешних API

| Класс                                                                                                                                                  | Роль                                                                        |
| ------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------- |
| [`PartnerAuthService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/PartnerAuth/PartnerAuthService.php)                    | Получение access-токена партнёра (с кэшированием/обновлением).              |
| [`PartnerAuthSender`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/PartnerAuth/PartnerAuthSender.php)                      | Транспорт запросов на выдачу и обновление токена партнёра.                  |
| [`AuthorizationService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/services_api/Api/Authorization/PartnerAuth/AuthorizationService.php) | Авторизация API-запросов к банкам: клиент, конфигурация, токен авторизации. |
| [`CiaAuthService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/CiaAuth/CiaAuthService.php)                                | Получение токена внешнего сервиса CIA.                                      |
| [`CiaAuthSender`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/CiaAuth/CiaAuthSender.php)                                  | Транспорт запросов создания и проверки токена CIA.                          |
| [`DcIdChecker`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/DcIdClient/DcIdChecker.php)                                               | Проверка клиента/заявки во внешнем DcID (в т.ч. режим RTDM).                |

### Безопасность клиента и заказа

| Класс                                                                                                                                                                                                                                                                                                                                                                                                                        | Роль                                                                                           |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| [`CustomerSecurityService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Secure/Customer/CustomerSecurityService.php)                                                                                                                                                                                                                                                                            | Определяет тип безопасности клиента и набор критериев проверки, сортирует критерии.            |
| [`CustomerPassportCheckService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Secure/Customer/CustomerPassportCheckService.php)                                                                                                                                                                                                                                                                  | Инициирует и обрабатывает проверку паспорта по сканам и результатам распознавания.             |
| [`EsppCheckService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Secure/Customer/EsppCheckService.php), [`BaseCheckService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Secure/Customer/BaseCheckService.php)                                                                                                                                                     | Проверка по ЕСПП и базовые проверки клиента.                                                   |
| [`OrderSecurityService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Secure/Order/OrderSecurityService.php)                                                                                                                                                                                                                                                                                     | Проверка подозрительности клиента по онлайн- и офлайн-заказу, фиксация даты просмотра агентом. |
| [`OrderSecurityDecisionService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Secure/Order/OrderSecurityDecisionService.php)                                                                                                                                                                                                                                                                     | Сохранение решения по безопасности заказа.                                                     |
| [`ProfileSecureFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Secure/Order/ProfileSecureFactory.php), [`ProfileSecureChecker`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Secure/Order/ProfileSecureChecker.php)                                                                                                                                           | Проверка прав на заказ в офлайн-анкетах 3.0/3.1.                                               |
| [`OnlineSecureChecker`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Secure/Order/OnlineSecureChecker.php), [`OfflineSecureChecker`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Secure/Order/OfflineSecureChecker.php), [`OnlineLkSecureChecker`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Secure/Order/OnlineLkSecureChecker.php) | Проверки доступа к заказу в онлайне, офлайне и ЛК.                                             |
| [`CheckoutSecurityManager`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/SecurityManager/CheckoutSecurityManager.php)                                                                                                                                                                                                                                                                            | Единая точка проверки безопасности на этапе чекаута.                                           |

### Пользователи, роли, подписание

| Класс                                                                                                                                                                                                                                        | Роль                                          |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| [`ClientRegisterService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/UserExtended/ClientRegisterService.php)                                                                                                   | Регистрация клиента как пользователя системы. |
| [`UserGroupService`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/UserGroupService.php), [`UserRoleFactory`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/UserRoles/Factory/UserRoleFactory.php) | Группы пользователей и фабрика ролей.         |
| [`AvailableSign`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/AvailableSign/AvailableSign.php), [`AvailableSignStep`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/AvailableSign/AvailableSignStep.php)     | Доступные типы подписания и шаги подписания.  |
| [`CaptchaEventHandler`](../../../../../newprod.direct-сredit.ru/local/modules/dc/lib/BitrixModel/Authentication/CaptchaEventHandler.php)                                                                                                     | Обработка капчи при авторизации/регистрации.  |

## Связь с API

| Action                                          | Что используют сервисы домена                                                                       |
| ----------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| `auth/form`                                     | `AvailableSign`, `CustomerSecurityService`, `DcIdChecker` — форма и предварительные проверки        |
| `auth/check`                                    | `CustomerSecurityService`, `CustomerPassportCheckService`, `CheckoutSecurityManager`, `DcIdChecker` |
| `passport-upload/*`                             | `CustomerPassportCheckService`, `EsppCheckService`, `OrderSecurityService`                          |
| `sms/send`                                      | `AvailableSign`, `CaptchaEventHandler` — подтверждение телефона                                     |
| `verify/*`                                      | `OrderSecurityService`, `OrderSecurityDecisionService`, `CheckoutSecurityManager`                   |
| `partner/set-url`, `partner/simplified-set-url` | `PartnerAuthService`, `ClientRegisterService` — токен и регистрация партнёрского клиента            |
| `sign/*`                                        | `AvailableSign`, `AvailableSignStep` — доступные типы подписания                                    |
| `lk/confirm`, `lk/refuse`                       | `OnlineLkSecureChecker`, `ProfileSecureChecker`, `UserGroupService` — доступ к заказу из ЛК         |

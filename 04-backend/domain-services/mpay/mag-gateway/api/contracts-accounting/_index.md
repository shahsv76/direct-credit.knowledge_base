---
type: page
title: "Contracts & Accounting API (MAG)"
tags: [mag, mpay, api, contracts, accounting]
owner: "Команда ЛК"
updated: "2026-10-23"
---

# contracts-accounting

Группа методов **Contracts & Accounting API** сервиса `mpay-api-gateway` (MAG). Отвечает за работу с договорами клиента: получение списка и карточки договора, фильтрацию по типам предложений и статусам, договоры на главный экран, график платежей и платёжные реквизиты, а также управление статусом согласия и получение ссылок на пакет документов.

## Методы

| Метод                                       | HTTP | Эндпоинт                      | Файл                                                     |
| ------------------------------------------- | ---- | ----------------------------- | -------------------------------------------------------- |
| **[DEPRECATED]** Чтение списка договоров v1 | POST | `/api/v1/contracts/read`      | [`contracts-read-v1.md`](./contracts-read-v1.md)         |
| Чтение списка договоров v2                  | POST | `/api/v2/contracts/read`      | [`contracts-read-v2.md`](./contracts-read-v2.md)         |
| Карточка договора v1                        | POST | `/api/v1/contract/read`       | [`contract-read.md`](./contract-read.md)                 |
| Карточка договора v2                        | POST | `/api/v2/contract/read`       | [`contract-read-v2.md`](./contract-read-v2.md)           |
| Список договоров с фильтрацией              | GET  | `/api/v1/contracts`           | [`contracts.md`](./contracts.md)                         |
| Все договоры (включая закрытые)             | GET  | `/api/v1/contracts/all`       | [`contracts-all.md`](./contracts-all.md)                 |
| Детали договора                             | GET  | `/api/v1/contract`            | [`contract-details.md`](./contract-details.md)           |
| Договоры на главный экран                   | GET  | `/api/v1/homeScreenContracts` | [`home-screen-contracts.md`](./home-screen-contracts.md) |
| Типы предложений                            | GET  | `/api/v1/offer/types`         | [`offer-types.md`](./offer-types.md)                     |
| График платежей                             | GET  | `/api/v1/schedule/read`       | [`schedule-read.md`](./schedule-read.md)                 |
| Данные платежа                              | GET  | `/api/v1/payment/read`        | [`payment-read.md`](./payment-read.md)                   |
| График по заявке                            | GET  | `/api/v1/schedule`            | [`schedule.md`](./schedule.md)                           |
| Статус согласия                             | POST | `/api/v1/agreement/status`    | [`agreement-status.md`](./agreement-status.md)           |
| Ссылки на пакет документов                  | GET  | `/api/v1/documents/url`       | [`documents-url.md`](./documents-url.md)                 |

## Справочники

| Документ                             | Назначение                                      |
| ------------------------------------ | ----------------------------------------------- |
| [`general-dto.md`](./general-dto.md) | Общие DTO (используются в двух и более методах) |
| [`enums.md`](./enums.md)             | ENUM, константы и коды                          |

## Общие страницы

- [Общие заголовки (MAG)](../../common-headers.md)
- [Общие ошибки (MAG)](../../common-errors.md)
- [Метрики (MAG)](../../metrics.md)
- [Конфигурация (MAG)](../../config.md)

## Смежные группы

Методы платежей (`/api/v1/contract/repayment`, `/api/v1/contract/early/repayment`, `/api/v1/contract/bank/logo/get`, `/api/v1/bestToPay`, `/api/v2/bestToPay`, `/api/v3/bestToPay`) описаны в группе **Best2Pay & Payments API**.

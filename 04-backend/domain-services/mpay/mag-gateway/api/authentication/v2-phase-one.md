---
type: method
title: "MAG: Первая фаза авторизации v2 (phase_one)"
tags: [mag, mpay, api, auth]
owner: "Команда ЛК"
updated: "2026-10-22"
---

# POST /api/v2/authentication/phase_one (MAG)

## Описание
| Тип запроса | Название метода | Описание метода                                                                                                             | URL                                | Тип  |
| ----------- | --------------- | --------------------------------------------------------------------------------------------------------------------------- | ---------------------------------- | ---- |
| POST        | PhaseOne        | Первая фаза авторизации (v2): определяет сценарий входа — по PIN-коду (если пароль и СОПД актуальны) или через OTP (иначе). | `/api/v2/authentication/phase_one` | SYNC |

## Входные параметры
### Header
| Название                                           | Тип | Обязательность | Описание           | Пример |
| -------------------------------------------------- | --- | -------------- | ------------------ | ------ |
| 🔗 [Общие заголовки (MAG)](../../common-headers.md) | —   | —              | См. общую страницу | —      |

### Body
| Название | Тип    | Обязательность | Описание                       | Пример                             |
| -------- | ------ | -------------- | ------------------------------ | ---------------------------------- |
| `token`  | String | Да             | Токен AUTH1, полученный от BFF | `0abfb32e303d35bc68cb3e70018c6551` |

## Выходные параметры
В зависимости от сценария возвращается один из двух вариантов.

**Вариант 1 — вход через OTP (нет пароля или СОПД):**
| Название                                                                                        | Тип    | Обязательность | Описание                             | Пример                                           |
| ----------------------------------------------------------------------------------------------- | ------ | -------------- | ------------------------------------ | ------------------------------------------------ |
| 🔗 [AuthenticationPhaseOneXTokenResponse](./general-dto.md#authenticationphaseonextokenresponse) | Object | Да             | Токен OTP-сессии и список соглашений | `{ "xOTPToken": "eyJ...", "agreements": [...] }` |

**Вариант 2 — вход через PIN (пароль и СОПД актуальны):**
| Название   | Тип    | Обязательность | Описание                         | Пример          |
| ---------- | ------ | -------------- | -------------------------------- | --------------- |
| `pinToken` | String | Да             | Токен для подтверждения PIN-кода | `eyJhbGciOi...` |

## Коды ответа
| Код ответа | Описание              | Причина                                                                                                                  |
| ---------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `201`      | CREATED               | Сценарий определён, токен (OTP или PIN) сформирован                                                                      |
| `400`      | BAD_REQUEST           | Некорректное тело запроса                                                                                                |
| `403`      | FORBIDDEN             | Превышен лимит запросов OTP — объект [`AuthenticationPhaseOneBadResponse`](./enums.md#authenticationphaseonebadresponse) |
| `500`      | INTERNAL_SERVER_ERROR | Ошибка внешних сервисов (статус, СОПД, блокировка SMS, соглашения, OTP, PIN)                                             |

🔗 *Все общие коды ошибок см. на странице [Общие ошибки (MAG)](../../common-errors.md).*

## Внешние сервисы
| Сервис                   | URL (env)            | Метод | Эндпоинт                          |
| ------------------------ | -------------------- | ----- | --------------------------------- |
| Authentication Service ⏳ | `AUTHENTICATION_URL` | POST  | `/api/v3/check_status`            |
| Client API ⏳             | `MFO_API_CLIENT_URL` | POST  | `/v1/clients/check_sopd`          |
| Authentication Service ⏳ | `AUTHENTICATION_URL` | POST  | `/api/v3/check_sms_block`         |
| Offer Service ⏳          | `OFFER_HTTP_URL`     | POST  | `/v1/getCommonDocsInfoByDocTypes` |
| OTP Service ⏳            | `OTP_URL`            | POST  | `/otp/v2/send`                    |
| Authentication Service ⏳ | `AUTHENTICATION_URL` | POST  | `/api/v3/fact_sms_send`           |
| Authentication Service ⏳ | `AUTHENTICATION_URL` | POST  | `/api/v3/create_pin_token`        |

## Метрика
| Параметр        | Значение                                                     |
| --------------- | ------------------------------------------------------------ |
| **Имя метрики** | [`Login User`](../../metrics.md) (`metrics.MetricLoginUser`) |
| **Топик**       | `auth`                                                       |
| **vendor**      | `mpay-api-gateway handler.Authentication.PhaseOne() N`       |

## Логика работы
1. **Приём и валидация:** MAG принимает запрос и валидирует поле `token`.
2. **Проверка статуса:** Вызывает Authentication Service (`/api/v3/check_status`) — получает `clientID`, `phone`, `partnerID`, признак блокировки и наличия пароля.
3. **Проверка СОПД:** Вызывает Client API (`/v1/clients/check_sopd`) для проверки актуальности соглашений.
4. **Выбор сценария:**
   - **OTP-сценарий** (нет пароля или СОПД не актуальны): проверяет блокировку SMS (`/api/v3/check_sms_block`), получает соглашения (Offer Service), отправляет OTP (`/otp/v2/send`), фиксирует факт отправки (`/api/v3/fact_sms_send`). При превышении лимита OTP (`429`) — ответ `403` с [`AuthenticationPhaseOneBadResponse`](./enums.md#authenticationphaseonebadresponse). Возвращает [`AuthenticationPhaseOneXTokenResponse`](./general-dto.md#authenticationphaseonextokenresponse).
   - **PIN-сценарий** (пароль и СОПД актуальны): вызывает Authentication Service (`/api/v3/create_pin_token`) и возвращает `pinToken`.
5. **Формирование ответа:** Возвращает клиенту токен соответствующего сценария с кодом `201`.

## Request
```bash
curl -X POST "https://gateway.direct-pay.ru/api/v2/authentication/phase_one" \
  -H "Content-Type: application/json" \
  -d '{ "token": "0abfb32e303d35bc68cb3e70018c6551" }'
```

---
type: page
title: "Общие DTO (Authentication)"
tags: [mag, mpay, dto, auth]
owner: "Команда ЛК"
updated: "2026-10-22"
---

# Общие DTO (Authentication)

Здесь описаны структуры данных (DTO), которые используются более чем в одном методе авторизации (Authentication API).

### AuthenticationCheckTokenRequest
Используется в методах: `POST /authentication/check`, `POST /authentication/check_short`.

| Название | Тип    | Обязательность | Описание                                       | Пример                    |
| -------- | ------ | -------------- | ---------------------------------------------- | ------------------------- |
| `token`  | String | Да             | Токен аутентификации (auf1), полученный от BFF | `eyJhbGciOiJIUzI1NiJ9...` |
| `secret` | String | Нет            | Секрет аутентификации (для внутренних вызовов) | —                         |

### AuthenticationCheckTokenResponse
Возвращается сервисом Authentication и используется в методах: `POST /authentication/check`, `POST /authentication/check_short`, `POST /api/v1/authentication/phase_one`.

| Название   | Тип    | Обязательность | Описание               | Пример        |
| ---------- | ------ | -------------- | ---------------------- | ------------- |
| `clientID` | String | Да             | Идентификатор клиента  | `client-123`  |
| `phone`    | String | Да             | Номер телефона клиента | `79990000000` |

### OtpCreateRequest
Используется в методах: `POST /authentication/check`, `POST /api/v1/authentication/phase_one`.

| Название   | Тип    | Обязательность | Описание                                     | Пример        |
| ---------- | ------ | -------------- | -------------------------------------------- | ------------- |
| `clientID` | String | Да             | Идентификатор клиента                        | `client-123`  |
| `phone`    | String | Да             | Номер телефона клиента                       | `79990000000` |
| `from`     | String | Нет            | Имя отправителя (для определенных партнеров) | `DCREDITPAY`  |
| `text`     | String | Нет            | Текст сообщения (для определенных партнеров) | `код...`      |

### OtpCreateResponse
Возвращается сервисом OTP и используется в методах: `POST /authentication/check`, `POST /authentication/resend`, `POST /api/v1/authentication/phase_one`.

| Название    | Тип    | Обязательность | Описание               | Пример          |
| ----------- | ------ | -------------- | ---------------------- | --------------- |
| `code`      | Int    | Да             | Код ответа сервиса OTP | `201`           |
| `xOTPToken` | String | Да             | Токен OTP-кода         | `otp-token-...` |
| `err`       | String | Нет            | Текст ошибки           | `""`            |

### AuthenticationCheckBlockSMSRequest
Используется в методах: `POST /authentication/resend`, `POST /api/v2/authentication/phase_one`.

| Название    | Тип    | Обязательность | Описание               | Пример          |
| ----------- | ------ | -------------- | ---------------------- | --------------- |
| `clientID`  | String | Нет            | Идентификатор клиента  | `client-123`    |
| `phone`     | String | Нет            | Номер телефона         | `79990000000`   |
| `partnerID` | String | Нет            | Идентификатор партнера | `dcpay-mobile`  |
| `xOTPToken` | String | Да             | Токен OTP              | `otp-token-...` |

### PreAgreement
Соглашение (СОПД), которое клиент подписывает при входе. Используется в методах: `POST /api/v1/authentication/phase_one`, `POST /api/v2/authentication/phase_one`, `POST /api/v1/authentication/phase_otp`, `POST /api/v2/authentication/phase_otp`, `POST /api/v2/authentication/forgot_password`.

| Название      | Тип     | Обязательность | Описание                           | Пример  |
| ------------- | ------- | -------------- | ---------------------------------- | ------- |
| `docID`       | Integer | Да             | Идентификатор документа-соглашения | `12345` |
| `docTypeID`   | Integer | Да             | Идентификатор типа документа       | `1`     |
| `consentFlag` | Boolean | Да             | Признак согласия клиента           | `true`  |

### AuthenticationPhaseOneXTokenResponse
Возвращается на первой фазе авторизации. Используется в методах: `POST /api/v1/authentication/phase_one`, `POST /api/v2/authentication/phase_one`.

| Название     | Тип                                                  | Обязательность | Описание                                      | Пример                                                      |
| ------------ | ---------------------------------------------------- | -------------- | --------------------------------------------- | ----------------------------------------------------------- |
| `xOTPToken`  | String                                               | Да             | Токен OTP-сессии, созданной при отправке кода | `eyJhbGciOi...`                                             |
| `agreements` | Array<[PreAgreement](./general-dto.md#preagreement)> | Да             | Список соглашений (СОПД) для подписания       | `[{ "docID": 12345, "docTypeID": 1, "consentFlag": true }]` |

### AuthenticationPhaseOTPRequest
Передаётся на второй фазе авторизации. Используется в методах: `POST /api/v1/authentication/phase_otp`, `POST /api/v2/authentication/phase_otp`.

| Название     | Тип                                                  | Обязательность | Описание                                    | Пример                                                      |
| ------------ | ---------------------------------------------------- | -------------- | ------------------------------------------- | ----------------------------------------------------------- |
| `code`       | String                                               | Да             | Код из СМС                                  | `111111`                                                    |
| `xOTPToken`  | String                                               | Да             | Токен OTP-сессии, полученный на первой фазе | `eyJhbGciOi...`                                             |
| `agreements` | Array<[PreAgreement](./general-dto.md#preagreement)> | Да             | Список подписанных соглашений (СОПД)        | `[{ "docID": 12345, "docTypeID": 1, "consentFlag": true }]` |

### SavePasswordRequest
Передаётся при сохранении пароля. Используется в методах: `POST /api/v2/authentication/save_password`, `POST /api/v2/authentication/save_password_retype`.

| Название      | Тип    | Обязательность | Описание                                          | Пример          |
| ------------- | ------ | -------------- | ------------------------------------------------- | --------------- |
| `newPinToken` | String | Да             | PIN-токен, полученный после подтверждения OTP     | `eyJhbGciOi...` |
| `password`    | String | Да             | Пароль (PIN-код) клиента, который нужно сохранить | `1234`          |
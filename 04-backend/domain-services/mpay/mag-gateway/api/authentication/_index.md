# authentication

Группа методов **Authentication API** сервиса `mpay-api-gateway` (MAG). Отвечает за авторизацию клиента mPay: проверку токенов, выдачу OTP, подтверждение кода, работу с PIN-кодом и восстановление пароля.

## Методы

| Метод                                     | HTTP | Эндпоинт                                      | Файл                                                         |
| ----------------------------------------- | ---- | --------------------------------------------- | ------------------------------------------------------------ |
| Проверка токена AUTH1                     | POST | `/authentication/check`                       | [`check.md`](./check.md)                                     |
| Короткая проверка токена с выдачей сессии | POST | `/authentication/check_short`                 | [`check-short.md`](./check-short.md)                         |
| Повторная отправка OTP                    | POST | `/authentication/resend`                      | [`resend.md`](./resend.md)                                   |
| Первая фаза авторизации v1                | POST | `/api/v1/authentication/phase_one`            | [`v1-phase-one.md`](./v1-phase-one.md)                       |
| Вторая фаза авторизации v1 (OTP)          | POST | `/api/v1/authentication/phase_otp`            | [`v1-phase-otp.md`](./v1-phase-otp.md)                       |
| Первая фаза авторизации v2                | POST | `/api/v2/authentication/phase_one`            | [`v2-phase-one.md`](./v2-phase-one.md)                       |
| Вторая фаза авторизации v2 (OTP)          | POST | `/api/v2/authentication/phase_otp`            | [`v2-phase-otp.md`](./v2-phase-otp.md)                       |
| Сохранение пароля                         | POST | `/api/v2/authentication/save_password`        | [`v2-save-password.md`](./v2-save-password.md)               |
| Проверка и сохранение пароля с повтором   | POST | `/api/v2/authentication/save_password_retype` | [`v2-save-password-retype.md`](./v2-save-password-retype.md) |
| Авторизация по паролю                     | POST | `/api/v2/authentication/phase_auth`           | [`v2-phase-auth.md`](./v2-phase-auth.md)                     |
| Восстановление пароля                     | POST | `/api/v2/authentication/forgot_password`      | [`v2-forgot-password.md`](./v2-forgot-password.md)           |

## Справочники

| Документ                             | Назначение                                         |
| ------------------------------------ | -------------------------------------------------- |
| [`general-dto.md`](./general-dto.md) | Общие DTO (используются в двух и более методах)    |
| [`enums.md`](./enums.md)             | ENUM, константы, коды и специальные объекты ошибок |

## Общие страницы

- [Общие заголовки (MAG)](../../common-headers.md)
- [Общие ошибки (MAG)](../../common-errors.md)
- [Метрики (MAG)](../../metrics.md)

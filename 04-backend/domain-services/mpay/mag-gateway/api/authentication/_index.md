# authentication

Группа методов **Authentication API** сервиса `mpay-api-gateway` (MAG). Отвечает за авторизацию клиента mPay: проверку токенов, выдачу OTP, подтверждение кода, работу с PIN-кодом и восстановление пароля.

## Методы

| Метод                                     | HTTP | Эндпоинт                                      | Файл                                                         |
| ----------------------------------------- | ---- | --------------------------------------------- | ------------------------------------------------------------ |
| Проверка токена AUTH1                     | POST | `/authentication/check`                       | [`check.md`](./check.md)                                     |
| Короткая проверка токена с выдачей сессии | POST | `/authentication/check_short`                 | [`check-short.md`](./check-short.md)                         |
| Повторная отправка OTP                    | POST | `/authentication/resend`                      | [`resend.md`](./resend.md)                                   |
| Первая фаза авторизации v1                | POST | `/api/v1/authentication/phase_one`            | [`phase-one-v1.md`](./phase-one-v1.md)                       |
| Вторая фаза авторизации v1 (OTP)          | POST | `/api/v1/authentication/phase_otp`            | [`phase-otp.md`](./phase-otp-v1.md)                          |
| Первая фаза авторизации v2                | POST | `/api/v2/authentication/phase_one`            | [`phase-one-v2.md`](./phase-one-v2.md)                       |
| Вторая фаза авторизации v2 (OTP)          | POST | `/api/v2/authentication/phase_otp`            | [`phase-otp-v2.md`](./phase-otp-v2.md)                       |
| Сохранение пароля                         | POST | `/api/v2/authentication/save_password`        | [`save-password-v2.md`](./save-password-v2.md)               |
| Проверка и сохранение пароля с повтором   | POST | `/api/v2/authentication/save_password_retype` | [`save-password-retype-v2.md`](./save-password-retype-v2.md) |
| Авторизация по паролю                     | POST | `/api/v2/authentication/phase_auth`           | [`phase-auth-v2.md`](./phase-auth-v2.md)                     |
| Восстановление пароля                     | POST | `/api/v2/authentication/forgot_password`      | [`forgot-password-v2.md`](./forgot-password-v2.md)           |

## Справочники

| Документ                             | Назначение                                         |
| ------------------------------------ | -------------------------------------------------- |
| [`general-dto.md`](./general-dto.md) | Общие DTO (используются в двух и более методах)    |
| [`enums.md`](./enums.md)             | ENUM, константы, коды и специальные объекты ошибок |

## Общие страницы

- [Общие заголовки (MAG)](../../common-headers.md)
- [Общие ошибки (MAG)](../../common-errors.md)
- [Метрики (MAG)](../../metrics.md)

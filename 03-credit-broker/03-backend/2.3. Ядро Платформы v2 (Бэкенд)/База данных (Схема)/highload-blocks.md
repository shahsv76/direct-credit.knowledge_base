---
type: db-schema
title: "Highload-блоки (HL) и справочники"
tags: [db, bitrix, highloadblock, schema]
owner: "Команда ЛК"
updated: "2026-10-28"
---

# Highload-блоки (HL) и справочники

## История изменений
| Дата | Задача | Автор | Изменение |
|---|---|---|---|
| 28.10.2026 | [CASH-4817a](https://jira.mvideo.ru/browse/CASH-4817a) | [Еременко Светлана Александровна(ДК)](https://wiki.mvideo.ru/display/~SEremenko)| Создание документации (через ИИ) |

## Назначение

Highload-блоки используются как справочники/ENUM-ы: пол, образование, социальный и семейный статус, гражданство, страховые продукты, типы товаров, банковские сервисы, шаблоны PDF, скоринг и подписание.

В коде нет жёсткого маппинга «константа → таблица `b_hlbd_*`»: сущность получается динамически по ID через `HighloadBlockTable::getById($id)` + `compileEntity()`. Поэтому **имена таблиц HL-блоков в репозитории отсутствуют** — восстановлению подлежат только ID и назначение.

Актуальный реестр: [`local/modules/dc/lib/HBHelper.php`](../../../local/modules/dc/lib/HBHelper.php:24) — класс `DC\HBHelper`.
Устаревший реестр: [`bitrix/php_interface/init.php`](../../../bitrix/php_interface/init.php:131) — `define('HB_...')`.

## Типовая структура справочника

Все «словарные» HL-блоки имеют одинаковый набор UF-полей (подтверждено на `hb_citizenship`, [`CitizenshipTable.php`](../../../local/modules/dc/lib/Iblock/CitizenshipTable.php:26) и использовании в `HbInsurance`):

| Поле | Назначение |
|---|---|
| `ID` | Первичный ключ |
| `UF_XML_ID` | Внешний идентификатор — **именно он хранится в свойствах инфоблоков и в `dc_*` таблицах** |
| `UF_NAME` | Отображаемое наименование |
| `UF_DEF` | Признак значения по умолчанию (`Y`/`N`) |
| `UF_SORT` | Сортировка в выпадающих списках |
| `UF_FILE` / `UF_DESCRIPTION` | Иконка и описание (используются не везде) |

Специфичные поля: `hb_citizenship` дополнительно содержит `UF_OKSM` (ОКСМ), `UF_ISO` (ISO-код страны), `UF_PASSPORT_MASK` (маска паспорта).

## Способы доступа

| Способ | Класс | Особенности |
|---|---|---|
| Прямой | `HBHelper::getEntityClass($hbId)` | Инициализация + статический кеш `self::$hBs` в рамках запроса |
| Справочник | `HBHelper::getDictionary($hbId, $key, $value)` | Возвращает `[UF_XML_ID => UF_NAME]` |
| Одно значение | `HBHelper::getDictionaryValue($hbId, $xmlId, 'UF_NAME')` | Поиск по `UF_XML_ID` |
| Список по XML_ID | `HBHelper::getHbItems($hbId, $xmlId)` | Возвращает полные записи |
| Кешируемый | [`CommonCachedHbManager`](../../../local/modules/dc/lib/BitrixModel/CachedProxy/CommonCachedHbManager.php:18) | Кеш `hblock_entity_id` / `hblock_values`, TTL 1 час |
| API-автокомплит | [`HbReference`](../../../local/modules/dc/lib/BitrixModel/AutocompleteApi/HbReference.php:13), [`HbReferenceProxy`](../../../local/modules/dc/lib/BitrixModel/AutocompleteApi/HbReferenceProxy.php:16) | Поиск значений для UI |

## Реестр HL-блоков (актуальный, `DC\HBHelper`)

| Константа | ID | Назначение | Класс-обёртка / использование |
|---|---|---|---|
| `HB_SEX` | 1 | Пол | [`BxProfile`](../../../local/modules/dc/lib/BitrixModel/BxProfile.php:26): `MALE_ALIAS = 1IG8Y7SX`, `FEMALE_ALIAS = YRFMIQES` |
| `HB_EDUCATION` | 2 | Образование | свойство `EDUCATION` анкеты |
| `HB_MARITAL_STATUS` | 3 | Семейное положение | [`HbMaritalStatus`](../../../local/modules/dc/lib/BitrixModel/HbMaritalStatus.php:16) |
| `HB_ACTIVITY_TYPE` | 4 | Вид деятельности | анкета |
| `HB_BUSINESS_TYPE` | 5 | Тип бизнеса | свойство `BUSINESS_TYPE` |
| `HB_POSITION_TYPE` | 6 | Тип должности | анкета |
| `HB_COMPANY_OWNERSHIP` | 7 | Форма собственности компании | анкета |
| `HB_SOCIAL_STATUS` | 8 | Социальный статус | [`HbSocialStatus`](../../../local/modules/dc/lib/BitrixModel/HbSocialStatus.php:15) |
| `HB_PROPERTY_TYPE` | 9 | Тип имущества | анкета |
| `HB_REALESTATE` | 10 | Недвижимость | свойство `REAL_ESTATE` |
| `HB_AUTOBRAND` | 11 | Марка автомобиля | свойство `AUTO_BRAND` |
| `HB_EMPLOYEES` | 13 | Численность сотрудников | анкета ЮЛ |
| `HB_INSURANCE` | 14 | Страховые продукты | [`HbInsurance`](../../../local/modules/dc/lib/BitrixModel/HbInsurance.php:23), [`HbInsuranceManager`](../../../local/modules/dc/lib/BitrixModel/HbInsuranceManager.php:21) |
| `HB_SOURCE_INCOME` | 16 | Источник дохода | [`HbSourceIncome`](../../../local/modules/dc/lib/BitrixModel/HbSourceIncome.php:16) |
| `HB_BANK_SERVICE` | 18 | Банковские услуги | `NotifierTelegram\HBChangesNotifier::HLBLOCK_BANK_SERVICE_NAME = 'BankService'` |
| `HB_ADDITIONAL_PRODUCT` | 19 | Дополнительные продукты (карты) | [`HbCardsManager`](../../../local/modules/dc/lib/BitrixModel/HbCardsManager.php:20), [`AdditionalProductService`](../../../local/modules/dc/lib/AdditionalProductService.php:463) |
| `HB_CUSTOMER_ESTIMATE` | 21 | Оценка клиента | [`HbCustomerEstimate`](../../../local/modules/dc/lib/BitrixModel/HbCustomerEstimate.php:18) |
| `HB_RELATION_DEGREE` | 23 | Степень родства | анкета |
| `HB_DEPENDENTS` | 24 | Иждивенцы | анкета |
| `HB_PLASTIC_CARD` | 28 | Пластиковые карты | доп. продукты |
| `HB_COUNTRY_OF_BIRTH` | 31 | Страна рождения | анкета |
| `HB_AGENT_STATUS` | 32 | Статусы агентов | [`BxAgent`](../../../local/modules/dc/lib/BitrixModel/BxAgent.php:469) |
| `HB_OUTLET_STATUS` | 34 | Статусы торговых точек | ТТ |
| `HB_ADDITIONAL_INCOME` | 35 | Дополнительный доход | анкета |
| `HB_ADDITIONAL_EXPENSE` | 36 | Дополнительный расход | анкета |
| `HB_JOINT_RESIDENCE` | 37 | Совместное проживание | анкета |
| `HB_WORK_POSITION` | 39 | Должность | анкета |
| `HB_DELIVERY_ADDRESS` | 40 | Адреса доставки | логистика |
| `HB_ANOTHER_CREDIT` | 42 | Наличие других кредитов | анкета |
| `HB_SAMA` | 43 | Типы САМА (рассрочка) | [`HbSamaModule`](../../../local/modules/dc/lib/BitrixModel/HbSamaModule.php:15), [`SamaTypeChecker`](../../../local/modules/dc/lib/BitrixModel/Sama/SamaTypeChecker.php:71) |
| `HB_AUTOPURCHASE` | 45 | Авто-покупка | программы |
| `HB_LEGAL_FORM_STATUSES` | 46 | Статусы юрформ | анкета ЮЛ |
| `HB_CALL_TIME` | 47 | Время для звонка | анкета |
| `HB_TZ` | 48 | Часовой пояс | [`CallCenter\Card`](../../../local/modules/dc/lib/CallCenter/Card.php:156) |
| `HB_REGISTRATION_TYPE` | 49 | Тип регистрации | анкета |
| `HB_WORK_FORM` | 50 | Форма занятости | анкета |
| `HB_TYPE_OF_GOOD_MAP` | 57 | Маппинг типов товаров | [`HbTypeOfGoodMap`](../../../local/modules/dc/lib/BitrixModel/HbTypeOfGoodMap.php:14), [`builder.php`](../../../local/modules/dc/online/api/json_cache/program/builder.php:229) |
| `HB_ACT_CARD` | 117 | Акты по картам | доп. продукты |
| `HB_ACT_FIELDS` | 121 | Поля актов | доп. продукты |
| `HB_SHOP_BRANCH` | 125 | Филиалы магазинов | [`BxAgent`](../../../local/modules/dc/lib/BitrixModel/BxAgent.php:1558) |
| `HB_SCORE` | 149 | Настройки скоринга | скоринг |
| `HB_SIGN_TYPE` | 157 | Настройки подписания | [`HbSignTypeSettings`](../../../local/modules/dc/lib/BitrixModel/HbSignTypeSettings.php:14) |
| `HB_BANK_CONTRACT_SETTINGS` | 161 | Настройки договоров банков | [`HbBankContractSettings`](../../../local/modules/dc/lib/BitrixModel/HbBankContractSettings.php:12) |
| `HB_PDF_TEMPLATES` | 165 | Шаблоны PDF | [`HbPdfTemplate`](../../../local/modules/dc/lib/BitrixModel/HbPdfTemplate.php:27) |

### Детально: `HB_SOCIAL_STATUS` (ID 8)

Значения — `UF_XML_ID`, зафиксированы константами в [`HbSocialStatus.php`](../../../local/modules/dc/lib/BitrixModel/HbSocialStatus.php:17):

| Константа | XML_ID | Описание |
|---|---|---|
| `SOCIAL_STATUS_HANDYMAN` | `rQrxbFSC` | Рабочий |
| `SOCIAL_STATUS_MATERNITY_LEAVE` | `okcnc8te` | В декретном отпуске |
| `SOCIAL_STATE_EMPLOYEE` | `lhPLvIuA` | Работающий |
| `SOCIAL_STATUS_COMMERCIAL` | `Kwt5ypR2` | Коммерсант |
| `SOCIAL_STATUS_PENSIONER` | `0vxo1sNN` | Пенсионер |
| `SOCIAL_STATUS_HOUSEWIFE` | `J6VBpaeU` | Домохозяйка |
| `SOCIAL_STATUS_UNEMPLOYED` | `DrBuzeAn` | Безработный |
| `SOCIAL_STATUS_STUDENT` | `1r8IYIt8` | Студент |
| `SOCIAL_STATUS_ENTREPRENEUR` | `bX2QDLZE` | Предприниматель |
| `SOCIAL_STATUS_INDIVIDUAL_ENTREPRENEUR` | `mtsip` | ИП |
| `SOCIAL_STATUS_OWNER` | `mtsowner` | Собственник |

Бизнес-правила на базе справочника: `getUnemployedStatuses()` = безработный + домохозяйка + студент; `getAllUnemployedStatuses()` добавляет пенсионера (используется в скоринге).

### Детально: `HB_SEX` (ID 1)

| XML_ID | Значение | Где |
|---|---|---|
| `1IG8Y7SX` | Мужской | `BxProfile::MALE_ALIAS` |
| `YRFMIQES` | Женский | `BxProfile::FEMALE_ALIAS` |

Сравнение выполняется через `mb_strtoupper($profile->getSex()) === BxProfile::MALE_ALIAS`.

### Детально: гражданство — `hb_citizenship`

Отдельная таблица справочника, не «словарного» формата: [`CitizenshipTable.php`](../../../local/modules/dc/lib/Iblock/CitizenshipTable.php:7), `getTableName() = hb_citizenship`. Поля: `UF_NAME`, `UF_XML_ID`, `UF_OKSM`, `UF_ISO`, `UF_PASSPORT_MASK`. Константа `RUSSIA = 'RUS'`. ID HL-блока в `HBHelper` не заведён — требует проверки, зарегистрирован ли блок в `b_hlbl` или это отдельная таблица.

### Детально: `HB_INSURANCE` (ID 14)

Справочник страховых продуктов; `UF_XML_ID` — код продукта. Бизнес-коды в [`HbInsurance.php`](../../../local/modules/dc/lib/BitrixModel/HbInsurance.php:25): группа «Финансовая защита» — `CODE_REN_FZ = '999'`, продукты `119, 120, 121, 122`; типы (`medical` / `work`) — из `dc_dict_insurance_type`. Используется совместно с `dc_insurance_replace*` (правила замены страховок) и `dc_shop_insurance_bank*` (настройки на магазин/ТТ/метод оплаты).

## Устаревший реестр в `init.php` и расхождения

[`init.php`](../../../bitrix/php_interface/init.php:131) содержит 26 определений, часть которых конфликтует с `HBHelper`:

| Константа (init.php) | ID | Проблема |
|---|---|---|
| `HB_MATERIAL_STATUS` | 3 | В `HBHelper` — `HB_MARITAL_STATUS` (правильное имя) |
| `HB_JOIN_RESIDENCE` | 37 | В `HBHelper` — `HB_JOINT_RESIDENCE` |
| `HB_CARD_LIMITS` | 189 | Только в `init.php`; в коде используется таблица `dc_…`/`CardLimitsTable` |
| `HB_CARD_SCHEMES` | 185 | То же — см. [`CardSchemesTable`](../../../local/modules/dc/lib/Table/CardSchemesTable.php) |
| `HB_SHOP_HISTORY` | 77 | Только в `init.php` (история магазинов) |
| `HB_PROFILE_LOG` | 93 | Только в `init.php` (лог изменений анкет) |
| `HB_INSURANCE_DETAIL` | 105 | Только в `init.php`; есть класс [`HbInsuranceDetail`](../../../local/modules/dc/lib/BitrixModel/HbInsuranceDetail.php:14) |
| `HB_TIMEZONE` | 48 | Дубль `HB_TZ` |
| `HB_CUSTOMER_ESTIMATE`, `HB_RELATION_DEGREE`, `HB_COUNTRY_OF_BIRTH`, `HB_WORK_POSITION`, `HB_AUTOPURCHASE`, `HB_TYPE_OF_GOOD_MAP` | 21/23/31/39/45/57 | Идентичны `HBHelper` |

В `HBHelper` также есть legacy-константы с `@todo удалить`: `HB_ADD_SERVICE = 18` и `HB_SERVICE_NOT_BANK = 19` — **конфликтуют по ID** с `HB_BANK_SERVICE = 18` и `HB_ADDITIONAL_PRODUCT = 19`.

## Классы-обёртки HL (доменный слой)

| Группа | Классы |
|---|---|
| База | [`HbBaseManager`](../../../local/modules/dc/lib/BitrixModel/HbBaseManager.php:12), [`HbBaseGood`](../../../local/modules/dc/lib/BitrixModel/HbBaseGood.php:11), [`BxBaseEntity`](../../../local/modules/dc/lib/BitrixModel/BxBaseEntity.php) |
| Анкета | `HbCitizenship`, `HbSocialStatus`, `HbMaritalStatus`, `HbSourceIncome`, `HbCustomerEstimate`, `HbJsonProfile`, `HbCustomerAddress`, `HbCustomerEstimateManager` |
| Продукты | `HbInsurance`, `HbInsuranceDetail`, `HbInsuranceNotBank`, `HbAdditionalService`, `HbTypeOfGoodMap`, `HbOfflineGood`, `HbOnlineGood` + соответствующие `*Manager` |
| Настройки/сервис | `HbShopSettings`, `HbNotifySettings`, `HbPdfTemplate`, `HbSmsStorage`, `HbImportTask`, `HbBankContractSettings`, `HbScanTypeSettings`, `HbSignTypeSettings`, `HbOrderTmpData`, `HbWaitStatusRequest`, `HbAgentOrganization`, `HbOrangeData`, `HbEntityTest`, `HbSamaModule` |
| Кеш-прокси | `CachedProxy/HbTypeOfGoodMapManagerProxy`, `HbShopSettingsManagerProxy`, `HbNotifySettingsManagerProxy`, `HbInsuranceManagerProxy`, `HbInsuranceNotBankManagerProxy`, `HbAdditionalServiceManagerProxy`, `HbEntityTestManagerProxy`, `HbOnlineGoodManagerProxy` |

## Открытые вопросы

1. Имена физических таблиц (`b_hlbd_*`) и `TABLE_NAME` для каждого ID — не хранятся в репозитории, требуют выгрузки из `b_hlbl`/`b_hlbl_field`.
2. Полный перечень UF-полей каждого справочника (кроме `hb_citizenship`) — в коде читаются только `UF_XML_ID`, `UF_NAME` и отдельные поля.
3. Наличие HL-блоков с ID `12, 15, 17, 20, 22, 25–27, 29, 30, 33, 38, 41, 44` — в константах отсутствуют, но могут существовать в БД.
4. Расхождения `HB_*` между `init.php` и `HBHelper` — требуется решение о едином источнике (техдолг).

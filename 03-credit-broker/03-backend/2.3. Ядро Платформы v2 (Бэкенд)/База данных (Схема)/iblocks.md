---
type: db-schema
title: "Инфоблоки (Информационные блоки)"
tags: [db, bitrix, iblock, schema]
owner: "Команда ЛК"
updated: "2026-10-28"
---

# Инфоблоки (Информационные блоки)

## История изменений
| Дата | Задача | Автор | Изменение |
|---|---|---|---|
| 28.10.2026 | [CASH-4817a](https://jira.mvideo.ru/browse/CASH-4817a) | [Еременко Светлана Александровна(ДК)](https://wiki.mvideo.ru/display/~SEremenko)| Создание документации (через ИИ) |

## Как устроены инфоблоки в проекте

ID инфоблоков заданы константами `IB_*` в [`bitrix/php_interface/init.php`](../../../bitrix/php_interface/init.php:96) (строки 96–130). Доступа к БД нет, поэтому **типы и полная структура каждого ИБ не восстанавливаются** — ниже детально описаны два ключевых ИБ домена «Анкета/Профиль» (`IB_PROFILE`, `IB_ANKET`), по остальным приведён реестр констант.

Чтение/запись выполняется двумя способами:

* через старый API — `CIBlockElement::GetList()`, `CIBlockElement::SetPropertyValuesEx()` (массово встречается в `api/`, `system/php/`, `local/modules/dc/lib/user.php`);
* через D7-обёртку — [`ElementTable.php`](../../../local/modules/dc/lib/Iblock/ElementTable.php:33) (`getTableName() = b_iblock_element`) и `ElementTable::createEntity($iblockId)`; в `getMap()` динамически подмешиваются свойства ИБ (`getMetadata()`, кэш `/dc/iblock/meta`, TTL 86400).

Доступ к свойствам в сущностях идёт по шаблону `PROPERTY_<CODE>_VALUE` (например, [`BxProfile.php`](../../../local/modules/dc/lib/BitrixModel/BxProfile.php:141) → `PROPERTY_BIRTHDAY_VALUE`).

---

## IB_PROFILE (Анкета клиента)

* **ID инфоблока:** `10` — константа `IB_PROFILE`, [`init.php`](../../../bitrix/php_interface/init.php:103)
* **Тип инфоблока:** `clients` (формируется в ссылках админки: `iblock_element_edit.php?IBLOCK_ID=...&type=clients`, [`order_list.php`](../../../local/modules/dc/admin/order_list.php:349))
* **Назначение:** персональные данные заёмщика — ФИО, паспорт, контакты, занятость, доход, семейное положение, супруг(а), контактные лица.
* **Расположение / ключевые классы:**
  * константа — `bitrix/php_interface/init.php`;
  * ORM-обёртка — [`ProfileElementTable.php`](../../../local/modules/dc/lib/Iblock/ProfileElementTable.php:9) (`static protected $_iblockId = IB_PROFILE`);
  * доменная сущность — [`BxProfile.php`](../../../local/modules/dc/lib/BitrixModel/BxProfile.php:24) (`getIBockId()` → `IB_PROFILE`);
  * менеджер чтения — [`BxProfileManager.php`](../../../local/modules/dc/lib/BitrixModel/BxProfileManager.php:32) (фильтр `IBLOCK_ID = IB_PROFILE`).
* **Служебные поля элемента:**
  * `NAME` — заполняется телефоном клиента ([`user.php`](../../../local/modules/dc/lib/user.php:1179));
  * `CODE`, `ACTIVE`, `MODIFIED_BY` — стандартные;
  * один профиль на пользователя: связь через свойство `USER`, поиск `PROPERTY_USER_VALUE`.
* **Особые правила:** редактирование профиля из админки ограничено проверкой прав (`ElementTable::onBeforeUpdate`, [`ElementTable.php`](../../../local/modules/dc/lib/Iblock/ElementTable.php:536) → `isCurrentUserAdmin()`).

### Свойства (по группам)

Значения в коде получены из методов `BxProfile` (`getBirthday()` → `PROPERTY_BIRTHDAY_VALUE`). Если в скобках указан справочник — хранится `UF_XML_ID` соответствующего HL-блока.

**Идентификация**

| Код свойства | Описание | Пример значения |
|---|---|---|
| `LAST_NAME` | Фамилия | `Иванов` |
| `FIRST_NAME` | Имя | `Иван` |
| `SECOND_NAME` | Отчество | `Иванович` |
| `BIRTHDAY` | Дата рождения | `15.04.1990` |
| `SEX` | Пол (HL `HB_SEX`) | `1IG8Y7SX` (муж.) / `YRFMIQES` (жен.) |
| `CITIZENSHIP` | Гражданство (справочник `hb_citizenship`) | `RUS` |
| `BIRTH_PLACE` | Место рождения (используется в актуальных методах) | `г. Москва` |
| `BIRTHPLACE` | Место рождения (устаревший дубль, читается отдельным методом) | `г. Москва` |
| `USER` | ID пользователя Битрикса, владельца профиля | `10245` |
| `HASH` | Хэш для идентификации/поиска профиля | `a3f5c9…` |

**Контакты**

| Код свойства | Описание | Пример значения |
|---|---|---|
| `PHONE_MOBILE` | Мобильный телефон (основной логин) | `79001234567` |
| `PHONE_HOME` | Домашний телефон | `74951234567` |
| `PHONE_WORK` | Рабочий телефон | `74957654321` |
| `PHONE_CONTACT` | Телефон контактного лица | `79009876543` |
| `PHONE_CONTACT_ADD` | Доп. телефон контактного лица | `79009876544` |
| `ADD_PHONE_1` | Дополнительный телефон №1 | `79001112233` |
| `ADD_PHONE_2` | Дополнительный телефон №2 | `79001112244` |
| `EMAIL` | Электронная почта | `ivanov@mail.ru` |

**Паспорт и документы**

| Код свойства | Описание | Пример значения |
|---|---|---|
| `SERIAL_PASSPORT` | Серия паспорта | `1111` |
| `NUMBER_PASSPORT` | Номер паспорта | `111111` |
| `DATE_ISSUE` | Дата выдачи | `21.06.2010` |
| `ISSUED_PASSPORT` | Кем выдан | `ОВД района Тверской` |
| `SUBDIVISION_CODE` | Код подразделения | `770-002` |
| `PREVIOUS_SERIAL_PASSPORT` | Серия предыдущего паспорта | `2222` |
| `PREVIOUS_NUMBER_PASSPORT` | Номер предыдущего паспорта | `222222` |
| `PREVIOUS_LAST_NAME` | Предыдущая фамилия | `Петров` |
| `CHANGE_LAST_NAME` | Флаг смены фамилии | `Y` |
| `INN_NUMBER` | ИНН | `770123456789` |
| `SNILS_NUMBER` | СНИЛС | `123-456-789 00` |

**Занятость и доходы**

| Код свойства | Описание | Пример значения |
|---|---|---|
| `MONTHLY_INCOME` | Ежемесячный доход | `50000` |
| `UTILITIES_MONTHLY_PAYMENTS` | Платежи по ЖКХ в месяц | `6000` |
| `FAMILY_MONTHLY_LOANS_WITHOUT_CLIENTS_PAYMENTS` | Кредитная нагрузка семьи без учёта текущего займа | `12000` |
| `SOCIAL_STATUS` | Социальный статус (HL `HB_SOCIAL_STATUS`) | `0vxo1sNN` (пенсионер) |
| `EDUCATION` | Образование (HL `HB_EDUCATION`) | `HIGH` |
| `BUSINESS_TYPE` | Тип бизнеса (HL `HB_BUSINESS_TYPE`) | `TRADE` |
| `WORK_IN_LAST_PLACE` | Стаж на последнем месте, мес. (прибавляется в `setField()`) | `36` |
| `REAL_ESTATE` | Наличие недвижимости (HL `HB_REALESTATE`) | `FLAT` |
| `AUTO_BRAND` | Марка автомобиля (HL `HB_AUTOBRAND`) | `LADA` |
| `CUSTOMER_ESTIMATE` | Оценка клиента (HL `HB_CUSTOMER_ESTIMATE`) | `GOOD` |
| `property_type_home` | Тип жилья — **код в нижнем регистре**, требует проверки актуальности | `own` |

**Семья и контактные лица**

| Код свойства | Описание | Пример значения |
|---|---|---|
| `MARITAL_STATUS` | Семейное положение (HL `HB_MARITAL_STATUS`) | `MARRIED` |
| `SPOUSE_LAST_NAME` | Фамилия супруга(и) | `Иванова` |
| `SPOUSE_FIRST_NAME` | Имя супруга(и) | `Мария` |
| `SPOUSE_SECOND_NAME` | Отчество супруга(и) | `Сергеевна` |
| `SPOUSE_BIRTHDAY` | Дата рождения супруга(и) | `20.08.1992` |
| `SPOUSE_STARTDATE` | Дата начала совместного проживания | `01.02.2015` |
| `MOTHER_LAST_NAME` | Девичья фамилия матери | `Сидорова` |
| `LAST_NAME_CONTACT` | Фамилия контактного лица | `Петров` |
| `NAME_CONTACT` | Имя контактного лица | `Пётр` |
| `SECOND_NAME_CONTACT` | Отчество контактного лица | `Петрович` |
| `LAST_NAME_CONTACT_ADD` | Фамилия доп. контактного лица | `Сидоров` |
| `NAME_CONTACT_ADD` | Имя доп. контактного лица | `Сидор` |
| `SECOND_NAME_CONTACT_ADD` | Отчество доп. контактного лица | `Сидорович` |

> В `BxProfile` также объявлены константы `PROPERTY_LAST_NAME` и `PROPERTY_PHONE_MOBILE` — это строки `PROPERTY_*` для select-запросов, а не отдельные свойства ИБ.
> Полный перечень кодов, извлечённый из `BxProfile.php` + `BaseProfile.php`, — 57 значений; часть из них (`PROPERTY_LAST_NAME`, `PROPERTY_PHONE_MOBILE`) — технические.

### Связи профиля с другими таблицами

| Связь | Реализация |
|---|---|
| Профиль ↔ адреса | `dc_rel_address_profile` ([`RelAddressProfileTable`](../../../local/modules/dc/lib/Table/Address/RelAddressProfileTable.php:15)) |
| Заказ → профиль | `dc_order.customer_profile_id`, join с `b_iblock_element` по `IBLOCK_ID = IB_PROFILE` ([`OrderTable.php`](../../../local/modules/dc/lib/Table/OrderTable.php:155)) |
| Заявка → профиль | join `ref.IBLOCK_ID = IB_PROFILE` ([`ApplicationTable.php`](../../../local/modules/dc/lib/Table/ApplicationTable.php:346)) |
| JSON-снимок анкеты | `dc_json_profile` ([`JsonProfileTable.php`](../../../local/modules/dc/lib/Iblock/JsonProfileTable.php:14)) |
| Плоский срез «короткого» профиля | `dc_short_profile` ([`ShortProfileTable`](../../../local/modules/dc/lib/Table/Settings/ShortProfileTable.php:24)) |

---

## IB_ANKET (Анкета юридического лица)

* **ID инфоблока:** `9` — константа `IB_ANKET`, [`init.php`](../../../bitrix/php_interface/init.php:102)
* **Назначение:** реквизиты юрлица/ИП — магазина-партнёра: наименование, ИНН/КПП/ОГРН, юридический и фактический адреса, банковские реквизиты, подписант.
* **Расположение / классы:** [`LegalElementTable.php`](../../../local/modules/dc/lib/Iblock/LegalElementTable.php:10) (`$_iblockId = IB_ANKET`), сущность — [`BxLegalEntity.php`](../../../local/modules/dc/lib/BitrixModel/BxLegalEntity.php:24) (свойства задаются через `setField('<code>')`).

### Свойства

| Код свойства | Описание | Пример значения |
|---|---|---|
| `type_sales` | Тип продаж | `offline` |
| `shop` | Привязка к магазину (ID элемента `IB_SHOP`) | `123` |
| `legal_form` | Организационно-правовая форма | `ООО` |
| `full_name` | Полное наименование | `ООО «МВМ»` |
| `url` | Сайт | `https://example.ru` |
| `email` | E-mail | `info@example.ru` |
| `inn` | ИНН | `7701234567` |
| `kpp` | КПП | `770101001` |
| `ogrn` | ОГРН | `1157746000000` |
| `org_code` | Код организации | `1234` |
| `okato` | ОКАТО | `45286565000` |
| `leg_kladr`, `leg_zip_code`, `leg_region`, `leg_district`, `leg_city`, `leg_locality`, `leg_street`, `leg_building`, `leg_office` | Юридический адрес (КЛАДР + дом/офис) | `г. Москва, ул. Тверская, д. 1` |
| `ph_kladr`, `ph_zip_code`, `ph_region`, `ph_district`, `ph_city`, `ph_locality`, `ph_street`, `ph_building`, `ph_office` | Фактический адрес | `125009` |
| `bank_name` | Наименование банка | `ПАО Сбербанк` |
| `bik` | БИК | `044525225` |
| `rs` | Расчётный счёт | `40702810000000000001` |
| `ks` | Корр. счёт | `30101810400000000225` |
| `bank_okpo` | ОКПО банка | `57972160` |
| `signer_contract` | Подписант договора | `Иванов И.И.` |
| `basis` | Основание полномочий | `Устав` |
| `fio_gen_dir` | ФИО генерального директора | `Иванов Иван Иванович` |
| `banks_on` | Подключённые банки | `1,14` |
| `banks_off` | Отключённые банки | `18` |
| `date_fill_anketa` | Дата заполнения анкеты | `28.10.2026` |
| `code_reward_ren` | Код вознаграждения (РЕН) | `999` |
| `online_tin_partner_id` | ID партнёра TIN (онлайн) | `1042` |
| `tin_partner_id` | ID партнёра TIN | `1042` |

---

## Полный реестр инфоблоков

Определение: [`bitrix/php_interface/init.php`](../../../bitrix/php_interface/init.php:96). Назначение указано по названию константы и способу использования в коде; точные названия ИБ в админке требуют проверки.

| Константа | ID | Назначение (по коду) |
|---|---|---|
| `IB_BANK` | 1 | Банки-партнёры (реквизиты, БИК), класс [`BankTable`](../../../local/modules/dc/lib/Iblock/BankTable.php:20) |
| `IB_TM` | 2 | Территориальные менеджеры, класс [`TmElementTable`](../../../local/modules/dc/lib/Iblock/TmElementTable.php) |
| `IB_PROGRAM_OFFLINE` | 4 | Кредитные программы (офлайн), класс [`ProgramElementTable`](../../../local/modules/dc/lib/Iblock/ProgramElementTable.php) |
| `IB_SHOP` | 6 | Магазины, классы [`ShopTable`](../../../local/modules/dc/lib/Iblock/ShopTable.php:20), [`ShopElementTable`](../../../local/modules/dc/lib/Iblock/ShopElementTable.php) |
| `IB_SHOP_AGENT` | 7 | Агенты магазинов/сотрудники, класс [`AgentElementTable`](../../../local/modules/dc/lib/Iblock/AgentElementTable.php) |
| `IB_TT` | 8 | Торговые точки (ТТ), класс [`OutletElementTable`](../../../local/modules/dc/lib/Iblock/OutletElementTable.php) |
| `IB_ANKET` | 9 | Анкеты юрлиц — см. выше |
| `IB_PROFILE` | 10 | Анкеты клиентов — см. выше |
| `IB_SPECIAL_ONLINE` | 12 | Специальные условия/акции (онлайн) |
| `IB_SPECIAL_OFFLINE` | 13 | Специальные условия/акции (офлайн) |
| `IB_REGIONS` | 15 | Регионы |
| `IB_GOODS` | 16 | Товары, класс [`TypeOfGoodsElementTable`](../../../local/modules/dc/lib/Iblock/TypeOfGoodsElementTable.php) |
| `IB_PLACE_TYPE` | 19 | Типы населённых пунктов (КЛАДР) |
| `IB_CITY_TYPE` | 20 | Типы городов (КЛАДР) |
| `IB_STREET_TYPE` | 22 | Типы улиц (КЛАДР) |
| `IB_ORDER_OFFLINE` | 23 | Заказы офлайн |
| `IB_CONTACT` | 25 | Контактные лица |
| `IB_LOGISTICS` | 27 | Логистика |
| `IB_STORES_DOCUMENTS` | 29 | Документы магазинов |
| `IB_DOCUMENTS` | 30 | Документы |
| `IB_OFFERS_OFFLINE` | 31 | Предложения банков (офлайн) |
| `IB_OFFERS_ONLINE` | 32 | Предложения банков (онлайн) |
| `IB_CALC_OFFLINE` | 33 | Расчёты/калькуляции офлайн |
| `IB_DM` | 36 | Дилерские менеджеры (ДМ), класс [`DmElementTable`](../../../local/modules/dc/lib/Iblock/DmElementTable.php) |
| `IB_RULES` | 39 | Правила |
| `IB_BANK_REQUEST` | 40 | Заявки на подключение банков |
| `IB_SERVICE_NOT_BANK` | 42 | Услуги, не связанные с банком |
| `IB_APP_DOSSIER` | 47 | Досье проверки документов |
| `IB_PROMO` | 49 | Подборки промо для онлайн-магазинов |
| `IB_DOC_TYPES` | 53 | Типы документов |
| `IB_LOGISTICS_INVOICE` | 61 | Накладные логистики |
| `IB_LOGISTICS_CITIES` | 65 | Города логистики |
| `IB_CP_POLITIC` | 69 | Политика обработки ПДн (согласия) |
| `IB_ENTRY_POINT` | 73 | Точки входа (виджеты/партнёры) |
| `IB_NEWS` | 81 | Новости |

## ORM-обёртки над инфоблоками

| Класс | Инфоблок | Файл |
|---|---|---|
| `ElementTable` (базовый) | любой (`createEntity($id)`) | [`ElementTable.php`](../../../local/modules/dc/lib/Iblock/ElementTable.php:33) |
| `ElementSPropTable` | свойства S-типа | [`ElementSPropTable.php`](../../../local/modules/dc/lib/Iblock/ElementSPropTable.php:19) |
| `ElementMPropTable` | свойства M-типа | [`ElementMPropTable.php`](../../../local/modules/dc/lib/Iblock/ElementMPropTable.php:19) |
| `ProfileElementTable` | `IB_PROFILE` | [`ProfileElementTable.php`](../../../local/modules/dc/lib/Iblock/ProfileElementTable.php:9) |
| `LegalElementTable` | `IB_ANKET` | [`LegalElementTable.php`](../../../local/modules/dc/lib/Iblock/LegalElementTable.php:10) |
| `BankTable`, `BankElementTable` | `IB_BANK` | [`BankTable.php`](../../../local/modules/dc/lib/Iblock/BankTable.php:20) |
| `ShopTable`, `ShopElementTable` | `IB_SHOP` | [`ShopTable.php`](../../../local/modules/dc/lib/Iblock/ShopTable.php:20) |
| `OutletElementTable` | `IB_TT` | [`OutletElementTable.php`](../../../local/modules/dc/lib/Iblock/OutletElementTable.php) |
| `AgentElementTable` | `IB_SHOP_AGENT` | [`AgentElementTable.php`](../../../local/modules/dc/lib/Iblock/AgentElementTable.php) |
| `TmElementTable` | `IB_TM` | [`TmElementTable.php`](../../../local/modules/dc/lib/Iblock/TmElementTable.php) |
| `DmElementTable` | `IB_DM` | [`DmElementTable.php`](../../../local/modules/dc/lib/Iblock/DmElementTable.php) |
| `ProgramElementTable` | `IB_PROGRAM_OFFLINE` | [`ProgramElementTable.php`](../../../local/modules/dc/lib/Iblock/ProgramElementTable.php) |
| `TypeOfGoodsElementTable` | `IB_GOODS` | [`TypeOfGoodsElementTable.php`](../../../local/modules/dc/lib/Iblock/TypeOfGoodsElementTable.php) |

## Числовые ID свойств, найденные в коде

Часть кода работает со свойствами по числовому ID (а не по коду):

| Константа | ID свойства | Где |
|---|---|---|
| `TmElementTable::PROPERTY_USER_ID` | 19 | [`TmElementTable.php`](../../../local/modules/dc/lib/Iblock/TmElementTable.php:13) |
| `DmElementTable::PROPERTY_USER_ID` | 712 | [`DmElementTable.php`](../../../local/modules/dc/lib/Iblock/DmElementTable.php:13) |
| `AgentElementTable::PROPERTY_USER_ID` | 214 | [`AgentElementTable.php`](../../../local/modules/dc/lib/Iblock/AgentElementTable.php:13) |
| `RuleTable::PROPERTY_ID_SHOP_INSURANCE` | 689 | [`RuleTable.php`](../../../local/modules/dc/lib/Table/InsuranceReplace/RuleTable.php:11) |
| `RuleTable::PROPERTY_ID_OUTLET_INSURANCE` | 690 | то же |
| `RuleTable::PROPERTY_ID_SHOP_ADDITIONALSERVICE` | 692 | то же |
| `RuleTable::PROPERTY_ID_OUTLET_ADDITIONALSERVICE` | 691 | то же |
| `PartnerNotifier\EventHandler::PROPERTY_CODE_SHOP` | 158 | [`EventHandler.php`](../../../local/modules/dc/lib/BitrixModel/PartnerNotifier/EventHandler.php:51) |
| `PartnerNotifier\EventHandler::PROPERTY_CODE_ORDER_NUMBER` | 159 | то же |
| `PartnerNotifier\EventHandler::PROPERTY_CODE_STATUS` | 165 | то же |

## Открытые вопросы (требуют проверки в админке/БД)

1. Тип инфоблока и символьные коды для всех ИБ, кроме `IB_PROFILE` (`type=clients`).
2. Полный список свойств `IB_PROFILE` в БД — код содержит 57 используемых кодов, но в инфоблоке могут быть неиспользуемые свойства и наоборот (код читает свойства, отсутствующие в ИБ, — например `property_type_home`).
3. `BIRTH_PLACE` vs `BIRTHPLACE` — какое из свойств является мастер-источником.
4. Числовые ID свойств (19/214/712/689–692/158–165) привязаны к конкретным ИБ и могут отличаться между средами (dev/test/prod) — константы в коде это не учитывают.

---
type: db-schema
title: "Кастомные таблицы D7 ORM"
tags: [db, bitrix, orm, schema]
owner: "Команда ЛК"
updated: "2026-10-28"
---

# Кастомные таблицы D7 ORM

## История изменений
| Дата | Задача | Автор | Изменение |
|---|---|---|---|
| 28.10.2026 | [CASH-4817a](https://jira.mvideo.ru/browse/CASH-4817a) | [Еременко Светлана Александровна(ДК)](https://wiki.mvideo.ru/display/~SEremenko)| Создание документации (через ИИ) |

## Как устроен слой ORM

Прикладные данные модуля `dc` хранятся в собственных таблицах, описанных через `\Bitrix\Main\Entity\DataManager` (D7 ORM):

* имя таблицы — `getTableName()`, описание полей — `getMap()`;
* базовые классы: [`ExtendedDataManager`](../../local/modules/bitrix/main/lib/entity/ExtendedDataManager.php:13), `DictionaryTableLight` (справочники), `AbstractSecurityLogTable` (логи безопасности), `SystemOptionsTable` (key–value);
* соглашения по префиксам описаны в [`_index.md`](_index.md).

Итого в коде объявлено **285 классов-наследников `DataManager`** и **288 уникальных имён таблиц** (`dc_*` — 282, `hb_*` — 6). Часть классов — абстрактные базовые (`ElementSPropTable`, `ElementMPropTable`, `BaseProgramPromoTable`, `BaseProgramGoodsTable`, `DictionaryTableLight`, `AbstractSecurityLogTable`), часть — обёртки над системными таблицами (`b_iblock_element`).

> Полный перечень ниже — «каркас»: имя таблицы + домен. Детальные колонки каждой таблицы восстанавливаются из `getMap()` соответствующего класса.

## Сводка по доменам

| Домен | Кол-во таблиц | Префиксы |
|---|---|---|
| Заказы, заявки, сканы | 58 | `dc_order*`, `dc_application*`, `dc_app*`, `dc_rel_order_*`, `dc_scan*` |
| Клиенты, профили, адреса, идентификация | 20 | `dc_address`, `dc_rel_address_*`, `dc_*_profile`, `dc_*esia*` |
| Банки, магазины, ТТ, оргструктура | 33 | `dc_shop_*`, `dc_bank_*`, `dc_outlet_*`, `dc_rel_*_position` |
| Настройки виджетов, UI и бизнес-правил | 83 | `dc_*_settings`, `dc_services_*`, `dc_step_text*` |
| Страхование, доп. продукты, услуги, документы | 45 | `dc_insurance_*`, `dc_additional_product_*`, `dc_agreement_*` |
| Справочники (ENUM) | 28 | `dc_dict_*`, `dc_*_type` |
| Инфраструктура и служебные | 15 | `dc_system_options`, `dc_entity_*`, `dc_event_*` |
| Справочные таблицы с префиксом `hb_` | 6 | `hb_citizenship`, `hb_vsk_shops*` |
| **Итого** | **288** | |

---

## 1. Заказы, заявки, сканы (58)

Ключевые:

| Таблица | Класс | Назначение |
|---|---|---|
| `dc_order` | [`LightOrderTable`](../../local/modules/dc/lib/Table/LightOrderTable.php:66) | Основная таблица заказов |
| `dc_application` | [`ApplicationTable`](../../local/modules/dc/lib/Table/ApplicationTable.php:90) | Заявки на кредит, привязка к заказу и профилю |
| `dc_order_history` | [`OrderHistoryTable`](../../local/modules/dc/lib/Table/OrderHistoryTable.php:21) | История изменений заказа |
| `dc_order_status_history` | [`OrderStatusHistoryTable`](../../local/modules/dc/lib/Table/OrderStatusHistoryTable.php:22) | История статусов заказа |
| `dc_status_order` / `dc_status_app` | [`StatusOrderTable`](../../local/modules/dc/lib/Table/StatusOrderTable.php:107), [`StatusAppTable`](../../local/modules/dc/lib/Table/StatusAppTable.php:140) | Статусы с захардкоженными ID (`ID_SAVED_ONLINE = 59`) |
| `dc_scan` | [`ScanTable`](../../local/modules/dc/lib/Table/ScanTable.php:38) | Сканы документов по заказу |
| `dc_order_security` | [`OrderSecurityTable`](../../local/modules/dc/lib/Table/OrderSecurityTable.php:20) | Антифрод-проверки заказа |
| `dc_order_split_*` | `OrderSplitRelationsTable`, `OrderSplitPropertiesTable`, `OrderSplitMethodsTable` | Разделение заказа (split checkout) |

Полный перечень:

```text
dc_order                            dc_application
dc_order_bad_passport_log           dc_application_client_value
dc_order_cash_attribute             dc_application_disable_cancellations
dc_order_fast_scoring_attribute     dc_application_insurance_price
dc_order_history                    dc_application_nominal_account
dc_order_params                     dc_application_status_history
dc_order_return_link                dc_app
dc_order_security                   dc_app_return
dc_order_security_customer_type     dc_app_return_app
dc_order_security_decision          dc_app_return_goods
dc_order_security_log               dc_app_sama
dc_order_skip_sama                  dc_app_scheduler
dc_order_special                    dc_app_score_error
dc_order_split_methods              dc_status_app
dc_order_split_properties           dc_status_order
dc_order_split_relations            dc_wait_status_request
dc_order_status_history             dc_checkout
dc_order_status_mapping             dc_scan
dc_order_status_mapping_lk          dc_scans_upload_qr_code
dc_order_status_mapping_lk_outlet   dc_scans_upload_qr_code_outlet
dc_order_tmp                        DC_SCANS_CANCEL
dc_rel_order_additional             DC_SCANS_RETURN
dc_rel_order_allowed_program        dc_espp_passport_check_order
dc_rel_order_espi_service           dc_esrp_passport_check_order
dc_rel_order_fake                   dc_esrp_passport_check_order_error
dc_rel_order_good
dc_rel_order_insurance
dc_rel_order_notbank
dc_rel_order_program
dc_rel_order_sign_type_allowed
dc_rel_application_notbank
dc_rel_application_replaces_insurance
dc_rel_application_replaces_service
```

> `DC_SCANS_CANCEL` / `DC_SCANS_RETURN` — имена в верхнем регистре (вероятно, константы типов, а не таблицы): требует проверки.

## 2. Клиенты, профили, адреса, идентификация (20)

| Таблица | Класс | Назначение |
|---|---|---|
| `dc_customer_profile` | [`CustomProfileTable`](../../local/modules/dc/custom_profile.php:10) | Дополнительные данные профиля клиента |
| `dc_short_profile` | [`ShortProfileTable`](../../local/modules/dc/lib/Table/Settings/ShortProfileTable.php:24) | Плоский срез анкеты для виджетов/списков |
| `dc_json_profile` | [`JsonProfileTable`](../../local/modules/dc/lib/Iblock/JsonProfileTable.php:14) | JSON-снимок анкеты |
| `dc_address` | [`AddressTable`](../../local/modules/dc/lib/Table/Address/AddressTable.php:20) | Адреса (единая таблица) |
| `dc_rel_address_profile` | [`RelAddressProfileTable`](../../local/modules/dc/lib/Table/Address/RelAddressProfileTable.php:15) | Профиль ↔ адрес |
| `dc_user_esia` | [`UserEsiaTable`](../../local/modules/dc/lib/Table/UserEsiaTable.php:37) | Идентификация через ЕСИА |
| `dc_verification` | [`VerificationTable`](../../local/modules/dc/lib/BitrixModel/Verification/VerificationTable.php:8) | Коды подтверждения (SMS/код-слово) |
| `dc_passport_issuer` | [`PassportIssuerTable`](../../local/modules/dc/lib/passport_issuer.php:20) | Справочник органов выдачи паспорта |

Полный перечень:

```text
dc_customer_profile         dc_rel_address_order
dc_short_profile            dc_rel_address_profile
dc_json_profile             dc_rel_address_service
dc_address                  dc_photo_stage
dc_user_esia                dc_photo_stage_mass_update
dc_verification             dc_passport_issuer
dc_sms_storage              dc_customer_suspicious
dc_subscribe                dc_customer_suspicious_type
dc_unsubscribed_mail        dc_special_promo_customer_choice
dc_my_finances_customer_choice
dc_dev_phones
```

## 3. Банки, магазины, ТТ, оргструктура (33)

| Таблица | Класс | Назначение |
|---|---|---|
| `dc_shop_settings` | [`ShopSettingsTable`](../../local/modules/dc/lib/Iblock/ShopSettingsTable.php:18) | Настройки витрины магазина |
| `dc_shop_contract_settings` | [`ShopContractSettingsTable`](../../local/modules/dc/lib/Iblock/ShopContractSettingsTable.php:33) | Настройки договоров магазина |
| `dc_bank_integration_settings` | [`BankIntegrationSettingsTable`](../../local/modules/dc/lib/Table/Settings/BankIntegrationSettingsTable.php:53) | Параметры интеграции с банком |
| `dc_bank_bik_account` | [`BankBikTable`](../../local/modules/dc/lib/bankbiktable.php:7) | БИК и корр. счета банков |
| `dc_outlet_group` | [`OutletGroupTable`](../../local/modules/dc/lib/Table/Settings/OutletGroupTable.php:83) | Группы торговых точек |
| `dc_rel_shop_user_position` / `dc_rel_outlet_user_position` / `dc_rel_legal_entity_user_position` | `ShopUserPositionTable`, `OutletUserPositionTable`, `LegalEntityUserPositionTable` | Должности сотрудников по уровням |
| `dc_responsible_subordinates` | [`ResponsibleSubordinatesTable`](../../local/modules/dc/lib/Table/Responsible/ResponsibleSubordinatesTable.php:24) | Подчинённость ответственных |

Полный перечень:

```text
dc_shop_settings                        dc_bank_bik_account
dc_shop_contract_settings               dc_bank_integration_settings
dc_shop_bank_priority                   dc_bank_integration_settings_outlets
dc_shop_insurance_bank                  dc_bank_protocol_settings
dc_shop_insurance_bank_outlets          dc_bank_steps
dc_shop_insurance_bank_payment_methods  dc_position
dc_shop_service_settings                dc_rel_shop_user_position
dc_shop_monitor                         dc_rel_outlet_user_position
dc_shop_monitor_app_alerts              dc_rel_legal_entity_user_position
dc_shop_monitor_rel_email               dc_rel_user_position
dc_shop_monitor_rel_entity_id           dc_rel_agent_agent
dc_shop_monitor_rel_sale_channel        dc_rel_agent_selected_outlet
dc_outlet_group                         dc_responsible_subordinates
dc_outlet_group_outlets                 dc_legal_face_type
dc_outlets_for_sopd_setting             dc_type_loan_program
dc_sopd_setting                         dc_credit_product_tag
                                        dc_credit_product_tag_rel
```

## 4. Настройки виджетов, UI и бизнес-правил (83)

| Таблица | Класс | Назначение |
|---|---|---|
| `dc_common_settings` | [`CommonSettingsTable`](../../local/modules/dc/lib/Table/Settings/CommonSettingsTable.php:75) | Глобальные переключатели (типы: `SETTINGS_TYPE_CONFIRM_TIME = 5`, `SETTINGS_TYPE_BLOCK_ONLINE_SMS = 6`, `SETTINGS_TYPE_POS_CARD_DISABLED = 235`) |
| `dc_settings_score_block` | [`ScoreBlockTable`](../../local/modules/dc/lib/Table/Settings/ScoreBlockTable.php:38) | Блокировки скоринга по банкам |
| `dc_score_error*` | `ScoreErrorTable`, `ScoreErrorShopTable`, `ScoreErrorOutletTable`, `ScoreErrorUGTable` | Причины ошибок скоринга (общие/магазин/ТТ/группа) |
| `dc_step_text` | [`StepTextTable`](../../local/modules/dc/lib/Iblock/StepTextTable.php:35) | Тексты шагов виджета |
| `dc_color_scheme_settings` | [`ColorSchemeSettingsTable`](../../local/modules/dc/lib/Table/Settings/ColorSchemeSettingsTable.php:30) | Цветовые схемы виджета |
| `dc_services_text` | [`ServicesTextTable`](../../local/modules/dc/lib/Table/Settings/ServicesTextTable.php:43) | Тексты услуг |
| `dc_print_template` | [`PrintTemplateTable`](../../local/modules/dc/lib/Table/PrintTemplate/PrintTemplateTable.php:26) | Шаблоны печати |
| `dc_notify_settings` | [`NotifySettingsTable`](../../local/modules/dc/lib/Iblock/NotifySettingsTable.php:16) | Настройки уведомлений |

Полный перечень:

```text
dc_alternative_select_settings                  dc_passport_step
dc_badges_calculator_step_settings              dc_passport_step_outlet
dc_badges_calculator_step_settings_objects      dc_payment_method_settings
dc_call_center_settings_rel_outlet              dc_payment_method_settings_descriptions
dc_color_scheme_settings                        dc_photo_upload_settings
dc_color_scheme_logo                            dc_print_template
dc_color_scheme_logo_outlets                    dc_print_template_additional_product
dc_common_button_settings                       dc_proxy_settings
dc_common_button_settings_outlet                dc_pushsms_token_settings
dc_common_settings                              dc_pushsms_token_settings_legal
dc_display_options_settings                     dc_pushsms_token_settings_shop
dc_document_type                                dc_redirect_client_type
dc_document_type_dep_additionalservice          dc_redirect_partner_url
dc_document_type_dep_insurance                  dc_rel_doctype_logistics
dc_document_type_dep_servicenotbank             dc_sama_score_settings
dc_document_type_servicenotbank                 dc_sama_shop_settings
dc_electronic_specification_settings            dc_score_error
dc_electronic_specification_settings_rel_outlets dc_score_error_outlet
dc_exclusion_service_from_specification_settings dc_score_error_shop
dc_exclusion_service_from_specification_settings_rel_banks dc_score_error_type
dc_exclusion_service_from_specification_settings_rel_outlets dc_score_error_ug
dc_exclusion_service_from_specification_settings_rel_services dc_service_api_settings
dc_get_payment_settings                         dc_service_api_settings_legal_person
dc_ignore_additional_product_settings_rel_additional_product dc_service_api_settings_outlet
dc_ignore_additional_product_settings_rel_outlet dc_service_api_settings_outlet_type
dc_ignore_additional_product_settings_rel_product_type dc_service_api_settings_scheme
dc_images_for_widget                            dc_service_api_settings_shop
dc_integration_settings_type                    dc_services_barcode_settings
dc_limit_order_goods                            dc_services_text
dc_limits_additional_diff                       dc_settings_auth_cancel_soap
dc_logistic_settings                            dc_settings_auth_cancel_soap_rel_id
dc_logo                                         dc_settings_auth_cancel_soap_rel_mail
dc_logo_type                                    dc_settings_nominal_account
dc_memo                                         dc_settings_nominal_account_bank
dc_memo_banks                                   dc_settings_nominal_account_outlet
dc_memo_outlets                                 dc_settings_redirect_client
dc_memo_payment_methods                         dc_settings_redirect_client_urls
dc_notify_settings                              dc_settings_return
dc_passport_dadata_settings                     dc_settings_score_block
                                                dc_step_text
                                                dc_step_text_outlet
                                                dc_step_text_useful_services
                                                dc_webim_settings
                                                dc_webim_window_type
```

## 5. Страхование, доп. продукты, услуги, документы (45)

| Таблица | Класс | Назначение |
|---|---|---|
| `dc_insurance_replace` | [`ReplaceTable`](../../local/modules/dc/lib/Table/InsuranceReplace/ReplaceTable.php:33) | Правила замены страховок |
| `dc_insurance_replace_rule` | [`RuleTable`](../../local/modules/dc/lib/Table/InsuranceReplace/RuleTable.php:20) | Правила с числовыми ID свойств (689–692) |
| `dc_insurance_history` | [`InsuranceHistoryTable`](../../local/modules/dc/lib/Table/Insurance/InsuranceHistoryTable.php:38) | История страховок по заказу |
| `dc_additional_product_relation_rule` | [`AdditionalProductRelationRuleTable`](../../local/modules/dc/lib/Table/AdditionalProduct/Relation/AdditionalProductRelationRuleTable.php:34) | Правила выдачи доп. продуктов |
| `dc_card_issue` | [`CardIssueTable`](../../local/modules/dc/lib/Table/AdditionalProduct/CardIssueTable.php:21) | Выдача карт |
| `dc_card_transaction` | [`CardTransactionTable`](../../local/modules/dc/lib/Table/CardTransactionTable.php:34) | Транзакции по картам |
| `dc_agreement_document` | [`AgreementDocumentTable`](../../local/modules/dc/lib/Table/AgreementDocument/AgreementDocumentTable.php:25) | Согласия/договоры клиента |
| `dc_service_param` | [`AdditionalServiceParamTable`](../../local/modules/dc/lib/Table/AdditionalService/AdditionalServiceParamTable.php:35) | Параметры доп. услуг |
| `dc_basket_services` | [`BasketServicesTable`](../../local/modules/dc/lib/Table/BasketServicesTable.php:28) | Услуги в корзине |

Полный перечень:

```text
dc_insurance_details                            dc_additional_product_relation_rule
dc_insurance_history                            dc_additional_product_relation_rule_additional_product
dc_insurance_relation_rule                      dc_additional_product_relation_rule_outlet
dc_insurance_relation_rule_relation_insurance   dc_additional_product_status_mapping
dc_insurance_relation_rule_relation_shop        dc_additional_product_status_mapping_outlet
dc_insurance_replace                            dc_agreement_document
dc_insurance_replace_outlets                    dc_agreement_document_sample
dc_insurance_replace_period                     dc_agreement_document_setting
dc_insurance_replace_rule                       dc_agreement_document_setting_type
dc_insurance_replace_rule_insurancenotbank      dc_agreement_document_type
dc_insurance_replace_rule_notbank_to_notbank    dc_basket_services
dc_replace_exclude_settings                     dc_basket_services_checkout
dc_rel_insurancenotbank_children                dc_basket_services_products
dc_rel_offers_insurance                         dc_calculations_additional_products
dc_additional_info_loan_program                 dc_card_issue
dc_additional_info_loan_program_program         dc_card_transaction
dc_additional_product_disable_sale              dc_service_history
dc_additional_product_disable_sale_add_prod     dc_service_param
dc_additional_product_disable_sale_bank         dc_goodsoff
dc_additional_product_display_on_timer          dc_goodson
dc_additional_product_display_on_timer_outlets  dc_program_offline_good
dc_program_offline_promo                        dc_program_online_good
dc_program_online_promo
```

## 6. Справочники / ENUM (28)

Реализованы на D7 (`dc_dict_*`), часть — наследники `DictionaryTableLight` ([`DictionaryTableLight.php`](../../local/modules/dc/lib/Table/Dict/DictionaryTableLight.php:19)).

| Таблица | Класс | Назначение |
|---|---|---|
| `dc_dict_sign_type` | [`DictSignTypeTable`](../../local/modules/dc/lib/Table/Dict/DictSignTypeTable.php:15) | Типы подписания (дублирует HL `HB_SIGN_TYPE`) |
| `dc_dict_insurance_type` | [`InsuranceTypeTable`](../../local/modules/dc/lib/Table/Dict/InsuranceTypeTable.php:35) | Типы страхования (`medical`, `work`) |
| `dc_dict_sale_channel` | [`SaleChannelTable`](../../local/modules/dc/lib/Table/Dict/SaleChannelTable.php:36) | Каналы продаж |
| `dc_dict_entity_type` | [`EntityTypeTable`](../../local/modules/dc/lib/Table/Dict/EntityTypeTable.php:27) | Типы сущностей для логов/снапшотов |

Полный перечень:

```text
dc_dict_additional_product_relation_status  dc_dict_my_finances_cancel_reason_type
dc_dict_address_type                        dc_dict_order_params_type
dc_dict_agent_property                      dc_dict_order_type
dc_dict_agreement_document_statuses         dc_dict_profile
dc_dict_agreement_type                      dc_dict_reason_disconnection
dc_dict_client_value_type                   dc_dict_relation_type
dc_dict_credit_proposal                     dc_dict_sale_channel
dc_dict_decision                            dc_dict_sama_type
dc_dict_direction                           dc_dict_scans_type
dc_dict_document_shared_type                dc_dict_shop_monitor_channels
dc_dict_entity_type                         dc_dict_sign_type
dc_dict_image_type                          dc_dict_special_promo_type
dc_dict_insurance_type                      dc_dict_type_services
dc_dict_widget_steps                        dc_my_finances_cancel_reason
```

## 7. Инфраструктура и служебные (15)

| Таблица | Класс | Назначение |
|---|---|---|
| `dc_system_options` | [`SystemOptionsTable`](../../local/modules/dc/lib/Iblock/SystemOptionsTable.php:77) | key–value опции (`name`, `value`) |
| `dc_entity_snapshot` | [`EntitySnapshotTable`](../../local/modules/dc/lib/Table/EntitySnapshotTable.php:22) | Снимки сущностей до изменения (аудит) |
| `dc_event_scheduler` | [`EventSchedulerTable`](../../local/modules/dc/lib/Table/EventSchedulerTable.php:137) | Планировщик событий/задач |
| `dc_import_task` | [`ImportTaskTable`](../../local/modules/dc/lib/Iblock/ImportTaskTable.php:13) | Задачи импорта |
| `dc_admin_access_rights` | [`AdminAccessRightsTable`](../../local/modules/dc/lib/Table/AdminAccessRightsTable.php:118) | Права доступа к разделам админки |

Полный перечень:

```text
dc_admin_access_rights          dc_monitoring_changed
dc_admin_access_rights_page     dc_monitoring_changed_rel_id
dc_entity_snapshot              dc_pcre_patterns
dc_entity_test                  dc_system_options
dc_entity_test_mass_upload      dc_token_action
dc_event_scheduler              dc_import_task
dc_event_scheduler_tag          dc_log_soap
dc_login_history
```

## 8. Таблицы с префиксом `hb_` (6)

Не являются HL-блоками в терминах API (обычные ORM-таблицы с префиксом `hb_`):

| Таблица | Класс | Назначение |
|---|---|---|
| `hb_citizenship` | [`CitizenshipTable`](../../local/modules/dc/lib/Iblock/CitizenshipTable.php:16) | Справочник гражданств: `UF_NAME`, `UF_XML_ID`, `UF_OKSM`, `UF_ISO`, `UF_PASSPORT_MASK`; константа `RUSSIA = 'RUS'` |
| `hb_signtype_settings` | [`SignTypeSettingsTable`](../../local/modules/dc/lib/Iblock/SignTypeSettingsTable.php:18) | Настройки подписания |
| `hb_scantype_settings` | [`ScanTypeSettingsTable`](../../local/modules/dc/lib/Iblock/ScanTypeSettingsTable.php:34) | Настройки типов сканов |
| `hb_vsk_shops` | [`VskShopTable`](../../local/modules/dc/lib/Table/InsuranceNotBank/VskShopTable.php:14) | Магазины ВСК (страхование not-bank) |
| `hb_vsk_shops_uf_shops` | [`VskShopShopIdTable`](../../local/modules/dc/lib/Table/InsuranceNotBank/VskShopShopIdTable.php:11) | Связь ВСК ↔ магазины |
| `hb_vsk_shops_uf_outlets` | [`VskShopOutletIdTable`](../../local/modules/dc/lib/Table/InsuranceNotBank/VskShopOutletIdTable.php:10) | Связь ВСК ↔ торговые точки |

## Технический долг и открытые вопросы

1. **Дублирование справочников**: `dc_dict_sign_type` ↔ HL `HB_SIGN_TYPE`, `dc_dict_insurance_type` ↔ HL `HB_INSURANCE` — нужно определить мастер-источник.
2. **Захардкоженные ID статусов** в `StatusAppTable` (`ID_REQUEST_ERROR = 853`, `ID_BANK_BLOCKED = 1361`, `ID_SAVED_ONLINE = 59`) — значения могут различаться между средами.
3. **Расхождение «класс ↔ таблица»**: `PaymentMethodTypeTable` → `dc_dict_special_promo_type`, `CustomerSecurityTypeTable` → `dc_order_security_customer_type`, `InsuranceRuleSettingsTable` → `dc_insurance_relation_rule_relation_shop`. Ориентироваться на `getTableName()`, а не на имя класса.
4. **Миграций/DDL в репозитории нет** — структура восстанавливается только по `getMap()`, индексы и внешние ключи неизвестны.
5. **Часть таблиц — legacy** (`dc_goodsoff`, `dc_goodson`, `dc_bank_bik_account`, `dc_custom_profile`-подобные) — объявлены вне каталога `lib/Table`, схема может создаваться вручную.

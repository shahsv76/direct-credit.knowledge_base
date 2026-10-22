---
type: page
title: "Фронтенд: Параметры запуска (URL)"
tags: [frontend, mpay, url]
owner: "Команда ЛК"
updated: "2026-10-21"
---

# Параметры запуска виджета (URL)

Виджет открывается по единому URL. Параметры передаются в query string и используются для инициализации сценария и сохранения контекста партнера.

**Формат URL:**
```text
https://{host}/new/new-online/#/default?order_id={order_id}&order_number={order_number}&color={color}&reorder={reorder}&dc_location={dc_location}&service={service}&tt_kladr={tt_kladr}&isWebView={isWebView}
# Диаграммы RTB

| Схема | Просмотр в репозитории | Интерактивная версия | Исходник |
| --- | --- | --- | --- |
| Компоненты через три месяца | [SVG](component.svg) | [HTML](component.html) | [JSON](component.json) |
| Ставка и расчётное событие | [SVG](sequence.svg) | [HTML](sequence.html) | [JSON](sequence.json) |

HTML нужно скачать и открыть в браузере. Все подписи решения написаны по-русски; фиксированный интерфейс Archify и атрибут `lang` остаются английскими. В sequence путь `/openrtb2/auction` показан без префикса `/dsp/{partner_id}`, а `billing.v1` обозначает topic `rtb.billing.v1`.

Component раскрывает RTB Gateway, Bidding и фоновые зависимости. Campaign и Finance пока находятся в монолите за адаптером; стрелка REST от Bidding выполняется только в фоне. Стрелки запросов подразумевают ответ по тому же соединению. Чтение billing модулем Finance показано отдельно на sequence. Browser/Delivery и Redis опущены, чтобы не смешивать путь ставки с выдачей баннера и прогревом.

Sequence показывает ставку и один последующий расчёт. Bidding использует локальную модель и квоту. Collector подтверждает callback после Kafka ack, Finance подтверждает offset после SQL commit. Между подтверждением Collector и началом работы Finance строгого порядка нет. Ошибки и повторы описаны в [interaction.md](../interaction.md#повторы-и-отказы).

## Воспроизведение

Диаграммы собраны Archify 2.16. Переменная `ARCHIFY_DIR` должна указывать на каталог установленного навыка. Из корня репозитория:

```bash
node "$ARCHIFY_DIR/bin/archify.mjs" validate architecture Task2/diagrams/component.json --quality showcase --json
node "$ARCHIFY_DIR/bin/archify.mjs" deliver architecture Task2/diagrams/component.json Task2/diagrams/component.html --quality showcase --json
node "$ARCHIFY_DIR/bin/archify.mjs" validate sequence Task2/diagrams/sequence.json --quality showcase --json
node "$ARCHIFY_DIR/bin/archify.mjs" deliver sequence Task2/diagrams/sequence.json Task2/diagrams/sequence.html --quality showcase --json
node "$ARCHIFY_DIR/bin/archify.mjs" visual-check Task2/diagrams/component.html --json
node "$ARCHIFY_DIR/bin/archify.mjs" visual-check Task2/diagrams/sequence.html --json
```

SVG получены штатной командой `Export → SVG` из проверенных HTML. После экспорта удалены пробелы в концах строк. SVG не содержат скриптов и подходят для просмотра в Markdown. После изменения JSON нужно пересобрать HTML и заново экспортировать SVG.

## Проверка артефактов

Обе схемы прошли 9/9 проверок showcase, без ошибок и предупреждений. Проверка Chrome подтвердила отсутствие переполнения на 1440×900, 1600×1000, 1920×1080 и 2048×1320. Светлая и тёмная темы просмотрены по снимкам; подписи и стрелки не перекрывают друг друга. Component потребовал одного исправления вертикальных интервалов после визуальной проверки; sequence — ни одного. Это проверка документации, не испытание RTB-сервиса.

| Артефакт | Размер, байт | SHA-256 |
| --- | --- | --- |
| `component.json` | 3405 | `87ec4af4a4d8e2c2e45bf6d8d7ca8850e59e6594a3e96b3c1c848ad3fa344b4b` |
| `component.html` | 709142 | `6fed28293a13381c7b291ef862295b3f27109b3c1b4afdba4dca5ab2f59117e1` |
| `sequence.json` | 2279 | `53eb8dcbcfb03269ac6805b6087275dcb1afb69453cb77f07a4a03ea41abedfc` |
| `sequence.html` | 707859 | `a6863062d5460ff734b9738d4c1118830f4ef13e9236ee827e60feb75dffd29b` |

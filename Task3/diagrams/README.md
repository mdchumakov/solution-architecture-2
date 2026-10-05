# Диаграммы данных

| Схема | Просмотр | Интерактивная версия | Исходник |
| --- | --- | --- | --- |
| Владельцы данных | [SVG](data-ownership.svg) | [HTML](data-ownership.html) | [JSON](data-ownership.json) |
| Репликация PostgreSQL | [SVG](replication.svg) | [HTML](replication.html) | [JSON](replication.json) |

## Владение данными

![Владельцы данных AdScale](data-ownership.svg)

Показаны логические владельцы, а не количество серверов. На трёхмесячном этапе Campaign DB и Finance DB ещё являются изолированными схемами монолита; через год это отдельные базы и ресурсы. Event Collector владеет входными topics Kafka, а Campaign и Finance — своими доменными topics, которые на этой схеме опущены. Finance и Analytics читают события через Kafka, не через таблицы Collector.

RAM и Redis объединены только как модель чтения Bidding. Ставка использует RAM, Redis нужен для фонового прогрева. Источник достоверного баланса — Finance DB.

## Репликация и копии

![Репликация и восстановление PostgreSQL](replication.svg)

Шаблон применяется отдельно к Campaign и Finance. Sync standby подтверждает локальную запись, DR standby в другом регионе получает WAL асинхронно к годовому этапу. Показан допустимый вариант каскадной репликации через standby. После локального failover downstream перенастраивается на действующий источник WAL.

Архив base backup/WAL находится вне отказной области production. Его стрелка к DR означает восстановление/PITR, а не второй канал активной записи. Координатор HA, fencing, read replicas и условия возврата финансовой записи описаны в [scaling.md](../scaling.md) и [reliability.md](../reliability.md).

## Воспроизведение и проверка

HTML скачивается и открывается в браузере без сервера. Подписи написаны по-русски; фиксированный интерфейс Archify и атрибут `lang` остаются английскими. Для пересборки нужен Archify 2.16, `ARCHIFY_DIR` указывает на каталог навыка:

```bash
node "$ARCHIFY_DIR/bin/archify.mjs" validate architecture Task3/diagrams/data-ownership.json --quality showcase --json
node "$ARCHIFY_DIR/bin/archify.mjs" deliver architecture Task3/diagrams/data-ownership.json Task3/diagrams/data-ownership.html --quality showcase --json
node "$ARCHIFY_DIR/bin/archify.mjs" validate architecture Task3/diagrams/replication.json --quality showcase --json
node "$ARCHIFY_DIR/bin/archify.mjs" deliver architecture Task3/diagrams/replication.json Task3/diagrams/replication.html --quality showcase --json
node "$ARCHIFY_DIR/bin/archify.mjs" visual-check Task3/diagrams/data-ownership.html --json
node "$ARCHIFY_DIR/bin/archify.mjs" visual-check Task3/diagrams/replication.html --json
```

SVG экспортированы через штатное меню `Export → SVG`; после экспорта удалены пробелы в концах строк. Они не содержат скриптов. После изменения JSON заново выполняются сборка, проверка и экспорт.

Обе схемы прошли 9/9 проверок showcase без ошибок и предупреждений. Chrome проверил размещение на 1440×900, 1600×1000, 1920×1080 и 2048×1320; светлая и тёмная темы просмотрены по снимкам. Диаграмма владельцев получила одну корректировку вертикальных интервалов после просмотра; схема репликации — ни одной. Проверка схем не подтверждает достижение RPO/RTO инфраструктуры.

| Артефакт | Размер, байт | SHA-256 |
| --- | --- | --- |
| `data-ownership.json` | 2948 | `7a978a6e4d53ee2a34d12813e2e2a644e51ebda44d78fb9bdbe64b5edb7f5eb5` |
| `data-ownership.html` | 708585 | `d0d56201b53e71b93c9c6f52df091631bf431f6ba2c76651ada062968a1d8cf4` |
| `replication.json` | 2121 | `31ca2539c9f1ae16506fb69eb249f027ad765ea08ca3fc262e31638f10673b3e` |
| `replication.html` | 704384 | `ae95f74cb5cddea5178d25d929fc07b766368036e5f5ead1acdfcbbefaef65bb` |

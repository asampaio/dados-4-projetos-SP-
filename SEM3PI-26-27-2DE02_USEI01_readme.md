# USEI01 - CSV Import into JCF Data Structures

## Scope

The planning model imports the official Sprint 1 ESINF datasets (`robots.csv`, `stations.csv` and `orders.csv`) into memory. The importer accepts the documented CSV format: UTF-8, comma separator and a header row. Robot rows must reference a station loaded from `stations.csv`.

`orders.csv` is line-oriented: several rows with the same `order_id` are grouped into one `Order`. The order exposes the total weight, total volume and whether any item needs cooling.

## Data structures

- `LinkedHashMap<String, Robot>` keeps robots indexed by identifier and preserves input order.
- `LinkedHashMap<String, RobotStation>` keeps stations indexed by identifier and preserves input order.
- `LinkedHashMap<String, Order>` groups order lines by identifier and preserves the first-seen order.
- Each order uses an immutable `List<OrderItem>` for its item lines.

No `PriorityQueue` or other heap-based structure is used, in accordance with the Sprint 1 restriction. Queue ordering belongs to USEI03, not to this import operation.

## Error handling

A malformed row is skipped and produces an `ImportError` containing the file, one-based line number, field and reason. Processing continues with the following rows. Duplicate robot and station identifiers, as well as robot references to unknown stations, are reported as errors; repeated order identifiers are expected when they represent additional item lines.

Missing files or invalid headers are fatal input errors and are reported by `IOException` because the dataset cannot be interpreted safely.

## API

`CsvDataImporter#importDirectory(Path)` reads the three required files and returns an `ImportResult` containing the imported maps and recoverable errors.

The tests cover clean data, repeated order lines, duplicate identifiers, malformed numeric values and continuation after invalid rows.
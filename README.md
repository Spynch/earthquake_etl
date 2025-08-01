# Earthquake ETL

Репозиторий демонстрирует работу ETL-пайплайна по загрузке и агрегированию данных о землетрясениях. Проект построен на базе Apache Airflow и использует DuckDB, Postgres и Minio.

## Структура проекта

- `dags/` – DAG-файлы Airflow.
  - `raw_from_api_to_s3.py` – загрузка данных из публичного API USGS и сохранение их в Minio в формате Parquet.
  - `raw_from_s3_to_pg.py` – перенос файлов из Minio в PostgreSQL в слой ODS.
  - `fct_avg_day_earthquake.py` – вычисление средней магнитуды землетрясений за день и запись результата в слой DM.
  - `fct_count_day_earthquake.py` – расчёт количества землетрясений за день и запись результата в слой DM.
- `docker-compose.yaml` – конфигурация окружения для локального запуска: Airflow с CeleryExecutor, PostgreSQL, Minio, Metabase и другие сервисы.
- `metabase/` – Dockerfile для Metabase с драйвером DuckDB для подключения к хранилищу.
- `requirements.txt` – список Python-зависимостей проекта.

## Используемые технологии

- **Apache Airflow** – оркестрация ETL-процессов.
- **DuckDB** – загрузка данных по HTTP и копирование их в Minio/PostgreSQL.
- **PostgreSQL** – хранилище данных (слои ODS и DM).
- **Minio** – S3-совместимое хранилище файлов.
- **Metabase** – инструмент визуализации данных.

## Процесс работы пайплайна

1. **raw_from_api_to_s3** – DAG ежедневно обращается к `https://earthquake.usgs.gov` и сохраняет полученные CSV данные в Minio в виде Parquet-файлов.
2. **raw_from_s3_to_pg** – после появления файла в Minio данные переносятся в таблицу `ods.fct_earthquake` в PostgreSQL.
3. **fct_avg_day_earthquake** и **fct_count_day_earthquake** – на основе таблицы `ods.fct_earthquake` считаются средняя магнитуда и количество землетрясений за сутки, результаты сохраняются в схему `dm`.
4. Сервисы поднимаются через `docker-compose up -d`, после чего DAG-и можно запускать из веб‑интерфейса Airflow.

Все переменные подключения (доступ к Minio, пароль Postgres и т.д.) задаются через Airflow Variables.

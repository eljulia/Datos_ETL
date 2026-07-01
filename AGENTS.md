# AGENTS.md

## Proyecto

Pipeline ETL de Netflix: extrae de una base MySQL transaccional, transforma con pandas, carga en un data warehouse dimensional.

## Configuracion

- MySQL via Docker en puerto 3310, root/root.
- Venv de Python en `ETL/venv/`. Activarlo antes de ejecutar el notebook.
- Dependencias en `ETL/requirements.txt` (SQLAlchemy, mysql-connector-python, pandas, jupyter).

```powershell
ETL/venv/Scripts/Activate.ps1
pip install -r ETL/requirements.txt
```

## Orden de inicio

1. `docker compose up -d` (inicia MySQL en 3310, ejecuta SQL de inicializacion automaticamente)
2. Abrir y ejecutar `ETL/ETL.ipynb`

## Bases de datos

- `db_movies_netflix_transact` -- BD transaccional origen (esquema en `mysql/db_movies_neflix_transact.sql`)
- `dw_netflix` -- BD destino dimensional (esquema en `mysql/data_warehouse_netflix.sql`)

Tablas: dimMovie, dimUser, FactWatchs.

## Strings de conexion (notebook)

- Extraccion: `mysql+mysqlconnector://root:root@127.0.0.1:3310/db_movies_netflix_transact`
- Carga: `mysql://root:root@127.0.0.1:3310/dw_netflix`

El driver `mysql+mysqlconnector` es obligatorio para la BD transaccional. El esquema `mysql://` falla si falta `MySQLdb`.

## Archivos de datos

- `ETL/data/users.csv` -- delimitado por pipe (`|`), leer con `sep='|'`
- `ETL/data/Awards_movie.csv` -- delimitado por coma, columna con typo `Aware` se renombra a `Award` en el notebook
- `ETL/data/Awards_participants.csv` -- no se usa en el notebook

## Flujo del notebook

1. Lee BD transaccional via SQLAlchemy -> DataFrame movies_data
2. Lee Awards_movie.csv -> merge por movieID
3. Elimina columna IdAward, renombra columnas para que coincidan con esquema dimMovie
4. Escribe a `dw_netflix.dimMovie` via `to_sql`
5. Lee users.csv -> renombra idUser a userID -> escribe a `dw_netflix.dimUser`
6. Cross-join de userIDs y movieIDs -> genera rating/timestamp aleatorio -> escribe a `dw_netflix.FactWatchs`

## Estructura de directorios

```
mysql/           Dockerfile + scripts SQL de inicializacion
ETL/             ETL.ipynb, data/, requirements.txt, venv/
```

## Notas

- No hay framework de tests, linter ni typechecker configurado.
- No hay `.gitignore` raiz (solo `ETL/venv/.gitignore`).
- No hay CI/CD automatizado.
- `Awards_movie.csv` tiene un typo en el encabezado de columna (`Aware` en vez de `Award`).

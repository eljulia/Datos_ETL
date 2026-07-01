<div align="center">

# Netflix ETL Pipeline

![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-2.0-150458?style=for-the-badge&logo=pandas&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=for-the-badge&logo=jupyter&logoColor=white)

**Pipeline ETL que extrae datos desde una base MySQL transaccional, los transforma con Pandas y los carga en un Data Warehouse dimensional para analisis de Netflix.**

</div>

---

## Indice

- [Descripcion General](#descripcion-general)
- [Arquitectura](#arquitectura)
- [Stack Tecnologico](#stack-tecnologico)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Base de Datos Transaccional](#base-de-datos-transaccional)
- [Base de Datos Dimensional](#base-de-datos-dimensional)
- [Requisitos Previos](#requisitos-previos)
- [Instalacion y Ejecucion](#instalacion-y-ejecucion)
- [Flujo del ETL](#flujo-del-etl)
- [Resultado Final](#resultado-final)

---

## Descripcion General

Este proyecto implementa un proceso **ETL** (Extract, Transform, Load) para datos de Netflix. El pipeline extrae informacion desde una base de datos transaccional MySQL, aplica transformaciones con **Pandas** y **Jupyter Notebooks**, y finalmente carga los datos en un modelo dimensional listo para analisis.

> El objetivo es transformar un esquema relacional normalizado en un esquema de estrella optimizado para consultas analiticas.

---

## Arquitectura

```
                  +------------------+
                  |  Awards_movie    |
                  |  (CSV)           |
                  +--------+---------+
                           |
+----------------+         v         +-------------------+
|  MySQL         |     Transform     |   MySQL           |
|  Transaccional +----> Pandas ----->+   Data Warehouse  |
|  (Origen)      |                   |   (Destino)       |
+----------------+                   +-------------------+
       |                                      |
       | - movie                              | - dimMovie
       | - person                             | - dimUser
       | - participant                        | - FactWatchs
       | - gender                             |
       | - movie_gender                       |
       +------------------+                   |
                          |                   |
                  +-------v--------+          |
                  |  users.csv     |          |
                  |  (Pipe-delim)  |          |
                  +----------------+          |
                                               |
                  +----------------+           |
                  | Docker Compose |<----------+
                  | (MySQL:3310)   |
                  +----------------+
```

---

## Stack Tecnologico

| Tecnologia | Proposito |
|---|---|
| ![Python](https://img.shields.io/badge/-Python-3776AB?style=flat-square&logo=python&logoColor=white) | Lenguaje principal del ETL |
| ![Pandas](https://img.shields.io/badge/-Pandas-150458?style=flat-square&logo=pandas&logoColor=white) | Transformacion y manipulacion de datos |
| ![MySQL](https://img.shields.io/badge/-MySQL-4479A1?style=flat-square&logo=mysql&logoColor=white) | Base de datos transaccional y data warehouse |
| ![Docker](https://img.shields.io/badge/-Docker-2496ED?style=flat-square&logo=docker&logoColor=white) | Contenedor de MySQL |
| ![Jupyter](https://img.shields.io/badge/-Jupyter-F37626?style=flat-square&logo=jupyter&logoColor=white) | Entorno de ejecucion del ETL |
| ![SQLAlchemy](https://img.shields.io/badge/-SQLAlchemy-D71F00?style=flat-square&logo=python&logoColor=white) | ORM para conexion a MySQL |

### Dependencias principales

- `SQLAlchemy==2.0.29`
- `mysql-connector-python==8.3.0`
- `mysqlclient==2.2.4`
- `pandas`
- `jupyter`

---

## Estructura del Proyecto

```
analisis_de_datos/
|
+-- mysql/                          # Configuracion de base de datos
|   +-- Dockerfile                  # Imagen MySQL con init scripts
|   +-- db_movies_neflix_transact.sql   # Esquema transaccional origen
|   +-- data_warehouse_netflix.sql      # Esquema dimensional destino
|
+-- ETL/                            # Pipeline de datos
|   +-- ETL.ipynb                   # Notebook principal del ETL
|   +-- requirements.txt            # Dependencias de Python
|   +-- data/                       # Archivos de datos fuente
|   |   +-- users.csv               # Usuarios (pipe-delimited)
|   |   +-- Awards_movie.csv        # Premios de peliculas
|   |   +-- Awards_participants.csv # Premios de participantes (no usado)
|   +-- venv/                       # Entorno virtual de Python
|
+-- docker-compose.yml              # Orquestacion del contenedor MySQL
+-- AGENTS.md                       # Instrucciones para agentes de IA
+-- README.md                       # Este archivo
```

---

## Base de Datos Transaccional

Base de origen (`db_movies_netflix_transact`) con esquema normalizado:

### Tabla: `movie`

| Columna | Tipo | Descripcion |
|---|---|---|
| movieID | VARCHAR(8) PK | Identificador de pelicula |
| movieTitle | VARCHAR(100) | Titulo |
| releaseDate | DATE | Fecha de estreno |
| originalLanguage | VARCHAR(100) | Idioma original |
| link | VARCHAR(50) | Enlace a Netflix |

### Tabla: `person`

| Columna | Tipo | Descripcion |
|---|---|---|
| personID | VARCHAR(8) PK | Identificador de persona |
| name | VARCHAR(100) | Nombre completo |
| birthday | DATE | Fecha de nacimiento |

### Tabla: `participant`

| Columna | Tipo | Descripcion |
|---|---|---|
| movieId | VARCHAR(8) PK, FK | Referencia a movie |
| personId | VARCHAR(8) | Referencia a person |
| participantRole | VARCHAR(30) | Rol (Actor, Director, etc.) |

### Tabla: `gender`

| Columna | Tipo | Descripcion |
|---|---|---|
| genderId | INTEGER PK | Identificador de genero |
| name | VARCHAR(100) | Nombre del genero |

### Tabla: `movie_gender`

| Columna | Tipo | Descripcion |
|---|---|---|
| movieId | VARCHAR(8) PK, FK | Referencia a movie |
| genderId | INTEGER | Referencia a gender |

---

## Base de Datos Dimensional

Data warehouse destino (`dw_netflix`) con esquema de estrella:

### Tabla: `dimMovie` (Dimension Pelicula)

| Columna | Tipo | Descripcion |
|---|---|---|
| movieID | VARCHAR(8) PK | Identificador de pelicula |
| title | VARCHAR(100) | Titulo |
| releaseMovie | DATE | Fecha de estreno |
| gender | VARCHAR(100) | Genero cinematografico |
| participantName | VARCHAR(100) | Nombre del participante |
| roleparticipant | VARCHAR(100) | Rol del participante |
| awardMovie | VARCHAR(20) | Premio obtenido |

### Tabla: `dimUser` (Dimension Usuario)

| Columna | Tipo | Descripcion |
|---|---|---|
| userID | INTEGER PK | Identificador de usuario |
| username | VARCHAR(100) | Nombre de usuario |
| country | VARCHAR(100) | Pais |
| subscription | VARCHAR(100) | Tipo de suscripcion |

### Tabla: `FactWatchs` (Tabla de Hechos)

| Columna | Tipo | Descripcion |
|---|---|---|
| userID | INTEGER FK | Referencia a dimUser |
| movieID | VARCHAR(8) FK | Referencia a dimMovie |
| rating | DECIMAL(2,1) | Calificacion (0.0 - 5.0) |
| timestamp | TIMESTAMP | Momento de la visualizacion |

> **Relaciones:** `FactWatchs.userID` -> `dimUser.userID`, `FactWatchs.movieID` -> `dimMovie.movieID`

---

## Requisitos Previos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado y en ejecucion
- [Python 3.11 o + ](https://www.python.org/downloads/) instalado
- Entorno virtual creado en `ETL/venv/`

---

## Instalacion y Ejecucion

### 1. Clonar el repositorio

```bash
git clone https://github.com/tu-usuario/analisis_de_datos.git
cd analisis_de_datos
```

### 2. Iniciar MySQL con Docker

```powershell
docker compose up -d
```

Esto levanta MySQL en el puerto `3310` con las bases de datos y tablas creadas automaticamente.

### 3. Activar entorno virtual e instalar dependencias

```powershell
ETL/venv/Scripts/Activate.ps1
pip install -r ETL/requirements.txt
```

> Si no tienes el entorno virtual creado, puedes crearlo con:
> ```powershell
> python -m venv ETL/venv
> ```

### 4. Ejecutar el ETL

Abrir `ETL/ETL.ipynb` en Jupyter y ejecutar todas las celdas en orden.

```powershell
jupyter notebook ETL/ETL.ipynb
```

---

## Flujo del ETL

### Paso 1: Extraccion

Se conecta a la base transaccional `db_movies_netflix_transact` usando SQLAlchemy con el driver `mysql+mysqlconnector` y ejecuta una consulta JOIN que combina las tablas `movie`, `participant`, `person`, `movie_gender` y `gender`.

```python
engine = db.create_engine("mysql+mysqlconnector://root:root@127.0.0.1:3310/db_movies_netflix_transact")
```

### Paso 2: Transformacion

- Se lee el archivo `Awards_movie.csv` y se mergea con los datos extraidos por `movieID`
- Se renombra la columna `Aware` (typo en el CSV) a `Award`
- Se elimina la columna `IdAward`
- Se renombran columnas para alinearse con el esquema dimensional (`releaseDate` -> `releaseMovie`, `Award` -> `awardMovie`)
- Se lee `users.csv` (delimitado por pipe `|`) y se renombra `idUser` a `userID`

### Paso 3: Carga

Los datos transformados se cargan en el data warehouse `dw_netflix`:

```python
movie_data.to_sql('dimMovie', conn, if_exists='append', index=False)
users.to_sql('dimUser', conn, if_exists='append', index=False)
```

### Paso 4: Generacion de Hechos

- Se genera un cross-join entre todos los `userID` y `movieID`
- Se asignan calificaciones aleatorias (0.0 - 5.0) y timestamps (Ene-Abr 2024)
- Los datos se cargan en `FactWatchs`

---

## Resultado Final

El pipeline produce un modelo dimensional listo para consultas analiticas como:

```sql
-- Peliculas mejor calificadas por pais
SELECT
    u.country,
    m.title,
    AVG(f.rating) as avg_rating,
    COUNT(*) as total_views
FROM FactWatchs f
JOIN dimUser u ON f.userID = u.userID
JOIN dimMovie m ON f.movieID = m.movieID
GROUP BY u.country, m.title
ORDER BY avg_rating DESC;
```

---

<div align="center">

**Proyecto desarrollado como parte del aprendizaje en ETL y modelamiento dimensional de datos**

</div>

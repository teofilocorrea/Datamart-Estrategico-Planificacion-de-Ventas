# 🧱 STAGING (Esquema `stg`) — Espejo de Fuentes + Auditoría

La capa **staging** es un **almacén temporal y fiel** de los archivos de origen (CSV/XLSX).  
Su objetivo es **preservar los datos tal como llegan**, habilitar **EDA** (conteos, nulos, duplicados, rangos) y aportar **trazabilidad** antes de transformarlos a las capas `dim`/`fact`.

> **Regla clave:** en `stg` mantenemos **los nombres originales de columnas** y solo añadimos **3 campos de auditoría** para control y reproducción.

---

## 🧭 Campos extra de staging (auditoría)

Estos 3 campos se agregan a **todas** las tablas de `stg`:

- **`ingestion_ts` (DATETIME2)**  
  Momento exacto en que se insertó la fila en staging.  
  *Usos:* auditoría de cargas, versionado de datos, reprocesos.

- **`source_name` (NVARCHAR(200))**  
  Nombre del archivo/hoja de origen (por ejemplo, `DimAccount.csv`, `FactStrategyPlan.xlsx`).  
  *Usos:* trazabilidad, diagnóstico de errores por fuente, mezcla de entregas (v1, v2…).

- **`row_hash` (CHAR(32))**  
  Hash (MD5) calculado a partir de columnas clave.  
  *Usos:* detección de **cambios** entre cargas (delta) y **duplicados exactos**, sin herramientas ETL.

> **Importante:** estos campos **no cambian los datos de negocio**; solo agregan **contexto técnico** y control.

**Leyenda de la columna “Fuente” en las tablas:**
- `CSV`/`XLSX` → proviene del archivo original.
- `Extra` → campo agregado por el proceso de staging (auditoría).

---

## 📂 Tablas del esquema `stg`

### 🗂 `stg.dim_account`
| Columna             | Tipo SQL Server | Fuente |
|---------------------|-----------------|--------|
| AccountDescription  | NVARCHAR(255)   | CSV    |
| AccountKey          | INT             | CSV    |
| AccountLabel        | INT             | CSV    |
| AccountName         | NVARCHAR(255)   | CSV    |
| AccountType         | NVARCHAR(50)    | CSV    |
| CustomMemberOptions | NVARCHAR(50)    | CSV    |
| CustomMembers       | NVARCHAR(255)   | CSV    |
| ETLLoadID           | INT             | CSV    |
| Operator            | NVARCHAR(10)    | CSV    |
| ParentAccountKey    | INT             | CSV    |
| ValueType           | NVARCHAR(50)    | CSV    |
| ingestion_ts        | DATETIME2       | Extra  |
| source_name         | NVARCHAR(200)   | Extra  |
| row_hash            | CHAR(32)        | Extra  |

---

### 🗂 `stg.dim_date`
> Si detectas mezcla de formatos de fecha, cambia `Datekey/FirstOfMonth/FirstOfYear` a `NVARCHAR(20)` aquí y tipifica en `dim_date`.

| Columna         | Tipo SQL Server | Fuente |
|-----------------|-----------------|--------|
| Datekey         | DATE            | CSV    |
| DayOfMonth      | TINYINT         | CSV    |
| MonthNumber     | TINYINT         | CSV    |
| FirstOfMonth    | DATE            | CSV    |
| ShortMonth      | CHAR(3)         | CSV    |
| MonthName       | NVARCHAR(30)    | CSV    |
| MonthNumberYear | NVARCHAR(30)    | CSV    |
| week            | TINYINT         | CSV    |
| WeekNumber      | TINYINT         | CSV    |
| DayOfWeek       | TINYINT         | CSV    |
| QuarterFull     | NVARCHAR(30)    | CSV    |
| Qrt             | CHAR(2)         | CSV    |
| QuarterYear     | NVARCHAR(30)    | CSV    |
| Year            | INT             | CSV    |
| FirstOfYear     | DATE            | CSV    |
| ingestion_ts    | DATETIME2       | Extra  |
| source_name     | NVARCHAR(200)   | Extra  |
| row_hash        | CHAR(32)        | Extra  |

---

### 🗂 `stg.dim_entity`
> Fechas de la fuente en formato ambiguo (`31/7/96`) → se guardan como **texto** en `stg` y se convierten a `DATE` en `dim`.

| Columna            | Tipo SQL Server  | Fuente |
|--------------------|------------------|--------|
| EntityKey          | INT              | CSV    |
| EntityLabel        | INT              | CSV    |
| ParentEntityKey    | INT NULL         | CSV    |
| ParentEntityLabel  | NVARCHAR(50) NULL| CSV    |
| EntityName         | NVARCHAR(150)    | CSV    |
| EntityDescription  | NVARCHAR(150)    | CSV    |
| EntityType         | NVARCHAR(50)     | CSV    |
| StartDate          | NVARCHAR(20)     | CSV    |
| EndDate            | NVARCHAR(20) NULL| CSV    |
| Status             | NVARCHAR(20)     | CSV    |
| ingestion_ts       | DATETIME2        | Extra  |
| source_name        | NVARCHAR(200)    | Extra  |
| row_hash           | CHAR(32)         | Extra  |

---

### 🗂 `stg.dim_product_category`
| Columna                     | Tipo SQL Server | Fuente |
|----------------------------|-----------------|--------|
| ProductCategoryKey         | INT             | CSV    |
| ProductCategoryLabel       | INT             | CSV    |
| ProductCategoryName        | NVARCHAR(100)   | CSV    |
| ProductCategoryDescription | NVARCHAR(150)   | CSV    |
| ETLLoadID                  | INT             | CSV    |
| ingestion_ts               | DATETIME2       | Extra  |
| source_name                | NVARCHAR(200)   | Extra  |
| row_hash                   | CHAR(32)        | Extra  |

---

### 🗂 `stg.dim_scenario`
| Columna             | Tipo SQL Server | Fuente |
|---------------------|-----------------|--------|
| ScenarioKey         | INT             | CSV    |
| ScenarioLabel       | INT             | CSV    |
| ScenarioName        | NVARCHAR(100)   | CSV    |
| ScenarioDescription | NVARCHAR(150)   | CSV    |
| ingestion_ts        | DATETIME2       | Extra  |
| source_name         | NVARCHAR(200)   | Extra  |
| row_hash            | CHAR(32)        | Extra  |

---

### 🗂 `stg.fact_strategy_plan`
> `Datekey` viene como `1/2/18 00:00:00` (ambigua). Se preserva como **NVARCHAR(20)** y se tipifica en la FACT final.

| Columna             | Tipo SQL Server | Fuente |
|---------------------|-----------------|--------|
| StrategyPlanKey     | INT             | XLSX   |
| Datekey             | NVARCHAR(20)    | XLSX   |
| EntityKey           | INT             | XLSX   |
| ScenarioKey         | INT             | XLSX   |
| AccountKey          | INT             | XLSX   |
| CurrencyKey         | INT             | XLSX   |
| ProductCategoryKey  | INT             | XLSX   |
| Amount              | NUMERIC(18,2)   | XLSX   |
| ingestion_ts        | DATETIME2       | Extra  |
| source_name         | NVARCHAR(200)   | Extra  |
| row_hash            | CHAR(32)        | Extra  |

---

## 🔄 Próximo paso (resumen)
1. Crear esquema: `CREATE SCHEMA stg;`  
2. Crear tablas `stg.*` (espejo + auditoría).  
3. Cargar archivos originales en `stg`.  
4. Ejecutar **EDA** en `stg` (nulos, duplicados, rangos, outliers).  
5. Diseñar `dim.*` y `fact.*` con **nombres normalizados** y **surrogate keys**, convirtiendo fechas/textos donde corresponda.
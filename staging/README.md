# 📂 Carpeta Staging

La carpeta **staging/** contiene los archivos **originales** en formato CSV y Excel que sirven como punto de partida para la construcción del DataMart.  
En este espacio se almacenan los datos **tal como fueron recibidos**, sin modificaciones ni transformaciones.

---

## 🎯 Propósito
- Centralizar los archivos de origen antes de cargarlos en la base de datos.  
- Mantener una copia “cruda” para trazabilidad y auditoría.  
- Facilitar el proceso de **EDA (Exploratory Data Analysis)** antes de aplicar reglas de limpieza y modelado dimensional.  

---

## 📑 Archivos esperados

| Archivo                 | Descripción                                | Tabla Staging   |
|--------------------------|--------------------------------------------|-----------------|
| `DimAccount.csv`        | Información de cuentas                     | `stg.dim_account` |
| `DimDate.csv`           | Calendario de fechas                       | `stg.dim_date` |
| `DimEntity.csv`         | Entidades de negocio                       | `stg.dim_entity` |
| `DimProductCategory.csv`| Categorías de producto                      | `stg.dim_product_category` |
| `DimScenario.csv`       | Escenarios de planificación                | `stg.dim_scenario` |
| `FactStrategyPlan.xlsx` | Planificación estratégica de ventas (hechos)| `stg.fact_strategy_plan` |

---

## ⚠️ Notas importantes
- Los datos en esta carpeta **no deben modificarse manualmente**.  
- Si un archivo completo no puede ser subido al repositorio (por tamaño), se incluirá una **muestra representativa** (`sample_*.csv`).  
- Para ejecutar los scripts de carga, asegúrate de colocar aquí los archivos originales.  

---

## 🔗 Próximos pasos
1. Cargar los archivos en las tablas `stg.*` de SQL Server mediante DataGrip.  
2. Ejecutar queries de **EDA en Staging** para validar calidad (nulos, duplicados, rangos de fechas, etc.).  
3. Documentar hallazgos en `docs/hallazgos_eda.md`.  
# COVID-19 Colombia ETL & Data Warehouse Project

## Overview

This project implements a comprehensive **ETL (Extract, Transform, Load)** solution for analyzing COVID-19 cases in Colombia using a **Star Schema** dimensional model in a Data Warehouse. The solution includes data integration with **SQL Server Integration Services (SSIS)**, a **Tabular Cube** for OLAP analysis, and interactive **Power BI** dashboards for visualization.

## 📋 Table of Contents

- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Data Model](#data-model)
- [ETL Process](#etl-process)
- [Technologies](#technologies)
- [Installation & Setup](#installation--setup)
- [Usage](#usage)
- [Database Backups](#database-backups)

---

## Architecture

The project follows a **Data Warehouse architecture** with three main layers:

1. **Source Layer**: Raw COVID-19 positive cases data from Colombia (`TBL_CASOS_POSITIVOS`)
2. **Data Warehouse Layer**: Star schema with dimension and fact tables
3. **Analysis Layer**: Tabular cube for OLAP operations
4. **Presentation Layer**: Power BI dashboards for data visualization

```
Source Data (CSV) → ETL Process (SSIS) → Data Warehouse (SQL Server) 
    → Tabular Cube (SSAS) → Dashboard (Power BI)
```

---

## Project Structure

```
ETL_COVID_19_COLOMBIA/
│
├── DATOS/                                  # Source data files
│   └── CASOS_POSITIVOS_COVID_19_COLOMBIA.csv
│
├── DISENO/                                 # Design artifacts
│   ├── DIAGRAMA/                           # Physical model diagram
│   │   └── ETL_COVID_19_COL_MODELO_ESTRELLA_FISICO.drawio
│   └── SCRIPT/                             # DDL scripts
│       └── COVID_19_COL_DDL_ETL_MODELO_ESTRELLA.sql
│
├── SOLUCION/                               # Implementation
│   ├── ETL_COVID_19_COL/                   # SSIS ETL solution
│   │   ├── ETL_ORQUESTADOR.dtsx           # Master orchestrator package
│   │   ├── ETL_DIM_TIEMPO_T0.dtsx         # Time dimension load
│   │   ├── ETL_DIM_UBICACION_T0.dtsx      # Location dimension load
│   │   ├── ETL_DIM_PERSONA_T0.dtsx        # Person dimension load
│   │   ├── ETL_DIM_CASO_T2.dtsx           # Case dimension load (SCD Type 2)
│   │   └── ETL_FACT_CASOS.dtsx            # Fact table load
│   │
│   └── CUBO_TABULAR_CASOS_COVID_19_COL/   # SSAS Tabular Model
│       └── Model.bim
│
├── POWER_BI/                               # Dashboard files
│   ├── DASHBOARD_COVID_19_COL.pbix
│   └── DASHBOARD_COVID_19_COL.png
│
└── RESPALDO/                               # Backups
    ├── DWH_COVID_19_COLOMBIA.bak          # Database backup
    └── CUBO_TABULAR_CASOS_COVID_19_COL.abf # Tabular cube backup
```

---

## Data Model

### Star Schema Design

The solution implements a **Star Schema** with 4 dimension tables and 1 fact table:

#### 📅 **Dimension: TBL_DIM_TIEMPO_T0** (Time Dimension - Type 0 SCD)
Stores time-related attributes for date analysis.

| Column | Type | Description |
|--------|------|-------------|
| SK_DIM_TIEMPO | BIGINT (PK) | Surrogate key (format: YYYYMMDD) |
| DT_FECHA_NOTIFICACION | DATE | Notification date |
| INT_DIA | SMALLINT | Day |
| INT_MES | SMALLINT | Month number |
| INT_ANIO | SMALLINT | Year |
| INT_PERIODO | INT | Period (YYYYMM) |
| INT_SEMANA_MES | SMALLINT | Week of month |
| STR_MES | NVARCHAR(50) | Month name |
| STR_SEMESTRE | NVARCHAR(50) | Semester |
| DT_FECHA_CARGUE | DATETIME | Load timestamp |

**Key Features:**
- Auto-generates dates between `START_DATE` and `END_DATE` parameters
- Prevents duplicate date entries
- Parametrized for flexible date range generation

---

#### 📍 **Dimension: TBL_DIM_UBICACION_T0** (Location Dimension - Type 0 SCD)
Contains geographic information about departments and municipalities.

| Column | Type | Description |
|--------|------|-------------|
| SK_DIM_UBICACION | INT (PK, Identity) | Surrogate key |
| STR_DEPARTAMENTO | NVARCHAR(50) | Department name |
| STR_MUNICIPIO | NVARCHAR(50) | Municipality name |
| STR_UBICACION_DEPARTAMENTO | NVARCHAR(100) | Formatted department location |
| STR_UBICACION_MUNICIPIO | NVARCHAR(150) | Formatted municipality location |
| DT_FECHA_CARGUE | DATETIME | Load timestamp |

**Key Features:**
- Extracts distinct department-municipality combinations
- Uses lookup transformation to avoid duplicates

---

#### 👤 **Dimension: TBL_DIM_PERSONA_T0** (Person Dimension - Type 0 SCD)
Stores demographic information about patients.

| Column | Type | Description |
|--------|------|-------------|
| SK_DIM_PERSONA | INT (PK, Identity) | Surrogate key |
| INT_EDAD | TINYINT | Age |
| STR_CATEGORIA_EDAD | NVARCHAR(50) | Age category |
| CHR_SEXO | NCHAR(10) | Gender |
| DT_FECHA_CARGUE | DATETIME | Load timestamp |

**Age Categories:**
- Early Childhood (1-5)
- Middle Childhood (6-11)
- Early Adolescence (12-14)
- Middle Adolescence (15-17)
- Late Adolescence (18-20)
- Emerging Adulthood (21-24)
- Young Adulthood (25-39)
- Middle Adulthood (40-54)
- Late Middle Adulthood (55-64)
- Early Senior Adulthood (65-74)
- Senior Adulthood (75-84)
- Advanced Old Age (85-94)
- Extreme Longevity (95-113)

---

#### 🏥 **Dimension: TBL_DIM_CASO_T2** (Case Dimension - Type 2 SCD)
Tracks case-specific attributes with historical tracking (Slowly Changing Dimension Type 2).

| Column | Type | Description |
|--------|------|-------------|
| SK_DIM_CASO | INT (PK, Identity) | Surrogate key |
| INT_ID_CASO | INT | Business key (case ID) |
| STR_TIPO_CONTAGIO | NVARCHAR(50) | Contagion type |
| STR_UBICACION_CASO | NVARCHAR(50) | Case location |
| STR_ESTADO | NVARCHAR(50) | Status |
| STR_CONDICION_FINAL | NVARCHAR(50) | Final condition |
| STR_TIPO_RECUPERACION | NVARCHAR(50) | Recovery type |
| DT_FECHA_REPORTE_WEB | DATE | Web report date |
| DT_FECHA_CARGUE | DATETIME | Load timestamp |
| DT_FECHA_INICIO | DATETIME | Start date (SCD Type 2) |
| DT_FECHA_VIGENCIA | DATETIME | End date (SCD Type 2) |
| INT_FLAG | TINYINT | Active record flag (1=active, 0=inactive) |

**Key Features:**
- Implements **Slowly Changing Dimension Type 2** for historical tracking
- Tracks changes in case attributes over time
- Uses flag to identify current vs. historical records

---

#### 📊 **Fact Table: TBL_FACT_CASOS** (Cases Fact Table)
Central fact table containing measures and foreign keys to dimensions.

| Column | Type | Description |
|--------|------|-------------|
| SK_DIM_CASO | INT (FK) | Foreign key to Case dimension |
| SK_DIM_PERSONA | INT (FK) | Foreign key to Person dimension |
| SK_DIM_UBICACION | INT (FK) | Foreign key to Location dimension |
| SK_DIM_TIEMPO | INT (FK) | Foreign key to Time dimension |
| BIT_RECUPERADO | BIT | Recovered flag |
| BIT_MUERTO | BIT | Deceased flag |
| INT_DIAS_REPORTE | INT | Days to report |
| INT_DIAS_RETRASO_DIAGNOSTICO | INT | Days of diagnostic delay |
| INT_DIAS_RECUPERACION | INT | Days to recovery |
| INT_DIAS_HASTA_MUERTE | INT | Days to death |
| DT_FECHA_CARGUE | DATETIME | Load timestamp |

**Metrics Included:**
- Recovery and mortality indicators
- Time-based measures for reporting, diagnosis, recovery, and death
- Full dimensional context for multidimensional analysis

---

## ETL Process

### Master Orchestrator (ETL_ORQUESTADOR.dtsx)

The ETL process is orchestrated through a master package that executes in **three sequential phases**:

```
Phase 1: SEQ_DIMS (Dimension Loading)
    ├── ETL_DIM_TIEMPO_T0
    ├── ETL_DIM_UBICACION_T0
    ├── ETL_DIM_PERSONA_T0
    └── ETL_DIM_CASO_T2
    
Phase 2: SEQ_FACT (Fact Table Loading)
    └── ETL_FACT_CASOS
    
Phase 3: SEQ_PROCESS_CUBE (Cube Processing)
    └── AS_PROCESSFULL_CUBO_TABULAR_CASOS_COVID_19_COL
```

### ETL Package Details

#### 1️⃣ **ETL_DIM_TIEMPO_T0.dtsx** - Time Dimension Load
- **Type**: SQL Execute Task
- **Process**: Generates date records dynamically using parameters
- **Parameters**: 
  - `START_DATE`: 2020-03-21
  - `END_DATE`: 2022-01-23
- **Features**:
  - Generates all dates within range
  - Calculates day, month, year, period, week of month
  - Assigns month names and semester
  - Prevents duplicates

#### 2️⃣ **ETL_DIM_UBICACION_T0.dtsx** - Location Dimension Load
- **Source**: `TBL_CASOS_POSITIVOS` (distinct department/municipality)
- **Transformations**:
  - Extract unique location combinations
  - Derived Column: Creates formatted location strings
  - Lookup: Checks for existing records
  - Conditional Split: Routes new vs. duplicate records
- **Destination**: `TBL_DIM_UBICACION_T0`
- **Auditing**: Row count for repeated records

#### 3️⃣ **ETL_DIM_PERSONA_T0.dtsx** - Person Dimension Load
- **Source**: `TBL_CASOS_POSITIVOS` (distinct age, gender, age category)
- **Transformations**:
  - Extract unique demographic combinations
  - Age categorization logic (CASE statement)
  - Derived Column: Create business key, data type conversions
  - Lookup: Check for existing records
  - Conditional Split: Separate new from duplicate records
- **Destination**: `TBL_DIM_PERSONA_T0`
- **Features**: Tracks 13 different age categories

#### 4️⃣ **ETL_DIM_CASO_T2.dtsx** - Case Dimension Load (SCD Type 2)
- **Source**: `TBL_CASOS_POSITIVOS` (case details)
- **Transformations**:
  - Conditional Split: Filter NULL case IDs
  - Derived Column: Create business keys and conversions
  - Lookup: Identify existing cases
  - Conditional Split: Separate INSERT vs. UPDATE operations
  - OLE DB Command: Update existing records (expire old version)
- **Destination**: `TBL_DIM_CASO_T2`
- **Features**: 
  - Implements SCD Type 2 (history preservation)
  - Manages record versioning with flags
  - Tracks start and end dates for each version

#### 5️⃣ **ETL_FACT_CASOS.dtsx** - Fact Table Load
- **Pre-Processing**: TRUNCATE TABLE (full refresh strategy)
- **Source**: `TBL_CASOS_POSITIVOS` with calculated measures
- **Transformations**:
  - Calculate time-based metrics (days to report, recovery, death, etc.)
  - Derived Column: Business keys and conversions
  - Lookup x3: Resolve dimension keys
    - `LKP_DIM_TIEMPO` (by date)
    - `LKP_DIM_UBICACION_T0` (by location)
    - `LKP_DIM_PERSONA_T0` (by demographics)
    - `LKP_DIM_CASO_T2` (by case ID)
- **Destination**: `TBL_FACT_CASOS`
- **Strategy**: Full load (truncate and reload)

#### 6️⃣ **Tabular Cube Processing**
- **Task**: Analysis Services Processing Task
- **Action**: Full refresh of all tables in tabular model
- **Tables Processed**:
  - TBL_DIM_TIEMPO_T0
  - TBL_DIM_UBICACION_T0
  - TBL_DIM_PERSONA_T0
  - TBL_DIM_CASO_T2
  - TBL_FACT_CASOS
- **Purpose**: Make updated data available for OLAP queries

---

## Technologies

| Technology | Purpose |
|------------|---------|
| **SQL Server 2022** | Database engine for Data Warehouse |
| **SSIS (SQL Server Integration Services)** | ETL tool for data integration |
| **SSAS Tabular (SQL Server Analysis Services)** | OLAP cube for multidimensional analysis |
| **Power BI Desktop** | Business intelligence and visualization |
| **Draw.io** | Data model diagram design |
| **Visual Studio / SSDT** | Development environment |

---

## Installation & Setup

### Prerequisites

- SQL Server 2022 (or compatible version)
- SQL Server Integration Services (SSIS)
- SQL Server Analysis Services Tabular Mode
- SQL Server Data Tools (SSDT) or Visual Studio
- Power BI Desktop

### Setup Steps

1. **Restore Database**
   ```sql
   RESTORE DATABASE [DWH_COVID_19_COLOMBIA]
   FROM DISK = 'path\to\RESPALDO\DWH_COVID_19_COLOMBIA.bak'
   WITH MOVE 'DWH_COVID_19_COLOMBIA' TO 'C:\...\DWH_COVID_19_COLOMBIA.mdf',
        MOVE 'DWH_COVID_19_COLOMBIA_log' TO 'C:\...\DWH_COVID_19_COLOMBIA_log.ldf'
   ```

2. **Load Source Data**
   - Import `DATOS/CASOS_POSITIVOS_COVID_19_COLOMBIA.csv` into `TBL_CASOS_POSITIVOS` table

3. **Configure Connections**
   - Open `SOLUCION/ETL_COVID_19_COL/ETL_COVID_19_COL.sln` in Visual Studio
   - Update connection managers:
     - `CNX_OLEDB_DWH_COVID_19_COLOMBIA`: Point to your SQL Server instance
     - `CNX_OLAP_TABULAR_UPTC`: Point to your SSAS Tabular instance

4. **Execute ETL**
   - Run `ETL_ORQUESTADOR.dtsx` package
   - Monitor execution in SSIS

5. **Deploy Tabular Model**
   - Open `SOLUCION/CUBO_TABULAR_CASOS_COVID_19_COL/CUBO_TABULAR_CASOS_COVID_19_COL.sln`
   - Deploy to SSAS Tabular server
   - Process the model

6. **Open Dashboard**
   - Open `POWER_BI/DASHBOARD_COVID_19_COL.pbix` in Power BI Desktop
   - Refresh data connections
   - Publish to Power BI Service (optional)

---

## Usage

### Running the ETL Process

Execute the master orchestrator package:
```powershell
dtexec /File "path\to\ETL_ORQUESTADOR.dtsx"
```

Or run from Visual Studio/SSDT:
1. Open `ETL_COVID_19_COL.sln`
2. Right-click `ETL_ORQUESTADOR.dtsx`
3. Select "Execute Package"

### Querying the Data Warehouse

Example queries for analysis:

```sql
-- Total cases by department
SELECT 
    u.STR_DEPARTAMENTO,
    COUNT(*) as TotalCases,
    SUM(CAST(f.BIT_RECUPERADO AS INT)) as Recovered,
    SUM(CAST(f.BIT_MUERTO AS INT)) as Deceased
FROM TBL_FACT_CASOS f
INNER JOIN TBL_DIM_UBICACION_T0 u ON f.SK_DIM_UBICACION = u.SK_DIM_UBICACION
GROUP BY u.STR_DEPARTAMENTO
ORDER BY TotalCases DESC;

-- Cases by age category and gender
SELECT 
    p.STR_CATEGORIA_EDAD,
    p.CHR_SEXO,
    COUNT(*) as TotalCases
FROM TBL_FACT_CASOS f
INNER JOIN TBL_DIM_PERSONA_T0 p ON f.SK_DIM_PERSONA = p.SK_DIM_PERSONA
GROUP BY p.STR_CATEGORIA_EDAD, p.CHR_SEXO
ORDER BY p.INT_EDAD;

-- Monthly trend analysis
SELECT 
    t.INT_ANIO,
    t.STR_MES,
    COUNT(*) as TotalCases,
    AVG(f.INT_DIAS_RETRASO_DIAGNOSTICO) as AvgDiagnosticDelay,
    AVG(f.INT_DIAS_RECUPERACION) as AvgRecoveryDays
FROM TBL_FACT_CASOS f
INNER JOIN TBL_DIM_TIEMPO_T0 t ON f.SK_DIM_TIEMPO = t.SK_DIM_TIEMPO
GROUP BY t.INT_ANIO, t.INT_MES, t.STR_MES
ORDER BY t.INT_ANIO, t.INT_MES;
```

---

## Database Backups

The project includes backup files in the `RESPALDO/` directory:

- **DWH_COVID_19_COLOMBIA.bak**: Full database backup
- **CUBO_TABULAR_CASOS_COVID_19_COL.abf**: SSAS Tabular model backup

These can be used to restore the complete solution in a new environment.

---

## Key Features

✅ **Complete ETL Pipeline**: Automated data integration from source to analytics  
✅ **Star Schema Design**: Optimized for OLAP and query performance  
✅ **SCD Type 2 Implementation**: Historical tracking for case dimension  
✅ **Data Quality Controls**: Duplicate detection and auditing  
✅ **Scalable Architecture**: Modular design for easy maintenance  
✅ **OLAP Cube**: Fast multidimensional analysis with SSAS Tabular  
✅ **Interactive Dashboards**: Power BI visualizations for insights  
✅ **Parametrized ETL**: Flexible date range configuration  

---

## Data Insights

The solution enables analysis of:
- 📈 **Temporal Trends**: Cases over time (daily, monthly, yearly)
- 📍 **Geographic Distribution**: Cases by department and municipality
- 👥 **Demographics**: Cases by age category and gender
- 🏥 **Case Outcomes**: Recovery rates, mortality rates
- ⏱️ **Time Metrics**: Diagnostic delays, recovery times, time to death
- 🔄 **Historical Changes**: Case status changes over time (via SCD Type 2)

---

## Contributing

This project is part of a Data Warehouse and Data Mining course. For contributions or questions, please contact the project maintainers.

---

## License

This project is for educational purposes.

---

## Project Timeline

- **Data Period**: March 21, 2020 - January 23, 2022
- **Development**: 2025

---

## Contact & Support

For questions or issues related to this project, please refer to the course materials or contact the instructor.

---

**Last Updated**: November 2025

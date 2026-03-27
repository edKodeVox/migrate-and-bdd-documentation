---
tags:
  - migracion
  - base-de-datos
  - sql
  - documentacion
fecha_creacion: 2026-03-12
estado: en-progreso
---

# Documentación de Migración de Bases de Datos

## Índice

- [[#Descripción General]]
- [[#Entorno y Requisitos Previos]]
- [[#Arquitectura de la Migración]]
- [[#Mapeo de Tablas y Campos]]
- [[#Proceso de Migración Paso a Paso]]
- [[#Scripts de Migración]]
- [[#Validación y Pruebas]]
- [[#Gestión de Errores]]
- [[#Rollback]]
- [[#Registro de Cambios]]

---

## Descripción General

Este documento describe el proceso de migración de datos desde la base de datos externa de origen hacia nuestra base de datos SQL de destino.

| Parámetro         | Valor                          |
| ----------------- | ------------------------------ |
| **BD Origen**     | `[Nombre de la BD externa]`    |
| **BD Destino**    | `[Nombre de nuestra BD SQL]`   |
| **Motor Destino** | `[MySQL / PostgreSQL / MSSQL]` |
| **Responsable**   | `[Nombre del responsable]`     |
| **Fecha inicio**  | `[YYYY-MM-DD]`                 |
| **Fecha fin est.**| `[YYYY-MM-DD]`                 |

> [!NOTE]
> Completar los valores entre corchetes `[]` antes de comenzar el proceso.

---

## Entorno y Requisitos Previos

### Herramientas necesarias

- [ ] Acceso a la base de datos de origen (credenciales y permisos de lectura)
- [ ] Acceso a la base de datos de destino (credenciales y permisos de escritura)
- [ ] Cliente SQL (DBeaver, HeidiSQL, pgAdmin, SSMS, etc.)
- [ ] Script de migración revisado y aprobado
- [ ] Backup completo de la BD de destino antes de iniciar

### Credenciales de Acceso

> [!WARNING]
> No almacenar contraseñas en este documento. Usar un gestor de secretos o variables de entorno.

**BD Origen:**
```
Host:     [host_origen]
Puerto:   [puerto_origen]
BD:       [nombre_bd_origen]
Usuario:  [usuario_origen]
```

**BD Destino:**
```
Host:     [host_destino]
Puerto:   [puerto_destino]
BD:       [nombre_bd_destino]
Usuario:  [usuario_destino]
```

---

## Arquitectura de la Migración

```
┌─────────────────────┐         ┌──────────────────────┐
│   BD EXTERNA        │         │   BD SQL DESTINO      │
│   (Origen)          │──ETL──▶ │   (Nuestra SQL)       │
│                     │         │                       │
│  Tabla A  ──────────┼────────▶│  Tabla A'             │
│  Tabla B  ──────────┼────────▶│  Tabla B'             │
│  Tabla C  ──────────┼────────▶│  Tabla C'             │
└─────────────────────┘         └──────────────────────┘
```

### Fases del proceso

1. **Extracción** – Leer registros de la BD externa
2. **Transformación** – Adaptar tipos de datos, normalizar valores y aplicar reglas de negocio
3. **Carga** – Insertar/actualizar registros en la BD destino
4. **Validación** – Verificar integridad y conteos

---

## Mapeo de Tablas y Campos

### Tabla: `[nombre_tabla_origen]` → `[nombre_tabla_destino]`

| Campo Origen         | Tipo Origen  | Campo Destino        | Tipo Destino  | Transformación                        |
| -------------------- | ------------ | -------------------- | ------------- | ------------------------------------- |
| `id`                 | INT          | `id`                 | INT           | Sin cambio                            |
| `nombre`             | VARCHAR(100) | `nombre`             | VARCHAR(255)  | Sin cambio                            |
| `fecha_registro`     | DATE         | `fecha_creacion`     | DATETIME      | Agregar hora `00:00:00`               |
| `estado`             | CHAR(1)      | `activo`             | BOOLEAN       | `'A'` → `true`, `'I'` → `false`       |
| `[campo_origen]`     | `[tipo]`     | `[campo_destino]`    | `[tipo]`      | `[descripción de transformación]`     |

> [!TIP]
> Añadir una fila por cada campo que requiera mapeo. Indicar `NULL` en campo destino si el campo origen no tiene equivalente.

---

## Proceso de Migración Paso a Paso

### Paso 1 – Backup previo

```sql
-- Ejecutar en BD destino antes de cualquier operación
-- Registrar la fecha y hora del backup
-- Verificar que el backup sea válido antes de continuar
```

- [ ] Backup realizado en: `[ruta/nombre_del_backup]`
- [ ] Backup verificado: `[ ]`

---

### Paso 2 – Preparar el entorno de destino

```sql
-- Deshabilitar restricciones de clave foránea (si aplica)
-- MySQL
SET FOREIGN_KEY_CHECKS = 0;

-- PostgreSQL
SET session_replication_role = 'replica';

-- SQL Server
EXEC sp_msforeachtable "ALTER TABLE ? NOCHECK CONSTRAINT all";
```

---

### Paso 3 – Extracción de datos desde la BD origen

```sql
-- Consulta de extracción ejemplo
SELECT
    id,
    nombre,
    fecha_registro,
    estado
FROM [nombre_tabla_origen]
WHERE [condición_de_filtro_si_aplica];
```

---

### Paso 4 – Transformación y Carga

```sql
-- Ejemplo de INSERT con transformación
INSERT INTO [nombre_tabla_destino] (id, nombre, fecha_creacion, activo)
SELECT
    id,
    nombre,
    CAST(fecha_registro AS DATETIME),
    CASE WHEN estado = 'A' THEN 1 ELSE 0 END
FROM [nombre_tabla_origen];
```

---

### Paso 5 – Rehabilitar restricciones

```sql
-- MySQL
SET FOREIGN_KEY_CHECKS = 1;

-- PostgreSQL
SET session_replication_role = 'origin';

-- SQL Server
EXEC sp_msforeachtable "ALTER TABLE ? WITH CHECK CHECK CONSTRAINT all";
```

---

## Scripts de Migración

| Script                           | Descripción                             | Estado        |
| -------------------------------- | --------------------------------------- | ------------- |
| `01_backup_destino.sql`          | Backup previo de la BD destino          | `[ ] Pendiente` |
| `02_preparar_entorno.sql`        | Desactivar FK y preparar tablas         | `[ ] Pendiente` |
| `03_migrar_[tabla].sql`          | Migración de tabla específica           | `[ ] Pendiente` |
| `04_validar_registros.sql`       | Consultas de validación y conteos       | `[ ] Pendiente` |
| `05_rollback.sql`                | Script de rollback en caso de error     | `[ ] Pendiente` |

---

## Validación y Pruebas

### Conteo de registros

```sql
-- Conteo en BD origen
SELECT COUNT(*) AS total_origen FROM [nombre_tabla_origen];

-- Conteo en BD destino
SELECT COUNT(*) AS total_destino FROM [nombre_tabla_destino];
```

| Tabla               | Registros Origen | Registros Destino | ¿Coincide? |
| ------------------- | :--------------: | :---------------: | :--------: |
| `[tabla_1]`         |       `?`        |        `?`        |    `[ ]`   |
| `[tabla_2]`         |       `?`        |        `?`        |    `[ ]`   |

### Validación de integridad referencial

```sql
-- Verificar que no existan registros huérfanos
SELECT d.*
FROM [tabla_detalle] d
LEFT JOIN [tabla_maestra] m ON d.id_maestro = m.id
WHERE m.id IS NULL;
```

### Validación de campos críticos

```sql
-- Verificar que no existan valores NULL en campos obligatorios
SELECT COUNT(*)
FROM [nombre_tabla_destino]
WHERE [campo_obligatorio] IS NULL;
```

---

## Gestión de Errores

### Errores comunes y soluciones

| Error                                    | Causa probable                        | Solución                                   |
| ---------------------------------------- | ------------------------------------- | ------------------------------------------ |
| `Duplicate entry for key 'PRIMARY'`      | Registros duplicados en origen        | Deduplicar antes de insertar               |
| `Cannot add or update a child row`       | FK violada, registro maestro faltante | Migrar tabla maestra primero               |
| `Data too long for column`               | Campo origen mayor que destino        | Aumentar tamaño del campo o truncar datos  |
| `Incorrect datetime value`               | Formato de fecha incompatible         | Convertir formato en la transformación     |
| `[otro error]`                           | `[causa]`                             | `[solución]`                               |

> [!IMPORTANT]
> Ante cualquier error que afecte la integridad de los datos, **detener la migración** y ejecutar el script de rollback antes de continuar.

---

## Rollback

En caso de error crítico durante la migración:

### Opción 1 – Restaurar backup

```bash
# MySQL
mysql -u [usuario] -p [nombre_bd] < backup_previo_migracion.sql

# PostgreSQL
psql -U [usuario] -d [nombre_bd] -f backup_previo_migracion.sql

# SQL Server
-- Restaurar desde SSMS o SQLCMD usando el archivo .bak
```

### Opción 2 – Script de rollback incremental

```sql
-- Ejecutar 05_rollback.sql para revertir solo los cambios de esta migración
-- (requiere haber registrado los IDs insertados durante el proceso)
DELETE FROM [nombre_tabla_destino]
WHERE id IN (SELECT id FROM migracion_log WHERE fecha_migracion >= '[fecha_inicio_migracion]');
```

---

## Registro de Cambios

| Fecha        | Versión | Responsable          | Descripción                              |
| ------------ | ------- | -------------------- | ---------------------------------------- |
| `2026-03-12` | `1.0`   | `[nombre]`           | Creación inicial del documento           |
| `[fecha]`    | `[ver]` | `[nombre]`           | `[descripción del cambio]`               |

---

*Documento gestionado con [Obsidian](https://obsidian.md) · Repositorio: `migrate-and-bdd-documentation`*

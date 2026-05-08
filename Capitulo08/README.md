# Validación de datos

## Metadatos

| Atributo         | Detalle                          |
|------------------|----------------------------------|
| **Duración**     | 30 minutos                       |
| **Complejidad**  | Media                            |
| **Nivel Bloom**  | Aplicar (Apply)                  |
| **Laboratorio**  | 08-00-01                         |
| **Versión**      | 1.0                              |

---

## Descripción General

Este laboratorio de cierre consolida el aprendizaje integral del curso mediante una auditoría práctica de calidad de datos sobre el dataset `CURSO_SQL`. El estudiante identificará valores nulos, detectará registros duplicados potenciales y verificará la integridad referencial básica entre tablas, aplicando en todo momento las buenas prácticas de formato y documentación SQL aprendidas en la Lección 8.1. Al finalizar, habrás construido desde cero una consulta de reporte de calidad de datos que integra `SELECT`, `WHERE`, `GROUP BY`, `HAVING`, `JOIN`, funciones de agregación y comentarios explicativos, todo correctamente estructurado y formateado según estándares profesionales.

---

## Objetivos de Aprendizaje

Al completar este laboratorio serás capaz de:

- [ ] Escribir consultas SQL bien estructuradas con indentación consistente, palabras clave en mayúsculas, alias descriptivos y comentarios explicativos (`--` y `/* */`).
- [ ] Identificar y cuantificar valores faltantes (`NULL`) en columnas clave del dataset usando `IS NULL` e `IS NOT NULL`.
- [ ] Detectar registros duplicados potenciales en tablas clave mediante `GROUP BY` y `HAVING COUNT(*) > 1`.
- [ ] Verificar la integridad referencial básica entre tablas relacionadas usando `JOIN` y condiciones de filtrado.
- [ ] Construir una consulta integradora de reporte de calidad de datos que consolide múltiples técnicas del curso en un solo bloque SQL bien documentado.

---

## Prerrequisitos

### Conocimiento Previo

- Haber completado los laboratorios 01-00-01 al 07-00-01, o demostrar dominio de:
  - `SELECT`, `FROM`, `WHERE`, `ORDER BY`
  - Funciones de agregación: `COUNT()`, `SUM()`, `AVG()`
  - `GROUP BY` y `HAVING`
  - `JOIN` (INNER, LEFT)
  - Funciones de texto y fecha básicas

### Acceso Requerido

- Cuenta Snowflake activa (trial o corporativa) con acceso a la interfaz **Snowsight**.
- Base de datos `CURSO_SQL` inicializada correctamente con el script de setup del instructor.
- Permisos de lectura (`SELECT`) sobre todas las tablas del schema `CURSO_SQL`.
- Virtual Warehouse activo (tamaño **X-Small** recomendado para minimizar consumo de créditos).

---

## Entorno del Laboratorio

### Hardware Recomendado

| Componente        | Mínimo Requerido                                      |
|-------------------|-------------------------------------------------------|
| Procesador        | Cualquier CPU moderna (el procesamiento ocurre en Snowflake) |
| RAM               | 4 GB                                                  |
| Conexión          | 10 Mbps estable                                       |
| Pantalla          | Resolución 1280×768 o superior                        |

### Software Requerido

| Software             | Versión / Detalle                                      |
|----------------------|--------------------------------------------------------|
| Navegador web        | Chrome 90+, Firefox 88+, Edge 90+ o Safari 14+         |
| Snowflake Snowsight  | Incluido en toda cuenta Snowflake activa               |
| Dataset del curso    | `CURSO_SQL` v1.0 (script provisto por el instructor)   |

---

## Paso 0 — Crear y poblar el dataset **VENTAS** en `CURSO_SQL`

**Objetivo:** Preparar el entorno de trabajo creando las tablas requeridas para el laboratorio.

### Instrucciones

1. Inicia sesión en tu cuenta Snowflake en [app.snowflake.com](https://app.snowflake.com).
2. En el menú lateral izquierdo, haz clic en **Projects → Workspaces**.
3. Selecciona el Workspace llamado **`Setup_CURSO_SQLSNOW`**.
4.  Da clic en **Add new** y crea un nuevo archivo tipo **SQL**
5. Escribe el siguiente nombre del archivo: **`Setup_Lab08_Pedidos`**
6. A la derecha selecciona el warehouse **`COMPUTE_WH`** o el warehouse asignado/creado al inicio de la creación de la cuenta.
7. Copia/Pega y ejecuta el siguiente script completo.

> **Nota:** En Snowsight puedes ejecutar todo el bloque completo. Si tu cuenta no permite crear bases de datos, ejecuta el script hasta donde tus permisos lo permitan o solicita apoyo del instructor.

```sql
-- ============================================================
-- LAB 08 - Script de inicialización para PEDIDOS y DETALLE_PEDIDOS
-- Base de datos: CURSO_SQL
-- Schema: PUBLIC
-- Requisitos previos:
--   1) CLIENTES debe existir y tener datos
--   2) PRODUCTOS debe existir y tener datos
-- Registros generados:
--   - PEDIDOS: 1,000
--   - DETALLE_PEDIDOS: 3,000
-- Uso recomendado: Ejecutar después del script de CLIENTES y PRODUCTOS
-- ============================================================

USE WAREHOUSE COMPUTE_WH;
USE DATABASE CURSO_SQL;
USE SCHEMA PUBLIC;

-- ============================================================
-- 1. Verificaciones previas
-- ============================================================

SELECT COUNT(*) AS total_clientes
FROM CURSO_SQL.PUBLIC.CLIENTES;

SELECT COUNT(*) AS total_productos
FROM CURSO_SQL.PUBLIC.PRODUCTOS;

-- ============================================================
-- 2. Recrear tablas dependientes
--    Importante: DETALLE_PEDIDOS depende de PEDIDOS, por eso se elimina primero.
-- ============================================================

DROP TABLE IF EXISTS CURSO_SQL.PUBLIC.DETALLE_PEDIDOS;
DROP TABLE IF EXISTS CURSO_SQL.PUBLIC.PEDIDOS;

CREATE OR REPLACE TABLE CURSO_SQL.PUBLIC.PEDIDOS (
    ID_PEDIDO     NUMBER(38,0) NOT NULL,
    ID_CLIENTE    NUMBER(38,0),
    FECHA_PEDIDO  DATE,
    TOTAL_PEDIDO  NUMBER(12,2),
    ESTADO        VARCHAR(50),
    CONSTRAINT PK_PEDIDOS PRIMARY KEY (ID_PEDIDO)
);

CREATE OR REPLACE TABLE CURSO_SQL.PUBLIC.DETALLE_PEDIDOS (
    ID_DETALLE      NUMBER(38,0) NOT NULL,
    ID_PEDIDO       NUMBER(38,0),
    ID_PRODUCTO     NUMBER(38,0),
    CANTIDAD        NUMBER(38,0),
    PRECIO_UNITARIO NUMBER(10,2),
    SUBTOTAL        NUMBER(12,2),
    CONSTRAINT PK_DETALLE_PEDIDOS PRIMARY KEY (ID_DETALLE)
);

-- ============================================================
-- 3. Insertar 1,000 pedidos
--    - ID_CLIENTE se toma desde CLIENTES para mantener integridad referencial.
--    - FECHA_PEDIDO cubre 2023, 2024 y 2025 para que funcionen filtros por año.
--    - ESTADO incluye algunos NULL para auditoría de calidad de datos.
--    - TOTAL_PEDIDO se calcula después con base en DETALLE_PEDIDOS.
-- ============================================================

INSERT INTO CURSO_SQL.PUBLIC.PEDIDOS (
    ID_PEDIDO,
    ID_CLIENTE,
    FECHA_PEDIDO,
    TOTAL_PEDIDO,
    ESTADO
)
WITH clientes_ordenados AS (
    SELECT
        ID_CLIENTE,
        ROW_NUMBER() OVER (ORDER BY ID_CLIENTE) AS RN
    FROM CURSO_SQL.PUBLIC.CLIENTES
), total_clientes AS (
    SELECT COUNT(*) AS TOTAL
    FROM clientes_ordenados
), base AS (
    SELECT
        ROW_NUMBER() OVER (ORDER BY SEQ4()) AS ID_PEDIDO
    FROM TABLE(GENERATOR(ROWCOUNT => 1000))
)
SELECT
    b.ID_PEDIDO,
    c.ID_CLIENTE,
    DATEADD(DAY, MOD(b.ID_PEDIDO * 2, 1095), DATE '2023-01-01') AS FECHA_PEDIDO,
    CAST(0 AS NUMBER(12,2)) AS TOTAL_PEDIDO,
    CASE
        WHEN MOD(b.ID_PEDIDO, 23) = 0 THEN NULL
        WHEN MOD(b.ID_PEDIDO, 5) = 0 THEN 'Pendiente'
        WHEN MOD(b.ID_PEDIDO, 5) = 1 THEN 'Pagado'
        WHEN MOD(b.ID_PEDIDO, 5) = 2 THEN 'Enviado'
        WHEN MOD(b.ID_PEDIDO, 5) = 3 THEN 'Entregado'
        ELSE 'Cancelado'
    END AS ESTADO
FROM base b
CROSS JOIN total_clientes tc
JOIN clientes_ordenados c
    ON c.RN = MOD(b.ID_PEDIDO - 1, tc.TOTAL) + 1;

-- ============================================================
-- 4. Insertar 3,000 líneas de detalle
--    - 3 líneas por pedido.
--    - ID_PRODUCTO se toma desde PRODUCTOS para mantener integridad referencial.
--    - La combinación ID_PEDIDO + ID_PRODUCTO queda sin duplicados intencionales.
-- ============================================================

INSERT INTO CURSO_SQL.PUBLIC.DETALLE_PEDIDOS (
    ID_DETALLE,
    ID_PEDIDO,
    ID_PRODUCTO,
    CANTIDAD,
    PRECIO_UNITARIO,
    SUBTOTAL
)
WITH productos_ordenados AS (
    SELECT
        ID_PRODUCTO,
        PRECIO,
        ROW_NUMBER() OVER (ORDER BY ID_PRODUCTO) AS RN
    FROM CURSO_SQL.PUBLIC.PRODUCTOS
), total_productos AS (
    SELECT COUNT(*) AS TOTAL
    FROM productos_ordenados
), base AS (
    SELECT
        ROW_NUMBER() OVER (ORDER BY SEQ4()) AS ID_DETALLE
    FROM TABLE(GENERATOR(ROWCOUNT => 3000))
), detalle_base AS (
    SELECT
        b.ID_DETALLE,
        CEIL(b.ID_DETALLE / 3.0) AS ID_PEDIDO,
        MOD(b.ID_DETALLE - 1, 3) + 1 AS LINEA_DETALLE,
        MOD(b.ID_DETALLE * 7, 5) + 1 AS CANTIDAD
    FROM base b
)
SELECT
    d.ID_DETALLE,
    d.ID_PEDIDO,
    p.ID_PRODUCTO,
    d.CANTIDAD,
    p.PRECIO AS PRECIO_UNITARIO,
    CAST(d.CANTIDAD * p.PRECIO AS NUMBER(12,2)) AS SUBTOTAL
FROM detalle_base d
CROSS JOIN total_productos tp
JOIN productos_ordenados p
    ON p.RN = MOD(d.ID_PEDIDO + ((d.LINEA_DETALLE - 1) * 53) - 1, tp.TOTAL) + 1;

-- ============================================================
-- 5. Actualizar TOTAL_PEDIDO con la suma real del detalle
-- ============================================================

UPDATE CURSO_SQL.PUBLIC.PEDIDOS p
SET TOTAL_PEDIDO = d.TOTAL_CALCULADO
FROM (
    SELECT
        ID_PEDIDO,
        CAST(SUM(SUBTOTAL) AS NUMBER(12,2)) AS TOTAL_CALCULADO
    FROM CURSO_SQL.PUBLIC.DETALLE_PEDIDOS
    GROUP BY ID_PEDIDO
) d
WHERE p.ID_PEDIDO = d.ID_PEDIDO;

-- ============================================================
-- 6. Validaciones generales de carga
-- ============================================================

SELECT 'CLIENTES' AS TABLA, COUNT(*) AS TOTAL_REGISTROS FROM CURSO_SQL.PUBLIC.CLIENTES
UNION ALL
SELECT 'PRODUCTOS' AS TABLA, COUNT(*) AS TOTAL_REGISTROS FROM CURSO_SQL.PUBLIC.PRODUCTOS
UNION ALL
SELECT 'PEDIDOS' AS TABLA, COUNT(*) AS TOTAL_REGISTROS FROM CURSO_SQL.PUBLIC.PEDIDOS
UNION ALL
SELECT 'DETALLE_PEDIDOS' AS TABLA, COUNT(*) AS TOTAL_REGISTROS FROM CURSO_SQL.PUBLIC.DETALLE_PEDIDOS;

-- ============================================================
-- 7. Validaciones de integridad referencial
--    Resultado esperado: todos los conteos deben ser 0.
-- ============================================================

SELECT COUNT(*) AS pedidos_sin_cliente_valido
FROM CURSO_SQL.PUBLIC.PEDIDOS p
    LEFT JOIN CURSO_SQL.PUBLIC.CLIENTES c
        ON p.ID_CLIENTE = c.ID_CLIENTE
WHERE c.ID_CLIENTE IS NULL;

SELECT COUNT(*) AS detalles_sin_pedido_valido
FROM CURSO_SQL.PUBLIC.DETALLE_PEDIDOS dp
    LEFT JOIN CURSO_SQL.PUBLIC.PEDIDOS p
        ON dp.ID_PEDIDO = p.ID_PEDIDO
WHERE p.ID_PEDIDO IS NULL;

SELECT COUNT(*) AS detalles_sin_producto_valido
FROM CURSO_SQL.PUBLIC.DETALLE_PEDIDOS dp
    LEFT JOIN CURSO_SQL.PUBLIC.PRODUCTOS pr
        ON dp.ID_PRODUCTO = pr.ID_PRODUCTO
WHERE pr.ID_PRODUCTO IS NULL;

-- ============================================================
-- 8. Validaciones útiles para el Lab 08
-- ============================================================

-- Auditoría de NULL en PEDIDOS
SELECT
    COUNT(*) AS total_pedidos,
    SUM(CASE WHEN ID_CLIENTE IS NULL THEN 1 ELSE 0 END) AS nulos_id_cliente,
    SUM(CASE WHEN FECHA_PEDIDO IS NULL THEN 1 ELSE 0 END) AS nulos_fecha_pedido,
    SUM(CASE WHEN TOTAL_PEDIDO IS NULL THEN 1 ELSE 0 END) AS nulos_total_pedido,
    SUM(CASE WHEN ESTADO IS NULL THEN 1 ELSE 0 END) AS nulos_estado
FROM CURSO_SQL.PUBLIC.PEDIDOS;

-- Duplicados por línea de detalle
-- Resultado esperado: 0 filas.
SELECT
    ID_PEDIDO,
    ID_PRODUCTO,
    COUNT(*) AS veces_repetido
FROM CURSO_SQL.PUBLIC.DETALLE_PEDIDOS
GROUP BY
    ID_PEDIDO,
    ID_PRODUCTO
HAVING COUNT(*) > 1
ORDER BY veces_repetido DESC
LIMIT 20;

-- Validación de totales: TOTAL_PEDIDO debe coincidir con la suma del detalle.
-- Resultado esperado: 0 filas.
SELECT
    p.ID_PEDIDO,
    p.TOTAL_PEDIDO,
    d.TOTAL_DETALLE,
    p.TOTAL_PEDIDO - d.TOTAL_DETALLE AS DIFERENCIA
FROM CURSO_SQL.PUBLIC.PEDIDOS p
JOIN (
    SELECT
        ID_PEDIDO,
        CAST(SUM(SUBTOTAL) AS NUMBER(12,2)) AS TOTAL_DETALLE
    FROM CURSO_SQL.PUBLIC.DETALLE_PEDIDOS
    GROUP BY ID_PEDIDO
) d
    ON p.ID_PEDIDO = d.ID_PEDIDO
WHERE p.TOTAL_PEDIDO <> d.TOTAL_DETALLE
ORDER BY p.ID_PEDIDO;

-- Muestra de datos para revisión visual.
SELECT *
FROM CURSO_SQL.PUBLIC.PEDIDOS
ORDER BY ID_PEDIDO
LIMIT 10;

SELECT *
FROM CURSO_SQL.PUBLIC.DETALLE_PEDIDOS
ORDER BY ID_DETALLE
LIMIT 10;
```

8. Ahora selecciona el script llamado **Setup_Lab01** borra el contenido del archivo.
9. Copia y pega el siguiente contenido para actualizar la realción Ventas/Empleados
10. Selecciona todo el contenido y ejecutalo.

```sql
-- ============================================================
-- Script de inicialización del dataset CURSO_SQL
-- Versión: 1.3
-- Uso: Ejecutar una sola vez antes del Lab 01
-- Ajuste: CLIENTES conserva NOMBRE y APELLIDO en columnas distintas
-- Ajuste adicional: algunos EMAIL quedan NULL para prácticas de validación de datos
-- ============================================================

-- 1. Seleccionar warehouse de trabajo
USE WAREHOUSE COMPUTE_WH;

-- 2. Crear base de datos y schema del curso
CREATE OR REPLACE DATABASE CURSO_SQL;
USE DATABASE CURSO_SQL;
CREATE SCHEMA IF NOT EXISTS PUBLIC;
USE SCHEMA PUBLIC;

-- 3. Recrear tablas para garantizar un estado limpio del laboratorio
DROP TABLE IF EXISTS CURSO_SQL.PUBLIC.PRODUCTOS;
DROP TABLE IF EXISTS CURSO_SQL.PUBLIC.CLIENTES;

CREATE OR REPLACE TABLE CURSO_SQL.PUBLIC.PRODUCTOS (
    ID_PRODUCTO      NUMBER(38,0) NOT NULL,
    NOMBRE_PRODUCTO  VARCHAR(200),
    CATEGORIA        VARCHAR(100),
    PRECIO           NUMBER(10,2),
    STOCK            NUMBER(38,0),
    PROVEEDOR        VARCHAR(150),
    FECHA_ALTA       DATE,
    CONSTRAINT PK_PRODUCTOS PRIMARY KEY (ID_PRODUCTO)
);

CREATE OR REPLACE TABLE CURSO_SQL.PUBLIC.CLIENTES (
    ID_CLIENTE       NUMBER(38,0) NOT NULL,
    NOMBRE           VARCHAR(100),
    APELLIDO         VARCHAR(100),
    EMAIL            VARCHAR(200),
    PAIS             VARCHAR(80),
    CIUDAD           VARCHAR(100),
    SEGMENTO         VARCHAR(50),
    FECHA_REGISTRO   DATE,
    TELEFONO         VARCHAR(30),
    CONSTRAINT PK_CLIENTES PRIMARY KEY (ID_CLIENTE)
);

-- 4. Insertar 150 productos de ejemplo
INSERT INTO CURSO_SQL.PUBLIC.PRODUCTOS (
    ID_PRODUCTO,
    NOMBRE_PRODUCTO,
    CATEGORIA,
    PRECIO,
    STOCK,
    PROVEEDOR,
    FECHA_ALTA
)
WITH base AS (
    SELECT SEQ4() + 1 AS ID_PRODUCTO
    FROM TABLE(GENERATOR(ROWCOUNT => 150))
), productos AS (
    SELECT
        ID_PRODUCTO,
        CASE MOD(ID_PRODUCTO, 12)
            WHEN 0 THEN 'Laptop Empresarial'
            WHEN 1 THEN 'Monitor Profesional'
            WHEN 2 THEN 'Teclado Mecánico'
            WHEN 3 THEN 'Mouse Inalámbrico'
            WHEN 4 THEN 'Silla Ergonómica'
            WHEN 5 THEN 'Escritorio Ajustable'
            WHEN 6 THEN 'Cuaderno Ejecutivo'
            WHEN 7 THEN 'Impresora Láser'
            WHEN 8 THEN 'Taladro Compacto'
            WHEN 9 THEN 'Router WiFi'
            WHEN 10 THEN 'Auriculares USB'
            ELSE 'Cámara Web HD'
        END AS TIPO_PRODUCTO,
        CASE MOD(ID_PRODUCTO, 6)
            WHEN 0 THEN 'Electrónica'
            WHEN 1 THEN 'Mobiliario'
            WHEN 2 THEN 'Papelería'
            WHEN 3 THEN 'Herramientas'
            WHEN 4 THEN 'Redes'
            ELSE 'Accesorios'
        END AS CATEGORIA,
        CASE MOD(ID_PRODUCTO, 6)
            WHEN 0 THEN 'TechSupply S.A.'
            WHEN 1 THEN 'OfficeWorld'
            WHEN 2 THEN 'Papelería Central'
            WHEN 3 THEN 'ToolPro México'
            WHEN 4 THEN 'NetworkPlus'
            ELSE 'DigitalGear'
        END AS PROVEEDOR
    FROM base
)
SELECT
    ID_PRODUCTO,
    TIPO_PRODUCTO || ' ' || LPAD(ID_PRODUCTO, 3, '0') AS NOMBRE_PRODUCTO,
    CATEGORIA,
    CAST(ROUND(25 + MOD(ID_PRODUCTO * 37, 5000) / 10, 2) AS NUMBER(10,2)) AS PRECIO,
    MOD(ID_PRODUCTO * 17, 250) + 5 AS STOCK,
    PROVEEDOR,
    DATEADD(DAY, -MOD(ID_PRODUCTO * 9, 730), CURRENT_DATE()) AS FECHA_ALTA
FROM productos;

-- 5. Insertar 300 clientes de ejemplo
INSERT INTO CURSO_SQL.PUBLIC.CLIENTES (
    ID_CLIENTE,
    NOMBRE,
    APELLIDO,
    EMAIL,
    PAIS,
    CIUDAD,
    SEGMENTO,
    FECHA_REGISTRO,
    TELEFONO
)
WITH base AS (
    SELECT SEQ4() + 1 AS ID_CLIENTE
    FROM TABLE(GENERATOR(ROWCOUNT => 300))
), clientes AS (
    SELECT
        ID_CLIENTE,
        CASE MOD(ID_CLIENTE, 12)
            WHEN 0 THEN 'Ana'
            WHEN 1 THEN 'Carlos'
            WHEN 2 THEN 'Lucía'
            WHEN 3 THEN 'Jorge'
            WHEN 4 THEN 'María'
            WHEN 5 THEN 'Andrés'
            WHEN 6 THEN 'Sofía'
            WHEN 7 THEN 'Daniel'
            WHEN 8 THEN 'Valeria'
            WHEN 9 THEN 'Miguel'
            WHEN 10 THEN 'Camila'
            ELSE 'Ricardo'
        END AS NOMBRE_BASE,
        CASE MOD(ID_CLIENTE, 12)
            WHEN 0 THEN 'Martínez'
            WHEN 1 THEN 'Gómez'
            WHEN 2 THEN 'Fernández'
            WHEN 3 THEN 'Ramírez'
            WHEN 4 THEN 'Torres'
            WHEN 5 THEN 'López'
            WHEN 6 THEN 'Castillo'
            WHEN 7 THEN 'Hernández'
            WHEN 8 THEN 'Morales'
            WHEN 9 THEN 'Sánchez'
            WHEN 10 THEN 'Vargas'
            ELSE 'Flores'
        END AS APELLIDO_BASE,
        CASE MOD(ID_CLIENTE, 8)
            WHEN 0 THEN 'México'
            WHEN 1 THEN 'Colombia'
            WHEN 2 THEN 'Argentina'
            WHEN 3 THEN 'España'
            WHEN 4 THEN 'Chile'
            WHEN 5 THEN 'Perú'
            WHEN 6 THEN 'Ecuador'
            ELSE 'Uruguay'
        END AS PAIS,
        CASE MOD(ID_CLIENTE, 8)
            WHEN 0 THEN 'Ciudad de México'
            WHEN 1 THEN 'Bogotá'
            WHEN 2 THEN 'Buenos Aires'
            WHEN 3 THEN 'Madrid'
            WHEN 4 THEN 'Santiago'
            WHEN 5 THEN 'Lima'
            WHEN 6 THEN 'Quito'
            ELSE 'Montevideo'
        END AS CIUDAD,
        CASE MOD(ID_CLIENTE, 4)
            WHEN 0 THEN 'Corporativo'
            WHEN 1 THEN 'Retail'
            WHEN 2 THEN 'Gobierno'
            ELSE 'PyME'
        END AS SEGMENTO
    FROM base
)
SELECT
    ID_CLIENTE,
    NOMBRE_BASE AS NOMBRE,
    APELLIDO_BASE || ' ' || LPAD(ID_CLIENTE, 3, '0') AS APELLIDO,
    CASE
        WHEN MOD(ID_CLIENTE, 13) = 0 THEN NULL
        ELSE
            LOWER(
                TRANSLATE(NOMBRE_BASE, 'ÁÉÍÓÚáéíóúÑñ', 'AEIOUaeiouNn')
            )
            || '.' ||
            LOWER(
                TRANSLATE(APELLIDO_BASE, 'ÁÉÍÓÚáéíóúÑñ', 'AEIOUaeiouNn')
            )
            || LPAD(ID_CLIENTE, 3, '0') ||
            CASE MOD(ID_CLIENTE, 5)
                WHEN 0 THEN '@gmail.com'
                WHEN 1 THEN '@outlook.com'
                WHEN 2 THEN '@empresa.com'
                WHEN 3 THEN '@example.com'
                ELSE '@correo.com'
            END
    END AS EMAIL,
    PAIS,
    CIUDAD,
    SEGMENTO,
    DATEADD(DAY, -MOD(ID_CLIENTE * 5, 1095), CURRENT_DATE()) AS FECHA_REGISTRO,
    CASE
        WHEN MOD(ID_CLIENTE, 7) = 0 THEN NULL
        ELSE '+52' || LPAD(MOD(ID_CLIENTE * 31, 9000000000) + 1000000000, 10, '0')
    END AS TELEFONO
FROM clientes;

-- 6. Validar que el dataset quedó creado correctamente
SELECT 'PRODUCTOS' AS TABLA, COUNT(*) AS TOTAL_REGISTROS FROM CURSO_SQL.PUBLIC.PRODUCTOS
UNION ALL
SELECT 'CLIENTES' AS TABLA, COUNT(*) AS TOTAL_REGISTROS FROM CURSO_SQL.PUBLIC.CLIENTES;

-- 7. Validar estructura y muestra de CLIENTES
DESCRIBE TABLE CURSO_SQL.PUBLIC.CLIENTES;

SELECT
    ID_CLIENTE,
    NOMBRE,
    APELLIDO,
    EMAIL,
    PAIS,
    CIUDAD,
    SEGMENTO,
    FECHA_REGISTRO,
    TELEFONO
FROM CURSO_SQL.PUBLIC.CLIENTES
ORDER BY ID_CLIENTE
LIMIT 10;

-- 8. Validaciones útiles para laboratorios de filtrado
SELECT SEGMENTO, COUNT(*) AS TOTAL_CLIENTES
FROM CURSO_SQL.PUBLIC.CLIENTES
GROUP BY SEGMENTO
ORDER BY SEGMENTO;

SELECT PAIS, COUNT(*) AS TOTAL_CLIENTES
FROM CURSO_SQL.PUBLIC.CLIENTES
GROUP BY PAIS
ORDER BY PAIS;

SELECT
    COUNT(*) AS TOTAL_CLIENTES,
    COUNT(CASE WHEN EMAIL IS NULL THEN 1 END) AS CLIENTES_SIN_EMAIL,
    COUNT(CASE WHEN EMAIL IS NOT NULL THEN 1 END) AS CLIENTES_CON_EMAIL,
    COUNT(CASE WHEN TELEFONO IS NULL THEN 1 END) AS CLIENTES_SIN_TELEFONO,
    COUNT(CASE WHEN TELEFONO IS NOT NULL THEN 1 END) AS CLIENTES_CON_TELEFONO
FROM CURSO_SQL.PUBLIC.CLIENTES;
```

#### Resultado Esperado

Al finalizar la ejecución del script, la tabla **`PEDIDOS`** queda disponible con datos suficientes para completar todos los pasos de la práctica y las tablas **CLIENTES/PRODUCTOS** quedaron ajustadas para las consultas.

---

### Configuración Inicial del Entorno

Antes de comenzar los ejercicios, debes verificar que tu entorno Snowflake está correctamente configurado. Sigue estos pasos de setup una sola vez al inicio de la sesión.

**Paso de configuración — Abrir un Worksheet y seleccionar el contexto correcto:**

1. Selecciona tu **Workspace** llamado **`SnowEssLabs`**
2. Agrega un nuevo archivo tipo **SQL** y escribe el siguiente nombre: **`Lab08_Validacion_Datos`**
3. En la barra superior derecha del workspace, selecciona el contexto de ejecución:
   - **Warehouse:** `COMPUTE_WH` (tamaño X-Small)
   - **Database:** `CURSO_SQL`
   - **Schema:** `PUBLIC`

4. Ejecuta el siguiente bloque de configuración para confirmar que el contexto está activo:

```sql
-- Configuración del contexto de trabajo
-- Ajusta los nombres según tu entorno específico

USE WAREHOUSE COMPUTE_WH;     -- Warehouse X-Small recomendado ajustalo si tu nombre es diferente.
USE DATABASE CURSO_SQL;
USE SCHEMA PUBLIC;
```

**Verificación del contexto:** Confirma que en la barra superior de Snowsight aparezcan correctamente la base de datos `CURSO_SQL`, el schema y el warehouse seleccionados antes de continuar.

---

## Instrucciones Paso a Paso

---

### Paso 1 — Exploración Inicial: Reconocer la Estructura del Dataset

**Objetivo:** Confirmar que todas las tablas del dataset `CURSO_SQL` están disponibles y revisar su estructura antes de iniciar la auditoría de calidad.

#### Instrucciones

1. Ejecuta la siguiente consulta para listar las tablas disponibles en el schema:

```sql
-- ============================================================
-- Lab 08-00-01: Validación de Datos
-- Paso 1: Exploración inicial del dataset CURSO_SQL
-- ============================================================

SHOW TABLES IN SCHEMA CURSO_SQL.PUBLIC;
```

2. A continuación, verifica el número de filas en cada tabla principal ejecutando las siguientes consultas una por una:

```sql
-- Conteo de registros por tabla principal
SELECT 'CLIENTES'   AS tabla, COUNT(*) AS total_registros FROM CLIENTES
UNION ALL
SELECT 'PRODUCTOS'  AS tabla, COUNT(*) AS total_registros FROM PRODUCTOS
UNION ALL
SELECT 'PEDIDOS'    AS tabla, COUNT(*) AS total_registros FROM PEDIDOS
UNION ALL
SELECT 'DETALLE_PEDIDOS' AS tabla, COUNT(*) AS total_registros FROM DETALLE_PEDIDOS;
```

3. Revisa la estructura de la tabla `CLIENTES` para identificar sus columnas:

```sql
-- Revisión de estructura de la tabla CLIENTES
DESCRIBE TABLE CLIENTES;
```

4. Repite el paso anterior para `PEDIDOS`:

```sql
-- Revisión de estructura de la tabla PEDIDOS
DESCRIBE TABLE PEDIDOS;
```

#### Resultado Esperado

Deberías ver una lista de tablas con sus respectivos conteos de filas. La tabla de conteos mostrará algo similar a:

| TABLA           | TOTAL_REGISTROS |
|-----------------|-----------------|
| CLIENTES        | ~500            |
| PRODUCTOS       | ~100            |
| PEDIDOS         | ~1000           |
| DETALLE_PEDIDOS | ~3000           |

> Los valores exactos dependen del script de inicialización del instructor. Lo importante es confirmar que todas las tablas tienen datos.

#### Verificación

✅ Confirma que `SHOW TABLES` lista al menos 4 tablas.
✅ Confirma que ninguna tabla devuelve `0` registros en el conteo.
✅ Confirma que `DESCRIBE TABLE` muestra las columnas con sus tipos de datos.

---

### Paso 2 — Auditoría de Valores Nulos: Identificación de Datos Faltantes

**Objetivo:** Aplicar `IS NULL` e `IS NOT NULL` de forma sistemática para identificar y cuantificar valores faltantes en columnas clave del dataset.

#### Instrucciones

1. Comienza con una auditoría de nulos en la tabla `CLIENTES`. Presta atención al formato de la consulta: palabras clave en mayúsculas, una columna por línea, alias descriptivos y comentarios:

```sql
-- ============================================================
-- Paso 2A: Auditoría de valores NULL en tabla CLIENTES
-- Objetivo: Identificar columnas con datos faltantes
-- ============================================================

SELECT
    -- Total de registros en la tabla
    COUNT(*)                                    AS total_registros,

    -- Conteo de NULLs por columna clave
    SUM(CASE WHEN nombre     IS NULL THEN 1 ELSE 0 END) AS nulos_nombre,
    SUM(CASE WHEN email      IS NULL THEN 1 ELSE 0 END) AS nulos_email,
    SUM(CASE WHEN telefono   IS NULL THEN 1 ELSE 0 END) AS nulos_telefono,
    SUM(CASE WHEN ciudad     IS NULL THEN 1 ELSE 0 END) AS nulos_ciudad,
    SUM(CASE WHEN pais       IS NULL THEN 1 ELSE 0 END) AS nulos_pais

FROM CLIENTES;
```

> **Nota de formato:** Observa cómo los alias `AS` están alineados verticalmente. Esta práctica, aunque opcional, mejora significativamente la legibilidad cuando hay múltiples columnas.

2. Ahora calcula el **porcentaje** de valores nulos para entender la proporción del problema:

```sql
-- ============================================================
-- Paso 2B: Porcentaje de valores NULL en tabla CLIENTES
-- Fórmula: (nulos / total) * 100
-- ============================================================

SELECT
    COUNT(*)                                                        AS total_registros,

    -- Porcentaje de nulos por columna (redondeado a 2 decimales)
    ROUND(SUM(CASE WHEN nombre   IS NULL THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS pct_nulos_nombre,
    ROUND(SUM(CASE WHEN email    IS NULL THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS pct_nulos_email,
    ROUND(SUM(CASE WHEN telefono IS NULL THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS pct_nulos_telefono,
    ROUND(SUM(CASE WHEN ciudad   IS NULL THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS pct_nulos_ciudad

FROM CLIENTES;
```

3. Ahora identifica los registros específicos que tienen `email` nulo para poder revisarlos:

```sql
-- ============================================================
-- Paso 2C: Listado de clientes sin email registrado
-- Útil para acciones correctivas de datos
-- ============================================================

SELECT
    id_cliente,
    nombre,
    ciudad,
    telefono,
    email           -- Será NULL en todos los resultados
FROM CLIENTES
WHERE email IS NULL
ORDER BY ciudad, nombre;
```

4. Realiza la misma auditoría de nulos para la tabla `PEDIDOS`, enfocándote en columnas críticas para el negocio:

```sql
-- ============================================================
-- Paso 2D: Auditoría de valores NULL en tabla PEDIDOS
-- ============================================================

SELECT
    COUNT(*)                                                              AS total_pedidos,
    SUM(CASE WHEN id_cliente      IS NULL THEN 1 ELSE 0 END)             AS nulos_id_cliente,
    SUM(CASE WHEN fecha_pedido    IS NULL THEN 1 ELSE 0 END)             AS nulos_fecha_pedido,
    SUM(CASE WHEN total_pedido    IS NULL THEN 1 ELSE 0 END)             AS nulos_total_pedido,
    SUM(CASE WHEN estado          IS NULL THEN 1 ELSE 0 END)             AS nulos_estado,

    -- Porcentaje de pedidos sin cliente asignado (problema crítico de integridad)
    ROUND(
        SUM(CASE WHEN id_cliente IS NULL THEN 1 ELSE 0 END) * 100.0 / COUNT(*),
        2
    )                                                                     AS pct_sin_cliente

FROM PEDIDOS;
```

#### Resultado Esperado

La consulta del Paso 2A devolverá una fila con los conteos de nulos por columna. Ejemplo ilustrativo:

| TOTAL_REGISTROS | NULOS_NOMBRE | NULOS_EMAIL | NULOS_TELEFONO | NULOS_CIUDAD | NULOS_PAIS |
|-----------------|--------------|-------------|----------------|--------------|------------|
| 500             | 0            | 47          | 112            | 5            | 0          |

La consulta del Paso 2C listará únicamente los clientes cuyo campo `email` es `NULL`.

#### Verificación

✅ La suma de `nulos_X + no_nulos_X` debe ser igual a `total_registros` para cada columna.
✅ El porcentaje calculado en 2B debe ser coherente con los conteos de 2A.
✅ La consulta 2C solo debe devolver registros donde `email` efectivamente sea `NULL`.

---

### Paso 3 — Detección de Duplicados: Registros Repetidos en Tablas Clave

**Objetivo:** Usar `GROUP BY` y `HAVING COUNT(*) > 1` para identificar posibles registros duplicados que comprometan la integridad del dataset.

#### Instrucciones

1. Detecta si existen clientes con el mismo `email` registrado más de una vez (duplicados por email):

```sql
-- ============================================================
-- Paso 3A: Detección de emails duplicados en CLIENTES
-- Un email debería identificar de forma única a un cliente
-- ============================================================

SELECT
    email,
    COUNT(*)    AS cantidad_registros
FROM CLIENTES
WHERE email IS NOT NULL          -- Excluimos NULLs: no son duplicados entre sí
GROUP BY email
HAVING COUNT(*) > 1              -- Solo mostramos los que aparecen más de una vez
ORDER BY cantidad_registros DESC;
```

2. Verifica si existen combinaciones duplicadas de `nombre` + `ciudad` en la tabla `CLIENTES`:

```sql
-- ============================================================
-- Paso 3B: Detección de posibles clientes duplicados
-- por combinación nombre + ciudad
-- ============================================================

SELECT
    nombre,
    ciudad,
    COUNT(*)        AS ocurrencias
FROM CLIENTES
GROUP BY
    nombre,
    ciudad
HAVING COUNT(*) > 1
ORDER BY ocurrencias DESC, nombre;
```

3. Detecta si existen productos con el mismo `nombre` registrados más de una vez:

```sql
-- ============================================================
-- Paso 3C: Detección de nombres de productos duplicados
-- ============================================================

SELECT
    nombre_producto,
    COUNT(*)            AS cantidad_registros,
    COUNT(DISTINCT id_producto) AS ids_distintos
FROM PRODUCTOS
GROUP BY nombre_producto
HAVING COUNT(*) > 1
ORDER BY cantidad_registros DESC;
```

4. Verifica si existen líneas de detalle duplicadas en `DETALLE_PEDIDOS` (mismo pedido con el mismo producto más de una vez):

```sql
-- ============================================================
-- Paso 3D: Detección de líneas duplicadas en DETALLE_PEDIDOS
-- Una combinación id_pedido + id_producto debería ser única
-- ============================================================

SELECT
    id_pedido,
    id_producto,
    COUNT(*)    AS veces_repetido
FROM DETALLE_PEDIDOS
GROUP BY
    id_pedido,
    id_producto
HAVING COUNT(*) > 1
ORDER BY veces_repetido DESC
LIMIT 20;                        -- Limitamos a los 20 casos más graves
```

5. Genera un **resumen de duplicados** para tener una vista consolidada:

```sql
-- ============================================================
-- Paso 3E: Resumen ejecutivo de duplicados por tabla
-- ============================================================

SELECT
    'CLIENTES - Email duplicado'            AS tipo_duplicado,
    COUNT(*)                                AS grupos_duplicados,
    SUM(cantidad - 1)                       AS registros_extra
FROM (
    SELECT email, COUNT(*) AS cantidad
    FROM CLIENTES
    WHERE email IS NOT NULL
    GROUP BY email
    HAVING COUNT(*) > 1
) sub

UNION ALL

SELECT
    'DETALLE_PEDIDOS - Línea duplicada'     AS tipo_duplicado,
    COUNT(*)                                AS grupos_duplicados,
    SUM(cantidad - 1)                       AS registros_extra
FROM (
    SELECT id_pedido, id_producto, COUNT(*) AS cantidad
    FROM DETALLE_PEDIDOS
    GROUP BY id_pedido, id_producto
    HAVING COUNT(*) > 1
) sub;
```

#### Resultado Esperado

- Si el dataset está limpio, las consultas 3A, 3B y 3D devolverán **0 filas** (sin duplicados detectados).
- Si existen duplicados, verás filas con `cantidad_registros > 1` o `veces_repetido > 1`.
- El resumen del Paso 3E mostrará cuántos grupos duplicados existen y cuántos registros "extra" hay en cada categoría.

#### Verificación

✅ Las consultas con `HAVING COUNT(*) > 1` solo deben devolver filas cuando genuinamente existen duplicados.
✅ Si una consulta devuelve 0 filas, el mensaje en Snowsight será "Query produced no results" — esto es el resultado correcto si no hay duplicados.
✅ Confirma que el filtro `WHERE email IS NOT NULL` en el Paso 3A está presente (sin él, los NULLs podrían agruparse incorrectamente).

---

### Paso 4 — Verificación de Integridad Referencial Básica

**Objetivo:** Usar `LEFT JOIN` para detectar registros "huérfanos" — filas que referencian un ID que no existe en la tabla padre correspondiente.

#### Instrucciones

1. Verifica si existen pedidos que referencian clientes que no existen en la tabla `CLIENTES`:

```sql
-- ============================================================
-- Paso 4A: Pedidos sin cliente válido (registros huérfanos)
-- Técnica: LEFT JOIN + WHERE tabla_padre IS NULL
-- ============================================================

SELECT
    p.id_pedido,
    p.id_cliente        AS id_cliente_en_pedido,
    p.fecha_pedido,
    p.total_pedido,
    c.id_cliente        AS id_cliente_en_tabla   -- Será NULL si no existe
FROM PEDIDOS p
    LEFT JOIN CLIENTES c
        ON p.id_cliente = c.id_cliente
WHERE c.id_cliente IS NULL                       -- Solo filas sin coincidencia
ORDER BY p.id_pedido;
```

2. Verifica si existen líneas de detalle que referencian pedidos inexistentes:

```sql
-- ============================================================
-- Paso 4B: Líneas de detalle sin pedido válido
-- ============================================================

SELECT
    dp.id_detalle,
    dp.id_pedido        AS id_pedido_en_detalle,
    dp.id_producto,
    dp.cantidad,
    p.id_pedido         AS id_pedido_en_tabla    -- Será NULL si no existe
FROM DETALLE_PEDIDOS dp
    LEFT JOIN PEDIDOS p
        ON dp.id_pedido = p.id_pedido
WHERE p.id_pedido IS NULL
ORDER BY dp.id_pedido;
```

3. Verifica si existen líneas de detalle que referencian productos inexistentes:

```sql
-- ============================================================
-- Paso 4C: Líneas de detalle sin producto válido
-- ============================================================

SELECT
    dp.id_detalle,
    dp.id_pedido,
    dp.id_producto      AS id_producto_en_detalle,
    pr.id_producto      AS id_producto_en_tabla  -- Será NULL si no existe
FROM DETALLE_PEDIDOS dp
    LEFT JOIN PRODUCTOS pr
        ON dp.id_producto = pr.id_producto
WHERE pr.id_producto IS NULL
ORDER BY dp.id_producto;
```

4. Genera un **resumen de integridad referencial** consolidado:

```sql
-- ============================================================
-- Paso 4D: Resumen de integridad referencial
-- Vista ejecutiva del estado de las relaciones entre tablas
-- ============================================================

SELECT
    'Pedidos sin cliente válido'            AS problema_detectado,
    COUNT(*)                                AS cantidad_registros_afectados
FROM PEDIDOS p
    LEFT JOIN CLIENTES c
        ON p.id_cliente = c.id_cliente
WHERE c.id_cliente IS NULL

UNION ALL

SELECT
    'Detalles sin pedido válido'            AS problema_detectado,
    COUNT(*)                                AS cantidad_registros_afectados
FROM DETALLE_PEDIDOS dp
    LEFT JOIN PEDIDOS p
        ON dp.id_pedido = p.id_pedido
WHERE p.id_pedido IS NULL

UNION ALL

SELECT
    'Detalles sin producto válido'          AS problema_detectado,
    COUNT(*)                                AS cantidad_registros_afectados
FROM DETALLE_PEDIDOS dp
    LEFT JOIN PRODUCTOS pr
        ON dp.id_producto = pr.id_producto
WHERE pr.id_producto IS NULL;
```

#### Resultado Esperado

En un dataset bien construido, las tres consultas devolverán **0 filas** (sin registros huérfanos). El resumen del Paso 4D mostrará:

| PROBLEMA_DETECTADO              | CANTIDAD_REGISTROS_AFECTADOS |
|---------------------------------|------------------------------|
| Pedidos sin cliente válido      | 0                            |
| Detalles sin pedido válido      | 0                            |
| Detalles sin producto válido    | 0                            |

Si algún valor es mayor que 0, indica un problema real de integridad referencial en el dataset.

#### Verificación

✅ Comprende la lógica del patrón `LEFT JOIN ... WHERE tabla_padre IS NULL`: este es el método estándar para detectar registros huérfanos.
✅ Distingue entre los `IS NULL` del `WHERE` (que detectan huérfanos) y los `IS NULL` del `SELECT` (que son solo etiquetas visuales).
✅ El resumen del Paso 4D debe devolver exactamente 3 filas (una por cada relación verificada).

---

### Paso 5 — Refactorización: Aplicando Buenas Prácticas de Formato

**Objetivo:** Tomar una consulta mal formateada y refactorizarla aplicando todos los estándares de escritura SQL aprendidos en la Lección 8.1.

#### Instrucciones

1. Observa la siguiente consulta **sin formato**. Analiza por qué es difícil de leer antes de refactorizarla:

```sql
-- CONSULTA ORIGINAL (mal formateada - NO ejecutar como referencia de estilo)
select c.ciudad, count(p.id_pedido) as pedidos, sum(p.total_pedido) as ventas, avg(p.total_pedido) as promedio from clientes c join pedidos p on c.id_cliente=p.id_cliente where p.fecha_pedido>='2024-01-01' group by c.ciudad having count(p.id_pedido)>5 order by ventas desc
```

2. Ahora escribe la versión refactorizada con todas las buenas prácticas aplicadas. Escríbela tú mismo en el worksheet **antes** de ver la solución:

```sql
/* ============================================================
   CONSULTA REFACTORIZADA: Resumen de ventas por ciudad
   Propósito : Analizar el desempeño comercial por ciudad
               para el año 2024
   Filtros   : Solo ciudades con más de 5 pedidos
   Autor     : [Tu nombre]
   Fecha     : [Fecha actual]
   ============================================================ */

SELECT
    c.ciudad                        AS ciudad,
    COUNT(p.id_pedido)              AS cantidad_pedidos,
    SUM(p.total_pedido)             AS total_ventas,
    ROUND(AVG(p.total_pedido), 2)   AS promedio_por_pedido

FROM CLIENTES c
    JOIN PEDIDOS p
        ON c.id_cliente = p.id_cliente

WHERE p.fecha_pedido >= '2024-01-01'

GROUP BY
    c.ciudad

HAVING COUNT(p.id_pedido) > 5

ORDER BY total_ventas DESC;
```

3. Compara ambas versiones y documenta en un comentario SQL (dentro del worksheet) al menos **3 diferencias** que mejoran la legibilidad. Ejemplo de cómo documentar:

```sql
/*
   ANÁLISIS DE DIFERENCIAS - Refactorización Paso 5
   
   Diferencia 1: Palabras clave en MAYÚSCULAS (SELECT, FROM, JOIN, WHERE...)
                 → Distingue visualmente las instrucciones SQL de los datos
   
   Diferencia 2: Una cláusula por línea con indentación de 4 espacios
                 → Permite leer la estructura de la consulta de un vistazo
   
   Diferencia 3: Alias descriptivos (cantidad_pedidos, total_ventas)
                 → El resultado es autoexplicativo sin necesidad de documentación adicional
   
   Diferencia 4: Comentario de bloque con propósito, filtros, autor y fecha
                 → Cualquier analista puede entender el contexto sin ejecutar la consulta
   
   Diferencia 5: ROUND() en el promedio
                 → Mejora la presentación del resultado con 2 decimales
*/
```

#### Resultado Esperado

Ambas versiones (original y refactorizada) deben devolver **exactamente el mismo resultado**. La diferencia es puramente de legibilidad y mantenibilidad.

| CIUDAD    | CANTIDAD_PEDIDOS | TOTAL_VENTAS | PROMEDIO_POR_PEDIDO |
|-----------|------------------|--------------|---------------------|
| Bogotá    | 127              | 458,320.00   | 3,609.61            |
| Medellín  | 89               | 312,150.50   | 3,507.31            |
| ...       | ...              | ...          | ...                 |

#### Verificación

✅ Ejecuta **ambas versiones** y confirma que producen resultados idénticos.
✅ La versión refactorizada debe tener: palabras clave en mayúsculas, una cláusula por línea, alias descriptivos y al menos un comentario.
✅ Documenta en el worksheet al menos 3 diferencias observadas.

---

### Paso 6 — Ejercicio Integrador: Reporte Completo de Calidad de Datos

**Objetivo:** Construir desde cero una consulta integradora que consolide todas las técnicas del laboratorio en un reporte ejecutivo de calidad de datos, siguiendo estándares profesionales de formato y documentación.

#### Instrucciones

1. Lee el requerimiento completo antes de escribir código:

> **Requerimiento:** El equipo de datos necesita un reporte que, para cada ciudad donde operamos, muestre: el total de clientes registrados, cuántos de esos clientes tienen email registrado, el porcentaje de clientes con email, el total de pedidos generados desde esa ciudad, y el valor total de ventas. Solo incluir ciudades con al menos 3 clientes. Ordenar por total de ventas descendente.

2. Construye la consulta **paso a paso**. Primero escribe el esqueleto con comentarios:

```sql
/* ============================================================
   REPORTE DE CALIDAD DE DATOS - RESUMEN POR CIUDAD
   
   Propósito : Auditoría de completitud de datos y actividad
               comercial por ciudad
   Tablas    : CLIENTES (fuente principal), PEDIDOS (ventas)
   Filtro    : Ciudades con al menos 3 clientes registrados
   Orden     : Por valor total de ventas (descendente)
   
   Métricas de calidad incluidas:
     - Completitud de email (% de clientes con email)
     - Actividad comercial (pedidos y ventas por ciudad)
   ============================================================ */

SELECT
    -- Dimensión de análisis
    c.ciudad                                                        AS ciudad,

    -- Métricas de calidad de datos
    COUNT(DISTINCT c.id_cliente)                                    AS total_clientes,

    COUNT(DISTINCT CASE WHEN c.email IS NOT NULL
                        THEN c.id_cliente END)                      AS clientes_con_email,

    ROUND(
        COUNT(DISTINCT CASE WHEN c.email IS NOT NULL
                            THEN c.id_cliente END) * 100.0
        / COUNT(DISTINCT c.id_cliente),
        1
    )                                                               AS pct_completitud_email,

    -- Métricas de actividad comercial
    COUNT(DISTINCT p.id_pedido)                                     AS total_pedidos,

    ROUND(
        COALESCE(SUM(p.total_pedido), 0),
        2
    )                                                               AS total_ventas

FROM CLIENTES c
    LEFT JOIN PEDIDOS p                  -- LEFT JOIN para incluir clientes sin pedidos
        ON c.id_cliente = p.id_cliente

WHERE c.ciudad IS NOT NULL               -- Excluimos clientes sin ciudad registrada

GROUP BY
    c.ciudad

HAVING COUNT(DISTINCT c.id_cliente) >= 3 -- Solo ciudades con al menos 3 clientes

ORDER BY total_ventas DESC;
```

3. Ejecuta la consulta y analiza los resultados. Luego responde las siguientes preguntas en comentarios dentro del worksheet:

```sql
/*
   ANÁLISIS DE RESULTADOS - Reporte de Calidad por Ciudad
   
   Pregunta 1: ¿Cuál es la ciudad con mayor porcentaje de completitud de email?
   Respuesta : [Escribe tu respuesta aquí]
   
   Pregunta 2: ¿Existe alguna ciudad con 0 pedidos? ¿Qué indica esto?
   Respuesta : [Escribe tu respuesta aquí]
   
   Pregunta 3: ¿Por qué usamos LEFT JOIN en lugar de INNER JOIN en este reporte?
   Respuesta : [Escribe tu respuesta aquí]
   
   Pregunta 4: ¿Por qué el filtro de ciudades va en HAVING y no en WHERE?
   Respuesta : [Escribe tu respuesta aquí]
*/
```

4. Como extensión opcional, agrega una columna adicional que clasifique la calidad del email como `'Alta'`, `'Media'` o `'Baja'`:

```sql
/* ============================================================
   EXTENSIÓN OPCIONAL: Clasificación de calidad por ciudad
   Agrega una columna de categoría de completitud
   ============================================================ */

SELECT
    c.ciudad                                                        AS ciudad,
    COUNT(DISTINCT c.id_cliente)                                    AS total_clientes,

    ROUND(
        COUNT(DISTINCT CASE WHEN c.email IS NOT NULL
                            THEN c.id_cliente END) * 100.0
        / COUNT(DISTINCT c.id_cliente),
        1
    )                                                               AS pct_completitud_email,

    -- Clasificación de calidad basada en porcentaje de completitud
    CASE
        WHEN COUNT(DISTINCT CASE WHEN c.email IS NOT NULL
                                 THEN c.id_cliente END) * 100.0
             / COUNT(DISTINCT c.id_cliente) >= 80 THEN 'Alta'
        WHEN COUNT(DISTINCT CASE WHEN c.email IS NOT NULL
                                 THEN c.id_cliente END) * 100.0
             / COUNT(DISTINCT c.id_cliente) >= 50 THEN 'Media'
        ELSE 'Baja'
    END                                                             AS calidad_email,

    COUNT(DISTINCT p.id_pedido)                                     AS total_pedidos,
    ROUND(COALESCE(SUM(p.total_pedido), 0), 2)                      AS total_ventas

FROM CLIENTES c
    LEFT JOIN PEDIDOS p
        ON c.id_cliente = p.id_cliente

WHERE c.ciudad IS NOT NULL

GROUP BY
    c.ciudad

HAVING COUNT(DISTINCT c.id_cliente) >= 3

ORDER BY
    calidad_email,              -- Primero agrupa por calidad
    total_ventas DESC;          -- Luego por ventas dentro de cada grupo
```

#### Resultado Esperado

El reporte integrará datos de calidad y actividad comercial en una sola vista. Ejemplo ilustrativo:

| CIUDAD    | TOTAL_CLIENTES | CLIENTES_CON_EMAIL | PCT_COMPLETITUD_EMAIL | TOTAL_PEDIDOS | TOTAL_VENTAS |
|-----------|----------------|--------------------|-----------------------|---------------|--------------|
| Bogotá    | 145            | 128                | 88.3                  | 127           | 458,320.00   |
| Medellín  | 98             | 71                 | 72.4                  | 89            | 312,150.50   |
| Cali      | 67             | 34                 | 50.7                  | 52            | 187,430.00   |
| ...       | ...            | ...                | ...                   | ...           | ...          |

La extensión opcional añadirá la columna `CALIDAD_EMAIL` con valores `'Alta'`, `'Media'` o `'Baja'`.

#### Verificación

✅ La consulta devuelve solo ciudades con `total_clientes >= 3`.
✅ El `pct_completitud_email` está entre 0 y 100 para todas las filas.
✅ Ciudades sin pedidos muestran `0` en `total_pedidos` (gracias al `COALESCE`).
✅ La consulta incluye: comentario de bloque, alias descriptivos, indentación consistente y palabras clave en mayúsculas.

---

## Validación y Pruebas del Laboratorio

Ejecuta las siguientes consultas de validación para confirmar que completaste correctamente todos los pasos del laboratorio:

```sql
/* ============================================================
   VALIDACIÓN FINAL - Lab 08-00-01
   Ejecuta cada bloque y verifica el resultado esperado
   ============================================================ */

-- VALIDACIÓN 1: Confirmar que la auditoría de nulos funciona correctamente
-- Resultado esperado: 1 fila con total_registros > 0
SELECT
    COUNT(*)                                                    AS total_registros,
    SUM(CASE WHEN email IS NULL THEN 1 ELSE 0 END)              AS nulos_email,
    ROUND(SUM(CASE WHEN email IS NULL THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS pct_nulos
FROM CLIENTES;

-- VALIDACIÓN 2: Confirmar que la detección de duplicados retorna resultado válido
-- Resultado esperado: 0 o más filas (sin error de sintaxis)
SELECT
    email,
    COUNT(*) AS ocurrencias
FROM CLIENTES
WHERE email IS NOT NULL
GROUP BY email
HAVING COUNT(*) > 1
ORDER BY ocurrencias DESC
LIMIT 5;

-- VALIDACIÓN 3: Confirmar que el patrón de integridad referencial funciona
-- Resultado esperado: 1 fila con un número >= 0
SELECT COUNT(*) AS pedidos_huerfanos
FROM PEDIDOS p
    LEFT JOIN CLIENTES c ON p.id_cliente = c.id_cliente
WHERE c.id_cliente IS NULL;

-- VALIDACIÓN 4: Confirmar que el reporte integrador funciona
-- Resultado esperado: Al menos 1 fila con todas las columnas correctamente calculadas
SELECT
    c.ciudad,
    COUNT(DISTINCT c.id_cliente)                                AS total_clientes,
    ROUND(
        COUNT(DISTINCT CASE WHEN c.email IS NOT NULL THEN c.id_cliente END) * 100.0
        / COUNT(DISTINCT c.id_cliente), 1
    )                                                           AS pct_completitud_email,
    COUNT(DISTINCT p.id_pedido)                                 AS total_pedidos
FROM CLIENTES c
    LEFT JOIN PEDIDOS p ON c.id_cliente = p.id_cliente
WHERE c.ciudad IS NOT NULL
GROUP BY c.ciudad
HAVING COUNT(DISTINCT c.id_cliente) >= 3
ORDER BY total_clientes DESC
LIMIT 3;
```

**Criterios de éxito:**

| Validación | Criterio de Éxito |
|------------|-------------------|
| Validación 1 | `total_registros > 0`, `pct_nulos` entre 0 y 100 |
| Validación 2 | Ejecuta sin error; devuelve 0 o más filas |
| Validación 3 | Devuelve exactamente 1 fila con un número entero |
| Validación 4 | Devuelve hasta 3 filas con todas las columnas numéricas válidas |

---

## Solución de Problemas

### Problema 1: Error "SQL compilation error: ambiguous column name"

**Síntoma:** Al ejecutar una consulta con `JOIN`, Snowflake devuelve el error:
```
SQL compilation error: ambiguous column name 'ID_CLIENTE'
```

**Causa:** Cuando dos tablas tienen una columna con el mismo nombre (por ejemplo, `id_cliente` existe tanto en `CLIENTES` como en `PEDIDOS`), Snowflake no puede determinar a cuál tabla te refieres si no especificas el alias de tabla.

**Solución:** Califica todas las columnas ambiguas con el alias de tabla correspondiente:

```sql
-- ❌ Incorrecto (ambiguo):
SELECT id_cliente, nombre, fecha_pedido
FROM CLIENTES
    JOIN PEDIDOS ON CLIENTES.id_cliente = PEDIDOS.id_cliente;

-- ✅ Correcto (calificado con alias):
SELECT
    c.id_cliente,       -- Especifica que viene de CLIENTES
    c.nombre,
    p.fecha_pedido      -- Especifica que viene de PEDIDOS
FROM CLIENTES c
    JOIN PEDIDOS p
        ON c.id_cliente = p.id_cliente;
```

**Prevención:** Como buena práctica, siempre usa alias de tabla (`c`, `p`, `dp`, `pr`) en todas las consultas con `JOIN` y califica todas las columnas con su alias correspondiente.

---

### Problema 2: La consulta de duplicados devuelve resultados inesperados con valores NULL

**Síntoma:** La consulta de detección de duplicados por email devuelve filas con `email = NULL` agrupadas juntas, o el conteo de duplicados parece inflado.

**Causa:** En SQL estándar y en Snowflake, `NULL` no es igual a `NULL` en comparaciones directas (`NULL = NULL` devuelve `NULL`, no `TRUE`). Sin embargo, en un `GROUP BY`, Snowflake agrupa todos los valores `NULL` juntos como si fueran un mismo valor. Esto hace que múltiples clientes sin email aparezcan como un "grupo duplicado" cuando en realidad no lo son.

**Solución:** Siempre filtrar los `NULL` con `WHERE columna IS NOT NULL` antes de aplicar la lógica de detección de duplicados:

```sql
-- ❌ Incorrecto (agrupa NULLs como si fueran duplicados):
SELECT
    email,
    COUNT(*) AS ocurrencias
FROM CLIENTES
GROUP BY email
HAVING COUNT(*) > 1;

-- ✅ Correcto (excluye NULLs antes de buscar duplicados):
SELECT
    email,
    COUNT(*) AS ocurrencias
FROM CLIENTES
WHERE email IS NOT NULL          -- ← Este filtro es ESENCIAL
GROUP BY email
HAVING COUNT(*) > 1
ORDER BY ocurrencias DESC;
```

**Regla general:** En cualquier auditoría de duplicados, siempre excluye los `NULL` de la columna analizada usando `WHERE columna IS NOT NULL`. Los `NULL` representan datos faltantes, no duplicados.

---

## Limpieza del Entorno

Al finalizar el laboratorio, realiza las siguientes acciones para mantener el entorno ordenado y minimizar el consumo de créditos Snowflake:

```sql
-- ============================================================
-- LIMPIEZA POST-LABORATORIO
-- ============================================================

-- 1. No es necesario eliminar tablas: este lab es de solo lectura.
--    Solo realizamos consultas SELECT, sin modificar datos.

-- 2. Guarda tu worksheet con un nombre descriptivo antes de cerrar:
--    Nombre sugerido: "Lab08_Validacion_Datos_[TuNombre]_[Fecha]"

-- 3. Suspende el Virtual Warehouse para detener el consumo de créditos:
ALTER WAREHOUSE CURSO_WH SUSPEND;

-- 4. Verifica que el warehouse quedó suspendido:
SHOW WAREHOUSES LIKE 'CURSO_WH';
-- La columna STATE debe mostrar: SUSPENDED
```

> **Importante:** Suspender el warehouse es especialmente crítico en cuentas trial con créditos limitados. Un warehouse activo sin consultas sigue consumiendo créditos. Verifica que el estado sea `SUSPENDED` antes de cerrar la sesión.

---

## Resumen

### Lo que Aprendiste en este Laboratorio

En este laboratorio de cierre aplicaste de forma integrada el conjunto completo de herramientas SQL del curso, enfocándote en la calidad de datos y las buenas prácticas profesionales:

| Técnica Aplicada                          | Paso(s) del Lab | Propósito                                              |
|-------------------------------------------|-----------------|--------------------------------------------------------|
| `IS NULL` / `IS NOT NULL`                 | 2, 4, 6         | Identificar y cuantificar datos faltantes              |
| `CASE WHEN ... IS NULL`                   | 2               | Contar nulos por columna en una sola consulta          |
| `GROUP BY + HAVING COUNT(*) > 1`          | 3               | Detectar registros duplicados potenciales              |
| `LEFT JOIN + WHERE padre IS NULL`         | 4               | Verificar integridad referencial (registros huérfanos) |
| Comentarios `--` y `/* */`                | 5, 6            | Documentar propósito, autor y lógica de negocio        |
| Indentación y alias descriptivos          | 5, 6            | Mejorar legibilidad y mantenibilidad del código        |
| `UNION ALL` para reportes consolidados    | 3E, 4D          | Combinar múltiples verificaciones en una sola vista    |
| `COALESCE()` para manejar NULLs           | 6               | Evitar resultados NULL en sumas con LEFT JOIN          |
| `ROUND()` para presentación de resultados | 2, 5, 6         | Mejorar la legibilidad de valores numéricos            |

### Conceptos Clave para Recordar

- **La calidad de los datos es una responsabilidad del analista:** Antes de construir cualquier reporte o análisis, es fundamental auditar el dataset para entender su completitud, consistencia e integridad.
- **El formato SQL no es opcional en entornos profesionales:** Una consulta bien estructurada (mayúsculas, indentación, alias, comentarios) es tan importante como una consulta correcta. El código se escribe una vez pero se lee muchas veces.
- **`NULL` requiere tratamiento especial:** Los valores `NULL` no se comportan como valores normales en comparaciones. Siempre usa `IS NULL` / `IS NOT NULL` y filtra los `NULL` antes de operaciones como detección de duplicados.
- **El patrón `LEFT JOIN + WHERE IS NULL` es fundamental:** Esta técnica es el método estándar para detectar registros huérfanos y verificar integridad referencial sin necesidad de subconsultas complejas.

### Recursos Adicionales

- [Documentación de Snowflake: Manejo de valores NULL](https://docs.snowflake.com/en/sql-reference/functions/is-null)
- [Documentación de Snowflake: Función COALESCE](https://docs.snowflake.com/en/sql-reference/functions/coalesce)
- [Guía de estilo SQL de GitLab — Referencia de formato profesional](https://handbook.gitlab.com/handbook/business-technology/data-team/platform/sql-style-guide/)
- [SQL Style Guide de Simon Holywell — Estándar de formato con ejemplos](https://www.sqlstyle.guide/)
- [Documentación de Snowflake: Mejores prácticas de consultas](https://docs.snowflake.com/en/user-guide/querying-best-practices)
- [Documentación de Snowflake: Sintaxis completa de SELECT](https://docs.snowflake.com/en/sql-reference/sql/select)

---

*Lab 08-00-01 — Versión 1.0 — Dataset CURSO_SQL v1.0*
*Snowflake Snowsight — Nivel: Aplicar (Bloom) — Duración: 30 minutos*

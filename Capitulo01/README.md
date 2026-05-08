# Exploración inicial de datos

## Metadatos

| Campo            | Detalle                          |
|------------------|----------------------------------|
| **Duración**     | 45 minutos                       |
| **Complejidad**  | Fácil                            |
| **Nivel Bloom**  | Aplicar (Apply)                  |
| **Módulo**       | 1 — Fundamentos de SQL en Snowflake |
| **Versión**      | 1.0                              |

---

## Descripción General

En este laboratorio explorarás por primera vez el entorno Snowflake Snowsight y las consultas SQL más fundamentales. Trabajando sobre la base de datos `CURSO_SQL`, examinarás la estructura de las tablas `PRODUCTOS` y `CLIENTES` —sus columnas, tipos de datos y registros— para construir una comprensión sólida antes de avanzar a operaciones más complejas. Aplicarás `SELECT`, `AS`, `DISTINCT` y `LIMIT` para recuperar, filtrar y presentar datos de forma controlada y legible.

---

## Objetivos de Aprendizaje

Al finalizar este laboratorio, serás capaz de:

- [ ] Identificar la estructura de una tabla en Snowflake reconociendo sus columnas, tipos de datos y registros.
- [ ] Ejecutar consultas `SELECT` básicas para recuperar todos los registros de una tabla y seleccionar columnas específicas.
- [ ] Aplicar alias con `AS` para mejorar la legibilidad de los resultados de una consulta.
- [ ] Utilizar `DISTINCT` para descubrir valores únicos en columnas categóricas.
- [ ] Emplear `LIMIT` para controlar la cantidad de filas devueltas por una consulta.

---

## Prerrequisitos

### Conocimientos previos

| Conocimiento | Nivel requerido |
|---|---|
| Concepto de tabla, fila y columna | Básico (conceptual) |
| Uso de hojas de cálculo (Excel / Google Sheets) como analogía | Referencial |
| Navegación general en un navegador web | Básico |
| Script de inicialización del dataset `CURSO_SQL` ejecutado exitosamente | **Obligatorio** |

### Acceso y recursos

| Recurso | Detalle |
|---|---|
| Cuenta Snowflake activa | Trial individual o corporativa con acceso a `CURSO_SQL` |
| Base de datos `CURSO_SQL` | Creada y poblada por el script del instructor (v1.0) |
| Navegador web | Chrome 90+, Firefox 88+, Edge 90+ o Safari 14+ |
| Conexión a internet | Mínimo 10 Mbps recomendado |

> **⚠️ Importante:** Si no has ejecutado el script de inicialización del dataset, detente aquí y solicita al instructor el archivo de setup antes de continuar. Todos los pasos de este laboratorio dependen de que las tablas `PRODUCTOS` y `CLIENTES` existan y contengan datos.

---

## Entorno del Laboratorio

### Hardware recomendado

| Componente | Mínimo | Recomendado |
|---|---|---|
| RAM | 4 GB | 8 GB |
| Resolución de pantalla | 1280 × 768 px | 1920 × 1080 px |
| Conexión a internet | 5 Mbps | 10 Mbps |

### Software

| Software | Versión | Notas |
|---|---|---|
| Snowflake (Snowsight) | Standard o superior | Acceso vía navegador |
| Navegador web | Chrome 90+ / Firefox 88+ | JavaScript habilitado obligatorio |

# Preparación obligatoria del dataset

Antes de iniciar los pasos del laboratorio, debes ejecutar el siguiente script. Este script crea la base de datos `CURSO_SQL`, el schema `PUBLIC`, las tablas `PRODUCTOS` y `CLIENTES`, e inserta registros suficientes para practicar consultas de exploración.

## ¿Qué crea este script?

| Objeto | Cantidad esperada | Descripción |
|---|---:|---|
| Base de datos | 1 | `CURSO_SQL` |
| Schema | 1 | `PUBLIC` |
| Tabla | 1 | `PRODUCTOS` |
| Tabla | 1 | `CLIENTES` |
| Registros de productos | 150 | Catálogo de productos con categorías, precios, stock y proveedor |
| Registros de clientes | 300 | Clientes con país, ciudad, segmento, correo y fecha de registro |

---

## Paso 0 — Crear y poblar el dataset **CLIENTES y PRODUCTOS** en `CURSO_SQL`

**Objetivo:** Preparar el entorno de trabajo creando las tablas requeridas para el laboratorio.

### Instrucciones

1. Inicia sesión en tu cuenta Snowflake en [app.snowflake.com](https://app.snowflake.com).
2. En el menú lateral izquierdo, haz clic en **Projects → Workspaces**.
3. Crea un nuevo private workspace.
4. Nombralo como **`Setup_CURSO_SQLSNOW`**.
5. Da clic en **Add new** y crea un nuevo archivo tipo **SQL**
6. Escribe el siguiente nombre del archivo: **`Setup_Lab01.sql`**
8. A la derecha selecciona el warehouse **`COMPUTE_WH`** o el warehouse asignado/creado al inicio de la creación de la cuenta.
9. Copia/Pega y ejecuta el siguiente script completo.

> **Nota:** En Snowsight puedes ejecutar todo el bloque completo. Si tu cuenta no permite crear bases de datos, ejecuta el script hasta donde tus permisos lo permitan o solicita apoyo del instructor.

```sql
-- ============================================================
-- Script de inicialización del dataset CURSO_SQL
-- Versión: 1.2
-- Uso: Ejecutar una sola vez antes del Lab 01
-- Ajuste: CLIENTES separa NOMBRE y APELLIDO en columnas distintas
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
    COUNT(CASE WHEN TELEFONO IS NULL THEN 1 END) AS CLIENTES_SIN_TELEFONO,
    COUNT(CASE WHEN TELEFONO IS NOT NULL THEN 1 END) AS CLIENTES_CON_TELEFONO
FROM CURSO_SQL.PUBLIC.CLIENTES;
```

### Resultado esperado del script

Al finalizar, la consulta de validación debe devolver:

| TABLA | TOTAL_REGISTROS |
|---|---:|
| PRODUCTOS | 150 |
| CLIENTES | 300 |

### Validación adicional del setup

Al final de la ultima consulta del script ejecuta estas consultas para confirmar que las tablas existen en el schema correcto:

```sql
SHOW TABLES IN SCHEMA CURSO_SQL.PUBLIC;
```

Resultado esperado: deben aparecer al menos las tablas `PRODUCTOS` y `CLIENTES`.

También puedes ejecutar:

```sql
SELECT CURRENT_DATABASE(), CURRENT_SCHEMA(), CURRENT_WAREHOUSE();
```

Resultado esperado:

| CURRENT_DATABASE() | CURRENT_SCHEMA() | CURRENT_WAREHOUSE() |
|---|---|---|
| CURSO_SQL | PUBLIC | COMPUTE_WH |

---

### Configuración inicial del entorno

Antes de comenzar los ejercicios, debes verificar que tu entorno Snowflake está correctamente configurado. Sigue estos pasos de setup una sola vez al inicio de la sesión.

**Paso de configuración — Abrir un Worksheet y seleccionar el contexto correcto:**

1. Haz clic en el botón **＋** (esquina superior derecha) para crear un nuevo Private Workspace.
2. Escribe el siguiente nombre: **`SnowEssLabs`**
3. Agrega un nuevo archivo tipo **SQL** y escribe el siguiente nombre: **`Lab_01_Exploracion_Inicial`**
4. En la barra superior derecha del workspace, selecciona el contexto de ejecución:
   - **Warehouse:** `COMPUTE_WH` (tamaño X-Small)
   - **Database:** `CURSO_SQL`
   - **Schema:** `PUBLIC`

5. Ejecuta el siguiente bloque de configuración para confirmar que el contexto está activo:

```sql
-- Configuración del contexto de trabajo
-- Ejecuta este bloque al inicio de cada sesión de laboratorio

USE WAREHOUSE COMPUTE_WH;
USE DATABASE CURSO_SQL;
USE SCHEMA PUBLIC;   -- Reemplaza PUBLIC por tu schema si trabajas en entorno compartido

-- Verificación: debe mostrar la base de datos y schema activos
SELECT CURRENT_DATABASE(), CURRENT_SCHEMA(), CURRENT_WAREHOUSE();
```

**Resultado esperado del setup:**

| CURRENT_DATABASE() | CURRENT_SCHEMA() | CURRENT_WAREHOUSE() |
|---|---|---|
| CURSO_SQL | PUBLIC | COMPUTE_WH |

> **💡 Nota sobre costos:** Utiliza siempre un Virtual Warehouse de tamaño **X-Small** durante este curso para minimizar el consumo de créditos. Recuerda apagar el warehouse al finalizar la sesión.

---

## Pasos del Laboratorio

---

### Paso 1 — Explorar la estructura de la tabla PRODUCTOS

**Objetivo:** Comprender qué columnas y tipos de datos contiene la tabla `PRODUCTOS` antes de consultarla, aplicando el concepto de estructura de tabla aprendido en la Lección 1.1.

#### Instrucciones

1. En tu archivo SQL `Lab_01_Exploracion_Inicial`, escribe y ejecuta la siguiente consulta para ver la definición completa de la tabla `PRODUCTOS`:

```sql
-- Paso 1.1: Describir la estructura de la tabla PRODUCTOS
-- Este comando muestra las columnas, tipos de datos y propiedades de cada campo

DESCRIBE TABLE PRODUCTOS;
```

2. Observa los resultados. Identifica y anota mentalmente (o en papel):
   - ¿Cuántas columnas tiene la tabla?
   - ¿Cuál parece ser la clave primaria (columna de identificación única)?
   - ¿Qué columnas contienen texto (`TEXT` / `VARCHAR`)?
   - ¿Qué columnas contienen números (`NUMBER` / `FLOAT`)?

3. Ahora ejecuta el mismo comando para la tabla `CLIENTES`:

```sql
-- Paso 1.2: Describir la estructura de la tabla CLIENTES

DESCRIBE TABLE CLIENTES;
```

4. Compara ambas estructuras. ¿Qué diferencias observas entre los tipos de datos de `PRODUCTOS` y `CLIENTES`?

#### Resultado esperado

Al ejecutar `DESCRIBE TABLE PRODUCTOS`, verás una tabla con columnas similares a estas (los nombres exactos dependen del script del instructor):

| name | type | kind | null? | default | primary key |
|---|---|---|---|---|---|
| ID_PRODUCTO | NUMBER(38,0) | COLUMN | N | — | Y |
| NOMBRE_PRODUCTO | VARCHAR(200) | COLUMN | Y | — | N |
| CATEGORIA | VARCHAR(100) | COLUMN | Y | — | N |
| PRECIO | NUMBER(10,2) | COLUMN | Y | — | N |
| STOCK | NUMBER(38,0) | COLUMN | Y | — | N |
| PROVEEDOR | VARCHAR(150) | COLUMN | Y | — | N |

> **📌 Referencia a la Lección 1.1:** Cada fila de este resultado corresponde a una **columna** de la tabla. La columna `type` muestra el **tipo de dato** (VARCHAR para texto, NUMBER para números). La columna marcada como `primary key = Y` es la **clave primaria** que identifica de forma única cada registro.

#### Verificación

✅ Confirma que puedes responder estas preguntas antes de continuar:
- ¿La tabla `PRODUCTOS` tiene una columna de tipo `NUMBER` que actúa como clave primaria?
- ¿La tabla `CLIENTES` tiene al menos una columna de tipo `DATE` o `TIMESTAMP`?

---

### Paso 2 — Recuperar todos los registros con SELECT *

**Objetivo:** Ejecutar una consulta `SELECT *` para obtener una vista completa de todos los datos almacenados en una tabla, y comprender por qué esta forma de consulta es útil para la exploración inicial pero no para producción.

#### Instrucciones

1. Escribe y ejecuta la siguiente consulta para recuperar **todos los registros y columnas** de la tabla `PRODUCTOS`:

```sql
-- Paso 2.1: Recuperar todos los registros de PRODUCTOS
-- El asterisco (*) es un comodín que significa "todas las columnas"

SELECT *
FROM PRODUCTOS;
```

2. Observa el panel de resultados en Snowsight. Nota:
   - El número total de filas devueltas (visible en la esquina inferior del panel de resultados).
   - Los nombres de las columnas en la fila de encabezado.
   - El tipo de valores en cada columna (texto, números, fechas).

3. Ahora repite el ejercicio para la tabla `CLIENTES`:

```sql
-- Paso 2.2: Recuperar todos los registros de CLIENTES

SELECT *
FROM CLIENTES;
```

4. Compara visualmente los resultados de ambas tablas. ¿Cuántos registros tiene cada una?

#### Resultado esperado

La consulta sobre `PRODUCTOS` debe devolver todas las filas de la tabla, con todas sus columnas visibles. El panel de resultados de Snowsight mostrará algo similar a:

| ID_PRODUCTO | NOMBRE_PRODUCTO | CATEGORIA | PRECIO | STOCK | PROVEEDOR |
|---|---|---|---|---|---|
| 1 | Laptop UltraBook Pro | Electrónica | 1299.99 | 45 | TechSupply S.A. |
| 2 | Silla Ergonómica Mesh | Mobiliario | 349.50 | 120 | OfficeWorld |
| 3 | Monitor 4K 27" | Electrónica | 599.00 | 30 | TechSupply S.A. |
| … | … | … | … | … | … |

> **💡 Buena práctica:** `SELECT *` es excelente para exploración inicial y para entender qué datos existen. Sin embargo, en consultas de producción o análisis, es mejor seleccionar solo las columnas que necesitas (lo harás en el Paso 3). Esto mejora el rendimiento y la legibilidad.

#### Verificación

✅ Antes de continuar, verifica:
- ¿La consulta devolvió filas con datos reales (no está vacía)?
- ¿Puedes identificar visualmente qué columnas contienen texto y cuáles contienen números?
- ¿El número de filas de `PRODUCTOS` y `CLIENTES` es diferente?

---

### Paso 3 — Seleccionar columnas específicas

**Objetivo:** Refinar las consultas `SELECT` para recuperar únicamente las columnas relevantes, reduciendo el volumen de datos devueltos y mejorando la claridad de los resultados.

#### Instrucciones

1. En lugar de usar `*`, especifica únicamente las columnas que te interesan. Ejecuta la siguiente consulta para obtener solo el nombre, categoría y precio de los productos:

```sql
-- Paso 3.1: Seleccionar columnas específicas de PRODUCTOS
-- Listamos los nombres de columna separados por comas

SELECT NOMBRE_PRODUCTO, CATEGORIA, PRECIO
FROM PRODUCTOS;
```

2. Compara este resultado con el del Paso 2. ¿Qué columnas desaparecieron? ¿Es más fácil leer la información ahora?

3. Ahora practica con la tabla `CLIENTES`. Recupera solo el nombre del cliente, su correo electrónico y su país de origen:

```sql
-- Paso 3.2: Seleccionar columnas específicas de CLIENTES
-- Ajusta los nombres de columna según la estructura que viste en el Paso 1

SELECT NOMBRE, CORREO, PAIS
FROM CLIENTES;
```

4. Intenta una variación: selecciona el `ID_PRODUCTO`, `NOMBRE_PRODUCTO` y `STOCK` de la tabla `PRODUCTOS`. Esto te da una vista de inventario simplificada:

```sql
-- Paso 3.3: Vista de inventario simplificada

SELECT ID_PRODUCTO, NOMBRE_PRODUCTO, STOCK
FROM PRODUCTOS;
```

#### Resultado esperado

La consulta del Paso 3.1 debe devolver exactamente 3 columnas (en lugar de todas):

| NOMBRE_PRODUCTO | CATEGORIA | PRECIO |
|---|---|---|
| Laptop UltraBook Pro | Electrónica | 1299.99 |
| Silla Ergonómica Mesh | Mobiliario | 349.50 |
| Monitor 4K 27" | Electrónica | 599.00 |
| … | … | … |

> **📌 Concepto clave:** Al listar columnas específicas después de `SELECT`, le dices a Snowflake exactamente qué información quieres ver. El orden en que las listas determina el orden en que aparecen en los resultados. Puedes listar las columnas en cualquier orden, independientemente de cómo estén definidas en la tabla.

#### Verificación

✅ Confirma que:
- La consulta 3.1 devuelve exactamente 3 columnas: `NOMBRE_PRODUCTO`, `CATEGORIA` y `PRECIO`.
- La consulta 3.2 devuelve columnas de texto (nombre, correo) y al menos una columna categórica (país).
- Si escribiste mal el nombre de una columna, Snowflake devuelve un error. ¿Puedes identificar el mensaje de error y corregirlo?

---

### Paso 4 — Aplicar alias con AS para mejorar la legibilidad

**Objetivo:** Usar la cláusula `AS` para renombrar columnas en los resultados de una consulta, haciendo los encabezados más descriptivos y amigables para el lector final.

#### Instrucciones

1. Los nombres de columna originales de la base de datos a veces son técnicos o abreviados. Usa `AS` para asignar nombres más legibles en los resultados. Ejecuta:

```sql
-- Paso 4.1: Aplicar alias a columnas de PRODUCTOS
-- AS renombra la columna SOLO en el resultado; no modifica la tabla

SELECT 
    NOMBRE_PRODUCTO  AS "Nombre del Producto",
    CATEGORIA        AS "Categoría",
    PRECIO           AS "Precio (USD)"
FROM PRODUCTOS;
```

2. Observa los encabezados de columna en el resultado. ¿Cómo cambiaron respecto al Paso 3?

3. Prueba también alias sin comillas dobles (para nombres de alias simples, sin espacios ni caracteres especiales):

```sql
-- Paso 4.2: Alias sin comillas (para nombres simples)
-- Cuando el alias no tiene espacios ni caracteres especiales, las comillas son opcionales

SELECT 
    ID_PRODUCTO      AS id,
    NOMBRE_PRODUCTO  AS producto,
    PRECIO           AS precio_usd,
    STOCK            AS unidades_disponibles
FROM PRODUCTOS;
```

4. Aplica alias a la tabla `CLIENTES` para crear una vista más presentable:

```sql
-- Paso 4.3: Alias en la tabla CLIENTES

SELECT 
    NOMBRE           AS "Cliente",
    CORREO           AS "Correo Electrónico",
    PAIS             AS "País de Origen",
    FECHA_REGISTRO   AS "Fecha de Alta"
FROM CLIENTES;
```

5. **Ejercicio adicional:** Combina una columna calculada simple con un alias. Snowflake permite hacer operaciones aritméticas directamente en el `SELECT`:

```sql
-- Paso 4.4: Alias en expresiones calculadas
-- Calculamos el valor total del inventario por producto (precio × stock)

SELECT 
    NOMBRE_PRODUCTO                AS "Producto",
    PRECIO                         AS "Precio Unitario",
    STOCK                          AS "Unidades",
    PRECIO * STOCK                 AS "Valor Total en Inventario"
FROM PRODUCTOS;
```

#### Resultado esperado

La consulta 4.1 debe mostrar encabezados con espacios y tildes, tal como los definiste:

| Nombre del Producto | Categoría | Precio (USD) |
|---|---|---|
| Laptop UltraBook Pro | Electrónica | 1299.99 |
| Silla Ergonómica Mesh | Mobiliario | 349.50 |
| Monitor 4K 27" | Electrónica | 599.00 |

La consulta 4.4 debe incluir una columna adicional calculada:

| Producto | Precio Unitario | Unidades | Valor Total en Inventario |
|---|---|---|---|
| Laptop UltraBook Pro | 1299.99 | 45 | 58499.55 |
| Silla Ergonómica Mesh | 349.50 | 120 | 41940.00 |

> **💡 Importante:** `AS` es una cláusula de presentación. Renombra la columna **únicamente en el resultado de la consulta**. La tabla original en la base de datos no se modifica en absoluto. Si en otra consulta intentas usar `"Nombre del Producto"` como referencia en un `WHERE`, obtendrás un error — debes usar siempre el nombre original de la columna.

> **📝 Nota de Snowflake:** En Snowflake, los alias entre comillas dobles son sensibles a mayúsculas/minúsculas. `AS "precio"` y `AS "PRECIO"` son alias diferentes. Para evitar confusiones, usa comillas dobles solo cuando el alias contenga espacios o caracteres especiales.

#### Verificación

✅ Confirma que:
- Los encabezados de columna en el resultado de 4.1 muestran texto con espacios (ej. "Nombre del Producto"), no el nombre original de la columna.
- La consulta 4.4 devuelve una columna numérica calculada con el producto de `PRECIO × STOCK`.
- Si eliminas `AS` y el alias de una columna, el encabezado vuelve al nombre original de la columna.

---

### Paso 5 — Usar DISTINCT para encontrar valores únicos

**Objetivo:** Aplicar `DISTINCT` para eliminar filas duplicadas en los resultados y descubrir los valores únicos presentes en columnas categóricas como `CATEGORIA` o `PAIS`.

#### Instrucciones

1. Primero, observa qué ocurre sin `DISTINCT` al consultar solo la columna `CATEGORIA` de `PRODUCTOS`:

```sql
-- Paso 5.1: Sin DISTINCT — muestra todos los valores, incluidos duplicados
-- Observa cuántas veces se repite cada categoría

SELECT CATEGORIA
FROM PRODUCTOS;
```

2. Ahora aplica `DISTINCT` para ver solo los valores únicos:

```sql
-- Paso 5.2: Con DISTINCT — elimina duplicados y muestra cada categoría una sola vez
-- Útil para descubrir qué categorías existen en el catálogo

SELECT DISTINCT CATEGORIA
FROM PRODUCTOS;
```

3. Compara el número de filas devueltas por 5.1 y 5.2. ¿Cuántas categorías únicas tiene el catálogo de productos?

4. Aplica `DISTINCT` a la tabla `CLIENTES` para descubrir los países únicos de los clientes:

```sql
-- Paso 5.3: Países únicos de clientes

SELECT DISTINCT PAIS
FROM CLIENTES;
```

5. Puedes usar `DISTINCT` con múltiples columnas. En ese caso, elimina filas donde la **combinación** de columnas es duplicada:

```sql
-- Paso 5.4: DISTINCT con múltiples columnas
-- Devuelve combinaciones únicas de categoría y proveedor

SELECT DISTINCT CATEGORIA, PROVEEDOR
FROM PRODUCTOS;
```

6. Combina `DISTINCT` con un alias para presentar el resultado de forma más clara:

```sql
-- Paso 5.5: DISTINCT con alias

SELECT DISTINCT 
    CATEGORIA  AS "Categoría de Producto",
    PROVEEDOR  AS "Proveedor"
FROM PRODUCTOS;
```

#### Resultado esperado

La consulta 5.1 devolverá tantas filas como productos haya en la tabla, con categorías repetidas. La consulta 5.2 devolverá solo una fila por categoría única, por ejemplo:

| CATEGORIA |
|---|
| Electrónica |
| Mobiliario |
| Papelería |
| Herramientas |

La consulta 5.3 mostrará los países únicos registrados en la tabla `CLIENTES`:

| PAIS |
|---|
| México |
| Colombia |
| Argentina |
| España |
| Chile |

> **💡 Caso de uso real:** `DISTINCT` es especialmente valioso cuando recibes una base de datos nueva y quieres entender qué valores existen en una columna categórica antes de escribir filtros. Por ejemplo, si quieres filtrar por categoría pero no sabes exactamente cómo están escritos los valores (¿"Electrónica" o "electronica" o "ELECTRONICA"?), `DISTINCT` te lo revela de inmediato.

> **⚠️ Consideración de rendimiento:** En tablas muy grandes (millones de filas), `DISTINCT` puede ser costoso computacionalmente porque debe comparar todos los registros. Para este laboratorio con el dataset `CURSO_SQL` no hay problema, pero tenlo en cuenta en entornos de producción.

#### Verificación

✅ Confirma que:
- La consulta 5.2 devuelve significativamente menos filas que la 5.1 (solo las categorías únicas).
- La consulta 5.4 devuelve combinaciones únicas de `CATEGORIA` + `PROVEEDOR`, no solo categorías únicas.
- Si la tabla `CLIENTES` tiene clientes de 5 países diferentes, la consulta 5.3 devuelve exactamente 5 filas.

---

### Paso 6 — Controlar el volumen de resultados con LIMIT

**Objetivo:** Usar `LIMIT` para restringir el número de filas devueltas por una consulta, lo cual es útil para obtener muestras rápidas de datos o para explorar la estructura de resultados sin procesar toda la tabla.

#### Instrucciones

1. Ejecuta la siguiente consulta para obtener solo las primeras 5 filas de `PRODUCTOS`:

```sql
-- Paso 6.1: Obtener solo las primeras 5 filas de PRODUCTOS
-- LIMIT controla cuántas filas devuelve la consulta

SELECT *
FROM PRODUCTOS
LIMIT 5;
```

2. Modifica el valor de `LIMIT` para obtener 10 filas:

```sql
-- Paso 6.2: Obtener las primeras 10 filas

SELECT *
FROM PRODUCTOS
LIMIT 10;
```

3. Combina `LIMIT` con la selección de columnas específicas para obtener una muestra limpia:

```sql
-- Paso 6.3: Muestra limpia con columnas específicas y LIMIT

SELECT 
    NOMBRE_PRODUCTO  AS "Producto",
    CATEGORIA        AS "Categoría",
    PRECIO           AS "Precio (USD)"
FROM PRODUCTOS
LIMIT 5;
```

4. Aplica `LIMIT` a la tabla `CLIENTES`:

```sql
-- Paso 6.4: Muestra de los primeros 3 clientes registrados

SELECT 
    NOMBRE           AS "Cliente",
    PAIS             AS "País",
    FECHA_REGISTRO   AS "Fecha de Registro"
FROM CLIENTES
LIMIT 3;
```

5. Combina todo lo aprendido hasta ahora en una sola consulta: selección de columnas + alias + LIMIT:

```sql
-- Paso 6.5: Consulta combinada — resumen ejecutivo del catálogo
-- Muestra las primeras 8 filas con columnas seleccionadas y alias descriptivos

SELECT 
    ID_PRODUCTO      AS "ID",
    NOMBRE_PRODUCTO  AS "Nombre del Producto",
    CATEGORIA        AS "Categoría",
    PRECIO           AS "Precio (USD)",
    STOCK            AS "Stock Disponible"
FROM PRODUCTOS
LIMIT 8;
```

#### Resultado esperado

La consulta 6.1 devuelve exactamente 5 filas (independientemente de cuántos registros tenga la tabla completa):

| ID_PRODUCTO | NOMBRE_PRODUCTO | CATEGORIA | PRECIO | STOCK | PROVEEDOR |
|---|---|---|---|---|---|
| 1 | Laptop UltraBook Pro | Electrónica | 1299.99 | 45 | TechSupply S.A. |
| 2 | Silla Ergonómica Mesh | Mobiliario | 349.50 | 120 | OfficeWorld |
| 3 | Monitor 4K 27" | Electrónica | 599.00 | 30 | TechSupply S.A. |
| 4 | Teclado Mecánico RGB | Electrónica | 89.99 | 200 | TechSupply S.A. |
| 5 | Escritorio Ajustable | Mobiliario | 499.00 | 55 | OfficeWorld |

La consulta 6.5 devuelve 8 filas con exactamente 5 columnas y encabezados con alias:

| ID | Nombre del Producto | Categoría | Precio (USD) | Stock Disponible |
|---|---|---|---|---|
| 1 | Laptop UltraBook Pro | Electrónica | 1299.99 | 45 |
| 2 | Silla Ergonómica Mesh | Mobiliario | 349.50 | 120 |
| … | … | … | … | … |

> **💡 Cuándo usar LIMIT:**
> - **Exploración inicial:** Para ver una muestra rápida de cómo se ven los datos sin cargar toda la tabla.
> - **Desarrollo de consultas:** Para probar que tu query funciona correctamente antes de ejecutarla sobre millones de filas.
> - **Demos y reportes:** Para mostrar solo los N registros más relevantes.
>
> **⚠️ Nota importante:** `LIMIT` no garantiza ningún orden específico de las filas devueltas. Sin una cláusula `ORDER BY` (que aprenderás en el Lab 03), las filas devueltas por `LIMIT` pueden variar entre ejecuciones. Para obtener resultados consistentes, combina siempre `LIMIT` con `ORDER BY`.

#### Verificación

✅ Confirma que:
- La consulta 6.1 devuelve exactamente 5 filas, no más.
- La consulta 6.5 devuelve exactamente 8 filas con 5 columnas y encabezados con alias.
- Si cambias `LIMIT 5` a `LIMIT 100` en una tabla con 50 registros, Snowflake devuelve solo los 50 registros disponibles (no genera error).

---

## Validación y Pruebas

Una vez completados todos los pasos, ejecuta las siguientes consultas de validación para confirmar que dominas los conceptos del laboratorio. Estas consultas integran múltiples conceptos en una sola instrucción.

### Validación 1 — Consulta integradora sobre PRODUCTOS

```sql
-- VALIDACIÓN 1: Integra SELECT columnas específicas + AS + LIMIT
-- Objetivo: Obtener un reporte de los primeros 10 productos con información de valor

SELECT 
    ID_PRODUCTO                AS "Código",
    NOMBRE_PRODUCTO            AS "Descripción del Producto",
    CATEGORIA                  AS "Línea",
    PRECIO                     AS "P.V.P. (USD)",
    STOCK                      AS "Disponibilidad",
    PRECIO * STOCK             AS "Valor en Inventario (USD)"
FROM PRODUCTOS
LIMIT 10;
```

**Resultado esperado:** 10 filas con 6 columnas. La última columna `"Valor en Inventario (USD)"` debe mostrar el resultado de multiplicar `PRECIO × STOCK` para cada producto.

### Validación 2 — Consulta integradora sobre CLIENTES con DISTINCT

```sql
-- VALIDACIÓN 2: Integra DISTINCT + AS para análisis de distribución geográfica
-- Objetivo: Obtener la lista de países únicos donde la empresa tiene clientes

SELECT DISTINCT 
    PAIS  AS "País"
FROM CLIENTES;
```

**Resultado esperado:** Una lista sin duplicados de todos los países representados en la tabla `CLIENTES`. Cada país debe aparecer exactamente una vez.

### Validación 3 — Consulta de exploración completa

```sql
-- VALIDACIÓN 3: Exploración completa combinando todos los conceptos
-- Objetivo: Obtener un catálogo de categorías y proveedores únicos con muestra limitada

SELECT DISTINCT 
    CATEGORIA   AS "Categoría",
    PROVEEDOR   AS "Proveedor Asociado"
FROM PRODUCTOS
LIMIT 5;
```

**Resultado esperado:** Máximo 5 filas (o menos si hay menos de 5 combinaciones únicas), mostrando pares únicos de categoría y proveedor.

### Lista de verificación final

Marca cada ítem una vez que hayas confirmado el resultado correcto:

- [ ] `DESCRIBE TABLE PRODUCTOS` muestra todas las columnas con sus tipos de datos correctos.
- [ ] `SELECT *` sobre `PRODUCTOS` devuelve todas las filas y columnas de la tabla.
- [ ] `SELECT NOMBRE_PRODUCTO, CATEGORIA, PRECIO FROM PRODUCTOS` devuelve exactamente 3 columnas.
- [ ] Los alias definidos con `AS` aparecen como encabezados de columna en los resultados.
- [ ] `SELECT DISTINCT CATEGORIA FROM PRODUCTOS` devuelve menos filas que `SELECT CATEGORIA FROM PRODUCTOS`.
- [ ] `SELECT * FROM PRODUCTOS LIMIT 5` devuelve exactamente 5 filas.
- [ ] La consulta de validación 1 muestra una columna calculada (`PRECIO * STOCK`) con alias descriptivo.

---

## Resolución de Problemas

### Problema 1 — Error: "Object does not exist" al consultar PRODUCTOS o CLIENTES

**Síntoma:**
Al ejecutar `SELECT * FROM PRODUCTOS` o `DESCRIBE TABLE CLIENTES`, Snowflake devuelve un mensaje de error similar a:
```
SQL compilation error: Object 'PRODUCTOS' does not exist or not authorized.
```

**Causa:**
El contexto de base de datos o schema no está configurado correctamente en el worksheet. Snowflake no sabe en qué base de datos o schema buscar la tabla `PRODUCTOS` porque no se ha seleccionado el contexto adecuado. Esto ocurre cuando el selector de Database/Schema en la barra superior del worksheet está vacío o apunta a una base de datos diferente.

**Solución:**
1. Verifica los selectores en la barra superior del worksheet: asegúrate de que **Database** esté en `CURSO_SQL` y **Schema** en `PUBLIC` (o el schema asignado por tu instructor).
2. Alternativamente, ejecuta explícitamente los comandos de contexto al inicio del worksheet:

```sql
-- Solución: Establecer el contexto explícitamente
USE DATABASE CURSO_SQL;
USE SCHEMA PUBLIC;

-- Luego reintenta tu consulta
SELECT * FROM PRODUCTOS;
```

3. Si el error persiste, verifica con tu instructor que el script de inicialización del dataset fue ejecutado correctamente y que tienes permisos de acceso a la base de datos `CURSO_SQL`.

---

### Problema 2 — Los resultados de DISTINCT no cambian o parecen incorrectos

**Síntoma:**
Al ejecutar `SELECT DISTINCT CATEGORIA FROM PRODUCTOS`, el número de filas devuelto es igual al de `SELECT CATEGORIA FROM PRODUCTOS`, o bien aparecen valores que parecen duplicados (ej. "Electrónica" y "electrónica" como dos categorías distintas).

**Causa:**
En Snowflake, `DISTINCT` es sensible a mayúsculas y minúsculas en los valores de datos. Si la columna `CATEGORIA` contiene valores con capitalización inconsistente (ej. `"Electrónica"`, `"ELECTRÓNICA"`, `"electrónica"`), `DISTINCT` los tratará como tres valores diferentes porque son cadenas de texto distintas. Este es un problema de calidad de datos en el dataset, no un error de SQL.

**Solución:**
Para verificar si el problema es de capitalización inconsistente, ejecuta:

```sql
-- Diagnóstico: Ver todos los valores de CATEGORIA con sus variaciones exactas
SELECT CATEGORIA, COUNT(*) AS cantidad
FROM PRODUCTOS
GROUP BY CATEGORIA
ORDER BY CATEGORIA;
```

Si confirmas que hay variaciones de capitalización, puedes normalizar temporalmente los valores usando la función `UPPER()` o `LOWER()` para la comparación:

```sql
-- Solución temporal: Normalizar a mayúsculas para obtener categorías únicas sin importar capitalización
SELECT DISTINCT UPPER(CATEGORIA) AS "Categoría (Normalizada)"
FROM PRODUCTOS;
```

> **Nota:** Esta solución es temporal para visualización. La corrección permanente requeriría actualizar los datos en la tabla, lo cual está fuera del alcance de este laboratorio.

---

## Limpieza del Entorno

Al finalizar el laboratorio, realiza las siguientes acciones para conservar los créditos de tu cuenta Snowflake y mantener el entorno ordenado.

### 1. Guardar el worksheet

Snowflake guarda los worksheets automáticamente, pero es buena práctica verificar:

1. Confirma que el worksheet se llama `Lab_01_Exploracion_Inicial`.
2. Snowsight guarda el contenido automáticamente al escribir. No es necesario hacer clic en "Guardar".

### 2. Suspender el Virtual Warehouse

Para detener el consumo de créditos, suspende el warehouse manualmente:

```sql
-- Suspender el Virtual Warehouse para evitar consumo innecesario de créditos
-- Ejecuta esto al finalizar CADA sesión de laboratorio

ALTER WAREHOUSE COMPUTE_WH SUSPEND;
```

**Resultado esperado:**
```
Statement executed successfully.
```

Alternativamente, desde la interfaz Snowsight:
1. Ve al menú lateral → **Manage → Compute → Warehouses**.
2. Localiza `COMPUTE_WH`.
3. Haz clic en los tres puntos (`⋯`) → **Suspend**.

### 3. Verificar que no hay consultas activas

```sql
-- Verificar que no hay queries en ejecución antes de cerrar sesión
SELECT *
FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY())
WHERE EXECUTION_STATUS = 'RUNNING'
LIMIT 5;
```

Si el resultado está vacío, es seguro cerrar la sesión.

> **⚠️ Recordatorio:** No elimines ninguna tabla ni datos del schema `CURSO_SQL`. Los laboratorios posteriores (Lab 02 en adelante) dependen del mismo dataset. La limpieza de este laboratorio consiste únicamente en suspender el warehouse y cerrar la sesión.

---

## Resumen

En este laboratorio completaste tu primera exploración de datos en Snowflake Snowsight. Aplicaste los conceptos fundamentales de la Lección 1.1 sobre estructura de tablas y los pusiste en práctica con SQL real sobre el dataset `CURSO_SQL`.

### Conceptos y comandos cubiertos

| Concepto / Comando | Qué aprendiste |
|---|---|
| `DESCRIBE TABLE` | Ver la estructura de una tabla: columnas, tipos de datos y clave primaria |
| `SELECT *` | Recuperar todos los registros y columnas de una tabla |
| `SELECT col1, col2` | Seleccionar solo las columnas necesarias para un análisis |
| `AS` (alias) | Renombrar columnas en los resultados para mejorar la legibilidad |
| `DISTINCT` | Eliminar duplicados y descubrir valores únicos en columnas categóricas |
| `LIMIT n` | Controlar el número de filas devueltas para muestras y exploración |

### Buenas prácticas aprendidas

- ✅ Usar `DESCRIBE TABLE` antes de consultar una tabla desconocida para entender su estructura.
- ✅ Preferir columnas específicas sobre `SELECT *` en consultas de análisis y producción.
- ✅ Aplicar alias descriptivos con `AS` para que los resultados sean autoexplicativos.
- ✅ Usar `DISTINCT` para explorar la diversidad de valores en columnas categóricas.
- ✅ Combinar `LIMIT` con otras cláusulas para obtener muestras rápidas durante el desarrollo.
- ✅ Suspender el warehouse al finalizar cada sesión para conservar créditos.

### Próximos pasos

En el **Lab 01-00-02** aprenderás a filtrar registros usando la cláusula `WHERE`, que te permitirá recuperar solo los datos que cumplen condiciones específicas. Construirás sobre las consultas de este laboratorio añadiendo filtros por texto, números y rangos de valores — el paso natural siguiente para pasar de "ver todos los datos" a "encontrar los datos que necesito".

---

## Recursos Adicionales

| Recurso | URL |
|---|---|
| Documentación Snowflake: Comando SELECT | https://docs.snowflake.com/en/sql-reference/sql/select |
| Documentación Snowflake: DESCRIBE TABLE | https://docs.snowflake.com/en/sql-reference/sql/desc-table |
| Documentación Snowflake: Tipos de datos | https://docs.snowflake.com/en/sql-reference/data-types |
| W3Schools: SQL SELECT Statement | https://www.w3schools.com/sql/sql_select.asp |
| W3Schools: SQL SELECT DISTINCT | https://www.w3schools.com/sql/sql_distinct.asp |
| Snowflake: Guía de inicio rápido con Snowsight | https://docs.snowflake.com/en/user-guide/ui-snowsight-worksheets |

---
*Lab 01-00-01 — Versión 1.0 — Curso SQL en Snowflake*

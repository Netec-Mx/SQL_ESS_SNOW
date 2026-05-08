# Filtrado de datos

## Metadatos

| Campo            | Detalle                                      |
|------------------|----------------------------------------------|
| **Duración**     | 60 minutos                                   |
| **Complejidad**  | Media                                        |
| **Nivel Bloom**  | Aplicar (Apply)                              |
| **Módulo**       | Módulo 2 – Filtrado de Datos                 |
| **Versión**      | 1.0                                          |
| **Dataset**      | CURSO_SQL (provisto por el instructor)       |

---

## Descripción General

En este laboratorio aplicarás la cláusula `WHERE` para segmentar datos reales de ventas y clientes almacenados en Snowflake. Comenzarás con condiciones simples de igualdad y comparación numérica, avanzarás hacia filtros combinados con `AND`, `OR` y `NOT`, y terminarás utilizando `IN`, `LIKE` e `IS NULL` para cubrir escenarios de análisis del mundo real. Al finalizar, habrás construido un conjunto progresivo de consultas que demuestran cómo extraer subconjuntos precisos de información a partir de tablas con miles de registros.

---

## Objetivos de Aprendizaje

Al completar este laboratorio, serás capaz de:

- [ ] Aplicar la cláusula `WHERE` para filtrar registros según condiciones específicas sobre columnas de texto, número y fecha.
- [ ] Utilizar operadores de comparación (`=`, `>`, `<`, `>=`, `<=`, `<>`) para definir criterios de filtrado numérico y de texto.
- [ ] Combinar múltiples condiciones con los operadores lógicos `AND`, `OR` y `NOT` para construir filtros compuestos.
- [ ] Emplear el operador `IN` y el operador `LIKE` (con `%` y `_`) para filtrar listas de valores y patrones de texto.
- [ ] Identificar registros con valores nulos o no nulos utilizando `IS NULL` e `IS NOT NULL`.

---

## Requisitos Previos

### Conocimientos

- Haber completado **Lab 01-00-01** o demostrar dominio de `SELECT`, selección de columnas específicas, `DISTINCT` y `LIMIT`.
- Comprensión de operadores matemáticos básicos de comparación (`=`, `>`, `<`).
- Familiaridad con la interfaz Snowsight: crear worksheets, ejecutar consultas y leer resultados.

### Acceso y Recursos

- Cuenta Snowflake activa (trial o corporativa) con acceso a Snowsight.
- Base de datos `CURSO_SQL` correctamente inicializada con el script del instructor (Lab 00 – Setup).
- Virtual Warehouse de tamaño **X-Small** disponible y en estado `STARTED`.
- Credenciales de acceso a Snowflake proporcionadas por el instructor o por la cuenta trial propia.

---

## Entorno del Laboratorio

### Hardware Recomendado

| Componente        | Mínimo Requerido                                          |
|-------------------|-----------------------------------------------------------|
| Conectividad      | 10 Mbps estables                                          |
| RAM               | 4 GB (para navegador con múltiples pestañas)              |
| Resolución        | 1280 × 768 px                                             |
| Sistema Operativo | Windows 10+, macOS 10.15+, o distribución Linux moderna   |

### Software Requerido

| Software              | Versión / Detalle                                          |
|-----------------------|------------------------------------------------------------|
| Navegador web         | Chrome 90+, Firefox 88+, Edge 90+ o Safari 14+             |
| Snowflake (Snowsight) | Edición Standard o superior – interfaz web incluida        |
| Dataset del curso     | `CURSO_SQL` v1.0 (script de inicialización del instructor) |

> **Recomendación:** Usa Chrome sin extensiones adicionales si experimentas problemas con la interfaz Snowsight. Asegúrate de tener JavaScript habilitado.

---

## Paso 0 — Crear y poblar el dataset **VENTAS** en `CURSO_SQL`

**Objetivo:** Preparar el entorno de trabajo creando las tablas requeridas para el laboratorio.

### Instrucciones

1. Inicia sesión en tu cuenta Snowflake en [app.snowflake.com](https://app.snowflake.com).
2. En el menú lateral izquierdo, haz clic en **Projects → Workspaces**.
3. Selecciona el Workspace llamado **`Setup_CURSO_SQLSNOW`**.
5. Da clic en **Add new** y crea un nuevo archivo tipo **SQL**
6. Escribe el siguiente nombre del archivo: **`Setup_Lab02_Ventas.sql`**
7. A la derecha selecciona el warehouse **`COMPUTE_WH`** o el warehouse asignado/creado al inicio de la creación de la cuenta.
8. Copia/Pega y ejecuta el siguiente script completo.

> **Nota:** En Snowsight puedes ejecutar todo el bloque completo. Si tu cuenta no permite crear bases de datos, ejecuta el script hasta donde tus permisos lo permitan o solicita apoyo del instructor.

```sql
-- ============================================================
-- LAB 02 - Script de inicialización de datos para tabla VENTAS
-- Base de datos: CURSO_SQL
-- Schema: PUBLIC
-- Requisitos:
--   1. La tabla CLIENTES debe existir previamente
--   2. La tabla PRODUCTOS debe existir previamente
-- Registros generados: 1,200 ventas
-- Ajuste aplicado:
--   - Se agrega ID_PRODUCTO a VENTAS
--   - PRODUCTO, CATEGORIA y PRECIO_UNITARIO se toman desde PRODUCTOS
--   - Esto permite que los JOIN con PRODUCTOS coincidan correctamente
-- ============================================================

USE DATABASE CURSO_SQL;
USE SCHEMA PUBLIC;
USE WAREHOUSE COMPUTE_WH;

-- Verificación previa: confirmar que CLIENTES existe y tiene datos
SELECT COUNT(*) AS total_clientes
FROM CLIENTES;

-- Verificación previa: confirmar que PRODUCTOS existe y tiene datos
SELECT COUNT(*) AS total_productos
FROM PRODUCTOS;

-- Crear la tabla VENTAS con la estructura requerida para laboratorios de filtrado y JOIN
CREATE OR REPLACE TABLE VENTAS (
    ID_VENTA NUMBER,
    ID_CLIENTE NUMBER,
    ID_PRODUCTO NUMBER,
    PRODUCTO VARCHAR,
    CATEGORIA VARCHAR,
    CANTIDAD NUMBER,
    PRECIO_UNITARIO NUMBER(10,2),
    TOTAL_VENTA NUMBER(10,2),
    FECHA_VENTA DATE,
    REGION VARCHAR,
    ESTADO_ENVIO VARCHAR,
    VENDEDOR VARCHAR
);

-- Insertar 1,200 registros sintéticos de ventas
-- Los ID_CLIENTE se toman desde CLIENTES
-- Los ID_PRODUCTO, PRODUCTO, CATEGORIA y PRECIO_UNITARIO se toman desde PRODUCTOS
-- para que los JOIN con PRODUCTOS sean consistentes.
INSERT INTO VENTAS (
    ID_VENTA,
    ID_CLIENTE,
    ID_PRODUCTO,
    PRODUCTO,
    CATEGORIA,
    CANTIDAD,
    PRECIO_UNITARIO,
    TOTAL_VENTA,
    FECHA_VENTA,
    REGION,
    ESTADO_ENVIO,
    VENDEDOR
)
WITH clientes_ordenados AS (
    SELECT
        ID_CLIENTE,
        ROW_NUMBER() OVER (ORDER BY ID_CLIENTE) AS rn
    FROM CLIENTES
),
total_clientes AS (
    SELECT COUNT(*) AS total
    FROM clientes_ordenados
),
productos_ordenados AS (
    SELECT
        ID_PRODUCTO,
        NOMBRE_PRODUCTO,
        CATEGORIA,
        PRECIO,
        ROW_NUMBER() OVER (ORDER BY ID_PRODUCTO) AS rn
    FROM PRODUCTOS
),
total_productos AS (
    SELECT COUNT(*) AS total
    FROM productos_ordenados
),
base AS (
    SELECT
        ROW_NUMBER() OVER (ORDER BY SEQ4()) AS n
    FROM TABLE(GENERATOR(ROWCOUNT => 1200))
),
ventas_base AS (
    SELECT
        b.n AS ID_VENTA,
        c.ID_CLIENTE,
        p.ID_PRODUCTO,
        p.NOMBRE_PRODUCTO AS PRODUCTO,
        p.CATEGORIA,
        MOD(b.n, 5) + 1 AS CANTIDAD,
        p.PRECIO AS PRECIO_UNITARIO,
        DATEADD(DAY, MOD(b.n * 3, 1095), DATE '2022-01-01') AS FECHA_VENTA,
        CASE MOD(b.n, 5)
            WHEN 0 THEN 'Norte'
            WHEN 1 THEN 'Sur'
            WHEN 2 THEN 'Centro'
            WHEN 3 THEN 'Occidente'
            ELSE 'Oriente'
        END AS REGION,
        CASE
            WHEN MOD(b.n, 11) = 0 THEN NULL
            WHEN MOD(b.n, 4) = 0 THEN 'Pendiente'
            WHEN MOD(b.n, 4) = 1 THEN 'Enviado'
            WHEN MOD(b.n, 4) = 2 THEN 'Entregado'
            ELSE 'Cancelado'
        END AS ESTADO_ENVIO,
        CASE MOD(b.n, 10)
            WHEN 0 THEN 'Laura Méndez'
            WHEN 1 THEN 'Roberto Salazar'
            WHEN 2 THEN 'Patricia Núñez'
            WHEN 3 THEN 'Fernando Rojas'
            WHEN 4 THEN 'Gabriela Silva'
            WHEN 5 THEN 'Héctor Molina'
            WHEN 6 THEN 'Natalia Reyes'
            WHEN 7 THEN 'Oscar Cabrera'
            WHEN 8 THEN 'Diana Paredes'
            ELSE 'Manuel Ortega'
        END AS VENDEDOR
    FROM base b
    CROSS JOIN total_clientes tc
    CROSS JOIN total_productos tp
    JOIN clientes_ordenados c
      ON c.rn = MOD(b.n - 1, tc.total) + 1
    JOIN productos_ordenados p
      ON p.rn = MOD(b.n - 1, tp.total) + 1
)
SELECT
    ID_VENTA,
    ID_CLIENTE,
    ID_PRODUCTO,
    PRODUCTO,
    CATEGORIA,
    CANTIDAD,
    PRECIO_UNITARIO,
    CAST(CANTIDAD * PRECIO_UNITARIO AS NUMBER(10,2)) AS TOTAL_VENTA,
    FECHA_VENTA,
    REGION,
    ESTADO_ENVIO,
    VENDEDOR
FROM ventas_base;

-- Validación general de carga
SELECT COUNT(*) AS total_ventas
FROM VENTAS;

-- Validación de relación VENTAS -> PRODUCTOS
-- Debe devolver 0. Si devuelve más de 0, existen ventas con productos inexistentes.
SELECT COUNT(*) AS ventas_sin_producto_relacionado
FROM VENTAS V
LEFT JOIN PRODUCTOS P
    ON V.ID_PRODUCTO = P.ID_PRODUCTO
WHERE P.ID_PRODUCTO IS NULL;

-- Validación de coincidencia entre VENTAS y PRODUCTOS
-- Debe devolver 0. Si devuelve más de 0, hay diferencias entre el producto/categoría/precio de VENTAS y PRODUCTOS.
SELECT COUNT(*) AS ventas_con_datos_de_producto_diferentes
FROM VENTAS V
JOIN PRODUCTOS P
    ON V.ID_PRODUCTO = P.ID_PRODUCTO
WHERE V.PRODUCTO <> P.NOMBRE_PRODUCTO
   OR V.CATEGORIA <> P.CATEGORIA
   OR V.PRECIO_UNITARIO <> P.PRECIO;

-- Validación de valores necesarios para la práctica
SELECT DISTINCT REGION
FROM VENTAS
ORDER BY REGION;

SELECT DISTINCT CATEGORIA
FROM VENTAS
ORDER BY CATEGORIA;

SELECT DISTINCT VENDEDOR
FROM VENTAS
ORDER BY VENDEDOR;

SELECT
    COUNT(*) AS ventas_sin_estado_envio
FROM VENTAS
WHERE ESTADO_ENVIO IS NULL;

SELECT *
FROM VENTAS
LIMIT 10;
```

#### Resultado Esperado

Al finalizar la ejecución del script, la tabla `VENTAS` queda disponible con datos suficientes para completar todos los pasos de la práctica.

---

## Desarrollo del Laboratorio

### Referencia Rápida: Estructura de las Tablas

Antes de filtrar datos, es fundamental conocer la estructura de las tablas con las que trabajarás.

**Tabla `CLIENTES`** — Información de clientes registrados:

| Columna          | Tipo de Dato   | Descripción                            |
|------------------|----------------|----------------------------------------|
| ID_CLIENTE       | NUMBER         | Identificador único del cliente        |
| NOMBRE           | VARCHAR        | Nombre completo del cliente            |
| EMAIL            | VARCHAR        | Correo electrónico                     |
| PAIS             | VARCHAR        | País de residencia                     |
| CIUDAD           | VARCHAR        | Ciudad de residencia                   |
| SEGMENTO         | VARCHAR        | Segmento comercial (Retail, Corporativo, etc.) |
| FECHA_REGISTRO   | DATE           | Fecha en que se registró el cliente    |
| TELEFONO         | VARCHAR        | Número de teléfono (puede ser NULL)    |

**Tabla `VENTAS`** — Registro de transacciones de venta:

| Columna          | Tipo de Dato   | Descripción                             |
|------------------|----------------|-----------------------------------------|
| ID_VENTA         | NUMBER         | Identificador único de la venta         |
| ID_CLIENTE       | NUMBER         | Cliente que realizó la compra           |
| PRODUCTO         | VARCHAR        | Nombre del producto vendido             |
| CATEGORIA        | VARCHAR        | Categoría del producto                  |
| CANTIDAD         | NUMBER         | Unidades vendidas                       |
| PRECIO_UNITARIO  | NUMBER(10,2)   | Precio por unidad                       |
| TOTAL_VENTA      | NUMBER(10,2)   | Monto total de la transacción           |
| FECHA_VENTA      | DATE           | Fecha de la venta                       |
| REGION           | VARCHAR        | Región geográfica de la venta           |
| ESTADO_ENVIO     | VARCHAR        | Estado del envío (puede ser NULL)       |

---

### Paso 1: Exploración Previa — Conocer los Datos Antes de Filtrar

**Objetivo:** Revisar el contenido completo de las tablas para entender qué valores existen en las columnas que usaremos para filtrar. Este paso evita errores comunes como filtrar por un valor que no existe o escribir mal una cadena de texto.

#### Instrucciones

1. Abre tu **Workspace** llamado **SnowEssLabs**
2. Crea un nuevo archivo tipo **SQL**. Nómbrala **`Lab02_Filtrado`**.
3. Selecciona tu **Compute-WH** 
4. Selecciona tu base de datos **CURSO_SQL** y esquema **PUBLIC**

5. Ejecuta la siguiente consulta para ver una muestra de la tabla `CLIENTES`:

```sql
-- Vista previa de la tabla CLIENTES
SELECT *
FROM CLIENTES
LIMIT 10;
```

3. Ejecuta esta consulta para ver los valores únicos de la columna `PAIS`:

```sql
-- ¿Qué países existen en la tabla?
SELECT DISTINCT PAIS
FROM CLIENTES
ORDER BY PAIS;
```

4. Ejecuta esta consulta para conocer los valores únicos de `SEGMENTO`:

```sql
-- ¿Qué segmentos de clientes existen?
SELECT DISTINCT SEGMENTO
FROM CLIENTES
ORDER BY SEGMENTO;
```

5. Ahora revisa la tabla `VENTAS` con una muestra y los valores únicos de `REGION` y `CATEGORIA`:

```sql
-- Vista previa de la tabla VENTAS
SELECT *
FROM VENTAS
LIMIT 10;
```
```sql
-- Regiones disponibles
SELECT DISTINCT REGION
FROM VENTAS
ORDER BY REGION;
```
```sql
-- Categorías de productos disponibles
SELECT DISTINCT CATEGORIA
FROM VENTAS
ORDER BY CATEGORIA;
```

#### Resultado Esperado

- La consulta de `CLIENTES` mostrará 10 filas con datos de clientes de distintos países.
- `DISTINCT PAIS` mostrará entre 3 y 6 países diferentes (ej. México, Colombia, Argentina, Chile, Perú).
- `DISTINCT SEGMENTO` mostrará entre 2 y 4 segmentos (ej. Retail, Corporativo, Gobierno, PYME).
- `DISTINCT REGION` en `VENTAS` mostrará entre 3 y 5 regiones geográficas.
- `DISTINCT CATEGORIA` mostrará entre 4 y 6 categorías de productos.

#### Verificación

Anota en papel o en un comentario de la worksheet los valores exactos que ves en `PAIS`, `SEGMENTO`, `REGION` y `CATEGORIA`. Los usarás en los pasos siguientes. Si alguna consulta devuelve 0 filas, el dataset no está cargado correctamente — consulta al instructor.

> **Buena práctica:** Siempre explora tus datos antes de escribir filtros. Conocer los valores reales evita errores de tipeo y resultados vacíos inesperados.

---

### Paso 2: Filtrado Simple con WHERE — Condiciones de Igualdad

**Objetivo:** Aplicar `WHERE` con el operador `=` para filtrar registros por valores exactos en columnas de texto y número.

#### Instrucciones

1. Filtra todos los clientes de un país específico. Usa uno de los países que encontraste en el Paso 1 (el ejemplo usa `'México'` — ajusta al valor exacto que viste en tu dataset):

```sql
-- Clientes de México
SELECT ID_CLIENTE, NOMBRE, PAIS, CIUDAD
FROM CLIENTES
WHERE PAIS = 'México';
```

2. Filtra los clientes que pertenecen al segmento `'Corporativo'` (ajusta al nombre exacto de tu dataset):

```sql
-- Clientes del segmento Corporativo
SELECT ID_CLIENTE, NOMBRE, SEGMENTO, PAIS
FROM CLIENTES
WHERE SEGMENTO = 'Corporativo';
```

3. Filtra las ventas donde la cantidad vendida fue exactamente 1 unidad:

```sql
-- Ventas de exactamente 1 unidad
SELECT ID_VENTA, PRODUCTO, CANTIDAD, TOTAL_VENTA
FROM VENTAS
WHERE CANTIDAD = 1;
```

4. Filtra las ventas de una región específica (usa un valor de `REGION` que hayas encontrado en el Paso 1):

```sql
-- Ventas de la región Norte (ajusta el valor según tu dataset)
SELECT ID_VENTA, PRODUCTO, REGION, TOTAL_VENTA
FROM VENTAS
WHERE REGION = 'Norte';
```

#### Resultado Esperado

- La consulta 1 devuelve solo clientes cuyo `PAIS` es exactamente `'México'`. No deben aparecer clientes de otros países.
- La consulta 2 muestra únicamente clientes con `SEGMENTO = 'Corporativo'`.
- La consulta 3 muestra ventas donde `CANTIDAD` es `1`. Observa que el número `1` no lleva comillas.
- La consulta 4 muestra ventas de la región seleccionada.

#### Verificación

Después de ejecutar la consulta 1, observa el contador de filas en la parte superior derecha de Snowsight. Luego ejecuta:

```sql
-- Verificación: el total de filas filtradas + el resto debe sumar el total de la tabla
SELECT COUNT(*) AS total_clientes FROM CLIENTES;
```
```sql
SELECT COUNT(*) AS clientes_mexico FROM CLIENTES WHERE PAIS = 'México';
```

Si el número de `clientes_mexico` es mayor que cero y menor que `total_clientes`, el filtro está funcionando correctamente.

> **Punto clave:** Los valores de texto van siempre entre comillas simples (`'México'`). Los valores numéricos nunca llevan comillas (`CANTIDAD = 1`). Mezclar esto genera errores de tipo de dato en Snowflake.

---

### Paso 3: Operadores de Comparación — Filtros Numéricos y de Rango

**Objetivo:** Utilizar los operadores `>`, `<`, `>=`, `<=` y `<>` para filtrar registros según rangos y exclusiones numéricas.

#### Instrucciones

1. Encuentra todas las ventas con un total mayor a $500:

```sql
-- Ventas con total superior a $500
SELECT ID_VENTA, PRODUCTO, TO_VARCHAR(TOTAL_VENTA, '$999,999,999.00'), FECHA_VENTA
FROM VENTAS
WHERE TOTAL_VENTA > 500;
```

2. Encuentra las ventas con precio unitario menor o igual a $50:

```sql
-- Productos económicos: precio unitario <= $50
SELECT ID_VENTA, PRODUCTO, PRECIO_UNITARIO, CATEGORIA
FROM VENTAS
WHERE PRECIO_UNITARIO <= 100;
```

3. Encuentra los clientes que se registraron a partir del año 2023 (fecha mayor o igual al 1 de enero de 2023):

```sql
-- Clientes registrados desde 2023 en adelante
SELECT ID_CLIENTE, NOMBRE, FECHA_REGISTRO, PAIS
FROM CLIENTES
WHERE FECHA_REGISTRO >= '2023-01-01';
```

4. Encuentra las ventas donde la cantidad vendida es diferente de 1 unidad (es decir, ventas de más de una unidad o ventas de volumen):

```sql
-- Ventas donde la cantidad NO es 1 (operador "diferente de")
SELECT ID_VENTA, PRODUCTO, CANTIDAD, TOTAL_VENTA
FROM VENTAS
WHERE CANTIDAD <> 1;
```

5. Aplica un filtro de rango: ventas cuyo total esté entre $100 y $1000 (sin usar BETWEEN — solo con operadores de comparación):

```sql
-- Ventas en rango $100 - $1000 usando solo operadores de comparación
-- Nota: En la siguiente lección aprenderás BETWEEN como alternativa más legible
SELECT ID_VENTA, PRODUCTO, TOTAL_VENTA
FROM VENTAS
WHERE TOTAL_VENTA >= 100
  AND TOTAL_VENTA <= 1000;
```

#### Resultado Esperado

- La consulta 1 devuelve solo ventas con `TOTAL_VENTA` estrictamente mayor a 500.
- La consulta 2 devuelve productos con precio unitario de 50 o menos (incluye el 50).
- La consulta 3 devuelve clientes cuya `FECHA_REGISTRO` es `2023-01-01` o posterior. Nota que la fecha va entre comillas simples en formato `'YYYY-MM-DD'`.
- La consulta 4 excluye las ventas de exactamente 1 unidad.
- La consulta 5 devuelve ventas en el rango indicado. Anticipa el uso de `AND` — lo estudiarás en detalle en el siguiente paso.

#### Verificación

```sql
-- Verifica que la consulta 1 no incluye ventas de exactamente $500
SELECT MIN(TOTAL_VENTA) AS minimo_resultado
FROM VENTAS
WHERE TOTAL_VENTA > 500;
-- El resultado debe ser mayor a 500, nunca igual
```

> **Diferencia entre `>` y `>=`:** El operador `>` es estrictamente mayor (excluye el valor exacto). El operador `>=` incluye el valor exacto. En análisis de datos, esta distinción es crítica, especialmente con fechas y umbrales financieros.

---

### Paso 4: Condiciones Compuestas con AND, OR y NOT

**Objetivo:** Combinar múltiples condiciones de filtrado para construir consultas más precisas que reflejen escenarios de análisis reales.

#### Instrucciones

1. **AND — Ambas condiciones deben ser verdaderas.** Encuentra ventas de alto valor en una región específica:

```sql
-- Ventas mayores a $500 EN la región Norte
-- (Ajusta 'Norte' al nombre de región que existe en tu dataset)
SELECT ID_VENTA, PRODUCTO, REGION, TOTAL_VENTA, FECHA_VENTA
FROM VENTAS
WHERE TOTAL_VENTA > 500
  AND REGION = 'Norte';
```

2. **AND con múltiples condiciones.** Clientes del segmento Corporativo registrados en 2023 o después:

```sql
-- Clientes corporativos recientes
SELECT ID_CLIENTE, NOMBRE, SEGMENTO, FECHA_REGISTRO, PAIS
FROM CLIENTES
WHERE SEGMENTO = 'Corporativo'
  AND FECHA_REGISTRO >= '2023-01-01';
```

3. **OR — Al menos una condición debe ser verdadera.** Ventas de dos categorías de producto (usa dos categorías reales de tu dataset):

```sql
-- Ventas de categoría Electrónica O Ropa
-- (Ajusta los nombres según los valores reales de tu dataset)
SELECT ID_VENTA, PRODUCTO, CATEGORIA, TOTAL_VENTA
FROM VENTAS
WHERE CATEGORIA = 'Electrónica'
   OR CATEGORIA = 'Ropa';
```

4. **NOT — Negación de una condición.** Clientes que NO son de México:

```sql
-- Clientes de cualquier país excepto México
SELECT ID_CLIENTE, NOMBRE, PAIS, SEGMENTO
FROM CLIENTES
WHERE NOT PAIS = 'México';
```

5. **Combinación de AND y OR con paréntesis.** Este es un caso realista: ventas de alto valor (más de $300) que sean de la categoría Electrónica O de la región Sur:

```sql
-- Ventas importantes: alto valor Y (Electrónica O región Sur)
-- Los paréntesis son CRÍTICOS para controlar el orden de evaluación
SELECT ID_VENTA, PRODUCTO, CATEGORIA, REGION, TOTAL_VENTA
FROM VENTAS
WHERE TOTAL_VENTA > 300
  AND (CATEGORIA = 'Electrónica' OR REGION = 'Sur');
```

6. **Sin paréntesis — comportamiento diferente.** Observa cómo cambia el resultado al quitar los paréntesis:

```sql
-- MISMA consulta SIN paréntesis — resultado diferente (AND tiene precedencia sobre OR)
-- Esto demuestra por qué los paréntesis son importantes
SELECT ID_VENTA, PRODUCTO, CATEGORIA, REGION, TOTAL_VENTA
FROM VENTAS
WHERE TOTAL_VENTA > 300
  AND CATEGORIA = 'Electrónica' OR REGION = 'Sur';
```

#### Resultado Esperado

- La consulta 1 devuelve solo ventas que cumplen AMBAS condiciones simultáneamente.
- La consulta 3 devuelve ventas de Electrónica más ventas de Ropa (unión de ambos grupos).
- La consulta 4 devuelve todos los clientes excepto los de México.
- Las consultas 5 y 6 producen resultados **diferentes** entre sí, lo que demuestra el impacto de los paréntesis.

#### Verificación

```sql
-- Compara el conteo de filas entre la consulta CON y SIN paréntesis
SELECT COUNT(*) AS con_parentesis
FROM VENTAS
WHERE TOTAL_VENTA > 300
  AND (CATEGORIA = 'Electrónica' OR REGION = 'Sur');

SELECT COUNT(*) AS sin_parentesis
FROM VENTAS
WHERE TOTAL_VENTA > 300
  AND CATEGORIA = 'Electrónica' OR REGION = 'Sur';
```

Si los dos conteos son diferentes (lo cual es probable), confirma que entiendes por qué. Discútelo con el instructor o un compañero.

> **Regla de precedencia:** En SQL, `AND` se evalúa antes que `OR`, igual que la multiplicación se evalúa antes que la suma en matemáticas. Siempre usa paréntesis cuando combines `AND` y `OR` para hacer tu intención explícita y evitar bugs difíciles de detectar.

---

### Paso 5: El Operador IN — Filtrar por Lista de Valores

**Objetivo:** Usar el operador `IN` como alternativa más limpia y legible a múltiples condiciones `OR` cuando se filtra por una lista de valores conocidos.

#### Instrucciones

1. Filtra clientes de tres países específicos usando `IN`:

```sql
-- Clientes de México, Colombia o Argentina
-- Equivalente a: WHERE PAIS = 'México' OR PAIS = 'Colombia' OR PAIS = 'Argentina'
SELECT ID_CLIENTE, NOMBRE, PAIS, CIUDAD, SEGMENTO
FROM CLIENTES
WHERE PAIS IN ('México', 'Colombia', 'Argentina');
```

2. Filtra ventas de múltiples categorías:

```sql
-- Ventas de tres categorías específicas
-- Ajusta los nombres de categoría según los valores reales de tu dataset
SELECT ID_VENTA, PRODUCTO, CATEGORIA, TOTAL_VENTA
FROM VENTAS
WHERE CATEGORIA IN ('Electrónica', 'Ropa', 'Hogar');
```

3. Usa `NOT IN` para excluir una lista de valores. Encuentra ventas de todas las regiones excepto dos:

```sql
-- Ventas de todas las regiones EXCEPTO Norte y Sur
SELECT ID_VENTA, PRODUCTO, REGION, TOTAL_VENTA
FROM VENTAS
WHERE REGION NOT IN ('Norte', 'Sur');
```

4. Combina `IN` con otras condiciones usando `AND`. Clientes de países seleccionados que además pertenecen al segmento Retail:

```sql
-- Clientes Retail de países específicos
SELECT ID_CLIENTE, NOMBRE, PAIS, SEGMENTO, FECHA_REGISTRO
FROM CLIENTES
WHERE PAIS IN ('México', 'Chile', 'Perú')
  AND SEGMENTO = 'Retail';
```

5. **Comparación directa:** Escribe la misma consulta del punto 1 pero usando `OR` en lugar de `IN`, y compara los resultados:

```sql
-- Misma lógica con OR (para comparar con IN)
SELECT ID_CLIENTE, NOMBRE, PAIS, CIUDAD, SEGMENTO
FROM CLIENTES
WHERE PAIS = 'México'
   OR PAIS = 'Colombia'
   OR PAIS = 'Argentina';
```

#### Resultado Esperado

- Las consultas 1 y 5 deben devolver exactamente el mismo número de filas y los mismos registros.
- La consulta 3 con `NOT IN` excluye las regiones listadas y muestra solo las demás.
- La consulta 4 aplica `IN` y `AND` juntos, devolviendo solo clientes que cumplen ambos criterios.

#### Verificación

```sql
-- Confirma que IN y OR producen el mismo resultado
SELECT COUNT(*) AS conteo_IN
FROM CLIENTES
WHERE PAIS IN ('México', 'Colombia', 'Argentina');

SELECT COUNT(*) AS conteo_OR
FROM CLIENTES
WHERE PAIS = 'México'
   OR PAIS = 'Colombia'
   OR PAIS = 'Argentina';
-- Ambos conteos deben ser iguales
```

> **¿Cuándo usar `IN` vs. `OR`?** Usa `IN` cuando tienes 3 o más valores en la lista — es más legible y menos propenso a errores. Con 2 valores, `OR` es igualmente claro. Con listas largas (10+ valores), `IN` es claramente superior. Además, `IN` tiene mejor rendimiento en algunos motores de base de datos.

---

### Paso 6: El Operador LIKE — Búsqueda de Patrones en Texto

**Objetivo:** Usar `LIKE` con los comodines `%` (cualquier cantidad de caracteres) y `_` (exactamente un carácter) para realizar búsquedas de patrones en columnas de texto.

#### Instrucciones

1. Encuentra clientes cuyo nombre comienza con la letra `'A'`:

```sql
-- Clientes cuyo nombre empieza con A
-- % significa "cualquier cantidad de caracteres después de A"
SELECT ID_CLIENTE, NOMBRE, PAIS, CORREO
FROM CLIENTES
WHERE NOMBRE LIKE 'A%';
```

2. Encuentra clientes cuyo nombre termina con `'ez'` (apellidos como López, Pérez, Gómez):

```sql
-- Clientes cuyo nombre termina en 'ez'
-- % al inicio significa "cualquier texto antes de ez"
SELECT ID_CLIENTE, NOMBRE, PAIS
FROM CLIENTES
WHERE NOMBRE LIKE '%1';
```

3. Encuentra clientes cuyo nombre o apellido contiene la cadena `'ar'` en cualquier posición:

```sql
-- Clientes cuyo nombre contiene 'ar' en cualquier posición
-- % en ambos lados: cualquier texto antes y después
SELECT ID_CLIENTE, NOMBRE, PAIS
FROM CLIENTES
WHERE NOMBRE LIKE '%ar%';
```

4. Busca productos cuyo nombre contiene la palabra `'Pro'` (productos de línea profesional):

```sql
-- Productos que contienen 'Pro' en el nombre
SELECT ID_VENTA, PRODUCTO, CATEGORIA, PRECIO_UNITARIO
FROM VENTAS
WHERE PRODUCTO LIKE '%Pro%';
```

5. Usa el comodín `_` (guion bajo) para buscar un patrón con longitud exacta. Encuentra clientes cuyo email tiene exactamente un carácter antes del símbolo `@` del dominio `gmail.com`:

```sql
-- Patrón con _ : exactamente UN carácter en esa posición
-- Ejemplo: busca emails donde el dominio empieza con una letra específica
-- _ representa exactamente 1 carácter
SELECT ID_CLIENTE, NOMBRE, CORREO
FROM CLIENTES
WHERE CORREO LIKE '%@example.com';
-- Esto coincide con @gmail.com, @amail.com, @zmail.com, etc.
```

6. Usa `NOT LIKE` para excluir patrones. Encuentra ventas de productos que NO sean de una línea específica:

```sql
-- Productos que NO contienen 'Basic' en el nombre
SELECT ID_VENTA, PRODUCTO, CATEGORIA, TOTAL_VENTA
FROM VENTAS
WHERE PRODUCTO NOT LIKE '%Basic%';
```

#### Resultado Esperado

- La consulta 1 devuelve clientes cuyo `NOMBRE` comienza exactamente con la letra `A` (mayúscula).
- La consulta 2 devuelve clientes cuyo nombre termina en `ez`.
- La consulta 4 devuelve todos los productos que contienen `'Pro'` en cualquier parte del nombre.
- La consulta 5 demuestra el uso de `_` para posiciones exactas.

#### Verificación

```sql
-- Verifica que todos los resultados de la consulta 1 empiezan con A
SELECT NOMBRE
FROM CLIENTES
WHERE NOMBRE LIKE 'A%'
ORDER BY NOMBRE;
-- Revisa visualmente que cada nombre en el resultado comience con A
```

> **Sensibilidad a mayúsculas/minúsculas en Snowflake:** Por defecto, Snowflake trata las comparaciones de texto como **insensibles a mayúsculas** en la cláusula `WHERE` con `=`, pero `LIKE` puede ser sensible dependiendo de la configuración del collation. Si `LIKE 'A%'` no devuelve resultados esperados, prueba con `ILIKE 'A%'` (Snowflake-specific: case-insensitive LIKE). Consulta al instructor si experimentas este comportamiento.

---

### Paso 7: IS NULL e IS NOT NULL — Identificar Datos Faltantes

**Objetivo:** Identificar registros con valores nulos (datos faltantes) e incluirlos o excluirlos del análisis según el caso de uso.

#### Instrucciones

1. Encuentra los clientes que **no tienen correo electronico registrado** (campo `CORREO` es NULL):

```sql
-- Clientes sin teléfono registrado
SELECT ID_CLIENTE, NOMBRE, PAIS, TELEFONO
FROM CLIENTES
WHERE TELEFONO IS NULL;
```

2. Encuentra los clientes que **sí tienen teléfono registrado**:

```sql
-- Clientes CON teléfono registrado
SELECT ID_CLIENTE, NOMBRE, PAIS, TELEFONO
FROM CLIENTES
WHERE TELEFONO IS NOT NULL;
```

3. Encuentra las ventas donde el `ESTADO_ENVIO` es desconocido (NULL), lo que podría indicar registros incompletos:

```sql
-- Ventas sin estado de envío registrado (registros incompletos)
SELECT ID_VENTA, PRODUCTO, FECHA_VENTA, ESTADO_ENVIO
FROM VENTAS
WHERE ESTADO_ENVIO IS NULL;
```

4. Demuestra por qué **no se puede usar `=` con NULL**. Ejecuta esta consulta incorrecta y observa que devuelve 0 filas aunque haya NULLs:

```sql
-- INCORRECTO: Esta consulta siempre devuelve 0 filas
-- NULL no es igual a nada, ni siquiera a NULL
SELECT ID_CLIENTE, NOMBRE, TELEFONO
FROM CLIENTES
WHERE TELEFONO = NULL;
-- Compara con la consulta IS NULL del punto 1 — verás la diferencia
```

5. Combina `IS NULL` con otras condiciones. Encuentra clientes mexicanos sin teléfono registrado (datos incompletos de clientes prioritarios):

```sql
-- Clientes de México con teléfono faltante (requieren seguimiento)
SELECT ID_CLIENTE, NOMBRE, PAIS, EMAIL, TELEFONO
FROM CLIENTES
WHERE PAIS = 'México'
  AND TELEFONO IS NULL;
```

6. Cuenta cuántos registros tienen valores nulos vs. no nulos para evaluar la calidad del dato:

```sql
-- Reporte de calidad de datos: teléfonos faltantes vs. registrados
SELECT
    COUNT(*) AS total_clientes,
    COUNT(CASE WHEN TELEFONO IS NULL THEN 1 END) AS sin_telefono,
    COUNT(CASE WHEN TELEFONO IS NOT NULL THEN 1 END) AS con_telefono
FROM CLIENTES;
```

> **Nota:** La instrucción `CASE WHEN` es un concepto avanzado que verás en módulos posteriores. Por ahora, observa el resultado — es una forma de contar NULLs y no-NULLs en una sola consulta.

#### Resultado Esperado

- La consulta 1 devuelve solo clientes con `TELEFONO = NULL`.
- La consulta 2 devuelve solo clientes con teléfono registrado.
- La consulta 4 devuelve **0 filas** aunque existan NULLs en la tabla, demostrando que `= NULL` no funciona.
- La consulta 6 muestra tres columnas con el total, los sin teléfono y los con teléfono — la suma de las últimas dos debe igualar el total.

#### Verificación

```sql
-- Verificación de integridad: IS NULL + IS NOT NULL debe sumar el total
SELECT COUNT(*) AS total FROM CLIENTES;
SELECT COUNT(*) AS nulos FROM CLIENTES WHERE TELEFONO IS NULL;
SELECT COUNT(*) AS no_nulos FROM CLIENTES WHERE TELEFONO IS NOT NULL;
-- nulos + no_nulos debe ser igual a total
```

> **Concepto fundamental — NULL en SQL:** `NULL` representa la **ausencia de un valor**, no el valor cero ni una cadena vacía. Por esta razón, ninguna comparación con `=`, `>` o `<` funciona con NULL. La única forma de verificar si un valor es NULL es usando `IS NULL` o `IS NOT NULL`. Este comportamiento es estándar en todos los motores SQL, incluyendo Snowflake.

---

### Paso 8: Ejercicio Integrador — Caso de Análisis Real

**Objetivo:** Aplicar todos los operadores y técnicas aprendidos en un escenario de análisis de negocio realista que combine múltiples tipos de filtros en una sola consulta.

#### Contexto del Escenario

El equipo de marketing necesita identificar clientes prioritarios para una campaña de reactivación. Los criterios son:

1. Clientes de México, Colombia o Chile.
2. Que pertenezcan al segmento Retail o Corporativo.
3. Que tengan email registrado (no nulo).
4. Que se hayan registrado antes del año 2023 (clientes con antigüedad).

#### Instrucciones

1. Construye la consulta paso a paso. Primero escribe el filtro de países:

```sql
-- Paso 1: Solo filtro de países
SELECT ID_CLIENTE, NOMBRE, PAIS, SEGMENTO, EMAIL, FECHA_REGISTRO
FROM CLIENTES
WHERE PAIS IN ('México', 'Colombia', 'Chile');
```

2. Agrega el filtro de segmento:

```sql
-- Paso 2: Países + segmento
SELECT ID_CLIENTE, NOMBRE, PAIS, SEGMENTO, EMAIL, FECHA_REGISTRO
FROM CLIENTES
WHERE PAIS IN ('México', 'Colombia', 'Chile')
  AND SEGMENTO IN ('Retail', 'Corporativo');
```

3. Agrega el filtro de email no nulo:

```sql
-- Paso 3: + email registrado
SELECT ID_CLIENTE, NOMBRE, PAIS, SEGMENTO, EMAIL, FECHA_REGISTRO
FROM CLIENTES
WHERE PAIS IN ('México', 'Colombia', 'Chile')
  AND SEGMENTO IN ('Retail', 'Corporativo')
  AND EMAIL IS NOT NULL;
```

4. Agrega el filtro de antigüedad (registrados antes de 2023):

```sql
-- Consulta final completa: todos los criterios de la campaña
SELECT ID_CLIENTE, NOMBRE, PAIS, SEGMENTO, EMAIL, FECHA_REGISTRO
FROM CLIENTES
WHERE PAIS IN ('México', 'Colombia', 'Chile')
  AND SEGMENTO IN ('Retail', 'Corporativo')
  AND EMAIL IS NOT NULL
  AND FECHA_REGISTRO < '2023-01-01'
ORDER BY PAIS, NOMBRE;
```

#### Resultado Esperado

- Cada versión de la consulta debe devolver menos filas que la anterior, ya que cada condición adicional reduce el conjunto de resultados.
- La consulta final devuelve el listado de clientes que cumplen todos los criterios de la campaña.

> **Buena práctica — Construcción incremental:** Construir consultas complejas paso a paso (como hiciste en este ejercicio) es una práctica profesional recomendada. Te permite verificar cada condición individualmente antes de combinarlas, lo que facilita la detección de errores y la comprensión del impacto de cada filtro.

---

## Limpieza del Entorno

Al finalizar el laboratorio, realiza las siguientes acciones para preservar los créditos de Snowflake y mantener el entorno ordenado.

```sql
-- Paso 1: No es necesario eliminar objetos — este laboratorio solo ejecutó consultas SELECT
-- Las consultas SELECT no modifican datos ni crean objetos permanentes en Snowflake

-- Paso 2: Verifica que no hayas creado objetos accidentalmente
SHOW TABLES IN SCHEMA PUBLIC;
-- La lista debe ser la misma que al inicio del laboratorio

-- Paso 3: Suspende el Virtual Warehouse para detener el consumo de créditos
-- Reemplaza COMPUTE_WH por el nombre de tu warehouse
ALTER WAREHOUSE COMPUTE_WH SUSPEND;
```

> **Importante para cuentas trial:** Las cuentas trial de Snowflake tienen un límite de créditos. Suspender el warehouse al finalizar cada sesión es una buena práctica que extiende la vida útil de tu cuenta. Snowflake también tiene auto-suspensión configurable, pero suspenderlo manualmente asegura que no consumas créditos innecesariamente.

**Checklist de cierre:**

- [ ] Todas las consultas del laboratorio ejecutadas y resultados revisados.
- [ ] Validaciones finales completadas (Sección de Validación).
- [ ] Virtual Warehouse suspendido.
- [ ] Worksheet guardada con un nombre descriptivo (ej. `Lab02_Filtrado_TuNombre`).

---

## Resumen

En este laboratorio aplicaste la cláusula `WHERE` y un conjunto completo de operadores de filtrado para segmentar datos reales de ventas y clientes en Snowflake. Los conceptos clave que practicaste fueron:

| Concepto                  | Operador / Sintaxis                            | Uso Principal                                         |
|---------------------------|------------------------------------------------|-------------------------------------------------------|
| Filtrado básico           | `WHERE columna = valor`                        | Igualdad exacta en texto, número o fecha              |
| Comparación numérica      | `>`, `<`, `>=`, `<=`, `<>`                     | Rangos, umbrales y exclusiones numéricas              |
| Condición compuesta       | `AND`, `OR`, `NOT`                             | Múltiples criterios simultáneos o alternativos        |
| Lista de valores          | `IN (v1, v2, ...)` / `NOT IN`                  | Alternativa limpia a múltiples `OR`                   |
| Búsqueda de patrones      | `LIKE '%patrón%'`, `LIKE 'A_'`, `NOT LIKE`     | Texto parcial, prefijos, sufijos, posiciones exactas  |
| Valores nulos             | `IS NULL` / `IS NOT NULL`                      | Identificar datos faltantes o incompletos             |

**Progresión del laboratorio:** Comenzaste con filtros simples de un solo criterio y terminaste construyendo una consulta de análisis real con cinco condiciones combinadas que integran `IN`, `IS NOT NULL`, comparación de fechas y `NOT LIKE`. Esta progresión refleja el trabajo real de un analista de datos.

**Conexión con el siguiente laboratorio:** En el Lab 03 aprenderás `ORDER BY` para organizar los resultados filtrados, y `BETWEEN` como alternativa a los rangos con `>=` y `<=`. Los filtros `WHERE` que dominaste hoy son la base sobre la que se construyen todas las consultas de análisis más avanzadas del curso.

### Recursos Adicionales

- [Documentación oficial de Snowflake — Cláusula WHERE](https://docs.snowflake.com/en/sql-reference/constructs/where)
- [Documentación de Snowflake — Operadores de comparación](https://docs.snowflake.com/en/sql-reference/operators-comparison)
- [Documentación de Snowflake — Operador LIKE / ILIKE](https://docs.snowflake.com/en/sql-reference/functions/like)
- [Tutorial interactivo de SQL WHERE en W3Schools](https://www.w3schools.com/sql/sql_where.asp)
- [Guía SQL de Mode Analytics — Filtering Data](https://mode.com/sql-tutorial/sql-where/)

---

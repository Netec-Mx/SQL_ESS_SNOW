---LAB_START---
LAB_ID: 07-00-01
---MARKDOWN---
# Integración de datos

## Metadatos

| Atributo         | Detalle                          |
|------------------|----------------------------------|
| **Duración**     | 60 minutos                       |
| **Complejidad**  | Alta                             |
| **Nivel Bloom**  | Aplicar (Apply)                  |
| **Laboratorio**  | 07-00-01                         |
| **Versión**      | 1.0                              |

---

## Descripción General

En este laboratorio explorarás el concepto más poderoso del SQL relacional: la combinación de tablas mediante `JOIN`. Partiendo del esquema relacional del dataset `CURSO_SQL`, aprenderás cómo las tablas `VENTAS`, `CLIENTES`, `PRODUCTOS` y `EMPLEADOS` se conectan entre sí a través de claves primarias y foráneas. Aplicarás `INNER JOIN` para obtener únicamente los registros con coincidencias en ambas tablas, y `LEFT JOIN` para detectar registros sin correspondencia, como clientes que nunca han realizado una compra. El laboratorio culmina con una consulta integradora que combina `JOIN`, `WHERE`, `GROUP BY` y `ORDER BY` en un reporte de análisis real.

---

## Objetivos de Aprendizaje

Al finalizar este laboratorio, serás capaz de:

- [ ] Identificar las claves primarias y foráneas del dataset `CURSO_SQL` y explicar cómo conectan las tablas entre sí.
- [ ] Aplicar `INNER JOIN` para combinar registros de dos o más tablas que tienen valores coincidentes en una columna relacionada.
- [ ] Utilizar `LEFT JOIN` para incluir todos los registros de la tabla izquierda, incluso cuando no tienen coincidencia en la tabla derecha.
- [ ] Emplear alias de tabla para mejorar la legibilidad de consultas con múltiples tablas y resolver ambigüedades de nombres de columna.
- [ ] Construir una consulta integradora que combine `JOIN` con `WHERE`, `GROUP BY` y `ORDER BY` para producir un reporte de análisis de ventas.

---

## Prerrequisitos

### Conocimiento Previo

- Haber completado los Labs 01-00-01 al 06-00-01, o demostrar dominio de:
  - Cláusulas `SELECT`, `FROM`, `WHERE`, `ORDER BY`
  - Funciones de agregación: `COUNT()`, `SUM()`, `AVG()`, `MAX()`, `MIN()`
  - Agrupación con `GROUP BY` y filtrado de grupos con `HAVING`
- Comprensión básica del concepto de clave primaria (PK) y clave foránea (FK) según la lección 7.1.

### Acceso y Configuración

- Cuenta activa en Snowflake (trial o corporativa, edición Standard o superior).
- Acceso a la base de datos `CURSO_SQL` con las tablas: `VENTAS`, `CLIENTES`, `PRODUCTOS`, `EMPLEADOS`.
- Script de inicialización del dataset ejecutado correctamente (provisto por el instructor, v1.0).
- Navegador web actualizado con JavaScript habilitado (Chrome 90+ recomendado).

---

## Entorno de Laboratorio

### Hardware Mínimo Requerido

| Componente            | Mínimo Recomendado                              |
|-----------------------|-------------------------------------------------|
| Conexión a internet   | 10 Mbps estables                               |
| RAM                   | 4 GB (para navegador con múltiples pestañas)   |
| Resolución de pantalla| 1280×768 píxeles                               |
| Sistema operativo     | Windows 10+, macOS 10.15+, o Linux moderno     |

### Software Requerido

| Software              | Versión / Detalle                               |
|-----------------------|-------------------------------------------------|
| Snowflake             | Edición Standard o superior                    |
| Snowsight             | Interfaz web incluida en toda cuenta Snowflake |
| Navegador web         | Chrome 90+, Firefox 88+, Edge 90+, Safari 14+ |
| Dataset del curso     | `CURSO_SQL` v1.0 (provisto por el instructor)  |

---

## Paso 0 — Crear y poblar el dataset **EMPLEADOS** en `CURSO_SQL`

**Objetivo:** Preparar el entorno de trabajo creando las tablas requeridas para el laboratorio.

### Instrucciones

1. Inicia sesión en tu cuenta Snowflake en [app.snowflake.com](https://app.snowflake.com).
2. En el menú lateral izquierdo, haz clic en **Projects → Workspaces**.
3. Selecciona el Workspace llamado **`Setup_CURSO_SQLSNOW`**.
4.  Da clic en **Add new** y crea un nuevo archivo tipo **SQL**
5. Escribe el siguiente nombre del archivo: **`Setup_Lab03_Empleados.sql`**
6. A la derecha selecciona el warehouse **`COMPUTE_WH`** o el warehouse asignado/creado al inicio de la creación de la cuenta.
7. Copia/Pega y ejecuta el siguiente script completo.

> **Nota:** En Snowsight puedes ejecutar todo el bloque completo. Si tu cuenta no permite crear bases de datos, ejecuta el script hasta donde tus permisos lo permitan o solicita apoyo del instructor.

```sql
-- ============================================================
-- Script de inicialización de datos para tabla EMPLEADOS
-- Base de datos: CURSO_SQL
-- Schema: PUBLIC
-- Uso: Ejecutar antes de recrear/cargar la tabla VENTAS
-- Registros generados: 20 empleados
-- ============================================================

USE DATABASE CURSO_SQL;
USE SCHEMA PUBLIC;
USE WAREHOUSE COMPUTE_WH;

-- Crear la tabla EMPLEADOS requerida para relacionar ventas con empleados
CREATE OR REPLACE TABLE EMPLEADOS (
    EMPLEADO_ID     NUMBER(38,0) NOT NULL,
    NOMBRE_EMPLEADO VARCHAR(150),
    REGION          VARCHAR(50),
    CARGO           VARCHAR(100),
    CONSTRAINT PK_EMPLEADOS PRIMARY KEY (EMPLEADO_ID)
);

-- Insertar empleados de ejemplo
INSERT INTO EMPLEADOS (
    EMPLEADO_ID,
    NOMBRE_EMPLEADO,
    REGION,
    CARGO
)
SELECT * FROM VALUES
    (1,  'Laura Méndez',      'Norte',     'Ejecutivo de Ventas'),
    (2,  'Roberto Salazar',   'Sur',       'Ejecutivo de Ventas'),
    (3,  'Patricia Núñez',    'Centro',    'Ejecutivo de Ventas'),
    (4,  'Fernando Rojas',    'Occidente', 'Ejecutivo de Ventas'),
    (5,  'Gabriela Silva',    'Oriente',   'Ejecutivo de Ventas'),
    (6,  'Héctor Molina',     'Norte',     'Gerente Regional'),
    (7,  'Natalia Reyes',     'Sur',       'Gerente Regional'),
    (8,  'Oscar Cabrera',     'Centro',    'Gerente Regional'),
    (9,  'Diana Paredes',     'Occidente', 'Gerente Regional'),
    (10, 'Manuel Ortega',     'Oriente',   'Gerente Regional'),
    (11, 'Claudia Benítez',   'Norte',     'Asesor Comercial'),
    (12, 'Raúl Herrera',      'Sur',       'Asesor Comercial'),
    (13, 'Mónica Fuentes',    'Centro',    'Asesor Comercial'),
    (14, 'Iván Cárdenas',     'Occidente', 'Asesor Comercial'),
    (15, 'Elena Márquez',     'Oriente',   'Asesor Comercial'),
    (16, 'Sergio Pineda',     'Norte',     'Supervisor de Ventas'),
    (17, 'Adriana Solís',     'Sur',       'Supervisor de Ventas'),
    (18, 'Tomás Aguilar',     'Centro',    'Supervisor de Ventas'),
    (19, 'Verónica Ramos',    'Occidente', 'Supervisor de Ventas'),
    (20, 'Luis Del Valle',    'Oriente',   'Supervisor de Ventas')
AS T(EMPLEADO_ID, NOMBRE_EMPLEADO, REGION, CARGO);

-- Validación general
SELECT COUNT(*) AS total_empleados
FROM EMPLEADOS;

SELECT *
FROM EMPLEADOS
ORDER BY EMPLEADO_ID;
```

8. Ahora selecciona el script llamado **Setup_Lab02_Ventas.sql** borra el contenido del archivo.
9. Copia y pega el siguiente contenido para actualizar la realción **Ventas/Empleados**
10. Selecciona todo el contenido y ejecutalo.

```sql
-- ============================================================
-- LAB 02 - Script de inicialización de datos para tabla VENTAS
-- Base de datos: CURSO_SQL
-- Schema: PUBLIC
-- Requisitos:
--   1. La tabla CLIENTES debe existir previamente
--   2. La tabla PRODUCTOS debe existir previamente
--   3. La tabla EMPLEADOS debe existir previamente
-- Registros generados: 1,200 ventas
-- Ajuste aplicado:
--   - Se agrega EMPLEADO_ID para permitir JOIN con EMPLEADOS
--   - Se mantiene ID_PRODUCTO para permitir JOIN con PRODUCTOS
--   - PRODUCTO, CATEGORIA y PRECIO_UNITARIO se toman desde PRODUCTOS
-- ============================================================

USE DATABASE CURSO_SQL;
USE SCHEMA PUBLIC;
USE WAREHOUSE COMPUTE_WH;

-- Verificación previa: confirmar que las tablas requeridas existen y tienen datos
SELECT COUNT(*) AS total_clientes
FROM CLIENTES;

SELECT COUNT(*) AS total_productos
FROM PRODUCTOS;

SELECT COUNT(*) AS total_empleados
FROM EMPLEADOS;

-- Crear la tabla VENTAS con las columnas necesarias para los JOINs
CREATE OR REPLACE TABLE VENTAS (
    ID_VENTA        NUMBER(38,0) NOT NULL,
    ID_CLIENTE      NUMBER(38,0),
    ID_PRODUCTO     NUMBER(38,0),
    EMPLEADO_ID     NUMBER(38,0),
    PRODUCTO        VARCHAR(200),
    CATEGORIA       VARCHAR(100),
    CANTIDAD        NUMBER(38,0),
    PRECIO_UNITARIO NUMBER(10,2),
    TOTAL_VENTA     NUMBER(10,2),
    FECHA_VENTA     DATE,
    REGION          VARCHAR(50),
    ESTADO_ENVIO    VARCHAR(50),
    CONSTRAINT PK_VENTAS PRIMARY KEY (ID_VENTA)
);

-- Insertar 1,200 registros sintéticos de ventas
-- CLIENTES, PRODUCTOS y EMPLEADOS se asignan de forma cíclica para mantener relaciones válidas
INSERT INTO VENTAS (
    ID_VENTA,
    ID_CLIENTE,
    ID_PRODUCTO,
    EMPLEADO_ID,
    PRODUCTO,
    CATEGORIA,
    CANTIDAD,
    PRECIO_UNITARIO,
    TOTAL_VENTA,
    FECHA_VENTA,
    REGION,
    ESTADO_ENVIO
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
empleados_ordenados AS (
    SELECT
        EMPLEADO_ID,
        REGION,
        ROW_NUMBER() OVER (ORDER BY EMPLEADO_ID) AS rn
    FROM EMPLEADOS
),
total_empleados AS (
    SELECT COUNT(*) AS total
    FROM empleados_ordenados
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
        e.EMPLEADO_ID,
        p.NOMBRE_PRODUCTO AS PRODUCTO,
        p.CATEGORIA,
        MOD(b.n, 5) + 1 AS CANTIDAD,
        p.PRECIO AS PRECIO_UNITARIO,
        DATEADD(DAY, MOD(b.n * 3, 1095), DATE '2022-01-01') AS FECHA_VENTA,
        e.REGION,
        CASE
            WHEN MOD(b.n, 11) = 0 THEN NULL
            WHEN MOD(b.n, 4) = 0 THEN 'Pendiente'
            WHEN MOD(b.n, 4) = 1 THEN 'Enviado'
            WHEN MOD(b.n, 4) = 2 THEN 'Entregado'
            ELSE 'Cancelado'
        END AS ESTADO_ENVIO
    FROM base b
    CROSS JOIN total_clientes tc
    CROSS JOIN total_productos tp
    CROSS JOIN total_empleados te
    JOIN clientes_ordenados c
      ON c.rn = MOD(b.n - 1, tc.total) + 1
    JOIN productos_ordenados p
      ON p.rn = MOD(b.n - 1, tp.total) + 1
    JOIN empleados_ordenados e
      ON e.rn = MOD(b.n - 1, te.total) + 1
)
SELECT
    ID_VENTA,
    ID_CLIENTE,
    ID_PRODUCTO,
    EMPLEADO_ID,
    PRODUCTO,
    CATEGORIA,
    CANTIDAD,
    PRECIO_UNITARIO,
    CAST(CANTIDAD * PRECIO_UNITARIO AS NUMBER(10,2)) AS TOTAL_VENTA,
    FECHA_VENTA,
    REGION,
    ESTADO_ENVIO
FROM ventas_base;

-- Validación general de carga
SELECT COUNT(*) AS total_ventas
FROM VENTAS;

-- Validación de integridad: todas las ventas deben tener cliente, producto y empleado relacionados
SELECT COUNT(*) AS ventas_sin_cliente_relacionado
FROM VENTAS V
LEFT JOIN CLIENTES C
    ON V.ID_CLIENTE = C.ID_CLIENTE
WHERE C.ID_CLIENTE IS NULL;

SELECT COUNT(*) AS ventas_sin_producto_relacionado
FROM VENTAS V
LEFT JOIN PRODUCTOS P
    ON V.ID_PRODUCTO = P.ID_PRODUCTO
WHERE P.ID_PRODUCTO IS NULL;

SELECT COUNT(*) AS ventas_sin_empleado_relacionado
FROM VENTAS V
LEFT JOIN EMPLEADOS E
    ON V.EMPLEADO_ID = E.EMPLEADO_ID
WHERE E.EMPLEADO_ID IS NULL;

-- Validación de valores necesarios para la práctica
SELECT DISTINCT REGION
FROM VENTAS
ORDER BY REGION;

SELECT DISTINCT CATEGORIA
FROM VENTAS
ORDER BY CATEGORIA;

SELECT
    COUNT(*) AS ventas_sin_estado_envio
FROM VENTAS
WHERE ESTADO_ENVIO IS NULL;

-- Prueba de JOIN con PRODUCTOS: ingreso promedio por categoría
SELECT
    P.CATEGORIA,
    COUNT(*)                        AS total_ventas,
    SUM(V.TOTAL_VENTA)              AS ingreso_total,
    ROUND(AVG(V.TOTAL_VENTA), 2)    AS ingreso_promedio,
    MIN(V.TOTAL_VENTA)              AS ingreso_minimo,
    MAX(V.TOTAL_VENTA)              AS ingreso_maximo
FROM VENTAS V
    JOIN PRODUCTOS P ON V.ID_PRODUCTO = P.ID_PRODUCTO
GROUP BY P.CATEGORIA
ORDER BY ingreso_total DESC;

-- Prueba de JOIN con EMPLEADOS: ventas por empleado
SELECT
    E.NOMBRE_EMPLEADO,
    E.REGION,
    E.CARGO,
    COUNT(*) AS total_ventas,
    SUM(V.TOTAL_VENTA) AS ingreso_total
FROM VENTAS V
    JOIN EMPLEADOS E ON V.EMPLEADO_ID = E.EMPLEADO_ID
GROUP BY
    E.NOMBRE_EMPLEADO,
    E.REGION,
    E.CARGO
ORDER BY ingreso_total DESC;

-- Muestra final
SELECT *
FROM VENTAS
ORDER BY ID_VENTA
LIMIT 10;
```

#### Resultado Esperado

Al finalizar la ejecución del script, la tabla `EMPLEADOS` queda disponible con datos suficientes para completar todos los pasos de la práctica y la tabla **VENTAS** quedara ajustada para las consultas.

---

### Configuración Inicial del Entorno

Antes de comenzar los ejercicios, ejecuta los siguientes comandos en un nuevo Worksheet de Snowsight para asegurarte de estar en el contexto correcto:

1. Clic en la sección **Projects** y luego clic en **Workspaces**.
2. Abre tu **Workspace** llamado **SnowEssLAbs**.
3. Crea un archivo de tipo SQL llamado **`Lab07_Integracion`**.
4. Ejecuta los siguientes comandos para asegurarte de estar trabajando en el contexto correcto:

```sql
-- 1. Seleccionar el rol de trabajo
USE ROLE SYSADMIN;

-- 2. Seleccionar el Virtual Warehouse (tamaño X-Small para minimizar créditos)
USE WAREHOUSE COMPUTE_WH;

-- 3. Seleccionar la base de datos del curso
USE DATABASE CURSO_SQL;

-- 4. Seleccionar el schema correspondiente
USE SCHEMA PUBLIC;

-- 5. Verificar que las tablas del laboratorio están disponibles
SHOW TABLES;
```

**Resultado esperado de `SHOW TABLES`:** Debes ver al menos las tablas `VENTAS`, `CLIENTES`, `PRODUCTOS` y `EMPLEADOS` en el listado.

> ⚠️ **Nota sobre créditos:** Utiliza siempre un Virtual Warehouse de tamaño **X-Small**. Apaga el warehouse al finalizar la sesión con `ALTER WAREHOUSE COMPUTE_WH SUSPEND;`.

---

## Instrucciones Paso a Paso

---

### Paso 1: Explorar el Esquema Relacional del Dataset

**Objetivo:** Comprender cómo están relacionadas las cuatro tablas del dataset antes de escribir cualquier `JOIN`.

#### Instrucciones

**1.1.** Examina la estructura de la tabla `VENTAS` para identificar sus columnas y claves:

```sql
-- Explorar la estructura de la tabla VENTAS
DESCRIBE TABLE VENTAS;
```

**1.2.** Repite el proceso para las demás tablas:

```sql
-- Explorar la estructura de CLIENTES
DESCRIBE TABLE CLIENTES;

-- Explorar la estructura de PRODUCTOS
DESCRIBE TABLE PRODUCTOS;

-- Explorar la estructura de EMPLEADOS
DESCRIBE TABLE EMPLEADOS;
```

**1.3.** Visualiza una muestra de datos de cada tabla para familiarizarte con el contenido:

```sql
-- Muestra de VENTAS
SELECT * FROM VENTAS LIMIT 5;

-- Muestra de CLIENTES
SELECT * FROM CLIENTES LIMIT 5;

-- Muestra de PRODUCTOS
SELECT * FROM PRODUCTOS LIMIT 5;

-- Muestra de EMPLEADOS
SELECT * FROM EMPLEADOS LIMIT 5;
```

**1.4.** Basándote en los resultados, completa mentalmente (o en papel) el siguiente diagrama de relaciones:

```
CLIENTES                    VENTAS                     PRODUCTOS
────────────────            ──────────────────         ─────────────────
ID_CLIENTE (PK)  ◄────────  ID_CLIENTE (FK)            ID_PRODUCTO (PK)
NOMBRE                      ID_VENTA (PK)    ─────────► ID_PRODUCTO (FK)  [*ver nota]
CORREO                      ID_PRODUCTO (FK)            NOMBRE_PRODUCTO
CIUDAD                      EMPLEADO_ID (FK) ──┐        CATEGORIA
SEGMENTO                    FECHA_VENTA        │        PRECIO_UNITARIO
                            CANTIDAD           │
                            TOTAL_VENTA        │        EMPLEADOS
                                               └──────► EMPLEADO_ID (PK)
                                                        NOMBRE_EMPLEADO
                                                        REGION
                                                        CARGO
```

> 📝 **Nota:** La columna exacta que conecta `VENTAS` con `PRODUCTOS` puede llamarse `ID_PRODUCTO` en ambas tablas. Ajusta el diagrama según lo que observes en tu `DESCRIBE TABLE`.

#### Resultado Esperado

- La tabla `VENTAS` contiene claves foráneas (`ID_CLIENTE`, `ID_PRODUCTO`, `EMPLEADO_ID`) que referencian las claves primarias de `CLIENTES`, `PRODUCTOS` y `EMPLEADOS` respectivamente.
- Cada venta pertenece a exactamente un cliente, un producto y un empleado (relaciones **uno a muchos**).

#### Verificación

Ejecuta el siguiente conteo para confirmar que las tablas tienen datos:

```sql
SELECT
    'VENTAS'    AS tabla, COUNT(*) AS total_registros FROM VENTAS
UNION ALL
SELECT 'CLIENTES',  COUNT(*) FROM CLIENTES
UNION ALL
SELECT 'PRODUCTOS', COUNT(*) FROM PRODUCTOS
UNION ALL
SELECT 'EMPLEADOS', COUNT(*) FROM EMPLEADOS;
```

Debes obtener cuatro filas, cada una con un conteo mayor a cero.

---

### Paso 2: Tu Primer INNER JOIN — Ventas con Información del Cliente

**Objetivo:** Aplicar `INNER JOIN` para combinar la tabla `VENTAS` con la tabla `CLIENTES` y obtener un reporte enriquecido con el nombre del cliente en cada venta.

#### Instrucciones

**2.1.** Escribe la consulta más básica de `INNER JOIN` entre `VENTAS` y `CLIENTES`:

```sql
-- INNER JOIN básico: VENTAS con CLIENTES
-- Usamos alias de tabla: v para VENTAS, c para CLIENTES
SELECT
    v.ID_VENTA,
    v.FECHA_VENTA,
    v.TOTAL_VENTA,
    c.NOMBRE,
    c.CIUDAD
FROM VENTAS AS v
INNER JOIN CLIENTES AS c
    ON v.ID_CLIENTE = c.ID_CLIENTE
LIMIT 10;
```

**2.2.** Observa el resultado. Nota que:

- Cada fila de `VENTAS` ahora muestra el nombre y ciudad del cliente correspondiente.
- Solo aparecen ventas que tienen un `ID_CLIENTE` que existe en la tabla `CLIENTES`.
- Los alias `v` y `c` permiten escribir `v.ID_CLIENTE` en lugar de `VENTAS.ID_VENTA`.

**2.3.** Amplía la consulta para incluir más columnas útiles y ordenar el resultado:

```sql
-- INNER JOIN enriquecido con más columnas y ordenamiento
SELECT
    v.ID_VENTA,
    v.FECHA_VENTA,
    c.NOMBRE          AS nombre_cliente,
    c.CIUDAD          AS ciudad_cliente,
    c.SEGMENTO        AS segmento_cliente,
    v.CANTIDAD,
    v.TOTAL_VENTA
FROM VENTAS AS v
INNER JOIN CLIENTES AS c
    ON v.ID_CLIENTE = c.ID_CLIENTE
ORDER BY v.FECHA_VENTA DESC
LIMIT 20;
```

**2.4.** Ahora prueba qué ocurre si **no usas alias** y hay columnas con el mismo nombre en ambas tablas. Ejecuta la siguiente consulta y observa el error:

```sql
-- ¿Qué pasa sin alias cuando hay columnas con el mismo nombre?
-- Esta consulta genera ambigüedad en ID_CLIENTE
SELECT
    ID_VENTA,
    FECHA_VENTA,
    ID_CLIENTE,       -- ← AMBIGUO: ¿de cuál tabla?
    NOMBRE,
    TOTAL_VENTA
FROM VENTAS
INNER JOIN CLIENTES
    ON VENTAS.ID_CLIENTE = CLIENTES.ID_CLIENTE
LIMIT 5;
```

> 💡 **Punto de aprendizaje:** Snowflake generará un error de ambigüedad en `ID_CLIENTE` porque esa columna existe en ambas tablas. La solución es siempre usar la notación `tabla.columna` o, mejor aún, **alias de tabla**.

**2.5.** Corrige la consulta anterior usando alias:

```sql
-- Versión corregida con alias para resolver ambigüedad
SELECT
    v.ID_VENTA,
    v.FECHA_VENTA,
    v.ID_CLIENTE,     -- Ahora es explícito: viene de VENTAS
    c.NOMBRE,
    v.TOTAL_VENTA
FROM VENTAS AS v
INNER JOIN CLIENTES AS c
    ON v.ID_CLIENTE = c.ID_CLIENTE
LIMIT 5;
```

#### Resultado Esperado

La consulta del paso 2.3 debe devolver filas como las siguientes (los valores exactos dependen de tu dataset):

| ID_VENTA | FECHA_VENTA | NOMBRE_CLIENTE | CIUDAD_CLIENTE | SEGMENTO_CLIENTE | CANTIDAD | TOTAL_VENTA |
|----------|-------------|----------------|----------------|------------------|----------|-------------|
| 1045     | 2024-03-15  | Ana Torres     | Ciudad de México| Corporativo     | 3        | 4500.00     |
| 1044     | 2024-03-14  | Luis Ramos     | Guadalajara    | Minorista        | 1        | 890.00      |
| ...      | ...         | ...            | ...            | ...              | ...      | ...         |

#### Verificación

Verifica que el `INNER JOIN` solo devuelve registros con coincidencia en ambas tablas:

```sql
-- El total de filas del JOIN no debe superar el total de VENTAS
SELECT COUNT(*) AS total_ventas_con_cliente
FROM VENTAS AS v
INNER JOIN CLIENTES AS c
    ON v.ID_CLIENTE = c.ID_CLIENTE;

-- Compara con el total de VENTAS
SELECT COUNT(*) AS total_ventas FROM VENTAS;
```

Si ambos conteos son iguales, todos los registros de `VENTAS` tienen un cliente correspondiente en `CLIENTES`. Si el JOIN devuelve menos, existen ventas con `ID_CLIENTE` sin coincidencia.

---

### Paso 3: JOIN con Tres Tablas — Reporte Enriquecido de Ventas

**Objetivo:** Combinar tres tablas simultáneamente (`VENTAS`, `CLIENTES`, `PRODUCTOS`) para construir un reporte de ventas completo con información de cliente y producto en cada fila.

#### Instrucciones

**3.1.** Comienza entendiendo la lógica de encadenamiento: cada `JOIN` adicional se apoya en la tabla base o en una tabla ya unida:

```sql
-- JOIN de tres tablas: VENTAS + CLIENTES + PRODUCTOS
SELECT
    v.ID_VENTA,
    v.FECHA_VENTA,
    c.NOMBRE          AS nombre_cliente,
    c.CIUDAD          AS ciudad_cliente,
    p.NOMBRE_PRODUCTO,
    p.CATEGORIA,
    v.CANTIDAD,
    p.PRECIO,
    v.TOTAL_VENTA
FROM VENTAS AS v
INNER JOIN CLIENTES AS c
    ON v.ID_CLIENTE = c.ID_CLIENTE
INNER JOIN PRODUCTOS AS p
    ON v.ID_PRODUCTO = p.ID_PRODUCTO
ORDER BY v.FECHA_VENTA DESC
LIMIT 15;
```

**3.2.** Agrega también la información del empleado que realizó la venta:

```sql
-- JOIN de cuatro tablas: VENTAS + CLIENTES + PRODUCTOS + EMPLEADOS
SELECT
    v.ID_VENTA,
    v.FECHA_VENTA,
    c.NOMBRE          AS nombre_cliente,
    c.CIUDAD          AS ciudad_cliente,
    p.NOMBRE_PRODUCTO,
    p.CATEGORIA,
    e.NOMBRE_EMPLEADO AS vendedor,
    e.REGION          AS region_vendedor,
    v.CANTIDAD,
    v.TOTAL_VENTA
FROM VENTAS AS v
INNER JOIN CLIENTES AS c
    ON v.ID_CLIENTE = c.ID_CLIENTE
INNER JOIN PRODUCTOS AS p
    ON v.ID_PRODUCTO = p.ID_PRODUCTO
INNER JOIN EMPLEADOS AS e
    ON v.EMPLEADO_ID = e.EMPLEADO_ID
ORDER BY v.TOTAL_VENTA DESC
LIMIT 20;
```

**3.3.** Aplica un filtro `WHERE` para enfocarte en una categoría de producto específica. Ajusta `'Electrónica'` según las categorías reales de tu dataset (puedes verificarlas con `SELECT DISTINCT CATEGORIA FROM PRODUCTOS`):

```sql
-- JOIN de cuatro tablas filtrado por categoría de producto
SELECT
    v.ID_VENTA,
    v.FECHA_VENTA,
    c.NOMBRE          AS nombre_cliente,
    p.NOMBRE_PRODUCTO,
    p.CATEGORIA,
    e.NOMBRE_EMPLEADO AS vendedor,
    v.TOTAL_VENTA
FROM VENTAS AS v
INNER JOIN CLIENTES AS c
    ON v.ID_CLIENTE = c.ID_CLIENTE
INNER JOIN PRODUCTOS AS p
    ON v.ID_PRODUCTO = p.ID_PRODUCTO
INNER JOIN EMPLEADOS AS e
    ON v.EMPLEADO_ID = e.EMPLEADO_ID
WHERE p.CATEGORIA = 'Electrónica'   -- Ajusta según tu dataset
ORDER BY v.TOTAL_VENTA DESC;
```

> 💡 **Buena práctica:** En consultas con múltiples tablas, escribe primero todos los `JOIN` y luego el `WHERE`. Esto hace que la lógica de unión sea fácil de leer de forma separada a la lógica de filtrado.

#### Resultado Esperado

La consulta del paso 3.2 debe producir un reporte donde cada fila representa una venta completa con datos del cliente, producto y empleado. No debe haber columnas con valores `NULL` (porque `INNER JOIN` solo incluye coincidencias completas).

#### Verificación

```sql
-- Verificar que no hay NULLs en columnas clave del resultado
SELECT
    COUNT(*)                              AS total_filas,
    COUNT(c.NOMBRE)                       AS filas_con_cliente,
    COUNT(p.NOMBRE_PRODUCTO)              AS filas_con_producto,
    COUNT(e.NOMBRE_EMPLEADO)              AS filas_con_empleado
FROM VENTAS AS v
INNER JOIN CLIENTES AS c
    ON v.ID_CLIENTE = c.ID_CLIENTE
INNER JOIN PRODUCTOS AS p
    ON v.ID_PRODUCTO = p.ID_PRODUCTO
INNER JOIN EMPLEADOS AS e
    ON v.EMPLEADO_ID = e.EMPLEADO_ID;
```

Los cuatro valores deben ser iguales. Si `COUNT(c.NOMBRE)` es menor que `COUNT(*)`, hay valores `NULL` inesperados.

---

### Paso 4: LEFT JOIN — Encontrar Clientes sin Compras

**Objetivo:** Utilizar `LEFT JOIN` para identificar clientes registrados que nunca han realizado una venta, demostrando la diferencia clave con `INNER JOIN`.

#### Instrucciones

**4.1.** Primero, ejecuta un `INNER JOIN` entre `CLIENTES` y `VENTAS` para ver cuántos clientes tienen al menos una venta:

```sql
-- INNER JOIN: solo clientes que tienen ventas
SELECT
    c.ID_CLIENTE,
    c.NOMBRE,
    c.CIUDAD,
    COUNT(v.ID_VENTA) AS total_ventas
FROM CLIENTES AS c
INNER JOIN VENTAS AS v
    ON c.ID_CLIENTE = v.ID_CLIENTE
GROUP BY c.ID_CLIENTE, c.NOMBRE, c.CIUDAD
ORDER BY total_ventas DESC;
```

**4.2.** Ahora ejecuta el mismo query pero con `LEFT JOIN` y observa la diferencia:

```sql
-- LEFT JOIN: TODOS los clientes, con o sin ventas
SELECT
    c.ID_CLIENTE,
    c.NOMBRE,
    c.CIUDAD,
    COUNT(v.ID_VENTA) AS total_ventas
FROM CLIENTES AS c
LEFT JOIN VENTAS AS v
    ON c.ID_CLIENTE = v.ID_CLIENTE
GROUP BY c.ID_CLIENTE, c.NOMBRE, c.CIUDAD
ORDER BY total_ventas ASC;   -- Los clientes sin ventas aparecen primero (total = 0)
```

**4.3.** Filtra específicamente los clientes que **nunca han comprado** usando `WHERE` después del `LEFT JOIN`:

```sql
-- Clientes sin ninguna venta registrada
-- La clave: cuando no hay coincidencia en VENTAS, ID_VENTA es NULL
SELECT
    c.ID_CLIENTE,
    c.NOMBRE,
    c.EMAIL,
    c.CIUDAD,
    c.SEGMENTO
FROM CLIENTES AS c
LEFT JOIN VENTAS AS v
    ON c.ID_CLIENTE = v.ID_CLIENTE
WHERE v.ID_VENTA IS NULL   -- Solo los que NO tienen coincidencia en VENTAS
ORDER BY c.NOMBRE;
```

> 💡 **Concepto clave:** Cuando un registro de la tabla izquierda (`CLIENTES`) no tiene coincidencia en la tabla derecha (`VENTAS`), el `LEFT JOIN` rellena todas las columnas de `VENTAS` con `NULL`. Por eso `WHERE v.ID_VENTA IS NULL` es el patrón estándar para encontrar registros sin correspondencia.

**4.4.** Compara los resultados de `INNER JOIN` vs `LEFT JOIN` con un conteo:

```sql
-- Comparación de resultados INNER JOIN vs LEFT JOIN
SELECT 'INNER JOIN' AS tipo_join, COUNT(DISTINCT c.ID_CLIENTE) AS clientes_incluidos
FROM CLIENTES AS c
INNER JOIN VENTAS AS v ON c.ID_CLIENTE = v.ID_CLIENTE

UNION ALL

SELECT 'LEFT JOIN (todos)', COUNT(DISTINCT c.ID_CLIENTE)
FROM CLIENTES AS c
LEFT JOIN VENTAS AS v ON c.ID_CLIENTE = v.ID_CLIENTE

UNION ALL

SELECT 'Sin ventas (LEFT JOIN + IS NULL)', COUNT(DISTINCT c.ID_CLIENTE)
FROM CLIENTES AS c
LEFT JOIN VENTAS AS v ON c.ID_CLIENTE = v.ID_CLIENTE
WHERE v.ID_VENTA IS NULL;
```

#### Resultado Esperado

| TIPO_JOIN                          | CLIENTES_INCLUIDOS |
|------------------------------------|--------------------|
| INNER JOIN                         | (número menor)     |
| LEFT JOIN (todos)                  | (total clientes)   |
| Sin ventas (LEFT JOIN + IS NULL)   | (diferencia)       |

La segunda fila debe coincidir con `SELECT COUNT(*) FROM CLIENTES`. La suma de la primera y la tercera fila debe igualar la segunda.

#### Verificación

```sql
-- Verificación matemática: INNER + sin_ventas = total_clientes
SELECT COUNT(*) AS total_clientes_en_tabla FROM CLIENTES;
```

---

### Paso 5: Diferencia Conceptual entre INNER JOIN y LEFT JOIN

**Objetivo:** Consolidar la comprensión de cuándo usar cada tipo de `JOIN` mediante un ejercicio comparativo directo.

#### Instrucciones

**5.1.** Ejecuta ambas versiones de la misma consulta y compara los resultados:

```sql
-- Versión A: INNER JOIN
-- Pregunta: ¿Cuánto ha comprado cada cliente?
-- Resultado: Solo clientes que han comprado algo
SELECT
    c.NOMBRE          AS nombre_cliente,
    c.SEGMENTO,
    SUM(v.TOTAL_VENTA) AS total_compras,
    COUNT(v.ID_VENTA)  AS numero_ventas
FROM CLIENTES AS c
INNER JOIN VENTAS AS v
    ON c.ID_CLIENTE = v.ID_CLIENTE
GROUP BY c.NOMBRE, c.SEGMENTO
ORDER BY total_compras DESC;
```

```sql
-- Versión B: LEFT JOIN
-- Pregunta: ¿Cuánto ha comprado cada cliente (incluyendo los que no han comprado)?
-- Resultado: Todos los clientes; los sin compras muestran NULL o 0
SELECT
    c.NOMBRE              AS nombre_cliente,
    c.SEGMENTO,
    COALESCE(SUM(v.TOTAL_VENTA), 0) AS total_compras,
    COUNT(v.ID_VENTA)                AS numero_ventas
FROM CLIENTES AS c
LEFT JOIN VENTAS AS v
    ON c.ID_CLIENTE = v.ID_CLIENTE
GROUP BY c.NOMBRE, c.SEGMENTO
ORDER BY total_compras DESC;
```

> 💡 **`COALESCE(valor, 0)`** reemplaza `NULL` por `0`, lo que hace el reporte más legible cuando un cliente no tiene ventas.

**5.2.** Observa la diferencia en el número de filas devueltas por cada consulta. Anota tu observación.

**5.3.** Reflexiona sobre cuándo usarías cada uno:

| Escenario de análisis                                          | JOIN recomendado |
|----------------------------------------------------------------|-----------------|
| Reporte de ventas (solo necesito ventas reales)                | `INNER JOIN`    |
| Lista completa de clientes con su actividad de compra          | `LEFT JOIN`     |
| Clientes inactivos que no han comprado en el último mes        | `LEFT JOIN` + `IS NULL` |
| Productos vendidos con detalle de categoría                    | `INNER JOIN`    |
| Empleados que no han registrado ninguna venta                  | `LEFT JOIN` + `IS NULL` |

#### Resultado Esperado

- La consulta con `INNER JOIN` devuelve menos filas que la de `LEFT JOIN`.
- En la consulta `LEFT JOIN`, los clientes sin ventas aparecen con `total_compras = 0` y `numero_ventas = 0`.

#### Verificación

```sql
-- Los clientes con total_compras = 0 en LEFT JOIN deben coincidir
-- con los clientes identificados en el paso 4.3
SELECT COUNT(*) AS clientes_sin_ventas
FROM CLIENTES AS c
LEFT JOIN VENTAS AS v ON c.ID_CLIENTE = v.ID_CLIENTE
WHERE v.ID_VENTA IS NULL;
```

---

### Paso 6: Consulta Integradora — Reporte de Ventas por Empleado y Categoría

**Objetivo:** Construir una consulta compleja que combine `JOIN` con `WHERE`, `GROUP BY` y `ORDER BY` para producir un reporte de análisis de ventas por empleado y categoría de producto.

#### Instrucciones

**6.1.** Construye el reporte paso a paso. Primero, une las cuatro tablas:

```sql
-- Paso base: unir las cuatro tablas
SELECT
    e.NOMBRE_EMPLEADO,
    e.REGION,
    p.CATEGORIA,
    v.TOTAL_VENTA,
    v.FECHA_VENTA
FROM VENTAS AS v
INNER JOIN CLIENTES AS c   ON v.ID_CLIENTE  = c.ID_CLIENTE
INNER JOIN PRODUCTOS AS p  ON v.ID_PRODUCTO = p.ID_PRODUCTO
INNER JOIN EMPLEADOS AS e  ON v.EMPLEADO_ID = e.EMPLEADO_ID
LIMIT 10;
```

**6.2.** Agrega agrupación para obtener totales por empleado y categoría:

```sql
-- Reporte: ventas totales por empleado y categoría de producto
SELECT
    e.NOMBRE_EMPLEADO                     AS vendedor,
    e.REGION,
    p.CATEGORIA,
    COUNT(v.ID_VENTA)                     AS numero_ventas,
    SUM(v.TOTAL_VENTA)                    AS ventas_totales,
    ROUND(AVG(v.TOTAL_VENTA), 2)          AS ticket_promedio
FROM VENTAS AS v
INNER JOIN CLIENTES AS c   ON v.ID_CLIENTE  = c.ID_CLIENTE
INNER JOIN PRODUCTOS AS p  ON v.ID_PRODUCTO = p.ID_PRODUCTO
INNER JOIN EMPLEADOS AS e  ON v.EMPLEADO_ID = e.EMPLEADO_ID
GROUP BY e.NOMBRE_EMPLEADO, e.REGION, p.CATEGORIA
ORDER BY ventas_totales DESC;
```

**6.3.** Agrega un filtro `WHERE` para analizar solo un año o período específico. Ajusta el año según el rango de fechas de tu dataset:

```sql
-- Reporte filtrado por año con HAVING para mostrar solo grupos relevantes
SELECT
    e.NOMBRE_EMPLEADO                     AS vendedor,
    e.REGION,
    p.CATEGORIA,
    COUNT(v.ID_VENTA)                     AS numero_ventas,
    SUM(v.TOTAL_VENTA)                    AS ventas_totales,
    ROUND(AVG(v.TOTAL_VENTA), 2)          AS ticket_promedio
FROM VENTAS AS v
INNER JOIN CLIENTES AS c   ON v.ID_CLIENTE  = c.ID_CLIENTE
INNER JOIN PRODUCTOS AS p  ON v.ID_PRODUCTO = p.ID_PRODUCTO
INNER JOIN EMPLEADOS AS e  ON v.EMPLEADO_ID = e.EMPLEADO_ID
WHERE YEAR(v.FECHA_VENTA) = 2024          -- Ajusta el año según tu dataset
GROUP BY e.NOMBRE_EMPLEADO, e.REGION, p.CATEGORIA
HAVING SUM(v.TOTAL_VENTA) > 1000          -- Solo grupos con ventas significativas
ORDER BY e.REGION, ventas_totales DESC;
```

**6.4.** Construye el reporte final: ranking de los 5 empleados con mayor volumen de ventas, incluyendo el número de clientes únicos atendidos:

```sql
-- Reporte final: Top 5 empleados por ventas totales con clientes únicos
SELECT
    e.NOMBRE_EMPLEADO                          AS vendedor,
    e.REGION,
    e.CARGO,
    COUNT(DISTINCT v.ID_CLIENTE)               AS clientes_unicos,
    COUNT(v.ID_VENTA)                          AS numero_ventas,
    SUM(v.TOTAL_VENTA)                         AS ventas_totales,
    ROUND(AVG(v.TOTAL_VENTA), 2)               AS ticket_promedio,
    MAX(v.TOTAL_VENTA)                         AS venta_maxima
FROM VENTAS AS v
INNER JOIN CLIENTES AS c   ON v.ID_CLIENTE  = c.ID_CLIENTE
INNER JOIN PRODUCTOS AS p  ON v.ID_PRODUCTO = p.ID_PRODUCTO
INNER JOIN EMPLEADOS AS e  ON v.EMPLEADO_ID = e.EMPLEADO_ID
GROUP BY e.NOMBRE_EMPLEADO, e.REGION, e.CARGO
ORDER BY ventas_totales DESC
LIMIT 5;
```

#### Resultado Esperado

El reporte del paso 6.4 debe mostrar una tabla con los 5 empleados más productivos, con columnas que resumen su actividad de ventas. Ejemplo de estructura:

| VENDEDOR        | REGION  | CARGO          | CLIENTES_UNICOS | NUMERO_VENTAS | VENTAS_TOTALES | TICKET_PROMEDIO | VENTA_MAXIMA |
|-----------------|---------|----------------|-----------------|---------------|----------------|-----------------|--------------|
| Carlos Méndez   | Norte   | Senior         | 45              | 78            | 234,500.00     | 3,006.41        | 18,900.00    |
| Laura Jiménez   | Centro  | Senior         | 38              | 65            | 198,300.00     | 3,050.77        | 22,100.00    |
| ...             | ...     | ...            | ...             | ...           | ...            | ...             | ...          |

#### Verificación

```sql
-- Verificar que el total de ventas del reporte
-- coincide con el total general de la tabla VENTAS
SELECT SUM(TOTAL_VENTA) AS gran_total FROM VENTAS;

-- Comparar con la suma del reporte (sin LIMIT)
SELECT SUM(ventas_totales) AS total_en_reporte
FROM (
    SELECT
        e.NOMBRE_EMPLEADO,
        SUM(v.TOTAL_VENTA) AS ventas_totales
    FROM VENTAS AS v
    INNER JOIN EMPLEADOS AS e ON v.EMPLEADO_ID = e.EMPLEADO_ID
    GROUP BY e.NOMBRE_EMPLEADO
) sub;
```

Ambos valores deben ser iguales (o muy cercanos si hay ventas sin empleado asignado).

---

### Paso 7: Ejercicio de Síntesis — Análisis por Segmento de Cliente

**Objetivo:** Aplicar de forma autónoma todos los conceptos del laboratorio en un análisis orientado al negocio.

#### Instrucciones

**7.1.** Escribe una consulta que responda la siguiente pregunta de negocio:

> *"¿Cuál es el desempeño de ventas por segmento de cliente y categoría de producto? Muestra los segmentos con mayor gasto promedio por transacción."*

Requisitos de la consulta:
- Unir `VENTAS`, `CLIENTES` y `PRODUCTOS` (mínimo tres tablas).
- Agrupar por `SEGMENTO` (de CLIENTES) y `CATEGORIA` (de PRODUCTOS).
- Calcular: número de ventas, suma total, promedio por venta y número de clientes únicos.
- Ordenar por promedio de venta descendente.
- Usar alias de tabla en todas las referencias de columna.

```sql
-- Tu consulta aquí:
-- Pista: la estructura base es similar al paso 6.2,
-- pero cambia las columnas de GROUP BY y ORDER BY.

SELECT
    c.SEGMENTO,
    p.CATEGORIA,
    COUNT(DISTINCT c.ID_CLIENTE)    AS clientes_unicos,
    COUNT(v.ID_VENTA)               AS numero_ventas,
    SUM(v.TOTAL_VENTA)              AS ventas_totales,
    ROUND(AVG(v.TOTAL_VENTA), 2)    AS ticket_promedio
FROM VENTAS AS v
INNER JOIN CLIENTES AS c   ON v.ID_CLIENTE  = c.ID_CLIENTE
INNER JOIN PRODUCTOS AS p  ON v.ID_PRODUCTO = p.ID_PRODUCTO
GROUP BY c.SEGMENTO, p.CATEGORIA
ORDER BY ticket_promedio DESC;
```

**7.2.** Extiende la consulta anterior para incluir solo los segmentos que tienen más de 10 ventas (usa `HAVING`):

```sql
-- Versión extendida con HAVING
SELECT
    c.SEGMENTO,
    p.CATEGORIA,
    COUNT(DISTINCT c.ID_CLIENTE)    AS clientes_unicos,
    COUNT(v.ID_VENTA)               AS numero_ventas,
    SUM(v.TOTAL_VENTA)              AS ventas_totales,
    ROUND(AVG(v.TOTAL_VENTA), 2)    AS ticket_promedio
FROM VENTAS AS v
INNER JOIN CLIENTES AS c   ON v.ID_CLIENTE  = c.ID_CLIENTE
INNER JOIN PRODUCTOS AS p  ON v.ID_PRODUCTO = p.ID_PRODUCTO
GROUP BY c.SEGMENTO, p.CATEGORIA
HAVING COUNT(v.ID_VENTA) > 10
ORDER BY ticket_promedio DESC;
```

**7.3.** Desafío adicional (opcional): Identifica qué empleados **no han registrado ninguna venta** usando `LEFT JOIN`:

```sql
-- Empleados sin ventas registradas
SELECT
    e.EMPLEADO_ID,
    e.NOMBRE_EMPLEADO,
    e.REGION,
    e.CARGO
FROM EMPLEADOS AS e
LEFT JOIN VENTAS AS v
    ON e.EMPLEADO_ID = v.EMPLEADO_ID
WHERE v.ID_VENTA IS NULL
ORDER BY e.REGION, e.NOMBRE_EMPLEADO;
```

#### Resultado Esperado

- La consulta del paso 7.1 devuelve una tabla con combinaciones de segmento y categoría, ordenadas de mayor a menor ticket promedio.
- La consulta del paso 7.2 filtra las combinaciones con pocas ventas, dejando solo los grupos estadísticamente relevantes.
- El paso 7.3 puede devolver cero filas si todos los empleados tienen ventas, lo cual también es un resultado válido e informativo.

#### Verificación

```sql
-- Verificar que el número de segmentos únicos en el resultado
-- coincide con los segmentos existentes en CLIENTES
SELECT COUNT(DISTINCT SEGMENTO) AS segmentos_en_clientes FROM CLIENTES;

SELECT COUNT(DISTINCT c.SEGMENTO) AS segmentos_en_reporte
FROM VENTAS AS v
INNER JOIN CLIENTES AS c ON v.ID_CLIENTE = c.ID_CLIENTE;
```

Si los números difieren, hay segmentos de clientes que no tienen ninguna venta asociada (lo cual es información de negocio relevante).

---

## Validación y Pruebas

Ejecuta las siguientes consultas de validación para confirmar que completaste el laboratorio correctamente:

```sql
-- ══════════════════════════════════════════════════════
-- VALIDACIÓN FINAL DEL LABORATORIO 07-00-01
-- ══════════════════════════════════════════════════════

-- Prueba 1: INNER JOIN básico funciona correctamente
-- Esperado: número de filas igual o menor al total de VENTAS
SELECT COUNT(*) AS test_inner_join
FROM VENTAS AS v
INNER JOIN CLIENTES AS c ON v.ID_CLIENTE = c.ID_CLIENTE;

-- Prueba 2: LEFT JOIN devuelve más registros que INNER JOIN
-- Esperado: resultado_left >= resultado_inner
SELECT COUNT(*) AS test_left_join
FROM CLIENTES AS c
LEFT JOIN VENTAS AS v ON c.ID_CLIENTE = v.ID_CLIENTE;

-- Prueba 3: Patrón IS NULL funciona para detectar sin coincidencia
-- Esperado: número de clientes sin ventas (puede ser 0)
SELECT COUNT(*) AS clientes_sin_ventas
FROM CLIENTES AS c
LEFT JOIN VENTAS AS v ON c.ID_CLIENTE = v.ID_CLIENTE
WHERE v.ID_VENTA IS NULL;

-- Prueba 4: JOIN de cuatro tablas es consistente
-- Esperado: los cuatro conteos son iguales
SELECT
    COUNT(*)                 AS total_filas,
    COUNT(c.NOMBRE)          AS con_cliente,
    COUNT(p.NOMBRE_PRODUCTO) AS con_producto,
    COUNT(e.NOMBRE_EMPLEADO) AS con_empleado
FROM VENTAS AS v
INNER JOIN CLIENTES AS c   ON v.ID_CLIENTE  = c.ID_CLIENTE
INNER JOIN PRODUCTOS AS p  ON v.ID_PRODUCTO = p.ID_PRODUCTO
INNER JOIN EMPLEADOS AS e  ON v.EMPLEADO_ID = e.EMPLEADO_ID;

-- Prueba 5: Reporte integrador produce resultados
-- Esperado: al menos 1 fila con todos los campos no nulos
SELECT
    e.NOMBRE_EMPLEADO,
    p.CATEGORIA,
    COUNT(v.ID_VENTA)        AS ventas,
    SUM(v.TOTAL_VENTA)       AS total
FROM VENTAS AS v
INNER JOIN PRODUCTOS AS p  ON v.ID_PRODUCTO = p.ID_PRODUCTO
INNER JOIN EMPLEADOS AS e  ON v.EMPLEADO_ID = e.EMPLEADO_ID
GROUP BY e.NOMBRE_EMPLEADO, p.CATEGORIA
ORDER BY total DESC
LIMIT 3;
```

**Criterios de éxito:**
- ✅ Prueba 1: Devuelve un número ≥ 0 sin errores.
- ✅ Prueba 2: El resultado es ≥ al resultado de la Prueba 1.
- ✅ Prueba 3: Devuelve un número (puede ser 0).
- ✅ Prueba 4: Los cuatro valores en la fila son idénticos.
- ✅ Prueba 5: Devuelve exactamente 3 filas con todos los campos completos.

---

## Resolución de Problemas

### Problema 1: Error "ambiguous column name" al ejecutar un JOIN

**Síntoma:**
Snowflake devuelve un error similar a:
```
SQL compilation error: ambiguous column name 'ID_CLIENTE'
```
La consulta falla al intentar seleccionar o referenciar una columna que existe con el mismo nombre en más de una de las tablas del `JOIN`.

**Causa:**
Cuando dos o más tablas unidas tienen una columna con el mismo nombre (por ejemplo, `ID_CLIENTE` existe en `VENTAS` y en `CLIENTES`), Snowflake no puede determinar de cuál tabla proviene la referencia si no se especifica explícitamente.

**Solución:**
Siempre usa la notación `alias.columna` para cualquier columna que pueda existir en más de una tabla. Revisa todas las columnas del `SELECT` y del `WHERE`:

```sql
-- ❌ Incorrecto — genera ambigüedad
SELECT ID_CLIENTE, NOMBRE FROM VENTAS INNER JOIN CLIENTES ON ...

-- ✅ Correcto — alias resuelve la ambigüedad
SELECT v.ID_CLIENTE, c.NOMBRE FROM VENTAS AS v INNER JOIN CLIENTES AS c ON ...
```

Como regla general, en cualquier consulta con más de una tabla, **prefija todas las columnas con el alias de su tabla**, incluso aquellas que no son ambiguas. Esto hace el código más legible y evita errores futuros cuando el esquema cambia.

---

### Problema 2: El LEFT JOIN devuelve el mismo resultado que el INNER JOIN

**Síntoma:**
Al ejecutar un `LEFT JOIN` y luego filtrar con `WHERE v.ID_VENTA IS NULL`, la consulta devuelve 0 filas cuando se esperaban encontrar registros sin coincidencia. O bien, el conteo de filas del `LEFT JOIN` es idéntico al del `INNER JOIN`.

**Causa:**
El error más común es colocar la condición de filtrado en la cláusula `WHERE` en lugar de en la cláusula `ON`. Cuando se escribe `WHERE v.COLUMNA = 'valor'` después de un `LEFT JOIN`, Snowflake primero realiza el `LEFT JOIN` (incluyendo los `NULL`) y luego aplica el `WHERE`, que **elimina todas las filas con `NULL`** en esa columna, convirtiendo el `LEFT JOIN` en un `INNER JOIN` efectivo.

```sql
-- ❌ Incorrecto — el WHERE elimina los NULLs del LEFT JOIN
SELECT c.NOMBRE, v.TOTAL_VENTA
FROM CLIENTES AS c
LEFT JOIN VENTAS AS v ON c.ID_CLIENTE = v.ID_CLIENTE
WHERE v.TOTAL_VENTA > 100;   -- Esto excluye clientes sin ventas (NULL > 100 = FALSE)

-- ✅ Correcto — mover el filtro de la tabla derecha al ON
SELECT c.NOMBRE, v.TOTAL_VENTA
FROM CLIENTES AS c
LEFT JOIN VENTAS AS v
    ON c.ID_CLIENTE = v.ID_CLIENTE
    AND v.TOTAL_VENTA > 100;   -- Filtro dentro del JOIN, preserva clientes sin ventas
```

**Solución:**
- Si quieres filtrar registros de la tabla **izquierda** (la que tiene todos los registros), usa `WHERE` normalmente.
- Si quieres filtrar registros de la tabla **derecha** (la que puede tener `NULL`) sin perder los registros de la izquierda, mueve la condición al `ON`.
- El patrón `WHERE tabla_derecha.columna IS NULL` es la excepción válida: se usa intencionalmente para encontrar registros sin coincidencia.

---

## Limpieza del Entorno

Al finalizar el laboratorio, ejecuta los siguientes comandos para liberar recursos y minimizar el consumo de créditos:

```sql
-- 1. Suspender el Virtual Warehouse para detener el consumo de créditos
ALTER WAREHOUSE COMPUTE_WH SUSPEND;

-- 2. (Opcional) Verificar que el warehouse está suspendido
SHOW WAREHOUSES LIKE 'COMPUTE_WH';
-- La columna STATE debe mostrar 'SUSPENDED'
```

> ⚠️ **Importante:** No elimines ninguna tabla del dataset `CURSO_SQL`. Las tablas `VENTAS`, `CLIENTES`, `PRODUCTOS` y `EMPLEADOS` son necesarias para los laboratorios siguientes.

> 💡 **Tip:** Guarda tus consultas del laboratorio en el Worksheet de Snowsight antes de cerrar el navegador. Puedes nombrar cada consulta con un comentario descriptivo para encontrarlas fácilmente en sesiones futuras.

---

## Resumen

En este laboratorio exploraste el concepto fundamental que hace posible el análisis de datos relacionales: la combinación de tablas mediante `JOIN`. Los puntos clave que trabajaste son:

| Concepto aprendido                    | Detalle                                                                                  |
|---------------------------------------|------------------------------------------------------------------------------------------|
| **Esquema relacional**                | Las tablas `VENTAS`, `CLIENTES`, `PRODUCTOS` y `EMPLEADOS` se conectan mediante claves primarias y foráneas. |
| **`INNER JOIN`**                      | Devuelve solo los registros que tienen coincidencia en **ambas** tablas. Ideal para reportes de datos completos. |
| **`LEFT JOIN`**                       | Devuelve **todos** los registros de la tabla izquierda, con `NULL` donde no hay coincidencia en la derecha. |
| **Patrón `IS NULL` con `LEFT JOIN`**  | Permite identificar registros sin correspondencia (clientes sin ventas, empleados sin actividad). |
| **Alias de tabla**                    | `AS v`, `AS c`, `AS p`, `AS e` reducen la verbosidad y resuelven ambigüedades de nombres de columna. |
| **JOIN + WHERE + GROUP BY + ORDER BY**| La combinación de estas cláusulas permite construir reportes de análisis complejos y orientados al negocio. |

### Progresión hacia el siguiente laboratorio

Los conceptos de `JOIN` aprendidos hoy son la base para análisis más avanzados. En los laboratorios siguientes podrás aplicar estas técnicas para construir dashboards de datos, calcular métricas de negocio complejas y trabajar con subconsultas que combinan múltiples niveles de análisis.

---

## Recursos Adicionales

- [Documentación oficial de Snowflake: JOIN](https://docs.snowflake.com/en/sql-reference/constructs/join)
- [Documentación oficial de Snowflake: Constraints (PK y FK)](https://docs.snowflake.com/en/sql-reference/constraints-overview)
- [Visual JOIN Guide — C.L. Moffatt (clásico)](https://www.codeproject.com/Articles/33052/Visual-Representation-of-SQL-Joins)
- [SQL JOINs explicados — Mode Analytics](https://mode.com/sql-tutorial/sql-joins/)
- [COALESCE en Snowflake](https://docs.snowflake.com/en/sql-reference/functions/coalesce)

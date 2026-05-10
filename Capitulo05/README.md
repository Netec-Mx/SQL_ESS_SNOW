



# Métricas básicas

## Metadatos

| Campo            | Detalle                          |
|------------------|----------------------------------|
| **Duración**     | 45 minutos                       |
| **Complejidad**  | Media                            |
| **Nivel Bloom**  | Aplicar (Apply)                  |
| **Módulo**       | 5 — Funciones de Agregación      |
| **Versión**      | 1.0                              |

---

## Descripción General

En este laboratorio aplicarás las cinco funciones de agregación fundamentales de SQL —`COUNT`, `SUM`, `AVG`, `MIN` y `MAX`— sobre las tablas **VENTAS** y **PRODUCTOS** del dataset `CURSO_SQL`. Comenzarás con consultas simples de conteo, avanzarás hacia cálculos de totales y promedios, y finalizarás construyendo una consulta única que integre todas las métricas en un resumen ejecutivo tipo *dashboard*. Este laboratorio es el puente directo hacia el análisis por grupos con `GROUP BY` del siguiente módulo.

---

## Objetivos de Aprendizaje

Al finalizar este laboratorio, serás capaz de:

- [ ] Utilizar `COUNT(*)`, `COUNT(columna)` y `COUNT(DISTINCT columna)` para contar registros totales, no nulos y únicos respectivamente.
- [ ] Aplicar `SUM` y `AVG` para calcular el total de ingresos y el ticket promedio de ventas.
- [ ] Identificar los valores extremos de una columna numérica usando `MIN` y `MAX`.
- [ ] Combinar funciones de agregación con la cláusula `WHERE` para obtener métricas filtradas por condición.
- [ ] Integrar múltiples funciones de agregación en una sola consulta `SELECT` con alias descriptivos para generar un resumen completo de métricas de negocio.

---

## Prerrequisitos

### Conocimientos Previos

| Requisito                                                                 | Nivel Esperado     |
|---------------------------------------------------------------------------|--------------------|
| Completar Lab 02-00-01 o demostrar dominio de `SELECT` con `WHERE`        | Obligatorio        |
| Comprensión de `ORDER BY` y alias con `AS`                                | Recomendado        |
| Entender suma, promedio, mínimo y máximo como operaciones matemáticas     | Obligatorio        |
| Familiaridad con la interfaz Snowsight (Worksheets)                       | Obligatorio        |

### Acceso y Recursos

| Recurso                                          | Estado Requerido                                              |
|--------------------------------------------------|---------------------------------------------------------------|
| Cuenta Snowflake activa (trial o corporativa)    | Activa y accesible                                            |
| Base de datos `CURSO_SQL` inicializada           | Script de setup ejecutado correctamente (ver sección 5)       |
| Virtual Warehouse disponible                     | Tamaño **X-Small**, estado **Started** o **Auto-resume ON**   |
| Navegador web compatible                         | Chrome 90+, Firefox 88+, Edge 90+ o Safari 14+                |

---

## Entorno del Laboratorio

### Hardware Mínimo Recomendado

| Componente        | Mínimo Requerido                                           |
|-------------------|------------------------------------------------------------|
| Conexión a internet | 10 Mbps estables                                         |
| RAM               | 4 GB (para operación fluida con múltiples pestañas)        |
| Resolución        | 1280×768 píxeles                                           |
| Sistema Operativo | Windows 10+, macOS 10.15+, o Linux moderno                 |

### Software Utilizado

| Software                    | Versión / Edición                          | Notas                                       |
|-----------------------------|--------------------------------------------|---------------------------------------------|
| Snowflake                   | Standard o superior                        | Acceso vía navegador web                    |
| Snowsight                   | Incluida en toda cuenta Snowflake activa   | JavaScript debe estar habilitado            |
| Navegador Web               | Chrome 90+ (recomendado)                   | Sin extensiones de seguridad agresivas      |
| Dataset `CURSO_SQL`         | v1.0 (provista por el instructor)          | Debe estar inicializado antes de comenzar   |

### Configuración Inicial del Entorno

Antes de comenzar los ejercicios, ejecuta los siguientes comandos en un nuevo Worksheet de Snowsight para establecer el contexto correcto. Esto garantiza que todas las consultas del laboratorio apunten al dataset correcto.

1. Clic en la sección **Projects** y luego clic en **Workspaces**.
2. Abre tu **Workspace** llamado **SnowEssLAbs**.
3. Crea un archivo de tipo SQL llamado **`Lab05_Metricas`**.
4. Ejecuta los siguientes comandos para asegurarte de estar trabajando en el contexto correcto:
   
> **⚠️ Nota:** Si la tabla **VENTAS** no aparece ve al **Workspace** llamado **Setup_CURSO_SQLSNOW** y ejecuta el script llamado **Setup_Lab02_Ventas**.

```sql
-- Paso 1: Seleccionar la base de datos del curso
USE DATABASE CURSO_SQL;

-- Paso 2: Seleccionar el schema (ajustar si usas schema personal)
USE SCHEMA PUBLIC;

-- Paso 3: Seleccionar el warehouse (usar X-Small para minimizar créditos)
USE WAREHOUSE COMPUTE_WH;

-- Paso 4: Verificar que las tablas necesarias existen
SHOW TABLES LIKE 'VENTAS';
SHOW TABLES LIKE 'PRODUCTOS';
```

**Resultado esperado de la verificación:** Deberías ver dos filas en la salida de `SHOW TABLES`, una para `VENTAS` y otra para `PRODUCTOS`. Si alguna tabla no aparece, contacta a tu instructor para ejecutar el script de inicialización del dataset.

```sql
-- Paso 5 (Opcional): Vista previa rápida de las tablas
SELECT * FROM VENTAS LIMIT 5;
SELECT * FROM PRODUCTOS LIMIT 5;
```

Tómate un momento para observar las columnas disponibles. Las columnas clave que utilizarás en este laboratorio son:

**Tabla VENTAS:**
- `ID_VENTA` — Identificador único de cada venta
- `ID_CLIENTE` — Identificador del cliente (puede contener NULLs)
- `ID_PRODUCTO` — Identificador único del producto
- `CANTIDAD` — Valor monetario de la venta
- `FECHA_VENTA` — Fecha en que se realizó la venta
- `ESTADO_ENVIO` — Estado de la venta (`completada`, `pendiente`, `cancelada`)

**Tabla PRODUCTOS:**
- `ID_PRODUCTO` — Identificador único del producto
- `NOMBRE_PRODUCTO` — Nombre del producto
- `CATEGORIA` — Categoría del producto
- `PRECIO` — Precio unitario del producto
- `STOCK` — Unidades disponibles en inventario

---

## Ejercicios Paso a Paso

### Ejercicio 1: Contando Registros con COUNT

**Objetivo:** Dominar las tres variantes de `COUNT` y entender cómo el tratamiento de valores `NULL` afecta los resultados.

#### Instrucciones

**1.1 — Contar el total de ventas registradas**

Escribe una consulta que cuente todas las filas de la tabla `VENTAS`, independientemente de si alguna columna tiene valores `NULL`.

```sql
-- Consulta 1.1: Total de ventas en la tabla
SELECT COUNT(*) AS total_ventas
FROM VENTAS;
```

**Resultado esperado:**

| TOTAL_VENTAS |
|:------------:|
| (número entero, ej. 500) |

> **¿Por qué `COUNT(*)`?** Esta variante cuenta todas las filas sin evaluar el contenido de ninguna columna. Es la forma más eficiente de obtener el tamaño total de una tabla en Snowflake.

---

**1.2 — Detectar ventas sin cliente asignado**

Ahora compara `COUNT(*)` con `COUNT(ID_CLIENTE)` para identificar cuántas ventas no tienen un cliente registrado.

```sql
-- Consulta 1.2: Comparación COUNT(*) vs COUNT(columna)
SELECT
    COUNT(*)          AS total_filas,
    COUNT(ID_CLIENTE) AS ventas_con_cliente,
    COUNT(*) - COUNT(ID_CLIENTE) AS ventas_sin_cliente
FROM VENTAS;
```

**Resultado esperado:**

| TOTAL_FILAS | VENTAS_CON_CLIENTE | VENTAS_SIN_CLIENTE |
|:-----------:|:------------------:|:------------------:|
| 500         | 487                | 13                 |

> **Nota:** Los números exactos dependerán de tu dataset. Lo importante es que `VENTAS_SIN_CLIENTE` muestre la diferencia entre ambos conteos, lo que revela cuántas filas tienen `NULL` en `ID_CLIENTE`.

> **Concepto clave:** `COUNT(columna)` ignora los valores `NULL`. Esta diferencia entre `COUNT(*)` y `COUNT(columna)` es una técnica estándar de auditoría de calidad de datos.

---

**1.3 — Contar clientes únicos que han comprado**

Utiliza `COUNT(DISTINCT columna)` para responder: *¿Cuántos clientes diferentes han realizado al menos una compra?*

```sql
-- Consulta 1.3: Clientes únicos con compras registradas
SELECT COUNT(DISTINCT ID_CLIENTE) AS clientes_unicos
FROM VENTAS;
```

**Resultado esperado:**

| CLIENTES_UNICOS |
|:---------------:|
| (número menor que VENTAS_CON_CLIENTE, ej. 120) |

> **Reflexión:** Si `VENTAS_CON_CLIENTE` es 487 y `CLIENTES_UNICOS` es, por ejemplo, 120, significa que en promedio cada cliente realizó aproximadamente 4 compras. Esta es la lógica detrás de métricas como "frecuencia de compra".

---

**1.4 — Contar ventas por estado usando WHERE**

Combina `COUNT(*)` con `WHERE` para contar solo las ventas con estado `'completada'`.

```sql
-- Consulta 1.4: Ventas completadas
SELECT COUNT(*) AS ventas_completadas
FROM VENTAS
WHERE ESTADO_ENVIO = 'Entregado';
```

Repite la consulta para los otros estados. Puedes ejecutarlas una por una o combinarlas:

```sql
-- Consulta 1.4b: Resumen de ventas por estado (anticipo de GROUP BY)
SELECT COUNT(*) AS ventas_pendientes
FROM VENTAS
WHERE ESTADO_ENVIO = 'Pendiente';
```
```sql
SELECT COUNT(*) AS ventas_canceladas
FROM VENTAS
WHERE ESTADO_ENVIO = 'Cancelado';
```

**Resultado esperado (ejemplo):**

| VENTAS_COMPLETADAS | VENTAS_PENDIENTES | VENTAS_CANCELADAS |
|:------------------:|:-----------------:|:-----------------:|
| 380                | 75                | 45                |

> **Buena práctica:** Siempre usa alias descriptivos con `AS` cuando el nombre de la columna resultado no es autoexplicativo. `COUNT(*)` como nombre de columna en un reporte es confuso; `ventas_completadas` es inmediatamente comprensible.

#### Verificación del Ejercicio 1

Confirma que tus resultados son coherentes respondiendo estas preguntas mentalmente:
- [ ] ¿El valor de `VENTAS_CON_CLIENTE` es menor o igual que `TOTAL_FILAS`? (Debe ser ≤)
- [ ] ¿El valor de `CLIENTES_UNICOS` es menor que `VENTAS_CON_CLIENTE`? (Debe ser <, a menos que cada cliente compre exactamente una vez)
- [ ] ¿La suma de `VENTAS_COMPLETADAS + VENTAS_PENDIENTES + VENTAS_CANCELADAS` es igual a `TOTAL_FILAS`? (Debe coincidir si no hay otros estados)

---

### Ejercicio 2: Calculando Totales con SUM

**Objetivo:** Aplicar `SUM` para calcular ingresos totales y combinarla con `WHERE` para obtener totales parciales filtrados.

#### Instrucciones

**2.1 — Calcular el ingreso total de todas las ventas**

```sql
-- Consulta 2.1: Ingreso total acumulado
SELECT SUM(CANTIDAD) AS ingreso_total
FROM VENTAS;
```

**Resultado esperado:**

| INGRESO_TOTAL |
|:-------------:|
| (valor numérico, ej. 485750.00) |

> **Nota sobre NULLs en SUM:** Al igual que `COUNT(columna)`, `SUM` ignora los valores `NULL`. Si una fila tiene `MONTO = NULL`, no se incluye en la suma. Esto es el comportamiento estándar de todas las funciones de agregación en SQL.

---

**2.2 — Calcular ingresos solo de ventas completadas**

```sql
-- Consulta 2.2: Ingresos de ventas completadas
SELECT SUM(CANTIDAD) AS ingresos_completadas
FROM VENTAS
WHERE ESTADO_ENVIO = 'Entregado';
```

**Resultado esperado:**

| INGRESOS_COMPLETADAS |
|:--------------------:|
| (valor menor que INGRESO_TOTAL) |

---

**2.3 — Comparar ingresos reales vs. ingresos perdidos**

Construye una consulta que calcule simultáneamente el total de ingresos por ventas completadas y el monto "perdido" por ventas canceladas.

```sql
-- Consulta 2.3: Ingresos realizados vs. ingresos perdidos
SELECT
    SUM(CASE WHEN ESTADO_ENVIO = 'Entregado' THEN CANTIDAD ELSE 0 END) AS ingresos_realizados,
    SUM(CASE WHEN ESTADO_ENVIO = 'Cancelado'  THEN CANTIDAD ELSE 0 END) AS ingresos_perdidos
FROM VENTAS;
```

> **Nota pedagógica:** Esta consulta introduce `CASE WHEN` de forma anticipada para mostrar el poder de `SUM` condicionada. No te preocupes si la sintaxis de `CASE WHEN` no te resulta familiar aún; lo importante es entender que `SUM` suma los valores que resulten de la expresión interna. En laboratorios posteriores se cubrirá `CASE WHEN` en detalle.

**Resultado esperado (ejemplo):**

| INGRESOS_REALIZADOS | INGRESOS_PERDIDOS |
|:-------------------:|:-----------------:|
| 380500.00           | 42300.00          |

**Alternativa más simple usando dos consultas separadas con WHERE:**

```sql
-- Alternativa 2.3: Dos consultas independientes con WHERE
SELECT SUM(CANTIDAD) AS ingresos_realizados
FROM VENTAS
WHERE ESTADO_ENVIO = 'Entregado';

SELECT SUM(CANTIDAD) AS ingresos_perdidos
FROM VENTAS
WHERE ESTADO_ENVIO = 'Cancelado';
```

#### Verificación del Ejercicio 2

- [ ] ¿`INGRESOS_REALIZADOS` es menor que `INGRESO_TOTAL`? (Debe serlo, ya que el total incluye todos los estados)
- [ ] ¿El resultado de `SUM(MONTO)` tiene formato numérico decimal? (En Snowflake, `SUM` sobre columnas `NUMBER` devuelve `NUMBER`)

---

### Ejercicio 3: Calculando Promedios con AVG

**Objetivo:** Usar `AVG` para calcular el ticket promedio de ventas y entender cómo los valores atípicos pueden influir en el promedio.

#### Instrucciones

**3.1 — Calcular el ticket promedio de todas las ventas**

```sql
-- Consulta 3.1: Ticket promedio general
SELECT AVG(CANTIDAD) AS ticket_promedio
FROM VENTAS;
```

**Resultado esperado:**

| TICKET_PROMEDIO |
|:---------------:|
| (valor decimal, ej. 971.50) |

> **Comportamiento de AVG con NULLs:** `AVG` ignora los valores `NULL` al calcular el promedio. Calcula la suma de los valores no nulos dividida entre el conteo de valores no nulos. Esto significa que los `NULL` no "jalan" el promedio hacia cero, lo cual es el comportamiento matemáticamente correcto.

---

**3.2 — Comparar el promedio general vs. el promedio de ventas completadas**

```sql
-- Consulta 3.2: Promedio general vs. promedio de ventas completadas
SELECT AVG(CANTIDAD) AS promedio_general
FROM VENTAS;

SELECT AVG(CANTIDAD) AS promedio_completadas
FROM VENTAS
WHERE ESTADO_ENVIO = 'Entregado';
```

> **Pregunta de análisis:** ¿El promedio de ventas completadas es mayor, menor o igual al promedio general? Reflexiona sobre por qué podría diferir.

---

**3.3 — Calcular el precio promedio de productos en el catálogo**

Ahora trabaja con la tabla `PRODUCTOS` para calcular el precio promedio del catálogo.

```sql
-- Consulta 3.3: Precio promedio del catálogo de productos
SELECT
    COUNT(*)           AS total_productos,
    AVG(PRECIO)        AS precio_promedio,
    AVG(STOCK)         AS stock_promedio
FROM PRODUCTOS;
```

**Resultado esperado (ejemplo):**

| TOTAL_PRODUCTOS | PRECIO_PROMEDIO | STOCK_PROMEDIO |
|:---------------:|:---------------:|:--------------:|
| 85              | 234.75          | 142.30         |

#### Verificación del Ejercicio 3

- [ ] ¿El resultado de `AVG` tiene decimales? (Snowflake devuelve un valor decimal incluso si los datos originales son enteros)
- [ ] ¿`PROMEDIO_COMPLETADAS` difiere de `PROMEDIO_GENERAL`? (Es esperado que difieran ligeramente)

---

### Ejercicio 4: Valores Extremos con MIN y MAX

**Objetivo:** Identificar los valores mínimo y máximo en columnas numéricas y de fecha para responder preguntas de negocio sobre rangos y extremos.

#### Instrucciones

**4.1 — Encontrar la venta más pequeña y la más grande**

```sql
-- Consulta 4.1: Rango de montos de venta
SELECT
    MIN(CANTIDAD) AS venta_minima,
    MAX(CANTIDAD) AS venta_maxima
FROM VENTAS;
```

**Resultado esperado (ejemplo):**

| VENTA_MINIMA | VENTA_MAXIMA |
|:------------:|:------------:|
| 5.99         | 4999.00      |

---

**4.2 — Encontrar las fechas de la primera y última venta**

`MIN` y `MAX` también funcionan sobre columnas de tipo fecha, devolviendo la fecha más antigua y la más reciente respectivamente.

```sql
-- Consulta 4.2: Rango temporal de las ventas
SELECT
    MIN(FECHA_VENTA) AS primera_venta,
    MAX(FECHA_VENTA) AS ultima_venta
FROM VENTAS;
```

**Resultado esperado (ejemplo):**

| PRIMERA_VENTA | ULTIMA_VENTA |
|:-------------:|:------------:|
| 2023-01-03    | 2024-12-28   |

> **Aplicación de negocio:** Esta consulta responde preguntas como *"¿Desde cuándo tenemos datos?"* y *"¿Cuál es el período que cubre nuestro dataset?"*. Es una verificación estándar al recibir un nuevo conjunto de datos.

---

**4.3 — Precio mínimo y máximo del catálogo de productos**

```sql
-- Consulta 4.3: Rango de precios en el catálogo
SELECT
    MIN(PRECIO) AS precio_minimo,
    MAX(PRECIO) AS precio_maximo,
    MAX(PRECIO) - MIN(PRECIO) AS rango_precios
FROM PRODUCTOS;
```

**Resultado esperado (ejemplo):**

| PRECIO_MINIMO | PRECIO_MAXIMO | RANGO_PRECIOS |
|:-------------:|:-------------:|:-------------:|
| 4.99          | 1299.00       | 1294.01       |

> **Concepto:** `RANGO_PRECIOS` es una expresión calculada dentro del `SELECT`. Puedes realizar operaciones aritméticas directamente sobre los resultados de funciones de agregación, lo que permite derivar métricas adicionales sin subconsultas.

---

**4.4 — Extremos filtrados: ventas completadas solamente**

```sql
-- Consulta 4.4: MIN y MAX solo para ventas completadas
SELECT
    MIN(CANTIDAD) AS min_completadas,
    MAX(CANTIDAD) AS max_completadas
FROM VENTAS
WHERE ESTADO_ENVIO = 'Entregado';
```

#### Verificación del Ejercicio 4

- [ ] ¿`VENTA_MINIMA` es un valor positivo mayor que cero? (No debería haber ventas con monto negativo o cero en el dataset)
- [ ] ¿`PRIMERA_VENTA` es anterior a `ULTIMA_VENTA`? (Verificación lógica básica de coherencia de datos)
- [ ] ¿`RANGO_PRECIOS` es igual a `PRECIO_MAXIMO - PRECIO_MINIMO`? (Verificación aritmética)

---

### Ejercicio 5: Dashboard de Métricas — Consulta Integradora

**Objetivo:** Construir una única consulta `SELECT` que integre todas las funciones de agregación estudiadas para generar un resumen ejecutivo completo de las ventas.

Este ejercicio representa el objetivo principal del laboratorio: demostrar que `COUNT`, `SUM`, `AVG`, `MIN` y `MAX` pueden coexistir en una sola consulta para producir un *dashboard* de métricas en una sola ejecución.

#### Instrucciones

**5.1 — Dashboard completo de métricas de ventas**

Construye la siguiente consulta paso a paso. Primero escríbela completa y luego ejecútala.

```sql
-- Consulta 5.1: Dashboard ejecutivo de métricas de ventas
SELECT
    -- Métricas de volumen (COUNT)
    COUNT(*)                    AS total_transacciones,
    COUNT(ID_CLIENTE)           AS transacciones_con_cliente,
    COUNT(DISTINCT ID_CLIENTE)  AS clientes_unicos,

    -- Métricas financieras (SUM)
    SUM(CANTIDAD)                  AS ingreso_total,

    -- Métricas de ticket (AVG)
    ROUND(AVG(CANTIDAD), 2)        AS ticket_promedio,

    -- Métricas de rango (MIN / MAX)
    MIN(CANTIDAD)                  AS venta_minima,
    MAX(CANTIDAD)                  AS venta_maxima,
    MIN(FECHA_VENTA)            AS primera_venta,
    MAX(FECHA_VENTA)            AS ultima_venta
FROM VENTAS;
```

> **Nota sobre `ROUND`:** `ROUND(AVG(MONTO), 2)` redondea el promedio a 2 decimales para mayor legibilidad. `ROUND` es una función escalar que puede envolver el resultado de una función de agregación.

**Resultado esperado (ejemplo):**

| TOTAL_TRANSACCIONES | TRANSACCIONES_CON_CLIENTE | CLIENTES_UNICOS | INGRESO_TOTAL | TICKET_PROMEDIO | VENTA_MINIMA | VENTA_MAXIMA | PRIMERA_VENTA | ULTIMA_VENTA |
|:-------------------:|:-------------------------:|:---------------:|:-------------:|:---------------:|:------------:|:------------:|:-------------:|:------------:|
| 500                 | 487                       | 120             | 485750.00     | 971.50          | 5.99         | 4999.00      | 2023-01-03    | 2024-12-28   |

---

**5.2 — Dashboard de métricas solo para ventas completadas**

Ahora replica el mismo dashboard pero filtrando únicamente las ventas con estado `'completada'`. Solo necesitas agregar una cláusula `WHERE`.

```sql
-- Consulta 5.2: Dashboard de métricas — solo ventas completadas
SELECT
    COUNT(*)                    AS total_transacciones,
    COUNT(ID_CLIENTE)           AS transacciones_con_cliente,
    COUNT(DISTINCT ID_CLIENTE)  AS clientes_unicos,
    SUM(CANTIDAD)                  AS ingreso_total,
    ROUND(AVG(CANTIDAD), 2)        AS ticket_promedio,
    MIN(CANTIDAD)                  AS venta_minima,
    MAX(CANTIDAD)                  AS venta_maxima,
    MIN(FECHA_VENTA)            AS primera_venta,
    MAX(FECHA_VENTA)            AS ultima_venta
FROM VENTAS
WHERE ESTADO_ENVIO = 'Entregado';
```

> **Observación clave:** La cláusula `WHERE` se aplica **antes** de que las funciones de agregación calculen sus resultados. Snowflake primero filtra las filas que cumplen `ESTADO = 'completada'` y luego aplica `COUNT`, `SUM`, `AVG`, `MIN` y `MAX` sobre ese subconjunto de filas. Este es el orden lógico de ejecución de SQL.

---

**5.3 — Dashboard de métricas del catálogo de productos**

Aplica el mismo enfoque a la tabla `PRODUCTOS` para generar un resumen del catálogo.

```sql
-- Consulta 5.3: Dashboard de métricas del catálogo de productos
SELECT
    COUNT(*)                    AS total_productos,
    COUNT(DISTINCT CATEGORIA)   AS categorias_unicas,
    ROUND(AVG(PRECIO), 2)       AS precio_promedio,
    MIN(PRECIO)                 AS precio_minimo,
    MAX(PRECIO)                 AS precio_maximo,
    SUM(STOCK)                  AS stock_total,
    ROUND(AVG(STOCK), 0)        AS stock_promedio_por_producto
FROM PRODUCTOS;
```

**Resultado esperado (ejemplo):**

| TOTAL_PRODUCTOS | CATEGORIAS_UNICAS | PRECIO_PROMEDIO | PRECIO_MINIMO | PRECIO_MAXIMO | STOCK_TOTAL | STOCK_PROMEDIO_POR_PRODUCTO |
|:---------------:|:-----------------:|:---------------:|:-------------:|:-------------:|:-----------:|:---------------------------:|
| 85              | 6                 | 234.75          | 4.99          | 1299.00       | 12095       | 142                         |

#### Verificación del Ejercicio 5

- [ ] ¿La consulta 5.1 devuelve exactamente **una sola fila** con múltiples columnas? (Todas las funciones de agregación sin `GROUP BY` siempre devuelven una sola fila)
- [ ] ¿Los valores de `TOTAL_TRANSACCIONES` en la consulta 5.2 son menores que en la consulta 5.1? (Deben serlo, ya que filtramos por estado)
- [ ] ¿`CLIENTES_UNICOS` es menor que `TRANSACCIONES_CON_CLIENTE` en ambas consultas? (Un cliente puede tener múltiples transacciones)
- [ ] ¿`TICKET_PROMEDIO` está entre `VENTA_MINIMA` y `VENTA_MAXIMA`? (El promedio siempre debe estar dentro del rango)

---

## Validación y Pruebas

Ejecuta las siguientes consultas de validación para confirmar que todos tus resultados son coherentes entre sí. Estas verificaciones cruzadas son una práctica profesional estándar en análisis de datos.

```sql
-- Validación 1: La suma de ventas por estado debe igualar el total de ventas
-- (Asumiendo que solo existen los estados: completada, pendiente, cancelada)
SELECT
    COUNT(*) AS total_verificacion,
    COUNT(CASE WHEN ESTADO_ENVIO = 'Entregado' THEN 1 END) +
    COUNT(CASE WHEN ESTADO_ENVIO = 'Pendiente'  THEN 1 END) +
    COUNT(CASE WHEN ESTADO_ENVIO = 'Cancelado'  THEN 1 END) AS suma_por_estado
FROM VENTAS;
-- Ambas columnas deben tener el mismo valor
```

```sql
-- Validación 2: El ingreso total debe ser mayor que el ingreso de solo completadas
SELECT
    (SELECT SUM(CANTIDAD) FROM VENTAS)                           AS ingreso_total,
    (SELECT SUM(CANTIDAD) FROM VENTAS WHERE ESTADO_ENVIO = 'Entregado') AS ingreso_completadas,
    (SELECT SUM(CANTIDAD) FROM VENTAS) >
    (SELECT SUM(CANTIDAD) FROM VENTAS WHERE ESTADO_ENVIO = 'Entregado') AS total_es_mayor
FROM DUAL;
-- total_es_mayor debe ser TRUE
```

> **Nota sobre `FROM DUAL`:** En Snowflake, `DUAL` es una tabla especial de una sola fila usada para ejecutar expresiones sin referenciar una tabla real. Es equivalente a `SELECT ... FROM (SELECT 1)`.

```sql
-- Validación 3: AVG debe estar entre MIN y MAX
SELECT
    MIN(CANTIDAD)  AS venta_minima,
    AVG(CANTIDAD)  AS ticket_promedio,
    MAX(CANTIDAD)  AS venta_maxima,
    (AVG(CANTIDAD) >= MIN(CANTIDAD) AND AVG(CANTIDAD) <= MAX(CANTIDAD)) AS promedio_en_rango
FROM VENTAS;
-- promedio_en_rango debe ser TRUE
```

```sql
-- Validación 4: COUNT(DISTINCT) debe ser <= COUNT(columna)
SELECT
    COUNT(ID_CLIENTE)           AS conteo_total,
    COUNT(DISTINCT ID_CLIENTE)  AS conteo_unico,
    COUNT(ID_CLIENTE) >= COUNT(DISTINCT ID_CLIENTE) AS logica_correcta
FROM VENTAS;
-- logica_correcta debe ser TRUE
```

**Criterios de éxito del laboratorio:**

| Validación                                                      | Resultado Esperado |
|-----------------------------------------------------------------|--------------------|
| Suma por estado == Total de ventas                              | TRUE               |
| Ingreso total > Ingreso de completadas                          | TRUE               |
| AVG está entre MIN y MAX                                        | TRUE               |
| COUNT(columna) >= COUNT(DISTINCT columna)                       | TRUE               |
| Dashboard de métricas devuelve exactamente 1 fila               | TRUE               |

---

## Solución de Problemas

### Problema 1: La consulta devuelve NULL en lugar de un número

**Síntoma:** Al ejecutar `SELECT SUM(CANTIDAD) FROM VENTAS` o `SELECT AVG(CANTIDAD) FROM VENTAS`, el resultado muestra `NULL` en lugar de un valor numérico.

**Causa:** Esto ocurre cuando **todos** los valores de la columna `MONTO` son `NULL`. Las funciones de agregación `SUM`, `AVG`, `MIN` y `MAX` devuelven `NULL` cuando no hay valores no nulos para procesar. También puede ocurrir si la tabla está vacía (cero filas) o si la cláusula `WHERE` filtra todas las filas.

**Solución:**

```sql
-- Paso 1: Verificar si la tabla tiene datos
SELECT COUNT(*) FROM VENTAS;
-- Si devuelve 0, la tabla está vacía. Ejecutar el script de inicialización del dataset.

-- Paso 2: Verificar si hay valores no nulos en MONTO
SELECT COUNT(CANTIDAD) AS filas_con_monto FROM VENTAS;
-- Si devuelve 0, todos los valores de CANTIDAD son NULL.

-- Paso 3: Si usaste WHERE, verificar que la condición no filtra todo
SELECT COUNT(*) FROM VENTAS WHERE ESTADO_ENVIO = 'Entregado';
-- Si devuelve 0, el valor del filtro no existe en los datos.
-- Verificar los valores existentes:
SELECT DISTINCT ESTADO_ENVIO FROM VENTAS;
```

Si la tabla está vacía, solicita al instructor que vuelva a ejecutar el script de inicialización del dataset `CURSO_SQL`.

---

### Problema 2: Error "SQL compilation error: ambiguous column name" al ejecutar el dashboard integrador

**Síntoma:** Al ejecutar la consulta del Ejercicio 5 (o cualquier consulta con múltiples funciones de agregación), Snowflake lanza el error `SQL compilation error: ambiguous column name 'CANTIDAD'` o similar.

**Causa:** Este error ocurre cuando se intenta hacer un `JOIN` implícito o cuando se referencian tablas sin calificar el nombre de la columna. En el contexto de este laboratorio, la causa más probable es que el estudiante haya modificado la consulta del dashboard para incluir columnas de ambas tablas (`VENTAS` y `PRODUCTOS`) sin especificar a qué tabla pertenece cada columna.

**Solución:**

```sql
-- INCORRECTO: Snowflake no sabe a qué tabla pertenece MONTO si hay JOIN
SELECT COUNT(*), SUM(CANTIDAD), AVG(PRECIO)
FROM VENTAS, PRODUCTOS;  -- Este tipo de JOIN sin condición es un producto cartesiano

-- CORRECTO: Calificar cada columna con su tabla de origen
SELECT
    COUNT(*)                AS total_ventas,
    SUM(v.CANTIDAD)            AS ingreso_total,
    AVG(p.PRECIO)           AS precio_promedio_catalogo
FROM VENTAS v
JOIN PRODUCTOS p ON v.ID_PRODUCTO = p.ID_PRODUCTO;

-- MÁS SIMPLE: Si solo necesitas métricas de una tabla, no hagas JOIN
-- Dashboard de VENTAS:
SELECT COUNT(*), SUM(CANTIDAD), AVG(CANTIDAD), MIN(CANTIDAD), MAX(CANTIDAD)
FROM VENTAS;

-- Dashboard de PRODUCTOS (consulta separada):
SELECT COUNT(*), AVG(PRECIO), MIN(PRECIO), MAX(PRECIO)
FROM PRODUCTOS;
```

> **Regla práctica:** En este laboratorio, cada consulta de dashboard debe referenciar **una sola tabla** en su cláusula `FROM`. Si necesitas métricas de ambas tablas, usa dos consultas separadas.

---

## Limpieza del Entorno

Al finalizar el laboratorio, realiza las siguientes acciones para liberar recursos y minimizar el consumo de créditos Snowflake.

```sql
-- Paso 1: Verificar el estado actual del warehouse
SELECT CURRENT_WAREHOUSE();

-- Paso 2: Suspender el Virtual Warehouse manualmente
-- (Si no tiene Auto-Suspend configurado)
ALTER WAREHOUSE COMPUTE_WH SUSPEND;
```

> **⚠️ Importante para cuentas trial:** Los créditos Snowflake se consumen mientras el warehouse está activo, incluso si no se ejecutan consultas. Siempre suspende el warehouse al terminar cada sesión de trabajo.

```sql
-- Paso 3 (Opcional): Verificar que el warehouse fue suspendido
SHOW WAREHOUSES LIKE 'COMPUTE_WH';
-- La columna STATE debe mostrar 'SUSPENDED'
```

**No es necesario eliminar tablas ni datos.** El dataset `CURSO_SQL` se utilizará en laboratorios posteriores. Solo se deben cerrar las conexiones activas y suspender el warehouse.

---

## Resumen

### Conceptos Aplicados en Este Laboratorio

| Función          | Propósito                                      | Comportamiento con NULL   | Ejemplo de uso                          |
|------------------|------------------------------------------------|---------------------------|-----------------------------------------|
| `COUNT(*)`       | Contar todas las filas                         | Incluye filas con NULL    | Total de transacciones                  |
| `COUNT(columna)` | Contar filas con valor no nulo                 | Excluye NULLs             | Transacciones con cliente asignado      |
| `COUNT(DISTINCT)`| Contar valores únicos                          | Excluye NULLs             | Número de clientes únicos               |
| `SUM(columna)`   | Sumar valores numéricos                        | Ignora NULLs              | Ingreso total                           |
| `AVG(columna)`   | Calcular promedio de valores no nulos          | Ignora NULLs              | Ticket promedio de venta                |
| `MIN(columna)`   | Valor mínimo (numérico, texto o fecha)         | Ignora NULLs              | Venta más pequeña, fecha más antigua    |
| `MAX(columna)`   | Valor máximo (numérico, texto o fecha)         | Ignora NULLs              | Venta más grande, fecha más reciente    |

### Patrones SQL Clave Aprendidos

```sql
-- Patrón 1: Conteo con diagnóstico de NULLs
SELECT COUNT(*), COUNT(columna), COUNT(*) - COUNT(columna) AS nulos
FROM tabla;

-- Patrón 2: Métrica filtrada con WHERE
SELECT COUNT(*), SUM(columna_numerica)
FROM tabla
WHERE condicion = 'valor';

-- Patrón 3: Dashboard integrador con alias descriptivos
SELECT
    COUNT(*)                   AS total_registros,
    SUM(columna_numerica)      AS total_valor,
    ROUND(AVG(columna_numerica), 2) AS promedio,
    MIN(columna_numerica)      AS valor_minimo,
    MAX(columna_numerica)      AS valor_maximo
FROM tabla;
```

### Buenas Prácticas Reforzadas

1. **Siempre usa alias descriptivos** con `AS` en funciones de agregación. `COUNT(*)` como nombre de columna en un reporte es ambiguo; `total_ventas` es autoexplicativo.
2. **Distingue `COUNT(*)` de `COUNT(columna)`** según tu intención: si quieres el total de filas, usa `COUNT(*)`; si quieres detectar NULLs, compara ambos.
3. **Verifica la coherencia** de tus métricas con validaciones cruzadas simples (ej. el promedio debe estar entre el mínimo y el máximo).
4. **Recuerda el orden lógico de ejecución**: `WHERE` filtra las filas **antes** de que las funciones de agregación calculen sus resultados.
5. **Usa `ROUND`** para controlar la precisión decimal de `AVG` y mejorar la legibilidad de los resultados.

### Próximos Pasos

En el **Lab 06**, aplicarás estas mismas funciones de agregación en combinación con `GROUP BY` para calcular métricas **por categoría, por estado o por período**. Por ejemplo, en lugar de obtener el ingreso total de todas las ventas (como hiciste aquí), calcularás el ingreso total **por categoría de producto** o **por mes**. Las funciones que dominaste en este laboratorio son exactamente las mismas; solo cambia la forma en que se agrupan los datos.

---

## Recursos Adicionales

- [Documentación oficial de COUNT en Snowflake](https://docs.snowflake.com/en/sql-reference/functions/count)
- [Documentación oficial de SUM en Snowflake](https://docs.snowflake.com/en/sql-reference/functions/sum)
- [Documentación oficial de AVG en Snowflake](https://docs.snowflake.com/en/sql-reference/functions/avg)
- [Documentación oficial de MIN en Snowflake](https://docs.snowflake.com/en/sql-reference/functions/min)
- [Documentación oficial de MAX en Snowflake](https://docs.snowflake.com/en/sql-reference/functions/max)
- [Guía de funciones de agregación — W3Schools](https://www.w3schools.com/sql/sql_count_avg_sum.asp)
- [Tutorial interactivo de agregaciones — SQLZoo](https://sqlzoo.net/wiki/SUM_and_COUNT)

---
*Lab 05-00-01 — Versión 1.0 — Dataset CURSO_SQL v1.0*

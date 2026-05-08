# Análisis por categorías

## Metadatos

| Campo            | Detalle                                      |
|------------------|----------------------------------------------|
| **Duración**     | 45 minutos                                   |
| **Complejidad**  | Media                                        |
| **Nivel Bloom**  | Aplicar (Apply)                              |
| **Laboratorio**  | 06-00-01                                     |
| **Dependencias** | Lab 05-00-01 (Funciones de Agregación)       |

---

## Descripción General

En este laboratorio aplicarás `GROUP BY` junto con funciones de agregación para segmentar datos de ventas y productos por categorías, regiones y otros criterios de negocio. Aprenderás a filtrar grupos usando `HAVING` y a distinguir cuándo usar `WHERE` versus `HAVING` en una consulta. El laboratorio culmina con la construcción de un ranking de categorías por desempeño que integra `GROUP BY`, `HAVING` y `ORDER BY` en una sola consulta de análisis.

---

## Objetivos de Aprendizaje

Al finalizar este laboratorio, serás capaz de:

- [ ] Aplicar `GROUP BY` para agrupar registros por una o más columnas y calcular métricas por cada grupo.
- [ ] Utilizar funciones de agregación (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) en combinación con `GROUP BY` para generar resúmenes por categoría.
- [ ] Emplear `HAVING` para filtrar grupos de resultados basándose en condiciones sobre valores agregados.
- [ ] Distinguir el uso correcto de `WHERE` (filtro de filas antes de agrupar) versus `HAVING` (filtro de grupos después de agrupar).
- [ ] Construir consultas de análisis categórico que respondan preguntas de negocio sobre segmentos de datos.

---

## Prerrequisitos

### Conocimientos Previos

- Haber completado el **Lab 05-00-01** o demostrar dominio de las funciones de agregación `COUNT`, `SUM`, `AVG`, `MIN` y `MAX`.
- Comprensión del concepto de agrupación (similar a las tablas dinámicas en hojas de cálculo como Excel o Google Sheets).
- Familiaridad con las cláusulas `SELECT`, `FROM`, `WHERE` y `ORDER BY`.

### Acceso y Recursos

- Cuenta activa en Snowflake (trial individual o corporativa con schema propio).
- Base de datos `CURSO_SQL` inicializada correctamente con el script provisto por el instructor.
- Acceso a la interfaz **Snowsight** desde un navegador moderno (Chrome 90+, Firefox 88+, Edge 90+ o Safari 14+).
- JavaScript habilitado en el navegador (requerido por Snowsight).

---

## Entorno del Laboratorio

### Requisitos de Hardware y Software

| Componente         | Requisito Mínimo                                      |
|--------------------|-------------------------------------------------------|
| Conexión a internet| 10 Mbps estables                                      |
| Sistema operativo  | Windows 10+, macOS 10.15+, o Linux moderno            |
| Navegador          | Chrome 90+ / Firefox 88+ / Edge 90+ / Safari 14+      |
| Resolución         | 1280×768 píxeles o superior                           |
| RAM                | 4 GB mínimo                                           |
| Snowflake          | Edición Standard o superior, interfaz Snowsight       |

### Configuración del Entorno

Antes de comenzar los ejercicios, ejecuta los siguientes comandos de configuración en un nuevo Worksheet de Snowsight. Esto garantiza que todas las consultas del laboratorio apunten al contexto correcto.

> **💡 Consejo de costos:** Usa un Virtual Warehouse de tamaño `X-Small` para minimizar el consumo de créditos en tu cuenta trial.

1. Clic en la sección **Projects** y luego clic en **Workspaces**.
2. Abre tu **Workspace** llamado **SnowEssLAbs**.
3. Crea un archivo de tipo SQL llamado **`Lab06_Analisis_Categorias`**.
4. Ejecuta los siguientes comandos para asegurarte de estar trabajando en el contexto correcto:

```sql
-- Paso 1: Seleccionar el rol de trabajo
USE ROLE SYSADMIN;

-- Paso 2: Seleccionar el Virtual Warehouse (tamaño X-Small recomendado)
USE WAREHOUSE CURSO_WH;

-- Paso 3: Seleccionar la base de datos del curso
USE DATABASE CURSO_SQL;

-- Paso 4: Seleccionar el schema correspondiente
-- En cuenta trial individual:
USE SCHEMA PUBLIC;

-- Paso 5: Verificar que las tablas del curso estén disponibles
SHOW TABLES;
```

**Resultado esperado del paso 5:** Deberías ver al menos las tablas `VENTAS` y `PRODUCTOS` en la lista. Si no aparecen, notifica a tu instructor para ejecutar el script de inicialización del dataset.

```sql
-- Paso 6 (opcional): Vista previa rápida de las tablas principales
SELECT * FROM VENTAS LIMIT 5;
SELECT * FROM PRODUCTOS LIMIT 5;
```

Tómate un momento para familiarizarte con las columnas disponibles en cada tabla antes de continuar. Anota mentalmente qué columnas son numéricas (candidatas para `SUM`, `AVG`) y cuáles son categóricas (candidatas para `GROUP BY`).

---

## Instrucciones Paso a Paso

---

### Paso 1 — Exploración de la Estructura de Datos

**Objetivo:** Comprender la estructura de las tablas `VENTAS` y `PRODUCTOS` para identificar qué columnas son útiles para agrupar y cuáles para agregar.

#### Instrucciones

1. Ejecuta la siguiente consulta para explorar la estructura completa de la tabla `VENTAS`:

```sql
-- Exploración de la tabla VENTAS
DESCRIBE TABLE VENTAS;
```

4. Luego, obtén una muestra de los primeros registros para ver datos reales:

```sql
-- Muestra de datos de VENTAS
SELECT *
FROM VENTAS
LIMIT 10;
```

5. Repite el proceso para la tabla `PRODUCTOS`:

```sql
-- Exploración de la tabla PRODUCTOS
DESCRIBE TABLE PRODUCTOS;

-- Muestra de datos de PRODUCTOS
SELECT *
FROM PRODUCTOS
LIMIT 10;
```

6. Finalmente, obtén un conteo total de registros en cada tabla para dimensionar el dataset:

```sql
-- Conteo total de registros
SELECT
    'VENTAS'    AS tabla,
    COUNT(*)    AS total_registros
FROM VENTAS

UNION ALL

SELECT
    'PRODUCTOS' AS tabla,
    COUNT(*)    AS total_registros
FROM PRODUCTOS;
```

#### Resultado Esperado

El `DESCRIBE` de `VENTAS` debe mostrar columnas como `ID_VENTA`, `FECHA_VENTA`, `ID_PRODUCTO`, `CANTIDAD`, `PRECIO_UNITARIO`, `MONTO_TOTAL`, `REGION`, `VENDEDOR`, entre otras. El `DESCRIBE` de `PRODUCTOS` debe mostrar columnas como `ID_PRODUCTO`, `NOMBRE_PRODUCTO`, `CATEGORIA`, `PRECIO`, `STOCK`.

El conteo final debe mostrar dos filas, una por tabla, con el número total de registros en cada una.

#### Verificación

Confirma que puedes responder estas preguntas antes de continuar:
- ¿Qué columna de `VENTAS` usarías para agrupar por zona geográfica?
- ¿Qué columna de `PRODUCTOS` usarías para agrupar por tipo de producto?
- ¿Qué columna numérica de `VENTAS` usarías para calcular ingresos totales?

---

### Paso 2 — Primera Agrupación: Ventas por Región

**Objetivo:** Aplicar `GROUP BY` por primera vez para obtener un resumen de ventas agrupado por región geográfica, utilizando `COUNT` y `SUM`.

#### Instrucciones

1. Escribe y ejecuta la siguiente consulta para contar el número de transacciones por región:

```sql
-- Consulta 2.1: Número de ventas por región
SELECT
    REGION,
    COUNT(*)            AS total_transacciones
FROM VENTAS
GROUP BY REGION;
```

2. Observa el resultado. Ahora amplía la consulta para incluir el monto total de ventas y el monto promedio por transacción:

```sql
-- Consulta 2.2: Resumen completo de ventas por región
SELECT
    REGION,
    COUNT(*)                        AS total_transacciones,
    SUM(TOTAL_VENTA)                AS ingreso_total,
    AVG(TOTAL_VENTA)                AS ticket_promedio,
    MIN(TOTAL_VENTA)                AS venta_minima,
    MAX(TOTAL_VENTA)                AS venta_maxima
FROM VENTAS
GROUP BY REGION
ORDER BY ingreso_total DESC;
```

3. Agrega `ORDER BY` para presentar las regiones de mayor a menor ingreso (ya incluido en la consulta anterior). Observa cómo el `ORDER BY` actúa **después** del `GROUP BY` en el orden de ejecución.

4. Intenta ahora escribir una versión **incorrecta** a propósito para ver el error de Snowflake. Esto te ayudará a reconocer el mensaje de error en el futuro:

```sql
-- Consulta 2.3: INTENCIONALMENTE INCORRECTA - columna no agregada fuera de GROUP BY
-- Ejecuta esto para ver el error; NO es una consulta válida
SELECT
    REGION,
    VENDEDOR,           -- Esta columna no está en GROUP BY ni es una agregación
    COUNT(*)            AS total_transacciones
FROM VENTAS
GROUP BY REGION;        -- Falta VENDEDOR aquí
```

5. Lee el mensaje de error que genera Snowflake. Luego **corrige** la consulta de una de estas dos formas:

```sql
-- Opción A: Agregar VENDEDOR al GROUP BY
SELECT
    REGION,
    VENDEDOR,
    COUNT(*)            AS total_transacciones
FROM VENTAS
GROUP BY REGION, VENDEDOR
ORDER BY REGION, total_transacciones DESC;
```
```sql
-- Opción B: Eliminar VENDEDOR del SELECT
SELECT
    REGION,
    COUNT(*)            AS total_transacciones
FROM VENTAS
GROUP BY REGION
ORDER BY total_transacciones DESC;
```

#### Resultado Esperado

La **Consulta 2.2** debe devolver una fila por cada región existente en la tabla `VENTAS`, con cinco métricas calculadas para cada una. Las regiones deben aparecer ordenadas de mayor a menor ingreso total.

La **Consulta 2.3** debe generar un error similar a:
```
SQL compilation error: ... is not a valid group by expression
```

La **Opción A** de la corrección debe mostrar una fila por cada combinación única de `REGION` + `VENDEDOR`.

#### Verificación

- [ ] La Consulta 2.2 devuelve exactamente una fila por región.
- [ ] Los valores de `ingreso_total` son coherentes (mayores que `venta_maxima` individual).
- [ ] La Consulta 2.3 genera un error reconocible.
- [ ] Ambas opciones de corrección ejecutan sin errores.

---

### Paso 3 — Agrupación por Categoría de Producto

**Objetivo:** Combinar las tablas `VENTAS` y `PRODUCTOS` mediante un `JOIN` implícito o explícito para agrupar ventas por categoría de producto y calcular métricas de ingreso por tipo.

> **📌 Nota:** Si aún no has cubierto `JOIN` en el curso, el instructor habrá creado una vista o tabla desnormalizada que incluye la columna `CATEGORIA` directamente en `VENTAS`. Usa la alternativa indicada a continuación.

#### Instrucciones

1. Primero, verifica si la tabla `VENTAS` ya contiene una columna `CATEGORIA`. Si es así, úsala directamente:

```sql
-- Verificación: ¿VENTAS tiene columna CATEGORIA?
SELECT COLUMN_NAME
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'VENTAS'
  AND TABLE_SCHEMA = CURRENT_SCHEMA()
  AND COLUMN_NAME = 'CATEGORIA';
```

2. **Si `VENTAS` contiene `CATEGORIA` directamente**, ejecuta:

```sql
-- Consulta 3.1A: Ingreso promedio por categoría (columna directa)
SELECT
    CATEGORIA,
    COUNT(*)                        AS total_ventas,
    SUM(TOTAL_VENTA)                AS ingreso_total,
    ROUND(AVG(TOTAL_VENTA), 2)      AS ingreso_promedio,
    MIN(TOTAL_VENTA)                AS ingreso_minimo,
    MAX(TOTAL_VENTA)                AS ingreso_maximo
FROM VENTAS
GROUP BY CATEGORIA
ORDER BY ingreso_total DESC;
```

3. **Si `VENTAS` NO contiene `CATEGORIA`**, combina ambas tablas con `JOIN`:

```sql
-- Consulta 3.1B: Ingreso promedio por categoría (con JOIN)
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
```

4. Ahora agrega una métrica adicional: la cantidad total de unidades vendidas por categoría:

```sql
-- Consulta 3.2: Unidades vendidas y participación por categoría
SELECT
    CATEGORIA,
    COUNT(*)                        AS total_transacciones,
    SUM(CANTIDAD)                   AS unidades_vendidas,
    SUM(TOTAL_VENTA)                AS ingreso_total,
    ROUND(AVG(TOTAL_VENTA), 2)      AS ticket_promedio
FROM VENTAS
GROUP BY CATEGORIA
ORDER BY unidades_vendidas DESC;
```

> **Nota:** Ajusta los nombres de columna (`CANTIDAD`, `TOTAL_VENTA`, `CATEGORIA`) según los que existan en tu dataset. Usa `DESCRIBE TABLE VENTAS` si tienes dudas.

5. Interpreta los resultados: ¿Qué categoría genera más ingresos? ¿Coincide con la que tiene más transacciones? ¿Por qué podrían diferir?

#### Resultado Esperado

Las consultas deben devolver una fila por cada categoría de producto con las métricas solicitadas. El número de filas debe coincidir con el número de categorías distintas en el dataset.

Ejemplo de estructura del resultado esperado:

| CATEGORIA     | TOTAL_VENTAS | INGRESO_TOTAL | INGRESO_PROMEDIO | INGRESO_MINIMO | INGRESO_MAXIMO |
|---------------|-------------|---------------|-----------------|----------------|----------------|
| Electrónica   | 245         | 125,430.00    | 512.37          | 15.00          | 2,100.00       |
| Ropa          | 312         | 48,960.00     | 156.92          | 8.00           | 890.00         |
| Hogar         | 189         | 67,200.00     | 355.56          | 20.00          | 1,500.00       |

*(Los valores son ilustrativos; los tuyos variarán según el dataset.)*

#### Verificación

- [ ] El número de filas en el resultado coincide con el número de categorías únicas.
- [ ] `ingreso_total` para cada categoría es mayor que `ingreso_maximo` (lógica básica de consistencia).
- [ ] La consulta no genera errores de columnas fuera del `GROUP BY`.

---

### Paso 4 — Agrupación por Múltiples Columnas

**Objetivo:** Demostrar cómo agrupar por dos columnas simultáneamente para obtener un análisis más granular (por ejemplo, región y categoría).

#### Instrucciones

1. Ejecuta la siguiente consulta para obtener el desglose de ventas por cada combinación de región y categoría:

```sql
-- Consulta 4.1: Ventas por región Y categoría (agrupación múltiple)
SELECT
    REGION,
    CATEGORIA,
    COUNT(*)                        AS total_ventas,
    SUM(TOTAL_VENTA)                AS ingreso_total
FROM VENTAS
GROUP BY REGION, CATEGORIA
ORDER BY REGION ASC, ingreso_total DESC;
```

2. Observa cuántas filas devuelve este resultado comparado con el del Paso 2 (solo por región) y el del Paso 3 (solo por categoría). La agrupación múltiple genera una fila por cada **combinación única** de los valores de ambas columnas.

3. Ahora agrega una tercera columna de agrupación para obtener un análisis mensual por región y categoría. Usa la función `DATE_TRUNC` de Snowflake para extraer el mes de la fecha de venta:

```sql
-- Consulta 4.2: Ventas por mes, región y categoría
SELECT
    DATE_TRUNC('MONTH', FECHA_VENTA)    AS mes_venta,
    REGION,
    CATEGORIA,
    COUNT(*)                            AS total_ventas,
    SUM(TOTAL_VENTA)                    AS ingreso_total
FROM VENTAS
GROUP BY DATE_TRUNC('MONTH', FECHA_VENTA), REGION, CATEGORIA
ORDER BY mes_venta ASC, REGION ASC, ingreso_total DESC;
```

> **💡 Tip de Snowflake:** Puedes usar `DATE_TRUNC('MONTH', columna_fecha)` para agrupar por mes sin necesidad de extraer el año y mes por separado. Esta función es específica de Snowflake y algunos otros motores modernos.

4. Reflexiona sobre la diferencia en el número de filas entre las tres consultas de este laboratorio hasta ahora. ¿Qué conclusión sacas sobre cómo la granularidad de `GROUP BY` afecta el volumen de resultados?

#### Resultado Esperado

La **Consulta 4.1** debe devolver un número de filas igual al producto de (número de regiones) × (número de categorías), asumiendo que todas las combinaciones existen en los datos.

La **Consulta 4.2** debe devolver significativamente más filas, ya que añade la dimensión temporal.

#### Verificación

- [ ] La Consulta 4.1 devuelve más filas que la del Paso 2 y más filas que la del Paso 3.
- [ ] Todas las columnas en el `SELECT` que no son funciones de agregación aparecen en el `GROUP BY`.
- [ ] La Consulta 4.2 ejecuta sin errores y el campo `mes_venta` muestra fechas con día 01 de cada mes.

---

### Paso 5 — Introducción a HAVING: Filtrado de Grupos

**Objetivo:** Aplicar la cláusula `HAVING` para filtrar grupos de resultados basándose en condiciones sobre valores agregados, y comprender por qué `WHERE` no puede cumplir esta función.

#### Instrucciones

1. Primero, intenta usar `WHERE` para filtrar por un valor agregado (esto **fallará** intencionalmente):

```sql
-- Consulta 5.1: INTENCIONALMENTE INCORRECTA - WHERE no puede filtrar agregaciones
-- Ejecuta esto para ver el error
SELECT
    CATEGORIA,
    COUNT(*)        AS total_ventas
FROM VENTAS
WHERE COUNT(*) > 100    -- ERROR: no se puede usar una función de agregación en WHERE
GROUP BY CATEGORIA;
```

2. Lee el mensaje de error. Snowflake indicará que no es válido usar una función de agregación en la cláusula `WHERE`.

3. Ahora usa `HAVING` para lograr el mismo objetivo correctamente:

```sql
-- Consulta 5.2: HAVING para filtrar grupos con más de 100 ventas
SELECT
    CATEGORIA,
    COUNT(*)        AS total_ventas
FROM VENTAS
GROUP BY CATEGORIA
HAVING COUNT(*) > 100
ORDER BY total_ventas DESC;
```

4. Amplía el filtro con `HAVING` para incluir una condición sobre el ingreso total:

```sql
-- Consulta 5.3: Categorías con más de 50 ventas Y con ingreso total superior a 20,000
SELECT
    CATEGORIA,
    COUNT(*)                    AS total_ventas,
    SUM(TOTAL_VENTA)            AS ingreso_total,
    ROUND(AVG(TOTAL_VENTA), 2)  AS ticket_promedio
FROM VENTAS
GROUP BY CATEGORIA
HAVING COUNT(*) > 50
   AND SUM(TOTAL_VENTA) > 20000
ORDER BY ingreso_total DESC;
```

> **📌 Nota:** Ajusta los umbrales (`> 50`, `> 20000`) según los datos de tu dataset. Si el resultado está vacío, reduce los valores hasta obtener al menos 2 filas de resultado.

5. Prueba también con `HAVING` usando el alias del `SELECT`:

```sql
-- Consulta 5.4: Usando alias en HAVING (comportamiento en Snowflake)
SELECT
    CATEGORIA,
    COUNT(*)                    AS total_ventas,
    SUM(TOTAL_VENTA)            AS ingreso_total
FROM VENTAS
GROUP BY CATEGORIA
HAVING total_ventas > 50        -- Snowflake permite usar alias de SELECT en HAVING
ORDER BY ingreso_total DESC;
```

> **⚠️ Nota de compatibilidad:** El uso de alias del `SELECT` en `HAVING` **funciona en Snowflake** pero **no es estándar SQL** y puede fallar en otros motores como MySQL o PostgreSQL. Es una conveniencia de Snowflake que debes conocer, pero úsala con precaución en código portable.

#### Resultado Esperado

La **Consulta 5.1** debe generar un error del tipo:
```
SQL compilation error: aggregate functions are not allowed in filter condition (WHERE clause)
```

Las **Consultas 5.2, 5.3 y 5.4** deben devolver solo las categorías que cumplen los criterios especificados en `HAVING`, con menos filas que la consulta sin filtro del Paso 3.

#### Verificación

- [ ] La Consulta 5.1 genera un error reconocible sobre el uso de agregaciones en `WHERE`.
- [ ] La Consulta 5.2 devuelve solo categorías con más de 100 ventas (o el umbral que hayas ajustado).
- [ ] La Consulta 5.3 devuelve solo las categorías que cumplen **ambas** condiciones del `HAVING`.
- [ ] La Consulta 5.4 produce el mismo resultado que la 5.2 (confirmando que el alias funciona en Snowflake).

---

### Paso 6 — WHERE vs. HAVING: Ejercicio Comparativo

**Objetivo:** Demostrar mediante ejemplos concretos la diferencia fundamental entre `WHERE` (filtra filas antes de agrupar) y `HAVING` (filtra grupos después de agrupar), y aplicar ambos en la misma consulta.

#### Instrucciones

1. Ejecuta estas dos consultas y compara los resultados:

```sql
-- Consulta 6.1: WHERE filtra ANTES de agrupar
-- Pregunta: ¿Cuántas ventas hay por categoría, considerando SOLO el año 2024?
SELECT
    CATEGORIA,
    COUNT(*)            AS total_ventas,
    SUM(TOTAL_VENTA)    AS ingreso_total
FROM VENTAS
WHERE YEAR(FECHA_VENTA) = 2024      -- Filtra filas: solo ventas del año 2024
GROUP BY CATEGORIA
ORDER BY ingreso_total DESC;
```

```sql
-- Consulta 6.2: Sin WHERE (incluye todos los años)
-- Pregunta: ¿Cuántas ventas hay por categoría en TODOS los años?
SELECT
    CATEGORIA,
    COUNT(*)            AS total_ventas,
    SUM(TOTAL_VENTA)    AS ingreso_total
FROM VENTAS
GROUP BY CATEGORIA
ORDER BY ingreso_total DESC;
```

2. Compara los resultados. La Consulta 6.1 debe tener valores menores de `total_ventas` e `ingreso_total` porque `WHERE` eliminó filas de otros años antes de que `GROUP BY` agrupara.

3. Ahora combina `WHERE` y `HAVING` en una sola consulta:

```sql
-- Consulta 6.3: WHERE + HAVING en la misma consulta
-- Pregunta: En el año 2024, ¿qué categorías superaron las 30 ventas?
SELECT
    CATEGORIA,
    COUNT(*)                    AS total_ventas,
    SUM(TOTAL_VENTA)            AS ingreso_total,
    ROUND(AVG(TOTAL_VENTA), 2)  AS ticket_promedio
FROM VENTAS
WHERE YEAR(FECHA_VENTA) = 2024      -- WHERE: filtra filas (solo 2024)
GROUP BY CATEGORIA
HAVING COUNT(*) > 30                -- HAVING: filtra grupos (solo los que superan 30 ventas)
ORDER BY total_ventas DESC;
```

4. Para consolidar la comprensión, observa el siguiente diagrama conceptual del orden de ejecución:

```
Orden de ejecución lógico en SQL:
┌─────────┐    ┌─────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  FROM   │ →  │  WHERE  │ →  │ GROUP BY │ →  │  HAVING  │ →  │  SELECT  │
│(tablas) │    │(filas)  │    │(grupos)  │    │(grupos)  │    │(columnas)│
└─────────┘    └─────────┘    └──────────┘    └──────────┘    └──────────┘
                    ↑                               ↑
              Filtra filas                    Filtra grupos
              individuales                    formados por
              ANTES de                        GROUP BY
              agrupar
```

5. Escribe en los comentarios de tu Worksheet una frase que explique con tus propias palabras la diferencia entre `WHERE` y `HAVING`. Esto te ayudará a recordarlo.

#### Resultado Esperado

La **Consulta 6.1** debe devolver los mismos grupos (categorías) que la 6.2, pero con valores numéricos menores o iguales, ya que parte de los datos fue excluida por el `WHERE`.

La **Consulta 6.3** combina ambas cláusulas y devuelve solo las categorías del año 2024 que superaron el umbral de ventas definido en `HAVING`.

#### Verificación

- [ ] Los valores de `total_ventas` en la Consulta 6.1 son menores o iguales a los de la Consulta 6.2 para cada categoría.
- [ ] La Consulta 6.3 ejecuta sin errores y devuelve un subconjunto de los resultados de la Consulta 6.1.
- [ ] Puedes articular verbalmente la diferencia entre `WHERE` y `HAVING`.

---

### Paso 7 — Ranking de Categorías por Desempeño (Consulta Integradora)

**Objetivo:** Construir una consulta de análisis completo que integre `GROUP BY`, múltiples funciones de agregación, `HAVING` y `ORDER BY` para producir un ranking de categorías por desempeño de negocio.

#### Instrucciones

1. Construye la consulta de ranking progresivamente. Comienza con la base:

```sql
-- Consulta 7.1: Base del ranking - todas las categorías con sus métricas
SELECT
    CATEGORIA,
    COUNT(*)                        AS total_ventas,
    SUM(CANTIDAD)                   AS unidades_vendidas,
    SUM(TOTAL_VENTA)                AS ingreso_total,
    ROUND(AVG(TOTAL_VENTA), 2)      AS ticket_promedio,
    MIN(TOTAL_VENTA)                AS venta_minima,
    MAX(TOTAL_VENTA)                AS venta_maxima
FROM VENTAS
GROUP BY CATEGORIA
ORDER BY ingreso_total DESC;
```

2. Ahora aplica un filtro de relevancia con `HAVING` para excluir categorías con muy pocas ventas (que podrían distorsionar el análisis):

```sql
-- Consulta 7.2: Ranking filtrado - solo categorías con actividad significativa
SELECT
    CATEGORIA,
    COUNT(*)                        AS total_ventas,
    SUM(CANTIDAD)                   AS unidades_vendidas,
    SUM(TOTAL_VENTA)                AS ingreso_total,
    ROUND(AVG(TOTAL_VENTA), 2)      AS ticket_promedio,
    MIN(TOTAL_VENTA)                AS venta_minima,
    MAX(TOTAL_VENTA)                AS venta_maxima
FROM VENTAS
GROUP BY CATEGORIA
HAVING COUNT(*) >= 20               -- Solo categorías con al menos 20 ventas
ORDER BY ingreso_total DESC;
```

3. Añade un contexto temporal con `WHERE` para enfocar el análisis en el período más reciente:

```sql
-- Consulta 7.3: Ranking completo - período reciente, categorías relevantes
SELECT
    CATEGORIA,
    COUNT(*)                        AS total_ventas,
    SUM(CANTIDAD)                   AS unidades_vendidas,
    ROUND(SUM(TOTAL_VENTA), 2)      AS ingreso_total,
    ROUND(AVG(TOTAL_VENTA), 2)      AS ticket_promedio,
    ROUND(MIN(TOTAL_VENTA), 2)      AS venta_minima,
    ROUND(MAX(TOTAL_VENTA), 2)      AS venta_maxima
FROM VENTAS
WHERE FECHA_VENTA >= DATEADD('year', -2, CURRENT_DATE())  -- Último año
GROUP BY CATEGORIA
HAVING COUNT(*) >= 20               -- Solo categorías con actividad significativa
ORDER BY ingreso_total DESC;        -- Ranking por ingreso total
```

> **💡 Función de Snowflake:** `DATEADD('year', -1, CURRENT_DATE())` devuelve la fecha de hace exactamente un año desde hoy. Es equivalente a `CURRENT_DATE() - INTERVAL '1 YEAR'` en SQL estándar.

4. Agrega una columna calculada que muestre el **rango** de precio (diferencia entre máximo y mínimo) como indicador de la amplitud del portafolio de cada categoría:

```sql
-- Consulta 7.4: Ranking final con métricas adicionales de análisis
SELECT
    CATEGORIA,
    COUNT(*)                                AS total_ventas,
    SUM(CANTIDAD)                           AS unidades_vendidas,
    ROUND(SUM(TOTAL_VENTA), 2)              AS ingreso_total,
    ROUND(AVG(TOTAL_VENTA), 2)              AS ticket_promedio,
    ROUND(MAX(TOTAL_VENTA)
        - MIN(TOTAL_VENTA), 2)              AS rango_precio,
    ROUND(MAX(TOTAL_VENTA), 2)              AS venta_maxima
FROM VENTAS
WHERE FECHA_VENTA >= DATEADD('year', -2, CURRENT_DATE())
GROUP BY CATEGORIA
HAVING COUNT(*) >= 20
ORDER BY ingreso_total DESC;
```

5. Guarda esta consulta en tu Worksheet. Es un ejemplo de análisis categórico completo que podrías reutilizar en proyectos reales.

#### Resultado Esperado

La **Consulta 7.4** debe devolver una tabla ordenada de mayor a menor ingreso, mostrando solo las categorías con actividad en el último año y con al menos 20 ventas. Cada fila representa una categoría con seis métricas de desempeño calculadas.

Ejemplo ilustrativo del resultado:

| CATEGORIA    | TOTAL_VENTAS | UNIDADES_VENDIDAS | INGRESO_TOTAL | TICKET_PROMEDIO | RANGO_PRECIO | VENTA_MAXIMA |
|--------------|-------------|-------------------|---------------|----------------|--------------|--------------|
| Electrónica  | 198         | 312               | 98,450.00     | 497.22         | 2,085.00     | 2,100.00     |
| Hogar        | 145         | 289               | 54,320.00     | 374.62         | 1,480.00     | 1,500.00     |
| Ropa         | 267         | 534               | 41,760.00     | 156.40         | 882.00       | 890.00       |

*(Los valores son ilustrativos.)*

#### Verificación

- [ ] La Consulta 7.4 ejecuta sin errores.
- [ ] El resultado está ordenado de mayor a menor `ingreso_total`.
- [ ] Solo aparecen categorías con al menos 20 ventas en el período especificado.
- [ ] La columna `rango_precio` muestra valores positivos (MAX > MIN).

---

### Paso 8 — Análisis por Región con Filtro Combinado

**Objetivo:** Aplicar los conceptos aprendidos para responder una pregunta de negocio concreta: identificar las regiones con mejor desempeño de ventas en un período específico.

#### Instrucciones

1. Responde la siguiente pregunta de negocio: **"¿Cuáles son las regiones con más de 50 ventas en el último año, ordenadas por ticket promedio de mayor a menor?"**

```sql
-- Consulta 8.1: Regiones de alto desempeño por ticket promedio
SELECT
    REGION,
    COUNT(*)                        AS total_ventas,
    ROUND(SUM(TOTAL_VENTA), 2)      AS ingreso_total,
    ROUND(AVG(TOTAL_VENTA), 2)      AS ticket_promedio,
    ROUND(MAX(TOTAL_VENTA), 2)      AS venta_maxima
FROM VENTAS
WHERE FECHA_VENTA >= DATEADD('year', -2, CURRENT_DATE())
GROUP BY REGION
HAVING COUNT(*) > 50
ORDER BY ticket_promedio DESC;
```

2. Ahora responde: **"¿Qué vendedor por región tiene el mayor ingreso total, considerando solo ventas mayores a $50?"**

```sql
-- Consulta 8.2: Top vendedores por región (ventas > $50)
SELECT
    REGION,
    VENDEDOR,
    COUNT(*)                        AS total_ventas,
    ROUND(SUM(TOTAL_VENTA), 2)      AS ingreso_total
FROM VENTAS
WHERE TOTAL_VENTA > 50              -- WHERE filtra filas individuales (ventas > $50)
GROUP BY REGION, VENDEDOR
HAVING SUM(TOTAL_VENTA) > 5000      -- HAVING filtra grupos (ingreso acumulado > $5,000)
ORDER BY REGION ASC, ingreso_total DESC;
```

> **Ajuste de umbrales:** Si la Consulta 8.2 devuelve cero filas, reduce el umbral de `HAVING` (por ejemplo, a `> 1000`) hasta obtener resultados. Los valores exactos dependen de tu dataset.

3. Reflexiona sobre el resultado de la Consulta 8.2. Para cada región, ¿aparecen múltiples vendedores o solo uno? ¿Qué significa eso para el análisis de negocio?

#### Resultado Esperado

La **Consulta 8.1** debe devolver solo las regiones con más de 50 ventas en el último año, ordenadas por ticket promedio descendente.

La **Consulta 8.2** debe devolver una fila por cada combinación región-vendedor que cumpla ambos criterios: ventas individuales mayores a $50 y un ingreso acumulado superior al umbral del `HAVING`.

#### Verificación

- [ ] La Consulta 8.1 devuelve menos filas que el total de regiones en el dataset (por el filtro de `HAVING`).
- [ ] La Consulta 8.2 agrupa correctamente por dos columnas sin errores.
- [ ] Ambas consultas combinan `WHERE` y `HAVING` en la misma query.

---

## Validación y Pruebas

Ejecuta las siguientes consultas de validación para confirmar que todos los ejercicios del laboratorio se completaron correctamente.

### Prueba de Validación 1: Consistencia de Agregaciones

```sql
-- VALIDACIÓN 1: El SUM total de todas las categorías debe igualar el SUM de toda la tabla
-- Resultado esperado: ambas columnas deben tener el mismo valor

SELECT
    (SELECT ROUND(SUM(TOTAL_VENTA), 2) FROM VENTAS)                 AS total_tabla_completa,
    (SELECT ROUND(SUM(ingreso_cat), 2)
     FROM (
         SELECT SUM(TOTAL_VENTA) AS ingreso_cat
         FROM VENTAS
         GROUP BY CATEGORIA
     ))                                                              AS suma_de_grupos;
```

**Resultado esperado:** Ambas columnas deben mostrar el mismo valor. Si difieren, hay un error en el dataset o en la comprensión del agrupamiento.

### Prueba de Validación 2: Conteo de Grupos

```sql
-- VALIDACIÓN 2: El número de grupos debe coincidir con los valores únicos de la columna
-- Resultado esperado: ambas columnas deben tener el mismo valor

SELECT
    (SELECT COUNT(DISTINCT CATEGORIA) FROM VENTAS)  AS categorias_unicas,
    (SELECT COUNT(*)
     FROM (
         SELECT CATEGORIA
         FROM VENTAS
         GROUP BY CATEGORIA
     ))                                              AS grupos_generados;
```

**Resultado esperado:** Ambos valores deben ser idénticos, confirmando que `GROUP BY` genera exactamente un grupo por valor único.

### Prueba de Validación 3: Verificación de HAVING

```sql
-- VALIDACIÓN 3: Todos los grupos devueltos por HAVING deben cumplir la condición
-- Resultado esperado: la columna 'cumple_condicion' debe ser TRUE en todas las filas

SELECT
    CATEGORIA,
    COUNT(*)                AS total_ventas,
    (COUNT(*) > 20)         AS cumple_condicion     -- Debe ser TRUE en todas las filas
FROM VENTAS
GROUP BY CATEGORIA
HAVING COUNT(*) > 20
ORDER BY total_ventas DESC;
```

**Resultado esperado:** La columna `cumple_condicion` debe mostrar `TRUE` en todas las filas devueltas.

---

## Solución de Problemas

### Problema 1: Error "is not a valid group by expression"

**Síntoma:**
Al ejecutar una consulta con `GROUP BY`, Snowflake devuelve un error similar a:
```
SQL compilation error: 'VENTAS.VENDEDOR' is not a valid group by expression
```

**Causa:**
Una columna que aparece en el `SELECT` no es una función de agregación y tampoco está incluida en la cláusula `GROUP BY`. Snowflake no puede determinar qué valor de esa columna mostrar para el grupo, ya que puede haber múltiples valores diferentes dentro del mismo grupo.

**Solución:**
Tienes dos opciones:

*Opción A:* Agrega la columna problemática al `GROUP BY`:
```sql
-- Antes (error):
SELECT REGION, VENDEDOR, COUNT(*) AS total
FROM VENTAS
GROUP BY REGION;

-- Después (correcto):
SELECT REGION, VENDEDOR, COUNT(*) AS total
FROM VENTAS
GROUP BY REGION, VENDEDOR;  -- VENDEDOR agregado al GROUP BY
```

*Opción B:* Elimina la columna del `SELECT` si no la necesitas en el resultado:
```sql
-- Después (correcto):
SELECT REGION, COUNT(*) AS total
FROM VENTAS
GROUP BY REGION;
```

---

### Problema 2: HAVING no devuelve resultados (resultado vacío)

**Síntoma:**
La consulta ejecuta sin errores pero devuelve 0 filas, incluso cuando se sabe que existen datos en la tabla.

**Causa:**
El umbral definido en `HAVING` es demasiado alto para los datos disponibles en el dataset. Por ejemplo, `HAVING COUNT(*) > 1000` cuando ninguna categoría tiene más de 300 ventas. También puede ocurrir si el `WHERE` previo ya filtró demasiadas filas, dejando grupos con muy pocos registros.

**Solución:**

Primero, ejecuta la consulta **sin `HAVING`** para ver los valores reales de la agregación:
```sql
-- Paso diagnóstico: ver los valores reales antes de filtrar
SELECT
    CATEGORIA,
    COUNT(*)            AS total_ventas,
    SUM(TOTAL_VENTA)    AS ingreso_total
FROM VENTAS
WHERE FECHA_VENTA >= DATEADD('year', -1, CURRENT_DATE())
GROUP BY CATEGORIA
-- Coloca aquí el HAVING
ORDER BY total_ventas DESC;
```

Con los valores reales visibles, ajusta el umbral del `HAVING` a un valor que tenga sentido para tus datos:
```sql
-- Ajusta el umbral según los valores que observaste
HAVING COUNT(*) > 20    -- Reduce desde 1000 a un valor realista
```

Si el problema persiste con el `WHERE` activo, verifica que el rango de fechas del `WHERE` contiene datos ejecutando:
```sql
SELECT MIN(FECHA_VENTA), MAX(FECHA_VENTA) FROM VENTAS;
```

---

## Limpieza del Entorno

Al finalizar el laboratorio, realiza los siguientes pasos para liberar recursos y evitar consumo innecesario de créditos en tu cuenta Snowflake.

```sql
-- Paso 1: Suspender el Virtual Warehouse para detener el consumo de créditos
-- IMPORTANTE: Ejecuta esto al terminar cada sesión de trabajo
ALTER WAREHOUSE CURSO_WH SUSPEND;
```

```sql
-- Paso 2 (opcional): Verificar que el warehouse quedó suspendido
SHOW WAREHOUSES LIKE 'CURSO_WH';
-- La columna 'state' debe mostrar 'SUSPENDED'
```

> **⚠️ Importante para cuentas trial:** Los créditos de una cuenta trial son limitados. Suspender el warehouse al terminar cada sesión puede extender significativamente la vida útil de tu cuenta. El warehouse se reactiva automáticamente la próxima vez que ejecutes una consulta.

> **Nota:** Las consultas y resultados guardados en tus Worksheets de Snowsight **no se eliminan** al suspender el warehouse. Tu trabajo queda guardado automáticamente en la nube.

---

## Resumen

### Conceptos Cubiertos

En este laboratorio aplicaste los conceptos fundamentales del análisis categórico con SQL en Snowflake:

| Concepto                        | Descripción                                                                 | Ejemplo Clave                              |
|---------------------------------|-----------------------------------------------------------------------------|--------------------------------------------|
| `GROUP BY` simple               | Agrupa registros por una columna y calcula métricas por grupo               | Ventas por región                          |
| `GROUP BY` múltiple             | Agrupa por la combinación de dos o más columnas                             | Ventas por región y categoría              |
| Funciones de agregación         | `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` operando sobre cada grupo               | Ingreso total y ticket promedio            |
| `HAVING`                        | Filtra grupos después de la agregación, basándose en valores calculados     | Solo categorías con más de 20 ventas       |
| `WHERE` vs. `HAVING`            | `WHERE` filtra filas antes de agrupar; `HAVING` filtra grupos ya formados   | `WHERE año=2024` + `HAVING COUNT>30`       |
| Orden de ejecución SQL          | FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY                       | Clave para entender qué filtra cada cláusula|
| Consulta de análisis completo   | Combinación de `WHERE`, `GROUP BY`, `HAVING` y `ORDER BY` en una sola query | Ranking de categorías por desempeño        |

### Diferencias Clave a Recordar

```
WHERE  → Actúa ANTES de GROUP BY → Filtra FILAS individuales
HAVING → Actúa DESPUÉS de GROUP BY → Filtra GRUPOS formados
```

### Buenas Prácticas Aplicadas

- ✅ Usar alias descriptivos (`AS total_ventas`, `AS ingreso_total`) para mejorar la legibilidad.
- ✅ Aplicar `ROUND()` a los resultados de `AVG` y `SUM` para presentar valores monetarios limpios.
- ✅ Combinar `ORDER BY` con `GROUP BY` para producir rankings legibles.
- ✅ Verificar umbrales de `HAVING` ejecutando primero la consulta sin filtro.
- ✅ Incluir en el `GROUP BY` todas las columnas no agregadas del `SELECT`.

### Próximos Pasos

El siguiente laboratorio introducirá las **subconsultas (subqueries)**, que te permitirán usar el resultado de una consulta como entrada de otra. Esto abrirá la puerta a análisis más complejos, como comparar el desempeño de cada categoría contra el promedio general, o identificar los productos por encima del ticket promedio de su categoría.

### Recursos Adicionales

- [Documentación oficial de Snowflake: GROUP BY](https://docs.snowflake.com/en/sql-reference/constructs/group-by)
- [Documentación oficial de Snowflake: HAVING](https://docs.snowflake.com/en/sql-reference/constructs/having)
- [Referencia de funciones de agregación en Snowflake](https://docs.snowflake.com/en/sql-reference/functions-aggregation)
- [Documentación de DATEADD en Snowflake](https://docs.snowflake.com/en/sql-reference/functions/dateadd)
- [Tutorial interactivo de SQL con GROUP BY — Mode Analytics](https://mode.com/sql-tutorial/sql-group-by/)
- [Guía de W3Schools: SQL GROUP BY con ejemplos](https://www.w3schools.com/sql/sql_groupby.asp)

---

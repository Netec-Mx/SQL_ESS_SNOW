# Transformación de datos

## Metadatos

| Campo            | Detalle                        |
|------------------|--------------------------------|
| **Duración**     | 45 minutos                     |
| **Complejidad**  | Media                          |
| **Nivel Bloom**  | Aplicar (Apply)                |
| **Módulo**       | 4 — Transformación de datos    |
| **Versión**      | 1.0                            |

---

## Descripción General

En este laboratorio aplicarás funciones escalares de texto y numéricas directamente dentro de consultas `SELECT` para transformar y estandarizar datos sin modificar los registros originales. Trabajarás sobre las tablas `CLIENTES` y `PRODUCTOS` de la base de datos `CURSO_SQL`, utilizando `UPPER`, `LOWER`, `CONCAT` y `ROUND` para resolver problemas reales de presentación y limpieza de datos. Al finalizar, habrás construido consultas que generan columnas calculadas con alias descriptivos, combinando múltiples funciones en una sola instrucción `SELECT`.

---

## Objetivos de Aprendizaje

Al completar este laboratorio, serás capaz de:

- [ ] Aplicar `UPPER` y `LOWER` para normalizar la capitalización de columnas de texto en resultados de consulta.
- [ ] Utilizar `CONCAT` para combinar valores de múltiples columnas en una sola cadena de texto descriptiva.
- [ ] Emplear `ROUND` para controlar la precisión decimal de valores numéricos en columnas calculadas.
- [ ] Nombrar correctamente las columnas calculadas usando el alias `AS` para mejorar la legibilidad del resultado.
- [ ] Integrar múltiples funciones de transformación en una sola consulta `SELECT` para resolver casos de análisis reales.

---

## Prerrequisitos

### Conocimientos Previos

- Haber completado **Lab 01-00-01** o demostrar dominio de `SELECT` con columnas específicas y alias `AS`.
- Comprensión básica de qué es una función en programación (recibe un valor, devuelve un resultado transformado).
- Haber revisado el contenido de la **Lección 4.1** (Funciones de texto: `UPPER` y `LOWER`).

### Acceso y Recursos

- Cuenta activa en Snowflake (trial o corporativa) con acceso a la interfaz **Snowsight**.
- Base de datos `CURSO_SQL` inicializada correctamente (script de setup ejecutado por el instructor).
- Virtual Warehouse de tamaño **X-Small** disponible y activo.
- Navegador web moderno (Chrome 90+, Firefox 88+, Edge 90+ o Safari 14+) con JavaScript habilitado.

---

## Entorno de Laboratorio

### Especificaciones de Hardware y Software

| Componente         | Requisito Mínimo                                      |
|--------------------|-------------------------------------------------------|
| **Procesador**     | Cualquier CPU moderna (el procesamiento ocurre en Snowflake) |
| **RAM**            | 4 GB (para el navegador con múltiples pestañas)       |
| **Conexión**       | Internet estable, mínimo 10 Mbps                      |
| **Sistema Operativo** | Windows 10+, macOS 10.15+, o Linux moderno         |
| **Navegador**      | Chrome 90+ / Firefox 88+ / Edge 90+ / Safari 14+      |
| **Resolución**     | Mínimo 1280×768 píxeles                               |
| **Plataforma SQL** | Snowflake Snowsight (Worksheets)                      |
| **Dataset**        | `CURSO_SQL` v1.0 (provisto por el instructor)         |

### Configuración Inicial del Entorno

Antes de comenzar los ejercicios, ejecuta los siguientes comandos en una nueva hoja de trabajo (*Worksheet*) en Snowsight para establecer el contexto correcto de la sesión.

**Paso de configuración — Establecer contexto de sesión:**

```sql
-- 1. Seleccionar la base de datos del curso
USE DATABASE CURSO_SQL;

-- 2. Seleccionar el schema correspondiente
--    En entorno compartido: reemplaza PUBLIC por tu schema asignado (ej. ESTUDIANTE_01)
USE SCHEMA PUBLIC;

-- 3. Activar el Virtual Warehouse
--    Reemplaza NOMBRE_WH por el nombre de tu warehouse (ej. COMPUTE_WH)
USE WAREHOUSE NOMBRE_WH;

-- 4. Verificar el contexto activo
SELECT CURRENT_DATABASE(), CURRENT_SCHEMA(), CURRENT_WAREHOUSE();
```

**Resultado esperado de la verificación:**

| CURRENT_DATABASE() | CURRENT_SCHEMA() | CURRENT_WAREHOUSE() |
|--------------------|------------------|---------------------|
| CURSO_SQL          | PUBLIC           | COMPUTE_WH          |

> **Nota para entornos compartidos:** Si tu instructor te asignó un schema personal (ej. `ESTUDIANTE_03`), reemplaza `PUBLIC` por ese nombre en el comando `USE SCHEMA`. Esto evita conflictos con otros estudiantes.

> **Consejo de costos:** Asegúrate de que tu warehouse sea de tamaño **X-Small**. Puedes verificarlo en Snowsight en **Admin → Warehouses**.

---

## Instrucciones Paso a Paso

---

### Paso 1: Explorar las Tablas de Trabajo

**Objetivo:** Familiarizarte con la estructura y contenido de las tablas `CLIENTES` y `PRODUCTOS` antes de aplicar transformaciones, identificando columnas de texto y columnas numéricas.

#### Instrucciones

1. En Snowsight, abre una nueva **Worksheet** haciendo clic en el ícono `+` en el panel izquierdo o seleccionando **Projects → Worksheets → + Worksheet**.

2. Asegúrate de que el contexto de sesión esté configurado (ver sección anterior). Verifica que en la parte superior de la Worksheet aparezcan seleccionados tu base de datos, schema y warehouse.

3. Ejecuta la siguiente consulta para explorar la tabla `CLIENTES`:

```sql
-- Explorar estructura y datos de la tabla CLIENTES
SELECT *
FROM CLIENTES
LIMIT 10;
```

4. Observa el resultado. Identifica las columnas de tipo texto (como `NOMBRE`, `APELLIDO`, `CIUDAD`, `EMAIL`) y anota cuáles muestran inconsistencias en mayúsculas/minúsculas.

5. Ahora explora la tabla `PRODUCTOS`:

```sql
-- Explorar estructura y datos de la tabla PRODUCTOS
SELECT *
FROM PRODUCTOS
LIMIT 10;
```

6. Identifica las columnas de texto (como `NOMBRE_PRODUCTO`, `CATEGORIA`) y las columnas numéricas (como `PRECIO`, `PRECIO_COSTO`). Anota qué columnas numéricas tienen más de dos decimales.

#### Resultado Esperado

Deberías ver filas con datos como los siguientes ejemplos (los valores exactos dependen del dataset del curso):

**CLIENTES (muestra):**

| ID_CLIENTE | NOMBRE   | APELLIDO | CIUDAD       | EMAIL                   |
|------------|----------|----------|--------------|-------------------------|
| 1          | juan     | perez    | GUADALAJARA  | juan.perez@email.com    |
| 2          | MARIA    | LOPEZ    | monterrey    | MARIA.LOPEZ@EMAIL.COM   |
| 3          | Carlos   | Ruiz     | Ciudad De Mexico | carlos.ruiz@email.com |

**PRODUCTOS (muestra):**

| ID_PRODUCTO | NOMBRE_PRODUCTO     | CATEGORIA    | PRECIO    |
|-------------|---------------------|--------------|-----------|
| 101         | laptop gamer        | ELECTRONICA  | 15999.999 |
| 102         | MOUSE INALAMBRICO   | electronica  | 349.5567  |
| 103         | Teclado Mecanico    | Electronica  | 899.1234  |

#### Verificación

Confirma que puedes responder estas preguntas antes de continuar:
- ¿Cuántas columnas de texto tiene la tabla `CLIENTES`?
- ¿La columna `CIUDAD` en `CLIENTES` tiene valores consistentes en mayúsculas/minúsculas?
- ¿La columna `PRECIO` en `PRODUCTOS` tiene valores con más de dos decimales?

---

### Paso 2: Aplicar UPPER para Estandarizar Texto a Mayúsculas

**Objetivo:** Usar la función `UPPER` para transformar columnas de texto a mayúsculas en el resultado de la consulta, sin modificar los datos originales de la tabla `CLIENTES`.

#### Instrucciones

1. En la misma Worksheet (o en una nueva), escribe y ejecuta la siguiente consulta:

```sql
-- Normalizar nombres y ciudades de clientes a mayúsculas
SELECT
    ID_CLIENTE,
    UPPER(NOMBRE)   AS nombre_mayusculas,
    UPPER(APELLIDO) AS apellido_mayusculas,
    UPPER(CIUDAD)   AS ciudad_mayusculas
FROM CLIENTES
LIMIT 10;
```

2. Observa el resultado. Verifica que todos los valores en las columnas `nombre_mayusculas`, `apellido_mayusculas` y `ciudad_mayusculas` aparezcan completamente en mayúsculas, independientemente de cómo estaban escritos originalmente.

3. Para confirmar que los datos originales **no fueron modificados**, ejecuta nuevamente la consulta básica del Paso 1:

```sql
-- Verificar que los datos originales no cambiaron
SELECT ID_CLIENTE, NOMBRE, APELLIDO, CIUDAD
FROM CLIENTES
LIMIT 10;
```

4. Compara ambos resultados. Los datos en la tabla deben ser idénticos a los que viste en el Paso 1.

#### Resultado Esperado

**Resultado de la consulta con UPPER:**

| ID_CLIENTE | nombre_mayusculas | apellido_mayusculas | ciudad_mayusculas    |
|------------|-------------------|---------------------|----------------------|
| 1          | JUAN              | PEREZ               | GUADALAJARA          |
| 2          | MARIA             | LOPEZ               | MONTERREY            |
| 3          | CARLOS            | RUIZ                | CIUDAD DE MEXICO     |

Observa que:
- Los alias `nombre_mayusculas`, `apellido_mayusculas` y `ciudad_mayusculas` aparecen como encabezados de columna.
- Todos los valores están en mayúsculas sin importar cómo estaban almacenados.
- La columna `ID_CLIENTE` no fue transformada porque es un número, no texto.

#### Verificación

✅ Todos los valores de texto en el resultado deben estar en **MAYÚSCULAS**.
✅ Los encabezados de columna deben mostrar los alias definidos con `AS`.
✅ La consulta de verificación sobre la tabla original debe mostrar los datos **sin cambios**.

> **Punto clave:** `UPPER` transforma el resultado de la consulta, no los datos almacenados. Esta es la naturaleza no destructiva de las funciones escalares en SQL.

---

### Paso 3: Aplicar LOWER para Estandarizar Texto a Minúsculas

**Objetivo:** Usar la función `LOWER` para normalizar columnas de texto a minúsculas, y aplicarla en la cláusula `WHERE` para realizar comparaciones insensibles a mayúsculas/minúsculas.

#### Instrucciones

1. Ejecuta la siguiente consulta para transformar la presentación de datos de `PRODUCTOS` a minúsculas:

```sql
-- Normalizar nombres de productos y categorías a minúsculas
SELECT
    ID_PRODUCTO,
    LOWER(NOMBRE_PRODUCTO) AS producto_minusculas,
    LOWER(CATEGORIA)       AS categoria_minusculas
FROM PRODUCTOS
LIMIT 10;
```

2. Ahora aplica `LOWER` en la cláusula `WHERE` para filtrar productos de una categoría sin preocuparte por la capitalización almacenada. Ejecuta:

```sql
-- Filtrar productos por categoría usando LOWER para comparación segura
SELECT
    ID_PRODUCTO,
    UPPER(NOMBRE_PRODUCTO) AS producto,
    UPPER(CATEGORIA)       AS categoria,
    PRECIO
FROM PRODUCTOS
WHERE LOWER(CATEGORIA) = 'electronica';
```

3. Analiza la consulta anterior. Nota que:
   - En `SELECT` usamos `UPPER` para presentar los datos en mayúsculas (formato de reporte).
   - En `WHERE` usamos `LOWER` para comparar de forma segura, convirtiendo el valor almacenado a minúsculas antes de compararlo con la cadena `'electronica'` (también en minúsculas).

4. Prueba qué sucede si ejecutas el filtro **sin** `LOWER`:

```sql
-- Comparación directa sin normalización (puede no retornar todos los resultados)
SELECT
    ID_PRODUCTO,
    NOMBRE_PRODUCTO,
    CATEGORIA
FROM PRODUCTOS
WHERE CATEGORIA = 'electronica';
```

5. Compara la cantidad de filas retornadas en los pasos 2 y 4.

#### Resultado Esperado

**Resultado con LOWER en WHERE (paso 2):**

| ID_PRODUCTO | producto            | categoria   | PRECIO    |
|-------------|---------------------|-------------|-----------|
| 101         | LAPTOP GAMER        | ELECTRONICA | 15999.999 |
| 102         | MOUSE INALAMBRICO   | ELECTRONICA | 349.5567  |
| 103         | TECLADO MECANICO    | ELECTRONICA | 899.1234  |

**Resultado sin normalización (paso 4):** Podría retornar 0 filas o menos filas, dependiendo de cómo estén escritos los valores en la tabla.

#### Verificación

✅ La consulta con `LOWER` en `WHERE` debe retornar **todos** los productos de electrónica, sin importar cómo está escrita la categoría en la tabla.
✅ La consulta sin normalización puede retornar menos filas o ninguna.
✅ Los encabezados deben mostrar los alias definidos.

> **Nota Snowflake:** Por defecto, Snowflake **no distingue entre mayúsculas y minúsculas** en comparaciones de strings (comportamiento `CASE_INSENSITIVE`). Sin embargo, usar `LOWER` en el `WHERE` es una buena práctica de portabilidad: garantiza el mismo comportamiento en motores como PostgreSQL o MySQL que sí son sensibles. El instructor puede demostrar este comportamiento en vivo.

---

### Paso 4: Usar CONCAT para Combinar Columnas de Texto

**Objetivo:** Aplicar la función `CONCAT` para unir valores de múltiples columnas en una sola cadena de texto, creando campos compuestos como nombre completo o descripción de producto.

#### Instrucciones

1. Ejecuta la siguiente consulta para crear una columna de **nombre completo** combinando `NOMBRE` y `APELLIDO` de la tabla `CLIENTES`:

```sql
-- Crear columna de nombre completo con CONCAT
SELECT
    ID_CLIENTE,
    CONCAT(NOMBRE, ' ', APELLIDO) AS nombre_completo,
    EMAIL
FROM CLIENTES
LIMIT 10;
```

2. Observa que el segundo argumento de `CONCAT` es un espacio literal `' '` entre comillas simples. Esto agrega un espacio entre el nombre y el apellido en el resultado.

3. Ahora combina `CONCAT` con `UPPER` para crear un nombre completo en mayúsculas y estandarizado:

```sql
-- Nombre completo estandarizado en mayúsculas
SELECT
    ID_CLIENTE,
    CONCAT(UPPER(NOMBRE), ' ', UPPER(APELLIDO)) AS nombre_completo_estandar,
    LOWER(EMAIL)                                 AS email_normalizado
FROM CLIENTES
LIMIT 10;
```

4. Ahora crea una descripción de producto combinando el nombre del producto con su categoría para la tabla `PRODUCTOS`:

```sql
-- Descripción compuesta de producto
SELECT
    ID_PRODUCTO,
    CONCAT(
        UPPER(NOMBRE_PRODUCTO),
        ' [',
        UPPER(CATEGORIA),
        ']'
    ) AS descripcion_producto,
    PRECIO
FROM PRODUCTOS
LIMIT 10;
```

5. Analiza el resultado. La columna `descripcion_producto` debe combinar el nombre del producto y su categoría en un formato como: `LAPTOP GAMER [ELECTRONICA]`.

#### Resultado Esperado

**Resultado del paso 1 (nombre completo básico):**

| ID_CLIENTE | nombre_completo | EMAIL                   |
|------------|-----------------|-------------------------|
| 1          | juan perez      | juan.perez@email.com    |
| 2          | MARIA LOPEZ     | MARIA.LOPEZ@EMAIL.COM   |
| 3          | Carlos Ruiz     | carlos.ruiz@email.com   |

**Resultado del paso 3 (nombre completo estandarizado):**

| ID_CLIENTE | nombre_completo_estandar | email_normalizado        |
|------------|--------------------------|--------------------------|
| 1          | JUAN PEREZ               | juan.perez@email.com     |
| 2          | MARIA LOPEZ              | maria.lopez@email.com    |
| 3          | CARLOS RUIZ              | carlos.ruiz@email.com    |

**Resultado del paso 4 (descripción de producto):**

| ID_PRODUCTO | descripcion_producto          | PRECIO    |
|-------------|-------------------------------|-----------|
| 101         | LAPTOP GAMER [ELECTRONICA]    | 15999.999 |
| 102         | MOUSE INALAMBRICO [ELECTRONICA] | 349.5567 |
| 103         | TECLADO MECANICO [ELECTRONICA] | 899.1234 |

#### Verificación

✅ La columna `nombre_completo` debe mostrar nombre y apellido separados por un espacio.
✅ La columna `nombre_completo_estandar` debe mostrar todo en MAYÚSCULAS.
✅ La columna `email_normalizado` debe mostrar todos los emails en minúsculas.
✅ La columna `descripcion_producto` debe tener el formato `NOMBRE [CATEGORIA]`.

> **Nota Snowflake — Variación de CONCAT:** En Snowflake, `CONCAT` acepta dos o más argumentos separados por comas. Algunos motores SQL solo aceptan dos argumentos y requieren anidar llamadas: `CONCAT(CONCAT(a, b), c)`. En Snowflake puedes escribir `CONCAT(a, b, c, d)` directamente. También puedes usar el operador `||` como alternativa: `NOMBRE || ' ' || APELLIDO`. El instructor puede mostrar ambas formas.

---

### Paso 5: Aplicar ROUND para Controlar la Precisión Decimal

**Objetivo:** Usar la función `ROUND` para redondear valores numéricos a un número específico de decimales, mejorando la presentación de cifras financieras en los resultados.

#### Instrucciones

1. Observa primero los precios sin redondear en la tabla `PRODUCTOS`:

```sql
-- Ver precios originales sin redondear
SELECT
    ID_PRODUCTO,
    NOMBRE_PRODUCTO,
    PRECIO
FROM PRODUCTOS
LIMIT 10;
```

2. Ahora aplica `ROUND` para mostrar los precios con exactamente **2 decimales** (formato monetario estándar):

```sql
-- Redondear precios a 2 decimales
SELECT
    ID_PRODUCTO,
    NOMBRE_PRODUCTO,
    PRECIO                    AS precio_original,
    ROUND(PRECIO, 2)          AS precio_redondeado
FROM PRODUCTOS
LIMIT 10;
```

3. La sintaxis de `ROUND` es: `ROUND(valor_numerico, numero_de_decimales)`. El segundo argumento indica cuántos decimales conservar.

4. Prueba también redondear a **0 decimales** (precio entero) y a **1 decimal**:

```sql
-- Comparar diferentes niveles de redondeo
SELECT
    ID_PRODUCTO,
    NOMBRE_PRODUCTO,
    PRECIO                    AS precio_original,
    ROUND(PRECIO, 2)          AS precio_2_decimales,
    ROUND(PRECIO, 1)          AS precio_1_decimal,
    ROUND(PRECIO, 0)          AS precio_entero
FROM PRODUCTOS
LIMIT 10;
```

5. Observa cómo varía el resultado según el número de decimales especificado.

#### Resultado Esperado

**Resultado del paso 4 (comparación de redondeos):**

| ID_PRODUCTO | NOMBRE_PRODUCTO     | precio_original | precio_2_decimales | precio_1_decimal | precio_entero |
|-------------|---------------------|-----------------|-------------------|------------------|---------------|
| 101         | laptop gamer        | 15999.999       | 16000.00          | 16000.0          | 16000         |
| 102         | MOUSE INALAMBRICO   | 349.5567        | 349.56            | 349.6            | 350           |
| 103         | Teclado Mecanico    | 899.1234        | 899.12            | 899.1            | 899           |

#### Verificación

✅ La columna `precio_2_decimales` debe mostrar exactamente 2 decimales en todos los valores.
✅ `ROUND(349.5567, 2)` debe retornar `349.56` (redondeo hacia arriba en el tercer decimal).
✅ `ROUND(15999.999, 2)` debe retornar `16000.00`.
✅ Los datos originales en la columna `precio_original` deben permanecer sin cambios.

> **Regla de redondeo:** `ROUND` aplica el redondeo matemático estándar: si el primer dígito descartado es ≥ 5, redondea hacia arriba; si es < 5, redondea hacia abajo.

---

### Paso 6: Construir una Consulta Integrada con Múltiples Funciones

**Objetivo:** Combinar `UPPER`, `LOWER`, `CONCAT` y `ROUND` en una sola consulta `SELECT` con alias descriptivos para generar un reporte de productos completo y estandarizado.

#### Instrucciones

1. Construye la siguiente consulta integradora sobre la tabla `PRODUCTOS`. Lee cada línea cuidadosamente antes de ejecutarla:

```sql
-- Reporte integrado de productos con transformaciones múltiples
SELECT
    ID_PRODUCTO,
    CONCAT(
        UPPER(NOMBRE_PRODUCTO),
        ' | ',
        UPPER(CATEGORIA)
    )                           AS descripcion_completa,
    ROUND(PRECIO, 2)            AS precio_publico,
    ROUND(PRECIO * 0.85, 2)    AS precio_con_descuento_15
FROM PRODUCTOS
ORDER BY PRECIO DESC
LIMIT 15;
```

2. Analiza cada transformación en la consulta:
   - `CONCAT(UPPER(NOMBRE_PRODUCTO), ' | ', UPPER(CATEGORIA))`: Combina nombre y categoría en mayúsculas con separador ` | `.
   - `ROUND(PRECIO, 2)`: Precio original redondeado a 2 decimales.
   - `ROUND(PRECIO * 0.85, 2)`: Calcula el 85% del precio (descuento del 15%) y lo redondea.

3. Ahora construye un reporte similar para la tabla `CLIENTES`:

```sql
-- Reporte integrado de clientes con transformaciones múltiples
SELECT
    ID_CLIENTE,
    CONCAT(UPPER(NOMBRE), ' ', UPPER(APELLIDO)) AS nombre_completo,
    LOWER(EMAIL)                                 AS email_normalizado,
    UPPER(CIUDAD)                                AS ciudad
FROM CLIENTES
ORDER BY UPPER(APELLIDO), UPPER(NOMBRE)
LIMIT 15;
```

4. Observa que en el `ORDER BY` también se puede usar `UPPER` para ordenar alfabéticamente sin que la capitalización afecte el orden.

#### Resultado Esperado

**Resultado del reporte de productos:**

| ID_PRODUCTO | descripcion_completa                    | precio_publico | precio_con_descuento_15 |
|-------------|------------------------------------------|----------------|------------------------|
| 101         | LAPTOP GAMER \| ELECTRONICA              | 16000.00       | 13600.00               |
| 103         | TECLADO MECANICO \| ELECTRONICA          | 899.12         | 764.25                 |
| 102         | MOUSE INALAMBRICO \| ELECTRONICA         | 349.56         | 297.13                 |

**Resultado del reporte de clientes:**

| ID_CLIENTE | nombre_completo | email_normalizado        | ciudad           |
|------------|-----------------|--------------------------|------------------|
| 2          | MARIA LOPEZ     | maria.lopez@email.com    | MONTERREY        |
| 1          | JUAN PEREZ      | juan.perez@email.com     | GUADALAJARA      |
| 3          | CARLOS RUIZ     | carlos.ruiz@email.com    | CIUDAD DE MEXICO |

#### Verificación

✅ La columna `descripcion_completa` debe tener el formato `NOMBRE_PRODUCTO | CATEGORIA` completamente en mayúsculas.
✅ La columna `precio_con_descuento_15` debe ser exactamente el 85% del precio original, redondeado a 2 decimales.
✅ El resultado de productos debe estar ordenado de mayor a menor precio (`ORDER BY PRECIO DESC`).
✅ El resultado de clientes debe estar ordenado alfabéticamente por apellido.

---

### Paso 7: Ejercicio de Aplicación — Reporte de Catálogo

**Objetivo:** Aplicar de forma autónoma todas las funciones aprendidas para construir un reporte de catálogo de productos con formato profesional.

#### Instrucciones

Construye **por tu cuenta** una consulta que cumpla todos los siguientes requisitos:

1. Selecciona de la tabla `PRODUCTOS`.
2. Crea una columna llamada `etiqueta_catalogo` que combine:
   - El nombre del producto en **MAYÚSCULAS**
   - Seguido de un guion entre espacios ` - `
   - Seguido de la categoría en **MAYÚSCULAS**
3. Crea una columna llamada `precio_formato` que muestre el precio redondeado a **2 decimales**.
4. Crea una columna llamada `precio_sin_iva` que calcule el precio dividido entre 1.16 (precio sin IVA del 16%) y lo redondee a **2 decimales**.
5. Ordena los resultados por `etiqueta_catalogo` de forma **ascendente**.
6. Limita el resultado a **20 filas**.

> **Pista:** La estructura general de la consulta es:
> ```sql
> SELECT
>     ID_PRODUCTO,
>     CONCAT(...)         AS etiqueta_catalogo,
>     ROUND(...)          AS precio_formato,
>     ROUND(... / 1.16, 2) AS precio_sin_iva
> FROM PRODUCTOS
> ORDER BY etiqueta_catalogo
> LIMIT 20;
> ```

#### Solución de Referencia

> ⚠️ **Intenta resolver el ejercicio antes de ver la solución.**

```sql
-- Solución: Reporte de catálogo de productos
SELECT
    ID_PRODUCTO,
    CONCAT(
        UPPER(NOMBRE_PRODUCTO),
        ' - ',
        UPPER(CATEGORIA)
    )                       AS etiqueta_catalogo,
    ROUND(PRECIO, 2)        AS precio_formato,
    ROUND(PRECIO / 1.16, 2) AS precio_sin_iva
FROM PRODUCTOS
ORDER BY etiqueta_catalogo
LIMIT 20;
```

#### Resultado Esperado

| ID_PRODUCTO | etiqueta_catalogo                       | precio_formato | precio_sin_iva |
|-------------|------------------------------------------|----------------|----------------|
| 101         | LAPTOP GAMER - ELECTRONICA               | 16000.00       | 13793.10       |
| 102         | MOUSE INALAMBRICO - ELECTRONICA          | 349.56         | 301.34         |
| 103         | TECLADO MECANICO - ELECTRONICA           | 899.12         | 774.24         |

#### Verificación

✅ La columna `etiqueta_catalogo` debe tener el formato `NOMBRE - CATEGORIA` en mayúsculas.
✅ La columna `precio_sin_iva` debe ser menor que `precio_formato` (precio sin impuesto es menor que precio con impuesto).
✅ El resultado debe estar ordenado **alfabéticamente** por `etiqueta_catalogo`.
✅ No debe haber más de 20 filas en el resultado.

---

## Validación y Pruebas del Laboratorio

Ejecuta las siguientes consultas de validación para confirmar que completaste correctamente todos los pasos del laboratorio.

### Validación 1 — Verificar que los datos originales no fueron modificados

```sql
-- Los datos originales deben estar intactos
SELECT
    COUNT(*)                         AS total_clientes,
    COUNT(DISTINCT UPPER(CIUDAD))    AS ciudades_unicas_normalizadas,
    COUNT(DISTINCT CIUDAD)           AS ciudades_raw
FROM CLIENTES;
```

**Criterio de éxito:** `ciudades_unicas_normalizadas` debe ser ≤ `ciudades_raw` si hay inconsistencias de capitalización en la tabla. Los datos originales (`ciudades_raw`) no deben haber cambiado entre ejecuciones.

### Validación 2 — Verificar comportamiento de ROUND

```sql
-- Verificar que ROUND funciona correctamente
SELECT
    ROUND(349.5567, 2)  AS debe_ser_349_56,
    ROUND(349.5567, 0)  AS debe_ser_350,
    ROUND(899.1234, 2)  AS debe_ser_899_12,
    ROUND(15999.999, 2) AS debe_ser_16000_00;
```

**Criterio de éxito:** Los resultados deben coincidir exactamente con los valores en los encabezados de columna.

### Validación 3 — Verificar CONCAT con múltiples argumentos

```sql
-- Verificar que CONCAT une correctamente múltiples cadenas
SELECT
    CONCAT('PRODUCTO', ' | ', 'ELECTRONICA')   AS test_concat_3_args,
    CONCAT(UPPER('laptop'), ' ', UPPER('gamer')) AS test_concat_con_upper;
```

**Criterio de éxito:**
- `test_concat_3_args` debe retornar `PRODUCTO | ELECTRONICA`
- `test_concat_con_upper` debe retornar `LAPTOP GAMER`

### Validación 4 — Consulta integradora completa

```sql
-- Validación final: todas las funciones en una consulta
SELECT
    ID_PRODUCTO,
    CONCAT(UPPER(NOMBRE_PRODUCTO), ' [', UPPER(CATEGORIA), ']') AS etiqueta,
    ROUND(PRECIO, 2)                                             AS precio_2dec,
    LOWER(CATEGORIA)                                             AS categoria_lower,
    UPPER(NOMBRE_PRODUCTO)                                       AS nombre_upper
FROM PRODUCTOS
WHERE LOWER(CATEGORIA) = 'electronica'
ORDER BY PRECIO DESC
LIMIT 5;
```

**Criterio de éxito:** La consulta debe ejecutarse sin errores y retornar filas donde:
- `etiqueta` tiene el formato `NOMBRE [CATEGORIA]` en mayúsculas.
- `precio_2dec` tiene exactamente 2 decimales.
- `categoria_lower` muestra `electronica` en minúsculas.
- `nombre_upper` muestra el nombre en mayúsculas.

---

## Solución de Problemas

### Problema 1: CONCAT retorna NULL cuando una columna tiene valores nulos

**Síntoma:**
La columna calculada con `CONCAT` muestra `NULL` en algunas filas, aunque las otras columnas de esa fila tienen valores.

**Ejemplo del síntoma:**
```sql
SELECT CONCAT(NOMBRE, ' ', APELLIDO) AS nombre_completo
FROM CLIENTES;
-- Resultado: algunas filas muestran NULL en nombre_completo
```

**Causa:**
En SQL estándar (y en Snowflake), si **cualquiera** de los argumentos de `CONCAT` es `NULL`, el resultado completo es `NULL`. Si la columna `APELLIDO` tiene valores nulos en algunos registros, toda la concatenación produce `NULL`.

**Solución:**
Usa la función `COALESCE` para reemplazar los valores `NULL` por una cadena vacía o un texto alternativo antes de concatenar:

```sql
-- Solución con COALESCE para manejar NULLs en CONCAT
SELECT
    ID_CLIENTE,
    CONCAT(
        COALESCE(NOMBRE, ''),
        ' ',
        COALESCE(APELLIDO, '')
    ) AS nombre_completo
FROM CLIENTES
LIMIT 10;
```

`COALESCE(columna, valor_alternativo)` retorna el primer valor no-nulo. Si `APELLIDO` es `NULL`, retorna `''` (cadena vacía), evitando que `CONCAT` produzca `NULL`.

---

### Problema 2: Los resultados de ROUND no muestran ceros finales (ej. 349.5 en lugar de 349.50)

**Síntoma:**
Al ejecutar `ROUND(PRECIO, 2)`, algunos valores aparecen con solo 1 decimal (ej. `349.50` se muestra como `349.5`) en lugar de siempre mostrar 2 decimales.

**Ejemplo del síntoma:**
```sql
SELECT ROUND(349.50, 2);
-- Muestra: 349.5  (en lugar del esperado 349.50)
```

**Causa:**
`ROUND` es una función **matemática** que retorna un valor numérico. Los ceros finales en decimales son una propiedad de **presentación/formato**, no del valor numérico en sí. El número `349.5` y `349.50` son matemáticamente idénticos. Snowflake (y la mayoría de motores SQL) no garantizan mostrar ceros finales en el tipo de dato `NUMBER`.

**Solución:**
Si necesitas formato visual con ceros finales (por ejemplo, para reportes financieros), usa la función `TO_CHAR` para convertir el número a texto con formato específico:

```sql
-- Mostrar siempre 2 decimales con ceros finales usando TO_CHAR
SELECT
    ID_PRODUCTO,
    NOMBRE_PRODUCTO,
    TO_CHAR(ROUND(PRECIO, 2), '999,999.00') AS precio_formato_visual
FROM PRODUCTOS
LIMIT 10;
```

> **Nota:** `TO_CHAR` convierte el número a `VARCHAR`. Si necesitas seguir operando matemáticamente con el valor, usa `ROUND` directamente. Usa `TO_CHAR` solo para la presentación final en reportes.

---

## Limpieza del Entorno

Al finalizar el laboratorio, realiza los siguientes pasos para liberar recursos y minimizar el consumo de créditos en tu cuenta Snowflake.

### 1. Guardar tu Trabajo

Antes de cerrar, guarda las consultas más importantes en la Worksheet:

```sql
-- Guardar el nombre de la worksheet para referencia futura
-- En Snowsight: haz clic en el nombre de la Worksheet en la parte superior
-- y renómbrala como "Lab04_Transformacion_Datos"
```

En Snowsight, haz clic en el nombre de la Worksheet (por defecto aparece como `Worksheet` con fecha y hora) y renómbrala como `Lab04_Transformacion_Datos` para poder encontrarla fácilmente en sesiones futuras.

### 2. Suspender el Virtual Warehouse

Ejecuta el siguiente comando para suspender el warehouse inmediatamente y detener el consumo de créditos:

```sql
-- Suspender el Virtual Warehouse al finalizar
-- Reemplaza NOMBRE_WH por el nombre de tu warehouse
ALTER WAREHOUSE NOMBRE_WH SUSPEND;
```

Alternativamente, en Snowsight ve a **Admin → Warehouses**, localiza tu warehouse y haz clic en **Suspend**.

### 3. Verificar que No Hay Objetos Temporales

Este laboratorio no crea tablas, vistas ni objetos permanentes. Todas las transformaciones se realizaron exclusivamente dentro de consultas `SELECT`. No es necesario ejecutar comandos `DROP` para limpiar.

```sql
-- Verificación: confirmar que no se crearon objetos nuevos
-- (Este laboratorio no crea objetos; esta consulta es solo informativa)
SHOW TABLES IN SCHEMA CURSO_SQL.PUBLIC;
```

El resultado debe mostrar únicamente las tablas originales del dataset (`CLIENTES`, `PRODUCTOS`, y otras del curso), sin ninguna tabla nueva creada durante el laboratorio.

---

## Resumen

### Lo que Aprendiste en este Laboratorio

En este laboratorio aplicaste las funciones de transformación de datos más fundamentales en SQL, trabajando directamente sobre las tablas `CLIENTES` y `PRODUCTOS` de la base de datos `CURSO_SQL`:

| Función         | Propósito                                              | Ejemplo                                      |
|-----------------|--------------------------------------------------------|----------------------------------------------|
| `UPPER(col)`    | Convierte texto a **MAYÚSCULAS**                       | `UPPER('juan')` → `'JUAN'`                   |
| `LOWER(col)`    | Convierte texto a **minúsculas**                       | `LOWER('MARIA')` → `'maria'`                 |
| `CONCAT(a,b,c)` | **Combina** múltiples cadenas en una sola              | `CONCAT('A', ' ', 'B')` → `'A B'`           |
| `ROUND(n, d)`   | **Redondea** un número a `d` decimales                 | `ROUND(349.5567, 2)` → `349.56`              |
| `AS alias`      | Asigna un **nombre descriptivo** a columnas calculadas | `UPPER(NOMBRE) AS nombre_mayusculas`         |

### Conceptos Clave Reforzados

- **Las funciones escalares son no destructivas:** Transforman el resultado de la consulta sin modificar los datos almacenados en la tabla. Los datos originales siempre permanecen intactos.
- **Las funciones se pueden anidar:** `CONCAT(UPPER(NOMBRE), ' ', UPPER(APELLIDO))` combina `UPPER` dentro de `CONCAT` en una sola expresión.
- **Las funciones se pueden usar en cualquier cláusula:** `UPPER` y `LOWER` son útiles en `SELECT` (presentación), `WHERE` (filtrado normalizado) y `ORDER BY` (ordenamiento consistente).
- **Los alias `AS` son esenciales en columnas calculadas:** Sin alias, las columnas calculadas muestran la expresión completa como encabezado, lo que reduce la legibilidad del resultado.

### Conexión con el Siguiente Laboratorio

En el **Lab 05**, comenzarás a trabajar con funciones de agregación (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) y la cláusula `GROUP BY`. Las funciones de transformación que aprendiste hoy se combinarán con agregaciones para producir reportes más sofisticados, por ejemplo: contar clientes por ciudad (normalizada con `UPPER`) o calcular el precio promedio por categoría (redondeado con `ROUND`).

---

## Recursos Adicionales

- [Documentación Snowflake: Función UPPER](https://docs.snowflake.com/en/sql-reference/functions/upper)
- [Documentación Snowflake: Función LOWER](https://docs.snowflake.com/en/sql-reference/functions/lower)
- [Documentación Snowflake: Función CONCAT](https://docs.snowflake.com/en/sql-reference/functions/concat)
- [Documentación Snowflake: Función ROUND](https://docs.snowflake.com/en/sql-reference/functions/round)
- [Snowflake SQL Reference — String & Binary Functions](https://docs.snowflake.com/en/sql-reference/functions-string)
- [Snowflake SQL Reference — Numeric Functions](https://docs.snowflake.com/en/sql-reference/functions-numeric)

---

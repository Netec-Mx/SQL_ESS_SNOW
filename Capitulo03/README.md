# Top N análisis

## 1. Metadatos

| Campo            | Detalle                                      |
|------------------|----------------------------------------------|
| **Duración**     | 50 minutos                                   |
| **Complejidad**  | Fácil                                        |
| **Nivel Bloom**  | Aplicar (Apply)                              |
| **Módulo**       | 3 — Ordenamiento y análisis Top N            |
| **Versión**      | 1.0                                          |
| **Dataset**      | CURSO_SQL (provisto por el instructor)       |

---

## 2. Descripción General

En este laboratorio aplicarás la cláusula `ORDER BY` para organizar los resultados de tus consultas SQL de forma ascendente y descendente, trabajando directamente sobre las tablas **VENTAS** y **PRODUCTOS** del dataset del curso. El objetivo central es construir análisis **Top N**: identificar los productos más vendidos, los clientes con mayor gasto y las transacciones más recientes, combinando `ORDER BY` con `LIMIT`. Al finalizar, habrás desarrollado la habilidad de transformar un conjunto de datos sin orden aparente en rankings claros y útiles para la toma de decisiones de negocio.

---

## 3. Objetivos de Aprendizaje

Al completar este laboratorio serás capaz de:

- [ ] Aplicar `ORDER BY` con dirección `ASC` y `DESC` para ordenar resultados por una o más columnas.
- [ ] Combinar `ORDER BY` con `WHERE` para filtrar y luego ordenar datos en una misma consulta.
- [ ] Construir consultas **Top N** utilizando `ORDER BY` + `LIMIT` para identificar los registros de mayor o menor valor.
- [ ] Ordenar resultados por múltiples columnas estableciendo jerarquías de clasificación.
- [ ] Interpretar correctamente el comportamiento de `NULL` en Snowflake al ordenar datos incompletos.

---

## 4. Prerrequisitos

### Conocimiento previo

| Requisito                                                                 | Nivel esperado       |
|---------------------------------------------------------------------------|----------------------|
| Completar **Lab 02-00-01** o demostrar dominio de `SELECT` con `WHERE`   | Obligatorio          |
| Uso de operadores lógicos `AND`, `OR`, `NOT` en cláusulas `WHERE`        | Obligatorio          |
| Familiaridad con el concepto de ordenamiento en hojas de cálculo          | Recomendado          |
| Conocimiento básico de tipos de datos: texto, número, fecha              | Recomendado          |

### Acceso y configuración

| Requisito                                                  | Estado esperado al iniciar    |
|------------------------------------------------------------|-------------------------------|
| Cuenta Snowflake activa (trial o corporativa)              | ✅ Activa y accesible          |
| Base de datos `CURSO_SQL` creada e inicializada            | ✅ Script de setup ejecutado   |
| Virtual Warehouse disponible (tamaño **X-Small**)          | ✅ Creado y en estado STARTED  |
| Acceso a Snowsight (interfaz web)                          | ✅ Sesión iniciada             |

> **⚠️ Nota para entornos compartidos:** Si compartes una cuenta Snowflake corporativa, asegúrate de usar tu schema personal asignado (ej. `ESTUDIANTE_01`). Consulta con tu instructor antes de iniciar.

---

## 5. Entorno de Laboratorio

### Hardware recomendado

| Componente       | Mínimo requerido                                              |
|------------------|---------------------------------------------------------------|
| Procesador       | Cualquier CPU moderna (el procesamiento ocurre en Snowflake) |
| RAM              | 4 GB (para navegador con múltiples pestañas)                 |
| Conexión         | 10 Mbps estables                                             |
| Pantalla         | Resolución mínima 1280×768 px                                |

### Software requerido

| Software         | Versión mínima         | Notas                                      |
|------------------|------------------------|--------------------------------------------|
| Navegador web    | Chrome 90+ / Firefox 88+ / Edge 90+ / Safari 14+ | Usar Chrome sin extensiones como respaldo |
| Snowflake        | Standard edition o superior | Acceso vía Snowsight                  |
| Dataset CURSO_SQL | v1.0 (instructor)     | Debe estar cargado antes de este lab       |

### Configuración inicial del entorno

Antes de comenzar los ejercicios, abre un nuevo **Worksheet** en Snowsight y ejecuta los siguientes comandos para asegurarte de estar trabajando en el contexto correcto:

```sql
-- 1. Seleccionar la base de datos del curso
USE DATABASE CURSO_SQL;

-- 2. Seleccionar el schema correspondiente
--    En cuenta individual:
USE SCHEMA PUBLIC;
--    En entorno compartido (reemplaza XX con tu número de estudiante):
-- USE SCHEMA ESTUDIANTE_XX;

-- 3. Activar el Virtual Warehouse (tamaño X-Small para minimizar créditos)
USE WAREHOUSE COMPUTE_WH;

-- 4. Verificar que las tablas necesarias están disponibles
SHOW TABLES;
```

**Resultado esperado de `SHOW TABLES`:** Deberías ver al menos las tablas `VENTAS`, `PRODUCTOS` y `CLIENTES` en el listado.

> **💡 Consejo de créditos:** Usa siempre un warehouse de tamaño **X-Small** durante este curso. Recuerda apagarlo al finalizar la sesión con `ALTER WAREHOUSE COMPUTE_WH SUSPEND;`

---

## 6. Pasos del Laboratorio

Este laboratorio está organizado en **5 ejercicios progresivos**. Cada ejercicio introduce un nuevo concepto o combinación de cláusulas. Lee cada sección completa antes de ejecutar el código.

---

### Ejercicio 1: Exploración inicial y primer ordenamiento

**Objetivo:** Familiarizarte con las tablas del laboratorio y aplicar tu primer `ORDER BY` simple para ordenar datos por una columna numérica.

#### Instrucciones

**Paso 1.1 — Explorar la estructura de la tabla PRODUCTOS**

Antes de ordenar, es importante entender qué columnas están disponibles.

```sql
-- Revisar la estructura de la tabla PRODUCTOS
DESCRIBE TABLE productos;
```

Observa las columnas disponibles: sus nombres, tipos de datos y si admiten valores NULL. Esto te ayudará a elegir correctamente las columnas de ordenamiento en los pasos siguientes.

**Paso 1.2 — Ver los primeros registros sin ordenamiento**

```sql
-- Ver una muestra de productos sin ningún orden específico
SELECT
    id_producto,
    nombre_producto,
    categoria,
    precio_unitario,
    stock_disponible
FROM productos
LIMIT 10;
```

**Paso 1.3 — Aplicar ORDER BY ascendente por precio**

```sql
-- Ordenar todos los productos por precio de menor a mayor
SELECT
    id_producto,
    nombre_producto,
    categoria,
    precio_unitario,
    stock_disponible
FROM productos
ORDER BY precio_unitario ASC;
```

**Paso 1.4 — Aplicar ORDER BY descendente por precio**

```sql
-- Ordenar todos los productos por precio de mayor a menor
SELECT
    id_producto,
    nombre_producto,
    categoria,
    precio_unitario,
    stock_disponible
FROM productos
ORDER BY precio_unitario DESC;
```

#### Resultado esperado

- El Paso 1.2 devuelve 10 filas en un orden no predecible (depende del almacenamiento interno de Snowflake).
- El Paso 1.3 muestra los productos comenzando por el de menor precio (ej. el producto más económico aparece en la primera fila).
- El Paso 1.4 muestra los productos comenzando por el de mayor precio (el producto más caro aparece primero).

#### Verificación

Confirma que tu consulta es correcta respondiendo estas preguntas:

- [ ] ¿El primer registro del Paso 1.3 tiene el `precio_unitario` más bajo de toda la tabla?
- [ ] ¿El primer registro del Paso 1.4 tiene el `precio_unitario` más alto de toda la tabla?
- [ ] ¿La cantidad total de filas es la misma en los Pasos 1.3 y 1.4? (Solo cambia el orden, no el número de registros.)

> **📌 Concepto clave:** `ORDER BY precio_unitario` sin especificar dirección es equivalente a `ORDER BY precio_unitario ASC`. Snowflake asume orden ascendente por defecto.

---

### Ejercicio 2: Ordenamiento por múltiples columnas

**Objetivo:** Aplicar `ORDER BY` con dos criterios de ordenamiento simultáneos para manejar situaciones donde el primer criterio produce empates.

#### Instrucciones

**Paso 2.1 — Identificar el problema de los empates**

Primero, observa qué ocurre cuando ordenamos solo por categoría:

```sql
-- Ordenar productos solo por categoría (primer criterio)
SELECT
    nombre_producto,
    categoria,
    precio_unitario
FROM productos
ORDER BY categoria ASC;
```

Observa que dentro de cada categoría, los productos no siguen ningún orden en particular. Esto es lo que resolveremos con el segundo criterio.

**Paso 2.2 — Ordenar por categoría y luego por precio**

```sql
-- Ordenar por categoría (A→Z) y dentro de cada categoría por precio (mayor→menor)
SELECT
    nombre_producto,
    categoria,
    precio_unitario
FROM productos
ORDER BY categoria ASC, precio_unitario DESC;
```

**Paso 2.3 — Ordenar por categoría y luego por nombre de producto**

```sql
-- Ordenar por categoría (A→Z) y dentro de cada categoría por nombre (A→Z)
SELECT
    nombre_producto,
    categoria,
    precio_unitario
FROM productos
ORDER BY categoria ASC, nombre_producto ASC;
```

**Paso 2.4 — Explorar la tabla VENTAS con ordenamiento múltiple**

```sql
-- Ver las ventas ordenadas por fecha (más reciente primero) y luego por monto (mayor primero)
SELECT
    id_venta,
    id_cliente,
    id_producto,
    monto_total,
    fecha_venta
FROM ventas
ORDER BY fecha_venta DESC, monto_total DESC;
```

#### Resultado esperado

- **Paso 2.2:** Los productos aparecen agrupados visualmente por categoría (todas las categorías iguales están juntas). Dentro de cada categoría, el producto más caro aparece primero.
- **Paso 2.3:** Mismo agrupamiento por categoría, pero dentro de cada grupo los productos están en orden alfabético.
- **Paso 2.4:** Las ventas más recientes aparecen primero. Si hay dos ventas del mismo día, la de mayor monto aparece antes.

#### Verificación

- [ ] En el Paso 2.2, ¿todos los productos de la misma categoría están agrupados juntos?
- [ ] En el Paso 2.2, ¿dentro de cada categoría el precio va de mayor a menor?
- [ ] En el Paso 2.4, ¿la primera fila corresponde a la venta más reciente con el monto más alto en ese día?

> **📌 Regla de múltiples columnas:** Snowflake evalúa los criterios de `ORDER BY` de izquierda a derecha. El segundo criterio solo se aplica cuando hay empate en el primero. Puedes usar diferentes direcciones (ASC/DESC) para cada columna de forma independiente.

---

### Ejercicio 3: Combinando WHERE y ORDER BY

**Objetivo:** Construir consultas que primero filtren datos con `WHERE` y luego los presenten ordenados con `ORDER BY`, replicando un flujo de análisis real.

#### Instrucciones

**Paso 3.1 — Ventas de un rango de fechas ordenadas por monto**

```sql
-- Ventas del primer trimestre de 2024, ordenadas de mayor a menor monto
SELECT
    id_venta,
    id_cliente,
    id_producto,
    monto_total,
    fecha_venta
FROM ventas
WHERE fecha_venta >= '2024-01-01'
  AND fecha_venta <= '2024-03-31'
ORDER BY monto_total DESC;
```

**Paso 3.2 — Productos de una categoría específica ordenados por precio**

```sql
-- Productos de la categoría 'Electrónica' ordenados de menor a mayor precio
-- (Reemplaza 'Electrónica' por una categoría real de tu dataset si es necesario)
SELECT
    nombre_producto,
    categoria,
    precio_unitario,
    stock_disponible
FROM productos
WHERE categoria = 'Electrónica'
ORDER BY precio_unitario ASC;
```

> **💡 Tip:** Si no conoces los nombres exactos de las categorías, ejecuta primero: `SELECT DISTINCT categoria FROM productos ORDER BY categoria;` para ver todas las opciones disponibles.

**Paso 3.3 — Clientes con ventas de alto valor en orden cronológico**

```sql
-- Ventas con monto superior a 500, ordenadas cronológicamente (más antigua primero)
SELECT
    id_venta,
    id_cliente,
    monto_total,
    fecha_venta
FROM ventas
WHERE monto_total > 500
ORDER BY fecha_venta ASC;
```

**Paso 3.4 — Productos con stock bajo ordenados por stock disponible**

```sql
-- Productos con stock menor a 20 unidades, ordenados de menor a mayor stock
-- (Útil para priorizar reabastecimiento)
SELECT
    nombre_producto,
    categoria,
    precio_unitario,
    stock_disponible
FROM productos
WHERE stock_disponible < 20
ORDER BY stock_disponible ASC;
```

#### Resultado esperado

- **Paso 3.1:** Solo aparecen ventas entre el 1 de enero y el 31 de marzo de 2024, con la venta de mayor monto en la primera fila.
- **Paso 3.2:** Solo aparecen productos de la categoría seleccionada, comenzando por el más económico.
- **Paso 3.3:** Solo aparecen ventas con monto superior a 500, ordenadas desde la más antigua hasta la más reciente.
- **Paso 3.4:** Solo aparecen productos con stock crítico, comenzando por el que tiene menos unidades disponibles.

#### Verificación

- [ ] ¿El Paso 3.1 muestra únicamente fechas dentro del rango especificado? (Revisa la primera y última fila.)
- [ ] ¿El Paso 3.4 muestra en la primera fila el producto con el stock más bajo de todos los productos con stock menor a 20?
- [ ] ¿En el Paso 3.3 todos los valores de `monto_total` son estrictamente mayores a 500?

> **📌 Orden de ejecución vs. orden de escritura:** Aunque escribimos `SELECT → FROM → WHERE → ORDER BY`, Snowflake ejecuta internamente `FROM → WHERE → SELECT → ORDER BY`. Esto significa que `ORDER BY` actúa sobre el conjunto ya filtrado por `WHERE`, no sobre la tabla completa. El resultado es más eficiente y coherente.

---

### Ejercicio 4: Análisis Top N con ORDER BY + LIMIT

**Objetivo:** Construir consultas Top N que combinen `ORDER BY` y `LIMIT` para identificar los mejores o peores N registros de un conjunto de datos.

Este es el ejercicio central del laboratorio. El patrón **`ORDER BY [columna] DESC + LIMIT N`** es uno de los más utilizados en análisis de negocio para construir rankings y reportes ejecutivos.

#### Instrucciones

**Paso 4.1 — Top 5 productos más caros**

```sql
-- Los 5 productos con mayor precio unitario
SELECT
    nombre_producto,
    categoria,
    precio_unitario
FROM productos
ORDER BY precio_unitario DESC
LIMIT 5;
```

**Paso 4.2 — Top 5 productos más económicos**

```sql
-- Los 5 productos con menor precio unitario
SELECT
    nombre_producto,
    categoria,
    precio_unitario
FROM productos
ORDER BY precio_unitario ASC
LIMIT 5;
```

**Paso 4.3 — Top 10 ventas de mayor monto**

```sql
-- Las 10 transacciones con el mayor monto total registrado
SELECT
    id_venta,
    id_cliente,
    id_producto,
    monto_total,
    fecha_venta
FROM ventas
ORDER BY monto_total DESC
LIMIT 10;
```

**Paso 4.4 — Las 3 ventas más recientes**

```sql
-- Las 3 transacciones más recientes registradas en el sistema
SELECT
    id_venta,
    id_cliente,
    id_producto,
    monto_total,
    fecha_venta
FROM ventas
ORDER BY fecha_venta DESC
LIMIT 3;
```

**Paso 4.5 — Top 10 productos con menor stock disponible**

```sql
-- Los 10 productos más cercanos a quedarse sin stock
-- Útil para alertas de reabastecimiento
SELECT
    nombre_producto,
    categoria,
    stock_disponible,
    precio_unitario
FROM productos
ORDER BY stock_disponible ASC
LIMIT 10;
```

**Paso 4.6 — Top N combinado con WHERE: Top 5 ventas más grandes del Q1 2024**

```sql
-- Las 5 ventas de mayor monto ocurridas en el primer trimestre de 2024
SELECT
    id_venta,
    id_cliente,
    monto_total,
    fecha_venta
FROM ventas
WHERE fecha_venta >= '2024-01-01'
  AND fecha_venta <= '2024-03-31'
ORDER BY monto_total DESC
LIMIT 5;
```

#### Resultado esperado

- **Paso 4.1:** Exactamente 5 filas con los productos de mayor precio. El producto más caro está en la fila 1.
- **Paso 4.2:** Exactamente 5 filas con los productos más económicos. El más barato está en la fila 1.
- **Paso 4.3:** Exactamente 10 filas con las transacciones de mayor valor monetario.
- **Paso 4.4:** Exactamente 3 filas. La fila 1 corresponde a la venta más reciente en el dataset.
- **Paso 4.5:** Exactamente 10 filas. La fila 1 corresponde al producto con menos unidades en stock.
- **Paso 4.6:** Exactamente 5 filas (o menos, si hay menos de 5 ventas en ese período), todas dentro del rango de fechas del Q1 2024.

#### Verificación

- [ ] ¿El Paso 4.1 devuelve exactamente 5 filas?
- [ ] ¿El valor de `precio_unitario` en la fila 1 del Paso 4.1 es mayor que el de la fila 2, y así sucesivamente?
- [ ] ¿En el Paso 4.4, la fecha de la fila 1 es más reciente que la fecha de la fila 2?
- [ ] ¿En el Paso 4.6, todas las fechas mostradas están entre el 1 de enero y el 31 de marzo de 2024?

> **📌 Patrón Top N:** El patrón `ORDER BY columna DESC LIMIT N` es la forma estándar de obtener los "mejores N" registros. Para los "peores N" (mínimos), simplemente cambia `DESC` por `ASC`. Este patrón es equivalente a `TOP N` en SQL Server o `ROWNUM` en Oracle.

> **⚠️ Importante sobre empates:** Si los registros en las posiciones N y N+1 tienen el mismo valor en la columna de ordenamiento, `LIMIT` corta arbitrariamente. En un análisis real, deberías ser consciente de esta situación. En lecciones futuras aprenderás a manejar empates con funciones de ventana (`RANK`, `DENSE_RANK`).

---

### Ejercicio 5: Desafío integrador — Reporte de rendimiento de ventas

**Objetivo:** Integrar todos los conceptos aprendidos en este laboratorio para construir un mini-reporte de rendimiento que simula una necesidad real de negocio.

**Contexto del caso:** El gerente de ventas necesita tres reportes rápidos para la reunión del lunes:
1. Los **5 clientes que más gastaron** en total (basado en la tabla de ventas).
2. El **historial de las 10 ventas más recientes** con todos sus detalles.
3. Los **3 productos más caros dentro de la categoría con mayor variedad de productos** (usa la categoría que identifiques en el sub-paso 5.1).

#### Instrucciones

**Paso 5.1 — Identificar las categorías disponibles**

```sql
-- Ver todas las categorías únicas ordenadas alfabéticamente
-- Esto te ayudará a seleccionar la categoría correcta en el Paso 5.3
SELECT DISTINCT
    categoria,
    COUNT(*) AS total_productos
FROM productos
GROUP BY categoria
ORDER BY total_productos DESC;
```

> **Nota:** `GROUP BY` y `COUNT(*)` se estudiarán en detalle en el Lab 05. Por ahora, úsalos como están escritos para obtener el conteo de productos por categoría. El resultado te mostrará qué categoría tiene más productos.

**Paso 5.2 — Reporte 1: Top 5 clientes por gasto total**

```sql
-- Top 5 clientes con mayor gasto acumulado en todas sus compras
SELECT
    id_cliente,
    SUM(monto_total) AS gasto_total,
    COUNT(id_venta)  AS numero_compras
FROM ventas
GROUP BY id_cliente
ORDER BY gasto_total DESC
LIMIT 5;
```

> **Nota:** `SUM()` y `COUNT()` son funciones de agregación que se estudiarán en el Lab 05. Úsalas como están escritas. El foco aquí es el `ORDER BY gasto_total DESC LIMIT 5`.

**Paso 5.3 — Reporte 2: Las 10 ventas más recientes con detalle completo**

```sql
-- Las 10 ventas más recientes registradas en el sistema
SELECT
    id_venta,
    id_cliente,
    id_producto,
    monto_total,
    fecha_venta
FROM ventas
ORDER BY fecha_venta DESC, monto_total DESC
LIMIT 10;
```

**Paso 5.4 — Reporte 3: Top 3 productos más caros de una categoría específica**

```sql
-- Reemplaza 'NOMBRE_CATEGORIA' con la categoría de mayor variedad
-- identificada en el Paso 5.1
SELECT
    nombre_producto,
    categoria,
    precio_unitario,
    stock_disponible
FROM productos
WHERE categoria = 'NOMBRE_CATEGORIA'
ORDER BY precio_unitario DESC
LIMIT 3;
```

**Paso 5.5 — Reflexión final: ¿Qué cambia si usamos ASC en lugar de DESC?**

Ejecuta la siguiente variante del Reporte 1 y compara los resultados:

```sql
-- ¿Qué obtenemos si ordenamos ASC en lugar de DESC?
SELECT
    id_cliente,
    SUM(monto_total) AS gasto_total,
    COUNT(id_venta)  AS numero_compras
FROM ventas
GROUP BY id_cliente
ORDER BY gasto_total ASC   -- Cambiado de DESC a ASC
LIMIT 5;
```

#### Resultado esperado

- **Paso 5.1:** Una lista de categorías con su conteo de productos. La primera fila indica la categoría con más productos.
- **Paso 5.2:** 5 filas mostrando los clientes con mayor gasto acumulado. El cliente con más gasto aparece primero.
- **Paso 5.3:** 10 filas con las ventas más recientes. Dos ventas del mismo día se desempatan por monto (mayor primero).
- **Paso 5.4:** 3 filas con los productos más caros de la categoría seleccionada.
- **Paso 5.5:** Los mismos 5 clientes del Paso 5.2 pero ahora son los de **menor** gasto acumulado. Esto demuestra el impacto crítico de elegir correctamente `ASC` vs `DESC` en un análisis Top N.

#### Verificación

- [ ] ¿El Paso 5.2 devuelve exactamente 5 filas con valores de `gasto_total` en orden decreciente?
- [ ] ¿El Paso 5.3 devuelve exactamente 10 filas con la fecha más reciente en la primera fila?
- [ ] ¿El Paso 5.4 devuelve solo productos de la categoría que especificaste?
- [ ] ¿Los resultados del Paso 5.5 son **diferentes** a los del Paso 5.2? (Deberían ser los clientes de menor gasto, no los de mayor.)

---

## 7. Validación y Pruebas Globales

Una vez completados todos los ejercicios, ejecuta las siguientes consultas de validación para confirmar que dominas los conceptos del laboratorio.

### Prueba de Validación 1: Verificar dominio de ORDER BY básico

```sql
-- Esta consulta debe devolver los productos ordenados correctamente
-- Criterio: primero por categoría A→Z, luego por precio Z→A
SELECT
    nombre_producto,
    categoria,
    precio_unitario
FROM productos
ORDER BY categoria ASC, precio_unitario DESC;

-- AUTOEVALUACIÓN: Verifica manualmente que:
-- 1. Los productos de la categoría que aparece primero alfabéticamente están al inicio
-- 2. Dentro de esa categoría, el producto más caro está en la fila 1
-- 3. El cambio de categoría "resetea" el orden de precios
```

### Prueba de Validación 2: Verificar patrón Top N

```sql
-- Obtener el Top 3 de ventas más antiguas con monto mayor a 100
SELECT
    id_venta,
    monto_total,
    fecha_venta
FROM ventas
WHERE monto_total > 100
ORDER BY fecha_venta ASC
LIMIT 3;

-- AUTOEVALUACIÓN:
-- 1. ¿Hay exactamente 3 filas (o menos si el dataset tiene pocos registros)?
-- 2. ¿Todos los montos son mayores a 100?
-- 3. ¿La fecha de la fila 1 es la más antigua entre los registros filtrados?
```

### Prueba de Validación 3: Verificar comprensión de la dirección del ordenamiento

```sql
-- Ejecuta estas dos consultas y confirma que los resultados son opuestos
-- Consulta A: Top 1 venta más cara
SELECT id_venta, monto_total FROM ventas ORDER BY monto_total DESC LIMIT 1;

-- Consulta B: Top 1 venta más barata
SELECT id_venta, monto_total FROM ventas ORDER BY monto_total ASC LIMIT 1;

-- AUTOEVALUACIÓN:
-- El monto de la Consulta A debe ser MAYOR que el de la Consulta B
-- Si son iguales, el dataset podría tener todos los montos idénticos (consulta al instructor)
```

### Lista de verificación final

Antes de dar por completado el laboratorio, confirma los siguientes puntos:

- [ ] Ejecuté al menos una consulta con `ORDER BY columna ASC` y una con `ORDER BY columna DESC`.
- [ ] Ejecuté al menos una consulta con `ORDER BY` usando dos columnas simultáneamente.
- [ ] Ejecuté al menos una consulta combinando `WHERE` + `ORDER BY`.
- [ ] Ejecuté al menos tres consultas con el patrón Top N (`ORDER BY` + `LIMIT`).
- [ ] Puedo explicar con mis propias palabras la diferencia entre `ASC` y `DESC` y cuándo usar cada uno.
- [ ] Puedo explicar por qué `ORDER BY` siempre va al final de la consulta.

---

## 8. Resolución de Problemas

### Problema 1: El orden de los resultados no cambia al agregar ORDER BY

**Síntoma:** Ejecutas una consulta con `ORDER BY precio_unitario DESC` pero los resultados parecen estar en el mismo orden que sin `ORDER BY`. No hay un error visible en Snowsight.

**Causa probable:** Existen dos causas comunes:
- **Causa A:** La columna especificada en `ORDER BY` tiene todos los valores idénticos o muy similares, por lo que el orden parece no cambiar visualmente.
- **Causa B:** Estás mirando una vista previa limitada (los primeros N registros) que por coincidencia tiene los mismos valores. El ordenamiento sí se aplicó, pero no es visible en la muestra.

**Solución:**

```sql
-- Paso 1: Verifica que la columna tiene valores variados
SELECT
    MIN(precio_unitario) AS precio_minimo,
    MAX(precio_unitario) AS precio_maximo,
    COUNT(DISTINCT precio_unitario) AS valores_unicos
FROM productos;

-- Si precio_minimo = precio_maximo, todos los precios son iguales
-- y el ORDER BY no producirá un cambio observable.

-- Paso 2: Si los valores son variados, verifica tu sintaxis
-- Asegúrate de que no hay un punto y coma (;) antes del ORDER BY
-- INCORRECTO:
-- SELECT nombre_producto FROM productos; ORDER BY precio_unitario DESC;

-- CORRECTO:
SELECT nombre_producto, precio_unitario
FROM productos
ORDER BY precio_unitario DESC;
```

---

### Problema 2: LIMIT devuelve menos filas de las esperadas

**Síntoma:** Ejecutas `ORDER BY monto_total DESC LIMIT 10` pero solo obtienes 3, 5 o 7 filas en lugar de 10.

**Causa probable:** La tabla tiene menos registros que el número especificado en `LIMIT`, **o** la cláusula `WHERE` anterior al `ORDER BY` filtró la mayoría de los registros, dejando menos filas disponibles que el límite solicitado.

**Solución:**

```sql
-- Paso 1: Cuenta cuántos registros hay en total en la tabla
SELECT COUNT(*) AS total_registros FROM ventas;

-- Paso 2: Si tienes WHERE, cuenta cuántos registros pasan el filtro
SELECT COUNT(*) AS registros_filtrados
FROM ventas
WHERE monto_total > 500;  -- Reemplaza con tu condición real

-- Si registros_filtrados < N (tu LIMIT), es normal que obtengas menos filas.
-- LIMIT devuelve "hasta N filas", no exactamente N filas.

-- Paso 3: Si el total de la tabla es menor a tu LIMIT, simplemente elimina el LIMIT
-- para ver todos los registros disponibles, o ajusta el LIMIT a un número menor.
SELECT id_venta, monto_total, fecha_venta
FROM ventas
ORDER BY monto_total DESC;  -- Sin LIMIT para ver todos los registros
```

> **Regla importante:** `LIMIT N` significa "devuelve **como máximo** N filas". Si la tabla (o el resultado filtrado) tiene menos de N filas, Snowflake devuelve todas las disponibles sin generar un error.

---

## 9. Limpieza del Entorno

Al finalizar el laboratorio, realiza los siguientes pasos para liberar recursos y minimizar el consumo de créditos Snowflake:

```sql
-- 1. Suspender el Virtual Warehouse para detener el consumo de créditos
--    (El warehouse se reactivará automáticamente cuando ejecutes la próxima consulta)
ALTER WAREHOUSE COMPUTE_WH SUSPEND;
```

> **⚠️ Importante:** No elimines las tablas `VENTAS`, `PRODUCTOS` o `CLIENTES` del schema `CURSO_SQL`. Estos datos son necesarios para los laboratorios siguientes (Lab 04 en adelante).

**Verificación de limpieza:**

```sql
-- Confirmar que el warehouse está suspendido
SHOW WAREHOUSES LIKE 'COMPUTE_WH';
-- Busca el valor 'SUSPENDED' en la columna 'state'
```

**Lista de limpieza:**

- [ ] Virtual Warehouse suspendido.
- [ ] Worksheet guardado con un nombre descriptivo (ej. `Lab03_TopN_[TuNombre]`).
- [ ] Ninguna tabla del dataset eliminada.
- [ ] Sesión de Snowsight cerrada si no continuarás trabajando.

---

## 10. Resumen

### Conceptos cubiertos en este laboratorio

En este laboratorio aplicaste los fundamentos del ordenamiento SQL en Snowflake para construir análisis Top N reales. Los conceptos clave que practicaste fueron:

| Concepto                          | Sintaxis clave                                      | Caso de uso típico                            |
|-----------------------------------|-----------------------------------------------------|-----------------------------------------------|
| Orden ascendente                  | `ORDER BY columna ASC`                              | Productos de menor a mayor precio             |
| Orden descendente                 | `ORDER BY columna DESC`                             | Ventas de mayor a menor monto                 |
| Ordenamiento por fecha            | `ORDER BY fecha DESC`                               | Transacciones más recientes primero           |
| Múltiples criterios               | `ORDER BY col1 ASC, col2 DESC`                      | Categoría A→Z, precio Z→A dentro de categoría|
| WHERE + ORDER BY                  | `WHERE condición ORDER BY columna`                  | Filtrar y luego ordenar resultados            |
| Patrón Top N (mejores)            | `ORDER BY columna DESC LIMIT N`                     | Top 10 clientes por gasto                     |
| Patrón Top N (peores/menores)     | `ORDER BY columna ASC LIMIT N`                      | 5 productos con menor stock                   |

### Reglas esenciales para recordar

1. **`ORDER BY` siempre va al final** de la consulta, después de `WHERE`.
2. **Sin dirección especificada**, Snowflake asume `ASC` por defecto.
3. **`LIMIT` sin `ORDER BY`** devuelve filas arbitrarias, no las "primeras" en ningún sentido significativo. Siempre combínalos.
4. **`LIMIT N` devuelve hasta N filas**, no exactamente N. Si hay menos registros disponibles, devuelve todos.
5. **Los valores `NULL`** aparecen al final en orden `ASC` y al inicio en orden `DESC` en Snowflake.

### Conexión con el siguiente laboratorio

En el **Lab 04**, aplicarás funciones de texto y fecha para transformar y formatear los datos antes de presentarlos. Combinarás estas transformaciones con `ORDER BY` para construir reportes más sofisticados donde el ordenamiento se aplica sobre valores calculados o formateados, no solo sobre columnas originales.

---

### Recursos adicionales

| Recurso                                                                 | Descripción                                                  |
|-------------------------------------------------------------------------|--------------------------------------------------------------|
| [Documentación Snowflake: ORDER BY](https://docs.snowflake.com/en/sql-reference/constructs/order-by) | Referencia oficial con todas las opciones y comportamientos |
| [Documentación Snowflake: LIMIT / FETCH](https://docs.snowflake.com/en/sql-reference/constructs/limit) | Detalles sobre LIMIT y su equivalente FETCH FIRST N ROWS    |
| [SQLBolt — Lección ORDER BY y LIMIT](https://sqlbolt.com/lesson/filtering_sorting_query_results) | Tutorial interactivo con ejercicios en el navegador          |
| [W3Schools SQL ORDER BY](https://www.w3schools.com/sql/sql_orderby.asp) | Referencia rápida con ejemplos editables en línea            |
| [Mode Analytics: Sorting Data](https://mode.com/sql-tutorial/sql-order-by/) | Tutorial con contexto de análisis de negocio                |

---

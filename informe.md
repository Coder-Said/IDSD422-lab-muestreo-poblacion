# Informe — Clasificación de variables y población vs. muestra

**Para:** Gerencia de RetailAndina S.A. · **De:** Equipo de Business Intelligence
**Fecha:** 21/10/2026 · **Asunto:** Base de fidelización: tipos de variables y estimación por muestreo

---

## Resumen ejecutivo

Clasificamos las 7 variables de la base de fidelización: 3 son numéricas (edad, número de hijos, ingresos) y 4 son categóricas, de las cuales 2 son ordinales (nivel de educación y satisfacción). Después estimamos el ingreso mensual promedio de los clientes con muestras de 20 tomadas de tres formas distintas, y comparamos cada resultado con el valor real de la base completa: **1330.72 USD**. La muestra por conveniencia (las primeras 20 filas) solo tomó clientes de Quito, y el sorteo aleatorio dio una cifra distinta en cada repetición (entre 1235 y 1491.50 USD). Antes de decidir con una muestra, la gerencia debe exigir que esté bien diseñada (por sorteo y, si importan las ciudades, estratificada) y que se reporte con su margen de error. Que la muestra tenga muchas filas no basta.

## Parte A — Clasificación de las variables

La verificación se hizo en Python (`dtypes`, `unique()`, `CategoricalDtype`) y en R (`str()`, `unique()`, `factor(..., ordered = TRUE)`), con los mismos resultados.

| Variable | Tipo | Justificación (criterio) | Verificación en código |
|---|---|---|---|
| `cliente_id` | Identificador (nominal, no analítica) | Es una etiqueta única por cliente; no tiene sentido promediarla ni ordenarla. | dtype `str`/`object`; 12 únicos = 12 filas. |
| `edad` | Cuantitativa continua (razón) | Tiene sentido la mitad del valor (24 es la mitad de 48); se registra en años enteros, pero el tiempo vivido es continuo y solo se redondea al medir. | dtype `int64`; 12 únicos; rango 22–48. |
| `numero_hijos` | Cuantitativa discreta | Resulta de contar: solo toma valores enteros (no existen 1.5 hijos). | dtype `int64`; 5 únicos. |
| `genero` | Cualitativa nominal | Categorías sin orden entre sí. | dtype `str`/`object`; 3 únicos (Femenino, Masculino, Otro). |
| `nivel_educacion` | Cualitativa ordinal | Hay orden (Primaria < Secundaria < Universidad < Posgrado) pero la distancia entre niveles no es medible. | dtype `str`/`object` → category ordenada; 4 únicos; mín. = Primaria, máx. = Posgrado. |
| `ciudad` | Cualitativa nominal | Quito, Guayaquil y Cuenca no tienen un orden natural. | dtype `str`/`object`; 3 únicos. |
| `ingresos_mensuales` | Cuantitativa continua (razón) | Es dinero, mide una magnitud con cero real; 2 000 es el doble de 1 000. Se reporta en dólares enteros, pero puede tomar cualquier valor. | dtype `int64`; 12 únicos. |
| `satisfaccion` | Cualitativa ordinal | Escala 1–5: hay orden, pero la distancia entre 2 y 3 no es necesariamente igual a la de 4 y 5; la mitad de un 4 no es un 2. | dtype `int64` (engaña) → category ordenada 1<…<5; 4 valores observados (2 a 5). |

**P1.** ¿Qué te dijo el dtype de `satisfaccion` y por qué no basta para clasificarla? ¿Qué pasa si una ordinal queda como texto sin declarar el orden?

El dtype de `satisfaccion` fue `int64`, es decir, se leyó como número. Pero el dtype solo dice cómo se almacena, no qué significa, así que calcular su media o decir que un 4 es el doble de 2 no es válido.

Con `nivel_educacion`, que llegó como texto, al ordenar sin declarar el orden se obtuvo un orden alfabético, donde Posgrado queda antes que Primaria; para solucionarlo era necesario declararlo con `CategoricalDtype`.

## Parte B — Población vs. muestra

### La población y su parámetro

| Concepto | Valor |
|---|---|
| Población (N) | 400 clientes (Quito 200 · Guayaquil 120 · Cuenca 80) |
| Variable de interés | `ingresos_mensuales` (ingreso mensual en USD) |
| Parámetro poblacional (media) | **1330.72 USD** |

*¿Por qué se muestrea si aquí el parámetro existe y se calculó?*

RetailAndina S.A. necesita demostrar que una muestra de 20 clientes puede representar a los 400 registros de la base. Por eso se calculó primero el parámetro poblacional y luego se comparó contra él cada uno de los tres diseños de muestreo (conveniencia, aleatorio simple y estratificado).

### Los tres diseños

Python y R usan generadores de números aleatorios distintos, así que con la misma semilla sortean clientes diferentes. Por eso se reportan ambos resultados (R entre paréntesis).

| Diseño | n | Media estimada | Composición por ciudad | Diferencia vs. parámetro |
|---|---|---|---|---|
| Conveniencia (`head`) | 20 | 1348.50 USD (R: 1348.50) | Quito 20 | +17.78 (R: +17.78) |
| Aleatorio simple (`sample`, semilla 1) | 20 | 1336.00 USD (R: 1318.00) | Quito 8 · Cuenca 8 · Guayaquil 4 (R: Quito 8 · Guayaquil 7 · Cuenca 5) | +5.28 (R: −12.72) |
| Estratificado (`groupby` + 5 %) | 20 | 1287.50 USD (R: 1226.00) | Quito 10 · Guayaquil 6 · Cuenca 4 | −43.22 (R: −104.72) |

*Medias de 5 sorteos distintos del aleatorio simple (semillas 1 a 5):*

| Semilla | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| Media (Python) | 1336.00 | 1405.00 | 1235.00 | 1491.50 | 1288.00 |
| Media (R) | 1318.00 | 1287.50 | 1233.00 | 1385.50 | 1283.00 |

**P2.** ¿Cuánto vale el parámetro poblacional y cuánto estimó cada diseño? ¿Cuál quedó más cerca y por qué eso **no** alcanza para declarar un ganador?

El parámetro es 1330.72 USD. En Python, la conveniencia estimó 1348.50 (+17.78), el aleatorio simple 1336.00 (+5.28) y el estratificado 1287.50 (−43.22). El más cercano fue el aleatorio simple con la semilla 1, pero eso no alcanza para declarar un ganador. Es un solo sorteo: con las semillas 2 a 5 el mismo diseño dio entre 1235.00 y 1491.50, a veces más lejos que el estratificado. Además, la conveniencia quedó más cerca que el estratificado aunque todos sus clientes son de Quito.

**P3.** La muestra por conveniencia también tiene 20 filas. ¿Qué defecto tiene y cómo lo delatas en la composición por ciudad? ¿Se corrige tomando más filas?

La muestra por conveniencia tiene sesgo de selección. Su composición fue 20 de 20 de Quito, sin un solo cliente de Guayaquil ni de Cuenca, que son la mitad de la población. El archivo está ordenado por ciudad, así que `head()` no es un corte al azar: toma a "los primeros que llegaron". No se corrige con más filas, porque las primeras 200 son todas de Quito. El defecto está en la forma de elegir, no en el tamaño.

**P4.** El sorteo aleatorio dio un número distinto en cada semilla. ¿Qué implica esa variabilidad para una empresa que va a decidir con un solo estudio?

Con el mismo diseño y el mismo tamaño, las cinco medias de Python fueron de 1235.00 a 1491.50 USD (diferencias de −95.72 a +160.78), y las de R de 1233.00 a 1385.50. Esa es la variabilidad del sorteo, que, a diferencia del sesgo, no empuja siempre hacia el mismo lado. Para una empresa que decide con un solo estudio, eso significa que su cifra puede ser una de las "afortunadas" o de las "desafortunadas" sin que lo sepa.

**P5.** ¿Cuándo usarías cada diseño en la práctica? Relaciona cada uno con un uso real (piloto, subgrupos que deben aparecer, conclusión sobre toda la población).

- **Conveniencia:** solo para un **piloto**, por ejemplo para probar un cuestionario o el flujo de datos, sin sacar conclusiones.
- **Aleatorio simple:** para una **conclusión sobre toda la población** cuando se tiene el listado completo de clientes.
- **Estratificado:** cuando hay **subgrupos que deben aparecer**, como las ciudades: garantiza 10, 6 y 4 clientes de Quito, Guayaquil y Cuenca, algo que el sorteo simple no asegura.

## Conclusiones

1. **Los tipos de datos deben manejarse como pistas dentro del análisis.** Ciertos errores de interpretación de cada lenguaje pueden eliminar datos sin mostrar errores si no se clasifica de buena manera.
2. **El diseño elegido determina qué tan representativa es la muestra.** A pesar de que todas tenían el mismo tamaño (20), indicaron resultados distintos.
3. **El azar de cada lenguaje produce variación, pero el sesgo no desaparece con más filas;** todo depende de la elección del diseño.

**Respuesta a la gerencia y alcance:** el ingreso mensual promedio de la base es 1330.72 USD. Una muestra de 20 puede estimarlo solo si se elige el sorteo como diseño, estratificada por ciudad si los resultados por ciudad importan, siempre y cuando se reporten con su margen de error para una correcta interpretación.

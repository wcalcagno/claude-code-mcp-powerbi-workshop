# Ejercicio 1 · Explorar el modelo

⏱️ **Tiempo estimado:** 10 minutos
📍 **Bloque:** [02 · MCP Power BI local](../README.md)

---

## Objetivo

Usar Claude Code para **entender la estructura de un modelo de Power BI que
tienes abierto en Power BI Desktop**: sus tablas, sus columnas y sus relaciones,
sin hacer un solo clic en la vista de modelo.

Es exactamente lo que harías al heredar el `.pbix` de un colega: ¿qué hay acá
adentro y cómo está armado?

---

## Antes de empezar

- [ ] **Power BI Desktop abierto con un `.pbix` cargado y con datos.**
- [ ] Claude Code abierto en la carpeta del workshop.
- [ ] `/mcp` muestra `powerbi-local` como `connected`.

> 🔴 Si `/mcp` no muestra el servidor conectado, no sigas: revisa el
> [troubleshooting del bloque](../README.md#troubleshooting). El 90% de las veces
> es que Power BI Desktop no tiene un reporte abierto.

---

## Paso 1 · Ver qué sabe hacer el servidor

Antes de pedir nada, mira qué herramientas ganó Claude Code al conectarse.

### Prompt sugerido

```
¿Qué herramientas del servidor MCP powerbi-local tienes disponibles?
Lista el nombre de cada una y explícame en una línea, en español,
qué hace cada una.
```

### Resultado esperado

Una lista de herramientas con nombres del estilo `list_tables`,
`get_table_columns`, `list_measures`, `execute_dax_query`, `get_relationships`
(los nombres exactos dependen de la versión del servidor), cada una con una
explicación en una línea.

**Fíjate en algo importante:** todos los verbos son de lectura — `list`, `get`,
`execute` (de consulta), `describe`. **No hay ningún `create`, `update` ni
`delete`.** Eso es el "solo lectura" hecho visible: no es una promesa, es lo
único que el servidor sabe hacer.

---

## Paso 2 · Listar las tablas del modelo

### Prompt sugerido

```
Usa el servidor MCP powerbi-local para listar todas las tablas del modelo
que tengo abierto en Power BI Desktop. Muéstramelas en una tabla con dos
columnas: nombre de la tabla y si parece ser de hechos o de dimensiones,
según su nombre y contenido.
```

### Resultado esperado

Claude pide permiso para usar la herramienta del MCP. Apruébalo. Luego devuelve
algo como:

| Tabla | Tipo probable |
|---|---|
| `Ventas` | Hechos |
| `Fecha` | Dimensión |
| `Producto` | Dimensión |
| `Cliente` | Dimensión |
| `Medidas` | Tabla de medidas |

Los nombres serán los **de tu modelo**, no estos. Si tu modelo tiene muchas
tablas, puede listarlas todas — está bien.

> 💡 **Contrástalo con la realidad:** abre la vista de Modelo en Power BI Desktop
> y compara. Deberían coincidir exactamente.

---

## Paso 3 · Explorar las columnas de una tabla

Elige una tabla del listado anterior — idealmente la de hechos principal.

### Prompt sugerido

```
Muéstrame todas las columnas de la tabla Ventas: nombre, tipo de dato y
si es una columna calculada o viene del origen. Ordénalas por nombre.
```

> 📌 Reemplaza `Ventas` por el nombre real de una tabla de tu modelo.

### Resultado esperado

Una tabla con las columnas del modelo, por ejemplo:

| Columna | Tipo de dato | Origen |
|---|---|---|
| `Cantidad` | Int64 | Origen |
| `FechaVenta` | DateTime | Origen |
| `IdProducto` | Int64 | Origen |
| `MontoNeto` | Decimal | Origen |
| `MontoTotal` | Decimal | Calculada |

Si el servidor no expone el dato de "calculada vs. origen", Claude te lo dirá en
vez de inventarlo. Eso también es una respuesta correcta.

---

## Paso 4 · Ver las relaciones del modelo

Esta es la parte donde más tiempo ahorras: entender cómo se conecta todo.

### Prompt sugerido

```
Lista todas las relaciones del modelo. Para cada una indícame:
tabla origen, columna origen, tabla destino, columna destino,
cardinalidad, dirección del filtro y si está activa.
Después dime en dos o tres frases si el modelo sigue un esquema
de estrella o si hay algo que te llame la atención.
```

### Resultado esperado

Primero, una tabla de relaciones:

| Origen | Columna | Destino | Columna | Cardinalidad | Dirección | Activa |
|---|---|---|---|---|---|---|
| `Ventas` | `IdProducto` | `Producto` | `IdProducto` | Muchos a uno | Simple | Sí |
| `Ventas` | `FechaVenta` | `Fecha` | `Fecha` | Muchos a uno | Simple | Sí |
| `Ventas` | `FechaDespacho` | `Fecha` | `Fecha` | Muchos a uno | Simple | **No** |

Y luego un comentario del estilo: *"El modelo sigue un esquema de estrella con
`Ventas` como tabla de hechos. Hay una relación inactiva entre `FechaDespacho` y
la tabla `Fecha`, lo que sugiere el uso de `USERELATIONSHIP` en alguna medida."*

> 💡 **Esa última observación es el valor real.** Detectar relaciones inactivas,
> filtros bidireccionales o cardinalidades muchos-a-muchos leyendo la vista de
> modelo a ojo toma bastante más tiempo.

---

## Paso 5 · Pedir un resumen ejecutivo

### Prompt sugerido

```
Con todo lo que exploraste, hazme un resumen del modelo en 5 puntos,
como si se lo tuviera que explicar a alguien que lo va a mantener:
cuál es la tabla de hechos, qué dimensiones hay, cómo se relacionan,
qué granularidad parece tener y cualquier riesgo que veas en el diseño.
No inventes nada que no hayas leído del modelo.
```

### Resultado esperado

Cinco puntos concretos referidos a **tu** modelo. La frase final del prompt
("no inventes nada") es importante: obliga a Claude a distinguir entre lo que
leyó y lo que supone.

---

## Comprueba que efectivamente fue solo lectura

### Prompt sugerido

```
¿Podrías crear una medida nueva en este modelo usando el MCP powerbi-local?
Responde solo con lo que las herramientas disponibles te permiten hacer.
```

### Resultado esperado

Claude responde que **no**: las herramientas del servidor son exclusivamente de
lectura y consulta, y no existe ninguna operación de creación o modificación.
Puede ofrecerte, como alternativa, **escribir la expresión DAX en un archivo**
para que tú la copies y pegues en Power BI Desktop.

Esa es precisamente la división de responsabilidades del taller:
**el MCP lee, Claude Code escribe archivos, y quien modifica el modelo eres tú.**

---

## ✅ Checklist

- [ ] Viste la lista de herramientas del MCP y notaste que todas son de lectura.
- [ ] Obtuviste el listado de tablas y lo contrastaste con Power BI Desktop.
- [ ] Exploraste las columnas de al menos una tabla.
- [ ] Obtuviste el mapa de relaciones.
- [ ] Confirmaste que el servidor no puede modificar el modelo.

---

➡️ **Siguiente:** [Ejercicio 2 · Consultar DAX](./02-consultar-dax.md)

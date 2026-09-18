# Ejercicio 2 · Consultar un modelo del Power BI Service / Fabric

⏱️ **Tiempo estimado:** 18 minutos
📍 **Bloque:** [03 · MCP Fabric / API](../README.md)

---

## Objetivo

Obtener el **ID de un semantic model** publicado en tu workspace, leer su
**esquema** y ejecutar una **consulta DAX contra el Service** — no contra tu
Power BI Desktop.

---

## Antes de empezar

- [ ] `/mcp` muestra `powerbi-fabric` como `connected`.
- [ ] Tienes permiso **Build** sobre al menos un semantic model.
- [ ] Estás en la carpeta del workshop.

> 💡 **Ya no necesitas Power BI Desktop abierto para este ejercicio.** Pero déjalo
> abierto igual: lo vamos a usar en el [bloque 4](../../04-flujo-completo/README.md).

---

## Paso 1 · Obtener el ID de tu semantic model

Acá hay algo que sorprende a todo el mundo, así que vale la pena decirlo directo:

> 🔴 **Este servidor no tiene una herramienta para listar tus workspaces ni tus
> semantic models.** No puedes pedirle "muéstrame mis workspaces". Trabaja sobre
> **un modelo que tú identificas por su ID**.

No es un olvido de Microsoft: el servidor está pensado para agentes que ya saben
sobre qué modelo trabajan. El descubrimiento lo haces tú, en el navegador.

### Cómo sacar el ID

1. Abre <https://app.powerbi.com>.
2. Entra al workspace que te interesa.
3. Haz clic en el **semantic model** (antes *dataset*) para abrir su página.
4. Mira la URL del navegador:

   ```
   https://app.powerbi.com/groups/{workspaceId}/datasets/{semanticModelId}
   ```

   | Parte | Qué es |
   |---|---|
   | `{workspaceId}` | El GUID de tu workspace → `TU_WORKSPACE_ID` |
   | `{semanticModelId}` | El GUID del modelo → `TU_DATASET_ID` ⬅️ **el que necesitas** |

5. **Copia el `semanticModelId`.** Es un GUID con formato
   `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

### Guárdalo donde Claude Code pueda verlo

En vez de pegarlo en cada prompt, déjalo en un archivo del repositorio.

#### Prompt sugerido

```
Crea el archivo 03-mcp-fabric-api/mis-modelos.md con una tabla que tenga
las columnas: alias corto, nombre del workspace, nombre del semantic model
y su ID. Agrega esta fila:

  alias: ventas-prod
  workspace: Ventas - Producción
  modelo: Ventas Corporativas
  id: PEGA_AQUI_TU_SEMANTIC_MODEL_ID

Agrega arriba una nota explicando que estos IDs no son secretos (identifican,
no autentican), pero que el acceso real depende de mis permisos en Entra ID.
```

> 📌 Reemplaza los valores por los tuyos. Un ID de semantic model **no es un
> secreto**: sin permisos sobre el modelo, no sirve de nada. Aun así, si tus
> nombres de workspace revelan información interna, considera no subir este
> archivo a un repositorio público.

**Resultado esperado:** el archivo creado. De aquí en adelante puedes decirle a
Claude *"usa el modelo ventas-prod"* y él busca el ID en el archivo.

---

## Paso 2 · Leer el esquema del modelo publicado

### Prompt sugerido

```
Usando el MCP powerbi-fabric, obtén el esquema del semantic model cuyo ID
está en @03-mcp-fabric-api/mis-modelos.md como "ventas-prod".

Muéstrame:
- Las tablas, indicando cuáles parecen de hechos y cuáles de dimensiones
- Las medidas, con su expresión DAX
- Las relaciones, con su cardinalidad

Preséntalo en tablas y no inventes nada que no venga en el esquema.
```

### Resultado esperado

Claude pide permiso para usar la herramienta **Get Semantic Model Schema**.
Apruébalo. Devuelve el esquema del modelo publicado: tablas, columnas, medidas,
relaciones, tipos de dato y jerarquías.

> 💡 **Si el autor del modelo preparó metadatos para IA** (descripciones,
> instrucciones, respuestas verificadas), también vienen en el esquema. Es la
> misma información que usa Copilot en Power BI, y mejora bastante la calidad de
> las respuestas. Vale la pena mirarlo: si tu modelo no tiene nada de eso, ya
> sabes qué agregar al volver al trabajo.

---

## Paso 3 · Comparar con lo que viste en el modelo local

### Prompt sugerido

```
Compara el esquema que acabas de leer del Service con el que exploramos
en el modelo local de Power BI Desktop en el bloque 2. ¿Son la misma
estructura o hay diferencias en tablas, medidas o relaciones?
Si hay diferencias, lístalas. Si son idénticos, dilo.
```

### Resultado esperado

Una comparación concreta. Posibles hallazgos:

- **Idénticos** → el `.pbix` local es el mismo que está publicado. Lo esperable.
- **El Service tiene medidas que el local no** → alguien editó el modelo
  directamente en la nube.
- **El local tiene medidas que el Service no** → tienes cambios sin publicar.

> 💡 Detectar esto a mano requiere abrir ambos y comparar a ojo. Es uno de los
> usos más prácticos de todo el taller.

---

## Paso 4 · Ejecutar una consulta DAX contra el Service

Este es el paso central del bloque.

### Prompt sugerido

```
Ejecuta una consulta DAX contra el semantic model ventas-prod en el Service,
que devuelva el total de ventas sin filtros. Usa la misma lógica que usamos
en el modelo local del bloque 2.

Antes de ejecutarla, muéstrame la consulta. Después dime el resultado con
separador de miles, e indícame explícitamente que este número viene del
SERVICE, no de Power BI Desktop.
```

### Resultado esperado

**Primero**, la consulta:

```dax
EVALUATE
    ROW(
        "Total Ventas",
        SUM( Ventas[MontoTotal] )
    )
```

**Después**, el resultado, identificado claramente:

```
Total Ventas (SERVICE - Ventas Corporativas): 12.435.890
```

> 📌 **La consulta DAX es exactamente la misma** que corriste en el bloque 2. El
> lenguaje no cambia: cambia dónde se ejecuta. Es el mismo motor de Analysis
> Services, en tu PC o en la nube de Microsoft.

> 🔐 **Tu RLS se aplica.** Si el modelo tiene Row-Level Security y tu usuario cae
> en un rol, el resultado viene filtrado — igual que si abrieras el reporte en el
> navegador. Eso es consecuencia directa de usar **identidad delegada**: con un
> *service principal*, Power BI **no** aplicaría RLS.

---

## Paso 5 · Una consulta agrupada, para tener algo comparable

### Prompt sugerido

```
Ahora ejecuta contra el Service la misma consulta agrupada por año que
hicimos en el modelo local, con el total de ventas por año ordenado de
mayor a menor. Guarda el resultado: lo vamos a comparar con el local
en el bloque siguiente.
```

### Resultado esperado

| Año | Total Ventas (SERVICE) |
|---|---|
| 2024 | 3.980.115 |
| 2023 | 5.220.110 |
| 2022 | 3.113.450 |

---

## Paso 6 · Leer la estructura de un reporte

Algo que el modelo local no te da de la misma forma: **cómo se usa realmente el
modelo** en un reporte publicado.

Para esto necesitas el **ID del reporte**, que sacas de su URL:

```
https://app.powerbi.com/groups/{workspaceId}/reports/{reportId}
```

### Prompt sugerido

```
Usando la herramienta de metadatos de reportes, léeme la estructura del
reporte con ID PEGA_AQUI_TU_REPORT_ID. Dime:
- Cuántas páginas tiene y cómo se llaman
- Qué visuales hay en cada página y qué campos usan
- Qué filtros están aplicados

Después dime qué medidas del modelo NO aparecen usadas en ningún visual.
```

### Resultado esperado

Las páginas, los visuales con sus campos y los filtros. Y, al final, la lista de
medidas que nadie está usando.

> ⭐ **Esa última pregunta es oro puro para limpiar un modelo.** Medidas que nadie
> usa son deuda técnica: alguien las creó para un análisis puntual y quedaron ahí.
>
> ⚠️ Con un matiz honesto: que una medida no aparezca en **este** reporte no
> significa que nadie la use. Puede estar en otro reporte, en un Excel conectado
> o en una app. Trata el resultado como **una lista de candidatas a revisar**,
> no como una orden de borrado.

---

## Paso 7 · Consultar sin gastar Copilot

El servidor incluye **Generate Query**, que genera DAX usando el motor de Copilot
de Power BI. Funciona bien, pero **requiere licencia de Copilot y consume
capacidad**.

### Prompt sugerido

```
Para el resto del taller, escribe tú el DAX directamente en vez de usar la
herramienta Generate Query, así no consumimos capacidad de Copilot.
Confírmame que entendiste y dime qué diferencia práctica tiene.
```

### Resultado esperado

Claude confirma que va a escribir el DAX por su cuenta y usar solo **Execute
Query** para ejecutarlo. La diferencia práctica: ninguna para el taller — es
exactamente lo que hizo en el bloque 2.

---

## Recordatorio: modo consulta

Todo lo que hicimos acá **lee**. Ninguna de las cuatro herramientas del servidor
de Consumption modifica nada.

Tres capas de protección, de la más fuerte a la más débil:

1. **El servidor elegido.** Usamos el endpoint de *Consumption*. El de *Authoring*
   —que sí escribe— quedó fuera del taller a propósito.
2. **Los permisos que pedimos.** La app de Entra ID solo tiene `Dataset.Read.All`
   y `Workspace.Read.All`. Nunca pedimos `SemanticModel.ReadWrite.All`.
3. **Tus permisos en Power BI.** No puedes hacer nada que tu cuenta no pudiera
   hacer ya desde el navegador, y tu RLS se respeta.

---

## ✅ Checklist

- [ ] Obtuviste el ID de un semantic model desde la URL del Service.
- [ ] Lo guardaste en `mis-modelos.md`.
- [ ] Leíste el esquema del modelo publicado.
- [ ] Lo comparaste con el modelo local del bloque 2.
- [ ] Ejecutaste al menos una consulta DAX contra el Service.
- [ ] Tienes a mano el resultado agrupado por año, para el bloque 4.

---

⬅️ **Anterior:** [Ejercicio 1 · Autenticación Entra ID](./01-autenticacion-entra-id.md)
➡️ **Siguiente:** [04 · Flujo completo](../../04-flujo-completo/README.md)

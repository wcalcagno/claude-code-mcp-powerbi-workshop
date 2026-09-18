# Ejercicio 2 · Consultar un workspace de Power BI Service / Fabric

⏱️ **Tiempo estimado:** 18 minutos
📍 **Bloque:** [03 · MCP Fabric / API](../README.md)

---

## Objetivo

Listar tus **workspaces** y **semantic models** en la nube, y ejecutar una
**consulta DAX contra el Service** — no contra tu Power BI Desktop. Es el mismo
tipo de pregunta del bloque 2, pero ahora contra el dato publicado.

---

## Antes de empezar

- [ ] `/mcp` muestra `powerbi-fabric` como `connected`.
- [ ] Sabes el nombre de al menos un workspace al que tengas acceso.
- [ ] Estás en la carpeta del workshop.

> 💡 **Ya no necesitas Power BI Desktop abierto para este ejercicio.** Pero
> déjalo abierto igual: lo vamos a usar en el [bloque 4](../../04-flujo-completo/README.md).

---

## Paso 1 · Listar tus workspaces

### Prompt sugerido

```
Usando el MCP powerbi-fabric, lista todos los workspaces de Power BI
Service / Fabric a los que tengo acceso. Muéstramelos en una tabla con:
nombre del workspace, su ID, el tipo de capacidad si está disponible,
y mi rol si el servidor lo expone.
```

### Resultado esperado

Claude pide permiso para usar la herramienta. Apruébalo. Luego devuelve algo como:

| Workspace | ID | Capacidad | Rol |
|---|---|---|---|
| `Mi área de trabajo` | `xxxxxxxx-xxxx-...` | Compartida | Admin |
| `Ventas - Producción` | `xxxxxxxx-xxxx-...` | Fabric F2 | Viewer |
| `Sandbox Analítica` | `xxxxxxxx-xxxx-...` | Premium PPU | Member |

Los workspaces serán **los tuyos**.

> ⭐ **Fíjate en esto:** esta lista es exactamente la misma que ves al entrar a
> <https://app.powerbi.com>. Ni uno más. **Eso es la autenticación delegada
> funcionando:** Claude Code está viendo el mundo con tus permisos, no con
> permisos de administrador.
>
> Si un compañero corre el mismo prompt, obtiene una lista distinta. Compáralo
> con quien tengas al lado — es la forma más rápida de entender el concepto.

**Anota el nombre y el ID del workspace** que vas a usar. En el resto del
ejercicio aparece como `TU_WORKSPACE_ID`.

---

## Paso 2 · Listar los contenidos del workspace

### Prompt sugerido

```
Lista los semantic models (datasets) que hay en el workspace
"Ventas - Producción". Para cada uno dime: nombre, ID, fecha de la última
actualización si está disponible, y si es un modelo importado o DirectQuery.
```

> 📌 Reemplaza el nombre del workspace por el tuyo.

### Resultado esperado

| Semantic model | ID | Última actualización | Modo |
|---|---|---|---|
| `Ventas Corporativas` | `xxxxxxxx-...` | 2026-09-17 06:00 | Import |
| `Inventario RT` | `xxxxxxxx-...` | — | DirectQuery |

> ⚠️ **La fecha de última actualización importa mucho** para lo que viene. Si el
> modelo se refrescó ayer y tu `.pbix` local tiene datos de hoy, **los números
> no van a coincidir — y eso es correcto**, no es un error. Guárdate ese dato.

**Anota el nombre y el ID del modelo.** Aparece como `TU_DATASET_ID`.

---

## Paso 3 · Explorar la estructura del modelo publicado

### Prompt sugerido

```
Del semantic model "Ventas Corporativas" en el workspace "Ventas - Producción",
muéstrame sus tablas y sus medidas. Compáralo brevemente con lo que exploramos
en el modelo local del bloque 2: ¿son la misma estructura o hay diferencias?
```

### Resultado esperado

Un listado de tablas y medidas del modelo publicado, y una comparación con lo que
Claude ya vio en el modelo local durante el bloque 2 (esa información sigue en el
contexto de la conversación).

Posibles hallazgos:

- **Idénticos** → el `.pbix` local es el mismo que está publicado. Lo esperable.
- **El Service tiene medidas que el local no** → alguien editó el modelo
  directamente en la nube.
- **El local tiene medidas que el Service no** → tienes cambios sin publicar.

> 💡 Detectar esto a mano requiere abrir los dos y comparar a ojo. Es uno de los
> usos más prácticos de todo el taller.

---

## Paso 4 · Ejecutar una consulta DAX contra el Service

Este es el paso central del bloque.

### Prompt sugerido

```
Ejecuta una consulta DAX contra el semantic model "Ventas Corporativas"
del workspace "Ventas - Producción", que devuelva el total de ventas
sin filtros. Usa la misma lógica que usamos en el modelo local.

Antes de ejecutarla, muéstrame la consulta. Después dime el resultado
con separador de miles, e indícame explícitamente que este número viene
del SERVICE, no de Power BI Desktop.
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

**Después**, el resultado, identificado claramente como proveniente del Service:

```
Total Ventas (Service - Ventas Corporativas): 12.435.890
```

> 📌 Fíjate en que **la consulta DAX es exactamente la misma** que corriste en el
> bloque 2. El lenguaje no cambia: cambia dónde se ejecuta. Es el mismo motor de
> Analysis Services, en tu PC o en la nube de Microsoft.

---

## Paso 5 · Una consulta agrupada, para tener algo comparable

### Prompt sugerido

```
Ahora ejecuta contra el Service la misma consulta agrupada por año que
hicimos en el modelo local, con el total de ventas por año ordenado de
mayor a menor. Guarda mentalmente el resultado: lo vamos a comparar con
el local en el bloque siguiente.
```

### Resultado esperado

| Año | Total Ventas (Service) |
|---|---|
| 2024 | 4.102.330 |
| 2023 | 5.220.110 |
| 2022 | 3.113.450 |

---

## Paso 6 · Consultar metadatos operacionales

Algo que el modelo local **no puede** decirte, porque solo existe en la nube.

### Prompt sugerido

```
Del semantic model "Ventas Corporativas", dime todo lo que puedas sobre su
operación: cuándo fue la última actualización, si tiene refresco programado
y con qué frecuencia, y si la última ejecución fue exitosa.
Si alguno de esos datos no está disponible con las herramientas que tienes,
dímelo en vez de suponerlo.
```

### Resultado esperado

Los datos que el servidor exponga (última actualización, programación de
refresco, estado) y una mención explícita de lo que **no** pudo obtener.

> ⭐ **Esa última instrucción del prompt es un buen hábito permanente.** "Si no lo
> sabes, dímelo" reduce mucho el riesgo de que un dato supuesto se te cuele como
> si fuera leído.

---

## Recordatorio: modo consulta

Todo lo que hicimos acá **lee**. No modificamos workspaces, no creamos ni
borramos modelos, no lanzamos refrescos.

Dos capas de protección:

1. **Nuestra disciplina:** nos quedamos en operaciones de consulta.
2. **Entra ID:** aunque quisieras, no puedes hacer nada que tu cuenta no pudiera
   hacer ya desde el navegador. Si tienes rol *Viewer*, sigues siendo *Viewer*.

Y una diferencia honesta con el bloque 2: allá el servidor **no podía** escribir
aunque lo intentaras. Acá **podría**, si expusiera esa herramienta y tus permisos
lo permitieran. Por eso, en este bloque, **leer el cuadro de permisos antes de
aprobar sí importa de verdad.**

---

## ✅ Checklist

- [ ] Listaste tus workspaces y verificaste que coinciden con app.powerbi.com.
- [ ] Listaste los semantic models de un workspace.
- [ ] **Anotaste la fecha de última actualización del modelo.**
- [ ] Ejecutaste al menos una consulta DAX contra el Service.
- [ ] Tienes a mano el resultado agrupado por año, para el bloque 4.

---

⬅️ **Anterior:** [Ejercicio 1 · Autenticación Entra ID](./01-autenticacion-entra-id.md)
➡️ **Siguiente:** [04 · Flujo completo](../../04-flujo-completo/README.md)

# Ejercicio 2 · Explorar y consultar contenido en Fabric

⏱️ **Tiempo estimado:** 18 minutos
📍 **Bloque:** [03 · Fabric IQ](../README.md)

---

## Objetivo

Encontrar un reporte o semantic model **por su nombre**, leer su estructura y
ejecutar una **consulta DAX contra Fabric** — no contra tu Power BI Desktop.

---

## Antes de empezar

- [ ] `/mcp` muestra `fabric-iq` como `connected` con 6 herramientas.
- [ ] Sabes el **nombre** de un reporte o semantic model que puedas abrir en
      <https://app.powerbi.com>.
- [ ] Estás en la carpeta del workshop.

> 💡 **Ya no necesitas Power BI Desktop abierto para este ejercicio.** Pero déjalo
> abierto igual: lo vamos a usar en el [bloque 4](../../04-flujo-completo/README.md).

---

## Paso 1 · Encontrar tu contenido por nombre

Acá está la gran diferencia con trabajar contra la API en crudo: **no necesitas
ningún ID**. Fabric IQ busca por nombre y resuelve los identificadores internos
por su cuenta.

### Prompt sugerido

```
Usando el MCP fabric-iq, busca en Microsoft Fabric el reporte o semantic
model llamado "Ventas Corporativas".

Dime qué encontraste: tipo de item (reporte o semantic model), en qué
workspace está, y cualquier otro dato que devuelva la búsqueda.
```

> 📌 Reemplaza `"Ventas Corporativas"` por el nombre real de tu contenido.

### Resultado esperado

Claude pide permiso para usar `DiscoverArtifacts`. Apruébalo. Devuelve el item
encontrado con su tipo y su ubicación.

> ⭐ **Esto es identidad delegada funcionando.** Fabric IQ solo devuelve contenido
> que **tu cuenta ya puede ver**. Si un compañero corre el mismo prompt, puede
> obtener un resultado distinto — o ninguno. Compáralo con quien tengas al lado:
> es la forma más rápida de entender el concepto.

### Si no lo encuentra

| Causa | Solución |
|---|---|
| El nombre es ambiguo o hay varios parecidos | Usa un nombre más específico y completo. |
| No tienes acceso | Verifica que puedes abrirlo en el navegador. |
| Es un tipo no soportado | Fabric IQ soporta **reportes y semantic models**. No dashboards, no reportes paginados (RDL), no apps. |

### Alternativa: pegar la URL

Si la búsqueda por nombre no acierta, pega la URL directamente.

```
Resuelve este item de Fabric y dime qué es:
https://app.powerbi.com/groups/xxxxxxxx/reports/yyyyyyyy
```

> ⚠️ **Usa la URL de la barra de direcciones de tu navegador, no un *share
> link*.** Los enlaces para compartir no funcionan con Fabric IQ. Claude usará
> `ResolveFabricItem` para obtener los identificadores internos.

---

## Paso 2 · Leer el esquema del modelo publicado

### Prompt sugerido

```
Obtén el esquema del semantic model que acabas de encontrar. Muéstrame:
- Las tablas, indicando cuáles parecen de hechos y cuáles de dimensiones
- Las medidas, con su expresión DAX
- Las relaciones, con su cardinalidad

Preséntalo en tablas y no inventes nada que no venga en el esquema.
```

### Resultado esperado

Claude usa `GetSemanticModelSchema` y devuelve la estructura del modelo
publicado: tablas, columnas, medidas y relaciones.

---

## Paso 3 · Comparar con lo que viste en el modelo local

### Prompt sugerido

```
Compara el esquema que acabas de leer de Fabric con el que exploramos
en el modelo local de Power BI Desktop en el bloque 2. ¿Son la misma
estructura o hay diferencias en tablas, medidas o relaciones?
Si hay diferencias, lístalas. Si son idénticos, dilo.
```

### Resultado esperado

Una comparación concreta. Posibles hallazgos:

- **Idénticos** → el `.pbix` local es el mismo que está publicado. Lo esperable.
- **Fabric tiene medidas que el local no** → alguien editó el modelo en la nube.
- **El local tiene medidas que Fabric no** → tienes cambios sin publicar.

> 💡 Detectar esto a mano requiere abrir ambos y comparar a ojo. Es uno de los
> usos más prácticos de todo el taller.

---

## Paso 4 · Ejecutar una consulta DAX contra Fabric

Este es el paso central del bloque.

### Prompt sugerido

```
Ejecuta una consulta DAX contra ese semantic model en Fabric, que devuelva
el total de ventas sin filtros. Usa la misma lógica que usamos en el modelo
local del bloque 2.

Antes de ejecutarla, muéstrame la consulta. Después dime el resultado con
separador de miles, e indícame explícitamente que este número viene de
FABRIC, no de Power BI Desktop.
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
Total Ventas (FABRIC - Ventas Corporativas): 12.435.890
```

> 📌 **La consulta DAX es exactamente la misma** que corriste en el bloque 2. El
> lenguaje no cambia: cambia dónde se ejecuta. Es el mismo motor de Analysis
> Services, en tu PC o en la nube de Microsoft.

> 🔐 **Tu RLS y tu OLS se aplican.** Si el modelo tiene Row-Level Security o
> Object-Level Security y tu usuario cae en un rol, el resultado viene filtrado
> — igual que si abrieras el reporte en el navegador. Es consecuencia directa de
> que Fabric IQ **solo** admita identidad delegada.

> ⚠️ **Una consulta, un modelo.** Cada `ExecuteQuery` apunta a un único semantic
> model: no hace joins entre modelos. Si necesitas combinar dos, Claude puede
> ejecutar dos consultas separadas y unir los resultados él mismo.

---

## Paso 5 · Una consulta agrupada, para tener algo comparable

### Prompt sugerido

```
Ahora ejecuta contra Fabric la misma consulta agrupada por año que hicimos
en el modelo local, con el total de ventas por año ordenado de mayor a
menor. Guarda el resultado: lo vamos a comparar con el local en el bloque
siguiente.
```

### Resultado esperado

| Año | Total Ventas (FABRIC) |
|---|---|
| 2024 | 3.980.115 |
| 2023 | 5.220.110 |
| 2022 | 3.113.450 |

> 💡 **Mantén los resultados chicos.** Las consultas grandes vuelven como un CSV
> embebido y pueden quedar truncadas. Usa agregaciones y filtros en vez de pedir
> tablas completas — que además es una buena práctica de DAX en general.

---

## Paso 6 · Buscar un valor dentro del modelo

`ValueSearch` es una herramienta que no tiene equivalente en el bloque 2, y
resuelve un problema muy concreto y muy cotidiano.

**El problema:** quieres filtrar por una región, un producto o un cliente, pero
no sabes cómo está escrito exactamente en el modelo. ¿Es `"Metropolitana"`,
`"Región Metropolitana"`, `"RM"` o `"XIII"`? Si aciertas mal, tu DAX devuelve
vacío y parece un error de datos.

### Prompt sugerido

```
Busca en el semantic model los valores almacenados que se parezcan a
"Metropolitana". Dime en qué tabla y columna están y cómo están escritos
exactamente.

Después, usa el valor exacto que encontraste para ejecutar una consulta DAX
que me dé el total de ventas solo de esa región.
```

> 📌 Reemplaza `"Metropolitana"` por algún valor que exista en tu modelo: una
> categoría de producto, un nombre de sucursal, un segmento de cliente.

### Resultado esperado

Primero, la ubicación y la grafía exacta del valor. Después, la consulta filtrada
y su resultado.

> ⭐ **Este es el paso que más tiempo ahorra en el día a día.** El clásico "mi
> medida devuelve blanco y no sé por qué" es, muchas veces, un filtro escrito con
> una tilde de más o una abreviatura distinta.

---

## Paso 7 · Leer la estructura de un reporte

Algo que el modelo local no te da: **cómo se usa realmente el modelo** en un
reporte publicado.

### Prompt sugerido

```
Lee los metadatos del reporte "Dashboard Comercial" en Fabric. Dime:
- Cuántas páginas tiene y cómo se llaman
- Qué visuales hay en cada página y qué campos usan
- Qué filtros están aplicados

Después dime qué medidas del semantic model NO aparecen usadas en ningún
visual de este reporte.
```

### Resultado esperado

Las páginas, los visuales con sus campos, los filtros. Y, al final, la lista de
medidas que ese reporte no usa.

> ⚠️ **Con un matiz honesto:** que una medida no aparezca en **este** reporte no
> significa que nadie la use. Puede estar en otro reporte, en un Excel conectado
> o en una app. Trata el resultado como **una lista de candidatas a revisar**, no
> como una orden de borrado.

---

## Recordatorio: solo lectura, en las tres capas

Todo lo que hiciste acá **lee**. Y la protección viene de tres lugares
independientes:

| Capa | Qué garantiza |
|---|---|
| **El servidor** | Fabric IQ expone seis herramientas de consumo. Ninguna crea, modifica ni administra. Microsoft separó eso en otros servidores MCP. |
| **Los permisos** | `Item.Read.All`, `Item.Execute.All`, `Dataset.Read.All`. Ningún permiso de escritura, porque el servidor no lo necesita. |
| **Tu identidad** | Solo ves lo que ya podías ver. RLS y OLS se aplican. No existe autenticación de service principal que pudiera saltárselos. |

---

## ✅ Checklist

- [ ] Encontraste contenido en Fabric **por su nombre**, sin usar ningún ID.
- [ ] Leíste el esquema del semantic model publicado.
- [ ] Lo comparaste con el modelo local del bloque 2.
- [ ] Ejecutaste al menos una consulta DAX contra Fabric.
- [ ] Usaste `ValueSearch` para encontrar la grafía exacta de un valor.
- [ ] Tienes a mano el resultado agrupado por año, para el bloque 4.

---

⬅️ **Anterior:** [Ejercicio 1 · Autenticación Entra ID](./01-autenticacion-entra-id.md)
➡️ **Siguiente:** [04 · Flujo completo](../../04-flujo-completo/README.md)

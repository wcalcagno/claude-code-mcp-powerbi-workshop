# Ejercicio 3 · Documentar las medidas del modelo

⏱️ **Tiempo estimado:** 12 minutos
📍 **Bloque:** [02 · MCP Power BI local](../README.md)

---

## Objetivo

Generar, en un par de minutos, **un documento Markdown con todas las medidas de
tu modelo**, su expresión DAX y una explicación en lenguaje de negocio — la
documentación que siempre se promete y nunca se escribe.

---

## Antes de empezar

- [ ] Power BI Desktop abierto con tu `.pbix`.
- [ ] `/mcp` muestra `powerbi-local` como `connected`.
- [ ] Estás en la carpeta del workshop.

---

## ⭐ Lee esto antes: quién hace qué en este ejercicio

Este ejercicio combina **dos capacidades distintas**, y entender la diferencia es
el aprendizaje central del bloque:

| Paso | Quién lo hace | Qué toca |
|---|---|---|
| 1. Leer las medidas del modelo | **El MCP `powerbi-local`** | Lee el modelo de Power BI. **Nada más.** |
| 2. Escribir el archivo `.md` en disco | **Claude Code** (herramienta de archivos) | Escribe en tu carpeta del repositorio. |
| 3. Commitear el archivo | **Claude Code** (Git) | Tu repositorio local. |

> 🔴 **El MCP nunca escribe.** No escribe en el modelo, y tampoco escribe el
> archivo `.md`. Su único rol acá es **entregar los datos**. La escritura del
> archivo la hace Claude Code directamente sobre el sistema de archivos, que es
> una capacidad completamente separada y que **sí te pide permiso** antes de actuar.

Esta separación es lo que hace que el flujo sea seguro: la herramienta que toca
tu modelo no puede escribir nada, y la herramienta que escribe no toca tu modelo.

---

## Paso 1 · Leer todas las medidas

### Prompt sugerido

```
Usando el MCP powerbi-local, lista TODAS las medidas del modelo que tengo
abierto en Power BI Desktop. Para cada una muéstrame:
- Nombre de la medida
- Tabla a la que pertenece
- Expresión DAX completa
- Formato de visualización, si está disponible
- Descripción, si la tiene

Muéstramelo primero en pantalla, todavía no escribas ningún archivo.
```

### Resultado esperado

Un listado con las medidas de tu modelo. Si tienes muchas, Claude puede agrupar
por tabla. Ejemplo de cómo debería verse una entrada:

```
Ventas Netas  (tabla: Medidas)
Formato: #,##0
DAX:
    Ventas Netas =
    CALCULATE(
        SUM( Ventas[MontoTotal] ),
        Ventas[Estado] = "Facturada"
    )
```

> ⚠️ **Si el listado sale vacío**, revisa que el modelo efectivamente tenga
> medidas (algunos modelos simples solo tienen columnas). Puedes crear una medida
> rápida en Power BI Desktop y volver a intentar.

---

## Paso 2 · Generar el archivo de documentación

Ahora sí, que escriba el archivo.

### Prompt sugerido

```
Con esas medidas, crea el archivo 02-mcp-powerbi-local/documentacion-medidas.md
con esta estructura:

1. Un título de nivel 1 con el nombre del modelo
2. Una línea indicando la fecha de generación y que fue generada con
   Claude Code leyendo el modelo vía MCP de solo lectura
3. Un índice con enlaces a cada medida
4. Una sección por cada medida con:
   - Título de nivel 2 con el nombre de la medida
   - Una tabla con: tabla contenedora, formato y dependencias
     (qué columnas y otras medidas usa)
   - La expresión DAX en un bloque de código con etiqueta dax
   - Un párrafo "Qué calcula" explicándola en lenguaje de negocio,
     sin jerga técnica, como para alguien de Gerencia Comercial
5. Al final, una sección "Observaciones" con cualquier cosa que te llame
   la atención: medidas duplicadas, medidas que no se usan en ninguna otra,
   o expresiones que se puedan simplificar

Todo en español. No inventes descripciones de negocio que no puedas
deducir de la expresión DAX: si no estás seguro, dilo explícitamente.
```

Claude te mostrará el contenido y **pedirá permiso para crear el archivo**.
Léelo antes de aprobar.

### Resultado esperado

Un mensaje confirmando la creación de
`02-mcp-powerbi-local/documentacion-medidas.md`, y un archivo con una estructura
como esta:

```markdown
# Documentación de medidas — Modelo Ventas

Generado el 18-09-2026 con Claude Code, leyendo el modelo abierto en
Power BI Desktop mediante un servidor MCP de solo lectura.

## Índice
- [Ventas Netas](#ventas-netas)
- [Margen %](#margen-)
...

## Ventas Netas

| Atributo | Valor |
|---|---|
| Tabla | Medidas |
| Formato | #,##0 |
| Depende de | Ventas[MontoTotal], Ventas[Estado] |

```dax
Ventas Netas =
CALCULATE(
    SUM( Ventas[MontoTotal] ),
    Ventas[Estado] = "Facturada"
)
```

**Qué calcula:** el monto total de las ventas, considerando únicamente las
transacciones que ya fueron facturadas. Excluye cotizaciones y pedidos
pendientes.
```

---

## Paso 3 · Revisar el archivo con ojo crítico

**Esta revisión es tuya y es obligatoria.** Claude leyó el DAX correctamente,
pero la interpretación de negocio puede estar equivocada: no sabe que en tu
empresa "Facturada" incluye las notas de crédito, por ejemplo.

### Prompt sugerido

```
Muéstrame el contenido de @02-mcp-powerbi-local/documentacion-medidas.md
```

Léelo con calma y busca:

- [ ] ¿Las expresiones DAX están completas y correctas?
- [ ] ¿Las explicaciones de negocio son ciertas para tu empresa?
- [ ] ¿Hay alguna medida que falte?
- [ ] ¿Las "Observaciones" tienen sentido?

### Si algo está mal, corrígelo conversando

```
En la medida Ventas Netas, la explicación de negocio está incompleta:
en nuestra empresa el estado "Facturada" también incluye las notas de
crédito, así que el monto ya viene neto de devoluciones. Corrige ese
párrafo en el archivo y deja el resto igual.
```

**Resultado esperado:** Claude edita solo ese párrafo, te muestra el cambio y
pide permiso para aplicarlo.

---

## Paso 4 · Guardar en el historial

### Prompt sugerido

```
Agrega documentacion-medidas.md a Git y haz un commit con un mensaje
en español que explique qué documento es y de qué modelo salió.
No hagas push.
```

### Resultado esperado

```
[main a91c4f2] Agrega documentación de medidas del modelo Ventas generada vía MCP
 1 file changed, 87 insertions(+)
 create mode 100644 02-mcp-powerbi-local/documentacion-medidas.md
```

---

## Para llevarte al trabajo

Este ejercicio es probablemente **lo más directamente aplicable de todo el
workshop**. Piensa en:

- Documentar un modelo heredado antes de tocarlo.
- Generar el anexo técnico de una entrega a un cliente.
- Hacer un inventario de medidas antes de una migración.
- Encontrar medidas duplicadas o muertas en un modelo que creció sin control.

Y todo sin riesgo de romper nada: **la herramienta que leyó el modelo no tiene
forma de modificarlo.**

---

## ✅ Checklist

- [ ] Listaste las medidas vía MCP.
- [ ] Generaste `documentacion-medidas.md`.
- [ ] **Revisaste el archivo y corregiste al menos una explicación de negocio.**
- [ ] Commiteaste el archivo.
- [ ] Tienes claro que el MCP leyó, pero que fue Claude Code quien escribió el
      archivo — dos capacidades separadas.

---

⬅️ **Anterior:** [Ejercicio 2 · Consultar DAX](./02-consultar-dax.md)
➡️ **Siguiente:** [03 · MCP Fabric / API](../../03-mcp-fabric-api/README.md)

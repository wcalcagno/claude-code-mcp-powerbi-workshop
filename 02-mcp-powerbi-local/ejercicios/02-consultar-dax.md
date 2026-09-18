# Ejercicio 2 · Consultar DAX

⏱️ **Tiempo estimado:** 10 minutos
📍 **Bloque:** [02 · MCP Power BI local](../README.md)

---

## Objetivo

Pedirle a Claude Code que **escriba y ejecute una consulta DAX** contra tu modelo
abierto en Power BI Desktop, y que te **explique el resultado** — incluyendo qué
hace cada parte de la consulta.

---

## Antes de empezar

- [ ] Power BI Desktop abierto con el mismo `.pbix` del ejercicio anterior.
- [ ] `/mcp` muestra `powerbi-local` como `connected`.
- [ ] Ya sabes los nombres de las tablas de tu modelo (ejercicio 1).

---

## Contexto: ¿qué es una "consulta DAX"?

En Power BI escribes DAX para crear **medidas** y **columnas calculadas**. Pero
DAX también sirve para escribir **consultas completas**, con una sintaxis muy
parecida a SQL:

```dax
EVALUATE
    tabla_o_expresión
```

`EVALUATE` significa "devuélveme esta tabla". Es la forma en que herramientas
externas — DAX Studio, Excel, y nuestro servidor MCP — le piden datos al modelo.

Si nunca escribiste una consulta DAX, no importa: **Claude la escribe por ti.**
Tu trabajo es entender lo que devuelve.

---

## Paso 1 · Una consulta muy simple para calentar

### Prompt sugerido

```
Usando el MCP powerbi-local, ejecuta una consulta DAX que me devuelva
las primeras 10 filas de la tabla Ventas. Muéstrame el resultado en
una tabla y dime cuántas columnas tiene.
```

> 📌 Reemplaza `Ventas` por el nombre de una tabla real de tu modelo.

### Resultado esperado

Claude muestra la consulta que va a ejecutar (algo como
`EVALUATE TOPN(10, Ventas)`), pide permiso para usar la herramienta del MCP, y
devuelve las 10 filas en una tabla.

> 💡 **Si tu tabla es muy ancha**, Claude puede resumir o mostrar solo algunas
> columnas. Pídele las que te interesen.

---

## Paso 2 · El total de ventas

Este es el ejercicio central: un número que puedes **verificar tú mismo**.

### Prompt sugerido

```
Ejecuta una consulta DAX contra el modelo que me devuelva el total de ventas
de todo el modelo, sin filtros. Usa la columna de monto que corresponda según
lo que ya exploraste del modelo.

Antes de ejecutarla, muéstrame la consulta y explícame en español qué hace
cada línea. Después ejecútala y dime el resultado con separador de miles.
```

### Resultado esperado

**Primero**, la consulta y su explicación. Algo así:

```dax
EVALUATE
    ROW(
        "Total Ventas",
        SUM( Ventas[MontoTotal] )
    )
```

Con una explicación del estilo:

| Línea | Qué hace |
|---|---|
| `EVALUATE` | Indica que lo que sigue es la tabla que debe devolverse. |
| `ROW(...)` | Construye una tabla de una sola fila. Sirve para devolver un valor escalar como si fuera tabla, porque `EVALUATE` siempre exige una tabla. |
| `"Total Ventas"` | El nombre que tendrá la columna del resultado. |
| `SUM( Ventas[MontoTotal] )` | Suma la columna de monto en todas las filas de la tabla de hechos. |

**Después**, el resultado:

```
Total Ventas: 12.435.890
```

El número será el de **tu** modelo.

---

## Paso 3 · Verifica el número (paso obligatorio)

Nunca confíes en un número que no verificaste al menos una vez.

1. Anda a **Power BI Desktop**.
2. Inserta una **tarjeta** (*Card*) en el lienzo.
3. Arrastra tu columna de monto al campo de la tarjeta.
4. Asegúrate de que el resumen sea **Suma**.

**Resultado esperado:** la tarjeta muestra **el mismo número** que devolvió la
consulta DAX.

> ⚠️ **Si no coinciden**, las causas típicas son:
> - El visual tiene un filtro aplicado (a nivel de visual, página o reporte).
> - Estás sumando una columna distinta.
> - Hay filtros a nivel de modelo (RLS) activos.
>
> Pregúntaselo a Claude: *"El total que devolviste es X pero mi tarjeta en Power
> BI Desktop muestra Y. ¿Qué podría explicar la diferencia?"*

---

## Paso 4 · Una consulta con agrupación

Ahora algo más parecido a lo que harías en el día a día.

### Prompt sugerido

```
Ejecuta una consulta DAX que me devuelva el total de ventas agrupado
por año, ordenado de mayor a menor año. Muéstrame la consulta antes de
ejecutarla y después interpreta el resultado: ¿hay alguna tendencia,
algún año incompleto o algún valor que se vea raro?
```

### Resultado esperado

Una consulta parecida a:

```dax
EVALUATE
    SUMMARIZECOLUMNS(
        Fecha[Año],
        "Total Ventas", SUM( Ventas[MontoTotal] )
    )
ORDER BY Fecha[Año] DESC
```

Un resultado tabular:

| Año | Total Ventas |
|---|---|
| 2024 | 4.102.330 |
| 2023 | 5.220.110 |
| 2022 | 3.113.450 |

Y una interpretación: por ejemplo, que 2024 parece un año parcial, o que hay una
caída, o que el primer año tiene menos datos porque el modelo arranca a mitad de
año.

> 💡 **Esa interpretación es la parte interesante.** El número lo saca el motor;
> el contexto lo aporta Claude leyendo la estructura del modelo. Aun así,
> **el criterio de negocio es tuyo** — Claude no sabe que 2022 fue el año en que
> se abrió la sucursal nueva.

---

## Paso 5 · Consulta sobre una medida existente

Si tu modelo tiene medidas, úsalas — es más realista que sumar columnas a mano.

### Prompt sugerido

```
Lista las medidas del modelo. Elige una que calcule un total o un monto,
ejecútala en una consulta DAX sin filtros, y dime:
1. El nombre de la medida
2. Su expresión DAX
3. El valor que devuelve
4. Qué hace esa medida explicada en lenguaje de negocio, no técnico
```

### Resultado esperado

Los cuatro puntos, con la expresión DAX real de tu modelo y una explicación en
lenguaje llano. Por ejemplo: *"`Ventas Netas` toma el monto total y le resta las
devoluciones, considerando solo las transacciones con estado 'Facturada'."*

---

## Recordatorio: esto sigue siendo solo lectura

Ejecutar una consulta DAX **no modifica el modelo**. Es equivalente a hacer un
`SELECT` en una base de datos: el motor calcula y devuelve, pero nada cambia en
el `.pbix`.

Si le pides a Claude que "cree esta medida en el modelo", te va a decir que no
puede hacerlo a través del MCP — y te va a ofrecer escribir la expresión en un
archivo para que la copies tú. Ese es el diseño que elegimos.

---

## ✅ Checklist

- [ ] Ejecutaste al menos una consulta DAX vía MCP.
- [ ] Entendiste qué hace cada línea de la consulta.
- [ ] **Verificaste el total contra un visual en Power BI Desktop.**
- [ ] Hiciste una consulta con agrupación y leíste su interpretación.

---

⬅️ **Anterior:** [Ejercicio 1 · Explorar el modelo](./01-explorar-modelo.md)
➡️ **Siguiente:** [Ejercicio 3 · Documentar medidas](./03-documentar-medidas.md)

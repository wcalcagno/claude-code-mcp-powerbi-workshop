# 04 · Flujo completo — Ejercicio integrador

⏱️ **Duración del bloque:** 20 minutos (15 de ejercicio + 5 de cierre)
📍 **Requisito:** haber completado los bloques [02](../02-mcp-powerbi-local/README.md) y [03](../03-mcp-fabric-api/README.md)

---

## El escenario

Es lunes. Alguien de Gerencia escribe:

> *"El total de ventas del dashboard no me cuadra con lo que me mostraron en la
> reunión del viernes. ¿Alguien puede revisar?"*

Ese es, con distintos disfraces, el ticket más frecuente en la vida de un analista
de BI. Y la respuesta casi siempre está en una de estas causas:

| Causa | Cómo se ve |
|---|---|
| El modelo publicado no se ha refrescado | El Service está atrasado respecto al origen |
| Hay cambios locales sin publicar | El `.pbix` tiene medidas nuevas que el Service no |
| Alguien editó el modelo en la nube | El Service tiene algo que el local no |
| Filtros distintos | Los números son correctos, la comparación no |
| Es un problema real del dato | El menos frecuente, pero el que más importa |

En este ejercicio vas a **responder ese ticket** usando todo lo del taller, y a
dejar la respuesta **documentada y versionada en GitHub**.

---

## Objetivo

Comparar un mismo indicador entre **el modelo local de Power BI Desktop** y **el
semantic model publicado en el Service / Fabric**, explicar la diferencia (o
confirmar que no hay), y dejarlo todo commiteado.

---

## Antes de empezar

- [ ] **Power BI Desktop abierto** con el `.pbix` del bloque 2.
- [ ] `/mcp` muestra **ambos** servidores como `connected`:
      `powerbi-local` y `powerbi-fabric`.
- [ ] Estás en la carpeta del workshop, con Claude Code abierto.

### Verifica que tienes las dos conexiones vivas

```
/mcp
```

**Resultado esperado:**

```
powerbi-local     ✔ connected    (N tools)
powerbi-fabric    ✔ connected    (M tools)
```

> 🔴 Si `powerbi-local` está caído, casi seguro cerraste Power BI Desktop.
> Ábrelo con un `.pbix` y reinicia Claude Code.
>
> 🔴 Si `powerbi-fabric` está caído, el token expiró. Escribe `/mcp` y vuelve a
> autenticarte — toma menos de un minuto.

> 💡 **Ideal:** que el `.pbix` local y el semantic model del Service sean **el
> mismo modelo** (uno publicado desde el otro). Si no es tu caso, el ejercicio
> igual funciona: compara indicadores equivalentes y la conversación sobre las
> diferencias va a ser incluso más rica.

---

## Paso 1 · Definir qué vas a comparar

No compares "todo". Elige **un indicador concreto** y defiéndelo.

### Prompt sugerido

```
Vamos a hacer un ejercicio de comparación entre mi modelo local de Power BI
Desktop (MCP powerbi-local) y el semantic model publicado en el Service
(MCP powerbi-fabric).

Primero, ayúdame a elegir qué comparar. Propón un indicador que:
- Exista en ambos modelos
- Sea fácil de verificar a ojo
- Se pueda desagregar por año o por alguna dimensión

Explícame por qué lo elegiste antes de ejecutar nada.
```

### Resultado esperado

Claude propone un indicador (típicamente el total de ventas, con desagregación
por año) y justifica la elección. Tú confirmas o propones otro.

---

## Paso 2 · Obtener el dato del modelo LOCAL

### Prompt sugerido

```
Usando el MCP powerbi-local, ejecuta la consulta DAX que devuelve el total
de ventas por año en el modelo que tengo abierto en Power BI Desktop.
Etiqueta claramente el resultado como "LOCAL".
Muéstrame también la consulta que usaste.
```

### Resultado esperado

| Año | Total Ventas (LOCAL) |
|---|---|
| 2024 | 4.102.330 |
| 2023 | 5.220.110 |
| 2022 | 3.113.450 |

---

## Paso 3 · Obtener el mismo dato del SERVICE

### Prompt sugerido

```
Ahora, usando el MCP powerbi-fabric, ejecuta la MISMA consulta DAX contra
el semantic model publicado en el Service.
Etiqueta el resultado como "SERVICE".
Dime también cuándo fue la última actualización de ese modelo.
```

### Resultado esperado

| Año | Total Ventas (SERVICE) |
|---|---|
| 2024 | 3.980.115 |
| 2023 | 5.220.110 |
| 2022 | 3.113.450 |

Más la fecha del último refresco.

---

## Paso 4 · Comparar y diagnosticar

Acá es donde el ejercicio se pone interesante.

### Prompt sugerido

```
Compara los dos resultados en una sola tabla, con columnas:
año, valor LOCAL, valor SERVICE, diferencia absoluta y diferencia porcentual.

Después, dame un diagnóstico:
- ¿Hay diferencias? ¿En qué años?
- ¿Cuál es la explicación más probable, considerando la fecha del último
  refresco del Service y lo que ya sabes de ambos modelos?
- ¿Qué tendría que revisar yo para confirmar esa hipótesis?

Sé explícito sobre qué es un hecho que leíste de los modelos y qué es una
hipótesis tuya. No inventes causas que no puedas sostener con los datos.
```

### Resultado esperado

Una tabla comparativa:

| Año | LOCAL | SERVICE | Diferencia | % |
|---|---|---|---|---|
| 2024 | 4.102.330 | 3.980.115 | **122.215** | **3,1%** |
| 2023 | 5.220.110 | 5.220.110 | 0 | 0,0% |
| 2022 | 3.113.450 | 3.113.450 | 0 | 0,0% |

Y un diagnóstico del estilo:

> **Hechos:** 2022 y 2023 coinciden exactamente. La diferencia está solo en 2024,
> el año en curso. El semantic model del Service se refrescó por última vez el
> 17-09-2026 a las 06:00.
>
> **Hipótesis:** el modelo local tiene datos más recientes que el publicado.
> La diferencia corresponde a las transacciones cargadas después del último
> refresco. La estructura del modelo es idéntica, así que no se trata de un
> cambio de lógica.
>
> **Para confirmar:** filtrar el modelo local a las transacciones posteriores al
> 17-09-2026 06:00 y verificar que suman aproximadamente 122.215.

> ⭐ **Esa separación entre hechos e hipótesis es lo que convierte esto en un
> análisis y no en una adivinanza.** Y la validación final sigue siendo tuya.

### Si tus dos modelos son idénticos

Perfecto — también es un resultado válido y vale la pena documentarlo.

```
Los dos resultados coinciden exactamente. Documenta eso como una
CONFIRMACIÓN: qué se comparó, con qué método, en qué fecha, y qué
significa que coincidan.
```

---

## Paso 5 · Documentar el hallazgo

### Prompt sugerido

```
Crea el archivo 04-flujo-completo/comparacion-local-vs-service.md con este
contenido:

1. Título: "Comparación modelo local vs. Power BI Service"
2. Ficha de la comparación: fecha y hora, nombre del modelo local, nombre del
   workspace y del semantic model, fecha del último refresco del Service,
   e indicador comparado
3. Sección "Método": qué consulta DAX se ejecutó en cada lado, en bloques
   de código con etiqueta dax, y aclarando que ambas lecturas se hicieron
   mediante servidores MCP de consulta, sin modificar ningún modelo
4. Sección "Resultados": la tabla comparativa completa
5. Sección "Diagnóstico": separando claramente HECHOS de HIPÓTESIS
6. Sección "Próximos pasos": qué habría que hacer para cerrar el caso
7. Al final, una nota de una línea indicando que el documento fue generado
   con Claude Code

Todo en español, tono profesional, como para enviárselo a Gerencia.
No uses placeholders: usa los datos reales de esta sesión.
```

Claude te mostrará el contenido y pedirá permiso para crear el archivo. **Léelo
antes de aprobar.**

### Resultado esperado

Un archivo `04-flujo-completo/comparacion-local-vs-service.md` con las seis
secciones, listo para compartir.

---

## Paso 6 · Revisar el documento

**Tuyo, no de Claude.** Revisa:

- [ ] ¿Los números coinciden con lo que viste en pantalla?
- [ ] ¿La hipótesis tiene sentido para tu organización?
- [ ] ¿Hay algo marcado como "hecho" que en realidad es una suposición?
- [ ] ¿Hay algún dato sensible que no debería quedar en GitHub?

### Prompt sugerido (si algo hay que ajustar)

```
Muéstrame el contenido de @04-flujo-completo/comparacion-local-vs-service.md
```

Y luego, en lenguaje natural, pide los cambios que necesites.

> 🔐 **Revisa especialmente lo último.** Si tus montos son información sensible,
> pídele a Claude que los reemplace por valores relativos o índices antes de
> commitear. Recuerda además que el `.gitignore` de este repo ignora `.pbix`,
> `.csv` y `.xlsx` justamente por esto.

---

## Paso 7 · Commitear a Git

### Prompt sugerido

```
Agrega el archivo de comparación a Git y haz un commit con un mensaje
en español que describa qué comparación se hizo y cuál fue la conclusión
principal. Después muéstrame git status y los últimos 3 commits.
No hagas push todavía.
```

### Resultado esperado

```
[main 7d3e91a] Documenta comparación de ventas entre modelo local y Service
 1 file changed, 64 insertions(+)
 create mode 100644 04-flujo-completo/comparacion-local-vs-service.md
```

Y luego el estado limpio más el historial del día:

```
7d3e91a Documenta comparación de ventas entre modelo local y Service
a91c4f2 Agrega documentación de medidas del modelo Ventas generada vía MCP
3f8a21c Agrega archivo de notas personales del workshop
```

**Ese historial es el resumen de tu workshop.**

---

## Paso 8 · Subir a GitHub

Este es el único paso que hacemos **a mano**, porque es el que manda tu trabajo
fuera de tu computador.

Sal de Claude Code:

```
/exit
```

Y desde PowerShell:

```powershell
git push origin main
```

### Resultado esperado

```
Enumerating objects: 12, done.
Counting objects: 100% (12/12), done.
Writing objects: 100% (8/8), 3.21 KiB | 1.07 MiB/s, done.
To https://github.com/TU_USUARIO/claude-code-mcp-powerbi-workshop.git
   1a2b3c4..7d3e91a  main -> main
```

Si es la primera vez, Windows te pedirá tu usuario y tu **token** (bloque 00).

> 📌 **Si no tienes permiso de escritura** sobre el repositorio original (porque
> es de quien dicta el taller), el push va a fallar con `403`. Es lo esperado.
> Opciones:
> - Haz un **fork** del repo a tu cuenta desde GitHub y cambia el remoto:
>   ```powershell
>   git remote set-url origin https://github.com/TU_USUARIO/claude-code-mcp-powerbi-workshop.git
>   ```
> - O crea un repositorio nuevo y vacío en tu cuenta y apunta el remoto ahí.
>
> Pídele ayuda a Claude Code con el error exacto: es un buen último ejercicio.

### Verifica en el navegador

Abre `https://github.com/TU_USUARIO/claude-code-mcp-powerbi-workshop` y confirma
que están tus archivos nuevos.

---

## Lo que recorriste en 2 horas

```
 Windows sin Git
      │
      ▼
 Git instalado + GitHub configurado            ← bloque 00
      │
      ▼
 Claude Code leyendo y commiteando archivos    ← bloque 01
      │
      ▼
 Claude Code conectado a Power BI Desktop      ← bloque 02
 (MCP de SOLO LECTURA)                            tablas, DAX, documentación
      │
      ▼
 Claude Code conectado al Service / Fabric     ← bloque 03
 (API REST + Entra ID, modo consulta)             workspaces, datasets, DAX
      │
      ▼
 Un análisis real, documentado y versionado    ← bloque 04
```

---

## Las tres ideas que vale la pena llevarse

### 1. El "solo lectura" fue una decisión, no una carencia

El MCP local **no puede** tocar tu modelo. Eso permitió que todos experimentaran
sin miedo durante dos horas. Cuando evalúes herramientas para tu trabajo diario,
hazte la misma pregunta: **¿cuál es el peor caso si alguien aprueba sin leer?**

### 2. La división de responsabilidades es la que hace esto seguro

| Quién | Qué hace | Qué NO hace |
|---|---|---|
| **MCP local** | Lee el modelo | Escribir en el modelo. Escribir archivos. |
| **MCP remoto** | Consulta el Service | Nada fuera de tus permisos de Entra ID. |
| **Claude Code** | Escribe archivos y usa Git | Tocar tus modelos de Power BI. |
| **Tú** | Aprobar, validar y decidir | Delegar el criterio. |

### 3. La validación sigue siendo tuya

Claude leyó los modelos correctamente y armó la tabla. Pero **si 2024 está bajo
porque el refresco falló, o porque se cerró una sucursal, eso lo sabes tú.**
La herramienta acelera el camino hasta la pregunta correcta. La respuesta la
sigues dando tú.

---

## Para seguir después del taller

| Idea | Dificultad |
|---|---|
| Documentar un modelo heredado antes de tocarlo | Baja |
| Generar el anexo técnico de una entrega a cliente | Baja |
| Comparar el mismo modelo entre DEV, QA y PROD | Media |
| Detectar medidas duplicadas o sin uso en un modelo grande | Media |
| Armar un checklist de revisión de modelos y pedirle a Claude que lo aplique | Media |
| Escribir tu propio `CLAUDE.md` con las convenciones DAX de tu equipo | Media |

> 💡 **El mejor primer paso mañana:** toma el `.pbix` más desordenado que tengas
> y pídele a Claude Code que te lo documente. Diez minutos, cero riesgo, y vas a
> encontrar cosas que no sabías que estaban ahí.

---

## ✅ Checklist final del workshop

- [ ] Git instalado y configurado con tu nombre y correo.
- [ ] Repositorio clonado y autenticación con GitHub funcionando.
- [ ] Claude Code instalado y con sesión iniciada.
- [ ] MCP local conectado y usado sobre Power BI Desktop.
- [ ] MCP remoto conectado con tu identidad de Entra ID.
- [ ] Al menos tres commits en tu historial.
- [ ] Documento de comparación creado, revisado y subido a GitHub.
- [ ] Te queda claro **por qué** el MCP local es de solo lectura.

---

⬅️ **Anterior:** [03 · MCP Fabric / API](../03-mcp-fabric-api/README.md)
📚 **Para seguir leyendo:** [recursos/enlaces.md](../recursos/enlaces.md)

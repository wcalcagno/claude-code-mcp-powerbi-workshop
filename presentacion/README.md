# Presentación del taller

El deck que se proyecta durante el workshop: **20 diapositivas**, con notas del
orador en cada una.

> 📌 Esta carpeta es material **para quien dicta el taller**. Los asistentes no
> la necesitan: su recorrido son las carpetas `00-setup/` a `04-flujo-completo/`.

---

## Ver y presentar

El deck está publicado como **artifact de Claude**, privado. Desde ahí se
presenta a pantalla completa, se pasan las diapositivas, se leen las notas del
orador y se descarga en PDF o PowerPoint.

**Cómo llegar a él:**

| Dónde estás | Qué hacer |
|---|---|
| Claude Code, en la terminal | Escribe `/artifacts` y elige el deck. `o` lo abre, `c` copia su enlace. |
| Claude Code, app de escritorio | `Ctrl+]` reabre el artifact más reciente de la sesión. |
| Navegador | Entra a <https://claude.ai/code/artifacts> y búscalo por su nombre. |

Busca el artifact llamado **"Claude Code + MCP para Power BI y Fabric"**.

> 🔒 Es **privado**: solo lo abre su dueño y quien reciba acceso explícito desde
> el menú *Share* de la página. Si vas a dictar el taller con otra persona,
> compártelo antes.
>
> 📌 **El enlace no está escrito en este repositorio a propósito**, para que
> pueda publicarse sin arrastrar un vínculo a contenido privado. Si trabajas en
> equipo, compártelo por el canal interno que ya usen.

---

## Las 20 diapositivas

| # | Archivo | Qué es |
|---|---|---|
| 01 | `portada.html` | Portada: título, duración y la agenda de los cinco bloques |
| 02 | `resultado.html` | Lo que te llevas — los tres resultados del taller |
| 03 | `claude-code.html` | Claude Code y Claude.ai — tabla comparativa |
| 04 | `mcp.html` | MCP en una frase — la analogía con Power Query |
| 05 | `solo-lectura.html` | ⭐ **Statement:** ningún servidor puede escribir en tus modelos |
| 06 | `div-00.html` | Separador · Bloque 00 · Setup |
| 07 | `setup.html` | Tres instalaciones y sus comandos de verificación |
| 08 | `div-01.html` | Separador · Bloque 01 · Claude Code básico |
| 09 | `comandos.html` | Los comandos esenciales |
| 10 | `div-02.html` | Separador · Bloque 02 · MCP local |
| 11 | `arquitectura-local.html` | La ruta hasta tu modelo — Power BI Desktop → MCP → Claude Code |
| 12 | `ejercicios-local.html` | Los tres ejercicios sobre tu modelo |
| 13 | `div-03.html` | Separador · Bloque 03 · Fabric IQ |
| 14 | `fabric-iq.html` | Las seis herramientas, todas de lectura |
| 15 | `autenticacion.html` | Tu identidad, tus permisos — Entra ID delegado |
| 16 | `div-04.html` | Separador · Bloque 04 · Flujo completo |
| 17 | `caso.html` | El ticket del lunes |
| 18 | `comparacion.html` | Local contra la nube — resultados y diagnóstico |
| 19 | `tres-ideas.html` | Tres ideas para llevarse |
| 20 | `cierre.html` | Cierre y primer paso después del taller |

---

## ⚠️ Placeholders que hay que completar antes de dictar

| Diapositiva | Placeholder | Reemplazar por |
|---|---|---|
| 01 · Portada | `[Relator]` · `[Fecha]` | Tu nombre y la fecha del taller |
| 20 · Cierre | `[Relator]` · `[correo]` | Tu nombre y tu correo de contacto |
| 20 · Cierre | `github.com/[TU_USUARIO]/...` | La URL real del repositorio |

La diapositiva 18 usa **cifras de ejemplo**, marcadas como tales en el propio
encabezado. Si prefieres mostrar números de un modelo real, reemplázalos — pero
revisa antes que no sean datos sensibles.

---

## El sistema visual

Tres tipos de diapositiva, cada uno con una función:

| Tipo | Fondo | Cuándo se usa |
|---|---|---|
| **Contenido** | Crema `#F4F1EA` | Tablas, tarjetas y diagramas. El título va siempre a la misma altura, para que no salte al pasar de una a otra. |
| **Separador** | Oscuro `#141B22` | Abre cada bloque, con su número, su duración y una línea de contexto. |
| **Statement** | Terracota `#D97757` | **Una sola en todo el deck:** la del solo lectura. |

Tipografías: **Rubik** para los textos y **JetBrains Mono** para todo lo que
representa la terminal — comandos, nombres de herramientas, números de bloque.

### La lógica narrativa

El deck está construido alrededor de **un argumento**, no de una lista de temas:

1. La diapositiva 05 establece que ningún servidor del taller puede escribir.
2. El bloque 02 lo demuestra en local: el servidor literalmente no tiene esas
   funciones.
3. El bloque 03 lo confirma en la nube: Microsoft separó consumo y administración
   en servidores distintos, y usamos el de consumo.
4. La diapositiva 19 lo convierte en un criterio que el asistente se lleva:
   *al evaluar una herramienta, pregunta qué es incapaz de hacer.*

Si adaptas el deck, esa es la columna vertebral que conviene no romper.

---

## Las notas del orador

Cada diapositiva trae notas escritas como **guion hablado**, no como viñetas.
Incluyen:

- Qué decir y en qué detenerse.
- Preguntas para hacerle a la sala.
- Dónde se atasca la gente y cómo destrabarla sin frenar al resto.
- Qué hacer si el bloque 03 no se puede ejecutar en vivo (hacerlo como demo).

Las lees en la vista de presentación del artifact.

---

## Editar el deck

**La forma simple:** ábrelo como indica *Ver y presentar* y edítalo en la propia
página. Los cambios quedan guardados en el artifact, no en estos archivos.

**Desde Claude Code:** pídeselo en lenguaje natural, por ejemplo

```
Lee presentacion/slides/cierre.html y reemplaza los placeholders
[Relator] y [correo] por "Ana Pérez" y "ana.perez@empresa.cl".
Después publica el cambio al deck.
```

---

## Cómo se organizan estos archivos

| Acá | En el artifact |
|---|---|
| `deck.json` | `project/deck.json` — índice: título, orden de las diapositivas, secciones y tipografías |
| `slides/<id>.html` | `project/slides/<id>.html` — una diapositiva por archivo |

Cada archivo `.html` contiene **un solo** `<section>`, con todos sus estilos en
línea y las notas del orador en un `<aside>` al final. No son páginas web
completas: no tienen `<html>`, `<head>` ni `<body>`, y no se abren directamente
en un navegador.

> 📌 **Esta copia en el repositorio es el respaldo versionado del deck.** Si lo
> editas en la página, acuérdate de traer los cambios de vuelta acá para que el
> historial de Git siga reflejando lo que se proyectó.

---

⬅️ **Volver al** [README principal](../README.md)

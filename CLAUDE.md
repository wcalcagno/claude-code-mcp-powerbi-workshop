# Contexto del proyecto para Claude Code

Este archivo lo lee Claude Code automáticamente al iniciar una sesión en este
repositorio. Define qué es este proyecto y cómo debes comportarte dentro de él.

---

## Qué es este repositorio

Es el material de un **workshop educativo de 2 horas** llamado
*"Claude Code + MCP para Power BI y Fabric"*.

La audiencia son **analistas de datos y BI** con perfil Power BI (DAX, Power Query,
modelado) y **poca o nula experiencia en terminal, Git o programación**.

El repositorio no contiene una aplicación: contiene **documentación didáctica en
Markdown**, archivos de configuración de ejemplo y ejercicios guiados.

---

## Cómo debes comportarte en este repositorio

### Idioma

- **Todo el contenido va en español neutro (Chile / LatAm).**
- Los términos técnicos establecidos se mantienen en inglés cuando es lo natural
  en el mundo BI: *workspace*, *dataset*, *semantic model*, *endpoint*, *commit*,
  *pull request*, *device code*. No los traduzcas forzadamente.

### Estilo del contenido

- **Tono didáctico.** Asume que quien lee quizá nunca abrió una terminal.
- Cada instrucción con comandos debe decir: qué hace el comando, cómo ejecutarlo,
  y **qué debería ver la persona** si salió bien.
- Prefiere pasos numerados, tablas y ejemplos concretos por sobre párrafos largos.
- Evita la jerga innecesaria. Si usas un término nuevo, defínelo en una frase.

### Estilo del código de ejemplo

- **Simple por sobre elegante.** Prioriza la claridad absoluta.
- **Comentado en español.** Cada bloque de código o configuración debe explicar
  qué hace cada parte relevante.
- Nada de abstracciones, patrones avanzados ni optimizaciones. Si hay dos formas
  de hacer algo, elige la que se entiende leyéndola una sola vez.

### Datos de conexión

- **Nunca inventes credenciales, IDs ni endpoints reales.**
- Usa siempre placeholders explícitos y en mayúsculas:
  `TU_TENANT_ID`, `TU_CLIENT_ID`, `TU_WORKSPACE_ID`, `TU_DATASET_ID`,
  `C:/RUTA/A/TU/SCRIPT.py`.
- **Los endpoints oficiales de Microsoft sí van escritos con su valor real**,
  porque están publicados en la documentación y no son secretos:
  - Power BI **Consumption** MCP (el del taller):
    `https://api.fabric.microsoft.com/v1/mcp/powerbi`
  - Power BI **Authoring** MCP (el que escribe, **no** se usa en el taller):
    `https://api.fabric.microsoft.com/v1/mcp/powerbi/authoring`
  - **Fabric IQ** MCP: `https://fabriciq.svc.cloud.microsoft/v1/mcp/FabricIQ`
- El endpoint de Consumption está **en preview**: cada vez que lo menciones,
  acompáñalo de esa advertencia y del enlace a la documentación oficial.
- Lo que sí son secretos, y nunca se escriben en un archivo: tokens, client
  secrets y contraseñas.

---

## Punto clave del taller: el MCP local es de SOLO LECTURA

El servidor MCP que este workshop usa contra **Power BI Desktop** es
**exclusivamente de lectura**. Lee metadatos del modelo (tablas, columnas,
relaciones, medidas, jerarquías) y ejecuta consultas DAX de lectura. **No crea,
no modifica y no elimina** medidas, tablas, columnas ni ninguna parte del modelo.

**Esto es intencional y es un punto de diseño pedagógico, no una limitación que
haya que disculpar.** La razón:

> En un taller, con personas usando la herramienta por primera vez, es esperable
> que alguien apruebe una acción de Claude Code sin leerla con atención. Si el
> servidor solo puede leer, el peor escenario posible es una consulta inútil.
> Nunca un modelo corrupto ni una medida sobrescrita.

**Cuando escribas o edites material de este repositorio:**

- Refuerza este punto donde sea relevante, con tono positivo ("elegimos un
  servidor de solo lectura porque…"), no defensivo ("lamentablemente no puede…").
- **No propongas** ejercicios que impliquen crear o modificar objetos del modelo
  a través del MCP local. No es posible, y sugerirlo confunde a la audiencia.
- Si un ejercicio necesita **escribir un archivo** (por ejemplo, documentación
  `.md` de las medidas), deja claro que esa escritura la hace **Claude Code
  directamente sobre el sistema de archivos**, no el MCP. El MCP solo aportó la
  lectura de los datos.
- El bloque de Fabric / Power BI Service también se trabaja en **modo consulta**,
  no de escritura.

---

## Estructura del repositorio

```
00-setup/              Instalación: Git, GitHub, Claude Code
01-claude-code-basico/ Fundamentos de Claude Code + primer commit
02-mcp-powerbi-local/  MCP de solo lectura sobre Power BI Desktop
03-mcp-fabric-api/     MCP remoto de Microsoft sobre Service / Fabric
04-flujo-completo/     Ejercicio integrador local vs. Service
recursos/              Enlaces de referencia
```

Cada carpeta de contenido tiene un `README.md` que la explica, y las carpetas
`ejercicios/` contienen ejercicios individuales.

---

## Formato obligatorio de cada ejercicio

Todo ejercicio de este repositorio tiene **las tres secciones**, en este orden:

1. **Objetivo** — qué vas a lograr, en una o dos frases.
2. **Prompt sugerido** — el texto exacto, en un bloque de código, que la persona
   puede copiar y pegar en Claude Code.
3. **Resultado esperado** — qué debería verse en pantalla si todo funcionó.

Si agregas un ejercicio nuevo, respeta esta estructura.

---

## Qué NO hacer en este repositorio

- No agregues dependencias, `package.json` ni herramientas de build. Este repo
  es documentación.
- No commitees archivos `.pbix` con datos reales, `.env`, tokens ni credenciales
  (ver `.gitignore`).
- No hagas `git push` sin que la persona te lo pida explícitamente.

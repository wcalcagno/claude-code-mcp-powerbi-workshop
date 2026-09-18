# 01 · Claude Code básico

⏱️ **Duración del bloque:** 15 minutos
📍 **Requisito:** haber completado [00 · Setup](../00-setup/)

---

## ¿Qué es Claude Code?

Claude Code es **Claude trabajando dentro de tu terminal, sobre una carpeta de tu
computador**.

No es un chat donde pegas texto y copias la respuesta. Es un asistente que:

- **Lee** los archivos de la carpeta donde lo abriste.
- **Escribe y edita** esos archivos directamente.
- **Ejecuta comandos** (por ejemplo, comandos de Git).
- **Se conecta a herramientas externas** vía MCP — como tu modelo de Power BI.

---

## Diferencia con Claude.ai

| | **Claude.ai** (navegador) | **Claude Code** (terminal) |
|---|---|---|
| Dónde vive | Página web | Tu computador |
| Ve tus archivos | Solo los que subes uno por uno | Toda la carpeta donde lo abriste |
| Modifica archivos | No, te devuelve texto para copiar | Sí, edita los archivos reales |
| Ejecuta comandos | No | Sí (te pide permiso antes) |
| Conecta con Power BI Desktop | No | Sí, vía MCP |
| Guarda en Git | No | Sí, puede hacer commits |

**La analogía útil:** Claude.ai es como pedirle a un colega una fórmula DAX por
chat y después pegarla tú en el modelo. Claude Code es como sentar a ese colega
frente a tu computador, con la pantalla compartida — pero pidiendo tu aprobación
antes de tocar nada.

---

## El principio de los permisos

Claude Code **te pide autorización** antes de acciones que modifican algo:
editar un archivo, ejecutar un comando, escribir en disco.

Cuando lo haga, verás algo así:

```
¿Permitir editar README.md?
❯ 1. Sí
  2. Sí, y no preguntar de nuevo por este archivo
  3. No, y dile qué hacer distinto
```

> ⚠️ **Lee lo que te está pidiendo antes de aprobar.** Es un buen hábito desde
> el primer día. Y es exactamente la razón por la que el MCP de Power BI que
> usaremos en el bloque 2 es **de solo lectura**: para que, incluso si alguien
> aprueba sin leer, no haya nada que dañar en el modelo.

---

## Comandos esenciales

### Desde PowerShell (antes de entrar)

| Comando | Qué hace |
|---|---|
| `cd RUTA` | Te mueve a la carpeta del proyecto. **Hazlo siempre antes de abrir Claude Code.** |
| `claude` | Inicia una sesión nueva. |
| `claude -c` | Retoma la conversación anterior en esa carpeta. |
| `claude --version` | Muestra la versión instalada. |

### Dentro de Claude Code

Lo que escribas normalmente es un mensaje para Claude. Lo que empieza con un
símbolo especial es una instrucción para la herramienta:

| Símbolo / comando | Qué hace |
|---|---|
| `/help` | Lista los comandos disponibles. |
| `/status` | Cuenta, modelo, carpeta de trabajo. |
| `/mcp` | Servidores MCP conectados y sus herramientas. |
| `/clear` | Limpia la conversación y parte de cero (útil entre bloques del taller). |
| `/exit` | Vuelve a PowerShell. |
| `@` | Escribe `@` y el nombre de un archivo para dárselo como contexto: `@README.md`. |
| `Esc` | Interrumpe a Claude si está haciendo algo que no querías. |
| `Ctrl+C` (dos veces) | Salida de emergencia. |

---

## Cómo dar buen contexto

La diferencia entre una respuesta útil y una inútil casi siempre está en el prompt.

### ❌ Vago

```
arregla el archivo
```

### ✅ Específico

```
En el archivo @README.md, la tabla de la agenda tiene los tiempos
desordenados. Ordénala de menor a mayor y deja el resto igual.
```

**Tres reglas simples:**

1. **Di qué archivo** — con `@` o por su nombre.
2. **Di qué resultado quieres**, no cómo lograrlo. Claude decide el cómo.
3. **Di qué no tocar**, si te importa preservar algo.

> 💡 Claude Code lee automáticamente el archivo `CLAUDE.md` de este repositorio al
> iniciar. Ahí ya está escrito que el contenido va en español, con tono didáctico
> y comentarios en español. Por eso no necesitas repetirlo en cada prompt: el
> `CLAUDE.md` es "el contexto permanente" del proyecto.

---

## Ejercicio guiado · Tu primer archivo y tu primer commit

### Objetivo

Comprobar que Claude Code puede crear un archivo en tu repositorio y guardarlo en
el historial de Git — el flujo completo que usaremos durante todo el workshop.

⏱️ 5 minutos.

---

### Paso 1 · Abre Claude Code en la carpeta del workshop

Desde PowerShell:

```powershell
cd $env:USERPROFILE\workshops\claude-code-mcp-powerbi-workshop
```

```powershell
claude
```

---

### Paso 2 · Pídele que cree el archivo

**Prompt sugerido** (cópialo tal cual):

```
Crea un archivo llamado mis-notas.md en la raíz del repositorio.
Que contenga:
- Un título de nivel 1: "Notas del workshop"
- Mi nombre y la fecha de hoy
- Una sección "Lo que quiero aprender hoy" con tres viñetas
  relacionadas con Power BI y Claude Code
Escríbelo en español.
```

Claude te mostrará el contenido propuesto y te pedirá permiso para crear el archivo.
**Léelo** y aprueba con la opción `1`.

**Resultado esperado:** un mensaje confirmando que se creó `mis-notas.md`.

---

### Paso 3 · Revisa el archivo tú mismo

**Prompt sugerido:**

```
Muéstrame el contenido de @mis-notas.md
```

**Resultado esperado:** Claude imprime el contenido del archivo. Debería tener el
título, tu nombre, la fecha y las tres viñetas.

> 💡 También puedes abrirlo en el Explorador de Windows o con el Bloc de notas.
> El archivo es real, está en tu disco.

---

### Paso 4 · Pídele que lo commitee

Un **commit** es un punto guardado en el historial, con un mensaje que lo explica.

**Prompt sugerido:**

```
Agrega mis-notas.md a Git y haz un commit con un mensaje en español
que describa qué es el archivo. No hagas push todavía.
```

Claude te pedirá permiso para ejecutar los comandos de Git. Verás algo como:

```
¿Ejecutar git add mis-notas.md?
¿Ejecutar git commit -m "Agrega archivo de notas personales del workshop"?
```

Apruébalos.

**Resultado esperado:**

```
[main 3f8a21c] Agrega archivo de notas personales del workshop
 1 file changed, 9 insertions(+)
 create mode 100644 mis-notas.md
```

Traducción de esa salida:

| Parte | Significado |
|---|---|
| `main` | La rama donde quedó guardado. |
| `3f8a21c` | El identificador único de este commit. |
| `1 file changed, 9 insertions(+)` | Cambió un archivo y se agregaron 9 líneas. |

---

### Paso 5 · Confirma el historial

**Prompt sugerido:**

```
Muéstrame los últimos 3 commits del repositorio en una línea cada uno.
```

**Resultado esperado:** una lista corta donde el commit más reciente es el tuyo.

---

## Lo que acabas de hacer

Sin escribir un solo comando de Git a mano, completaste el ciclo de trabajo
profesional: **crear → revisar → guardar en el historial**.

Ese mismo ciclo es el que vas a repetir en el bloque 4, pero con contenido real
extraído de tus modelos de Power BI.

---

## ✅ Checklist antes de continuar

- [ ] Existe el archivo `mis-notas.md` en la raíz del repositorio.
- [ ] Hiciste al menos un commit.
- [ ] Sabes cómo salir (`/exit`) y volver a entrar (`claude`).
- [ ] Sabes qué hace el comando `/mcp` (lo usaremos en el bloque siguiente).

---

⬅️ **Anterior:** [00 · Setup](../00-setup/03-instalar-claude-code.md)
➡️ **Siguiente:** [02 · MCP Power BI local](../02-mcp-powerbi-local/README.md)

# 03 · Instalar Claude Code

⏱️ **Tiempo estimado:** 5 minutos
📍 **Bloque:** 00 · Setup

---

## ¿Qué vamos a instalar?

**Claude Code** es Claude funcionando **dentro de tu terminal**, con acceso a la
carpeta en la que lo abras. Puede leer tus archivos, editarlos, ejecutar comandos
y conectarse a herramientas externas (los famosos MCP, que veremos en el bloque 2).

---

## Requisito previo

Necesitas una **cuenta de Claude** con acceso a Claude Code: un plan Pro o Max, o
una cuenta de la API de Anthropic con crédito disponible. Si trabajas con la cuenta
de tu empresa, confirma antes con quien la administra.

---

## Paso 1 · Instalar

Elige **una** de las dos opciones.

### Opción A · Instalador nativo (recomendado en Windows)

No requiere tener Node.js instalado. Abre **PowerShell** y ejecuta:

```powershell
irm https://claude.ai/install.ps1 | iex
```

Qué hace cada parte:

| Parte | Significado |
|---|---|
| `irm` | Abreviación de `Invoke-RestMethod`: descarga el contenido de una URL. |
| `\|` | El símbolo *pipe*: pasa lo que salió del comando anterior al siguiente. |
| `iex` | Abreviación de `Invoke-Expression`: ejecuta lo descargado. |

En conjunto: descarga el script oficial de instalación de Anthropic y lo ejecuta.

> ⚠️ Si tu empresa bloquea la ejecución de scripts descargados, usa la Opción B.

### Opción B · Con npm (si ya tienes Node.js)

Requiere **Node.js 18 o superior**. Verifica con:

```powershell
node --version
```

Si responde con un número igual o mayor a `v18`, instala:

```powershell
npm install -g @anthropic-ai/claude-code
```

El `-g` significa *global*: queda disponible desde cualquier carpeta.

Si `node` no está instalado y quieres esta vía:

```powershell
winget install OpenJS.NodeJS.LTS
```

Luego **cierra y abre PowerShell** antes de ejecutar el `npm install`.

---

## Paso 2 · Verificar la instalación

**Cierra PowerShell y abre una ventana nueva** (igual que con Git: la terminal
solo reconoce programas nuevos al reiniciarse).

```powershell
claude --version
```

**Resultado esperado:** un número de versión, por ejemplo:

```
2.0.14 (Claude Code)
```

El número exacto cambia con cada actualización. Lo importante es que responda
con una versión y no con un error.

---

## Paso 3 · Primer inicio de sesión

### 1. Ubícate en la carpeta del workshop

Esto es **importante**: Claude Code trabaja sobre la carpeta desde la que lo
abres. Si lo abres en el lugar equivocado, no verá los archivos del taller.

```powershell
cd $env:USERPROFILE\workshops\claude-code-mcp-powerbi-workshop
```

Confirma que estás en el lugar correcto:

```powershell
ls
```

Deberías ver las carpetas `00-setup`, `01-claude-code-basico`, etc.

### 2. Iniciar Claude Code

```powershell
claude
```

### 3. Autenticarte

La primera vez, Claude Code te guía por una configuración breve:

1. **Elige un tema** (claro / oscuro). Usa las flechas y `Enter`.
2. **Método de autenticación:**
   - *Claude account with subscription* → si tienes plan Pro o Max.
   - *Anthropic Console account* → si usas la API con facturación.
3. Se abre tu **navegador** en la página de login de Anthropic. Inicia sesión y
   autoriza el acceso.
4. Vuelve a la terminal. Debería decir que la autenticación fue exitosa.

> 💡 Si el navegador no se abre solo, la terminal muestra una URL. Cópiala y
> pégala manualmente en tu navegador.

### 4. Confirma que estás dentro

Cuando la sesión inicia, verás la caja de entrada de Claude Code:

```
╭──────────────────────────────────────────────────╮
│ >                                                │
╰──────────────────────────────────────────────────╯
```

Ese `>` es donde escribes. **Ya no estás escribiendo comandos de PowerShell:**
ahora le hablas a Claude en español normal.

---

## Paso 4 · Verificación con un comando simple

Escribe esto en la caja de Claude Code y presiona `Enter`:

```
¿En qué carpeta estás trabajando y qué archivos ves en el primer nivel?
```

**Resultado esperado:** Claude responde con la ruta de la carpeta del workshop y
un listado de las carpetas `00-setup`, `01-claude-code-basico`,
`02-mcp-powerbi-local`, `03-mcp-fabric-api`, `04-flujo-completo` y `recursos`,
más `README.md` y `CLAUDE.md`.

Si menciona el archivo `CLAUDE.md` o el contenido del workshop, mejor todavía:
significa que **leyó automáticamente el contexto del proyecto**.

---

## Comandos útiles de la sesión

Dentro de Claude Code, las instrucciones que empiezan con `/` son comandos de la
herramienta, no mensajes para Claude:

| Comando | Qué hace |
|---|---|
| `/help` | Lista los comandos disponibles. |
| `/status` | Muestra cuenta, modelo y estado de la sesión. |
| `/clear` | Borra el contexto de la conversación y parte de cero. |
| `/mcp` | Muestra los servidores MCP conectados. **Lo usaremos mucho en el bloque 2.** |
| `/exit` | Cierra Claude Code y vuelve a PowerShell. |

Para salir también puedes presionar `Ctrl+C` dos veces.

---

## Problemas frecuentes

| Síntoma | Causa probable | Solución |
|---|---|---|
| `El término 'claude' no se reconoce` | La terminal se abrió antes de instalar. | Cierra y abre PowerShell. Si persiste, reinicia el PC. |
| `No se puede cargar el archivo ... install.ps1 porque la ejecución de scripts está deshabilitada` | Política de ejecución de PowerShell. | Usa la Opción B (npm), o pide apoyo a TI. |
| `npm : El término 'npm' no se reconoce` | Node.js no está instalado. | Instala Node.js o usa la Opción A. |
| El navegador no abre para el login | Navegador por defecto no configurado. | Copia la URL que muestra la terminal y pégala a mano. |
| `EACCES` / permisos al instalar con npm | Instalación global sin permisos. | Usa la Opción A (instalador nativo). |
| Claude no ve los archivos del workshop | Lo abriste en otra carpeta. | Sal con `/exit`, haz `cd` a la carpeta correcta y vuelve a ejecutar `claude`. |

---

## ✅ Checklist antes de continuar

- [ ] `claude --version` responde con un número de versión.
- [ ] Iniciaste sesión correctamente.
- [ ] Abriste Claude Code **dentro** de la carpeta del workshop.
- [ ] Claude te describió correctamente el contenido de la carpeta.

---

➡️ **Siguiente:** [01 · Claude Code básico](../01-claude-code-basico/README.md)

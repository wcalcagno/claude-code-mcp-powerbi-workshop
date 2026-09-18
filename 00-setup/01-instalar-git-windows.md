# 01 · Instalar Git en Windows

⏱️ **Tiempo estimado:** 5 minutos
📍 **Bloque:** 00 · Setup

---

## ¿Qué es Git y para qué lo necesito?

Git es un sistema que **guarda el historial de tus archivos**. Piensa en el
"Control de versiones" de SharePoint o en el historial de versiones de un archivo
de Excel en OneDrive, pero mucho más preciso: guarda exactamente qué línea de
qué archivo cambió, cuándo y por qué.

En este workshop lo necesitamos por dos razones:

1. **Claude Code lo usa** para registrar los cambios que hace en tus archivos.
   Así siempre puedes ver qué modificó y deshacerlo si no te gusta.
2. **Es lo que conecta tu PC con GitHub**, donde vas a guardar el trabajo del taller.

---

## Antes de empezar: ¿qué es "la terminal"?

La terminal es una ventana donde escribes comandos en vez de hacer clic en botones.
En Windows vamos a usar **PowerShell**.

**Para abrirla:**

1. Presiona la tecla `Windows`.
2. Escribe `powershell`.
3. Presiona `Enter`.

Se abre una ventana con fondo azul o negro y un texto parecido a:

```
PS C:\Users\TU_USUARIO>
```

Ese `PS C:\Users\TU_USUARIO>` se llama **prompt**. Significa "estoy listo,
escríbeme algo". Cuando este material te pida "ejecutar un comando", significa:
escribirlo ahí y presionar `Enter`.

> 💡 **Tip:** en la terminal, `Ctrl+V` pega texto normalmente. Si copias un
> comando desde este documento, puedes pegarlo tal cual.

---

## Paso 1 · Verifica si Git ya está instalado

Puede que ya lo tengas. Ejecuta:

```powershell
git --version
```

**Si ves algo como esto**, ya tienes Git y puedes saltar directo al Paso 3:

```
git version 2.47.1.windows.1
```

**Si ves un error** como `El término 'git' no se reconoce...`, sigue al Paso 2.

---

## Paso 2 · Instalar Git

Tienes dos caminos. **El A es más rápido**; usa el B si el A no funciona.

### Opción A · Con winget (recomendado)

`winget` es el instalador de aplicaciones que viene incluido en Windows 10 y 11.

```powershell
winget install --id Git.Git -e --source winget
```

Qué hace cada parte:

| Parte | Significado |
|---|---|
| `winget install` | Instala una aplicación. |
| `--id Git.Git` | El identificador exacto del paquete oficial de Git. |
| `-e` | *Exact*: coincidencia exacta del identificador, para no instalar otra cosa. |
| `--source winget` | Usa el repositorio oficial de Microsoft. |

Durante la instalación verás una barra de progreso. Cuando termine, aparecerá
un mensaje como `Se instaló correctamente`.

> ⚠️ Es posible que Windows te pida permisos de administrador. Acepta.

### Opción B · Con el instalador oficial

1. Abre <https://git-scm.com/download/win> en tu navegador.
2. La descarga del instalador de 64 bits empieza sola. Si no, elige
   **"64-bit Git for Windows Setup"**.
3. Ejecuta el archivo `.exe` descargado.
4. **Acepta todas las opciones por defecto.** El instalador tiene muchas pantallas
   con opciones técnicas; los valores predeterminados son correctos para este taller.
   Presiona `Next` hasta llegar a `Install`.
5. Al final, puedes desmarcar "View Release Notes" y presionar `Finish`.

---

## Paso 3 · Verifica que quedó instalado

**Cierra la ventana de PowerShell y abre una nueva.** Esto es importante: la
terminal solo reconoce programas nuevos al reiniciarse.

Ejecuta:

```powershell
git --version
```

**Resultado esperado:**

```
git version 2.47.1.windows.1
```

El número puede ser distinto al de este ejemplo. Lo que importa es que aparezca
la palabra `git version` seguida de un número.

---

## Paso 4 · Configuración mínima

Git necesita saber **quién eres** para poder firmar los cambios que guardas.
Esto se configura una sola vez por computador.

Ejecuta estos dos comandos, **reemplazando los valores por los tuyos**:

```powershell
git config --global user.name "Tu Nombre Apellido"
```

```powershell
git config --global user.email "tu.correo@empresa.cl"
```

Qué significan:

| Parte | Significado |
|---|---|
| `git config` | Cambia la configuración de Git. |
| `--global` | Aplica a todos tus proyectos en este computador (no solo a uno). |
| `user.name` | El nombre que aparecerá junto a cada cambio que guardes. |
| `user.email` | El correo asociado. **Usa el mismo que vas a usar en GitHub**, así GitHub reconoce que los cambios son tuyos. |

> 📌 Las comillas son necesarias si tu nombre tiene espacios. Déjalas siempre.

### Verifica la configuración

```powershell
git config --global --list
```

**Resultado esperado** (además de otras líneas que pueda mostrar):

```
user.name=Tu Nombre Apellido
user.email=tu.correo@empresa.cl
```

---

## Paso 5 · Define la rama por defecto (opcional pero recomendado)

Por razones históricas, Git llama `master` a la rama principal, pero GitHub usa
`main`. Para evitar confusiones:

```powershell
git config --global init.defaultBranch main
```

---

## Problemas frecuentes

| Síntoma | Causa probable | Solución |
|---|---|---|
| `El término 'git' no se reconoce como nombre de un cmdlet` | La terminal se abrió antes de instalar Git. | Cierra PowerShell y ábrelo de nuevo. Si persiste, reinicia el computador. |
| `winget` tampoco se reconoce | Windows desactualizado o sin App Installer. | Usa la **Opción B** (instalador oficial). |
| La instalación pide credenciales de administrador | Política de tu empresa. | Pide apoyo a TI, o usa la Opción B que a veces permite instalación por usuario. |
| `git config` no muestra tu nombre | Escribiste el comando sin `--global`. | Vuelve a ejecutarlo incluyendo `--global`. |

---

## ✅ Checklist antes de continuar

- [ ] `git --version` responde con un número de versión.
- [ ] `git config --global --list` muestra tu `user.name` y tu `user.email`.

---

➡️ **Siguiente:** [02 · Configurar GitHub](./02-configurar-github.md)

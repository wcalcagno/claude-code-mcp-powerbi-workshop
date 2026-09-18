# 02 · Configurar GitHub

⏱️ **Tiempo estimado:** 5 minutos
📍 **Bloque:** 00 · Setup

---

## ¿Qué es GitHub?

Si **Git** es el historial de versiones que vive en tu computador, **GitHub** es
el lugar en internet donde ese historial se respalda y se comparte.

Comparación rápida con tu mundo:

| En Power BI / Office | En este taller |
|---|---|
| Un `.pbix` en tu carpeta local | Un repositorio Git en tu PC |
| Ese `.pbix` publicado en un workspace | Ese repositorio subido a GitHub |
| Historial de versiones de OneDrive | Historial de commits de Git |

Un **repositorio** (o "repo") es simplemente una carpeta cuyo historial Git está
siguiendo.

---

## Paso 1 · Crear tu cuenta (si no tienes)

1. Entra a <https://github.com/signup>.
2. Ingresa tu correo. **Recomendación:** usa el mismo correo que configuraste en
   `git config --global user.email` en el paso anterior. Así GitHub asocia
   automáticamente tus cambios con tu perfil.
3. Elige una contraseña y un nombre de usuario (`username`). Tu nombre de usuario
   será visible públicamente y aparecerá en las URLs de tus repositorios.
4. Confirma el correo con el código que te llega.
5. Cuando pregunte por plan, **Free** es suficiente para todo este workshop.

> 📌 Anota tu nombre de usuario. Lo vas a necesitar en varios comandos. En este
> material aparece como `TU_USUARIO`.

---

## Paso 2 · Activar verificación en dos pasos (2FA)

GitHub la exige para todas las cuentas. Si te la pide, configúrala con la app de
autenticación que uses habitualmente (Microsoft Authenticator, Google Authenticator,
etc.).

**Guarda los códigos de recuperación** que GitHub te muestra. Son tu única forma
de entrar si pierdes el teléfono.

---

## Paso 3 · Elegir cómo te vas a autenticar

Cuando subas cambios a GitHub desde tu PC, GitHub necesita confirmar que eres tú.
Hay dos formas. **Para este workshop recomendamos la Opción A.**

| | Opción A · Token (HTTPS) | Opción B · Llave SSH |
|---|---|---|
| Dificultad | Baja | Media |
| Requiere terminal | No, todo por web | Sí |
| Funciona detrás de proxies corporativos | Casi siempre | A veces bloqueado |
| Recomendado para este taller | ✅ Sí | Solo si ya la usas |

---

## Opción A · Token de acceso personal (fine-grained)

Un **token** es como una contraseña de un solo propósito: la generas para una
tarea específica, le das permisos limitados y le pones fecha de vencimiento.
Un token *fine-grained* ("de grano fino") te deja elegir exactamente a qué
repositorios y con qué permisos aplica.

### Generarlo

1. Entra a GitHub y haz clic en tu **foto de perfil** (arriba a la derecha) →
   **Settings**.
2. En el menú lateral, baja hasta el final: **Developer settings**.
3. **Personal access tokens** → **Fine-grained tokens**.
4. Botón **Generate new token**.
5. Completa así:

   | Campo | Qué poner |
   |---|---|
   | **Token name** | `workshop-claude-code-powerbi` |
   | **Expiration** | 30 días (o lo que permita tu organización). Nunca "No expiration". |
   | **Resource owner** | Tu propio usuario. |
   | **Repository access** | `Only select repositories` → elige tu copia del repo del workshop. Si aún no existe, puedes usar `All repositories` y ajustarlo después. |
   | **Repository permissions** | Busca **Contents** y ponlo en **Read and write**. Eso basta para clonar y hacer push. |

6. Presiona **Generate token**.
7. **Copia el token ahora.** GitHub lo muestra **una sola vez**. Empieza con
   `github_pat_...`.

> 🔐 **Trátalo como una contraseña.** No lo pegues en un chat, ni en un archivo
> del repositorio, ni se lo dictes a nadie. El `.gitignore` de este repo está
> configurado para ignorar archivos de tokens, pero la mejor protección es no
> escribirlo en ningún archivo.

### Usarlo

No hay que "instalarlo". La primera vez que hagas una operación contra GitHub
desde tu PC, Windows te pedirá credenciales:

- **Usuario:** tu nombre de usuario de GitHub.
- **Contraseña:** pega el **token** (no tu contraseña de GitHub).

El Administrador de credenciales de Windows lo guarda y no te lo vuelve a pedir.

---

## Opción B · Llave SSH

Solo si prefieres este método o ya lo usas en tu trabajo.

### 1. Generar la llave

```powershell
ssh-keygen -t ed25519 -C "tu.correo@empresa.cl"
```

- Cuando pregunte dónde guardarla, presiona `Enter` para aceptar la ruta por defecto.
- Cuando pida *passphrase*, puedes dejarla vacía (`Enter` dos veces) o poner una
  frase que recuerdes.

### 2. Copiar la llave pública al portapapeles

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub | Set-Clipboard
```

> ⚠️ El archivo que termina en `.pub` es la llave **pública** — esa sí se comparte.
> El archivo sin `.pub` es la llave **privada** y nunca se comparte con nadie.

### 3. Registrarla en GitHub

1. GitHub → foto de perfil → **Settings** → **SSH and GPG keys**.
2. **New SSH key**.
3. **Title:** `PC workshop`. **Key:** pega con `Ctrl+V`.
4. **Add SSH key**.

### 4. Probar la conexión

```powershell
ssh -T git@github.com
```

**Resultado esperado:**

```
Hi TU_USUARIO! You've successfully authenticated, but GitHub does not provide shell access.
```

Ese mensaje, aunque diga "does not provide shell access", significa que **funcionó**.

---

## Paso 4 · Primer ejercicio: clonar este repositorio

"Clonar" significa **descargar una copia del repositorio a tu computador**,
incluyendo todo su historial.

### 1. Ubícate en una carpeta de trabajo

Vamos a crear una carpeta `workshops` en tu directorio de usuario:

```powershell
mkdir $env:USERPROFILE\workshops
```

```powershell
cd $env:USERPROFILE\workshops
```

| Comando | Qué hace |
|---|---|
| `mkdir` | Crea una carpeta (*make directory*). |
| `cd` | Entra a una carpeta (*change directory*). |
| `$env:USERPROFILE` | Variable de Windows que apunta a `C:\Users\TU_USUARIO`. |

El prompt ahora debería mostrar `PS C:\Users\TU_USUARIO\workshops>`.

### 2. Clonar

Reemplaza `TU_USUARIO` por el usuario de GitHub donde está el repositorio
(el de quien dicta el workshop, o el tuyo si hiciste un *fork*):

```powershell
git clone https://github.com/TU_USUARIO/claude-code-mcp-powerbi-workshop.git
```

**Resultado esperado:**

```
Cloning into 'claude-code-mcp-powerbi-workshop'...
remote: Enumerating objects: 28, done.
remote: Counting objects: 100% (28/28), done.
remote: Compressing objects: 100% (20/20), done.
Receiving objects: 100% (28/28), done.
```

Si es la primera vez, aquí es donde Windows te pide usuario y **token**.

### 3. Entrar a la carpeta clonada

```powershell
cd claude-code-mcp-powerbi-workshop
```

### 4. Ver qué hay adentro

```powershell
ls
```

**Resultado esperado:** un listado con `00-setup`, `01-claude-code-basico`,
`02-mcp-powerbi-local`, `03-mcp-fabric-api`, `04-flujo-completo`, `recursos`,
`README.md` y `CLAUDE.md`.

### 5. Confirmar que Git está siguiendo esta carpeta

```powershell
git status
```

**Resultado esperado:**

```
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

Traducción: "estás en la rama `main`, estás al día con GitHub, y no hay cambios
pendientes". Ese es el estado limpio de partida.

---

## Glosario rápido

| Término | Significado |
|---|---|
| **Repositorio / repo** | Carpeta cuyo historial sigue Git. |
| **Clonar** | Descargar una copia del repo con todo su historial. |
| **Commit** | Un punto guardado en el historial, con un mensaje que lo describe. |
| **Push** | Subir tus commits locales a GitHub. |
| **Pull** | Bajar a tu PC los commits que hay en GitHub. |
| **Rama / branch** | Una línea de trabajo. La principal se llama `main`. |
| **Fork** | Tu copia personal, en tu cuenta de GitHub, del repo de otra persona. |

---

## Problemas frecuentes

| Síntoma | Solución |
|---|---|
| `remote: Support for password authentication was removed` | Estás usando tu contraseña de GitHub. Usa el **token** como contraseña. |
| `Authentication failed` con el token correcto | Windows guardó una credencial vieja. Abre *Administrador de credenciales* → *Credenciales de Windows* → elimina la entrada `git:https://github.com` y reintenta. |
| `fatal: repository not found` | Revisa la URL, o pide acceso si el repositorio es privado. |
| `Permission denied (publickey)` | Estás usando SSH y la llave no quedó registrada. Repite la Opción B desde el paso 2. |

---

## ✅ Checklist antes de continuar

- [ ] Tienes cuenta de GitHub y puedes entrar.
- [ ] Generaste un token fine-grained (o configuraste tu llave SSH).
- [ ] Clonaste el repositorio y `git status` responde `working tree clean`.
- [ ] Estás parado dentro de la carpeta `claude-code-mcp-powerbi-workshop`.

---

➡️ **Siguiente:** [03 · Instalar Claude Code](./03-instalar-claude-code.md)

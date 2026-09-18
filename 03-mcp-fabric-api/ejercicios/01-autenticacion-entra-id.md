# Ejercicio 1 · Autenticación con Entra ID

⏱️ **Tiempo estimado:** 12 minutos
📍 **Bloque:** [03 · MCP Fabric / API](../README.md)

---

## Objetivo

Completar el inicio de sesión con Entra ID para que Claude Code pueda consultar
tus semantic models de Power BI Service / Fabric **con tu propia identidad y tus
propios permisos**, sin que tu contraseña pase nunca por la terminal.

---

## Antes de empezar

- [ ] Tu administrador habilitó el tenant setting *"Users can use the Power BI
      Model Context Protocol server endpoint (preview)"*.
- [ ] Registraste la app en Entra ID con el redirect URI
      `http://localhost:8080/callback` y los permisos delegados
      (ver [README del bloque](../README.md#configuración)).
- [ ] Registraste el servidor con `claude mcp add ... --callback-port 8080`.
- [ ] Reiniciaste Claude Code.
- [ ] Tu navegador tiene sesión iniciada en <https://app.powerbi.com>.

---

## Cómo funciona el inicio de sesión

Claude Code usa **OAuth 2.0 con authorization code + PKCE**. Suena complicado; el
flujo real es simple:

```
  ┌─────────────────┐                        ┌──────────────────┐
  │  Claude Code    │                        │    Microsoft     │
  │   (terminal)    │                        │    Entra ID      │
  └────────┬────────┘                        └────────┬─────────┘
           │                                          │
   1. Abre un servidor local temporal                 │
      escuchando en localhost:8080                    │
           │                                          │
           │  2. Abre tu navegador en la página       │
           │     de login de Microsoft                │
           ├─────────────────────────────────────────►│
           │                                          │
  ┌────────▼────────┐  3. Inicias sesión, completas   │
  │   TU NAVEGADOR  │     MFA y das consentimiento    │
  │  (ya logueado)  ├─────────────────────────────────►
  └────────┬────────┘                                 │
           │                                          │
           │  4. Microsoft redirige tu navegador a    │
           │     http://localhost:8080/callback       │
           │◄─────────────────────────────────────────┤
           │                                          │
   5. Claude Code recibe el código en ese puerto      │
      y lo canjea por un token de acceso             │
           │                                          │
   6. Ya puede consultar el Service                   │
```

**Lo importante del paso 3:** tu contraseña y tu MFA se manejan **solo en el
navegador, en el dominio de Microsoft**. Claude Code recibe al final un **token
temporal**, nunca tu credencial.

> 📌 **Por eso el puerto importa tanto.** El paso 4 solo funciona si el redirect
> URI que registraste en Entra ID es **exactamente** el mismo que Claude Code está
> escuchando. De ahí que fijemos `--callback-port 8080` y registremos
> `http://localhost:8080/callback`. Una diferencia de un dígito y el login falla
> con `AADSTS50011`.

---

## Paso 1 · Iniciar el flujo

Dentro de Claude Code, escribe:

```
/mcp
```

### Resultado esperado

Una lista de servidores MCP con sus estados:

```
powerbi-local     ✔ connected
powerbi-fabric    ⚠ needs authentication
```

Selecciona `powerbi-fabric` con las flechas, presiona `Enter` y elige la opción
de **autenticar / conectar**.

> 💡 **Alternativa desde la terminal**, sin abrir una sesión de Claude Code:
> ```powershell
> claude mcp login powerbi-fabric
> ```

---

## Paso 2 · Iniciar sesión en el navegador

Claude Code abre tu navegador en la página de login de Microsoft.

1. **Elige tu cuenta corporativa** — la misma con la que entras a Power BI Service.

   > ⚠️ Si aparecen varias cuentas (personal, de otra empresa, de otro tenant),
   > elige con cuidado. La cuenta que elijas determina **a qué modelos vas a
   > poder acceder**.

2. Completa el **MFA** si te lo pide (Authenticator, SMS, o el método de tu
   empresa).

> 💡 Si el navegador no se abre solo, la terminal muestra una URL. Cópiala y
> pégala a mano.

---

## Paso 3 · Leer la pantalla de consentimiento

Aparece una pantalla que enumera los permisos que estás autorizando:

```
¿Está intentando iniciar sesión en Power BI MCP - Claude Code?

Esta aplicación podrá:
  • Ver todos los conjuntos de datos          (Dataset.Read.All)
  • Ver todos los grupos de trabajo           (Workspace.Read.All)
```

**Léela. No es un trámite.**

- ✅ Deberías ver **solo permisos de lectura** (`.Read.All`).
- 🔴 Si aparece `SemanticModel.ReadWrite.All`, **cancela**. Ese es el permiso del
  servidor de *Authoring*, que este taller no usa. Significa que alguien lo
  agregó al registro de la app, y estarías autorizando escritura sobre tus
  modelos sin necesitarla.

> ⭐ **Este es el momento pedagógico del bloque.** En el bloque 2 la protección era
> estructural: el servidor *no sabía* escribir. Acá la protección eres **tú
> leyendo esta pantalla**, más los permisos que decidiste no pedir. Es un
> recordatorio útil de que, fuera del taller, esta pantalla es a menudo lo único
> que separa una integración prudente de una peligrosa.

Presiona **Aceptar**.

Si tu tenant exige consentimiento de administrador, acá te va a bloquear con
`AADSTS65001`: no es un error tuyo, es la política de tu empresa. Un admin debe
otorgarlo una vez en el registro de la app.

---

## Paso 4 · Volver a la terminal

El navegador muestra una página de confirmación y puedes cerrarla. Vuelve a
Claude Code.

### Resultado esperado

```
powerbi-fabric    ✔ connected    (4 tools)
```

Si sigue diciendo `needs authentication`, espera unos segundos y escribe `/mcp`
de nuevo.

---

## Paso 5 · Verificar qué herramientas tienes

### Prompt sugerido

```
Usando el MCP powerbi-fabric, lista las herramientas que tienes disponibles.
Para cada una dime, en español y en una línea, qué hace y qué datos necesita
como entrada. Después indícame explícitamente cuáles son de lectura y si
alguna puede modificar algo en Power BI Service.
```

### Resultado esperado

Las cuatro herramientas del servidor de Consumption:

| Herramienta | Qué hace | Entrada |
|---|---|---|
| **Get Semantic Model Schema** | Tablas, columnas, medidas, relaciones y jerarquías | ID del semantic model |
| **Execute Query** | Ejecuta DAX y devuelve el resultado | ID del modelo + DAX |
| **Get Report Metadata** | Páginas, visuales y filtros de un reporte | ID del reporte |
| **Generate Query** | Genera DAX desde lenguaje natural con Copilot | ID del modelo + pregunta |

Y la conclusión: **ninguna modifica nada**. Las cuatro leen o calculan.

> 📌 Fíjate también en lo que **no** hay: ninguna herramienta lista workspaces ni
> semantic models. Este servidor trabaja sobre un modelo que tú identificas por
> su ID. Es el tema del [ejercicio siguiente](./02-consultar-workspace-fabric.md).

---

## Anexo · El flujo *device code*

Puede que veas mencionado el **device code flow** en la documentación de Entra ID
o en otras herramientas. No es lo que usa Claude Code, pero vale la pena
conocerlo porque lo vas a encontrar.

Es un método de inicio de sesión para programas que **no pueden abrir un
navegador ni recibir un redirect** — servidores sin interfaz gráfica, sesiones
SSH, contenedores.

Funciona así:

1. La herramienta muestra un **código corto** (ej. `A1B2-C3D4`) y una URL.
2. Abres <https://microsoft.com/devicelogin> en cualquier navegador, incluso en
   tu teléfono.
3. Pegas el código, eliges tu cuenta y confirmas.
4. La herramienta detecta que autorizaste y continúa.

| | **Authorization code + PKCE** (Claude Code) | **Device code** |
|---|---|---|
| Necesita navegador en la misma máquina | Sí | No |
| Necesita un puerto de callback | Sí (`localhost:8080`) | No |
| Dónde escribes la contraseña | En el navegador, dominio Microsoft | En el navegador, dominio Microsoft |
| Cuándo se usa | Apps de escritorio y CLI con navegador | Servidores, SSH, dispositivos sin teclado |

> 🔐 **Precaución de seguridad real, y por eso lo incluimos:** el device code es
> una técnica conocida de *phishing*. Si alguna vez recibes un código de este tipo
> **por correo o por chat, sin haberlo pedido tú**, no lo ingreses: alguien estaría
> usando tu sesión para autorizar **su** aplicación. Solo ingresa un código que
> tú mismo acabas de generar desde tu propia terminal.

---

## Sobre la duración del token

| Situación | Qué pasa | Qué hacer |
|---|---|---|
| Uso normal | Claude Code refresca el token solo. | Nada. |
| Token expirado sin refresco | Aparece `401 Unauthorized`. | `/mcp` y vuelve a autenticarte. |
| Cambiaste de cuenta | Sigues viendo los permisos de la anterior. | `claude mcp logout powerbi-fabric` y vuelve a entrar. |

---

## Problemas frecuentes

| Código / síntoma | Qué significa | Solución |
|---|---|---|
| `failed` apenas conectas | El tenant setting no está habilitado. | **Causa #1.** Tu admin de Power BI debe habilitarlo. |
| `AADSTS50011` | El redirect URI no coincide. | Debe ser exactamente `http://localhost:8080/callback` y el `--callback-port` debe ser `8080`. |
| `AADSTS65001` | Falta consentimiento del administrador. | Un admin de Entra ID debe otorgarlo en la app. Es el bloqueo más común en empresas. |
| `AADSTS50020` | Tu cuenta no pertenece a ese tenant. | Elegiste la cuenta equivocada al iniciar sesión. |
| `AADSTS50076` | Se requiere MFA. | Completa el segundo factor. |
| `AADSTS53003` | Política de acceso condicional. | Consulta con TI; a veces basta con estar en la red corporativa o en un equipo administrado. |
| `AADSTS700016` | El client ID no existe en el tenant. | Revisa que copiaste el *Application (client) ID* correcto. |
| El navegador se abre pero queda cargando | El puerto 8080 está ocupado por otra aplicación. | Usa otro puerto: regístralo en Entra ID y vuelve a agregar el servidor con ese `--callback-port`. |

> 🤖 Copia el código `AADSTS` completo y pégalo en Claude Code:
> *"¿Qué significa este error de Entra ID y qué tengo que hacer?"*.

---

## ✅ Checklist

- [ ] `/mcp` muestra `powerbi-fabric` como `connected`.
- [ ] **Leíste la pantalla de consentimiento** y confirmaste que solo pedía lectura.
- [ ] Viste las cuatro herramientas disponibles.
- [ ] Entendiste que tu contraseña nunca pasó por la terminal.
- [ ] Sabes qué es el device code flow y por qué no ingresarías uno que no pediste.

---

⬅️ **Anterior:** [03 · README del bloque](../README.md)
➡️ **Siguiente:** [Ejercicio 2 · Consultar un modelo del Service](./02-consultar-workspace-fabric.md)

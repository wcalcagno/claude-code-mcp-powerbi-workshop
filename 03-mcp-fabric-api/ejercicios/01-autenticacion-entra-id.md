# Ejercicio 1 · Autenticación con Entra ID

⏱️ **Tiempo estimado:** 12 minutos
📍 **Bloque:** [03 · Fabric IQ](../README.md)

---

## Objetivo

Completar el inicio de sesión con Entra ID para que Claude Code pueda consultar
tus reportes y semantic models de Fabric **con tu propia identidad y tus propios
permisos**, sin que tu contraseña pase nunca por la terminal.

---

## Antes de empezar

- [ ] Registraste la app en Entra ID con el redirect URI
      `http://localhost:8080/callback` y los tres permisos delegados
      (ver [README del bloque](../README.md#configuración)).
- [ ] Registraste el servidor con `claude mcp add ... --callback-port 8080`.
- [ ] Reiniciaste Claude Code.
- [ ] Tu navegador tiene sesión iniciada en <https://app.powerbi.com>.

---

## Cómo funciona el inicio de sesión

Fabric IQ usa **OAuth 2.0 delegado** a través de Microsoft Entra ID. Claude Code
implementa el lado del cliente con *authorization code + PKCE*. Suena complicado;
el flujo real es simple:

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
  ┌────────▼────────┐  3. Eliges tu cuenta, completas │
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
   6. Ya puede consultar Fabric                       │
```

**Lo importante del paso 3:** tu contraseña y tu MFA se manejan **solo en el
navegador, en el dominio de Microsoft**. Claude Code recibe al final un **token
temporal**, nunca tu credencial. Después lo almacena, lo refresca y lo envía en
cada solicitud, sin que tengas que volver a intervenir.

> 📌 **Por eso el puerto importa tanto.** El paso 4 solo funciona si el redirect
> URI que registraste en Entra ID es **exactamente** el que Claude Code está
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
fabric-iq         ⚠ needs authentication
```

Selecciona `fabric-iq` con las flechas, presiona `Enter` y elige la opción de
**autenticar / conectar**.

> 💡 **Alternativa desde la terminal**, sin abrir una sesión de Claude Code:
> ```powershell
> claude mcp login fabric-iq
> ```

---

## Paso 2 · Iniciar sesión en el navegador

Claude Code abre tu navegador en la página de login de Microsoft.

1. **Elige tu cuenta de trabajo o educación** — la misma con la que entras a
   Power BI Service.

   > ⚠️ Si aparecen varias cuentas (personal, de otra empresa, de otro tenant),
   > elige con cuidado. La cuenta que elijas determina **qué contenido de Fabric
   > vas a poder ver**.

2. Completa el **MFA** si te lo pide.

> 💡 Si el navegador no se abre solo, la terminal muestra una URL. Cópiala y
> pégala a mano.

---

## Paso 3 · Leer la pantalla de consentimiento

Aparece una pantalla que enumera los permisos que estás autorizando:

```
¿Está intentando iniciar sesión en Fabric IQ - Claude Code?

Esta aplicación podrá:
  • Leer todos los items                      (Item.Read.All)
  • Ejecutar operaciones sobre los items      (Item.Execute.All)
  • Ver todos los conjuntos de datos          (Dataset.Read.All)
```

**Léela. No es un trámite.**

Deberías ver **exactamente esos tres permisos**, y nada más.

> ⭐ **Mira los verbos: `Read`, `Execute`, `Read`.** Ningún `Write`, ningún
> `ReadWrite`, ningún `Manage`.
>
> Y fíjate en algo más fino: **no es que hayamos elegido pedir poco.** Fabric IQ
> pide exactamente lo que necesita para leer y ejecutar consultas, porque no hace
> nada más. Servidor y permisos están alineados.
>
> Cuando evalúes cualquier integración de IA en tu trabajo, **esta pantalla es el
> mejor chequeo de cordura que tienes**: si una herramienta que dice "solo
> consultar" te pide permisos de escritura, algo no cuadra.

Presiona **Aceptar**.

> 📌 Estos tres permisos **no requieren consentimiento de administrador por
> defecto**: puedes consentirlos tú. Pero si tu tenant restringe el consentimiento
> de usuarios, verás una pantalla pidiendo aprobación de un admin. No es un error
> tuyo: es la política de tu empresa, y alguien de TI debe aprobarlo una vez.

---

## Paso 4 · Volver a la terminal

El navegador muestra una página de confirmación y puedes cerrarla. Vuelve a
Claude Code.

### Resultado esperado

```
fabric-iq    ✔ connected    (6 tools)
```

Si sigue diciendo `needs authentication`, espera unos segundos y escribe `/mcp`
de nuevo.

---

## Paso 5 · Verificar qué herramientas tienes

### Prompt sugerido

```
Usando el MCP fabric-iq, lista las herramientas que tienes disponibles.
Para cada una dime, en español y en una línea, qué hace y qué necesita
como entrada. Después indícame explícitamente si alguna de ellas puede
crear, modificar o eliminar algo en Fabric.
```

### Resultado esperado

Las seis herramientas de Fabric IQ:

| Herramienta | Qué hace |
|---|---|
| `DiscoverArtifacts` | Busca reportes y semantic models por nombre |
| `ResolveFabricItem` | Resuelve la URL de un item de Fabric |
| `GetSemanticModelSchema` | Esquema del modelo: tablas, columnas, medidas, relaciones |
| `GetReportMetadata` | Metadatos del reporte: páginas, visuales, filtros |
| `ValueSearch` | Busca valores almacenados dentro de un modelo |
| `ExecuteQuery` | Ejecuta una consulta DAX contra un semantic model |

Y la conclusión: **ninguna modifica nada.** Las seis leen, buscan o calculan.

> 💡 Si las herramientas que ves no coinciden con estas seis, revisa el header
> `X-Variants` (debe ser `Fabric.Routing.FabricIQ.V1`) y reinicia la conexión.

---

## Paso 6 · Comprobar que el "solo lectura" es real

### Prompt sugerido

```
¿Podrías crear un workspace nuevo, publicar un reporte o modificar una
medida en Fabric usando el MCP fabric-iq? Responde solo en base a las
herramientas que realmente tienes disponibles.
```

### Resultado esperado

Claude responde que **no**: las seis herramientas son de consumo y no existe
ninguna operación de creación, modificación o administración. Puede mencionar
que para eso existen otros servidores MCP de Fabric, que este taller no usa.

> ⭐ **Compara esto con el bloque 2.** Allá el servidor local no podía escribir
> porque quien lo programó no incluyó esas funciones. Acá **Microsoft separó
> deliberadamente el consumo de la administración en servidores distintos**.
>
> Es la misma idea, aplicada por una empresa que tiene que dar garantías a
> clientes corporativos: **cuando una herramienta es incapaz de hacer daño, no
> necesitas confiar en que nadie apruebe apurado.**

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

Claude Code **adquiere, almacena, refresca y envía el token automáticamente**.
En condiciones normales no tienes que hacer nada durante todo el taller.

| Situación | Qué hacer |
|---|---|
| Uso normal | Nada. Se refresca solo. |
| Aparece un error de autenticación | `/mcp` y vuelve a autenticarte. |
| Cambiaste de cuenta | `claude mcp logout fabric-iq` y vuelve a entrar. |

> ⚠️ **No agregues una cabecera `Authorization` con un token escrito a mano.**
> Microsoft advierte que una cabecera `Authorization` configurada **anula** el
> flujo OAuth interactivo y la gestión automática de tokens: cuando ese token
> expire, la conexión queda rota hasta que quites la cabecera.

---

## Problemas frecuentes

| Síntoma | Qué significa | Solución |
|---|---|---|
| El navegador no se abre | El cliente necesita el registro de app. | Confirma que pasaste `--client-id` al registrar el servidor. |
| `AADSTS50011` | El redirect URI no coincide. | Debe ser exactamente `http://localhost:8080/callback` y `--callback-port 8080`. |
| Falla la autenticación | Faltan permisos, o el tenant restringe el consentimiento. | Verifica los tres permisos delegados. Si pide aprobación de admin, esa es la política de tu tenant. |
| `AADSTS50020` | Tu cuenta no pertenece a ese tenant. | Elegiste la cuenta equivocada al iniciar sesión. |
| `AADSTS50076` | Se requiere MFA. | Completa el segundo factor. |
| `AADSTS53003` | Política de acceso condicional. | Consulta con TI; a veces basta con estar en la red corporativa. |
| `AADSTS700016` | El client ID no existe en el tenant. | Revisa que copiaste el *Application (client) ID* correcto. |
| El navegador se abre pero queda cargando | El puerto 8080 está ocupado. | Usa otro puerto: regístralo en Entra ID y vuelve a agregar el servidor con ese `--callback-port`. |
| Las herramientas no son las seis documentadas | Header `X-Variants` ausente o incorrecto. | Debe ser `Fabric.Routing.FabricIQ.V1`. Reinicia la conexión. |

> 🤖 Copia el código `AADSTS` completo y pégalo en Claude Code:
> *"¿Qué significa este error de Entra ID y qué tengo que hacer?"*.

---

## ✅ Checklist

- [ ] `/mcp` muestra `fabric-iq` como `connected`.
- [ ] **Leíste la pantalla de consentimiento** y viste los tres permisos de lectura.
- [ ] Viste las seis herramientas disponibles.
- [ ] Confirmaste que ninguna puede modificar nada en Fabric.
- [ ] Entendiste que tu contraseña nunca pasó por la terminal.
- [ ] Sabes qué es el device code flow y por qué no ingresarías uno que no pediste.

---

⬅️ **Anterior:** [03 · README del bloque](../README.md)
➡️ **Siguiente:** [Ejercicio 2 · Explorar y consultar en Fabric](./02-consultar-workspace-fabric.md)

# Ejercicio 1 · Autenticación con Entra ID (device code)

⏱️ **Tiempo estimado:** 12 minutos
📍 **Bloque:** [03 · MCP Fabric / API](../README.md)

---

## Objetivo

Completar el flujo **device code** para que Claude Code pueda consultar tu
Power BI Service / Fabric **con tu propia identidad y tus propios permisos**,
sin que tu contraseña pase nunca por la terminal.

---

## Antes de empezar

- [ ] Registraste el servidor `powerbi-fabric` (ver [README del bloque](../README.md#configuración)).
- [ ] Tienes tu navegador abierto y con sesión iniciada en <https://app.powerbi.com>.
- [ ] Reiniciaste Claude Code después de registrar el servidor.

---

## Cómo funciona el device code, en un dibujo

```
  ┌─────────────────┐                        ┌──────────────────┐
  │  Claude Code    │                        │    Microsoft     │
  │   (terminal)    │                        │    Entra ID      │
  └────────┬────────┘                        └────────┬─────────┘
           │                                          │
           │  1. "Necesito autenticar a alguien"      │
           ├─────────────────────────────────────────►│
           │                                          │
           │  2. "Dale este código: A1B2-C3D4"        │
           │◄─────────────────────────────────────────┤
           │                                          │
   3. Te muestra el código y la URL                   │
           │                                          │
           │                                          │
  ┌────────▼────────┐  4. Abres la URL, pegas el      │
  │   TU NAVEGADOR  │     código, confirmas con tu    │
  │  (ya logueado)  ├─────────────────────────────────►
  └─────────────────┘     cuenta corporativa          │
           │                                          │
           │  5. "Listo, aquí está el token"          │
           │◄─────────────────────────────────────────┤
           │                                          │
   6. Claude Code ya puede consultar el Service       │
```

**Lo importante del paso 4:** tu contraseña (y tu MFA) se manejan **solo en el
navegador, en el dominio de Microsoft**. Claude Code recibe al final un **token
temporal**, nunca tu credencial.

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

Selecciona `powerbi-fabric` con las flechas y presiona `Enter`. Elige la opción
de **autenticar / conectar**.

> 💡 Según la configuración, Claude Code puede abrir directamente una ventana del
> navegador (flujo OAuth estándar) en vez de mostrarte un código. **Si el
> navegador se abre solo y te pide iniciar sesión, salta al Paso 4.** El
> resultado es el mismo; lo que sigue describe la variante con código, que es la
> que verás en entornos donde el navegador no se puede abrir automáticamente.

---

## Paso 2 · Leer el código que aparece en la terminal

### Resultado esperado

Un mensaje parecido a este:

```
Para iniciar sesión, abre https://microsoft.com/devicelogin en un navegador
e ingresa el código  A1B2-C3D4  para autenticarte.
```

Fíjate en dos cosas:

| Elemento | Qué es |
|---|---|
| `https://microsoft.com/devicelogin` | La página oficial de Microsoft donde se ingresa el código. **Verifica que el dominio sea `microsoft.com`.** |
| `A1B2-C3D4` | Tu código de un solo uso. **Expira en unos 15 minutos.** |

> 🔐 **Precaución de seguridad real:** el flujo device code es también una técnica
> conocida de phishing. Si alguna vez recibes un código de este tipo **por correo
> o por chat, sin haberlo pedido tú**, no lo ingreses. En este ejercicio es
> legítimo porque **tú acabas de iniciar el flujo** desde tu propia terminal.

---

## Paso 3 · Abrir la página e ingresar el código

1. **Copia el código** desde la terminal (selecciónalo y `Ctrl+C`).
2. Abre <https://microsoft.com/devicelogin> en tu navegador.
3. Pega el código en el campo **Código** y presiona **Siguiente**.

### Resultado esperado

La página te pide confirmar con qué cuenta quieres continuar.

---

## Paso 4 · Elegir tu cuenta y autorizar

1. **Elige tu cuenta corporativa** — la misma con la que entras a Power BI Service.

   > ⚠️ Si aparecen varias cuentas (personal, de otra empresa, de otro tenant),
   > elige con cuidado. La cuenta que elijas determina **qué workspaces vas a
   > ver**.

2. Si te lo pide, completa el **MFA** (Authenticator, SMS, o el método de tu
   empresa).

3. Aparece una pantalla de **consentimiento** que dice qué permisos está pidiendo
   la aplicación. Algo como:

   ```
   ¿Está intentando iniciar sesión en <nombre de la aplicación>?

   Esta aplicación podrá:
     • Leer todos los conjuntos de datos
     • Leer todos los grupos (workspaces)
     ...
   ```

   **Léela.** No es un trámite: es la lista exacta de lo que estás autorizando.
   Para este taller deberían ser permisos de **lectura** (`.Read.All`).
   Si ves permisos de escritura (`.ReadWrite.All`) y no los esperabas, **cancela
   y consulta** antes de continuar.

4. Presiona **Aceptar** / **Continuar**.

### Resultado esperado

El navegador muestra:

```
Ha iniciado sesión en la aplicación <nombre> en el dispositivo.
Ya puede cerrar esta ventana.
```

---

## Paso 5 · Volver a la terminal

Vuelve a la ventana de Claude Code. Debería haber detectado la autorización sola,
en pocos segundos.

### Resultado esperado

```
powerbi-fabric    ✔ connected    (N tools)
```

Si sigue diciendo `needs authentication`, espera unos segundos más y vuelve a
escribir `/mcp`.

---

## Paso 6 · Verificar que quedaste autenticado como quien corresponde

Antes de consultar datos, confirma **con qué identidad** estás conectado.

### Prompt sugerido

```
Usando el MCP powerbi-fabric, dime con qué cuenta estoy autenticado
y qué herramientas tienes disponibles. Lista cada herramienta con una
explicación en una línea en español, e indícame cuáles son de lectura
y cuáles podrían modificar algo.
```

### Resultado esperado

1. La cuenta con la que quedaste autenticado (o, si el servidor no la expone,
   Claude te lo dirá en vez de inventarla).
2. Una lista de herramientas con nombres del estilo `list_workspaces`,
   `list_datasets`, `execute_dax`, `get_dataset_metadata`.
3. Una clasificación clara entre herramientas de consulta y, si las hubiera, de
   escritura.

> ⭐ **Punto de atención del taller.** A diferencia del MCP local —que
> *literalmente no puede* escribir— este servidor podría exponer operaciones de
> escritura. Dos reglas para el resto del bloque:
>
> 1. **Nos quedamos en modo consulta.** No ejecutamos nada que cree, modifique
>    o elimine en el Service.
> 2. **Lee el cuadro de permisos antes de aprobar.** Acá sí importa: tus permisos
>    de Entra ID son reales y el entorno es compartido.
>
> Tu red de seguridad de fondo sigue siendo Entra ID: no puedes hacer nada que tu
> cuenta no pudiera hacer ya desde el navegador.

---

## Sobre la duración del token

El token de acceso **expira**, típicamente en una hora.

| Situación | Qué pasa | Qué hacer |
|---|---|---|
| Durante el taller | Probablemente no expire. | Nada. |
| Se acabó el token | Aparece `401 Unauthorized`. | Escribe `/mcp` y repite el flujo. Es rápido. |
| Cambiaste de cuenta | Sigues viendo los workspaces de la anterior. | Desconecta el servidor en `/mcp` y vuelve a autenticarte. |

---

## Problemas frecuentes

| Código / síntoma | Qué significa | Solución |
|---|---|---|
| `AADSTS70016` | El código aún no fue ingresado. | Completa el paso del navegador. |
| `AADSTS70019` / *expired* | El código expiró (pasaron más de ~15 min). | Escribe `/mcp` y genera uno nuevo. |
| `AADSTS50020` | Tu cuenta no pertenece a ese tenant. | Elegiste la cuenta equivocada, o el `TU_TENANT_ID` está mal. |
| `AADSTS65001` / *consent required* | Falta el consentimiento del administrador. | Un admin de Entra ID debe otorgarlo. Es el bloqueo más común en empresas. |
| `AADSTS50076` | Se requiere MFA. | Completa el segundo factor. |
| `AADSTS53003` | Bloqueado por una política de acceso condicional. | Política de tu empresa. Consulta con TI; a veces basta con estar en la red corporativa o en un equipo administrado. |
| El navegador pide credenciales de nuevo | Sesión distinta o ventana de incógnito. | Inicia sesión normalmente. |

> 🤖 Copia el código `AADSTS` completo y pégalo en Claude Code:
> *"¿Qué significa este error de Entra ID y qué tengo que hacer?"*.

---

## ✅ Checklist

- [ ] `/mcp` muestra `powerbi-fabric` como `connected`.
- [ ] Sabes con qué cuenta quedaste autenticado.
- [ ] Viste la lista de herramientas disponibles.
- [ ] Entendiste que tu contraseña nunca pasó por la terminal.
- [ ] Tienes claro que trabajamos en **modo consulta** en este bloque.

---

⬅️ **Anterior:** [03 · README del bloque](../README.md)
➡️ **Siguiente:** [Ejercicio 2 · Consultar workspace de Fabric](./02-consultar-workspace-fabric.md)

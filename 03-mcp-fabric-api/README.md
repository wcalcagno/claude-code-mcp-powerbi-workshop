# 03 · MCP para Power BI Service y Fabric — Fabric IQ

⏱️ **Duración del bloque:** 35 minutos
📍 **Requisito:** haber completado [02 · MCP Power BI local](../02-mcp-powerbi-local/README.md)

---

## Qué cambia respecto al bloque anterior

En el bloque 2 conectamos Claude Code a **tu computador**. Ahora lo conectamos a
**la nube**.

| | **Bloque 2 · MCP local** | **Bloque 3 · Fabric IQ (este)** |
|---|---|---|
| Qué consulta | El modelo abierto en **Power BI Desktop** | Los reportes y semantic models publicados en **Fabric / Power BI Service** |
| Dónde corre el servidor | En tu PC (proceso Python) | En la infraestructura de **Microsoft** |
| Cómo se conecta | Puerto local XMLA / ADOMD.NET | **MCP sobre HTTPS** (Streamable HTTP) |
| Credenciales | Ninguna (es local) | **Entra ID** (tu cuenta corporativa) |
| Qué alcanza | Un solo `.pbix`, el que tengas abierto | Todo lo que **tú** puedas ver en Fabric |
| Requiere Power BI Desktop abierto | **Sí, obligatorio** | No |
| **Permisos** | **Solo lectura** | **Solo lectura** |
| Madurez | Proyecto comunitario | **Oficial de Microsoft, disponible de forma general (GA)** |

**Lo que se mantiene, y es el hilo conductor del taller:** en ambos bloques
trabajamos con servidores que **leen y no escriben**. En el bloque 2 por elección
entre alternativas de la comunidad; acá porque **Microsoft diseñó Fabric IQ
explícitamente así**.

---

## Qué es Fabric IQ

**Fabric IQ MCP** es el servidor MCP remoto oficial de Microsoft para consumir
datos de Power BI. Conecta agentes de IA — como Claude Code — a tus **reportes y
semantic models** de Microsoft Fabric.

La documentación de Microsoft lo define como un servidor que expone
**un conjunto de herramientas de solo lectura** para que un agente encuentre
contenido de Power BI, inspeccione metadatos de reportes y modelos, ubique
valores y ejecute consultas DAX sobre datos que el usuario autenticado ya puede
ver.

### El endpoint oficial

```text
https://fabriciq.svc.cloud.microsoft/v1/mcp/fabriciq
```

| Situación | Endpoint |
|---|---|
| Normal | `https://fabriciq.svc.cloud.microsoft/v1/mcp/fabriciq` |
| Si tu organización usa **private links** | `https://api.fabric.microsoft.com/v1/mcp/fabriciq` |

> 📌 **Si la conexión falla con un 404**, prueba con la variante en mayúsculas
> (`.../v1/mcp/FabricIQ`): la documentación de Microsoft muestra ambas grafías en
> páginas distintas, y las rutas suelen distinguir mayúsculas. El valor de arriba
> es el de la página dedicada a Fabric IQ, que es la más específica y reciente.

---

## ⭐ Punto clave: Fabric IQ también es de solo lectura

Igual que en el bloque 2, y ahora por diseño oficial de Microsoft:

Fabric IQ **puede**:

- ✅ Buscar reportes y semantic models por nombre.
- ✅ Leer el esquema de un modelo: tablas, columnas, medidas, relaciones.
- ✅ Leer la estructura de un reporte: páginas, visuales, filtros.
- ✅ Buscar valores almacenados dentro de un modelo.
- ✅ Ejecutar consultas DAX y devolver resultados.

Fabric IQ **no puede**:

- ❌ Crear, modificar ni eliminar semantic models o reportes.
- ❌ Administrar workspaces.
- ❌ Crear items de ningún tipo en Fabric.
- ❌ Acceder a nada que tu cuenta no pueda ver ya.

Microsoft lo dice sin ambigüedad: *Fabric IQ admite escenarios de consumo de solo
lectura; no ofrece administración de workspaces, desarrollo ni creación de items.*
Para esas operaciones existen **otros** servidores MCP de Fabric — que este taller
no usa, deliberadamente.

### Por qué esto cierra bien el taller

En el bloque 2 elegimos un servidor de solo lectura entre alternativas que sí
escribían. Acá **el propio Microsoft separó las responsabilidades** en servidores
distintos: uno que consume y otros que administran y crean.

> 💬 **Esa separación es la lección para llevarse.** No es una limitación de
> producto: es el principio de menor privilegio aplicado al diseño. Cuando
> evalúes herramientas de IA para tu área, la pregunta correcta no es *"¿qué
> puede hacer?"* sino *"¿qué es incapaz de hacer, aunque alguien apruebe sin
> leer?"*.

---

## Las seis herramientas

| Herramienta | Qué hace |
|---|---|
| `DiscoverArtifacts` | Busca reportes y semantic models **por nombre**. |
| `ResolveFabricItem` | Resuelve la URL de un item de Fabric para que las demás herramientas la usen. |
| `GetSemanticModelSchema` | Devuelve el esquema del modelo: tablas, columnas, medidas, relaciones. |
| `GetReportMetadata` | Devuelve los metadatos de un reporte: páginas, visuales, filtros. |
| `ValueSearch` | Busca valores concretos almacenados dentro de un modelo. |
| `ExecuteQuery` | Ejecuta una consulta DAX contra un semantic model. |

> 💡 **Las seis leen. Ninguna escribe.**

Dos detalles prácticos:

- **No necesitas IDs para partir.** Basta el **nombre** de un reporte o modelo al
  que tengas acceso, o pegar su **URL del navegador**. Fabric IQ resuelve los
  identificadores internos por su cuenta. (No uses un *share link*: usa la URL de
  la barra de direcciones.)
- **Fabric IQ no responde preguntas en lenguaje natural por sí solo.** No tiene
  una herramienta de "respóndeme esto". Es **Claude Code** quien elige las
  herramientas, compone el DAX e interpreta el resultado. Fabric IQ entrega el
  acceso autenticado y ejecuta.

---

## Autenticación: Entra ID, identidad delegada

Claude Code actúa **en tu nombre**, con **tu identidad y tus permisos**.

| | **Delegada (la única posible)** | **Service principal** |
|---|---|---|
| Quién es | Tú, la persona | Una aplicación |
| Permisos | Exactamente los tuyos | Los que un admin le otorgue |
| **RLS y OLS** | **Se aplican** | — |
| Soportado por Fabric IQ | ✅ Sí | ❌ **No soportado** |

> 🔐 **Fabric IQ no admite autenticación de service principal ni *application-only*.**
> No es una opción que descartamos: es una que no existe. Cada persona usa su
> propia identidad, y **Row-Level Security y Object-Level Security siguen
> aplicándose** exactamente igual que en el navegador.
>
> Dos personas pueden ejecutar el mismo prompt sobre el mismo modelo y obtener
> resultados distintos. Eso no es un error: es tu RLS funcionando.

---

## Prerrequisitos de este bloque

Buenas noticias respecto de otras alternativas: **Fabric IQ es GA y pide bastante
menos**.

| Requisito | Detalle |
|---|---|
| **Cuenta de trabajo o educación de Entra ID** | La misma con la que entras a Power BI Service. |
| **Acceso a un reporte o semantic model** | Cualquiera que ya puedas abrir. |
| **Región del tenant compatible** | El *home region* de tu tenant debe soportar todas las cargas de trabajo de Fabric. |
| **Registro de app en Entra ID** | Necesario para Claude Code. Ver Paso 1. |

**Lo que NO necesitas** — y conviene subrayarlo:

- ❌ **No** necesitas permiso **Build** sobre el semantic model.
- ❌ **No** necesitas rol en el workspace.
- ❌ **No** necesitas el ID del workspace, del reporte ni del modelo.
- ❌ **No** necesitas capacidad Fabric ni Premium: los modelos pueden estar en
  capacidad compartida.
- ❌ **No** requiere un tenant setting especial de preview.

> 🟡 **Lo único que puede necesitar a tu área de TI:** los permisos delegados
> `Item.Read.All`, `Item.Execute.All` y `Dataset.Read.All` **no exigen
> consentimiento de administrador por defecto** — tú puedes consentirlos. Pero un
> administrador puede haber restringido el consentimiento de usuarios en tu
> tenant. Si te aparece una pantalla pidiendo aprobación de un admin, eso es lo
> que pasó.

### Dónde NO funciona

| Limitación | Detalle |
|---|---|
| Regiones | No está disponible en regiones *Power BI-only* ni en nubes soberanas. |
| Tipos de contenido | Solo **reportes y semantic models**. No dashboards, no reportes paginados (RDL), no apps de Power BI. |
| Ontologías y data agents de Fabric | No soportados todavía. |
| Consultas entre modelos | Cada `ExecuteQuery` apunta a **un** semantic model. No hace joins entre modelos. |

### Verifica tu acceso antes de empezar

1. Abre <https://app.powerbi.com>.
2. Confirma que puedes abrir al menos un **reporte** o **semantic model**.
3. **Anota su nombre exacto.** Es lo único que necesitas.

> 💡 **¿Sin contenido publicado?** Publica en "Mi área de trabajo" el `.pbix` que
> usaste en el bloque 2. Así el
> [ejercicio integrador del bloque 4](../04-flujo-completo/README.md) compara
> **el mismo modelo** en local y en la nube, que es el escenario ideal.

---

## Configuración

### Paso 1 · Registrar una aplicación en Entra ID

Claude Code necesita un **client ID** propio. La razón técnica: Entra ID no
soporta *dynamic client registration*, así que un cliente MCP externo no puede
auto-registrarse — hay que darle un ID de aplicación creado a mano.

1. Entra a <https://entra.microsoft.com> con tu cuenta de trabajo.
2. **App registrations** → **New registration**.
3. Completa:

   | Campo | Valor |
   |---|---|
   | **Name** | `Fabric IQ - Claude Code` |
   | **Supported account types** | *Accounts in this organizational directory only (Single tenant)* |
   | **Redirect URI** | Déjalo vacío por ahora |

4. **Register**.
5. En **Overview**, copia el **Application (client) ID** → es tu `TU_CLIENT_ID`.

### Paso 2 · Configurar el redirect URI

Claude Code completa el login abriendo un servidor local temporal y esperando ahí
la respuesta de Microsoft. Por eso el redirect apunta a `localhost`.

1. En tu app → **Authentication** → **Add a platform** → **Mobile and desktop
   applications**.
2. Agrega exactamente:

   ```
   http://localhost:8080/callback
   ```

3. **Configure** para guardar.

> 📌 **El puerto tiene que coincidir.** Usamos `8080` porque en el Paso 4 lo
> fijamos con `--callback-port 8080`. Si no lo fijas, Claude Code usa un puerto al
> azar y el login falla con `AADSTS50011`, porque el redirect registrado no
> coincidiría.

### Paso 3 · Agregar los permisos delegados

1. En tu app → **API permissions** → **Add a permission**.
2. **APIs my organization uses** → busca **Power BI Service**.
3. **Delegated permissions**, y agrega **los tres**:

   | Permiso | Para qué |
   |---|---|
   | `Item.Read.All` | Leer los items de Fabric a los que tienes acceso |
   | `Item.Execute.All` | Ejecutar operaciones sobre esos items |
   | `Dataset.Read.All` | Leer los semantic models a los que tienes acceso |

4. **Add permissions**.

> ⭐ **Mira los tres verbos: `Read`, `Execute`, `Read`.** Ningún `Write`, ningún
> `ReadWrite`. No es que hayamos elegido no pedirlos: **Fabric IQ no los usa,
> porque no hace nada que los necesite.** El servidor y sus permisos están
> alineados, y eso es exactamente lo que uno quiere ver al evaluar una
> integración.

> 📌 Estos tres permisos **no requieren consentimiento de administrador por
> defecto**. Si tu tenant restringe el consentimiento de usuarios, un admin
> tendrá que aprobarlos una vez.

### Paso 4 · Registrar el servidor en Claude Code

Desde PowerShell, en la carpeta del workshop:

```powershell
claude mcp add --transport http --client-id TU_CLIENT_ID --callback-port 8080 -H "X-Variants: Fabric.Routing.FabricIQ.V1" fabric-iq https://fabriciq.svc.cloud.microsoft/v1/mcp/fabriciq
```

| Parte | Significado |
|---|---|
| `--transport http` | El servidor es remoto, se habla por HTTPS. No se levanta nada en tu PC. |
| `--client-id TU_CLIENT_ID` | El *Application (client) ID* del Paso 1. |
| `--callback-port 8080` | Fija el puerto de retorno del login. **Debe coincidir con el redirect URI registrado.** |
| `-H "X-Variants: ..."` | Fija la versión del contrato de herramientas (ver abajo). |
| `fabric-iq` | El nombre que verás al escribir `/mcp`. |
| `https://fabriciq...` | El endpoint oficial. |

> 📌 **Sin `--client-secret`.** Es una aplicación de cliente público (de
> escritorio): no maneja secretos. La seguridad la dan el redirect URI registrado
> y PKCE, no un secreto compartido.

#### Sobre el header `X-Variants`

Fabric IQ usa ese header para elegir **la versión del contrato de herramientas**
(sus nombres, esquemas de entrada y comportamiento). La versión pública actual es
`Fabric.Routing.FabricIQ.V1`.

Fijarlo es opcional pero **recomendable para un taller**: garantiza que todos
vean las mismas seis herramientas aunque Microsoft publique una versión nueva
como predeterminada. Si algún día las herramientas no calzan con las
documentadas, revisa este header primero.

> ⚠️ El `v1` de la URL **no** es la versión del contrato: es parte de la
> dirección del endpoint. Son dos cosas distintas.

**Alternativa:** editar el archivo de configuración a mano, usando el ejemplo de
este repositorio → 📄 **[`mcp-config-fabric-ejemplo.json`](./mcp-config-fabric-ejemplo.json)**

### Paso 5 · Autenticarte

1. Sal de Claude Code (`/exit`) y vuelve a entrar (`claude`).
2. Escribe `/mcp`. Verás `fabric-iq` como **needs authentication**.
3. Sigue el
   [Ejercicio 1 · Autenticación Entra ID](./ejercicios/01-autenticacion-entra-id.md).

> 💡 También puedes autenticarte sin abrir una sesión:
> ```powershell
> claude mcp login fabric-iq
> ```

---

## Un paréntesis útil: Skills for Fabric

Microsoft publica una **Fabric IQ skill** que le enseña al agente cómo combinar
estas seis herramientas para responder preguntas de negocio. Está pensada para
**GitHub Copilot CLI** y se instala desde su marketplace de plugins, así que no
la usamos en este taller.

> 💬 **Pero fíjate en la idea, porque ya la aplicaste:** el servidor MCP entrega
> el **acceso**; algo aparte entrega el **criterio** de cómo usarlo. En Copilot
> CLI eso es la skill. **En Claude Code, ese papel lo cumple el archivo
> [`CLAUDE.md`](../CLAUDE.md) de este repositorio** — las instrucciones de
> proyecto que Claude lee al iniciar.
>
> Escribir un buen `CLAUDE.md` con las convenciones DAX y las reglas de negocio de
> tu equipo es, probablemente, lo que más rendimiento te va a dar cuando vuelvas
> al trabajo.

---

## Ejercicios de este bloque

| # | Ejercicio | Qué practicas |
|---|---|---|
| 1 | [Autenticación Entra ID](./ejercicios/01-autenticacion-entra-id.md) | El flujo de sign-in, paso a paso |
| 2 | [Explorar y consultar en Fabric](./ejercicios/02-consultar-workspace-fabric.md) | Descubrir contenido, leer esquemas y ejecutar DAX en la nube |

---

## Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| El navegador no se abre para el login | El cliente necesita el registro de app. | Confirma que pasaste `--client-id`. Revisa el Paso 1. |
| `AADSTS50011` — *redirect URI mismatch* | El puerto no coincide. | Debe ser **exactamente** `http://localhost:8080/callback` y `--callback-port 8080`. |
| Falla la autenticación | Faltan permisos, o el tenant restringe el consentimiento. | Verifica que agregaste `Item.Read.All`, `Item.Execute.All` y `Dataset.Read.All`. Si pide aprobación, un admin debe otorgarla. |
| `404` al conectar | Grafía del endpoint, o private links. | Prueba `.../v1/mcp/FabricIQ` (mayúsculas) o, con private links, `https://api.fabric.microsoft.com/v1/mcp/fabriciq`. |
| `/mcp` no muestra las herramientas esperadas | Conexión obsoleta, o el header `X-Variants` está mal. | Reinicia Claude Code. Verifica que el header sea `Fabric.Routing.FabricIQ.V1`. |
| No encuentra un reporte que sí existe | Nombre ambiguo, tipo no soportado, o sin acceso. | Usa un nombre más específico, o pega la **URL del navegador** (no un *share link*). Recuerda: no soporta dashboards, reportes paginados ni apps. |
| Error de permisos al consultar | Tu identidad no puede ver ese contenido, o RLS/OLS lo restringe. | Verifica en el navegador que puedes abrirlo. |
| Error de *rate limit* | Demasiadas solicitudes seguidas. | Espera unos segundos y reintenta. Reduce el tamaño de los resultados con filtros y agregaciones. |
| El resultado sale truncado | Consulta muy grande. | Los resultados grandes vuelven como CSV embebido. Usa agregaciones y filtros para achicarlos. |
| Los datos están desactualizados | El modelo no se ha refrescado. | Revisa el historial de actualización en el Service. **Es el tema del [bloque 4](../04-flujo-completo/README.md).** |
| No está disponible en mi tenant | Región no compatible. | Fabric IQ requiere un *home region* que soporte todas las cargas de Fabric. No está en regiones *Power BI-only* ni en nubes soberanas. |

> 🤖 Copia el mensaje de error completo y pégalo en Claude Code:
> *"¿Qué significa este error y cómo lo resuelvo?"*.

---

⬅️ **Anterior:** [02 · MCP Power BI local](../02-mcp-powerbi-local/README.md)
➡️ **Siguiente:** [Ejercicio 1 · Autenticación Entra ID](./ejercicios/01-autenticacion-entra-id.md)

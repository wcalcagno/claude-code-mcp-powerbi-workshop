# 03 · MCP para Power BI Service y Fabric (vía API)

⏱️ **Duración del bloque:** 35 minutos
📍 **Requisito:** haber completado [02 · MCP Power BI local](../02-mcp-powerbi-local/README.md)

---

## Qué cambia respecto al bloque anterior

En el bloque 2 conectamos Claude Code a **tu computador**. Ahora lo conectamos a
**la nube**.

| | **Bloque 2 · MCP local** | **Bloque 3 · MCP remoto (este)** |
|---|---|---|
| Qué consulta | El modelo abierto en **Power BI Desktop** | Los semantic models publicados en el **Service / Fabric** |
| Dónde corre el servidor | En tu PC (proceso Python) | En la infraestructura de **Microsoft** |
| Cómo se conecta | Puerto local XMLA / ADOMD.NET | **API REST** sobre HTTPS |
| Credenciales | Ninguna (es local) | **Entra ID** (tu cuenta corporativa) |
| Qué alcanza | Un solo `.pbix`, el que tengas abierto | Los semantic models a los que **tú** tengas acceso |
| Requiere Power BI Desktop abierto | **Sí, obligatorio** | No |
| Modo de trabajo | **Solo lectura** | **Consulta** (no escritura) |
| Madurez | Proyecto comunitario | **Oficial de Microsoft, en preview** |

**Lo que se mantiene:** en ambos casos trabajamos **leyendo, no escribiendo**.

---

## El endpoint oficial

Microsoft publica **dos servidores MCP hospedados** para Power BI. La diferencia
entre ellos es exactamente el punto pedagógico de este taller:

| Servidor | Endpoint | Qué hace |
|---|---|---|
| **Power BI Consumption MCP server** ⬅️ **el que usamos** | `https://api.fabric.microsoft.com/v1/mcp/powerbi` | **Consulta.** Lee el esquema de un semantic model y ejecuta DAX. |
| Power BI Authoring MCP server — **NO lo usamos** | `https://api.fabric.microsoft.com/v1/mcp/powerbi/authoring` | **Lectura y escritura.** Crea, modifica y elimina tablas, columnas, medidas, relaciones y roles de seguridad. |

Existe además **Fabric IQ**, que Microsoft señala hoy como el camino preferente
para escenarios de consumo:

| Servidor | Endpoint |
|---|---|
| Fabric IQ MCP | `https://fabriciq.svc.cloud.microsoft/v1/mcp/FabricIQ` |

### Por qué el taller usa el endpoint de Consumption

La misma lógica del bloque 2, ahora aplicada a la nube: **elegimos deliberadamente
el servidor que no puede escribir.**

El servidor de *Authoring* existe, es oficial y es una herramienta legítima para
tu trabajo diario — con control de versiones, revisión de pares y un entorno de
desarrollo detrás. Pero pide el permiso `SemanticModel.ReadWrite.All`, y eso
significa que **una aprobación apurada en un salón de clases podría modificar un
modelo productivo**. No es el riesgo que queremos correr hoy.

> 📌 **Nota sobre el estado del producto.** Microsoft indica que, para consumo de
> semantic models, hoy conviene usar **Fabric IQ**, y describe el endpoint de
> Consumption como el endpoint de consulta *anterior*, que sigue documentado y en
> **preview**. Para este taller usamos Consumption porque su alcance —consultar un
> modelo y ejecutar DAX— es exactamente el del ejercicio, y su configuración es
> más simple. Si después del taller vas a montar algo que dure, **evalúa Fabric IQ**.
> Ver [recursos/enlaces.md](../recursos/enlaces.md).

> ⚠️ **Está en PREVIEW.** Las definiciones de las herramientas, los formatos de
> solicitud y los esquemas de respuesta **pueden cambiar**. No lo uses en procesos
> productivos todavía, y verifica la documentación oficial el día que hagas el taller.

---

## Qué sabe hacer el servidor de Consumption

Son **cuatro herramientas**, todas de consulta:

| Herramienta | Qué hace | Qué necesita |
|---|---|---|
| **Get Semantic Model Schema** | Devuelve tablas, columnas, medidas, relaciones, tipos de dato y jerarquías del modelo. | ID del semantic model |
| **Execute Query** | Ejecuta una consulta DAX contra el modelo y devuelve el resultado. | ID del semantic model + expresión DAX |
| **Get Report Metadata** | Devuelve la estructura de un reporte: páginas, visuales, campos usados y filtros. | ID del reporte |
| **Generate Query** | Genera DAX a partir de una pregunta en lenguaje natural, usando el motor de Copilot. | ID del modelo + la pregunta. **Requiere licencia de Copilot.** |

> 🔴 **Importante, y es distinto de lo que uno esperaría:** **no hay una
> herramienta para listar tus workspaces ni tus semantic models.** El servidor
> trabaja sobre **un modelo que tú identificas por su ID**. Ese ID lo sacas de la
> URL de Power BI Service — te mostramos cómo en el
> [ejercicio 2](./ejercicios/02-consultar-workspace-fabric.md).

> 💡 **Sobre Generate Query:** consume capacidad de Copilot. Si tu organización no
> tiene licencia, o prefieres no gastarla, simplemente no la uses: Claude Code
> escribe el DAX perfectamente bien por su cuenta, tal como lo hizo en el bloque 2.

---

## Autenticación: Entra ID, identidad delegada

**Entra ID** (antes Azure Active Directory) es el sistema de identidad de
Microsoft: la misma cuenta con la que entras a Power BI Service, Teams y Outlook.

### Delegada, no service principal

Claude Code actúa **en tu nombre**, con **tu identidad y tus permisos**.

| | **Delegada (la que usamos)** | **Service principal (la que NO usamos)** |
|---|---|---|
| Quién es | Tú, la persona | Una aplicación, sin persona detrás |
| Permisos | Exactamente los tuyos | Los que un administrador le otorgue |
| Quién queda en la auditoría | Tu nombre | El nombre de la app |
| **Row-Level Security (RLS)** | **Se aplica** | **No se aplica** ⚠️ |
| Requiere permisos de admin | Para el registro de la app, sí | Sí |

> 🔐 **Ese punto de RLS es crítico y está documentado por Microsoft:** cuando se
> usa un *service principal*, **Power BI no aplica RLS**, y la consulta puede ver
> todos los datos. Con identidad delegada, tu RLS se respeta igual que en el
> navegador. Es otra razón de peso para que en un taller cada persona use su
> propia identidad.

---

## Prerrequisitos de este bloque

Este bloque tiene **más fricción que el resto**, y conviene saberlo de antemano:
dos de los requisitos dependen de tu área de TI, no de ti.

| Requisito | Quién lo resuelve |
|---|---|
| **Tenant setting habilitado:** *"Users can use the Power BI Model Context Protocol server endpoint (preview)"* | 🔴 **Tu administrador de Power BI.** Sin esto, el endpoint no responde. |
| **Registro de aplicación en Entra ID** con los permisos delegados | 🟡 Tú, si puedes registrar apps; si no, tu administrador de Entra ID. |
| **Permiso *Build*** sobre al menos un semantic model | 🟡 El dueño del workspace. Ojo: el rol *Viewer* **no siempre basta**. |
| Licencia de Power BI (Pro, PPU o capacidad Fabric/Premium) | 🟢 Ya la tienes si usas Power BI Service. |
| Navegador con sesión iniciada en tu cuenta corporativa | 🟢 Tú. |

> 🔴 **Gestiona esto ANTES del taller.** Habilitar el tenant setting y registrar
> la app puede tomar días en una empresa grande. Si llegas al bloque 3 sin esto
> resuelto, no vas a poder completar los ejercicios en vivo — pero puedes leerlos,
> entender el flujo, y ejecutarlos después.

### Verifica tu acceso antes de empezar

1. Abre <https://app.powerbi.com>.
2. Entra a un workspace y abre un **semantic model** (antes llamado *dataset*).
3. Mira la URL. Debería verse así:

   ```
   https://app.powerbi.com/groups/{workspaceId}/datasets/{semanticModelId}
   ```

4. **Copia ese `semanticModelId`** a un bloc de notas. Es el dato que vas a usar
   en todos los ejercicios de este bloque.

> 💡 **¿Sin workspace propio?** Publica en "Mi área de trabajo" el `.pbix` que
> usaste en el bloque 2. Así el
> [ejercicio integrador del bloque 4](../04-flujo-completo/README.md) compara
> **el mismo modelo** en local y en la nube, que es el escenario ideal.

---

## Configuración

### Paso 1 · Registrar una aplicación en Entra ID

Claude Code necesita un **client ID** propio. La razón técnica: Entra ID no
soporta *dynamic client registration*, así que un cliente MCP externo no puede
auto-registrarse — hay que darle un ID de aplicación creado a mano.

1. Entra a <https://entra.microsoft.com> con una cuenta que pueda registrar apps.
2. **App registrations** → **New registration**.
3. Completa:

   | Campo | Valor |
   |---|---|
   | **Name** | `Power BI MCP - Claude Code` |
   | **Supported account types** | *Accounts in this organizational directory only (Single tenant)* |
   | **Redirect URI** | Déjalo vacío por ahora |

4. **Register**.
5. En **Overview**, copia el **Application (client) ID**. Es tu `TU_CLIENT_ID`.
   Copia también el **Directory (tenant) ID** — es tu `TU_TENANT_ID`.

### Paso 2 · Configurar el redirect URI

Claude Code completa el login abriendo un servidor local temporal y esperando la
respuesta de Microsoft ahí. Por eso el redirect URI apunta a `localhost`.

1. En tu app → **Authentication** → **Add a platform** → **Mobile and desktop
   applications**.
2. Agrega este redirect URI:

   ```
   http://localhost:8080/callback
   ```

3. **Configure** para guardar.

> 📌 **El puerto tiene que coincidir.** Usamos `8080` porque en el paso 4 vamos a
> fijarlo con `--callback-port 8080`. Si no lo fijas, Claude Code usa un puerto al
> azar y el login falla, porque el redirect URI registrado no coincidiría.
> **Cualquier diferencia entre ambos hace fallar el sign-in.**

### Paso 3 · Agregar los permisos delegados de Power BI

1. En tu app → **API permissions** → **Add a permission**.
2. **Microsoft APIs** → **Power BI Service**.
3. **Delegated permissions**, y agrega:

   | Permiso | Para qué | ¿Necesario acá? |
   |---|---|---|
   | `Dataset.Read.All` | Leer los semantic models a los que tienes acceso | ✅ Sí |
   | `Workspace.Read.All` | Leer los workspaces a los que tienes acceso | ✅ Sí |
   | `MLModel.Execute.All` | Ejecutar modelos de ML | ⬜ Opcional |
   | `SemanticModel.ReadWrite.All` | **Leer y escribir** semantic models | ❌ **NO lo agregues** |

   > ⭐ **Esa última fila es el taller en una línea.** Ese permiso es el que pide
   > el servidor de *Authoring*. No lo agregamos: si la aplicación nunca recibe
   > el permiso de escritura, no hay aprobación apurada que pueda modificar nada.
   > **El principio de menor privilegio, aplicado de verdad.**

4. **Add permissions**.
5. Si tu tenant lo exige, un administrador debe presionar **Grant admin consent**.
   Si no, se te pedirá consentimiento la primera vez que inicies sesión.

### Paso 4 · Registrar el servidor en Claude Code

Desde PowerShell, en la carpeta del workshop:

```powershell
claude mcp add --transport http --client-id TU_CLIENT_ID --callback-port 8080 powerbi-fabric https://api.fabric.microsoft.com/v1/mcp/powerbi
```

| Parte | Significado |
|---|---|
| `--transport http` | El servidor es remoto y se habla por HTTPS, no es un proceso local. |
| `--client-id TU_CLIENT_ID` | El *Application (client) ID* del paso 1. |
| `--callback-port 8080` | Fija el puerto de retorno del login. **Debe coincidir con el redirect URI registrado.** |
| `powerbi-fabric` | El nombre que verás al escribir `/mcp`. |
| `https://api.fabric.microsoft.com/v1/mcp/powerbi` | El endpoint oficial de Consumption. |

> 📌 **Sin `--client-secret`.** Es una aplicación de cliente público (una app de
> escritorio): no maneja secretos. La seguridad la da el redirect URI registrado
> más PKCE, no un secreto compartido.

**Alternativa:** editar el archivo de configuración a mano, usando el ejemplo de
este repositorio → 📄 **[`mcp-config-fabric-ejemplo.json`](./mcp-config-fabric-ejemplo.json)**

### Paso 5 · Autenticarte

1. Sal de Claude Code (`/exit`) y vuelve a entrar (`claude`).
2. Escribe `/mcp`. Verás `powerbi-fabric` como **needs authentication**.
3. Sigue el
   [Ejercicio 1 · Autenticación Entra ID](./ejercicios/01-autenticacion-entra-id.md).

> 💡 También puedes autenticarte sin abrir una sesión:
> ```powershell
> claude mcp login powerbi-fabric
> ```

---

## Ejercicios de este bloque

| # | Ejercicio | Qué practicas |
|---|---|---|
| 1 | [Autenticación Entra ID](./ejercicios/01-autenticacion-entra-id.md) | El flujo de sign-in, paso a paso |
| 2 | [Consultar un modelo del Service](./ejercicios/02-consultar-workspace-fabric.md) | Esquema del modelo y DAX contra la nube |

---

## Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| `/mcp` muestra `failed` de entrada | El **tenant setting** no está habilitado. | Es la causa #1. Tu admin de Power BI debe habilitar *"Users can use the Power BI Model Context Protocol server endpoint (preview)"*. |
| El login abre el navegador y falla al volver | El redirect URI no coincide. | Debe ser **exactamente** `http://localhost:8080/callback`, y el `--callback-port` debe ser `8080`. |
| `AADSTS50011` — *redirect URI mismatch* | Lo mismo de arriba. | Revisa el puerto en ambos lados. |
| `AADSTS65001` / *consent required* | Falta consentimiento del administrador. | Un admin de Entra ID debe otorgarlo en la app. |
| `AADSTS50020` | Tu cuenta no pertenece a ese tenant. | Elegiste la cuenta equivocada al iniciar sesión. |
| `AADSTS50076` | Se requiere MFA. | Completa el segundo factor. |
| `AADSTS53003` | Política de acceso condicional. | Consulta con TI; a veces basta con estar en la red corporativa. |
| `403 Forbidden` al consultar el modelo | Te falta permiso **Build** sobre el semantic model. | Rol *Viewer* no siempre alcanza. Pide *Build* al dueño del workspace. |
| `401 Unauthorized` después de un rato | El token expiró. | `/mcp` y vuelve a autenticarte. Claude Code normalmente lo refresca solo. |
| Claude "no encuentra" tus workspaces | **No existe** una herramienta para listarlos. | Entrega el **ID del semantic model**, sacado de la URL de Power BI Service. |
| `Generate Query` falla | Falta licencia de Copilot. | No la uses: pídele el DAX directamente a Claude Code. |
| La consulta al Service no coincide con la local | El modelo publicado está desactualizado. | Revisa la fecha del último refresco. **Es el tema del [bloque 4](../04-flujo-completo/README.md).** |

> 🤖 Los códigos `AADSTS` son de Entra ID. Pégalos completos en Claude Code:
> *"¿Qué significa este error y cómo lo resuelvo?"*.

---

⬅️ **Anterior:** [02 · MCP Power BI local](../02-mcp-powerbi-local/README.md)
➡️ **Siguiente:** [Ejercicio 1 · Autenticación Entra ID](./ejercicios/01-autenticacion-entra-id.md)

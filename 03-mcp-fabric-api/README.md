# 03 · MCP para Power BI Service y Fabric (vía API)

⏱️ **Duración del bloque:** 35 minutos
📍 **Requisito:** haber completado [02 · MCP Power BI local](../02-mcp-powerbi-local/README.md)

---

## Qué cambia respecto al bloque anterior

En el bloque 2 conectamos Claude Code a **tu computador**. Ahora lo conectamos a
**la nube**.

| | **Bloque 2 · MCP local** | **Bloque 3 · MCP remoto (este)** |
|---|---|---|
| Qué consulta | El modelo abierto en **Power BI Desktop** | Los workspaces del **Power BI Service / Fabric** |
| Dónde corre el servidor | En tu PC (proceso Python) | En la infraestructura de **Microsoft** |
| Cómo se conecta | Puerto local XMLA / ADOMD.NET | **API REST** sobre HTTPS |
| Credenciales | Ninguna (es local) | **Entra ID** (tu cuenta corporativa) |
| Qué alcanza | Un solo `.pbix`, el que tengas abierto | Todos los workspaces a los que **tú** tengas acceso |
| Requiere Power BI Desktop abierto | **Sí, obligatorio** | No |
| Modo de trabajo | **Solo lectura** | **Consulta** (no escritura) |
| Madurez | Proyecto comunitario | **Oficial de Microsoft, en preview** |

**Lo que se mantiene:** en ambos casos trabajamos **leyendo, no escribiendo**.
En el bloque 2 porque el servidor literalmente no sabe escribir. En este bloque
porque nos limitamos a operaciones de consulta, y porque tus permisos en Entra ID
son el límite real de lo que puedes hacer.

---

## El servidor: Microsoft Remote MCP Server para Power BI / Fabric

Microsoft publica un **servidor MCP remoto oficial** para Power BI y Microsoft
Fabric. No lo instalas: ya está corriendo en la nube de Microsoft. Tú solo
registras su dirección en Claude Code y te autenticas.

> ⚠️ **Está en PREVIEW.** Eso significa, concretamente:
> - El endpoint, el nombre de las herramientas y los permisos **pueden cambiar
>   sin aviso**.
> - Puede no estar habilitado en el tenant de tu empresa.
> - **No lo uses para procesos productivos** todavía.
> - **Verifica siempre la URL y los requisitos actuales en la documentación
>   oficial** antes de configurarlo: ver [recursos/enlaces.md](../recursos/enlaces.md).
>
> Por eso el archivo de configuración de este bloque usa un **placeholder** para
> el endpoint: el valor correcto es el que esté publicado en la documentación de
> Microsoft **el día que hagas el taller**, no el que aparezca escrito acá.

---

## Autenticación: Entra ID con device code

**Entra ID** (antes Azure Active Directory) es el sistema de identidad de
Microsoft. Es la misma cuenta con la que entras a Power BI Service, Teams y
Outlook en tu empresa.

### Autenticación delegada, no service principal

Vamos a usar autenticación **delegada**: Claude Code actúa **en tu nombre**,
usando **tu identidad y tus permisos**.

| | **Delegada (la que usamos)** | **Service principal (la que NO usamos)** |
|---|---|---|
| Quién es | Tú, la persona | Una aplicación, sin persona detrás |
| Permisos | Exactamente los tuyos | Los que un administrador le otorgue |
| Quién queda en la auditoría | Tu nombre | El nombre de la app |
| Requiere permisos de admin | No | **Sí** |

**Por qué elegimos delegada para el taller:**

1. **Cada asistente ve solo lo suyo.** Nadie accede a workspaces que no le
   corresponden. Los permisos que ya tienes en Power BI Service son exactamente
   los que vas a tener acá, ni uno más.
2. **No necesitamos molestar al área de TI** para registrar aplicaciones ni
   otorgar permisos de tenant antes del taller.
3. **La trazabilidad es correcta.** Si alguien revisa los logs de auditoría de
   Power BI, verá tu nombre en las consultas — que es la verdad.

### ¿Qué es el "device code flow"?

Es un método de inicio de sesión pensado para programas que **no pueden abrir
una ventana de navegador con un formulario propio** — como una herramienta de
terminal.

Funciona así:

1. La herramienta te muestra un **código corto** (ej. `A1B2-C3D4`) y una URL.
2. Tú abres esa URL **en tu navegador**, donde ya estás logueado con tu cuenta
   corporativa.
3. Pegas el código y confirmas.
4. La herramienta detecta que autorizaste y continúa.

> 🔐 **La ventaja de seguridad es importante:** tu contraseña **nunca pasa por la
> terminal ni por Claude Code**. La escribes — si acaso — solo en la página
> oficial de Microsoft, en tu navegador, con el candado de HTTPS a la vista.
> Claude Code recibe un token temporal, nunca tu credencial.

El paso a paso detallado está en el
[Ejercicio 1 · Autenticación Entra ID](./ejercicios/01-autenticacion-entra-id.md).

---

## Prerrequisitos de este bloque

| Requisito | Detalle |
|---|---|
| **Cuenta de Entra ID** | La cuenta corporativa con la que entras a Power BI Service. |
| **Acceso a un workspace** | De Power BI Service o Fabric, con rol **Viewer** como mínimo. Con **Member** o **Contributor** aprovechas más el ejercicio. |
| **Licencia de Power BI** | Pro, PPU, o un workspace en capacidad Fabric / Premium. |
| **Preview habilitado en tu tenant** | Puede requerir que un administrador lo habilite. Verifícalo antes del taller. |
| **Navegador con sesión iniciada** | Para completar el device code sin volver a escribir credenciales. |

### Verifica tu acceso antes de empezar

1. Abre <https://app.powerbi.com> en tu navegador.
2. Confirma que ves al menos un **workspace** además de "Mi área de trabajo".
3. Entra a ese workspace y confirma que hay al menos un **semantic model**
   (antes llamado *dataset*).
4. **Anota el nombre del workspace.** Lo vas a usar en los ejercicios.

> 💡 **¿Sin workspace propio?** "Mi área de trabajo" (*My workspace*) también
> sirve: publica ahí el `.pbix` que usaste en el bloque 2 y tendrás un modelo
> para consultar. Eso además hace que el
> [ejercicio integrador del bloque 4](../04-flujo-completo/README.md) compare el
> mismo modelo en local y en la nube, que es el escenario ideal.

---

## Configuración

### Paso 1 · Obtener los datos que necesitas

| Dato | Dónde sacarlo |
|---|---|
| **Endpoint del MCP remoto** | De la documentación oficial de Microsoft (está en preview, cópialo de ahí). Ver [recursos/enlaces.md](../recursos/enlaces.md). |
| **`TU_TENANT_ID`** | Power BI Service → ícono de ayuda `?` → **Acerca de Power BI**. Aparece como *Tenant ID* o *Tenant URL*. También en <https://portal.azure.com> → Microsoft Entra ID → *Overview* → *Tenant ID*. |
| **`TU_CLIENT_ID`** | El de la aplicación cliente que uses para autenticarte. Si Microsoft documenta un client ID público para el preview, usa ese. Si tu empresa registró una app propia, pídeselo a TI. |

> 📌 Un **tenant ID** y un **client ID** son identificadores con formato GUID:
> `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`. **No son secretos** — identifican, no
> autentican. Lo que sí es secreto es el **token** que recibes después de
> autenticarte, y ese nunca se escribe en un archivo de configuración.

### Paso 2 · Registrar el servidor en Claude Code

#### Opción A · Con `claude mcp add`

Desde PowerShell, en la carpeta del workshop:

```powershell
claude mcp add --transport http powerbi-fabric ENDPOINT_OFICIAL_MICROSOFT_MCP
```

Reemplaza `ENDPOINT_OFICIAL_MICROSOFT_MCP` por la URL que obtuviste de la
documentación.

| Parte | Significado |
|---|---|
| `--transport http` | El servidor es remoto y se habla por HTTP, no por proceso local. |
| `powerbi-fabric` | El nombre que verás al escribir `/mcp`. |
| `ENDPOINT_...` | La dirección del servidor de Microsoft. |

#### Opción B · Editando el archivo de configuración

Usa el ejemplo de este repositorio:

📄 **[`mcp-config-fabric-ejemplo.json`](./mcp-config-fabric-ejemplo.json)**

### Paso 3 · Reiniciar y autenticarte

1. Sal de Claude Code (`/exit`) y vuelve a entrar (`claude`).
2. Escribe `/mcp`.
3. Verás `powerbi-fabric` en estado **needs authentication** o similar.
4. Sigue el
   [Ejercicio 1 · Autenticación Entra ID](./ejercicios/01-autenticacion-entra-id.md).

---

## Ejercicios de este bloque

| # | Ejercicio | Qué practicas |
|---|---|---|
| 1 | [Autenticación Entra ID](./ejercicios/01-autenticacion-entra-id.md) | El flujo device code, paso a paso |
| 2 | [Consultar workspace de Fabric](./ejercicios/02-consultar-workspace-fabric.md) | Listar workspaces, datasets y ejecutar DAX en el Service |

---

## Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| `/mcp` muestra `failed` | Endpoint incorrecto o preview no habilitado en tu tenant. | Verifica la URL en la documentación oficial. Consulta con tu administrador si el preview está habilitado. |
| `AADSTS50020` | Tu cuenta no pertenece al tenant indicado. | Revisa que `TU_TENANT_ID` sea el de tu organización, no el de otra. |
| `AADSTS65001` / *consent required* | Falta consentimiento para la aplicación. | Un administrador de Entra ID debe otorgar el consentimiento. |
| `AADSTS70016` | El device code aún no fue ingresado. | Completa el paso del navegador. |
| `403 Forbidden` al listar workspaces | Tu cuenta no tiene acceso a ninguno. | Pide acceso, o usa "Mi área de trabajo". |
| `401 Unauthorized` después de un rato | El token expiró. | Vuelve a autenticarte con `/mcp`. |
| No aparecen datasets del workspace | Permisos insuficientes sobre los modelos. | Rol **Viewer** no siempre basta para consultar datos; puede requerir permiso *Build* sobre el semantic model. |
| La consulta DAX al Service falla pero en local funciona | El modelo publicado es distinto o está sin refrescar. | Revisa la última actualización del semantic model en el Service. |

> 🤖 Los códigos que empiezan con `AADSTS` son errores de Entra ID. Pégalos
> completos en Claude Code y pídele que te los traduzca: son bastante crípticos
> pero están todos documentados.

---

⬅️ **Anterior:** [02 · MCP Power BI local](../02-mcp-powerbi-local/README.md)
➡️ **Siguiente:** [Ejercicio 1 · Autenticación Entra ID](./ejercicios/01-autenticacion-entra-id.md)

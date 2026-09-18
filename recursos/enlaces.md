# Recursos y enlaces

Todo lo que se mencionó en el workshop, en un solo lugar.

> ⚠️ Los enlaces a documentación de Microsoft y de Anthropic cambian con cierta
> frecuencia. Si alguno no funciona, busca el título en el buscador del sitio
> oficial correspondiente.

---

## Git y GitHub

| Recurso | Enlace | Para qué |
|---|---|---|
| **Git for Windows** | <https://git-scm.com/download/win> | Descargar el instalador oficial ([bloque 00](../00-setup/01-instalar-git-windows.md)) |
| Documentación oficial de Git | <https://git-scm.com/doc> | Referencia completa |
| Libro *Pro Git* (en español, gratis) | <https://git-scm.com/book/es/v2> | La mejor introducción larga a Git |
| Crear cuenta de GitHub | <https://github.com/signup> | Registro |
| Tokens de acceso personal (fine-grained) | <https://docs.github.com/es/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens> | Generar el token del [bloque 00](../00-setup/02-configurar-github.md) |
| Conectar con SSH | <https://docs.github.com/es/authentication/connecting-to-github-with-ssh> | Alternativa al token |
| GitHub Docs en español | <https://docs.github.com/es> | Documentación general |

---

## Claude Code

| Recurso | Enlace | Para qué |
|---|---|---|
| **Documentación de Claude Code** | <https://docs.claude.com/en/docs/claude-code/overview> | Punto de partida |
| Instalación | <https://docs.claude.com/en/docs/claude-code/setup> | Instalar en Windows ([bloque 00](../00-setup/03-instalar-claude-code.md)) |
| Referencia de comandos | <https://docs.claude.com/en/docs/claude-code/cli-reference> | Todos los comandos y flags |
| Comandos de la sesión (`/help`, `/mcp`, …) | <https://docs.claude.com/en/docs/claude-code/slash-commands> | Los comandos que empiezan con `/` |
| **Configurar servidores MCP** | <https://code.claude.com/docs/en/mcp> | `claude mcp add`, `.mcp.json`, OAuth con `--client-id` y `--callback-port` ([bloques 02](../02-mcp-powerbi-local/README.md) y [03](../03-mcp-fabric-api/README.md)) |
| Memoria y `CLAUDE.md` | <https://docs.claude.com/en/docs/claude-code/memory> | Cómo funciona el archivo `CLAUDE.md` de este repo |
| Permisos y seguridad | <https://docs.claude.com/en/docs/claude-code/iam> | El cuadro de aprobación que ves antes de cada acción |

---

## MCP (Model Context Protocol)

| Recurso | Enlace | Para qué |
|---|---|---|
| Sitio oficial de MCP | <https://modelcontextprotocol.io> | Qué es y cómo funciona |
| Especificación | <https://modelcontextprotocol.io/specification> | Detalle técnico del protocolo |
| Servidores de ejemplo | <https://github.com/modelcontextprotocol/servers> | Catálogo de servidores MCP |
| Anuncio original de MCP | <https://www.anthropic.com/news/model-context-protocol> | El contexto de por qué existe |

---

## MCP para Power BI Desktop (bloque 02 · local, solo lectura)

| Recurso | Enlace | Para qué |
|---|---|---|
| **Power BI Desktop MCP Server** (dheerajkumar97) | <https://github.com/dheerajkumar97/power-bi-custom-mcp-server-with-python--claude-ai-integration> | El servidor que usamos. **Solo lectura.** |
| Perfil del autor | <https://github.com/dheerajkumar97> | Otros proyectos y actualizaciones |
| `pythonnet` | <https://github.com/pythonnet/pythonnet> | El puente Python ↔ .NET que necesita el servidor |
| Documentación de `pythonnet` | <https://pythonnet.github.io/> | Instalación y solución de problemas |
| Descargar Python | <https://www.python.org/downloads/windows/> | Python 3.8 o superior |
| **Librerías cliente de Analysis Services (ADOMD.NET)** | <https://learn.microsoft.com/es-es/analysis-services/client-libraries> | Si te aparece un error mencionando `ADOMD` |
| Conectividad XMLA | <https://learn.microsoft.com/es-es/power-bi/enterprise/service-premium-connect-tools> | Qué es el endpoint XMLA y cómo se usa |

> 📌 **Recordatorio:** este servidor es de **solo lectura**. Lee el modelo y
> ejecuta consultas DAX, pero no crea, modifica ni elimina nada. Es una decisión
> de diseño del taller — ver el [README del bloque 02](../02-mcp-powerbi-local/README.md#-punto-clave-este-servidor-es-de-solo-lectura).

### La alternativa oficial, para cuando necesites escribir

| Recurso | Enlace | Para qué |
|---|---|---|
| **Power BI Authoring MCP server** | <https://learn.microsoft.com/es-es/power-bi/developer/mcp/power-bi-authoring-mcp> | El MCP **oficial de Microsoft** para Power BI Desktop y archivos PBIP. **Sí crea y modifica** medidas, tablas, relaciones y roles. Tiene versión local (`stdio`) y hospedada. |
| Archivos Power BI Project (PBIP) | <https://learn.microsoft.com/es-es/power-bi/developer/projects/projects-overview> | Modelos como archivos de texto, versionables en Git. El entorno correcto para usar un MCP de escritura. |
| TMDL | <https://learn.microsoft.com/es-es/analysis-services/tmdl/tmdl-overview> | El formato de texto de los modelos tabulares |

---

## MCP oficial de Microsoft para Power BI / Fabric (bloque 03)

### Los endpoints

| Servidor | Endpoint | Escribe |
|---|---|---|
| **Power BI Consumption MCP** ⬅️ el del taller | `https://api.fabric.microsoft.com/v1/mcp/powerbi` | ❌ No |
| Power BI Authoring MCP (hospedado) | `https://api.fabric.microsoft.com/v1/mcp/powerbi/authoring` | ✅ Sí |
| Fabric IQ MCP | `https://fabriciq.svc.cloud.microsoft/v1/mcp/FabricIQ` | ❌ No |

> ⚠️ **El endpoint de Consumption está en preview.** Las definiciones de las
> herramientas, los formatos de solicitud y los esquemas de respuesta pueden
> cambiar. **Verifica la documentación antes de configurarlo.**
>
> 📌 Microsoft señala hoy **Fabric IQ** como el camino preferente para escenarios
> de consumo, y describe el endpoint de Consumption como el endpoint de consulta
> *anterior*, que sigue documentado. El taller usa Consumption por simplicidad y
> por alcance; para algo que vaya a durar, evalúa Fabric IQ.

### Documentación

| Recurso | Enlace | Para qué |
|---|---|---|
| **Documentación MCP de Power BI** | <https://learn.microsoft.com/es-es/power-bi/developer/mcp/> | Punto de entrada de todo el bloque 3 |
| **Comparativa de los servidores MCP** | <https://learn.microsoft.com/es-es/power-bi/developer/mcp/mcp-servers-overview> | Cuál usar según lo que necesites. Explica Authoring vs. consumo. |
| **Consumption MCP server · Get started** | <https://learn.microsoft.com/es-es/power-bi/developer/mcp/remote-mcp-server-get-started> | Endpoint, prerrequisitos, tenant setting y las 4 herramientas |
| **Herramientas del Consumption server** | <https://learn.microsoft.com/es-es/power-bi/developer/mcp/remote-mcp-server-tools> | Detalle de Execute Query, Get Schema, Get Report Metadata, Generate Query |
| **Registrar el server en clientes externos** | <https://learn.microsoft.com/es-es/power-bi/developer/mcp/remote-mcp-server-external-clients> | El registro de app en Entra ID, redirect URI y permisos delegados |
| **Power BI Authoring MCP server** | <https://learn.microsoft.com/es-es/power-bi/developer/mcp/power-bi-authoring-mcp> | El que **sí escribe**, local y hospedado. Para tu trabajo diario, no para el taller. |
| **Fabric IQ MCP** | <https://learn.microsoft.com/es-es/fabric/iq/connectors/fabric-iq-mcp> | El camino preferente de Microsoft para consumo |
| Preparar modelos para IA | <https://learn.microsoft.com/es-es/power-bi/create-reports/copilot-prepare-data-ai> | Mejora mucho la calidad de las respuestas sobre tu modelo |
| Permisos Build sobre semantic models | <https://learn.microsoft.com/es-es/power-bi/connect-data/service-datasets-build-permissions> | El permiso que necesitas para consultar (el rol *Viewer* no siempre basta) |
| REST API de Power BI | <https://learn.microsoft.com/es-es/rest/api/power-bi/> | Lo que hay debajo del MCP remoto |
| Ejecutar consultas DAX vía API | <https://learn.microsoft.com/es-es/rest/api/power-bi/datasets/execute-queries> | El endpoint detrás de la herramienta *Execute Query* |
| Row-Level Security en Fabric | <https://learn.microsoft.com/es-es/fabric/security/service-admin-row-level-security> | Por qué la identidad delegada respeta RLS y el service principal no |
| Asegurar servidores MCP | <https://learn.microsoft.com/es-es/azure/api-management/secure-mcp-servers> | Guía de seguridad de Microsoft para MCP |
| Blog de Microsoft Fabric | <https://blog.fabric.microsoft.com/> | Anuncios de nuevas capacidades y previews |

---

## Entra ID y autenticación (bloque 03)

| Recurso | Enlace | Para qué |
|---|---|---|
| **Registrar una aplicación en Entra ID** | <https://learn.microsoft.com/es-es/entra/identity-platform/quickstart-register-app> | El paso 1 de la configuración del bloque 3 |
| Centro de administración de Entra | <https://entra.microsoft.com> | Donde se registra la app |
| Authorization code flow con PKCE | <https://learn.microsoft.com/es-es/entra/identity-platform/v2-oauth2-auth-code-flow> | **El flujo que usa Claude Code** (navegador + callback en localhost) |
| Device code flow | <https://learn.microsoft.com/es-es/entra/identity-platform/v2-oauth2-device-code> | El flujo alternativo, explicado en el anexo del [ejercicio 01](../03-mcp-fabric-api/ejercicios/01-autenticacion-entra-id.md) |
| Página de device login | <https://microsoft.com/devicelogin> | Donde se ingresa el código, si usas ese flujo |
| Permisos delegados vs. de aplicación | <https://learn.microsoft.com/es-es/entra/identity-platform/permissions-consent-overview> | Por qué usamos identidad delegada y no service principal |
| Códigos de error `AADSTS` | <https://learn.microsoft.com/es-es/entra/identity-platform/reference-error-codes> | Traducir los errores de autenticación |
| Service principals en Power BI | <https://learn.microsoft.com/es-es/power-bi/enterprise/service-premium-service-principal> | El camino que **no** usamos en el taller. ⚠️ Recuerda: con service principal, Power BI **no aplica RLS**. |

---

## Power BI y DAX

| Recurso | Enlace | Para qué |
|---|---|---|
| Descargar Power BI Desktop | <https://powerbi.microsoft.com/es-es/desktop/> | El requisito del bloque 02 |
| Referencia de funciones DAX | <https://learn.microsoft.com/es-es/dax/dax-function-reference> | Referencia oficial |
| `EVALUATE` (consultas DAX) | <https://learn.microsoft.com/es-es/dax/dax-queries> | La sintaxis de consulta que usamos en los ejercicios |
| **DAX Guide** | <https://dax.guide> | La mejor referencia de DAX, función por función |
| SQLBI | <https://www.sqlbi.com/> | Artículos de modelado y DAX avanzado |
| DAX Studio | <https://daxstudio.org/> | Herramienta clásica para consultar modelos locales. Útil para contrastar lo que devuelve el MCP. |
| Tabular Editor | <https://tabulareditor.com/> | Edición avanzada de modelos. **Sí escribe** — la alternativa para cuando necesites modificar. |

---

## Si algo no funciona

1. **Copia el mensaje de error completo** y pégalo en Claude Code preguntando qué
   significa. Es lo que mejor hace, y practicas el flujo del taller.
2. Revisa el troubleshooting del bloque correspondiente:
   - [Bloque 00 · Git](../00-setup/01-instalar-git-windows.md#problemas-frecuentes)
   - [Bloque 00 · GitHub](../00-setup/02-configurar-github.md#problemas-frecuentes)
   - [Bloque 00 · Claude Code](../00-setup/03-instalar-claude-code.md#problemas-frecuentes)
   - [Bloque 02 · MCP local](../02-mcp-powerbi-local/README.md#troubleshooting)
   - [Bloque 03 · MCP Fabric](../03-mcp-fabric-api/README.md#troubleshooting)
3. Las dos causas más frecuentes de todo el taller:
   - **Power BI Desktop cerrado** → el MCP local deja de funcionar.
   - **No reiniciaste Claude Code** → los servidores MCP se cargan al iniciar la
     sesión.

---

⬅️ **Volver al** [README principal](../README.md)

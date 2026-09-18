# Claude Code + MCP para Power BI y Fabric

Workshop práctico de 2 horas para analistas de datos y BI que quieren usar
**Claude Code** como asistente sobre sus modelos de Power BI, tanto en el
escritorio (Power BI Desktop) como en la nube (Power BI Service / Microsoft Fabric).

---

## Objetivo

Al terminar el workshop vas a poder:

1. Tener **Git instalado y funcionando** en tu PC con Windows, y sincronizar tu
   trabajo con GitHub.
2. Usar **Claude Code** desde la terminal para leer, escribir y versionar archivos.
3. Conectar Claude Code a tu **modelo local de Power BI Desktop** mediante un
   servidor MCP **de solo lectura**, para explorar tablas, relaciones, medidas y
   correr consultas DAX.
4. Conectar Claude Code a **Microsoft Fabric / Power BI Service** vía **Fabric IQ**,
   el MCP remoto oficial de Microsoft — también de solo lectura —, autenticándote
   con tu propia identidad de Entra ID.
5. Documentar hallazgos en Markdown y dejarlos **commiteados en GitHub**.

> No necesitas saber programar. Si nunca abriste una terminal, este material está
> escrito para ti: cada comando viene con el contexto de qué hace y qué deberías ver.

---

## Audiencia

Analistas de datos / BI con perfil **Power BI** (DAX, Power Query, modelado),
con poca o nula experiencia en terminal, Git o herramientas de desarrollo.

---

## Duración

**2 horas** (120 minutos), en formato taller guiado.

---

## Prerrequisitos

Revisa esto **antes** de llegar al workshop. Si algo falla, el bloque de Setup
está pensado para resolverlo, pero llegar con esto listo te da más tiempo práctico.

| Requisito | Detalle |
|---|---|
| **Windows** | Windows 10 u 11. |
| **Power BI Desktop** | Instalado, y con **un modelo abierto** durante el workshop (cualquier `.pbix` con datos). Es obligatorio: el endpoint local que usa el MCP solo existe mientras Power BI Desktop tiene un reporte abierto. |
| **Python 3.8 o superior** | Necesario para el servidor MCP local. Verifica con `python --version`. |
| **Cuenta de GitHub** | Gratuita. Si no tienes, la creamos en el bloque de Setup. |
| **Acceso a un reporte o semantic model** | En Power BI Service o Fabric. Basta con que puedas abrirlo: **no necesitas permiso Build ni rol en el workspace.** Para el bloque 3. |
| **Cuenta de Claude** | Con acceso a Claude Code (plan Pro / Max, o API). |

### ⚠️ Un requisito del bloque 3 que conviene resolver antes

El bloque de Fabric usa **Fabric IQ**, el servidor MCP oficial de Microsoft
(disponible de forma general). Para conectarse desde Claude Code hace falta un
**registro de aplicación en Entra ID** con tres permisos delegados de lectura.

| Requisito | Quién lo hace |
|---|---|
| Registro de app en **Entra ID** con `Item.Read.All`, `Item.Execute.All` y `Dataset.Read.All` | Tú, si puedes registrar apps; si no, tu admin de Entra ID |

> 🟡 **Los tres permisos no requieren consentimiento de administrador por
> defecto** — tú mismo puedes consentirlos. Pero si tu tenant restringe el
> consentimiento de usuarios, vas a necesitar que alguien de TI lo apruebe una
> vez. **Averígualo con anticipación.** El paso a paso está en el
> [README del bloque 3](./03-mcp-fabric-api/README.md#configuración).
>
> 📍 **Fabric IQ no está disponible** en regiones *Power BI-only* ni en nubes
> soberanas: el *home region* de tu tenant debe soportar todas las cargas de
> trabajo de Fabric.

---

## Agenda

| Tiempo | Bloque | Contenido |
|---|---|---|
| **15 min** | [00 · Setup](./00-setup/) | Instalar Git en Windows, configurar GitHub, instalar Claude Code. |
| **15 min** | [01 · Claude Code básico](./01-claude-code-basico/README.md) | Qué es, en qué se diferencia de Claude.ai, comandos esenciales, primer commit. |
| **35 min** | [02 · MCP Power BI local](./02-mcp-powerbi-local/README.md) | Servidor MCP **de solo lectura** sobre Power BI Desktop: explorar el modelo, consultar DAX, documentar medidas. |
| **35 min** | [03 · MCP Fabric / API](./03-mcp-fabric-api/README.md) | **Fabric IQ**, el MCP oficial de Microsoft (GA, solo lectura): autenticación Entra ID delegada, descubrir contenido por nombre, leer esquemas y ejecutar DAX contra Fabric. |
| **20 min** | [04 · Flujo completo](./04-flujo-completo/README.md) | Ejercicio integrador: comparar local vs. Service, documentar y commitear. Cierre y preguntas. |

---

## Recorrido del repositorio (en orden)

1. **[00-setup/](./00-setup/)**
   - [01 · Instalar Git en Windows](./00-setup/01-instalar-git-windows.md)
   - [02 · Configurar GitHub](./00-setup/02-configurar-github.md)
   - [03 · Instalar Claude Code](./00-setup/03-instalar-claude-code.md)
2. **[01-claude-code-basico/](./01-claude-code-basico/README.md)**
3. **[02-mcp-powerbi-local/](./02-mcp-powerbi-local/README.md)**
   - [Ejercicio 1 · Explorar el modelo](./02-mcp-powerbi-local/ejercicios/01-explorar-modelo.md)
   - [Ejercicio 2 · Consultar DAX](./02-mcp-powerbi-local/ejercicios/02-consultar-dax.md)
   - [Ejercicio 3 · Documentar medidas](./02-mcp-powerbi-local/ejercicios/03-documentar-medidas.md)
4. **[03-mcp-fabric-api/](./03-mcp-fabric-api/README.md)**
   - [Ejercicio 1 · Autenticación Entra ID](./03-mcp-fabric-api/ejercicios/01-autenticacion-entra-id.md)
   - [Ejercicio 2 · Explorar y consultar en Fabric](./03-mcp-fabric-api/ejercicios/02-consultar-workspace-fabric.md)
5. **[04-flujo-completo/](./04-flujo-completo/README.md)**
6. **[recursos/enlaces.md](./recursos/enlaces.md)**

---

## Una nota importante: los dos MCP son de solo lectura

**Ninguno de los servidores que vas a usar puede modificar tus modelos.**

| Bloque | Servidor | Qué puede hacer |
|---|---|---|
| 02 | Power BI Desktop MCP Server | Leer el modelo y ejecutar DAX. Nada más. |
| 03 | **Fabric IQ** (oficial de Microsoft) | Descubrir contenido, leer esquemas y ejecutar DAX. Nada más. |

Esto es una **decisión de diseño del taller**, no una carencia. En un taller con
muchas personas aprendiendo a la vez, es esperable que alguien apruebe una acción
de Claude Code sin leerla con atención. Con servidores de solo lectura, el peor
caso posible es una consulta que no sirve — nunca un modelo dañado.

Existen servidores MCP que **sí** crean y modifican modelos, incluidos los
oficiales de Microsoft. Son herramientas legítimas para tu trabajo diario, con
control de versiones y revisión de cambios detrás. **Los dejamos fuera a
propósito**, y el material explica dónde encontrarlos cuando los necesites.

---

## Cómo pedir ayuda durante el workshop

Si un comando falla, **copia el mensaje de error completo** y pégalo en Claude Code
preguntando qué significa. Es una de las cosas que mejor hace, y de paso practicas
el flujo de trabajo del taller.

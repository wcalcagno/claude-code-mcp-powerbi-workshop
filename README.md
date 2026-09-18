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
4. Conectar Claude Code al **Power BI Service / Fabric** vía el MCP remoto oficial
   de Microsoft (en preview), autenticándote con tu propia identidad de Entra ID.
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
| **Acceso a un semantic model** | En Power BI Service o Fabric, con permiso **Build** (el rol *Viewer* no siempre basta). Para el bloque 3. |
| **Cuenta de Claude** | Con acceso a Claude Code (plan Pro / Max, o API). |

### ⚠️ Dos requisitos del bloque 3 que dependen de tu área de TI

El bloque de Fabric usa el endpoint oficial de Microsoft, que está en **preview**
y exige dos cosas que **no puedes resolver tú solo el día del taller**:

| Requisito | Quién lo habilita |
|---|---|
| Tenant setting *"Users can use the Power BI Model Context Protocol server endpoint (preview)"* | Tu **administrador de Power BI** |
| Un registro de aplicación en **Entra ID** con permisos delegados de lectura | Tú, si puedes registrar apps; si no, tu admin de Entra ID |

> 🔴 **Gestiónalos con varios días de anticipación.** En una empresa grande esto
> puede tomar tiempo. Si llegas al bloque 3 sin resolverlo, vas a poder leer y
> entender el flujo, pero no ejecutarlo en vivo. El detalle está en el
> [README del bloque 3](./03-mcp-fabric-api/README.md#prerrequisitos-de-este-bloque).

---

## Agenda

| Tiempo | Bloque | Contenido |
|---|---|---|
| **15 min** | [00 · Setup](./00-setup/) | Instalar Git en Windows, configurar GitHub, instalar Claude Code. |
| **15 min** | [01 · Claude Code básico](./01-claude-code-basico/README.md) | Qué es, en qué se diferencia de Claude.ai, comandos esenciales, primer commit. |
| **35 min** | [02 · MCP Power BI local](./02-mcp-powerbi-local/README.md) | Servidor MCP **de solo lectura** sobre Power BI Desktop: explorar el modelo, consultar DAX, documentar medidas. |
| **35 min** | [03 · MCP Fabric / API](./03-mcp-fabric-api/README.md) | **Power BI Consumption MCP server** oficial de Microsoft (preview), autenticación Entra ID delegada, consultar el esquema y ejecutar DAX contra el Service. |
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
   - [Ejercicio 2 · Consultar un modelo del Service](./03-mcp-fabric-api/ejercicios/02-consultar-workspace-fabric.md)
5. **[04-flujo-completo/](./04-flujo-completo/README.md)**
6. **[recursos/enlaces.md](./recursos/enlaces.md)**

---

## Una nota importante: el MCP local es de solo lectura

El servidor MCP que usamos sobre Power BI Desktop es **de solo lectura**.
Puede *leer* el modelo (tablas, columnas, relaciones, medidas) y *ejecutar
consultas DAX*, pero **no crea, no modifica ni elimina nada** en tu modelo.

Esto es una **decisión de diseño del taller**, no una carencia. En un taller con
muchas personas aprendiendo a la vez, es esperable que alguien apruebe una acción
de Claude Code sin leerla con atención. Con un servidor de solo lectura, el peor
caso posible es una consulta que no sirve — nunca un modelo dañado.

Lo mismo aplica al bloque de Fabric: trabajamos en modo **consulta**, no de escritura.

---

## Cómo pedir ayuda durante el workshop

Si un comando falla, **copia el mensaje de error completo** y pégalo en Claude Code
preguntando qué significa. Es una de las cosas que mejor hace, y de paso practicas
el flujo de trabajo del taller.

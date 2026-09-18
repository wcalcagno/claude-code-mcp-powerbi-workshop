# 02 · MCP para Power BI Desktop (local, solo lectura)

⏱️ **Duración del bloque:** 35 minutos
📍 **Requisito:** haber completado [01 · Claude Code básico](../01-claude-code-basico/README.md)

---

## ¿Qué es MCP? (en una frase)

> **MCP es un enchufe estándar que permite a Claude Code conectarse a una
> herramienta externa — en nuestro caso, tu modelo de Power BI Desktop — y
> trabajar con ella.**

MCP son las siglas de *Model Context Protocol*. La idea es la misma que un
conector de Power Query: en vez de que cada herramienta invente su propia forma
de conectarse, existe un estándar. Quien quiera exponer algo (una base de datos,
un modelo de Power BI, una API) publica un **servidor MCP**, y cualquier
asistente que hable MCP — como Claude Code — puede usarlo.

| En Power BI | En Claude Code |
|---|---|
| Un conector de Power Query | Un servidor MCP |
| "Obtener datos → SQL Server" | "Conectar servidor MCP de Power BI" |
| El conector expone tablas | El servidor MCP expone *herramientas* |

Cuando conectas un servidor MCP, Claude Code gana **herramientas nuevas**:
además de leer archivos, ahora puede preguntarle cosas a tu modelo.

---

## El servidor que vamos a usar

**Power BI Desktop MCP Server** — de [dheerajkumar97](https://github.com/dheerajkumar97).

| Característica | Detalle |
|---|---|
| Lenguaje | Python |
| Cómo se conecta | Al endpoint **XMLA** local de Power BI Desktop, vía **ADOMD.NET** |
| Dónde corre | En tu propio computador |
| Credenciales | **Ninguna.** Es una conexión local a un proceso que ya está corriendo en tu PC. |
| Permisos | **SOLO LECTURA** |

### Qué significa "vía XMLA / ADOMD.NET"

Cuando abres un `.pbix`, Power BI Desktop levanta en segundo plano **un motor de
Analysis Services** en tu máquina — el mismo motor que corre en el Service.
Ese motor escucha en un **puerto local** y acepta consultas por el protocolo
**XMLA**. `ADOMD.NET` es la librería de Microsoft que permite hablar ese protocolo.

Consecuencia práctica, y es la causa del 90% de los problemas del taller:

> 🔴 **El endpoint solo existe mientras Power BI Desktop tiene un reporte abierto.**
> Si cierras Power BI Desktop, el puerto desaparece y el MCP deja de funcionar.

---

## ⭐ Punto clave: este servidor es de SOLO LECTURA

Este servidor MCP **puede**:

- ✅ Listar tablas, columnas, relaciones y jerarquías del modelo.
- ✅ Listar medidas con su expresión DAX.
- ✅ Ejecutar consultas DAX de lectura y devolver resultados.
- ✅ Describir la estructura del modelo.

Este servidor MCP **no puede**:

- ❌ Crear, modificar ni eliminar medidas.
- ❌ Agregar, cambiar ni borrar tablas o columnas.
- ❌ Alterar relaciones.
- ❌ Modificar el archivo `.pbix` de ninguna forma.

### Por qué elegimos esto para el taller

No es una limitación que haya que disculpar: **es el diseño correcto para este
contexto.**

En un taller, con muchas personas usando la herramienta por primera vez, la
secuencia realista es esta: Claude Code propone una acción, aparece el cuadro de
permisos, y alguien presiona `1` sin leerlo con atención. Es humano y va a pasar.

Con un servidor de solo lectura, **el peor resultado posible de ese clic apurado
es una consulta que no sirve**. No hay medida sobrescrita, no hay relación rota,
no hay modelo que reconstruir con el reloj corriendo.

Ese es el trato: **cedemos capacidad de escritura a cambio de poder experimentar
sin miedo.** Y para lo que hace un analista el 90% del tiempo — entender un
modelo que heredó, validar un número, documentar medidas — leer es suficiente.

> 💬 **Y si mañana quieres escritura:** existen servidores MCP que sí modifican
> modelos (por ejemplo, vía TMDL o Tabular Editor). Son herramientas legítimas
> para tu trabajo diario, con control de versiones y un entorno de desarrollo
> detrás. No son la herramienta correcta para un salón de clases.

---

## Prerrequisitos de este bloque

| Requisito | Cómo verificarlo |
|---|---|
| **Power BI Desktop abierto, con un modelo cargado** | Abre cualquier `.pbix` con datos. **Déjalo abierto todo el bloque.** |
| **Python 3.8 o superior** | `python --version` en PowerShell |
| **Paquete `pythonnet`** | Lo instalamos en el Paso 2 |
| **Git** | Ya lo instalaste en el bloque 00 |

### Verifica Python

```powershell
python --version
```

**Resultado esperado:** `Python 3.11.9` (o cualquier versión ≥ 3.8).

Si te abre la Microsoft Store o dice que no se reconoce, instálalo:

```powershell
winget install Python.Python.3.12
```

Luego **cierra y abre PowerShell** y verifica de nuevo.

---

## Instalación

### Paso 1 · Clonar el servidor MCP

Lo dejamos **fuera** de la carpeta del workshop, para no mezclar el código del
servidor con el material del taller.

```powershell
cd $env:USERPROFILE\workshops
```

```powershell
git clone https://github.com/dheerajkumar97/power-bi-custom-mcp-server-with-python--claude-ai-integration.git powerbi-mcp-server
```

> 💡 El último argumento (`powerbi-mcp-server`) renombra la carpeta local a algo
> más corto. Es opcional, pero te ahorra escribir una ruta larguísima después.

Entra a la carpeta y mira qué hay:

```powershell
cd powerbi-mcp-server
```

```powershell
ls
```

**Resultado esperado:** archivos `.py` y, probablemente, un `README.md` y un
`requirements.txt`. **Anota el nombre del archivo Python principal** (suele ser
algo como `server.py` o `powerbi_mcp_server.py`) — lo necesitas en el Paso 3.

> 🤖 **Puedes preguntárselo a Claude Code.** Ábrelo en esa carpeta y pídele:
> *"¿Cuál es el archivo principal que levanta el servidor MCP y qué dependencias
> necesita según el README?"*

### Paso 2 · Instalar las dependencias de Python

`pythonnet` es el puente que permite a Python usar librerías de .NET — en este
caso, `ADOMD.NET`, que es la que habla con Power BI.

```powershell
pip install pythonnet
```

Si el repositorio trae un `requirements.txt`, mejor instala todo de una vez:

```powershell
pip install -r requirements.txt
```

**Resultado esperado:** líneas de descarga terminando en
`Successfully installed pythonnet-3.x.x` (y las demás dependencias).

#### Verifica que `pythonnet` quedó bien

```powershell
python -c "import clr; print('pythonnet OK')"
```

**Resultado esperado:**

```
pythonnet OK
```

> 📌 Sí, el paquete se llama `pythonnet` pero se importa como `clr`
> (*Common Language Runtime*). Es confuso, pero es así.

### Paso 3 · Anotar la ruta completa del script

Estando dentro de la carpeta del servidor:

```powershell
(Get-Location).Path
```

**Resultado esperado:** algo como
`C:\Users\TU_USUARIO\workshops\powerbi-mcp-server`

La ruta completa al script será esa ruta + `\` + el nombre del archivo principal.
Por ejemplo:

```
C:\Users\TU_USUARIO\workshops\powerbi-mcp-server\server.py
```

**Cópiala a un bloc de notas.** La necesitas en el paso siguiente.

### Paso 4 · Registrar el servidor en Claude Code

Tienes dos formas. **La A es más simple.**

#### Opción A · Con el comando `claude mcp add`

Vuelve a la carpeta del workshop:

```powershell
cd $env:USERPROFILE\workshops\claude-code-mcp-powerbi-workshop
```

Y registra el servidor (reemplaza la ruta por la tuya):

```powershell
claude mcp add powerbi-local --scope local -- python "C:\Users\TU_USUARIO\workshops\powerbi-mcp-server\server.py"
```

Qué significa cada parte:

| Parte | Significado |
|---|---|
| `claude mcp add` | Registra un servidor MCP nuevo. |
| `powerbi-local` | El nombre con que lo verás en Claude Code. Puedes elegir otro. |
| `--scope local` | Queda registrado solo para ti, en este proyecto. No se comparte por Git. |
| `--` | Separador: lo que viene después es el comando que levanta el servidor. |
| `python "C:\...\server.py"` | Ejecutar ese script con Python. |

#### Opción B · Editando el archivo de configuración

Copia el archivo de ejemplo de este repositorio y ajústalo:

📄 **[`mcp-config-ejemplo.json`](./mcp-config-ejemplo.json)**

Ese archivo tiene las instrucciones adentro. Resumen: creas un `.mcp.json` en la
raíz del proyecto con la ruta a tu script.

> 🤖 **Atajo:** pídeselo a Claude Code.
> *"Lee @02-mcp-powerbi-local/mcp-config-ejemplo.json y créame un .mcp.json en la
> raíz del proyecto, con la ruta C:/Users/TU_USUARIO/workshops/powerbi-mcp-server/server.py,
> sin los campos de comentarios."*

### Paso 5 · Reiniciar Claude Code y verificar

Los servidores MCP se cargan **al iniciar la sesión**. Si Claude Code está
abierto, sal con `/exit` y vuelve a entrar:

```powershell
claude
```

Dentro de Claude Code:

```
/mcp
```

**Resultado esperado:** una lista con tu servidor y el estado **connected**:

```
powerbi-local    ✔ connected    (N tools)
```

Si dice `failed` o `connecting`, revisa la sección de problemas más abajo.

---

## Antes de los ejercicios: checklist de 10 segundos

- [ ] **Power BI Desktop está abierto, con un reporte cargado.** (El más importante.)
- [ ] `/mcp` muestra `powerbi-local` como `connected`.
- [ ] Estás en la carpeta del workshop.

---

## Ejercicios de este bloque

| # | Ejercicio | Qué practicas |
|---|---|---|
| 1 | [Explorar el modelo](./ejercicios/01-explorar-modelo.md) | Tablas, columnas y relaciones |
| 2 | [Consultar DAX](./ejercicios/02-consultar-dax.md) | Ejecutar una consulta y entender el resultado |
| 3 | [Documentar medidas](./ejercicios/03-documentar-medidas.md) | Leer con MCP + escribir un `.md` con Claude Code |

---

## Troubleshooting

### 🔴 "No se encuentra el puerto de Analysis Services"

Variantes del mensaje: *"No Power BI Desktop instance found"*, *"Could not detect
Analysis Services port"*, *"no running instance"*.

**Causa:** no hay ningún Power BI Desktop con un reporte abierto. El motor local
— y por lo tanto el puerto — solo existe mientras hay un `.pbix` cargado.

**Solución, en orden:**

1. **Abre Power BI Desktop** y carga un archivo `.pbix` **con datos**.
   Un Power BI Desktop recién abierto, en la pantalla de bienvenida, **no cuenta**.
2. Espera a que el modelo termine de cargar (que no haya spinner de "Cargando").
3. Verifica que realmente hay datos: ve a la vista **Datos** (el ícono de tabla
   en la barra izquierda) y confirma que ves filas.
4. Sal de Claude Code (`/exit`) y vuelve a entrar. La conexión se establece al
   iniciar la sesión.

**Comprobación manual del puerto** (opcional, para curiosos). Power BI Desktop
escribe el puerto en un archivo temporal:

```powershell
Get-Process msmdsrv -ErrorAction SilentlyContinue | Select-Object Id, ProcessName
```

Si eso no devuelve nada, **no hay motor corriendo**: el problema es 100% que no
hay un modelo abierto.

> 📌 **Si tienes varios `.pbix` abiertos**, hay varios puertos. Para el taller,
> lo más simple es **dejar uno solo abierto**.

---

### 🔴 "pythonnet no está instalado" / `ModuleNotFoundError: No module named 'clr'`

**Causa:** falta el paquete, o lo instalaste en una instalación de Python distinta
a la que usa el servidor.

**Solución:**

1. Instálalo:

   ```powershell
   pip install pythonnet
   ```

2. Verifica:

   ```powershell
   python -c "import clr; print('pythonnet OK')"
   ```

3. Si falla, revisa que `pip` y `python` sean del mismo intérprete:

   ```powershell
   python -m pip install pythonnet
   ```

   Usar `python -m pip` garantiza que instalas en el Python correcto.

4. Confirma cuál Python se está usando:

   ```powershell
   (Get-Command python).Source
   ```

   Esa ruta debe ser la misma que pusiste en la configuración del MCP. Si en tu
   `.mcp.json` escribiste solo `"python"`, se usará el primero del `PATH` —
   normalmente el correcto.

---

### 🔴 Otros problemas

| Síntoma | Causa probable | Solución |
|---|---|---|
| `/mcp` no lista el servidor | No reiniciaste Claude Code. | `/exit` y `claude` de nuevo. |
| `/mcp` dice `failed` | Ruta al script incorrecta. | Verifica con `Test-Path "C:\ruta\a\server.py"`. Debe responder `True`. |
| `failed` y la ruta es correcta | Error al arrancar el script. | Pruébalo a mano: `python "C:\ruta\a\server.py"`. El error se verá en pantalla. |
| Error mencionando `ADOMD` o `Microsoft.AnalysisServices` | Falta la librería cliente de Analysis Services. | Instala los *SQL Server Analysis Services client libraries* (ADOMD.NET) desde la documentación de Microsoft. |
| Las consultas DAX devuelven vacío | El modelo está abierto pero sin datos cargados. | Refresca el modelo en Power BI Desktop. |
| Funcionaba y dejó de funcionar | Cerraste Power BI Desktop, o el puerto cambió. | Abre el `.pbix` de nuevo y reinicia Claude Code. |
| Rutas con espacios dan error | Faltan comillas o barras invertidas mal escapadas. | En JSON usa barras normales: `C:/Users/...`. Siempre entre comillas. |

> 🤖 **El mejor troubleshooting:** copia el mensaje de error completo y pégaselo a
> Claude Code preguntando *"¿qué significa este error y cómo lo resuelvo?"*.

---

⬅️ **Anterior:** [01 · Claude Code básico](../01-claude-code-basico/README.md)
➡️ **Siguiente:** [Ejercicio 1 · Explorar el modelo](./ejercicios/01-explorar-modelo.md)

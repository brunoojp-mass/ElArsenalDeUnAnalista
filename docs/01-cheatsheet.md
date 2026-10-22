# 01 — CheatSheet de x64dbg

> Convención: **x64dbg** depura binarios de 64 bits, **x32dbg** los de 32 bits. La interfaz y
> los atajos son idénticos; solo cambian los registros (`RIP/RSP/RBP` vs `EIP/ESP/EBP`).

---

## La interfaz (ventana CPU y pestañas)

| Panel / pestaña | Atajo | Para qué sirve |
|-----------------|-------|----------------|
| **CPU** (disassembly) | `Alt+C` | Vista principal: ensamblador, registros, dump y pila |
| **Graph** | `G` (en CPU) | Vista de grafo del flujo de la función actual |
| **Log** | `Alt+L` | Mensajes, eventos y salida de comandos |
| **Breakpoints** | `Alt+B` | Lista y gestión de todos los breakpoints |
| **Memory Map** | `Alt+M` | Mapa de memoria del proceso (regiones, permisos) |
| **Call Stack** | `Alt+K` | Pila de llamadas actual |
| **SEH** | — | Cadena de manejadores de excepciones |
| **Script** | — | Editor de scripts de x64dbg |
| **Symbols** | — | Símbolos y módulos cargados |
| **References** | — | Resultados de búsquedas (strings, llamadas…) |
| **Threads / Handles** | — | Hilos y handles del proceso |

Dentro de la ventana CPU hay 4 sub-paneles: **disassembly** (arriba-izq), **registros**
(arriba-der), **dump** (abajo-izq) y **pila/stack** (abajo-der).

---

## Control de ejecución (lo que más vas a usar)

| Acción | Atajo | Comando | Comentario |
|--------|-------|---------|------------|
| Ejecutar / continuar | `F9` | `run` / `g` | Corre hasta el siguiente breakpoint o excepción |
| Step **into** | `F7` | `sti` / `t` | Entra dentro de las `call` |
| Step **over** | `F8` | `sto` / `p` | Ejecuta la `call` entera sin entrar |
| Ejecutar hasta `ret` | `Ctrl+F9` | `rtr` | Sale de la función actual |
| Ejecutar hasta código de usuario | `Alt+F9` | — | Útil para salir de librerías del sistema |
| Ejecutar hasta la selección | `F4` | — | "Run to" sobre la línea seleccionada |
| Pausar | `F12` | `pause` | Detiene la ejecución |
| Reiniciar | `Ctrl+F2` | `restart` | Vuelve a cargar el binario |
| Cerrar / terminar | `Alt+F2` | `stop` | Detiene el debuggee |

> Regla mnemotécnica: **F7 entra, F8 salta, F9 corre**. `Ctrl+F9` para "déjame salir de aquí".

---

## Breakpoints

| Tipo | Atajo / acción | Comando | Cuándo usarlo |
|------|----------------|---------|---------------|
| **Software** (INT3) | `F2` sobre la instrucción | `bp <addr/símbolo>` | El habitual. Ej.: `bp VirtualAlloc` |
| **Hardware** | menú contextual | `bph <addr>` | Cuando no puedes modificar bytes (código protegido) |
| **Memoria** (R/W/X) | menú contextual | `bpm <addr>, <r\|w\|x>` | Romper cuando se **lee/escribe/ejecuta** una zona |
| Condicional | editar el bp | — | Rompe solo si se cumple una expresión |
| Listar / limpiar | — | `bl` / `bc` | `bl` lista, `bc` limpia |

Ejemplos de comandos útiles en la barra inferior:

```text
bp kernel32.VirtualAlloc        ; parar en una API clave
bpm 0x0019FF00, w               ; parar cuando se escriba en esa dirección (ideal para BOF)
bph 0x00401000, x               ; breakpoint hardware de ejecución
bc                              ; limpiar todos los breakpoints
```

> **Para analizar un Buffer Overflow**, un `bpm` de **escritura** sobre la dirección de retorno
> en la pila te dice el instante exacto en que tu entrada la sobrescribe.

---

## Navegación y búsqueda

| Acción | Atajo | Comando | Comentario |
|--------|-------|---------|------------|
| Ir a expresión/dirección (disasm) | `Ctrl+G` | `disasm <expr>` | Acepta símbolos, registros y aritmética |
| Ir a dirección (dump) | `Ctrl+G` (en dump) | `dump <expr>` | Ej.: `dump esp` |
| Seguir en dump / disasm | `Enter` / menú | — | Sigue el operando bajo el cursor |
| Buscar patrón de bytes | `Ctrl+B` | `find <addr>, <patrón>` | Búsqueda binaria |
| Buscar instrucción | — | `findasm "jmp esp"` | Busca una instrucción ensamblada |
| **String references** | menú *Search for* | — | Lista strings del módulo (pistas rápidas) |
| **Intermodular calls** | menú *Search for* | — | Lista llamadas a APIs (muy útil en malware) |

---

## Registros, memoria y parcheo

| Acción | Cómo | Comentario |
|--------|------|------------|
| Modificar un registro | doble clic en el panel de registros | O por comando: `mov eip, 0x401000` |
| Fijar el origen aquí | `Ctrl+*` | Pone EIP/RIP en la instrucción seleccionada |
| Editar/ensamblar instrucción | `Space` | Escribe ensamblador directamente (patch) |
| Editar bytes | menú *Binary → Edit* | Modifica el hex en memoria |
| Guardar el patch a disco | menú *Patches* (`Ctrl+P`) | Exporta el ejecutable parcheado |
| Volcar memoria | menú del dump → *Follow/Dump* | O `savedata <archivo>, <addr>, <tamaño>` |

Ejemplos:

```text
mov eip, esp          ; redirigir ejecución (solo en laboratorio, PoC de control)
mov [esp], 0x41414141 ; escribir en memoria
```

---

## Scripting (automatizar lo repetitivo)

x64dbg trae su propio lenguaje de script (pestaña *Script*). Útil para preparar un entorno de
análisis siempre igual:

```text
// Ejemplo mínimo: parar en una API y loguear
bp VirtualAlloc
log "Breakpoint puesto en VirtualAlloc"
run
```

Comandos frecuentes en scripts: `bp`, `bpm`, `run`, `sti`, `sto`, `log`, `msg`, `find`,
`alloc`, `free`, `loadlib`, `mov`.

---

## Plugins imprescindibles

| Plugin | Para qué | Nota |
|--------|----------|------|
| **ScyllaHide** | Anti-anti-debug (oculta el debugger) | Clave para malware que detecta depuradores |
| **Scylla** | Volcado de procesos y reconstrucción de IAT | Unpacking de binarios empaquetados |
| **xAnalyzer** | Análisis extendido (comentarios de APIs, argumentos) | Mejora la legibilidad del desensamblado |
| **ERC.Xdbg** | *pattern create/offset*, búsqueda de gadgets ROP, SEH | El equivalente a `mona` para x64dbg |
| **Labelless** | Sincroniza etiquetas con IDA | Integra análisis estático y dinámico |

> Para **desarrollo de exploits**, **ERC.Xdbg** es el plugin estrella: genera patrones cíclicos,
> calcula offsets y busca gadgets, justo lo que necesitas tras un crash.

---

## Mini-tabla "quiero hacer X"

| Quiero… | Hago… |
|---------|-------|
| Ver dónde cae mi entrada tras un crash | Mirar los registros y `dump esp` en el momento del fallo |
| Saber el offset hasta EIP/RIP | Patrón cíclico (ERC: `!pattern_create`) → leer EIP → `!pattern_offset` |
| Encontrar un `jmp esp` para el salto | `findasm "jmp esp"` o ERC para gadgets |
| Parar justo al corromper la pila | `bpm <dir_retorno>, w` |
| Entender qué APIs usa un binario | *Search for → Intermodular calls* |
| Saltarme un chequeo | `Space` para ensamblar un `nop`/`jmp` (patch de laboratorio) |
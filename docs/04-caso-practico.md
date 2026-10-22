# 04 — Caso práctico: x64dbg frente a un Stack Buffer Overflow

Para demostrar uso **real** (no solo teoría), aplico x64dbg al análisis del crash de un binario
vulnerable escrito por mí (`vulnerable.exe`, un Stack Buffer Overflow de 32 bits), en una **VM
aislada**. Es el mismo binario del laboratorio de la tarea individual, así que ambos trabajos se
complementan.

> ⚠️ Todo se hace sobre **código propio** en laboratorio aislado, con fines educativos. Me quedo
> en el nivel de **prueba de concepto de control de flujo**, sin shellcode ofensivo.

---

## Objetivo

Usar x64dbg (en realidad **x32dbg**, por ser binario de 32 bits) para:

1. Reproducir el crash y confirmar que se controla **`EIP`**.
2. Calcular el **offset** exacto hasta `EIP`.
3. Localizar un **`jmp esp`** que sirva de trampolín.
4. Dejar documentado el camino, con capturas.

---

## Paso 1 — Cargar el binario y pasar la entrada

- Abro **x32dbg** → *File → Open* → `vulnerable.exe`.
- *File → Change Command Line* para pasar la entrada maliciosa como argumento.
- El debugger se detiene en el EntryPoint. Pulso `F9` para ejecutar.

---

## Paso 2 — Provocar el crash y leer los registros

Con una entrada suficientemente larga, el programa casca y x32dbg captura la **excepción de
violación de acceso**. En el panel de **registros** compruebo el valor de `EIP`.

Si en lugar de `A`*N uso un **patrón cíclico**, el valor que cae en `EIP` me permite calcular el
offset. Con el plugin **ERC.Xdbg**:

```text
!pattern_create 1200        ; genero el patrón, lo uso como entrada
; ... provoco el crash ...
!pattern_offset 0x41366441  ; (valor leído en EIP) -> me devuelve el offset exacto
```

Cuando el offset es correcto, al sobrescribir con un marcador reconocible veo:

```text
EIP = 42424242   ("BBBB")   -> control de flujo CONFIRMADO
```

*TODO: captura del panel de registros con EIP = 42424242 y el estado de ESP.*

---

## Paso 3 — Inspeccionar la pila en el momento del fallo

En la barra de comandos:

```text
dump esp          ; veo el contenido de la pila; aquí localizo dónde cae mi entrada
```

Confirmo que justo después de `EIP` tengo bytes que controlo (apuntados por `ESP`), lo que hace
viable la técnica del trampolín **`jmp esp`**.

---

## Paso 4 — Buscar un trampolín `jmp esp`

```text
findasm "jmp esp"     ; busco la instrucción en los módulos cargados
```

De los resultados elijo una dirección en un módulo **sin ASLR** (en este lab, sin protecciones).
Esa dirección será la que coloque en `EIP`, de modo que la ejecución salte a mi relleno en la
pila.

> Con ERC también puedo buscar gadgets y construir cadenas, pero para este PoC basta un `jmp esp`.

---

## Paso 5 — Validar el control (PoC de laboratorio)

La estructura de la entrada queda:

```
[ relleno hasta EIP ][ dirección de jmp esp ][ código de control en la pila ]
     offset bytes          4 bytes (x86)
```

Para **validar** el control de ejecución, el estándar didáctico es ejecutar algo inocuo y
visible (abrir `calc.exe` o mostrar un mensaje). Con eso queda demostrado el control sin
construir nada ofensivo.

> Me detengo aquí a propósito: el objetivo del recurso es **enseñar a usar x64dbg** para llegar
> hasta el control de flujo, no entregar un exploit armado.

---

## Qué aporta x64dbg en este caso (y qué no)

| x64dbg me da | x64dbg NO me da |
|--------------|-----------------|
| El valor exacto de `EIP`/`ESP` en el crash | El mapa estático completo del binario (mejor Ghidra/IDA) |
| Inspección de pila y memoria en tiempo real | Decompilación a pseudocódigo |
| Búsqueda de `jmp esp` / gadgets (con ERC) | Diffing con la versión parcheada (eso es BinDiff) |
| Parcheo y prueba inmediata de hipótesis | Depuración de kernel (eso es WinDbg) |

Esto conecta con la idea de la guía: x64dbg es la pieza de **análisis dinámico**, y se integra
con el estático (Ghidra/IDA) y el diffing (BinDiff) para una metodología completa.

---

## Conclusiones

1. x64dbg convierte un crash opaco en información accionable: registros, pila y offset.
2. El plugin **ERC.Xdbg** lo hace especialmente cómodo para exploiting (patrón/offset/gadgets).
3. El control de `EIP` es el punto de inflexión del análisis; a partir de ahí la metodología
   decide cómo continuar según las protecciones.

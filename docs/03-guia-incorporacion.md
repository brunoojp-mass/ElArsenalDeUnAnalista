# 03 — Guía rápida de incorporación

Pensada para un analista que se incorpora al equipo: **cuándo** tirar de x64dbg, **cuándo no**,
qué **evitar** y con qué **combinarla**.

---

## ¿Cuándo usar x64dbg?

Úsala cuando necesites **ver qué hace un binario de Windows en tiempo real**:

- **Análisis dinámico de malware**: comportamiento, APIs llamadas, desempaquetado, persistencia.
- **Desarrollo y depuración de exploits**: análisis de crashes, control de `EIP/RIP`, cálculo de
  offsets, prueba de gadgets.
- **Unpacking**: binarios empaquetados/protegidos (con Scylla + ScyllaHide).
- **Comprensión de lógica interna**: validaciones, cifrado, rutinas que el estático no deja
  claras.
- **Parcheo**: neutralizar chequeos o modificar comportamiento en laboratorio.

---

## ¿Cuándo NO es la mejor opción?

| Necesidad | Mejor herramienta |
|-----------|-------------------|
| Depurar **drivers / kernel** | **WinDbg** (x64dbg es solo **modo usuario**) |
| **Reversing estático** a gran escala, decompilación | **IDA Pro** / **Ghidra** |
| Binarios de **Linux/Unix** | **GDB** (+ GEF/pwndbg) |
| **Diffing** entre versiones de un binario | **BinDiff** / **Diaphora** |
| Análisis de **tráfico de red** | **Wireshark** |
| Pentesting **web** | **Burp Suite** |

> Punto importante que a veces se olvida: **x64dbg es solo modo usuario**. Para algo que corra en
> el kernel, necesitas WinDbg.

---

## Qué EVITAR (errores típicos del que empieza)

- **Usar el ejecutable equivocado**: x32dbg para 32 bits, x64dbg para 64 bits. Si no, no carga
  bien.
- **Ejecutar malware sin aislar**: siempre VM dedicada, snapshot y sin red (o *host-only*).
- **Olvidar los símbolos**: sin símbolos configurados, todo son direcciones y se pierde contexto.
- **No usar ScyllaHide con muestras que detectan debugger**: muchas abortan o cambian de
  comportamiento si notan que las depuras.
- **Abusar del "Run" a ciegas** (`F9` sin breakpoints): pierdes el control; pon breakpoints antes.
- **Confundir breakpoint software y hardware**: en código que verifica su propia integridad, el
  INT3 (software) se detecta; usa hardware o de memoria.

---

## Combinaciones que potencian su utilidad

x64dbg brilla **integrado** en una metodología, no en solitario:

```
   ESTÁTICO                 DINÁMICO                  DIFFING
 Ghidra / IDA   ─────►      x64dbg        ◄─────     BinDiff
 (mapa del       localiza   (confirma en   el parche  (qué cambió
  binario)       funciones   ejecución)    guía dónde  entre versiones)
                 de interés               mirar
            └──────────────── Labelless sincroniza etiquetas ───────────┘

   PLUGINS DE APOYO EN x64dbg:
   ScyllaHide (anti-anti-debug) · Scylla (unpacking) · ERC.Xdbg (exploiting)
```

- **Ghidra/IDA → x64dbg**: identificas la función sospechosa en estático y la confirmas en
  dinámico (Labelless sincroniza las etiquetas).
- **BinDiff → x64dbg**: el diffing te dice *qué* función cambió en un parche; x64dbg te deja ver
  *cómo* se comporta esa función en ejecución.
- **ERC.Xdbg dentro de x64dbg**: patrón cíclico, offset y gadgets ROP para la fase de
  explotación.
- **Scylla + ScyllaHide**: desempaquetar y reconstruir binarios protegidos.

---

## Primeros 15 minutos con la herramienta (ruta exprés)

1. Abre **x32dbg** y carga un binario simple propio (ej.: el `vulnerable.exe` del caso práctico).
2. Observa la ventana **CPU**: localiza los 4 paneles (disasm, registros, dump, pila).
3. Pon un breakpoint con `F2` y ejecuta con `F9`.
4. Avanza con `F7`/`F8` y mira cómo cambian los registros.
5. Abre **Memory Map** (`Alt+M`) y **Breakpoints** (`Alt+B`).
6. Prueba `Ctrl+G` para ir a una dirección y `dump esp` para ver la pila.

Con eso ya tienes la base para el flujo completo de [`02-flujo-trabajo.md`](02-flujo-trabajo.md).

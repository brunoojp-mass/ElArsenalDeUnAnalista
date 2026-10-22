# 05 — Limitaciones, alternativas e integración

Conocer los límites de una herramienta es parte de saber usarla. Aquí comparo x64dbg con sus
alternativas y explico cómo encaja en una metodología de análisis.

---

## Limitaciones de x64dbg

- **Solo modo usuario.** No depura el kernel ni drivers. Para eso: **WinDbg**.
- **Solo Windows.** Para Linux/Unix: **GDB** (con GEF o pwndbg).
- **Dos binarios separados.** x32dbg (32 bits) y x64dbg (64 bits); hay que elegir el correcto.
- **Sin decompilador integrado.** Ofrece desensamblado y grafos, pero no pseudocódigo como
  IDA/Ghidra. Para lógica compleja conviene apoyarse en el estático.
- **Binarios protegidos/empaquetados** requieren plugins (ScyllaHide, Scylla) y trabajo manual;
  no es automático.
- **Análisis estático a gran escala** no es su fuerte: para mapear un binario grande, IDA/Ghidra
  son más eficientes.

---

## x64dbg frente a otras herramientas

| Herramienta | Tipo | Fuerte en | x64dbg frente a ella |
|-------------|------|-----------|----------------------|
| **WinDbg** | Debugger (user+kernel) | Depuración de kernel, volcados, scripting potente | x64dbg es más cómodo en user-mode y más visual |
| **OllyDbg** | Debugger (user, 32 bits) | El clásico de 32 bits | x64dbg es su sucesor espiritual, soporta 64 bits y sigue mantenido |
| **IDA Pro** | Desensamblador/decompilador | Estático avanzado, decompilación, multiarch | x64dbg es dinámico; se complementan (Labelless los conecta) |
| **Ghidra** | Desensamblador/decompilador | Estático gratuito y potente | Igual: estático vs dinámico, se complementan |
| **GDB** | Debugger | Linux/Unix | x64dbg cubre el hueco en Windows con GUI |
| **BinDiff** | Diffing | Comparar dos binarios | x64dbg no hace diffing; BinDiff dice *dónde* mirar |

---

## Dónde encaja x64dbg en la metodología

Mapeado sobre las fases de análisis de vulnerabilidades:

| Fase de la metodología | Papel de x64dbg |
|------------------------|-----------------|
| Identificación del fallo | Reproduce y observa el fallo en ejecución |
| Clasificación | Ayuda a ver si es BOF, UAF, etc., por el comportamiento en memoria |
| Contexto | Memory Map y módulos cargados → protecciones y layout |
| **Exploración del vector** | **Fase estrella**: registros, pila, dump, control de EIP/RIP |
| Construcción del exploit | Offset, gadgets y prueba de PoC (con ERC.Xdbg) |
| Validación | Reproduce de forma determinista; parcheo para comprobar mitigaciones |

> Resumen: x64dbg es la herramienta de **análisis dinámico** que convierte hipótesis en
> observaciones. No sustituye al estático ni al diffing: los **complementa**. Un analista
> competente no elige "una herramienta", sino que **orquesta varias** según la fase.

---

## Regla práctica para elegir

```
¿Es Windows?           NO ──► GDB (+GEF/pwndbg)
   │ SÍ
   ▼
¿Es kernel/driver?     SÍ ──► WinDbg
   │ NO
   ▼
¿Necesito ver el       SÍ ──► x64dbg / x32dbg   ◄── (aquí brilla)
 comportamiento en
 ejecución?
   │ NO
   ▼
¿Solo mapa estático?   SÍ ──► Ghidra / IDA
```
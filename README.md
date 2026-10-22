# El Arsenal de un Analista — x64dbg

> **Máster en Ciberseguridad — Módulo 6: Vulnerabilidades**
> Tarea colaborativa: *El Arsenal de un Analista*
>
> **Autor:** `Bruno Juste Pérez`
> **Repositorio:** `https://github.com/brunoojp-mass/ElArsenalDeUnAnalista`
> **Fecha:** `22/10/2026`

---

## Qué es este recurso

Un recurso técnico **práctico y reutilizable** sobre **x64dbg**, el depurador de modo usuario
para Windows más usado en reversing, análisis de malware y desarrollo de exploits. No es una
copia de la documentación oficial: es la guía que me gustaría tener a mano (y poder enseñar en
una entrevista) para resolver problemas reales con esta herramienta.

Combina tres formatos, tal y como propone el enunciado:

| Formato | Documento | Para qué sirve |
|---------|-----------|----------------|
| **CheatSheet** comentado | [`docs/01-cheatsheet.md`](docs/01-cheatsheet.md) | Atajos, comandos y acciones de referencia rápida |
| **Flujo visual** de trabajo | [`docs/02-flujo-trabajo.md`](docs/02-flujo-trabajo.md) | Cómo se usa de principio a fin en un análisis |
| **Guía de incorporación** | [`docs/03-guia-incorporacion.md`](docs/03-guia-incorporacion.md) | Cuándo usarla, qué evitar y con qué combinarla |

Y, para demostrar uso real y criterio:

| Documento | Contenido |
|-----------|-----------|
| [`docs/04-caso-practico.md`](docs/04-caso-practico.md) | x64dbg aplicado a un caso real: análisis del crash de un Stack Buffer Overflow |
| [`docs/05-limitaciones-alternativas.md`](docs/05-limitaciones-alternativas.md) | Limitaciones frente a otras herramientas e integración en una metodología |
| [`referencias.md`](referencias.md) | Documentación oficial, plugins y recursos |

---

## ¿Qué es x64dbg? (resumen de 30 segundos)

- **Depurador open source** de **modo usuario** para **Windows**, con interfaz gráfica.
- Dos ejecutables: **`x64dbg`** (binarios de 64 bits) y **`x32dbg`** (binarios de 32 bits).
- Permite **ejecutar un binario paso a paso**, poner breakpoints, inspeccionar y modificar
  registros y memoria, trazar la ejecución, parchear instrucciones y analizar cómo se comporta
  un programa en tiempo real — incluso **sin tener el código fuente**.
- Muy popular entre analistas de malware, desarrolladores de exploits y formadores por su
  potencia, facilidad de uso y comunidad activa de **plugins**.

> Es la herramienta de **análisis dinámico** por excelencia en Windows. Se complementa con el
> análisis **estático** (Ghidra / IDA) y con el **diffing** (BinDiff). Ver
> [`docs/05-limitaciones-alternativas.md`](docs/05-limitaciones-alternativas.md).

---

## Cómo usar este repositorio

1. ¿Buscas un atajo o comando concreto? → [`docs/01-cheatsheet.md`](docs/01-cheatsheet.md).
2. ¿Primera vez con la herramienta o con un análisis nuevo? → [`docs/02-flujo-trabajo.md`](docs/02-flujo-trabajo.md).
3. ¿Te incorporas a un equipo y quieres saber cuándo tirar de x64dbg? → [`docs/03-guia-incorporacion.md`](docs/03-guia-incorporacion.md).
4. ¿Quieres ver un uso real de principio a fin? → [`docs/04-caso-practico.md`](docs/04-caso-practico.md).

---

## Entorno de referencia

- **SO:** Windows 10/11 en VM aislada (red *host-only*).
- **Versión de x64dbg:** `Windows 11` (snapshot/build).
- **Plugins instalados:** `Ghidra, x64dbg / x32dbg, BinDiff, AFL++ / script propio, Python 3`.

> Los atajos de teclado y algún comando pueden variar entre
> builds y se pueden reasignar en *Options → Preferences → Hotkeys*. Yo no tengo acceso a
> internet: contrasta los detalles con la documentación oficial (ver `referencias.md`).


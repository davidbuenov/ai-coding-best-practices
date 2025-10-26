# 🐍 Guía de Buenas Prácticas en Python Moderno (3.10+)

Esta guía recoge prácticas profesionales de Python pensadas para trabajar bien con IAs generativas (Copilot, ChatGPT, Gemini, Claude): código claro, tipado, con una sola salida por función y ejemplos BIEN/MAL.

---

## 📑 Tabla de contenidos

- [🎯 Objetivo](#-objetivo)
- [🧩 Requisitos](#-requisitos)
- [🆕 Características de Python Moderno](#-características-de-python-moderno)
  - [1️⃣ Tipado Moderno y Data Models](#1️⃣-tipado-moderno-y-data-models)
  - [2️⃣ Result Pattern (éxito/error tipado)](#2️⃣-result-pattern-éxitoerror-tipado)
  - [3️⃣ UN SOLO return + Guard Clauses](#3️⃣-un-solo-return--guard-clauses)
  - [4️⃣ Context Managers y Recursos](#4️⃣-context-managers-y-recursos)
  - [5️⃣ Match/Case y Enums](#5️⃣-matchcase-y-enums)
  - [6️⃣ Docstrings y Precondiciones](#6️⃣-docstrings-y-precondiciones)
  - [7️⃣ Manejo de Excepciones Específicas](#7️⃣-manejo-de-excepciones-específicas)
- [8️⃣ "Lección del Mundo Real": Versiones](#8️⃣-lección-del-mundo-real-versiones)
- [🤖 Prompt de Sistema para IAs Generativas (Python)](#-prompt-de-sistema-para-ias-generativas-python)
  - [Cómo usar](#cómo-usar)
  - [Versión completa del prompt](#versión-completa-del-prompt)
  - [Versión corta del prompt](#versión-corta-del-prompt)
- [✅ Checklist de Código Python Profesional](#-checklist-de-código-python-profesional)
- [📚 Referencias](#-referencias)

---

## 🎯 Objetivo

- ✅ Legibilidad y mantenibilidad
- ✅ Tipos estáticos con `typing` modernos
- ✅ Manejo explícito de errores (sin silencios)
- ✅ UN SOLO return + guard clauses
- ✅ Prompts listos para proyectos serios y para uso rápido

---

## 🧩 Requisitos

- Python 3.10+ (se usa `|` para uniones, `match`, `dataclasses`, etc.)
- Recomendado: `mypy` y `ruff` para análisis estático/estilo

---

## 🆕 Características de Python Moderno

- Python 3.10: tipos unión con `X | Y` (PEP 604), `match/case` (PEP 634)
- Tipado más expresivo: `TypedDict`, `TypeAlias`, `Literal`, `dataclasses(slots=True)`
- Patrones funcionales y excepciones específicas fomentadas
- Mínimo recomendado: Python 3.10 (3.11+ mejora rendimiento y typing)

---

## 1️⃣ Tipado Moderno y Data Models

```python
from __future__ import annotations
from dataclasses import dataclass
from typing import TypedDict, Literal, TypeAlias, Optional

# ✅ BIEN: TypeAlias, Literal, TypedDict, dataclass
UserId: TypeAlias = int
Rol = Literal["admin", "user", "guest"]

class UsuarioDTO(TypedDict):
    id: UserId
    nombre: str
    rol: Rol

@dataclass(slots=True)
class Usuario:
    id: UserId
    nombre: str
    rol: Rol = "user"
```

```python
# ❌ MAL: sin tipos ni modelos claros
usuario = {"id": 1, "nombre": "Ana", "rol": "loquesea"}  # rol inválido
```

---

## 2️⃣ Result Pattern (éxito/error tipado)

```python
from dataclasses import dataclass
from typing import Generic, TypeVar, Union

T = TypeVar("T")

@dataclass(slots=True)
class Ok(Generic[T]):
    value: T

@dataclass(slots=True)
class Err:
    message: str
    cause: Exception | None = None

Result: TypeAlias = Ok[T] | Err

# ✅ BIEN: funciones que retornan Result

def parse_int(texto: str) -> Result[int]:
    try:
        return Ok(int(texto))
    except ValueError as e:
        return Err("No es un entero", e)
```

```python
# ❌ MAL: retornar None o lanzar sin control
def parse_int_mal(texto: str):
    return int(texto)  # lanza ValueError sin contexto
```

---

## 3️⃣ UN SOLO return + Guard Clauses

```python
# ✅ BIEN: validaciones planas y un único return
from typing import Sequence

def promedio(xs: Sequence[float]) -> float:
    resultado: float
    if not xs:
        resultado = 0.0
    else:
        total = sum(xs)
        resultado = total / len(xs)
    return resultado  # UN SOLO return
```

```python
# ❌ MAL: múltiples returns dispersos
def promedio_mal(xs: list[float]) -> float:
    if not xs:
        return 0.0
    return sum(xs) / len(xs)
```

---

## 4️⃣ Context Managers y Recursos

```python
# ✅ BIEN: with para archivos
from pathlib import Path

def leer_lineas(ruta: str) -> list[str]:
    path = Path(ruta)
    with path.open("r", encoding="utf-8") as f:
        return [l.strip() for l in f if l.strip()]
```

```python
# ❌ MAL: abrir/cerrar manualmente
def leer_lineas_mal(ruta: str) -> list[str]:
    f = open(ruta, "r", encoding="utf-8")
    data = [l.strip() for l in f if l.strip()]
    f.close()
    return data
```

---

## 5️⃣ Match/Case y Enums

```python
from enum import Enum

class Estado(Enum):
    OK = "ok"
    ERROR = "error"


def describir(obj: object) -> str:
    match obj:
        case str(s):
            return f"Texto: {s.upper()}"
        case int(n):
            return f"Número: {n * 2}"
        case Estado.OK:
            return "Todo bien"
        case Estado.ERROR:
            return "Algo falló"
        case None:
            return "Nulo"
        case _:
            return f"Tipo: {type(obj).__name__}"
```

---

## 6️⃣ Docstrings y Precondiciones

```python
# ✅ BIEN: Google-style docstring con validaciones
from typing import Iterable


def filtrar_positivos(xs: Iterable[int]) -> list[int]:
    """Devuelve los enteros positivos de xs.

    Args:
        xs: Iterable de enteros

    Returns:
        Lista con los valores > 0 en el mismo orden

    Raises:
        TypeError: Si xs no es iterable de int
    """
    resultado: list[int]
    try:
        resultado = [x for x in xs if x > 0]
    except TypeError as e:
        raise TypeError("xs debe ser Iterable[int]") from e
    return resultado
```

---

## 7️⃣ Manejo de Excepciones Específicas

```python
class UsuarioNoEncontrado(Exception):
    pass

class EmailDuplicado(Exception):
    pass


def crear_usuario(email: str, repo) -> Usuario:
    """Crea un usuario si no existe.

    Raises:
        EmailDuplicado: si el email ya existe
    """
    resultado: Usuario
    if repo.exists(email):
        raise EmailDuplicado(email)
    else:
        resultado = repo.save(Usuario(id=0, nombre=email.split("@")[0], rol="user"))
    return resultado
```

---

## 8️⃣ "Lección del Mundo Real": Versiones

En este proyecto detectamos una discrepancia entre el README (decía 3.8+) y el código (usaba 3.10+). Conclusión:

- Establece versión mínima realista (3.10+) y valídala al inicio de los notebooks/scripts.
- Evita usar sintaxis de versiones superiores sin documentarlo.

```python
import sys

MIN = (3, 10)
if sys.version_info < MIN:
    raise RuntimeError(f"Se requiere Python {MIN[0]}.{MIN[1]}+")
```

---

## 🤖 Prompt de Sistema para IAs Generativas (Python)

### Cómo usar

Usa la versión completa para proyectos y la corta para peticiones rápidas.

### Versión completa del prompt

````markdown
═══════════════════════════════════════════════════════════════════
🔽 INICIO DEL PROMPT - Copia desde aquí 🔽
═══════════════════════════════════════════════════════════════════

# Prompt de Sistema para Generación de Código Python 3.10+ Profesional

Eres un asistente Python que genera código limpio, tipado y mantenible.

## Reglas OBLIGATORIAS (Python 3.10+)

1. Tipado moderno
   - ✅ Usa `|` para uniones, `TypedDict`, `TypeAlias`, `Literal`, `dataclass(slots=True)`
   - ✅ Anota tipos en entradas, salidas y variables importantes

2. Result pattern
   - ✅ Retorna `Ok[T] | Err` en funciones con error recuperable
   - ✅ No retornes `None` ambiguo; no silencies excepciones

3. UN SOLO return + Guard Clauses
   - ✅ Un único `return` por función
   - ✅ Validaciones planas (sin pirámide de ifs)

4. Recursos y errores
   - ✅ Usa context managers (`with`)
   - ✅ Lanza excepciones específicas con contexto

5. Estilo y docs
   - ✅ `ruff` + `mypy` recomendados
   - ✅ Docstrings estilo Google con Args/Returns/Raises

## Ejemplo correcto

```python
from dataclasses import dataclass
from typing import TypeAlias, TypeVar, Generic

T = TypeVar("T")

@dataclass(slots=True)
class Ok(Generic[T]):
    value: T

@dataclass(slots=True)
class Err:
    message: str

Result: TypeAlias = Ok[T] | Err

def dividir(a: float, b: float) -> Result[float]:
    resultado: Result[float]
    if b == 0:
        resultado = Err("División por cero")
    else:
        resultado = Ok(a / b)
    return resultado  # UN SOLO return
```

## Importante

- Python 3.10+ mínimo
- UN función = UN solo return
- Prefiere claridad, tipado y manejo explícito de errores

═══════════════════════════════════════════════════════════════════
🔼 FIN DEL PROMPT - Copia hasta aquí 🔼
═══════════════════════════════════════════════════════════════════
````

---

### Versión corta del prompt

````markdown
═══════════════════════════════════════════════════════════════════
🔽 INICIO DEL PROMPT CORTO - Copia desde aquí 🔽
═══════════════════════════════════════════════════════════════════

Genera Python 3.10+ profesional con estas reglas:
1) Tipado moderno (|, TypedDict, dataclass(slots))
2) Result pattern (Ok[T] | Err), sin None ambiguo
3) UN SOLO return + guard clauses
4) with para recursos; excepciones específicas
5) Docstrings con Args/Returns/Raises

Ejemplo:
```python
from dataclasses import dataclass
@dataclass(slots=True)
class Ok: value: int
@dataclass(slots=True)
class Err: message: str

def parse(s: str):
    if not s.isdigit(): return Err("No entero")
    return Ok(int(s))
```

═══════════════════════════════════════════════════════════════════
🔼 FIN DEL PROMPT CORTO - Copia hasta aquí 🔼
═══════════════════════════════════════════════════════════════════
````

---

## ✅ Checklist de Código Python Profesional

- [ ] ¿Anoté tipos (entradas/salidas) y usé `|`, `TypedDict`, `Literal`, `TypeAlias`?
- [ ] ¿Modelé estados con `Ok[T] | Err` cuando aplica?
- [ ] ¿Cada función tiene UN SOLO return y guard clauses?
- [ ] ¿Usé context managers para recursos?
- [ ] ¿Documenté con docstrings (Args/Returns/Raises)?
- [ ] ¿Usé excepciones específicas con contexto?
- [ ] ¿Pasé `ruff` y `mypy` (o equivalente)?

---

## 📚 Referencias

- [PEP 484 – Type Hints](https://peps.python.org/pep-0484/)
- [PEP 604 – X | Y Union Types](https://peps.python.org/pep-0604/)
- [PEP 557 – Data Classes](https://peps.python.org/pep-0557/)
- [typing — docs](https://docs.python.org/3/library/typing.html)
- [dataclasses — docs](https://docs.python.org/3/library/dataclasses.html)

---

## 📘 Sobre esta guía

Esta guía forma parte del repositorio **Buenas Prácticas y Prompts para IAs de Programación**.

👉 **Ver más guías**: [Repositorio completo](../README.md)

---

**Autor**: [David Bueno Vallejo](https://davidbuenov.com/) | [LinkedIn](https://www.linkedin.com/in/davidbueno/) | [GitHub](https://github.com/davidbuenov)

**Licencia**: MIT

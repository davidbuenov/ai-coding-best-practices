# 📚 Guía de Buenas Prácticas y Prompts para IAs de Programación

Este documento agrupa en un solo lugar las guías y prompts para generar código profesional con IAs (Copilot, ChatGPT, Gemini, Claude). Está pensado como el README principal de un repositorio específico de "buenas prácticas + prompts".

---

## Tabla de contenidos

- [Proposito](#proposito)
- [Contenido](#contenido)
- [Guias por Lenguaje](#guias-por-lenguaje)
- [Requisitos de Versiones](#requisitos-de-versiones)
- [Estructura actual del repositorio (propuesta aplicada)](#estructura-actual-del-repositorio-propuesta-aplicada)
- [Autor](#-autor)
- [Agradecimientos Especiales](#-agradecimientos-especiales)
- [Licencia](#-licencia)

---

## Proposito

- Ayudar a estudiantes y profesionales a producir código moderno, limpio y mantenible.
- Estandarizar reglas que faciliten a las IAs sugerir y modificar código correctamente.
- Proveer prompts con delimitadores para un copiado/pegado sin errores.

---

## Contenido

- Guías por lenguaje con ejemplos BIEN/MAL
- Prompts (versión completa en cada guía)
- Requisitos de versiones
- Estructura del repositorio

---

## Guias por Lenguaje

- Java (17+/21): `java/buenaspracticas-java.md`
  - Records, sealed classes, pattern matching, Optional, Streams, try-with-resources
  - Regla de UN SOLO return + guard clauses
  - Incluye prompt completo y corto con delimitadores

- C++ (C++20/23): `cpp/buenaspracticas-cpp.md`
  - RAII, smart pointers, enum class, using, string_view, span, ranges
  - optional/variant/expected, UN SOLO return + guard clauses
  - Incluye prompt completo y corto con delimitadores
  - Unreal Engine 5.6 (C++): `cpp/buenaspracticas-unreal-cpp.md`
    - UCLASS/UPROPERTY/UFUNCTION, GC y referencias seguras, componentes, timers y OnRep
    - Replicación básica con RPCs, logging/check/ensure, Build.cs y categorías de log

- Web (HTML/CSS/JS): `web/buenaspracticas-web.md`
  - HTML semántico + a11y (WCAG 2.2), CSS moderno (variables, clamp, reduced-motion), JS ESM sin inline
  - Seguridad (CSP, sanitización, cookies seguras, CSRF), rendimiento (imágenes modernas, fuentes swap)
  - Incluye prompt completo y corto con delimitadores

- TypeScript + Node.js: `typescript/buenaspracticas-typescript.md`
  - TS estricto, ESM, uniones discriminadas/Result, Zod en entradas, AbortController/timeout
  - Error cause + logging estructurado, env validado, Vitest/ESLint/Prettier, TSDoc

- SQL (general: PostgreSQL/MySQL/SQLite): `sql/buenaspracticas-sql.md`
  - Consultas parametrizadas, CTEs/window functions, índices, EXPLAIN/ANALYZE, paginación keyset, migraciones

- Oracle SQL/PLSQL: `sql/oracle/buenaspracticas-oracle-sql-plsql.md`
  - Binds, paquetes SPEC/BODY, excepciones propias, BULK COLLECT/FORALL, DBMS_XPLAN/TKPROF

- C# (.NET 8+): `csharp/buenaspracticas-csharp.md`
  - Nullable reference types, records, async/await+CT, DI+ILogger, Result pattern, XML docs

- C# Unity 6.1: `csharp/unity/buenaspracticas-unity-csharp.md`
  - MonoBehaviour, ScriptableObjects, Jobs+Burst, ECS, [SerializeField], un solo punto de salida

- Python (3.10+): `python/buenaspracticas-python.md`
  - Type hints modernos, funciones puras, manejo de errores con Result-pattern
  - Validaciones de versión y notas pedagógicas
  - Incluye prompt completo y corto con delimitadores

---

---

## Requisitos de Versiones

- Java: 17+ (ideal 21 LTS)
- C++: C++20+ (ideal C++23)
- C#: .NET 8+; Unity 6.1
- Python: 3.10+
- TypeScript: 5.x + Node 20+
- SQL: PostgreSQL/MySQL/SQLite modernos; Oracle 12c+

---

## Estructura actual del repositorio (propuesta aplicada)

- `/java/buenaspracticas-java.md`
- `/cpp/buenaspracticas-cpp.md`
- `/cpp/buenaspracticas-unreal-cpp.md`
- `/web/buenaspracticas-web.md`
- `/typescript/buenaspracticas-typescript.md`
- `/sql/buenaspracticas-sql.md`
- `/sql/oracle/buenaspracticas-oracle-sql-plsql.md`
- `/csharp/buenaspracticas-csharp.md`
- `/csharp/unity/buenaspracticas-unity-csharp.md`
- `/python/buenaspracticas-python.md`
- `/docs/` (diagrama y notas pedagógicas)

---

## 👨‍🏫 Autor

**David Bueno Vallejo** — Profesor universitario de informática, apasionado por la enseñanza práctica, la IA agentic y la innovación educativa.

- 🌐 Web: <https://davidbuenov.com/>
- 💼 LinkedIn: <https://www.linkedin.com/in/davidbueno/>
- 🎥 YouTube: <https://www.youtube.com/user/davidbueno>
- 💻 GitHub: <https://github.com/davidbuenov>

---

## ✨ Agradecimientos Especiales

Un agradecimiento especial a las IAs que colaboraron en la creación y mejora de este proyecto: **Gemini de Google** y **Copilot de GitHub**. Su asistencia fue fundamental para la depuración de código, la generación de explicaciones y la elaboración de esta documentación.

---

## 📄 Licencia

Este proyecto está bajo licencia MIT. Puedes usarlo, modificarlo y compartirlo libremente con fines educativos o personales.

---

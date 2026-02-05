# 📚 Guía de Buenas Prácticas y Prompts para IAs de Programación

Este documento agrupa en un solo lugar las guías y prompts para generar código profesional con IAs (Copilot, ChatGPT, Gemini, Claude). Está pensado como el README principal de un repositorio específico de "buenas prácticas + prompts".

---

## Tabla de contenidos

- [Proposito](#proposito)
- [La Filosofía: Colaboración Abierta](#la-filosofía-colaboración-abierta)
  - [Tu Opinión Importa](#tu-opinión-importa)
  - [Cómo Puedes Contribuir](#cómo-puedes-contribuir)
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

## La Filosofía: Colaboración Abierta

Este repositorio **no pretende ser la verdad absoluta** ni un manual cerrado. Al contrario, es un proyecto vivo que debe evolucionar con:

- Las nuevas versiones de lenguajes y frameworks
- Las tendencias emergentes en desarrollo de software
- Las experiencias y conocimientos de la comunidad
- Los casos de uso reales que encuentran los desarrolladores

### Tu Opinión Importa

Estoy completamente abierto a:

✅ **Sugerencias de mejora**: ¿Crees que falta algo importante? ¿Hay alguna práctica que debería añadirse o modificarse?

✅ **Nuevos lenguajes o frameworks**: ¿Trabajas con tecnologías que no están incluidas? Me encantaría expandir el repositorio.

✅ **Correcciones**: Si encuentras algo incorrecto o desactualizado, por favor házmelo saber.

✅ **Casos de uso**: Comparte cómo estás utilizando estas guías y qué resultados estás obteniendo.

✅ **Pull requests**: El repositorio está abierto a contribuciones directas. Si quieres añadir contenido o mejorar algo, ¡adelante!

### Cómo Puedes Contribuir

Hay varias formas de participar:

1. **Abre un issue** en GitHub con tus sugerencias o comentarios
2. **Crea un pull request** con tus mejoras o nuevas guías
3. **Comparte tu experiencia** sobre cómo estás usando estas guías con herramientas de IA
4. **Dale una estrella** ⭐ al repositorio si te parece útil
5. **Difunde el proyecto** para que llegue a más desarrolladores

---

## Contenido

- Guías por lenguaje con ejemplos BIEN/MAL
- Prompts (versión completa en cada guía)
- Requisitos de versiones
- Estructura del repositorio

---


## Guias por Lenguaje

- Java 17 (base): `java/buneaspracticas-java17.md`
  - Records, sealed classes, pattern matching, Optional, Streams, try-with-resources
  - Regla de UN SOLO return + guard clauses
  - Incluye prompt completo y corto con delimitadores
- Java 21 (UMA · primeros cursos): `java/buenaspracticas-java21-UMA.md`
  - Enfoque didáctico con ejemplos BIEN/MAL y código típico de IA
  - Explicación simple de Java 21 y checklist
- Java 21 (Pro): `java/buenaspracticas-java21-pro.md`
  - Reglas profesionales + concurrencia moderna + prompt reforzado
- Java 21 (Pruebas unitarias): `java/buenaspracticas-java21-pruebasunitarias.md`
  - AAA, given-when-then, JUnit 5, Mockito, @DisplayName, @Nested
- Java 25 (Pro): `java/buenaspracticas-java25-pro.md`
  - Novedades desde JDK 21 con ejemplos y guía profesional

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

- Node.js (JavaScript, Node 20+): `nodejs/buenaspracticas-nodejs.md`
  - ESM por defecto, validación de entorno (Zod), logging estructurado (pino), Result pattern Ok/Err
  - Seguridad básica (helmet, rate limit), AbortController para timeouts, graceful shutdown
- Node.js Firebase Cloud Functions (JavaScript CommonJS 2nd gen): `nodejs/buenaspracticas-nodejs-firebase.md`
  - Adaptación a entorno serverless, CommonJS, functions.logger, cold start, idempotencia, JSDoc
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


- PHP (8.2–8.4): `php/buenaspracticas-php.md`
  - `strict_types=1`, tipos unión/intersección, propiedades tipadas, readonly, promoted props
  - Enums (8.1+), match, nullsafe, `#[Override]` (8.3+), PDO con transacciones y SQL parametrizado
  - Incluye prompt completo y corto con delimitadores

- Mermaid (diagramas como código): `mermaid/buenaspracticas-mermaid.md`
  - Flowcharts, sequence, gantt, class, ER, state, pie, quadrant, timeline, user journey
  - Prompts para IA, integración con Markdown, GitHub, VS Code, Notion, Docusaurus, MkDocs
  - Checklist, ejemplos BIEN/MAL, validación y preview

---

---

## Requisitos de Versiones

- Java: 17+ (ideal 25 LTS; 21 LTS válido)
- C++: C++20+ (ideal C++23)
- C#: .NET 8+; Unity 6.1
- Python: 3.10+
- TypeScript: 5.x + Node 20+
- Node.js: 20+ (fetch nativo, AbortController, test runner)
- SQL: PostgreSQL/MySQL/SQLite modernos; Oracle 12c+
- PHP: 8.2+ (ideal 8.4)

---

## Estructura actual del repositorio (propuesta aplicada)

- `/java/buneaspracticas-java17.md`
- `/java/buenaspracticas-java21-UMA.md`
- `/java/buenaspracticas-java21-pro.md`
- `/java/buenaspracticas-java21-pruebasunitarias.md`
- `/java/buenaspracticas-java25-pro.md`
- `/cpp/buenaspracticas-cpp.md`
- `/cpp/buenaspracticas-unreal-cpp.md`
- `/web/buenaspracticas-web.md`
- `/typescript/buenaspracticas-typescript.md`
- `/sql/buenaspracticas-sql.md`
- `/sql/oracle/buenaspracticas-oracle-sql-plsql.md`
- `/csharp/buenaspracticas-csharp.md`
- `/csharp/unity/buenaspracticas-unity-csharp.md`
- `/python/buenaspracticas-python.md`
- `/php/buenaspracticas-php.md`
- `/nodejs/buenaspracticas-nodejs.md`
- `/nodejs/buenaspracticas-nodejs-firebase.md`
- `/mermaid/buenaspracticas-mermaid.md`

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

# ☕ Buenas Prácticas Java 25 (Profesional)

Guía profesional para Java 25 LTS, con **todas las novedades relevantes** integradas desde Java 21 y recomendaciones de uso en proyectos reales.

> Fuente oficial: OpenJDK JDK 25 y listado de JEPs desde JDK 21.

---

## 📑 Tabla de contenidos

- [🎯 Objetivo](#-objetivo)
- [🆕 Novedades Java 25 (desde JDK 21)](#-novedades-java-25-desde-jdk-21)
  - [Lenguaje](#lenguaje)
  - [Bibliotecas](#bibliotecas)
  - [Runtime/HotSpot](#runtimehotspot)
  - [GC](#gc)
  - [JFR](#jfr)
  - [Seguridad y Criptografía](#seguridad-y-criptografía)
  - [Herramientas](#herramientas)
  - [Integrity by Default](#integrity-by-default)
  - [Preview & Incubating](#preview--incubating)
  - [Deprecaciones y Eliminaciones](#deprecaciones-y-eliminaciones)
  - [🧱 Buenas Prácticas Base (Java 21+)](#-buenas-prácticas-base-java-21)
  - [🧰 Programación Funcional y Streams (uso correcto)](#-programación-funcional-y-streams-uso-correcto)
  - [🧵 Concurrencia Moderna](#-concurrencia-moderna)
- [✅ Recomendaciones Profesionales](#-recomendaciones-profesionales)
- [📋 Prompt de Sistema para IAs Generativas (Java 25 Pro)](#-prompt-de-sistema-para-ias-generativas-java-25-pro)
  - [🤖 Cómo Usar](#-cómo-usar)
  - [📄 Versión Completa del Prompt](#-versión-completa-del-prompt)
  - [⚡ Versión Corta del Prompt (Uso Rápido)](#-versión-corta-del-prompt-uso-rápido)
- [✅ Checklist Profesional](#-checklist-profesional)
- [📚 Referencias](#-referencias)

---

## 🎯 Objetivo

- ✅ Código limpio, mantenible y con features actuales de Java 25.
- ✅ Uso consciente de previews/incubators y estabilidad en producción.
- ✅ Mejoras de rendimiento y observabilidad aplicadas con criterio.

---

## 🆕 Novedades Java 25 (desde JDK 21)

> **Nota**: Los ejemplos de *Preview/Incubator* se señalan claramente. No los uses en producción sin decisión explícita.

### Lenguaje

**JEP 512: Compact Source Files and Instance Main Methods (25)** — permite scripts y demos con un `main` más simple, reduciendo boilerplate cuando no necesitas una clase pública.

```java
// ✅ main simplificado (sin clase explícita)
void main() {
  System.out.println("Hola, Java 25");
}
```

**JEP 511: Module Import Declarations (25)** — facilita declarar dependencias de módulos de forma explícita y más clara en proyectos modulares.

```java
// ✅ importación de módulos
import module java.sql;
import module java.logging;
```

**JEP 513: Flexible Constructor Bodies (25)** — permite preparar o transformar datos antes de llamar a `super(...)`, mejorando legibilidad y validaciones.

```java
// ✅ puedes preparar datos antes de super(...)
class Base { Base(String x) {} }
class Sub extends Base {
  Sub(String raw) {
    String limpio = raw.trim();
    super(limpio);
  }
}
```

**JEP 456: Unnamed Variables & Patterns (22)** — ignora variables que no necesitas y evita nombres “de relleno”, haciendo el código más limpio.

```java
// ✅ ignora valores que no necesitas
if (obj instanceof Punto(int x, int _)) {
  System.out.println(x);
}
```

### Bibliotecas

**JEP 506: Scoped Values (25)** — alternativa segura a `ThreadLocal` para propagar contexto sin fugas ni acoplamientos peligrosos.

```java
// ✅ alternativa segura a ThreadLocal
static final ScopedValue<String> USUARIO = ScopedValue.newInstance();

void ejecutar() {
  ScopedValue.where(USUARIO, "ana").run(() -> {
    log(USUARIO.get());
  });
}
```

**JEP 485: Stream Gatherers (24)** — añade operaciones avanzadas en streams (ventanas, agrupaciones) de forma declarativa y legible.

```java
// ✅ agrupar en ventanas (ejemplo simple)
List<List<Integer>> lotes = numeros.stream()
  .gather(Gatherers.windowFixed(3))
  .toList();
```

**JEP 484: Class-File API (24)** — API estándar para leer y escribir bytecode, útil en tooling y análisis avanzados.

```java
// ✅ inspección de bytecode (uso avanzado)
// ClassFile cf = ClassFile.of();
// cf.read(classBytes); // ejemplo conceptual
```

**JEP 454: Foreign Function & Memory API (22)** — acceso seguro a memoria y llamadas nativas sin JNI, pensado para alto rendimiento.

```java
// ✅ llamada nativa (uso avanzado)
// Linker linker = Linker.nativeLinker();
// ... configuración de funciones nativas
```

### Runtime/HotSpot

**JEP 519: Compact Object Headers (25)** — reduce el tamaño de cabeceras de objetos para mejorar memoria sin cambiar el código.

**JEP 514 / 515 / 483** — mejoran arranque y warmup mediante AOT y perfiles, normalmente sin cambios en el código fuente.

**JEP 491: Synchronize Virtual Threads without Pinning (24)** — reduce bloqueos al sincronizar, mejorando virtual threads en I/O.

### GC

Mejoras del recolector (no requieren cambios de código, pero sí **monitorización**):

- **JEP 521: Generational Shenandoah (25)**
- **JEP 474 / 490: ZGC generacional por defecto (23/24)**
- **JEP 475 / 423: mejoras en G1 (24/22)**

### JFR

Observabilidad mejorada con JFR (sin tocar lógica de negocio):

- **JEP 518: JFR Cooperative Sampling (25)**
- **JEP 509: JFR CPU-Time Profiling (Experimental) (25)**
- **JEP 520: JFR Method Timing & Tracing (25)**

### Seguridad y Criptografía

**JEP 510: Key Derivation Function API (25)** — API estándar para derivar claves seguras (ej. HKDF) sin bibliotecas externas.

```java
// ✅ uso típico (derivación de clave)
// var kdf = KeyDerivationFunction.of("HKDF");
// byte[] clave = kdf.deriveKey(...);
```

- **JEP 497 / 496: algoritmos resistentes a cuántica (24)**
- **JEP 486: Security Manager deshabilitado permanentemente (24)**

### Herramientas

**JEP 458: Launch Multi-File Source-Code Programs (22)** — ejecuta múltiples archivos fuente sin compilación manual previa.

```bash
# ✅ ejecutar varios .java sin compilación explícita
java src/Main.java src/Util.java
```

**JEP 467: Markdown Documentation Comments (23)** — permite escribir Javadoc con Markdown para documentación más clara.

```java
/// ## Descripción
/// - Lista de pasos
/// - Markdown en Javadoc
```

**JEP 493: Linking Run-Time Images without JMODs (24)** — simplifica `jlink` y la creación de runtimes personalizados.

Mejora `jlink` para imágenes runtime (sin cambios de código).

### Integrity by Default

- **JEP 472: Prepare to Restrict the Use of JNI (24)**
- **JEP 498: Warn upon Use of Memory-Access Methods in sun.misc.Unsafe (24)**

**Recomendación**: evita JNI/Unsafe salvo necesidad real y documentada.

### Preview & Incubating

> **Importante**: requiere `--enable-preview` cuando aplique.

**JEP 507: Primitive Types in Patterns, instanceof, and switch (Third Preview) (25)** — extiende pattern matching para usar primitivos de forma directa.

```java
// ✅ preview: patrones con primitivos
static String tipo(int n) {
  return switch (n) {
    case int i -> "int: " + i;
  };
}
```

**JEP 505: Structured Concurrency (Fifth Preview) (25)** — organiza tareas concurrentes en un bloque estructurado con manejo correcto de errores.

```java
// ✅ preview: tareas estructuradas
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
  var t1 = scope.fork(() -> servicio1());
  var t2 = scope.fork(() -> servicio2());
  scope.join();
  scope.throwIfFailed();
  return List.of(t1.get(), t2.get());
}
```

**JEP 502: Stable Values (Preview) (25)** — valores asignables una vez, útiles para caches y configuración segura.

```java
// ✅ preview: valores estables
// StableValue<String> token = StableValue.of();
// token.set("abc");
```

**JEP 508: Vector API (Tenth Incubator) (25)** — API para vectorización explícita en cálculos numéricos.

```java
// ✅ incubator: vectorización
// var va = IntVector.fromArray(SPECIES, a, 0);
```

**JEP 470: PEM Encodings of Cryptographic Objects (Preview) (25)** — soporte estándar para PEM en claves y certificados.

```java
// ✅ preview: PEM
// Pem.encode(key);
```

### Deprecaciones y Eliminaciones

- **JEP 503: Remove the 32-bit x86 Port (25)**
- **JEP 479: Remove the Windows 32-bit x86 Port (24)**
- **JEP 471: Deprecate the Memory-Access Methods in sun.misc.Unsafe for Removal (23)**
- **JEP 501: Deprecate the 32-bit x86 Port for Removal (24)**

---

## ✅ Recomendaciones Profesionales

1. **No uses features Preview/Incubator en producción sin decisión explícita.**
2. **Documenta si el proyecto requiere `--enable-preview`.**
3. **Adopta Scoped Values con criterios claros** (evita `ThreadLocal` sin necesidad).
4. **Usa Stream Gatherers cuando aporten claridad; evita pipelines gigantes.**
5. **Aprovecha JFR** para diagnosticar rendimiento y latencias reales.
6. **Usa el main simplificado** en scripts o demos; en producción, mantén estructura clara.
7. **Para salida/IO simple**, usa `System.out.println`, `printf` y *text blocks* cuando mejoren legibilidad.
8. **Elimina dependencias de 32-bit** y revisa pipelines CI/CD.
9. **Revisa dependencias JNI/Unsafe** por las restricciones futuras.

---

## 🧱 Buenas Prácticas Base (Java 21+)

Estas prácticas siguen siendo clave en Java 25.

### 1) Records para DTOs y Value Objects

```java
public record Usuario(String nombre, String email, int edad) {}
```

### 2) Sealed Classes para jerarquías controladas

```java
public sealed interface Resultado<T> permits Exito, Error {}
public record Exito<T>(T valor) implements Resultado<T> {}
public record Error<T>(String mensaje) implements Resultado<T> {}
```

### 3) Pattern Matching + Switch Expressions

```java
public String procesar(Object entrada) {
  return switch (entrada) {
    case String s -> s.toUpperCase();
    case Integer i -> "#" + i;
    case null -> "NULO";
    default -> "OTRO";
  };
}
```

### 4) Optional en lugar de null

```java
public Optional<Usuario> buscar(String email) {
  return repository.findByEmail(email);
}
```

### 5) Streams API para colecciones

```java
List<String> emails = usuarios.stream()
  .filter(Usuario::activo)
  .map(Usuario::email)
  .sorted()
  .toList();
```

### 6) Try-with-Resources

```java
try (var reader = Files.newBufferedReader(Path.of(ruta))) {
  return reader.lines().toList();
}
```

### 7) SRP + Guard Clauses con un solo return

```java
public Resultado<Usuario> crearUsuario(String nombre, String email, int edad) {
  Resultado<Usuario> resultado;
  if (nombre == null || nombre.isBlank()) {
    resultado = new Error<>("Nombre requerido");
  } else if (email == null || !email.contains("@")) {
    resultado = new Error<>("Email inválido");
  } else if (edad < 18) {
    resultado = new Error<>("Debe ser mayor de edad");
  } else {
    Usuario u = new Usuario(nombre, email, edad);
    repository.save(u);
    resultado = new Exito<>(u);
  }
  return resultado;
}
```

### 8) Complejidad ciclomática ≤ 10

Si un método supera 10, se divide en funciones más pequeñas.

---

## 🧰 Programación Funcional y Streams (uso correcto)

### ✅ Usa Streams para transformaciones, no para efectos secundarios

```java
// ✅ BIEN: transformación pura
List<String> nombres = usuarios.stream()
  .filter(Usuario::activo)
  .map(Usuario::nombre)
  .sorted()
  .toList();
```

```java
// ❌ MAL: efectos secundarios dentro de map
usuarios.stream()
  .map(u -> { log.info(u.email()); return u; })
  .toList();
```

### ✅ Usa forEach solo para efectos secundarios finales

```java
usuarios.forEach(u -> log.info(u.email()));
```

### ✅ Prefiere métodos terminales claros

- `anyMatch`, `allMatch`, `noneMatch` en lugar de bucles manuales.
- `findFirst`, `findAny` cuando solo necesitas uno.

```java
boolean hayInactivos = usuarios.stream().anyMatch(u -> !u.activo());
```

### ✅ Usa `Optional` con API fluida

```java
String email = usuarioOpt
  .map(Usuario::email)
  .orElse("sin-email");
```

### ✅ Evita streams gigantes: divide y nombra pasos

```java
Stream<Usuario> activos = usuarios.stream().filter(Usuario::activo);
List<String> emails = activos.map(Usuario::email).toList();
```

---

## 🧵 Concurrencia Moderna

### ✅ Virtual Threads (cuando haya muchas tareas I/O)

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
  executor.submit(() -> servicio.procesar());
}
```

### ✅ Estructurar tareas en unidades pequeñas

- Evita estados compartidos.
- Prefiere inmutabilidad.
- Protege recursos críticos.

---

## 📋 Prompt de Sistema para IAs Generativas (Java 25 Pro)

### 🤖 Cómo Usar

Usa el prompt completo para proyectos profesionales con Java 25.

### 📄 Versión Completa del Prompt

````markdown
═══════════════════════════════════════════════════════════════════
🔽 INICIO DEL PROMPT - Copia desde aquí 🔽
═══════════════════════════════════════════════════════════════════

Eres un asistente experto en Java 25 profesional.

Reglas obligatorias:
1) Usa records para DTOs y value objects.
2) Usa sealed classes para jerarquías cerradas.
3) Usa pattern matching y switch expressions.
4) Usa Optional<T> en lugar de null.
5) Usa Streams API para transformaciones.
6) Usa try-with-resources para todo recurso.
7) Cada método: una sola responsabilidad y un solo return.
8) Mantén complejidad ciclomática ≤ 10.
9) Javadoc completo en APIs públicas.
10) Excepciones específicas con mensajes claros.
11) Concurrencia moderna: virtual threads cuando proceda.

Reglas de calidad:
12) No inventes clases/métodos/APIs inexistentes.
13) Código completo y compilable.
14) Si hay ambigüedad, pregunta antes.
15) Evita efectos secundarios en streams.

Reglas sobre Java 25:
16) No uses Preview/Incubator sin mencionarlo y justificarlo.
17) Si usas `--enable-preview`, indícalo explícitamente.
18) Considera Scoped Values frente a ThreadLocal.
19) Evita JNI/Unsafe salvo necesidad justificada.

Salida: código claro, legible, profesional, con explicación breve.

═══════════════════════════════════════════════════════════════════
🔼 FIN DEL PROMPT - Copia hasta aquí 🔼
═══════════════════════════════════════════════════════════════════
````

---

### ⚡ Versión Corta del Prompt (Uso Rápido)

````markdown
═══════════════════════════════════════════════════════════════════
🔽 INICIO DEL PROMPT CORTO - Copia desde aquí 🔽
═══════════════════════════════════════════════════════════════════

Java 25 pro: records, sealed, pattern matching, Optional, streams, try-with-resources, SRP, un return, complejidad ≤10, Javadoc, excepciones específicas, virtual threads. No inventes APIs. Código compilable. Preview/incubator solo si se justifica e indicando --enable-preview.

═══════════════════════════════════════════════════════════════════
🔼 FIN DEL PROMPT CORTO - Copia hasta aquí 🔼
═══════════════════════════════════════════════════════════════════
````

---

## ✅ Checklist Profesional

- [ ] ¿Uso records para DTOs y value objects?
- [ ] ¿Uso sealed classes para jerarquías cerradas?
- [ ] ¿Uso pattern matching + switch expressions?
- [ ] ¿Uso Optional<T> en lugar de null?
- [ ] ¿Uso Streams API en transformaciones?
- [ ] ¿Uso try-with-resources para recursos?
- [ ] ¿Uso features estables (no preview) salvo decisión explícita?
- [ ] ¿Documento `--enable-preview` si aplica?
- [ ] ¿Evito `Unsafe`/JNI salvo necesidad?
- [ ] ¿Uso Scoped Values en lugar de ThreadLocal si procede?
- [ ] ¿Uso JFR para observabilidad cuando haya problemas?
- [ ] ¿Cumplo SRP, un return y complejidad ≤ 10?
- [ ] ¿Javadoc completo y excepciones específicas?

---

## 📚 Referencias

- [JDK 25 (OpenJDK)](https://openjdk.org/projects/jdk/25/)
- [JEPs desde JDK 21 a JDK 25](https://openjdk.org/projects/jdk/25/jeps-since-jdk-21)
- [JEP 511: Module Import Declarations](https://openjdk.org/jeps/511)
- [JEP 512: Compact Source Files and Instance Main Methods](https://openjdk.org/jeps/512)
- [JEP 513: Flexible Constructor Bodies](https://openjdk.org/jeps/513)
- [JEP 506: Scoped Values](https://openjdk.org/jeps/506)
- [JEP 485: Stream Gatherers](https://openjdk.org/jeps/485)
- [JEP 505: Structured Concurrency (Preview)](https://openjdk.org/jeps/505)
- [JEP 507: Primitive Types in Patterns (Preview)](https://openjdk.org/jeps/507)
- [JEP 508: Vector API (Incubator)](https://openjdk.org/jeps/508)
- [JEP 470: PEM Encodings (Preview)](https://openjdk.org/jeps/470)

---

## 📘 Sobre esta guía

Guía profesional para Java 25.

👉 **Ver más guías**: [Repositorio completo](../README.md)

---

**Autor**: [David Bueno Vallejo](https://davidbuenov.com/) | [LinkedIn](https://www.linkedin.com/in/davidbueno/) | [GitHub](https://github.com/davidbuenov)

**Licencia**: MIT

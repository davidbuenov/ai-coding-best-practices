# ☕ Buenas Prácticas Java 21 (Profesional)

Guía profesional para escribir código Java 21 moderno, limpio, mantenible y fácil de revisar por equipos e IAs.

---

## 📑 Tabla de contenidos

- [🎯 Objetivo](#-objetivo)
- [🆕 Java 21: Recomendaciones Clave](#-java-21-recomendaciones-clave)
- [🧱 Diseño y Limpieza de Código](#-diseño-y-limpieza-de-código)
- [🧰 Programación Funcional y Streams (uso correcto)](#-programación-funcional-y-streams-uso-correcto)
- [🧵 Concurrencia en Java 21](#-concurrencia-en-java-21)
- [📋 Prompt de Sistema para IAs Generativas (Java 21 Pro)](#-prompt-de-sistema-para-ias-generativas-java-21-pro)
  - [🤖 Cómo Usar](#-cómo-usar)
  - [📄 Versión Completa del Prompt](#-versión-completa-del-prompt)
  - [⚡ Versión Corta del Prompt (Uso Rápido)](#-versión-corta-del-prompt-uso-rápido)
- [✅ Checklist Profesional](#-checklist-profesional)
- [📚 Referencias](#-referencias)

---

## 🎯 Objetivo

- ✅ Código expresivo y seguro en Java 21.
- ✅ Arquitecturas claras y métodos pequeños.
- ✅ Concurrencia moderna y eficiente.
- ✅ Resultado fácil de entender y mantener por equipos.

---

## 🆕 Java 21: Recomendaciones Clave

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

---

## 🧱 Diseño y Limpieza de Código

### ✅ SRP: métodos pequeños y una responsabilidad

### ✅ Javadoc claro en APIs públicas

### ✅ Excepciones específicas y mensajes con contexto

### ✅ Guard clauses con un solo return

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

### ✅ Complejidad ciclomática ≤ 10

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
// ✅ BIEN: efecto secundario final
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

## 🧵 Concurrencia en Java 21

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

## 📋 Prompt de Sistema para IAs Generativas (Java 21 Pro)

### 🤖 Cómo Usar

Usa el prompt completo para proyectos profesionales.

### 📄 Versión Completa del Prompt

````markdown
═══════════════════════════════════════════════════════════════════
🔽 INICIO DEL PROMPT - Copia desde aquí 🔽
═══════════════════════════════════════════════════════════════════

Eres un asistente experto en Java 21 profesional.

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

Reglas de calidad (para minimizar errores):
12) NO inventes clases, métodos o APIs inexistentes. Si no estás seguro, pide aclaración.
13) Mantén coherencia de tipos: no mezcles genéricos ni uses casts inseguros.
14) No uses código incompleto: devuelve clases y métodos totalmente compilables.
15) Si hay dependencias externas, indícalas explícitamente.
16) Valida nulos en entradas públicas y documenta precondiciones.
17) Evita efectos secundarios en streams (no uses map para logging).

Salida esperada:
- Código claro, legible y profesional.
- Explica decisiones clave en 2–4 líneas.
- Si detectas ambigüedad, pregunta antes de generar.

Prioriza mantenibilidad sobre micro-optimización.

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

Java 21 pro: records, sealed, pattern matching, Optional, streams, try-with-resources, SRP, un return, complejidad ≤10, Javadoc, excepciones específicas, virtual threads cuando aplique.

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
- [ ] ¿Cada método tiene un solo return?
- [ ] ¿Cada método hace una sola cosa?
- [ ] ¿Complejidad ciclomática ≤ 10?
- [ ] ¿Javadoc completo en APIs públicas?
- [ ] ¿Excepciones específicas con mensajes claros?
- [ ] ¿Uso virtual threads cuando hay I/O masivo?

---

## 📚 Referencias

- [Java 21 LTS](https://docs.oracle.com/en/java/javase/21/)
- [JEP 440: Record Patterns](https://openjdk.org/jeps/440)
- [JEP 441: Pattern Matching for switch](https://openjdk.org/jeps/441)
- [JEP 444: Virtual Threads](https://openjdk.org/jeps/444)
- Effective Java (3rd Ed.) — Joshua Bloch
- Clean Code — Robert C. Martin

---

## 📘 Sobre esta guía

Guía profesional para Java 21.

👉 **Ver más guías**: [Repositorio completo](../README.md)

---

**Autor**: [David Bueno Vallejo](https://davidbuenov.com/) | [LinkedIn](https://www.linkedin.com/in/davidbueno/) | [GitHub](https://github.com/davidbuenov)

**Licencia**: MIT

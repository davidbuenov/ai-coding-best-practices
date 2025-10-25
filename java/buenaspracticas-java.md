# ☕ Guía de Buenas Prácticas en Java Moderno (17+)

Este documento recopila las mejores prácticas profesionales de código Java pensadas para trabajar de forma eficaz con IAs generativas (GitHub Copilot, ChatGPT, Gemini, Claude): código limpio, mantenible, robusto y fácil de sugerir por una IA.

---

## 📑 Tabla de contenidos

- [🎯 Objetivo Pedagógico](#-objetivo-pedagógico)
- [🆕 Características de Java Moderno](#-características-de-java-moderno)
- [1️⃣ Records para DTOs y Value Objects](#1️⃣-records-para-dtos-y-value-objects)
- [2️⃣ Sealed Classes para Jerarquías Controladas](#2️⃣-sealed-classes-para-jerarquías-controladas)
- [3️⃣ Pattern Matching y Switch Expressions](#3️⃣-pattern-matching-y-switch-expressions)
- [4️⃣ Optional para Ausencia de Valores](#4️⃣-optional-para-ausencia-de-valores)
- [5️⃣ Streams y Programación Funcional](#5️⃣-streams-y-programación-funcional)
- [6️⃣ Try-with-Resources para Manejo de Recursos](#6️⃣-try-with-resources-para-manejo-de-recursos)
- [7️⃣ Un Solo Return + Guard Clauses](#7️⃣-un-solo-return--guard-clauses)
- [8️⃣ SRP: Métodos Pequeños con Una Responsabilidad](#8️⃣-srp-métodos-pequeños-con-una-responsabilidad)
- [9️⃣ Javadoc Completo y Útil](#9️⃣-javadoc-completo-y-útil)
- [🔟 Excepciones Específicas y Manejo Explícito](#-excepciones-específicas-y-manejo-explícito)
- [📋 Prompt de Sistema para IAs Generativas (Java)](#-prompt-de-sistema-para-ias-generativas-java)
    - [🤖 Cómo Usar](#-cómo-usar)
    - [📄 Versión Completa del Prompt](#-versión-completa-del-prompt)
    - [⚡ Versión Corta del Prompt (Uso Rápido)](#-versión-corta-del-prompt-uso-rápido)
- [✅ Checklist de Código Java Profesional](#-checklist-de-código-java-profesional)
- [📚 Referencias y Recursos](#-referencias-y-recursos)

---

## 🎯 Objetivo Pedagógico

Java moderno (17+) ha evolucionado notablemente. Si adoptas sus herramientas modernas:

- ✅ Más conciso sin perder seguridad de tipos
- ✅ Más legible y expresivo
- ✅ Más robusto con pattern matching y sealed classes
- ✅ Más fácil de entender para IAs generativas


---

## 🆕 Características de Java Moderno

- Java 17 LTS: Records, Sealed Classes, Pattern Matching
- Java 21 LTS: Virtual Threads, Pattern Matching mejorado
- Java 23+: Iteraciones sobre pattern matching y librerías

Requisito mínimo recomendado: Java 17 (ideal: Java 21 LTS).

---

## 1️⃣ Records para DTOs y Value Objects

### ✅ Usa Records (Java 16+)

Los `record` son clases inmutables perfectas para DTOs, value objects y resultados de queries.

```java
// ✅ BIEN: Record conciso y claro
public record Usuario(
    String nombre,
    String email,
    int edad
) {}

// Uso automático: equals, hashCode, toString, getters
Usuario user = new Usuario("Ana", "ana@example.com", 25);
System.out.println(user.nombre());  // Getter automático
System.out.println(user);           // ToString automático
```

```java
// ❌ MAL: Clase verbosa tradicional
public class UsuarioViejo {
    private final String nombre;
    private final String email;
    private final int edad;
    
    public UsuarioViejo(String nombre, String email, int edad) {
        this.nombre = nombre;
        this.email = email;
        this.edad = edad;
    }
    
    public String getNombre() { return nombre; }
    public String getEmail() { return email; }
    public int getEdad() { return edad; }
    
    @Override public boolean equals(Object o) { /* 15 líneas más */ }
    @Override public int hashCode() { /* 5 líneas más */ }
    @Override public String toString() { /* 3 líneas más */ }
}
// ~50 líneas vs 5 con record
```

### 🎯 Records con Validación

```java
// ✅ BIEN: Record con validación
public record Email(String valor) {
    public Email {
        if (valor == null || !valor.contains("@")) {
            throw new IllegalArgumentException("Email inválido: " + valor);
        }
    }
}

// Uso
Email email = new Email("usuario@example.com");   // ✅ OK
Email invalido = new Email("mal-email");          // ❌ Lanza excepción
```

---

## 2️⃣ Sealed Classes para Jerarquías Controladas

### ✅ Usa Sealed Classes (Java 17+)

Las `sealed` classes permiten controlar qué clases pueden extenderlas, perfecto para modelar estados o tipos de resultados.

```java
// ✅ BIEN: Sealed class con Result Type Pattern
public sealed interface Resultado<T> permits Exito, Error {}
public record Exito<T>(T valor) implements Resultado<T> {}
public record Error<T>(String mensaje, Throwable causa) implements Resultado<T> {}

// Uso con pattern matching
Resultado<String> resultado = obtenerDatos();
String mensaje = switch (resultado) {
    case Exito<String>(var valor) -> "Éxito: " + valor;
    case Error<String>(var msg, var causa) -> "Error: " + msg;
};
```

```java
// ❌ MAL: Jerarquía abierta con instanceof
public interface ResultadoViejo {}
public class ExitoViejo implements ResultadoViejo { public Object valor; }
public class ErrorViejo implements ResultadoViejo { public String mensaje; }

// Uso verboso y propenso a errores
ResultadoViejo r = obtenerDatos();
if (r instanceof ExitoViejo ex) {
    System.out.println(ex.valor);
} else if (r instanceof ErrorViejo er) {
    System.out.println(er.mensaje);
}
// ¿Qué pasa si alguien crea OtroTipoDeResultado?
```

---

## 3️⃣ Pattern Matching y Switch Expressions

```java
// ✅ BIEN: Pattern matching con switch expression
public String procesarEntrada(Object entrada) {
    return switch (entrada) {
        case String s   -> "Texto: " + s.toUpperCase();
        case Integer i  -> "Número: " + (i * 2);
        case List<?> xs -> "Lista con " + xs.size() + " elementos";
        case null       -> "Entrada nula";
        default         -> "Tipo desconocido: " + entrada.getClass().getName();
    };
}
```

```java
// ❌ MAL: instanceof en cascada con casts
public String procesarEntradaViejo(Object entrada) {
    if (entrada instanceof String) {
        String s = (String) entrada;
        return "Texto: " + s.toUpperCase();
    } else if (entrada instanceof Integer) {
        Integer i = (Integer) entrada;
        return "Número: " + (i * 2);
    } else if (entrada instanceof List) {
        List<?> lista = (List<?>) entrada;
        return "Lista con " + lista.size() + " elementos";
    } else if (entrada == null) {
        return "Entrada nula";
    } else {
        return "Tipo desconocido: " + entrada.getClass().getName();
    }
}
```

---

## 4️⃣ Optional para Ausencia de Valores

```java
// ✅ BIEN: Optional con API fluida
public Optional<Usuario> buscarUsuario(String email) {
    return repository.findByEmail(email);
}

String nombre = buscarUsuario("ana@example.com")
    .map(Usuario::nombre)
    .orElse("Usuario no encontrado");

// Guard clause
Optional<Usuario> u = buscarUsuario("ana@example.com");
if (u.isEmpty()) { throw new UsuarioNoEncontradoException("Usuario no existe"); }
Usuario usuario = u.get();
```

```java
// ❌ MAL: Retornar null
public Usuario buscarUsuarioViejo(String email) {
    Usuario usuario = repository.findByEmail(email);
    return usuario;  // Puede ser null ❌
}
```

---

## 5️⃣ Streams y Programación Funcional

```java
// ✅ BIEN: Stream API
List<Usuario> usuarios = repository.findAll();
List<String> emailsActivos = usuarios.stream()
    .filter(u -> u.edad() >= 18)
    .filter(Usuario::activo)
    .map(Usuario::email)
    .sorted()
    .toList();

int sumaEdades = usuarios.stream()
    .filter(Usuario::activo)
    .mapToInt(Usuario::edad)
    .sum();
```

```java
// ❌ MAL: Imperativo
List<String> emailsActivos = new ArrayList<>();
for (Usuario u : usuarios) {
    if (u.edad() >= 18 && u.activo()) {
        emailsActivos.add(u.email());
    }
}
Collections.sort(emailsActivos);
```

---

## 6️⃣ Try-with-Resources para Manejo de Recursos

```java
// ✅ BIEN: Try-with-resources
public List<String> leerArchivo(String ruta) throws IOException {
    try (var reader = Files.newBufferedReader(Path.of(ruta))) {
        return reader.lines().filter(l -> !l.isBlank()).toList();
    }
}

// Con base de datos
public List<Usuario> consultarUsuarios() throws SQLException {
    String sql = "SELECT * FROM usuarios WHERE activo = true";
    try (Connection conn = dataSource.getConnection();
         PreparedStatement stmt = conn.prepareStatement(sql);
         ResultSet rs = stmt.executeQuery()) {
        List<Usuario> usuarios = new ArrayList<>();
        while (rs.next()) {
            usuarios.add(new Usuario(
                rs.getString("nombre"), rs.getString("email"), rs.getInt("edad")));
        }
        return usuarios;
    }
}
```

---

## 7️⃣ Un Solo Return + Guard Clauses

Principio: UN método = UN solo return. Usa validaciones planas (if-else), evita la pirámide.

```java
// ✅ BIEN: Guard clauses con UN SOLO return
public Resultado<Usuario> crearUsuario(String nombre, String email, int edad) {
    Resultado<Usuario> resultado;
    if (nombre == null || nombre.isBlank()) {
        resultado = new Error<>("Nombre requerido", null);
    } else if (email == null || !email.contains("@")) {
        resultado = new Error<>("Email inválido", null);
    } else if (edad < 18) {
        resultado = new Error<>("Usuario debe ser mayor de edad", null);
    } else {
        Usuario usuario = new Usuario(nombre, email, edad);
        repository.save(usuario);
        resultado = new Exito<>(usuario);
    }
    return resultado;  // ✅ UN SOLO RETURN
}
```

```java
// ❌ MAL: Pirámide de la muerte
public Resultado<Usuario> crearUsuarioViejo(String nombre, String email, int edad) {
    if (nombre != null && !nombre.isBlank()) {
        if (email != null && email.contains("@")) {
            if (edad >= 18) {
                Usuario usuario = new Usuario(nombre, email, edad);
                repository.save(usuario);
                return new Exito<>(usuario);
            } else {
                return new Error<>("Usuario debe ser mayor de edad", null);
            }
        } else {
            return new Error<>("Email inválido", null);
        }
    } else {
        return new Error<>("Nombre requerido", null);
    }
}
```

---

## 8️⃣ SRP: Métodos Pequeños con Una Responsabilidad

```java
// ✅ BIEN: Métodos pequeños con UN SOLO return cada uno
public class ServicioUsuario {
    public Resultado<Usuario> procesarRegistro(DatosRegistro datos) {
        Resultado<Usuario> resultado;
        Optional<DatosRegistro> validacion = validarDatos(datos);
        if (validacion.isEmpty()) {
            resultado = new Error<>("Datos inválidos", null);
        } else {
            Usuario usuario = crearUsuario(datos);
            enviarEmailBienvenida(usuario);
            registrarEvento("USUARIO_CREADO", usuario.email());
            resultado = new Exito<>(usuario);
        }
        return resultado;  // ✅ UN SOLO RETURN
    }

    private Optional<DatosRegistro> validarDatos(DatosRegistro datos) {
        Optional<DatosRegistro> resultado;
        if (datos.email() == null || !datos.email().contains("@")) {
            resultado = Optional.empty();
        } else {
            resultado = Optional.of(datos);
        }
        return resultado;  // ✅ UN SOLO RETURN
    }

    private Usuario crearUsuario(DatosRegistro datos) {
        Usuario resultado = repository.save(new Usuario(
            datos.nombre(), datos.email(), datos.edad()));
        return resultado;  // ✅ UN SOLO RETURN
    }

    private void enviarEmailBienvenida(Usuario usuario) {
        emailService.enviar(usuario.email(), "Bienvenido", "Gracias por registrarte");
    }

    private void registrarEvento(String tipo, String detalle) {
        auditService.log(tipo, detalle);
    }
}
```

---

## 9️⃣ Javadoc Completo y Útil

```java
// ✅ BIEN: Javadoc completo
/**
 * Busca un usuario por su dirección de email.
 *
 * @param email la dirección de email del usuario (debe contener '@')
 * @return Optional con el usuario si existe, Optional.empty() si no
 * @throws IllegalArgumentException si el email es null o inválido
 * @throws RepositoryException si hay error de conexión con la base de datos
 */
public Optional<Usuario> buscarPorEmail(String email) {
    if (email == null || !email.contains("@")) {
        throw new IllegalArgumentException("Email inválido: " + email);
    }
    return repository.findByEmail(email);
}
```

---

## 🔟 Excepciones Específicas y Manejo Explícito

```java
// ✅ BIEN: Excepciones específicas con UN SOLO return
public class UsuarioNoEncontradoException extends RuntimeException {
    public UsuarioNoEncontradoException(String email) {
        super("Usuario no encontrado con email: " + email);
    }
}

public class EmailDuplicadoException extends RuntimeException {
    public EmailDuplicadoException(String email) {
        super("Ya existe un usuario con email: " + email);
    }
}

public Usuario buscarUsuario(String email) {
    Usuario resultado = repository.findByEmail(email)
        .orElseThrow(() -> new UsuarioNoEncontradoException(email));
    return resultado;  // ✅ UN SOLO RETURN
}

public Usuario crearUsuario(DatosRegistro datos) {
    Usuario resultado;
    if (repository.existsByEmail(datos.email())) {
        throw new EmailDuplicadoException(datos.email());
    } else {
        resultado = repository.save(new Usuario(
            datos.nombre(), datos.email(), datos.edad()));
    }
    return resultado;  // ✅ UN SOLO RETURN
}
```

---

## 📋 Prompt de Sistema para IAs Generativas (Java)

### 🤖 Cómo Usar

Usa la versión completa para proyectos serios y la corta para prompts rápidos.

### 📄 Versión Completa del Prompt

````markdown
═══════════════════════════════════════════════════════════════════
🔽 INICIO DEL PROMPT - Copia desde aquí 🔽
═══════════════════════════════════════════════════════════════════

# Prompt de Sistema para Generación de Código Java Profesional

Eres un asistente de programación Java especializado en código limpio, moderno y profesional.

## Reglas OBLIGATORIAS para Java 17+:

1. **Records para DTOs y Value Objects**:
   - ✅ Usa `record` para clases inmutables de datos
   - ✅ Añade validaciones en constructor compacto si es necesario
   - ❌ NO uses clases verbosas con getters/setters/equals/hashCode manuales

1. **Sealed Classes para Jerarquías Controladas**:
   - ✅ Usa `sealed interface/class` para tipos conocidos y limitados
   - ✅ Combina con pattern matching en switch expressions
   - ❌ NO uses jerarquías abiertas con instanceof en cascada

1. **Pattern Matching y Switch Expressions**:
   - ✅ Usa pattern matching con `switch` y `instanceof`
   - ✅ Usa switch expressions (retornan valor)
   - ❌ NO uses if-else con instanceof y casts manuales

1. **Optional en lugar de null**:
   - ✅ Retorna `Optional<T>` cuando un valor puede estar ausente
   - ✅ Usa API fluida: `.map()`, `.filter()`, `.orElse()`, `.orElseThrow()`
   - ❌ NO retornes null ni uses null checks manuales

1. **Streams API**:
   - ✅ Usa streams para operaciones sobre colecciones
   - ✅ Prefiere declarativo sobre imperativo
   - ❌ NO uses loops for/while para transformaciones simples

1. **Try-with-Resources**:
   - ✅ Usa try-with-resources para TODOS los recursos (Connection, Stream, Reader)
   - ✅ Usa `var` para reducir verbosidad
   - ❌ NO cierres recursos manualmente con finally

1. **UN SOLO Return + Guard Clauses**:
   - ✅ Cada método tiene UN ÚNICO punto de retorno
   - ✅ Usa if-else planas para validaciones (no anidadas)
   - ✅ Declara variable de resultado al inicio
   - ❌ NO uses múltiples `return` en diferentes lugares
   - ❌ NO uses if-else anidados (pirámide de la muerte)

1. **Single Responsibility Principle**:
   - ✅ Cada método hace UNA cosa
   - ✅ Métodos de 10-20 líneas máximo
   - ✅ Nombres descriptivos que explican el propósito
   - ❌ NO crees métodos que hagan validación + lógica + email + logging

1. **Javadoc Completo**:
   ```java
   /**
    * Descripción breve del método.
    *
    * @param parametro descripción del parámetro
    * @return descripción del valor de retorno
    * @throws TipoExcepcion cuándo y por qué se lanza
    */
   ```

1. **Excepciones Específicas**:
    - ✅ Crea excepciones custom por tipo de error
    - ✅ Mensajes descriptivos con contexto
    - ❌ NO uses Exception/RuntimeException genéricas

## Ejemplos de Código CORRECTO:

```java
// Record con validación
public record Email(String valor) {
    public Email {
        if (valor == null || !valor.contains("@")) {
            throw new IllegalArgumentException("Email inválido: " + valor);
        }
    }
}

// Sealed class con pattern matching
public sealed interface Resultado<T> permits Exito, Error {}
public record Exito<T>(T valor) implements Resultado<T> {}
public record Error<T>(String mensaje) implements Resultado<T> {}

// Uso
String resultado = switch (operacion) {
    case Exito<String>(var valor) -> "OK: " + valor;
    case Error<String>(var msg) -> "Error: " + msg;
};

// UN SOLO return con guard clauses
public Resultado<Usuario> validar(String email) {
    Resultado<Usuario> resultado;
    if (email == null || email.isBlank()) {
        resultado = new Error<>("Email vacío", null);
    } else if (!email.contains("@")) {
        resultado = new Error<>("Email inválido", null);
    } else {
        Usuario usuario = repository.findByEmail(email);
        resultado = new Exito<>(usuario);
    }
    return resultado;  // UN SOLO RETURN
}
```

## IMPORTANTE

- Java 17 LTS es el mínimo
- UN método = UN solo return (facilita debugging y comprensión de IAs)
- Código que una IA pueda entender y modificar fácilmente
- Prioriza legibilidad sobre brevedad
- Si tienes duda entre tradicional y moderno, elige moderno

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

Genera código Java 17+ profesional siguiendo estas reglas:
1) Records para DTOs (no clases con getters/setters)
2) Sealed classes + pattern matching (no instanceof)
3) Optional<T> en lugar de null
4) Streams API (no loops imperativos)
5) Try-with-resources para TODO recurso
6) UN SOLO return por método (validaciones planas)
7) Métodos 10–20 líneas, una responsabilidad
8) Javadoc completo (@param, @return, @throws)
9) Excepciones específicas (no RuntimeException genérica)

Ejemplo:
```java
public record Email(String valor) {
    public Email {
        if (valor == null || !valor.contains("@")) {
            throw new IllegalArgumentException("Email inválido");
        }
    }
}

public Resultado<Usuario> validar(String email) {
    Resultado<Usuario> resultado;
    if (email == null) { resultado = new Error<>("Email nulo", null); }
    else { resultado = new Exito<>(repository.findByEmail(email).orElse(null)); }
    return resultado;  // UN SOLO RETURN
}
```

═══════════════════════════════════════════════════════════════════
🔼 FIN DEL PROMPT CORTO - Copia hasta aquí 🔼
═══════════════════════════════════════════════════════════════════
````

---

## ✅ Checklist de Código Java Profesional

- [ ] ¿Usé `record` para DTOs y value objects?
- [ ] ¿Usé `sealed class/interface` para jerarquías controladas?
- [ ] ¿Usé `Optional<T>` en lugar de `null`?
- [ ] ¿Usé pattern matching con switch expressions?
- [ ] ¿Usé Streams API?
- [ ] ¿Usé try-with-resources para TODOS los recursos?
- [ ] ¿Creé excepciones específicas con mensajes con contexto?
- [ ] ¿Cada método tiene UN SOLO return? (sin múltiples returns)
- [ ] ¿Usé guard clauses (if-else planas) en lugar de anidar?
- [ ] ¿Cada método hace UNA sola cosa y < 20–30 líneas?
- [ ] ¿Los nombres de métodos/variables son descriptivos?
- [ ] ¿Cada método público tiene Javadoc completo?

---

## 📚 Referencias y Recursos

- [JEP 395: Records](https://openjdk.org/jeps/395) — Java 16
- [JEP 409: Sealed Classes](https://openjdk.org/jeps/409) — Java 17
- [JEP 441: Pattern Matching for switch](https://openjdk.org/jeps/441) — Java 21
- [Java 17 LTS](https://docs.oracle.com/en/java/javase/17/)
- [Java 21 LTS](https://docs.oracle.com/en/java/javase/21/)
- Effective Java (3rd Ed.) — Joshua Bloch
- Modern Java in Action — Raoul-Gabriel Urma
- Clean Code — Robert C. Martin

---

## 📘 Sobre esta guía

Esta guía forma parte del repositorio **Buenas Prácticas y Prompts para IAs de Programación**.

👉 **Ver más guías**: [Repositorio completo](../README.md)

---

**Autor**: [David Bueno Vallejo](https://davidbuenov.com/) | [LinkedIn](https://www.linkedin.com/in/davidbueno/) | [GitHub](https://github.com/davidbuenov)

**Licencia**: MIT

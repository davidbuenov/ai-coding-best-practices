# ☕ Buenas Prácticas Java 21 (UMA · Primeros Cursos)

Guía didáctica para alumnado de ingeniería informática (UMA) que empieza con Java 21. Incluye ejemplos **claros**, errores típicos y “código basura” que suele generar una IA si no se le da un buen prompt.

---

## 📑 Tabla de contenidos

- [🎯 Objetivo Pedagógico](#-objetivo-pedagógico)
- [✅ Reglas Básicas para Principiantes](#-reglas-básicas-para-principiantes)
- [🧪 Errores Típicos que Genera una IA (y cómo arreglarlos)](#-errores-típicos-que-genera-una-ia-y-cómo-arreglarlos)
- [🆕 Java 21: Características Importantes (explicadas sin lío)](#-java-21-características-importantes-explicadas-sin-lío)
- [🧩 Buen Diseño en Métodos (SRP, Javadoc y Excepciones)](#-buen-diseño-en-métodos-srp-javadoc-y-excepciones)
- [🧯 Try-with-Resources (sin fugas)](#-try-with-resources-sin-fugas)
- [📐 Complejidad Ciclomática (recomendación ≤ 10)](#-complejidad-ciclomática-recomendación--10)
- [📋 Prompt de Sistema para IAs Generativas (Java 21 · UMA)](#-prompt-de-sistema-para-ias-generativas-java-21--uma)
  - [🤖 Cómo Usar](#-cómo-usar)
  - [📄 Versión Completa del Prompt](#-versión-completa-del-prompt)
  - [⚡ Versión Corta del Prompt (Uso Rápido)](#-versión-corta-del-prompt-uso-rápido)
- [✅ Checklist de Entrega](#-checklist-de-entrega)
- [📚 Referencias y Recursos](#-referencias-y-recursos)

---

## 🎯 Objetivo Pedagógico

Al empezar con Java 21, lo más importante es:

- ✅ Aprender **código limpio y legible**
- ✅ Evitar errores lógicos comunes
- ✅ Escribir programas fáciles de revisar por el profesor
- ✅ Entender lo que haces (no copiar ciegamente de la IA)

---

## ✅ Reglas Básicas para Principiantes

1. **Un método = una responsabilidad.**
2. **Evita `null` si puedes; valida datos temprano.**
3. **Nombres claros:** `calcularMedia`, `esValido`, `buscarAlumno`.
4. **Control de flujo simple:** `if`, `else`, `while`, `for` bien usados.
5. **Evita “magia”:** números mágicos y condiciones raras.

---

## 🧪 Errores Típicos que Genera una IA (y cómo arreglarlos)

Estas son cosas que suelen salir cuando la IA no tiene un buen prompt.

### 1) ❌ Múltiples `return` sin necesidad

```java
// ❌ MAL: muchos returns sin justificación
public static boolean esPar(int n) {
    if (n == 0) return true;
    if (n == 1) return false;
    if (n == 2) return true;
    return n % 2 == 0;
}
```


### 3.1) Pattern Matching en `switch` (explicación simple)

```java
// ✅ BIEN: un flujo simple y claro
public static boolean esPar(int n) {
    boolean resultado = (n % 2 == 0);
    return resultado;
}
```

---

### 2) ❌ Variable booleana dentro de un `for` y no salir


```java
// ❌ MAL: recorre todo aunque ya encontró la respuesta
public static boolean contieneNegativo(int[] datos) {
    boolean hayNegativo = false;
    for (int x : datos) {
        if (x < 0) {
            hayNegativo = true;
        }
    }
    return hayNegativo;
}
```

```java
// ✅ BIEN: usa while y corta cuando encuentra un negativo
public static boolean contieneNegativo(int[] datos) {
    int i = 0;
    boolean hayNegativo = false;
    while (i < datos.length && !hayNegativo) {
        if (datos[i] < 0) {
            hayNegativo = true;
        }
        i++;
    }
    return hayNegativo;
}
```

---

### 3) ❌ Usar `for-each` cuando necesitas índice

```java
// ❌ MAL: mezcla for-each con índice manual
public static int posicionPrimeraA(String[] nombres) {
    int pos = -1;
    int i = 0;
    for (String s : nombres) {
        if (s.startsWith("A")) {
            pos = i;
        }
        i++;
    }
    return pos;
}
```

```java
// ✅ BIEN: usa for clásico cuando necesitas índice
public static int posicionPrimeraA(String[] nombres) {
    int pos = -1;
    for (int i = 0; i < nombres.length && pos == -1; i++) {
        if (nombres[i].startsWith("A")) {
            pos = i;
        }
    }
    return pos;
}
```

---

### 4) ❌ “Números mágicos” sin explicación

```java
// ❌ MAL: no se sabe qué significa 0.21
public static double calcularIVA(double precio) {
    return precio * 0.21;
}
```

```java
// ✅ BIEN: constante con nombre
private static final double IVA = 0.21;

public static double calcularIVA(double precio) {
    return precio * IVA;
}
```

---

### 5) ❌ Validaciones confusas y anidadas

```java
// ❌ MAL: pirámide de la muerte
public static boolean accesoPermitido(int edad, boolean tieneCredencial) {
    if (edad >= 18) {
        if (tieneCredencial) {
            return true;
        } else {
            return false;
        }
    } else {
        return false;
    }
}
```

```java
// ✅ BIEN: simple y directo
public static boolean accesoPermitido(int edad, boolean tieneCredencial) {
    boolean permitido = (edad >= 18) && tieneCredencial;
    return permitido;
}
```

---

### 6) ❌ Comparar `String` con `==`

```java
// ❌ MAL: comparación de referencias
public static boolean esAdmin(String rol) {
    return rol == "ADMIN";
}
```

```java
// ✅ BIEN: comparación de contenido
public static boolean esAdmin(String rol) {
    boolean resultado = "ADMIN".equals(rol);
    return resultado;
}
```

---

### 7) ❌ Usar `null` sin control

```java
// ❌ MAL: puede romper el programa
public static int longitud(String s) {
    return s.length();
}
```

```java
// ✅ BIEN: validación simple
public static int longitud(String s) {
    int resultado;
    if (s == null) {
        resultado = 0;
    } else {
        resultado = s.length();
    }
    return resultado;
}
```

---

## 🆕 Java 21: Características Importantes (explicadas sin lío)

> **Nota**: Algunas características son avanzadas (records, sealed classes). Aquí solo se explican para que sepas que existen, pero no son obligatorias en primeros cursos.

### 1) `var` (inferencia de tipos)

```java
// ✅ BIEN: se entiende el tipo
var nombres = new ArrayList<String>();
```

Sabemos que `var` existe, pero **recomendamos declarar el tipo completo** en los primeros cursos para que el código sea más claro:

```java
// ✅ RECOMENDADO PARA PRIMERO
ArrayList<String> nombres = new ArrayList<String>();
```

---

### 2) Text Blocks (cadenas multilínea)

```java
String mensaje = """
    Hola,
    Bienvenido a Java 21.
    """;
```

---

### 3) `switch` mejorado

```java
// ✅ BIEN: switch expression
public static String dia(int n) {
    return switch (n) {
        case 1 -> "Lunes";
        case 2 -> "Martes";
        default -> "Otro";
    };
}
```

---

### 4) Records (avanzado, solo para conocer)

Un `record` es una clase simple e inmutable. No es obligatorio en primero.

```java
// ✅ Ejemplo básico
public record Alumno(String nombre, int edad) {}
```

---

### 5) Sealed Classes (avanzado, solo para conocer)

Permiten controlar qué clases heredan. Útil en proyectos grandes, no obligatorio en primero.

```java
public sealed interface Resultado permits Exito, Error {}
public record Exito(String mensaje) implements Resultado {}
public record Error(String mensaje) implements Resultado {}
```

---

## 🧩 Buen Diseño en Métodos (SRP, Javadoc y Excepciones)

### ✅ SRP (Single Responsibility Principle)

Un método debe hacer **una sola cosa**.

```java
// ✅ BIEN: método pequeño y con una sola tarea
public static boolean esMayorDeEdad(int edad) {
    boolean resultado = (edad >= 18);
    return resultado;
}
```

```java
// ❌ MAL: hace demasiadas cosas en un solo método
public static void registrarAlumno(String nombre, int edad) {
    if (edad >= 18) {
        System.out.println("Mayor de edad");
    }
    System.out.println("Registrando: " + nombre);
    // Guardar, enviar email, log, etc... (todo junto)
}
```

### ✅ Javadoc básico y útil

```java
/**
 * Calcula la media de dos números.
 *
 * @param a primer número
 * @param b segundo número
 * @return media de a y b
 */
public static double media(double a, double b) {
    double resultado = (a + b) / 2.0;
    return resultado;
}
```

### ✅ Excepciones específicas (cuando algo no puede continuar)

```java
// ✅ BIEN: excepción con nombre claro
public class NotaInvalidaException extends RuntimeException {
    public NotaInvalidaException(int nota) {
        super("Nota inválida: " + nota);
    }
}

public static void validarNota(int nota) {
    if (nota < 0 || nota > 10) {
        throw new NotaInvalidaException(nota);
    }
}
```

---

## 🧯 Try-with-Resources (sin fugas)

```java
// ✅ BIEN: el recurso se cierra solo
public static String leerPrimeraLinea(String ruta) throws IOException {
    String resultado;
    try (BufferedReader br = new BufferedReader(new FileReader(ruta))) {
        resultado = br.readLine();
    }
    return resultado;
}
```

---

## 📐 Complejidad Ciclomática (recomendación ≤ 10)

**Recomendación:** la complejidad ciclomática de cada método **no debe superar 10**.

¿Qué significa? Que no debe haber demasiados `if`, `else`, `switch`, `for`, `while` en un mismo método. Si pasa de 10, **divide el método**.

```java
// ✅ BIEN: lógica separada en métodos más pequeños
public static boolean accesoPermitido(int edad, boolean credencial) {
    boolean mayor = esMayorDeEdad(edad);
    boolean resultado = mayor && credencial;
    return resultado;
}
```

---

## 📋 Prompt de Sistema para IAs Generativas (Java 21 · UMA)

### 🤖 Cómo Usar

Copia el prompt completo al inicio de la conversación con la IA. Así evitas errores típicos y obtienes código más claro.

### 📄 Versión Completa del Prompt

````markdown
═══════════════════════════════════════════════════════════════════
🔽 INICIO DEL PROMPT - Copia desde aquí 🔽
═══════════════════════════════════════════════════════════════════

Eres un asistente de Java 21 para alumnos de primeros cursos de ingeniería informática (UMA).

Reglas obligatorias:
1) Código simple, didáctico y muy legible.
2) Evita “pirámide de la muerte” y anidamientos innecesarios.
3) No uses múltiples returns salvo que sea imprescindible.
4) No uses booleanos en bucles si puedes cortar antes con while.
5) No uses for-each si necesitas índice; usa for clásico.
6) Usa nombres claros: calcularMedia, esValido, buscarAlumno.
7) Explica brevemente el porqué de las decisiones.
8) No uses streams ni features avanzadas salvo que el usuario lo pida.
9) Si usas Java 21 features, explícalas con ejemplos sencillos.
10) Evita null sin validación previa.

Reglas de seguridad y calidad (muy importantes):
11) No inventes clases o métodos que no existan en el enunciado.
12) Si falta información del problema, pide aclaración antes de generar código.
13) Entrega siempre código completo y compilable.
14) No uses magia ni abreviaturas confusas.
15) Revisa que el ejemplo haga EXACTAMENTE lo que se pide, sin extras.

Incluye siempre:
- Un ejemplo correcto y uno incorrecto cuando sea útil.
- Una explicación breve y directa.

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

Java 21 para principiantes (UMA). Código simple y legible. Evita múltiples returns, booleans inútiles en bucles, pirámides de ifs, nulls sin validar. Usa for clásico si necesitas índice y while si quieres cortar. No inventes clases/métodos. Si falta información, pregunta antes. Entrega código completo y compilable. Explica brevemente cada decisión.

═══════════════════════════════════════════════════════════════════
🔼 FIN DEL PROMPT CORTO - Copia hasta aquí 🔼
═══════════════════════════════════════════════════════════════════
````

---

## ✅ Checklist de Entrega

- [ ] ¿El código es legible y fácil de explicar en clase?
- [ ] ¿Evité `if` anidados innecesarios?
- [ ] ¿No usé múltiples `return` sin necesidad?
- [ ] ¿Usé `while` si quería cortar pronto un bucle?
- [ ] ¿Usé `for` clásico cuando necesitaba índice?
- [ ] ¿Validé valores nulos antes de usarlos?
- [ ] ¿Evité números mágicos con constantes?
- [ ] ¿Los nombres de variables y métodos son claros?
- [ ] ¿El ejemplo se puede entender sin conocimientos avanzados?
- [ ] ¿La complejidad ciclomática de cada método es ≤ 10?

---

## 📚 Referencias y Recursos

- [Java 21 LTS](https://docs.oracle.com/en/java/javase/21/)
- [Guía de estilo Java (Google)](https://google.github.io/styleguide/javaguide.html)
- Effective Java (3rd Ed.) — Joshua Bloch (capítulos básicos)
- Clean Code — Robert C. Martin (capítulos básicos)

---

## 📘 Sobre esta guía

Guía pensada para alumnado de **primeros cursos UMA**.

👉 **Ver más guías**: [Repositorio completo](../README.md)

---

**Autor**: [David Bueno Vallejo](https://davidbuenov.com/) | [LinkedIn](https://www.linkedin.com/in/davidbueno/) | [GitHub](https://github.com/davidbuenov)

**Licencia**: MIT

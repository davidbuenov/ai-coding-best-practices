# 🧠 Guía de Buenas Prácticas en C++ Moderno (C++20/23)

Este documento resume prácticas profesionales para escribir C++ moderno, robusto y legible, optimizado para colaborar con IAs generativas (Copilot, ChatGPT, Gemini, Claude): código claro, con RAII, tipado fuerte y APIs expresivas.

---

## 📑 Tabla de contenidos

- [🎯 Objetivo](#-objetivo)
- [🆕 Requisitos y versiones](#-requisitos-y-versiones)
- [1) RAII y Smart Pointers](#1-raii-y-smart-pointers)
- [2) Tipos Fuertes: enum class y using](#2-tipos-fuertes-enum-class-y-using)
- [3) string_view y span](#3-string_view-y-span)
- [4) Ranges y Algoritmos (C++20)](#4-ranges-y-algoritmos-c20)
- [5) optional, variant y expected (C++23)](#5-optional-variant-y-expected-c23)
- [6) UN SOLO return + Guard Clauses](#6-un-solo-return--guard-clauses)
- [7) Errores: Excepciones vs expected](#7-errores-excepciones-vs-expected)
- [8) Documentación con Doxygen](#8-documentación-con-doxygen)
- [🤖 Prompt de Sistema para IAs Generativas (C++)](#-prompt-de-sistema-para-ias-generativas-c)
    - [Cómo usar](#cómo-usar)
    - [Versión completa del prompt](#versión-completa-del-prompt)
    - [Versión corta del prompt](#versión-corta-del-prompt)
- [✅ Checklist de C++ Moderno](#-checklist-de-c-moderno)
- [📚 Referencias](#-referencias)

---

## 🎯 Objetivo

C++ moderno ofrece herramientas potentes para reducir errores y mejorar la claridad:

- ✅ RAII y smart pointers evitan fugas de recursos
- ✅ Tipos fuertes: enum class, using aliases
- ✅ Eficiencia y claridad: string_view, span
- ✅ Algoritmos y ranges expresivos (C++20)
- ✅ Modelado explícito de estados con optional/variant/expected
- ✅ Principio de UN SOLO return y guard clauses

---

## 🆕 Requisitos y versiones

- Recomendado: C++20 mínimo; ideal: C++23
- Compiladores con soporte: GCC 13+, Clang 16+, MSVC VS2022 actualizado

---

## 1) RAII y Smart Pointers

```cpp
// ✅ BIEN: RAII con unique_ptr
#include <memory>
#include <utility>

struct Recurso {
    void usar() {}
};

void ejemplo_raii() {
    std::unique_ptr<Recurso> r = std::make_unique<Recurso>();
    r->usar();
} // r se libera automáticamente aquí
```

```cpp
// ❌ MAL: new/delete manual (propenso a fugas y excepciones)
void ejemplo_mal() {
    Recurso* r = new Recurso();
    r->usar();
    delete r; // si hay excepción antes, fuga
}
```

---

## 2) Tipos Fuertes: enum class y using

```cpp
// ✅ BIEN: enum class con ámbito y tipo subyacente
#include <cstdint>

enum class Estado : std::uint8_t { Ok, Error, Desconocido };

using UsuarioId = std::uint64_t;   // alias semántico

struct Usuario {
    UsuarioId id{};
    Estado estado{Estado::Ok};
};
```

```cpp
// ❌ MAL: enum tradicional sin ámbito
enum EstadoMal { Ok, Error, Desconocido }; // colisiona con otros símbolos
```

---

## 3) string_view y span

```cpp
#include <string_view>
#include <span>
#include <vector>
#include <cstddef>

// ✅ BIEN: funciones no dueñas usan string_view/span
std::size_t contar_letras_a(std::string_view s) {
    return std::count(s.begin(), s.end(), 'a');
}

int suma(std::span<const int> xs) {
    int total = 0;
    for (int x : xs) total += x;
    return total;
}

void ejemplo_views() {
    std::string texto = "banana";
    std::vector<int> v{1,2,3,4};
    auto n = contar_letras_a(texto); // sin copias
    auto t = suma(v);                 // sin copias
}
```

---

## 4) Ranges y Algoritmos (C++20)

```cpp
#include <ranges>
#include <vector>
#include <algorithm>

std::vector<int> cuadrados_pares(const std::vector<int>& xs) {
    auto view = xs | std::views::filter([](int x){ return x % 2 == 0; })
                   | std::views::transform([](int x){ return x * x; });
    std::vector<int> out;
    std::ranges::copy(view, std::back_inserter(out));
    return out; // ✅ UN SOLO return al final
}
```

```cpp
// ❌ MAL: bucles imperativos largos y propensos a errores
std::vector<int> cuadrados_pares_mal(const std::vector<int>& xs) {
    std::vector<int> out;
    for (auto x : xs) {
        if (x % 2 == 0) {
            out.push_back(x * x);
        }
    }
    return out;
}
```

---

## 5) optional, variant y expected (C++23)

```cpp
#include <optional>
#include <variant>
#if __has_include(<expected>)
  #include <expected>  // C++23
#endif

struct Error {
    std::string mensaje;
};

std::optional<int> parse_int(std::string_view s) {
    try {
        return std::stoi(std::string{s});
    } catch (...) {
        return std::nullopt; // ausencia explícita
    }
}

std::variant<int, Error> parse_int_variant(std::string_view s) {
    try {
        return std::stoi(std::string{s});
    } catch (...) {
        return Error{"No es un entero"};
    }
}

#if __cpp_lib_expected >= 202211L
std::expected<int, Error> parse_int_expected(std::string_view s) {
    try { return std::stoi(std::string{s}); }
    catch (...) { return std::unexpected(Error{"No es un entero"}); }
}
#endif
```

---

## 6) UN SOLO return + Guard Clauses

```cpp
// ✅ BIEN: validaciones planas y un único return
#include <string>

struct Usuario { std::string nombre; int edad{}; };
struct Resultado {
    bool ok{}; std::string mensaje; Usuario valor{};
};

Resultado crear_usuario(std::string_view nombre, int edad) {
    Resultado r{}; // estado acumulado
    if (nombre.empty()) {
        r.ok = false; r.mensaje = "Nombre requerido";
    } else if (edad < 18) {
        r.ok = false; r.mensaje = "Debe ser mayor de edad";
    } else {
        r.ok = true; r.valor = Usuario{std::string{nombre}, edad};
    }
    return r; // ✅ UN SOLO return
}
```

```cpp
// ❌ MAL: múltiples returns dispersos
Resultado crear_usuario_mal(std::string_view nombre, int edad) {
    if (nombre.empty()) return Resultado{false, "Nombre requerido", {}};
    if (edad < 18) return Resultado{false, "Debe ser mayor de edad", {}};
    return Resultado{true, "", Usuario{std::string{nombre}, edad}};
}
```

---

## 7) Errores: Excepciones vs expected

- Usa excepciones para errores excepcionales: fallos de IO, precondiciones rotas, invariantes
- Usa `std::expected` (o alternativa) para flujos normales donde error es posible y frecuente
- Evita códigos de error mágicos; modela el error con un tipo

```cpp
// ✅ BIEN: flujo con expected (C++23)
#if __cpp_lib_expected >= 202211L
std::expected<Usuario, Error> registrar(std::string_view nombre, int edad) {
    if (nombre.empty()) return std::unexpected(Error{"Nombre requerido"});
    if (edad < 18) return std::unexpected(Error{"Debe ser mayor de edad"});
    return Usuario{std::string{nombre}, edad};
}
#endif
```

---

## 8) Documentación con Doxygen

```cpp
/// Crea un usuario válido.
/// @param nombre Nombre no vacío.
/// @param edad Debe ser >= 18.
/// @return Usuario creado o error.
Resultado crear_usuario(std::string_view nombre, int edad);
```

### Por qué importa

- Claridad y contrato: explica intención, pre/postcondiciones y efectos.
- Onboarding rápido: nuevos miembros entienden APIs sin leer implementación.
- Sinergia con IAs: mejores sugerencias al tener contexto explícito.
- Tooling: genera documentación navegable y verifica tags en CI.

### Plantilla mínima (función)

```cpp
/// @brief Descripción corta y accionable.
/// @details Contexto suficiente: precondiciones, efectos, invariantes.
/// @param foo Descripción del parámetro foo.
/// @param bar Descripción del parámetro bar.
/// @return Qué devuelve y en qué casos.
/// @note Notas relevantes de uso/rendimiento.
/// @warning Riesgos, undefined behavior, errores comunes.
/// @see OtroSimboloRelacionado
Resultado mi_funcion(TFoo foo, TBar bar);
```

### Plantilla mínima (clase)

```cpp
/// @brief Responsabilidad principal de la clase.
/// @details Colabora con X, no hace Y; ejemplos de uso breves.
class GestorRecursos {
 public:
    /// @brief Construye el gestor con límites opcionales.
    /// @param limite Máximo de recursos (0 = sin límite).
    explicit GestorRecursos(std::size_t limite = 0);
};
```

---

## 🤖 Prompt de Sistema para IAs Generativas (C++)

### Cómo usar

Usa la versión completa para proyectos y la corta para peticiones rápidas.

### Versión completa del prompt

````markdown
═══════════════════════════════════════════════════════════════════
🔽 INICIO DEL PROMPT - Copia desde aquí 🔽
═══════════════════════════════════════════════════════════════════

# Prompt de Sistema para Generación de Código C++20/23 Profesional

Eres un asistente C++ que genera código moderno, seguro y legible.

## Reglas OBLIGATORIAS (C++20/23)

1. RAII y Smart Pointers
   - ✅ Usa `std::unique_ptr` por defecto; `std::shared_ptr` solo si hay compartición
   - ✅ Evita `new/delete` manuales

2. Tipos fuertes
   - ✅ `enum class` con tipo subyacente
   - ✅ `using` para aliases semánticos

3. Vistas no dueñas
   - ✅ `std::string_view` y `std::span`
   - ✅ Evita copias innecesarias de `std::string`/`std::vector`

4. Ranges/Algoritmos (C++20)
   - ✅ Usa `std::ranges` y `std::views` en transformaciones

5. Modelado explícito de estados
   - ✅ `std::optional`, `std::variant`, `std::expected` (C++23)
   - ✅ Evita códigos de error mágicos

6. UN SOLO return + Guard Clauses
   - ✅ Un único `return` por función
   - ✅ Validaciones planas (sin pirámide)

7. Errores
   - ✅ Excepciones para casos excepcionales
   - ✅ `std::expected` para flujos normales con error

8. Documentación
   - ✅ Doxygen: `///` con `@param`, `@return`

## Ejemplo correcto

```cpp
struct Error { std::string mensaje; };

std::variant<int, Error> parse_int(std::string_view s) {
    try { return std::stoi(std::string{s}); }
    catch (...) { return Error{"No es un entero"}; }
}

struct Resultado { bool ok{}; std::string msg; };

Resultado validar_nombre(std::string_view n) {
    Resultado r{};
    if (n.empty()) r = {false, "Nombre vacío"};
    else r = {true, ""};
    return r; // UN SOLO return
}
```

## Importante

- C++20 mínimo, ideal C++23
- UN método/función = UN solo return
- Prefiere claridad, RAII, y tipos fuertes

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

Genera C++20/23 profesional con estas reglas:
1) RAII + unique_ptr (no new/delete)
2) enum class + using aliases
3) string_view/span en entradas no dueñas
4) Ranges + algorithms en transformaciones
5) optional/variant/expected (no códigos mágicos)
6) UN SOLO return por función + guard clauses
7) Doxygen con @param @return

Ejemplo:
```cpp
std::optional<int> parse(std::string_view s) {
    try { return std::stoi(std::string{s}); }
    catch (...) { return std::nullopt; }
}
```

═══════════════════════════════════════════════════════════════════
🔼 FIN DEL PROMPT CORTO - Copia hasta aquí 🔼
═══════════════════════════════════════════════════════════════════
````

---

## ✅ Checklist de C++ Moderno

- [ ] ¿Usé RAII y smart pointers (sin new/delete)?
- [ ] ¿Usé enum class y aliases con using?
- [ ] ¿Evité copias con string_view/span cuando aplica?
- [ ] ¿Usé ranges/algoritmos en lugar de bucles manuales?
- [ ] ¿Modelé estados con optional/variant/expected?
- [ ] ¿Cada función tiene UN SOLO return y guard clauses?
- [ ] ¿Documenté con Doxygen?

---

## 📚 Referencias

- [cppreference: RAII](https://en.cppreference.com/w/cpp/language/raii)
- [cppreference: smart pointers](https://en.cppreference.com/w/cpp/memory)
- [cppreference: string_view](https://en.cppreference.com/w/cpp/string/basic_string_view)
- [cppreference: span](https://en.cppreference.com/w/cpp/container/span)
- [cppreference: ranges](https://en.cppreference.com/w/cpp/ranges)
- [cppreference: optional](https://en.cppreference.com/w/cpp/utility/optional)
- [cppreference: variant](https://en.cppreference.com/w/cpp/utility/variant)
- [cppreference: expected (C++23)](https://en.cppreference.com/w/cpp/utility/expected)

---

## 📘 Sobre esta guía

Esta guía forma parte del repositorio **Buenas Prácticas y Prompts para IAs de Programación**.

👉 **Ver más guías**: [Repositorio completo](../README.md)

---

**Autor**: [David Bueno Vallejo](https://davidbuenov.com/) | [LinkedIn](https://www.linkedin.com/in/davidbueno/) | [GitHub](https://github.com/davidbuenov)

**Licencia**: MIT

# ✅ Buenas Prácticas de Pruebas Unitarias en Java 21

Guía breve y práctica para escribir pruebas unitarias claras, mantenibles y fáciles de leer.

> **Agradecimientos**: Gracias al profesor **Antonio J. Nebro** por sus aportaciones y recomendaciones.

---

## 📑 Tabla de contenidos

- [🎯 Objetivo](#-objetivo)
- [🧪 Patrón AAA (Arrange, Act, Assert)](#-patrón-aaa-arrange-act-assert)
- [📝 Nombres de pruebas: given-when-then](#-nombres-de-pruebas-given-when-then)
- [🧰 Herramientas recomendadas: JUnit 5 y Mockito](#-herramientas-recomendadas-junit-5-y-mockito)
- [📄 Legibilidad: @DisplayName y @Nested](#-legibilidad-displayname-y-nested)
- [📋 Prompt de Sistema para IAs Generativas (Pruebas Unitarias)](#-prompt-de-sistema-para-ias-generativas-pruebas-unitarias)
    - [🤖 Cómo Usar](#-cómo-usar)
    - [📄 Versión Completa del Prompt](#-versión-completa-del-prompt)
    - [⚡ Versión Corta del Prompt (Uso Rápido)](#-versión-corta-del-prompt-uso-rápido)
- [✅ Checklist de pruebas unitarias](#-checklist-de-pruebas-unitarias)
- [📚 Referencias](#-referencias)

---

## 🎯 Objetivo

- ✅ Tests fáciles de entender a primera vista.
- ✅ Organización clara por escenarios.
- ✅ Informes de pruebas legibles para el profesor y el equipo.

---

## 🧪 Patrón AAA (Arrange, Act, Assert)

Estructura **todas** las pruebas con este patrón:

1. **Arrange**: preparar datos y dependencias.
2. **Act**: ejecutar la acción principal.
3. **Assert**: verificar el resultado.

```java
@Test
void givenValidGrades_whenCalculateAverage_thenReturnsExpectedValue() {
    // Arrange
    int[] notas = {8, 6, 10};
    CalculadoraNotas calculadora = new CalculadoraNotas();

    // Act
    double media = calculadora.media(notas);

    // Assert
    assertEquals(8.0, media, 0.001);
}
```

---

## 📝 Nombres de pruebas: given-when-then

Usa el esquema **given-when-then** en el nombre del test:

- ✅ `givenValidInput_whenProcess_thenExpectedResult`
- ❌ `test1` o `calcularMedia`

```java
@Test
void givenNullArray_whenAverage_thenReturnsZero() {
    // Arrange
    CalculadoraNotas calculadora = new CalculadoraNotas();

    // Act
    double media = calculadora.media(null);

    // Assert
    assertEquals(0.0, media, 0.001);
}
```

---

## 🧰 Herramientas recomendadas: JUnit 5 y Mockito

- **JUnit 5** para estructura de tests.
- **Mockito** para simular dependencias.

```java
@ExtendWith(MockitoExtension.class)
class ServicioUsuarioTest {

    @Mock
    RepositorioUsuario repositorio;

    @InjectMocks
    ServicioUsuario servicio;

    @Test
    void givenExistingEmail_whenCreateUser_thenThrowsException() {
        // Arrange
        when(repositorio.existe("ana@uma.es")).thenReturn(true);

        // Act + Assert
        assertThrows(EmailDuplicadoException.class, () ->
            servicio.crear("Ana", "ana@uma.es")
        );
    }
}
```

---

## 📄 Legibilidad: @DisplayName y @Nested

Usa `@DisplayName` para descripciones más claras y `@Nested` para agrupar escenarios.

```java
@DisplayName("CalculadoraNotas")
class CalculadoraNotasTest {

    @Nested
    @DisplayName("media()")
    class MediaTests {

        @Test
        @DisplayName("Dado array válido, devuelve la media correcta")
        void givenValidArray_whenAverage_thenReturnsExpectedValue() {
            // Arrange
            int[] notas = {6, 8, 10};
            CalculadoraNotas calculadora = new CalculadoraNotas();

            // Act
            double media = calculadora.media(notas);

            // Assert
            assertEquals(8.0, media, 0.001);
        }
    }
}
```

---

## 📋 Prompt de Sistema para IAs Generativas (Pruebas Unitarias)

### 🤖 Cómo Usar

Copia el prompt completo al inicio de la conversación con la IA para obtener tests claros y coherentes.

### 📄 Versión Completa del Prompt

````markdown
═══════════════════════════════════════════════════════════════════
🔽 INICIO DEL PROMPT - Copia desde aquí 🔽
═══════════════════════════════════════════════════════════════════

Eres un asistente experto en pruebas unitarias con Java 21.

Reglas obligatorias:
1) Usa JUnit 5 y Mockito.
2) Estructura todos los tests con el patrón AAA (Arrange, Act, Assert).
3) Nombra tests con esquema given-when-then.
4) Usa @DisplayName para descripciones legibles.
5) Agrupa escenarios con clases anidadas (@Nested).
6) Cada test debe ser corto, claro y sin lógica compleja.

Incluye siempre ejemplos correctos y explica brevemente la intención del test.

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

Genera tests Java 21 con JUnit 5 y Mockito. Usa AAA y nombres given-when-then. Añade @DisplayName y @Nested. Tests cortos y legibles.

═══════════════════════════════════════════════════════════════════
🔼 FIN DEL PROMPT CORTO - Copia hasta aquí 🔼
═══════════════════════════════════════════════════════════════════
````

---

## ✅ Checklist de pruebas unitarias

- [ ] ¿El nombre del test sigue **given-when-then**?
- [ ] ¿El test está separado en **Arrange / Act / Assert**?
- [ ] ¿Uso **JUnit 5**?
- [ ] ¿Uso **Mockito** cuando hay dependencias externas?
- [ ] ¿Uso `@DisplayName` para legibilidad?
- [ ] ¿Agrupo escenarios con `@Nested`?
- [ ] ¿La prueba es corta y fácil de entender?

---

## 📚 Referencias

- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Mockito Documentation](https://site.mockito.org/)
- [Good Unit Test Naming Practices](https://martinfowler.com/bliki/UnitTest.html)

---

## 📘 Sobre esta guía

Guía de pruebas unitarias para Java 21.

👉 **Ver más guías**: [Repositorio completo](../README.md)

---

**Autor**: [David Bueno Vallejo](https://davidbuenov.com/) | [LinkedIn](https://www.linkedin.com/in/davidbueno/) | [GitHub](https://github.com/davidbuenov)

**Licencia**: MIT

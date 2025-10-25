# Guía de Buenas Prácticas C# (.NET 8+) 2025

Recomendaciones modernas para C# en .NET 8+ con nullable reference types, async/await, DI, logging y testing, pensadas para colaboración con IAs generativas.

## Tabla de contenidos

- [Alcance y requisitos](#alcance-y-requisitos)
- [Nullable reference types](#nullable-reference-types)
- [Records y tipos inmutables](#records-y-tipos-inmutables)
- [Async/await y cancelación](#asyncawait-y-cancelación)
- [Dependency Injection y logging](#dependency-injection-y-logging)
- [Errores y Result pattern](#errores-y-result-pattern)
- [Testing y calidad](#testing-y-calidad)
- [Documentación XML](#documentación-xml)
- [Checklist C# .NET 8+](#checklist-c-net-8)
- [Ejemplo mínimo](#ejemplo-mínimo)
- [Prompt completo (C# .NET 8+)](#prompt-completo-c-net-8)
- [Prompt corto (C# .NET 8+)](#prompt-corto-c-net-8)
- [Referencias](#referencias)

## Alcance y requisitos

- C# 12, .NET 8+, nullable reference types activos.
- Minimal APIs / ASP.NET Core para web; DI nativo.

## Nullable reference types

- Activa en .csproj: `<Nullable>enable</Nullable>`.
- Marca parámetros/retornos nullables explícitamente (`string?`).
- Evita `!` salvo justificación clara; prefiere guard clauses.

```csharp
public record Usuario(string Nombre, string? Email);

public Usuario? ObtenerUsuario(int id)
{
    // Retorna null explícitamente si no se encuentra.
    return id > 0 ? new Usuario("Pepe", null) : null;
}
```

## Records y tipos inmutables

- Usa `record` para DTOs, eventos y value objects; `record struct` para pequeños valores.
- Prefer immutability; `with` para copias modificadas.

```csharp
public record Pedido(int Id, DateTime Fecha, decimal Total);

var p2 = pedido with { Total = 150m };
```

## Async/await y cancelación

- Propaga `CancellationToken` en operaciones I/O.
- Usa `ConfigureAwait(false)` en librerías; omite en aplicaciones ASP.NET Core.
- Un solo punto de salida por método (acumula resultado y retorna al final).

```csharp
public async Task<Usuario?> ObtenerUsuarioAsync(int id, CancellationToken ct)
{
    Usuario? resultado = null;
    if (id > 0)
    {
        await Task.Delay(100, ct);
        resultado = new Usuario("Pepe", null);
    }
    return resultado; // un solo return
}
```

## Dependency Injection y logging

- Registra servicios en `Program.cs` con DI nativa.
- Usa `ILogger<T>` para logging estructurado; evita string interpolation en logs (usa placeholders).

```csharp
public class UsuarioService(ILogger<UsuarioService> logger)
{
    public Usuario? Obtener(int id)
    {
        logger.LogInformation("Obteniendo usuario {Id}", id);
        return id > 0 ? new Usuario("Pepe", null) : null;
    }
}
```

## Errores y Result pattern

- Excepciones para casos excepcionales; Result pattern para flujos normales con fallo posible.

```csharp
public record Result<T>(bool Ok, T? Value, string? Error);

public static Result<T> Success<T>(T value) => new(true, value, null);
public static Result<T> Failure<T>(string error) => new(false, default, error);
```

## Testing y calidad

- xUnit/NUnit con FluentAssertions; Moq/NSubstitute para mocks.
- Arrange/Act/Assert; nombres descriptivos; tests independientes.

## Documentación XML

- Usa `/// <summary>` y tags (`<param>`, `<returns>`, `<remarks>`).

```csharp
/// <summary>
/// Obtiene un usuario por su identificador.
/// </summary>
/// <param name="id">Identificador del usuario.</param>
/// <param name="ct">Token de cancelación.</param>
/// <returns>Usuario encontrado o null.</returns>
public async Task<Usuario?> ObtenerAsync(int id, CancellationToken ct) { ... }
```

## Checklist C# .NET 8+

- [ ] Nullable reference types activo
- [ ] Records para DTOs/eventos
- [ ] Async/await + CancellationToken
- [ ] DI nativa + ILogger con placeholders
- [ ] Result pattern para flujos normales con fallo
- [ ] Un solo punto de salida por método
- [ ] XML docs en APIs públicas
- [ ] Tests con xUnit/NUnit + FluentAssertions

## Ejemplo mínimo

```csharp
public record CreateUserRequest(string Name, string? Email);
public record User(int Id, string Name, string? Email);

public class UserService(ILogger<UserService> logger)
{
    private int _nextId = 1;

    public Result<User> Create(CreateUserRequest req)
    {
        Result<User> result;
        if (string.IsNullOrWhiteSpace(req.Name))
        {
            result = Result<User>.Failure("Name required");
        }
        else
        {
            var user = new User(_nextId++, req.Name, req.Email);
            logger.LogInformation("User created {UserId}", user.Id);
            result = Result<User>.Success(user);
        }
        return result; // un solo return
    }
}

public record Result<T>(bool Ok, T? Value, string? Error)
{
    public static Result<T> Success(T value) => new(true, value, null);
    public static Result<T> Failure(string error) => new(false, default, error);
}
```

## Prompt completo (C# .NET 8+)

````markdown
```INICIO DEL PROMPT PARA COPIAR (C# .NET 8+)
Eres un asistente experto en C# .NET 8+. Genera código moderno, seguro y testable.

REQUISITOS
- Nullable reference types activo; marca `?` explícitamente.
- Records para DTOs; immutability cuando convenga.
- Async/await + CancellationToken en I/O.
- DI nativa; ILogger con placeholders (no string interpolation en logs).
- Result pattern para flujos normales con fallo; excepciones para casos excepcionales.
- Un solo punto de salida por método.
- XML docs en APIs públicas (summary, param, returns, remarks).

ENTREGA
- Código completo con namespaces y using mínimos.
- Tests si aplica (xUnit/NUnit + FluentAssertions).
- Explica decisiones brevemente.
```
```FIN DEL PROMPT PARA COPIAR (C# .NET 8+)
````

## Prompt corto (C# .NET 8+)

````markdown
```INICIO DEL PROMPT CORTO (C# .NET 8+)
C# .NET 8+: nullable on, records, async+CT, DI+ILogger (placeholders), Result pattern, un solo return, XML docs; tests xUnit/FluentAssertions.
```
```FIN DEL PROMPT CORTO (C# .NET 8+)
````

## Referencias

- C# Docs: <https://learn.microsoft.com/dotnet/csharp/>
- .NET 8 Docs: <https://learn.microsoft.com/dotnet/core/>
- ASP.NET Core: <https://learn.microsoft.com/aspnet/core/>

---

## 📘 Sobre esta guía

Esta guía forma parte del repositorio **Buenas Prácticas y Prompts para IAs de Programación**.

👉 **Ver más guías**: [Repositorio completo](../README.md)

---

**Autor**: [David Bueno Vallejo](https://davidbuenov.com/) | [LinkedIn](https://www.linkedin.com/in/davidbueno/) | [GitHub](https://github.com/davidbuenov)

**Licencia**: MIT

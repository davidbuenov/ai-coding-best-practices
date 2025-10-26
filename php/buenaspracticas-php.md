# 🐘 Guía de Buenas Prácticas en PHP Moderno (8.2–8.4)

Esta guía recoge prácticas profesionales de PHP pensadas para trabajar bien con IAs generativas (Copilot, ChatGPT, Gemini, Claude): código tipado, claro, con una sola salida por función y ejemplos BIEN/MAL.

---

## 📑 Tabla de contenidos

- [🎯 Objetivo](#-objetivo)
- [🧩 Requisitos](#-requisitos)
- [🆕 Características de PHP Moderno](#-características-de-php-moderno)
  - [1️⃣ Tipado Moderno y Modelos de Dominio](#1️⃣-tipado-moderno-y-modelos-de-dominio)
  - [2️⃣ Result Pattern (éxito/error explícito)](#2️⃣-result-pattern-éxitoerror-explícito)
  - [3️⃣ UN SOLO return + Guard Clauses](#3️⃣-un-solo-return--guard-clauses)
  - [4️⃣ Recursos, Errores y Transacciones](#4️⃣-recursos-errores-y-transacciones)
  - [5️⃣ match, enums y nullsafe](#5️⃣-match-enums-y-nullsafe)
  - [6️⃣ Documentación, PSR y Override](#6️⃣-documentación-psr-y-override)
  - [7️⃣ Excepciones Específicas y Contexto](#7️⃣-excepciones-específicas-y-contexto)
- [8️⃣ "Lección del Mundo Real": Versiones](#8️⃣-lección-del-mundo-real-versiones)
- [🤖 Prompt de Sistema para IAs Generativas (PHP)](#-prompt-de-sistema-para-ias-generativas-php)
  - [Cómo usar](#cómo-usar)
  - [Versión completa del prompt](#versión-completa-del-prompt)
  - [Versión corta del prompt](#versión-corta-del-prompt)
- [✅ Checklist de Código PHP Profesional](#-checklist-de-código-php-profesional)
- [📚 Referencias](#-referencias)

---

## 🎯 Objetivo

- ✅ Legibilidad y mantenibilidad
- ✅ Tipos estrictos y propiedades tipadas en todo el código
- ✅ Manejo explícito de errores (sin nulls ambiguos)
- ✅ UN SOLO return + guard clauses
- ✅ Prompts listos para proyectos serios y para uso rápido

---

## 🧩 Requisitos

- PHP 8.2+ (ideal: 8.4)
- Habilitar `declare(strict_types=1);` en TODOS los archivos
- Recomendado: PHPStan o Psalm; PHPCS/PHPCBF o PHP-CS-Fixer; Rector
- Composer con autoload PSR-4 y configuración "platform.php" cuando corresponda

---

## 🆕 Características de PHP Moderno

- PHP 8.0: union types, attributes, match, nullsafe, constructor property promotion
- PHP 8.1: enums, `readonly` properties, fibers
- PHP 8.2: `readonly` classes
- PHP 8.3: `#[Override]`, typed class constants
- Objetivo recomendado del proyecto: PHP 8.2+ (ideal 8.4)

---

## 1️⃣ Tipado Moderno y Modelos de Dominio

```php
<?php
declare(strict_types=1);

namespace App\Dominio;

use Attribute; // 8.0+

#[Attribute]
class Auditable {}

// ✅ BIEN: Value Object inmutable con propiedades readonly (8.1+)
final class Email
{
    public function __construct(
        public readonly string $valor,
    ) {
        if (!str_contains($valor, '@')) {
            throw new \InvalidArgumentException("Email inválido: {$valor}");
        }
    }
}

// ✅ BIEN: DTO tipado con promoción de propiedades y tipos unión (8.0+)
final class UsuarioDTO
{
    public function __construct(
        public int $id,
        public string $nombre,
        public string $rol = 'user',
        public string|int|null $telefono = null, // unión
    ) {}
}
```

```php
<?php
// ❌ MAL: sin tipos, mutable y con validaciones ausentes
class EmailMalo {
    public $valor; // sin tipo
}
$e = new EmailMalo();
$e->valor = 'noesemail'; // sin validación
```

---

---

## 2️⃣ Result Pattern (éxito/error explícito)

```php

En PHP no hay genéricos nativos; usa clases simples o docblocks para expresar intención.

```php
<?php
declare(strict_types=1);

namespace App\Tipos;

interface Resultado {}

final class Ok implements Resultado {
    public function __construct(public mixed $value) {}
}

final class Err implements Resultado {
    public function __construct(public string $message, public ?\Throwable $cause = null) {}
}

function parseInt(string $texto): Resultado {
    $resultado = null; // variable resultado
    if ($texto === '' || !ctype_digit($texto)) {
        $resultado = new Err('No es un entero');
    } else {
        $resultado = new Ok((int) $texto);
    }
    return $resultado; // ✅ UN SOLO return
}
```

```php
<?php
// ❌ MAL: devuelve null o lanza sin contexto
function parseIntMalo(string $texto) { // sin tipos de retorno
    return (int) $texto; // ValueError y sin mensaje útil
}
```

---

## 3️⃣ UN SOLO return + Guard Clauses

```php
<?php
use App\Tipos\{Resultado, Ok, Err};

function dividir(int|float $a, int|float $b): Resultado {
    $resultado = null;
    if ($b === 0) {
        $resultado = new Err('División por cero');
    } else {
        $resultado = new Ok($a / $b);
    }
    return $resultado; // ✅ UN SOLO return
}
```

```php
<?php
// ❌ MAL: múltiples returns dispersos
function dividirMalo($a, $b) { // sin tipos
    if ($b === 0) { return null; }
    return $a / $b;
}
```

---

## 4️⃣ Recursos, Errores y Transacciones

```php
<?php
// ✅ BIEN: PDO con excepciones, try/catch y transacción
function crearUsuario(PDO $pdo, string $nombre, Email $email): int {
    $resultado = 0;
    try {
        $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        $pdo->beginTransaction();

        $stmt = $pdo->prepare('INSERT INTO usuarios(nombre, email) VALUES (?, ?)');
        $stmt->execute([$nombre, $email->valor]);
        $resultado = (int) $pdo->lastInsertId();

        $pdo->commit();
    } catch (Throwable $e) {
        if ($pdo->inTransaction()) { $pdo->rollBack(); }
        throw new RuntimeException('Error creando usuario', 0, $e);
    }
    return $resultado; // ✅ UN SOLO return
}
```

```php
<?php
// ❌ MAL: sin transacción ni manejo de errores
function crearUsuarioMalo($pdo, $nombre, $email) {
    $pdo->query("INSERT INTO usuarios(nombre, email) VALUES ('$nombre', '$email')"); // inyección ⚠️
}
```

---

## 5️⃣ match, enums y nullsafe

```php
<?php
// ✅ BIEN: match expression (8.0+) y nullsafe (8.0+)
function clasificarEdad(int $edad): string {
    return match (true) {
        $edad < 0 => 'inválida',
        $edad < 18 => 'menor',
        $edad < 65 => 'adulto',
        default => 'senior',
    };
}

// ✅ Enum (8.1+) con método
enum Rol: string { case Admin = 'admin'; case User = 'user'; }

function esAdmin(Rol $rol): bool { return $rol === Rol::Admin; }

// ✅ Nullsafe
$ciudad = $usuario?->direccion?->ciudad ?? 'desconocida';
```

```php
<?php
// ❌ MAL: switch verboso y ifs anidados
function clasificarEdadMala($edad) {
    if ($edad < 0) { return 'inválida'; }
    if ($edad < 18) { return 'menor'; }
    if ($edad < 65) { return 'adulto'; }
    return 'senior';
}
```

---

## 6️⃣ Documentación, PSR y Override

- Sigue PSR-12 para estilo y PSR-4 para autoload.
- Usa PHPDoc donde el tipo no quede obvio o para documentar excepciones.
- Usa `#[\Override]` (8.3+) para métodos que sobrescriben, ayuda a las herramientas.

```php
<?php
interface Repo {
    /** @throws DomainException si no existe */
    public function buscarPorId(int $id): UsuarioDTO;
}

final class RepoSql implements Repo {
    #[\Override]
    public function buscarPorId(int $id): UsuarioDTO { /* ... */ }
}
```

---

## 7️⃣ Excepciones Específicas y Contexto

```php
<?php
final class UsuarioNoEncontrado extends DomainException
{
    public function __construct(public readonly int $id)
    { parent::__construct("Usuario no encontrado: {$id}"); }
}

function obtener(int $id, Repo $repo): UsuarioDTO
{
    $resultado = null;
    try {
        $resultado = $repo->buscarPorId($id);
    } catch (Throwable $e) {
        throw new UsuarioNoEncontrado($id);
    }
    return $resultado; // ✅ UN SOLO return
}
```

---

## 8️⃣ "Lección del Mundo Real": Versiones

- Define versión mínima real: PHP 8.2+ (ideal 8.4). Evita usar rasgos no disponibles si el runtime no los soporta.
- Valida versión cuando sea crítico: `if (PHP_VERSION_ID < 80200) { /* error */ }`.
- En `composer.json` usa `"platform": {"php": "8.2"}` para fijar compatibilidad en CI.

```php
<?php
if (PHP_VERSION_ID < 80200) {
    throw new RuntimeException('Se requiere PHP 8.2+');
}
```

---

## 🤖 Prompt de Sistema para IAs Generativas (PHP)

### Cómo usar

Usa la versión completa para proyectos y la corta para peticiones rápidas.

### Versión completa del prompt

````markdown
═══════════════════════════════════════════════════════════════════
🔽 INICIO DEL PROMPT - Copia desde aquí 🔽
═══════════════════════════════════════════════════════════════════

# Prompt de Sistema para Generación de Código PHP 8.2–8.4 Profesional

Eres un asistente PHP que genera código limpio, tipado y mantenible.

## Reglas OBLIGATORIAS (PHP 8.2–8.4)

1. Tipado estricto y propiedades tipadas
   - ✅ `declare(strict_types=1);` en TODOS los archivos
   - ✅ Tipos escalares, unión/intersección, `readonly`, `enum`

2. Result pattern
   - ✅ Retorna `Ok|Err` en errores recuperables
   - ✅ No retornes `null` ambiguo; no silencies excepciones

3. UN SOLO return + Guard Clauses
   - ✅ Un único `return` por función
   - ✅ Validaciones planas, evita pirámides

4. Recursos y errores
   - ✅ PDO con excepciones, transacciones, `try/catch/finally`
   - ✅ Prepara sentencias (SQL parametrizado)

5. Estilo y docs
   - ✅ PSR-12/PSR-4, PHPDoc donde aporte
   - ✅ `#[\Override]` (8.3+) al sobrescribir métodos

6. Moderno > tradicional
   - ✅ `match`, `enum`, nullsafe, promoted properties
   - ✅ Clases `final` por defecto salvo justificación

## Ejemplo correcto

```php
<?php
declare(strict_types=1);

final class Ok { public function __construct(public mixed $value) {} }
final class Err { public function __construct(public string $message) {} }

function dividir(int|float $a, int|float $b): Ok|Err {
    $resultado = null;
    if ($b === 0) { $resultado = new Err('División por cero'); }
    else { $resultado = new Ok($a / $b); }
    return $resultado; // UN SOLO return
}
```

## Importante

- PHP 8.2+ mínimo (ideal 8.4)
- Una función = un solo return
- Prioriza claridad, tipado y manejo explícito de errores

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

Genera PHP 8.2–8.4 profesional con estas reglas:
1) `declare(strict_types=1);` + tipos en todo
2) Result pattern (Ok | Err), sin `null` ambiguo
3) UN SOLO return + guard clauses
4) PDO con transacciones y SQL parametrizado
5) Usa `match`, `enum`, nullsafe; PSR-12/PSR-4; `#[\\Override]`

Ejemplo:
```php
final class Ok { public function __construct(public mixed $value) {} }
final class Err { public function __construct(public string $message) {} }
function parseo(string $s): Ok|Err {
  $r = null; if ($s === '') { $r = new Err('vacío'); } else { $r = new Ok($s); }
  return $r; // UN SOLO return
}
```

═══════════════════════════════════════════════════════════════════
🔼 FIN DEL PROMPT CORTO - Copia hasta aquí 🔼
═══════════════════════════════════════════════════════════════════
````

---

## ✅ Checklist de Código PHP Profesional

- [ ] ¿Declaré `strict_types=1` en todos los archivos?
- [ ] ¿Usé tipos estrictos, uniones/intersecciones y propiedades tipadas?
- [ ] ¿Modelé estados con `Ok|Err` cuando aplica?
- [ ] ¿Cada función tiene UN SOLO return y guard clauses?
- [ ] ¿Usé PDO con excepciones, transacciones y SQL parametrizado?
- [ ] ¿Aproveché `match`, `enum`, nullsafe y `readonly`?
- [ ] ¿Seguí PSR-12/PSR-4 y documenté con PHPDoc donde aporta?
- [ ] ¿Usé `#[\\Override]` al sobrescribir métodos (8.3+)?
- [ ] ¿Fijé versión mínima y validé en runtime/CI si corresponde?

---

## 📚 Referencias

- PSR-12, PSR-4 — PHP-FIG
- PHP Manual — Types, Enums, Attributes, Match, PDO
- PHPStan / Psalm — Análisis estático
- PHPCS / PHP-CS-Fixer — Estilo
- Rector — Migraciones de código

---

## 📘 Sobre esta guía

Esta guía forma parte del repositorio **Buenas Prácticas y Prompts para IAs de Programación**.

👉 **Ver más guías**: [Repositorio completo](../README.md)

---

**Autor**: [David Bueno Vallejo](https://davidbuenov.com/) | [LinkedIn](https://www.linkedin.com/in/davidbueno/) | [GitHub](https://github.com/davidbuenov)

**Licencia**: MIT

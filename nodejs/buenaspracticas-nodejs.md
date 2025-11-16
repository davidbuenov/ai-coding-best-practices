# 🚀 Guía de Buenas Prácticas en Node.js Moderno (JavaScript, Node 20+)

Esta guía cubre prácticas profesionales para proyectos backend en **Node.js (JavaScript sin TypeScript)** optimizados para trabajo con IAs (Copilot, ChatGPT, Gemini, Claude). Foco en claridad, mantenibilidad, seguridad y despliegue. Incluye ejemplos BIEN/MAL y prompts listos.

---

## 📑 Tabla de contenidos

- [🎯 Objetivo](#-objetivo)
- [🧩 Requisitos](#-requisitos)
- [🆕 Características de Node.js Moderno](#-características-de-nodejs-moderno)
  - [1️⃣ Módulos ESM y Estructura de Proyecto](#1️⃣-módulos-esm-y-estructura-de-proyecto)
  - [2️⃣ Gestión de Dependencias y package.json Sano](#2️⃣-gestión-de-dependencias-y-packagejson-sano)
  - [3️⃣ Configuración y Variables de Entorno](#3️⃣-configuración-y-variables-de-entorno)
  - [4️⃣ Manejo de Errores y Result Pattern](#4️⃣-manejo-de-errores-y-result-pattern)
  - [5️⃣ Logging Estructurado y Trazabilidad](#5️⃣-logging-estructurado-y-trazabilidad)
  - [6️⃣ Seguridad Básica en API](#6️⃣-seguridad-básica-en-api)
  - [7️⃣ Rendimiento y Concurrencia Controlada](#7️⃣-rendimiento-y-concurrencia-controlada)
  - [8️⃣ Streams y Operaciones I/O Eficientes](#8️⃣-streams-y-operaciones-io-eficientes)
  - [9️⃣ Graceful Shutdown y Señales](#9️⃣-graceful-shutdown-y-señales)
  - [🔟 Testing y Calidad (sin TS)](#-testing-y-calidad-sin-ts)
- [🤖 Prompt de Sistema para IAs Generativas (Node.js)](#-prompt-de-sistema-para-ias-generativas-nodejs)
  - [Versión completa del prompt](#versión-completa-del-prompt)
  - [Versión corta del prompt](#versión-corta-del-prompt)
- [✅ Checklist de Código Node.js Profesional](#-checklist-de-código-nodejs-profesional)
- [📚 Referencias](#-referencias)

---

## 🎯 Objetivo

- ✅ Código claro y modular (módulos ESM)
- ✅ Manejo explícito y centralizado de errores
- ✅ Seguridad mínima razonable aplicada por defecto
- ✅ Logging estructurado y trazabilidad
- ✅ Prompts listos para acelerar desarrollo asistido por IA

---

## 🧩 Requisitos

- Node.js 20+ (fetch nativo, test runner integrado, AbortController estable)
- Gestor de paquetes: npm 9+, pnpm 8+ o yarn moderno (preferible pnpm por eficiencia)
- Linters recomendados: ESLint + Prettier

---

## 🆕 Características de Node.js Moderno

- **ESM por defecto**: `"type": "module"` en `package.json`
- **fetch API nativa**: ya no dependes de `node-fetch`
- **AbortController** integrado para cancelar peticiones/operaciones
- **Test runner experimental** en Node 20 (`node --test`)
- **Streams y promesas**: `stream/promises`, `fs/promises`
- **Top-level await** soportado en ESM

---

## 1️⃣ Módulos ESM y Estructura de Proyecto

```bash
mi-proyecto/
  package.json
  src/
    index.js
    config/
      env.js
    api/
      routes.js
      controllers/
        userController.js
    services/
      userService.js
    lib/
      logger.js
  test/
    user.test.js
```

`package.json` mínimo:

```jsonc
{
  "name": "mi-proyecto-node",
  "version": "0.1.0",
  "type": "module",
  "main": "src/index.js",
  "scripts": {
    "dev": "node --watch src/index.js",
    "start": "NODE_ENV=production node src/index.js",
    "test": "node --test",
    "lint": "eslint ."
  }
}
```

```js
// ✅ BIEN: ESM, import explícito
import { crearServidor } from './api/routes.js';
import { logger } from './lib/logger.js';

crearServidor().listen(3000, () => {
  logger.info('Servidor escuchando en puerto 3000');
});
```

```js
// ❌ MAL: CommonJS mezclado y rutas mágicas
const express = require('express'); // Mezcla estilos
const srv = express();
srv.listen(3000);
```

---

## 2️⃣ Gestión de Dependencias y package.json Sano

- Usa versiones semver explícitas, evita dependencias abandonadas
- Auditoría: `npm audit --omit=dev`
- Evita instalar paquetes para cosas triviales (e.g. `left-pad`)
- Usa scripts cortos y claros; evita cadenas complejas difíciles de mantener

```bash
# ✅ BIEN
npm install express zod pino dotenv helmet
```

```bash
# ❌ MAL: instalar sin necesidad
npm install moment lodash axios --save   # (quizás puedes usar Intl, fetch, utilidades propias)
```

---

## 3️⃣ Configuración y Variables de Entorno

```js
// src/config/env.js
import 'dotenv/config';
import { z } from 'zod';

const EnvSchema = z.object({
  NODE_ENV: z.enum(['development','test','production']),
  PORT: z.string().regex(/^[0-9]+$/).transform(Number).default('3000'),
  DB_URL: z.string().url(),
  LOG_LEVEL: z.enum(['debug','info','warn','error']).default('info')
});

const parsed = EnvSchema.safeParse(process.env);
if (!parsed.success) {
  console.error('❌ Variables de entorno inválidas', parsed.error.format());
  process.exit(1); // Fail fast
}

export const env = parsed.data;
```

```js
// ✅ BIEN: uso centralizado
import { env } from '../config/env.js';
console.log('Puerto:', env.PORT);
```

```js
// ❌ MAL: acceder disperso y sin validación
console.log(process.env.PORT);
```

---

## 4️⃣ Manejo de Errores y Result Pattern

```js
// src/lib/result.js
export class Ok {
  constructor(value){ this.value = value; }
}
export class Err {
  constructor(message, cause){ this.message = message; this.cause = cause; }
}
export const isOk = r => r instanceof Ok;
export const isErr = r => r instanceof Err;
```

```js
// src/services/userService.js
import { Ok, Err } from '../lib/result.js';

export async function getUserById(id, repo){
  try {
    const user = await repo.findById(id);
    if (!user) return new Err('Usuario no encontrado');
    return new Ok(user);
  } catch (e){
    return new Err('Fallo accediendo a usuario', e);
  }
}
```

```js
// ✅ BIEN: manejo explícito
const r = await getUserById('123', repo);
if (r instanceof Err){
  logger.warn({ err: r }, 'No se pudo obtener usuario');
} else {
  logger.info({ user: r.value }, 'Usuario obtenido');
}
```

```js
// ❌ MAL: try/catch vacío
try {
  const u = await repo.findById(id);
} catch (e) {}
```

---

## 5️⃣ Logging Estructurado y Trazabilidad

```js
// src/lib/logger.js
import pino from 'pino';
export const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: process.env.NODE_ENV === 'development' ? {
    target: 'pino-pretty', options: { colorize: true }
  } : undefined
});
```

```js
// ✅ BIEN: contexto
logger.info({ route: '/users', method: 'GET', durationMs: 42 }, 'Petición completada');
```

```js
// ❌ MAL: string suelto
console.log('Petición completada');
```

---

## 6️⃣ Seguridad Básica en API

- Usa `helmet` para cabeceras seguras
- Valida input con `zod`/`joi`
- Limita tamaño de payload (`express.json({ limit: '1mb' })`)
- Rate limiting (e.g. `express-rate-limit`)
- Nunca loguees secretos o tokens
- Sanitiza salida HTML si generas vistas

```js
// ✅ BIEN
import express from 'express';
import helmet from 'helmet';
import rateLimit from 'express-rate-limit';
const app = express();
app.use(helmet());
app.use(express.json({ limit: '1mb' }));
app.use(rateLimit({ windowMs: 60_000, max: 100 }));
```

```js
// ❌ MAL: sin protección
const app = require('express')();
app.use(require('express').json());
```

---

## 7️⃣ Rendimiento y Concurrencia Controlada

- Usa `AbortController` para timeouts
- Evita crear pools de conexión ad-hoc sin reusar
- Usa `Promise.allSettled` cuando no quieres fallar todo el lote
- Crea funciones idempotentes para tareas repetidas

```js
// ✅ BIEN: timeout fetch
const controller = new AbortController();
const t = setTimeout(() => controller.abort(), 5_000);
try {
  const res = await fetch('https://api.ejemplo.com/datos', { signal: controller.signal });
  const data = await res.json();
} finally { clearTimeout(t); }
```

```js
// ❌ MAL: sin control de timeout
const res = await fetch('https://api.ejemplo.com/datos');
```

---

## 8️⃣ Streams y Operaciones I/O Eficientes

```js
import { createReadStream, createWriteStream } from 'node:fs';
import { pipeline } from 'node:stream/promises';

// ✅ BIEN: pipeline controlado
await pipeline(
  createReadStream('entrada.csv'),
  transformCsvStream(),
  createWriteStream('salida.json')
);
```

```js
// ❌ MAL: cargar todo en memoria
const contenido = await fs.promises.readFile('entrada.csv','utf-8');
procesar(contenido); // riesgo de alto consumo
```

---

## 9️⃣ Graceful Shutdown y Señales

```js
// src/index.js
import { crearServidor } from './api/routes.js';
import { logger } from './lib/logger.js';

const server = crearServidor().listen(process.env.PORT || 3000, () => {
  logger.info('Servidor iniciado');
});

function shutdown(signal){
  logger.warn({ signal }, 'Iniciando apagado gracioso');
  server.close(err => {
    if (err) {
      logger.error({ err }, 'Error cerrando servidor');
      process.exit(1);
    }
    logger.info('Servidor cerrado correctamente');
    process.exit(0);
  });
}

['SIGINT','SIGTERM'].forEach(s => process.on(s, () => shutdown(s)));
```

```js
// ❌ MAL: salir brusco
delete process.env.DB_URL;
process.exit(0);
```

---

## 🔟 Testing y Calidad (sin TS)

- Usa el test runner nativo: `node --test` o Jest/Vitest si necesitas más
- Aísla side-effects (no pegar a la base en unit tests)
- Usa mocks ligeros para servicios externos
- ESLint: reglas para `no-unused-vars`, `eqeqeq`, `no-implicit-coercion`
- Prettier para formato consistente

```js
// ✅ BIEN: test con runner nativo (Node 20)
import assert from 'node:assert';
import { describe, it } from 'node:test';
import { suma } from '../src/lib/math.js';

describe('suma', () => {
  it('suma dos números', () => {
    assert.strictEqual(suma(2,3),5);
  });
});
```

```js
// ❌ MAL: test improvisado
if (suma(2,3) !== 5) throw new Error('Falla');
```

---

## 🤖 Prompt de Sistema para IAs Generativas (Node.js)

### Versión completa del prompt

````markdown
Eres un generador experto de código backend Node.js usando JavaScript (NO TypeScript). Sigue estas reglas:

1. Usa módulos ESM ("type": "module") y `import`/`export`
2. Escribe funciones pequeñas, puras cuando sea posible y con UN SOLO `return` (con guard clauses)
3. Maneja errores con try/catch específicos o un pequeño Result pattern (Ok/Err)
4. Valida `process.env` mediante un esquema (Zod/Joi) centralizado
5. Logging estructurado (pino) con contexto (no `console.log` salvo debug puntual)
6. Usa `AbortController` para timeout en fetch/operaciones externas
7. Evita dependencias innecesarias y comenta cuando uses una
8. NO mezcles CommonJS y ESM
9. Incluye BIEN/MAL si procede para mostrar contraste
10. No generes TypeScript, solo JavaScript moderno
11. Al devolver código, solo un bloque por archivo y sin explicaciones extra

Ejemplo de Result pattern minimalista:
```js
export class Ok { constructor(v){ this.value = v; } }
export class Err { constructor(m,c){ this.message = m; this.cause = c; } }
```

IMPORTANTE: El código debe ser válido para Node 20+, listo para ejecutar.
````

### Versión corta del prompt

````markdown
Genera código Node.js (JavaScript, ESM) para [descripción]. Usa Ok/Err para errores, valida env si aplica, logging con pino, sin TypeScript.
````

---

## ✅ Checklist de Código Node.js Profesional

- [ ] ¿`type": "module"` en package.json?
- [ ] ¿Imports relativos claros sin rutas mágicas?
- [ ] ¿Variables de entorno validadas con esquema?
- [ ] ¿Errores manejados con contexto (mensaje + cause)?
- [ ] ¿Logging estructurado (sin `console.log` suelto)?
- [ ] ¿Timeouts y aborts en fetch/IO externo?
- [ ] ¿Graciaful shutdown implementado (SIGINT/SIGTERM)?
- [ ] ¿Tests presentes (al menos unidad)?
- [ ] ¿Scripts npm para dev/lint/test claros?
- [ ] ¿Dependencias auditadas y mínimas?

---

## 📚 Referencias

- [Node.js Docs](https://nodejs.org/docs/latest/api/)
- [Fetch API](https://developer.mozilla.org/docs/Web/API/Fetch_API)
- [AbortController](https://developer.mozilla.org/docs/Web/API/AbortController)
- [Pino logger](https://getpino.io/)
- [Zod](https://zod.dev/)
- [Express](https://expressjs.com/)

---

**Autor**: [David Bueno Vallejo](https://davidbuenov.com/) | [LinkedIn](https://www.linkedin.com/in/davidbueno/) | [GitHub](https://github.com/davidbuenov)

**Licencia**: MIT

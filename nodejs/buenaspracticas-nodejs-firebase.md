# ☁️ Guía de Buenas Prácticas: Firebase Cloud Functions (Node.js 20+, JavaScript CommonJS)

Esta guía adapta los principios de la guía general de Node.js al contexto específico de **Firebase Cloud Functions (2nd gen)** cuando usas **JavaScript sin TypeScript**. Aquí priman la compatibilidad con la plataforma, los tiempos de arranque (cold start), la observabilidad nativa y la simplicidad operacional.

---

## 📑 Tabla de Contenidos

- [🎯 Objetivo](#-objetivo)
- [⚠️ Diferencias clave respecto a la guía Node.js general](#️-diferencias-clave-respecto-a-la-guía-nodejs-general)
- [🧩 Requisitos y contexto Firebase](#-requisitos-y-contexto-firebase)
  - [1️⃣ CommonJS vs ESM](#1️⃣-commonjs-vs-esm)
  - [2️⃣ Logging: usar functions.logger](#2️⃣-logging-usar-functionslogger)
  - [3️⃣ Configuración y secretos](#3️⃣-configuración-y-secretos)
  - [4️⃣ Manejo de errores y respuestas HTTP](#4️⃣-manejo-de-errores-y-respuestas-http)
  - [5️⃣ Cold Start y rendimiento](#5️⃣-cold-start-y-rendimiento)
  - [6️⃣ Seguridad y validaciones](#6️⃣-seguridad-y-validaciones)
  - [7️⃣ Idempotencia y reintentos](#7️⃣-idempotencia-y-reintentos)
  - [8️⃣ Estructura de archivos recomendada](#8️⃣-estructura-de-archivos-recomendada)
  - [9️⃣ Guard Clauses y UN SOLO return](#9️⃣-guard-clauses-y-un-solo-return)
  - [10 JSDoc y claridad de API](#10-jsdoc-y-claridad-de-api)
- [🤖 Prompts para IAs generativas (Firebase)](#-prompts-para-ias-generativas-firebase)
  - [Versión completa](#versión-completa)
  - [Versión corta](#versión-corta)
- [✅ Checklist Cloud Functions](#-checklist-cloud-functions)
- [📚 Referencias](#-referencias)

---

## 🎯 Objetivo

- ✅ Mantener compatibilidad plena con Firebase Functions (CommonJS)
- ✅ Reducir complejidad innecesaria (evitar dependencias superfluas)
- ✅ Mejorar claridad (JSDoc, guard clauses, separación de responsabilidades)
- ✅ Aprovechar logging nativo y configuración oficial
- ✅ Minimizar tiempos de arranque y consumo de memoria

---

## ⚠️ Diferencias clave respecto a la guía Node.js general

| Aspecto | Guía Node.js General | Firebase Cloud Functions |
|---------|----------------------|---------------------------|
| Módulos | ESM (`import/export`) | CommonJS (`require/module.exports`) |
| Logging | pino estructurado | `functions.logger` (nativo) |
| Env Validation | Zod/Joi + process.env | `functions.config()` + mínimo process.env |
| Result Pattern | Ok/Err opcional | Normalmente res.status() + manejo explícito |
| Long-running | Servidor persistente | Ejecuciones efímeras (stateless) |
| Graceful Shutdown | Manejo de señales | No aplica (lifecycle gestionado por plataforma) |
| Streams heavy | Sí según caso | Evitar cargas innecesarias en cold start |

---

## 🧩 Requisitos y contexto Firebase

- Node.js runtime administrado por Firebase (elige versión 20+ en configuración si disponible)
- Proyecto con `firebase-tools` instalado localmente
- Usa 2nd gen para mejor escalado y rendimiento (cuando aplique)

---

## 1️⃣ CommonJS vs ESM

Firebase Functions sigue usando CommonJS en la mayoría de ejemplos oficiales. No fuerces ESM salvo que toda la toolchain esté preparada.

```js
// ✅ BIEN (index.js)
const functions = require('firebase-functions');
const express = require('express');
const { createUserController } = require('./controllers/user');

const app = express();
app.post('/users', createUserController);

exports.api = functions.https.onRequest(app);
```

```js
// ❌ MAL: mezclar ESM
import express from 'express'; // Puede romper despliegue
export const api = functions.https.onRequest(app);
```

---

## 2️⃣ Logging: usar functions.logger

No añadas pino sin necesidad: Firebase ya captura severidades y estructura.

```js
// ✅ BIEN
const functions = require('firebase-functions');
functions.logger.info('Usuario creado', { userId: id });
functions.logger.warn('Input inválido', { raw: body });
functions.logger.error('Fallo en servicio externo', { error: err.message });
```

```js
// ❌ MAL
console.log('Usuario creado', id); // No añade contexto estructurado
```

---

## 3️⃣ Configuración y secretos

Usa `functions.config()` para parámetros en vez de acceder disperso a `process.env`.

```js
// ✅ BIEN
const functions = require('firebase-functions');
const config = functions.config();
const API_KEY = config.external?.apikey; // Definido vía: firebase functions:config:set external.apikey="..."

function buildUrl(id){
  if (!API_KEY) throw new Error('API_KEY no configurada');
  return `https://api.ejemplo.com/entity/${id}?key=${API_KEY}`;
}
```

```js
// ❌ MAL
const API_KEY = process.env.API_KEY; // Disperso y sin control central
```

---

## 4️⃣ Manejo de errores y respuestas HTTP

Evita patrones exagerados (Result pattern) si el flujo HTTP ya es claro.

```js
// ✅ BIEN
function createUserController(req, res){
  if (!req.body || !req.body.email){
    return res.status(400).json({ error: 'email requerido' }); // guard clause
  }
  try {
    const user = { id: Date.now().toString(), email: req.body.email };
    return res.status(201).json({ user });
  } catch (e){
    functions.logger.error('Error creando usuario', { error: e.message });
    return res.status(500).json({ error: 'interno' });
  }
}
```

```js
// ❌ MAL
function createUserController(req, res){
  try {
    // sin validación, posible crash
    const user = { id: Date.now(), email: req.body.email };
    res.status(201).json(user);
  } catch(e){ res.send(e); }
}
```

---

## 5️⃣ Cold Start y rendimiento

- Coloca inicializaciones pesadas (SDKs externos) fuera de control si se reutilizan entre invocaciones, pero evita cargar lo que no se usa.
- No hagas lectura masiva de archivos en cada petición.
- Mantén dependencias al mínimo.

```js
// ✅ BIEN: carga condicional
let client;
function getClient(){
  if (!client){
    const { ExternalClient } = require('external-sdk');
    client = new ExternalClient();
  }
  return client;
}
```

```js
// ❌ MAL: inicializar cada vez
function handler(req, res){
  const { ExternalClient } = require('external-sdk');
  const c = new ExternalClient();
  // ...
}
```

---

## 6️⃣ Seguridad y validaciones

- Valida input siempre (tipo, formato, requerido)
- No expongas stack traces al cliente
- Usa reglas de Firestore/Storage adecuadas (esto es fuera del código de función)
- Sanear datos si envías HTML (evita XSS en plantillas)

```js
// ✅ BIEN
function parseCreateUser(body){
  if (!body || typeof body.email !== 'string') throw new Error('email inválido');
  return { email: body.email.trim().toLowerCase() };
}
```

---

## 7️⃣ Idempotencia y reintentos

Las funciones pueden reintentarse (según configuración). Evita efectos duplicados.

```js
// ✅ BIEN: comprobar si ya existe
async function processPayment(event){
  if (await paymentAlreadyProcessed(event.id)) return; // idempotencia
  await createPaymentRecord(event);
}
```

---

## 8️⃣ Estructura de archivos recomendada

```
functions/
  index.js
  controllers/
    user.js
  lib/
    validation.js
    external.js
```

- `index.js`: registro de endpoints/exports
- `controllers/`: lógica HTTP
- `lib/`: helpers y adaptadores externos

---

## 9️⃣ Guard Clauses y UN SOLO return

Simplifica la lectura evitando pirámides de if.

```js
// ✅ BIEN
function handler(req, res){
  if (!req.query.id) return res.status(400).json({ error: 'id requerido' });
  try {
    const data = obtener(req.query.id);
    return res.status(200).json({ data });
  } catch (e){
    functions.logger.error('Error en handler', { error: e.message });
    return res.status(500).json({ error: 'interno' });
  }
}
```

---

## 10 JSDoc y claridad de API

```js
/**
 * Crea un usuario nuevo.
 * @param {import('express').Request} req
 * @param {import('express').Response} res
 * @returns {import('express').Response}
 */
function createUserController(req, res){ /* ... */ }
```

---

## 🤖 Prompts para IAs generativas (Firebase)

### Versión completa

````markdown
Eres un generador experto de código para Firebase Cloud Functions (2nd gen) usando Node.js 20+ (JavaScript) con CommonJS. Sigue estas reglas:

1. Usa require/module.exports (NO import/export)
2. Usa functions.logger para logging (NO console.log ni pino)
3. Valida input de request (sessionId, message, etc.) con checks simples al inicio
4. Emplea guard clauses y UN SOLO return por función
5. Evita dependencias innecesarias; usa solo las que ya estén en package.json
6. Maneja errores con try/catch específicos; nunca expongas stack traces al cliente
7. Inicializa recursos externos (DB, API clients) lazy para reducir cold start
8. Documenta endpoints y funciones complejas con JSDoc
9. No implementes Result pattern salvo petición explícita
10. Extrae constantes de configuración (límites, mensajes, URLs) al inicio del archivo
11. Devuelve SOLO el código final sin explicaciones extra, listo para ejecutar

IMPORTANTE: Código compatible con Firebase Cloud Functions 2nd gen, Node 20+.
````

### Versión corta

````markdown
Genera código Firebase Functions (CommonJS) para [tarea]. Logging con functions.logger, validación input mínima, guard clauses, sin ESM.
````

---

## ✅ Checklist Cloud Functions

- [ ] ¿Uso de CommonJS (require/module.exports)?
- [ ] ¿Logging con functions.logger y niveles adecuados?
- [ ] ¿Validación de body/params realizada?
- [ ] ¿Guard clauses presentes para entradas inválidas?
- [ ] ¿Sin dependencias superfluas?
- [ ] ¿Inicialización lazy de recursos pesados?
- [ ] ¿Sin exposición de stack traces al cliente?
- [ ] ¿Idempotencia en operaciones críticas?
- [ ] ¿JSDoc en controladores principales?

---

## 📚 Referencias

- [Firebase Functions Docs](https://firebase.google.com/docs/functions)
- [Logging](https://firebase.google.com/docs/functions/writing-and-viewing-logs)
- [Config (functions.config)](https://firebase.google.com/docs/functions/config-env)
- [2nd gen deployment](https://firebase.google.com/docs/functions/2nd-gen)

---

**Autor**: [David Bueno Vallejo](https://davidbuenov.com/) | [LinkedIn](https://www.linkedin.com/in/davidbueno/) | [GitHub](https://github.com/davidbuenov)

**Licencia**: MIT

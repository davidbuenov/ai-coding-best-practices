# Guía de Buenas Prácticas TypeScript + Node.js (2025)

Prácticas modernas para servicios y CLIs en Node.js con TypeScript estricto, orientadas a colaborar con IAs generativas.

## Tabla de contenidos

- [Alcance y versiones](#alcance-y-versiones)
- [tsconfig y estructura](#tsconfig-y-estructura)
- [Tipos y modelado](#tipos-y-modelado)
- [Validación de entradas](#validación-de-entradas)
- [Async, AbortController y tiempo de espera](#async-abortcontroller-y-tiempo-de-espera)
- [Errores y logging](#errores-y-logging)
- [Entorno y configuración](#entorno-y-configuración)
- [Testing y calidad](#testing-y-calidad)
- [Checklist TS + Node 2025](#checklist-ts--node-2025)
- [Ejemplo mínimo](#ejemplo-mínimo)
- [Prompt completo (TS + Node)](#prompt-completo-ts--node)
- [Prompt corto (TS + Node)](#prompt-corto-ts--node)
- [Referencias](#referencias)

## Alcance y versiones

- TypeScript 5.x, Node.js 20+ (ESM nativo), npm o pnpm.
- ESM con "type": "module" y módulo TS dirigido a ES2022+.

## tsconfig y estructura

- Activa strict completo: strict, noImplicitAny, noUncheckedIndexedAccess, exactOptionalPropertyTypes.
- moduleResolution: node16/bundler según tooling; skipLibCheck true para velocidad.
- Estructura clara: src/ (código), test/ (pruebas), scripts/ (tareas), dist/ (build).

## Tipos y modelado

- Nunca any; prefiere unknown y refina.
- Uniones discriminadas; tipos de resultado con éxito/error.
- TSDoc en funciones públicas.

```ts
export type Ok<T> = { ok: true; value: T };
export type Err<E> = { ok: false; error: E };
export type Result<T, E = Error> = Ok<T> | Err<E>;

export function ok<T>(value: T): Ok<T> { return { ok: true, value }; }
export function err<E>(error: E): Err<E> { return { ok: false, error }; }
```

## Validación de entradas

- Valida inputs en los bordes (HTTP, CLI) con Zod/valibot; nunca confíes en el tipo en tiempo de compilación.

```ts
import { z } from 'zod';

export const CreateUser = z.object({
  name: z.string().min(1),
  age: z.number().int().gte(18)
});
export type CreateUser = z.infer<typeof CreateUser>;
```

## Async, AbortController y tiempo de espera

- Propaga AbortSignal; implementa timeouts.

```ts
export async function withTimeout<T>(p: Promise<T>, ms: number, signal?: AbortSignal): Promise<T> {
  const ac = new AbortController();
  const timer = setTimeout(() => ac.abort("timeout"), ms);
  const onAbort = () => ac.abort("aborted");
  signal?.addEventListener('abort', onAbort, { once: true });
  try {
    return await Promise.race([p, new Promise<T>((_, rej) => ac.signal.addEventListener('abort', () => rej(new Error(String(ac.signal.reason))))) ]);
  } finally {
    clearTimeout(timer);
    signal?.removeEventListener('abort', onAbort);
  }
}
```

## Errores y logging

- Usa Error con cause; registra estructuradamente (pino/winston).
- Un solo punto de salida por función; no apiles returns.

```ts
try {
  // ...
} catch (e) {
  throw new Error('fallo al procesar', { cause: e });
}
```

## Entorno y configuración

- Valida process.env con Zod antes de usar.

```ts
import { z } from 'zod';
const Env = z.object({ PORT: z.string().default('3000').transform(Number) });
export const env = Env.parse(process.env);
```

## Testing y calidad

- Vitest para unit; Supertest para HTTP; ESLint + Prettier; Type checking en CI.

## Checklist TS + Node 2025

- [ ] tsconfig estricto y ESM
- [ ] Sin any; Result y uniones discriminadas
- [ ] Zod/valibot en inputs
- [ ] AbortController y timeouts
- [ ] Error cause + logging estructurado
- [ ] process.env validado
- [ ] Vitest + ESLint + Prettier
- [ ] TSDoc en APIs públicas

## Ejemplo mínimo

```ts
import { z } from 'zod';
import http from 'node:http';

const CreateUser = z.object({ name: z.string().min(1), age: z.number().int().gte(18) });

const server = http.createServer(async (req, res) => {
  if (req.method === 'POST' && req.url === '/users') {
    const chunks: Buffer[] = [];
    for await (const c of req) chunks.push(c as Buffer);
    const body = JSON.parse(Buffer.concat(chunks).toString('utf8'));
    const data = CreateUser.parse(body);
    res.writeHead(201, { 'content-type': 'application/json' });
    res.end(JSON.stringify({ ok: true, user: data }));
    return;
  }
  res.writeHead(404).end();
});

server.listen(3000);
```

## Prompt completo (TS + Node)

````markdown
```INICIO DEL PROMPT PARA COPIAR (TS + Node)
Eres un asistente experto en TypeScript + Node.js (2025). Genera código ESM con tsconfig estricto y práctica profesional.

REQUISITOS
- TypeScript estricto: sin any; uniones discriminadas; Result pattern; TSDoc.
- Validación en bordes: Zod/valibot; nunca confíes solo en tipos TS.
- Async: AbortController + timeouts; sin fugas de listeners.
- Errores: Error con cause; logging estructurado.
- Entorno: valida process.env con Zod.
- Testing y linting: Vitest, ESLint, Prettier.

CONTRATO
- Estructura clara (src/, test/); ESM; imports mínimos.
- Un solo punto de salida por función.
- Documenta funciones públicas con TSDoc.

A PARTIR DE AQUÍ, IMPLEMENTA EL PEDIDO DEL USUARIO RESPETANDO TODO LO ANTERIOR.
```
```FIN DEL PROMPT PARA COPIAR (TS + Node)
````

## Prompt corto (TS + Node)

````markdown
```INICIO DEL PROMPT CORTO (TS + Node)
TS estricto (sin any), ESM, Result/uniones, Zod para inputs, AbortController+timeout, Error cause + logging, env validado, Vitest/ESLint/Prettier, TSDoc; un solo punto de salida.
```
```FIN DEL PROMPT CORTO (TS + Node)
````

## Referencias

- TypeScript Handbook: <https://www.typescriptlang.org/docs/>
- Node.js Docs: <https://nodejs.org/docs/latest/api/>
- Zod: <https://zod.dev/>
- Vitest: <https://vitest.dev/>

---

## 📘 Sobre esta guía

Esta guía forma parte del repositorio **Buenas Prácticas y Prompts para IAs de Programación**.

👉 **Ver más guías**: [Repositorio completo](../README.md)

---

**Autor**: [David Bueno Vallejo](https://davidbuenov.com/) | [LinkedIn](https://www.linkedin.com/in/davidbueno/) | [GitHub](https://github.com/davidbuenov)

**Licencia**: MIT

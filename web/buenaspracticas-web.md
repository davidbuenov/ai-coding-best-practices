# Guía de Buenas Prácticas Web 2025 (HTML, CSS, JS)

Prácticas modernas para construir interfaces accesibles, seguras y rápidas, pensadas para trabajar bien con generadores de código (Copilot, ChatGPT, Gemini, Claude).

## Tabla de contenidos

- [Alcance y requisitos](#alcance-y-requisitos)
- [HTML: semántica y accesibilidad](#html-semántica-y-accesibilidad)
- [CSS: arquitectura y responsive](#css-arquitectura-y-responsive)
- [JS: módulos, calidad y DX](#js-módulos-calidad-y-dx)
- [Rendimiento (Core Web Vitals)](#rendimiento-core-web-vitals)
- [Seguridad (CSP, XSS, CSRF)](#seguridad-csp-xss-csrf)
- [Testing y calidad](#testing-y-calidad)
- [Checklist rápida Web 2025](#checklist-rápida-web-2025)
- [Ejemplo mínimo accesible](#ejemplo-mínimo-accesible)
- [Starter estático (index/styles/js)](#starter-estático-indexstylesjs)
- [Prompt completo (Web)](#prompt-completo-web)
- [Prompt corto (Web)](#prompt-corto-web)
- [Referencias](#referencias)

## Alcance y requisitos

- HTML5, CSS3 y ES2022+ (módulos ESM). Navegadores evergreen.
- Node LTS opcional para tooling (Vite/Next/Astro) y testing (Vitest/Playwright).

## HTML: semántica y accesibilidad

- Usa etiquetas semánticas: header, nav, main, section, article, aside, footer.
- Landmarks claros y orden lógico de encabezados h1..h6.
- Formularios con label por cada control; usa fieldset/legend para agrupaciones.
- Texto alternativo significativo en imágenes (alt). Idioma en html lang.
- Evita div soup y ARIA innecesario; ARIA sólo cuando no hay equivalente nativo.
- Meta viewport, título descriptivo, meta description única por página.

```html
<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Accesible y Rápida</title>
    <meta name="description" content="Ejemplo accesible con HTML5 semántico" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  </head>
  <body>
    <header>
      <h1>Accesible y Rápida</h1>
      <nav aria-label="principal">
        <a href="#contenido">Saltar al contenido</a>
      </nav>
    </header>
    <main id="contenido">
      <article>
        <h2>Formulario</h2>
        <form>
          <label for="email">Correo</label>
          <input id="email" name="email" type="email" autocomplete="email" required />
          <button type="submit">Enviar</button>
        </form>
      </article>
    </main>
    <footer>© 2025</footer>
    <script type="module" src="./main.js" defer></script>
  </body>
</html>
```

## CSS: arquitectura y responsive

- Arquitectura: BEM, ITCSS o Utility-first; usa capas (CSS Cascade Layers) cuando aplique.
- Variables CSS (custom properties) para temas/espaciados/colores.
- Responsive: diseño fluido, clamp() para tipografía/espacios, container queries.
- Accesibilidad: contrastes AA/AAA, prefers-reduced-motion, focus visible.
- Propiedades lógicas (margin-inline, padding-block) para RTL.

```css
/* Variables de tema */
:root {
  --color-fg: #111;
  --color-bg: #fff;
  --space-2: 0.5rem;
  --space-4: 1rem;
}

/* Tipografía fluida */
html { font-size: clamp(16px, 1.6vw, 18px); }

/* Componentes: BEM */
.card { background: var(--color-bg); color: var(--color-fg); padding: var(--space-4); }
.card__title { font-weight: 700; margin-block-end: var(--space-2); }

/* Accesibilidad */
@media (prefers-reduced-motion: reduce) {
  * { animation: none !important; transition: none !important; }
}

:focus-visible { outline: 2px solid #0af; outline-offset: 2px; }
```

## JS: módulos, calidad y DX

- ESM nativo + defer; evita inlines y globales.
- Progresive Enhancement: funcionalidad JS mejora el HTML base, no lo rompe.
- Código robusto: async/await con AbortController, manejo de errores/timeout.
- Accesibilidad: sincroniza aria-expanded/hidden según estado de UI.
- JSDoc en funciones públicas; nombres claros; single responsibility.

```js
/**
 * Alterna un panel accesible.
 * @param {HTMLElement} button Botón que controla el panel.
 * @param {HTMLElement} panel Elemento del panel (role="region").
 */
export function togglePanel(button, panel) {
  const next = button.getAttribute('aria-expanded') !== 'true';
  button.setAttribute('aria-expanded', String(next));
  panel.hidden = !next;
}

// Progressive enhancement
document.addEventListener('click', (e) => {
  const btn = e.target.closest('[data-toggle]');
  if (!btn) return;
  const target = document.querySelector(btn.dataset.toggle);
  if (target) togglePanel(btn, target);
});
```

## Rendimiento (Core Web Vitals)

- Imágenes en AVIF/WebP; `loading="lazy"`; tamaños y `sizes` correctos.
- Fuentes: subset + `font-display: swap`; preconnect/preload medido.
- JS/CSS: code splitting, minificación, HTTP/2, caches y ETags.
- Medición continua: Lighthouse, WebPageTest, RUM.

## Seguridad (CSP, XSS, CSRF)

- CSP estricta (sin inline); usa nonces o hashes con SRI.
- Sanitiza entradas y contenido dinámico (DOMPurify para HTML no confiable).
- Cookies `HttpOnly`, `Secure`, `SameSite=Lax/Strict`; tokens CSRF.
- Siempre HTTPS; cabeceras: `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`.

## Testing y calidad

- Unit: Vitest/Jest. E2E: Playwright. a11y: axe-core/pa11y.
- Linting: ESLint + Stylelint. Formateo: Prettier. Type checking opcional: TypeScript/JSDoc.

## Checklist rápida Web 2025

- [ ] HTML semántico, headings y landmarks correctos
- [ ] Labels, alt text, focus visible, contrastes AA/AAA
- [ ] CSS con variables, responsive fluido, prefers-reduced-motion
- [ ] JS modular ESM, sin inline, progressive enhancement
- [ ] JSDoc en APIs públicas
- [ ] Imágenes lazy + formatos modernos; fuentes con swap
- [ ] CSP sin inline; sanitización; cookies seguras; CSRF
- [ ] Linting + tests (unit/E2E/a11y)

## Ejemplo mínimo accesible

```html
<button aria-expanded="false" aria-controls="panel" data-toggle="#panel">Más detalles</button>
<section id="panel" role="region" aria-live="polite" hidden>
  <h3>Detalles</h3>
  <p>Contenido adicional con lectura de pantalla anunciable.</p>
</section>
```

```js
import { togglePanel } from './toggle.js';

// Mejora progresiva por data-attribute (ver sección JS)
```

## Starter estático (index/styles/js)

Estructura lista para abrir en el navegador:

- `web/ejemplos/index.html`
- `web/ejemplos/styles.css`
- `web/ejemplos/main.js`
- `web/ejemplos/toggle.js`

Notas:

- HTML semántico con skip-link y landmarks.
- JS ESM con JSDoc y progressive enhancement (data-toggle).
- CSS con variables, focus-visible y reduced-motion.

## Prompt completo (Web)

````markdown
```INICIO DEL PROMPT PARA COPIAR (Web)
Eres un asistente experto en Web (HTML, CSS, JS). Genera código accesible (WCAG 2.2), seguro y rápido.

REQUISITOS OBLIGATORIOS
- HTML semántico con landmarks; formularios con label; alt text; lang en <html>.
- CSS moderno: variables, responsive fluido (clamp), prefers-reduced-motion, focus visible.
- JS ESM + defer; sin inline handlers; progressive enhancement; sincroniza aria-* con estado.
- Seguridad: evita inline scripts; asume CSP; sanitiza contenido dinámico; cookies seguras/CSRF.
- Rendimiento: imágenes AVIF/WebP con lazy; fuentes con swap; code splitting si aplica.
- Documentación: JSDoc en funciones públicas (brief, params, return, notas).

CONTRATO DE ENTREGA
- Estructura separada: index.html, styles.css, main.js (y módulos); sin globales.
- Accesibilidad validable (labels, headings, aria-* mínimo y correcto).
- Comentarios JSDoc en funciones JS expuestas; nombres claros.
- Evita dependencias innecesarias; puro estándar cuando sea posible.

A PARTIR DE AQUÍ, IMPLEMENTA EL PEDIDO DEL USUARIO RESPETANDO TODO LO ANTERIOR.
```
```FIN DEL PROMPT PARA COPIAR (Web)
````

## Prompt corto (Web)

````markdown
```INICIO DEL PROMPT CORTO (Web)
HTML semántico + a11y (labels/alt/focus), CSS moderno (variables, clamp, reduced-motion), JS ESM sin inline (progressive enhancement, aria sincronizado). Seguridad: CSP sin inline, sanitización, cookies seguras, CSRF. Rendimiento: imágenes modernas + lazy, fuentes swap. JSDoc en funciones. Entrega index.html/styles.css/main.js separados.
```
```FIN DEL PROMPT CORTO (Web)
````

## Referencias

- HTML Living Standard: <https://html.spec.whatwg.org/>
- WCAG 2.2: <https://www.w3.org/TR/WCAG22/>
- ARIA Authoring Practices: <https://www.w3.org/WAI/ARIA/apg/>
- MDN Web Docs (HTML/CSS/JS): <https://developer.mozilla.org/>
- CSP: <https://developer.mozilla.org/docs/Web/HTTP/CSP>

---

## 📘 Sobre esta guía

Esta guía forma parte del repositorio **Buenas Prácticas y Prompts para IAs de Programación**.

👉 **Ver más guías**: [Repositorio completo](../README.md)

---

**Autor**: [David Bueno Vallejo](https://davidbuenov.com/) | [LinkedIn](https://www.linkedin.com/in/davidbueno/) | [GitHub](https://github.com/davidbuenov)

**Licencia**: MIT

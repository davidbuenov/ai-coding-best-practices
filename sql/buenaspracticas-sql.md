# Guía de Buenas Prácticas SQL (MySQL/PostgreSQL/SQLite) 2025

Recomendaciones para escribir SQL correcto, seguro y performante en motores populares (sin extensiones propietarias).

## Tabla de contenidos

- [Alcance](#alcance)
- [Modelado y esquema](#modelado-y-esquema)
- [Estilo de consultas](#estilo-de-consultas)
- [Validación del resultado](#validación-del-resultado)
- [Parámetros y seguridad](#parámetros-y-seguridad)
- [Transacciones e isolamento](#transacciones-e-isolamento)
- [Índices y rendimiento](#índices-y-rendimiento)
- [Migraciones](#migraciones)
- [Checklist SQL general](#checklist-sql-general)
- [Ejemplos](#ejemplos)
- [Prompt completo (SQL general)](#prompt-completo-sql-general)
- [Prompt corto (SQL general)](#prompt-corto-sql-general)
- [Referencias](#referencias)

## Alcance

- SQL estándar con foco en PostgreSQL/MySQL/SQLite.
- Sin funciones propietarias; evita anti‑patrones como SELECT * en producción.

## Modelado y esquema

- Nombres consistentes (snake_case), claves primarias sintéticas (id BIGINT/UUID) y naturales cuando apliquen.
- Normaliza hasta 3FN salvo motivos claros; documenta denormalizaciones.
- Tipos correctos: usa timestamptz (Postgres) para mitigar zonas horarias; boolean en lugar de tinyint(1).

## Estilo de consultas

- SELECT con columnas explícitas; CTEs (WITH) para legibilidad; window functions para agregados avanzados.
- Paginación keyset (WHERE id < ?) mejor que OFFSET/LIMIT en grandes volúmenes.
- Evita OR múltiples; considera índices compuestos o UNION ALL.

## Validación del resultado

- Define qué representa una fila y qué población debe incluir el resultado antes de escribir la consulta.
- Comprueba la cardinalidad de los JOIN: varias filas de detalle pueden repetir un importe de la tabla principal.
- Si sólo necesitas comprobar que existe una relación, considera EXISTS. SUM(DISTINCT importe) no elimina entidades duplicadas: también colapsa importes iguales de entidades distintas.
- Prueba con datos sintéticos y un resultado calculado de antemano; incluye relaciones repetidas, importes iguales y entidades sin relación. Que la consulta se ejecute no demuestra que responda a la pregunta.

## Parámetros y seguridad

- SIEMPRE usa sentencias preparadas / placeholders.
- Postgres: $1, $2...; MySQL/SQLite: ?.
- Nunca concatenes entradas del usuario; sanitiza y valida antes de llegar a la DB.

## Transacciones e isolamento

- Agrupa operaciones relacionadas en una transacción; define nivel de aislamiento según necesidad.
- Usa SERIALIZABLE sólo si comprendes el coste; por defecto READ COMMITTED (PG) o REPEATABLE READ (MySQL).

## Índices y rendimiento

- Índices en FK y columnas de búsqueda/ordenación; evita sobredimensionar índices.
- Usa EXPLAIN/EXPLAIN ANALYZE para entender planes (Postgres) y EXPLAIN en MySQL/SQLite.
- Cuida la cardinalidad y selectividad; evita funciones sobre columnas indexadas en predicados.

## Migraciones

- Herramienta: Flyway/Liquibase/Knex/Prisma Migrate; versiona y revisa.
- Backfills con lotes pequeños; ventanas de mantenimiento; rollbacks controlados.

## Checklist SQL general

- [ ] Columnas explícitas (no SELECT *)
- [ ] Consultas parametrizadas
- [ ] ÍNDICES en FK y filtros frecuentes
- [ ] EXPLAIN antes de ir a prod
- [ ] Transacciones para consistencia
- [ ] Paginación keyset cuando aplique
- [ ] Migraciones versionadas
- [ ] Población y granularidad definidas; resultado contrastado con un ejemplo conocido

## Ejemplos

```sql
-- CTE para legibilidad
WITH activos AS (
  SELECT id, nombre FROM clientes WHERE activo = true
)
SELECT a.id, a.nombre, p.total
FROM activos a
JOIN pedidos p ON p.cliente_id = a.id
WHERE p.fecha >= CURRENT_DATE - INTERVAL '30 days'
ORDER BY p.fecha DESC
LIMIT 100;
```

```sql
-- Paginación keyset (Postgres)
SELECT id, nombre
FROM clientes
WHERE id < $1
ORDER BY id DESC
LIMIT $2;
```

```sql
-- Índice compuesto sugerido
CREATE INDEX IF NOT EXISTS idx_pedidos_cliente_fecha ON pedidos (cliente_id, fecha DESC);
```

### Sumar pedidos con algún intento de pago, sin duplicarlos

El objetivo de este ejemplo es sumar el importe de los pedidos que tienen al menos un intento de pago; **no** calcular ingresos cobrados. Los datos son sintéticos. Los pedidos 1 y 2 tienen el mismo importe y el pedido 3 no tiene intentos.

```sql
WITH pedidos AS (
  SELECT 1 AS id, 100 AS importe
  UNION ALL SELECT 2, 100
  UNION ALL SELECT 3, 50
), intentos AS (
  SELECT 1 AS id, 1 AS pedido_id
  UNION ALL SELECT 2, 1
  UNION ALL SELECT 3, 2
)
SELECT
  -- MAL: el pedido 1 aparece dos veces; devuelve 300.
  (SELECT SUM(p.importe)
   FROM pedidos p JOIN intentos i ON i.pedido_id = p.id) AS suma_join,
  -- MAL: los pedidos 1 y 2 comparten importe; devuelve 100.
  (SELECT SUM(DISTINCT p.importe)
   FROM pedidos p JOIN intentos i ON i.pedido_id = p.id) AS suma_distinct,
  -- BIEN: una contribución por pedido elegible; devuelve 200.
  (SELECT SUM(p.importe)
   FROM pedidos p
   WHERE EXISTS (
     SELECT 1 FROM intentos i WHERE i.pedido_id = p.id
   )) AS suma_por_pedido;
```

Añadir otro intento a un pedido ya incluido no debe cambiar `suma_por_pedido`. Si ningún pedido tiene intentos, SUM devuelve NULL: aplica COALESCE sólo si el contrato del informe define ese conjunto vacío como cero.

### Placeholders en Node.js

```js
// PostgreSQL (pg)
await client.query('SELECT * FROM users WHERE email = $1', [email]);

// MySQL2
await conn.execute('SELECT * FROM users WHERE email = ?', [email]);
```

## Prompt completo (SQL general)

````markdown
```INICIO DEL PROMPT PARA COPIAR (SQL general)
Eres un asistente experto en SQL (PostgreSQL/MySQL/SQLite). Genera consultas seguras y performantes.

REQUISITOS
- Columnas explícitas, no SELECT *.
- Consultas parametrizadas (placeholders: $1.. o ? según motor).
- Índices adecuados (FK y filtros frecuentes); sugiere índices compuestos cuando apliquen.
- Paginación keyset en grandes tablas.
- Usa CTEs para legibilidad; window functions cuando convengan.
- Define población y granularidad; evita duplicar entidades al agregar tras un JOIN.
- Incluye un caso sintético con resultado esperado y distingue ejecución correcta de resultado correcto.
- Explica brevemente decisiones de rendimiento y riesgos.

ENTREGA
- SQL listo para ejecutar y, si aplica, comando de creación de índice.
- Opcional: ejemplo de EXPLAIN/ANALYZE esperado.
```
```FIN DEL PROMPT PARA COPIAR (SQL general)
````

## Prompt corto (SQL general)

````markdown
```INICIO DEL PROMPT CORTO (SQL general)
SQL seguro y rápido: columnas explícitas, placeholders ($1 o ?), índices en FK/filtros, keyset pagination, CTEs/window functions; incluye índices sugeridos y nota de rendimiento. Define población y granularidad; valida el resultado con un caso sintético de total conocido.
```
```FIN DEL PROMPT CORTO (SQL general)
````

## Referencias

- PostgreSQL Docs: <https://www.postgresql.org/docs/>
- MySQL Docs: <https://dev.mysql.com/doc/>
- SQLite Docs: <https://sqlite.org/docs.html>
- Use The Index, Luke!: <https://use-the-index-luke.com/>

---

## 📘 Sobre esta guía

Esta guía forma parte del repositorio **Buenas Prácticas y Prompts para IAs de Programación**.

👉 **Ver más guías**: [Repositorio completo](../README.md)

---

**Autor**: [David Bueno Vallejo](https://davidbuenov.com/) | [LinkedIn](https://www.linkedin.com/in/davidbueno/) | [GitHub](https://github.com/davidbuenov)

**Licencia**: MIT

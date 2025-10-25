# Guía de Buenas Prácticas Oracle SQL y PL/SQL (2025)

Recomendaciones para escribir SQL y PL/SQL seguros, mantenibles y performantes en Oracle Database.

## Tabla de contenidos

- [Alcance](#alcance)
- [SQL en Oracle: claves](#sql-en-oracle-claves)
- [PL/SQL: paquetes y errores](#plsql-paquetes-y-errores)
- [Rendimiento y diagnóstico](#rendimiento-y-diagnóstico)
- [Checklist Oracle](#checklist-oracle)
- [Ejemplos](#ejemplos)
- [Prompt completo (Oracle SQL/PLSQL)](#prompt-completo-oracle-sqlplsql)
- [Prompt corto (Oracle SQL/PLSQL)](#prompt-corto-oracle-sqlplsql)
- [Referencias](#referencias)

## Alcance

- SQL portable en Oracle y PL/SQL idiomático (paquetes, excepciones, tipos).
- Evita dependencias innecesarias en features poco portables salvo justificar el beneficio.

## SQL en Oracle: claves

- BIND variables SIEMPRE (placeholders `:1`, `:name`), evita concatenación.
- Indexa FK y columnas de filtro/orden; considera índices bitmap con cuidado (lectura intensiva).
- Usa `FETCH FIRST N ROWS ONLY`/`OFFSET` (12c+), y keyset cuando aplique.
- Evita funciones sobre columnas indexadas en predicados; usa columnas derivadas persistidas si es necesario.
- Controla el uso de hints (`/*+ */`) y solo tras medir; documenta.

## PL/SQL: paquetes y errores

- Organiza en paquetes (SPEC/BODY); expón API clara, oculta detalles.
- Excepciones: define propias (`-20000..-20999`), usa `raise_application_error` con mensaje claro.
- Un solo punto de salida por función; variables locales para acumular resultado; `RETURN` al final.
- Bulk operations: `FORALL`, `BULK COLLECT` con límites; evita loops fila a fila.
- Tipos: `%ROWTYPE`, RECORDs, tablas indexadas para batch.
- Autonomía: `PRAGMA AUTONOMOUS_TRANSACTION` sólo si es imprescindible (ej. logging independiente).

## Rendimiento y diagnóstico

- Planes: `EXPLAIN PLAN` + `DBMS_XPLAN.DISPLAY`.
- Trazas: `SQL_TRACE`/`TKPROF`.
- AWR/ASH: usa con criterio (licenciamiento/impacto) para diagnóstico en producción.
- Estadísticas actualizadas; evita parámetros de sesión que rompan el optimizer.

## Checklist Oracle

- [ ] Bind variables (nunca concatenación)
- [ ] Índices en FK/filtros; medir antes de hints
- [ ] Paginación moderna o keyset
- [ ] Paquetes con API clara (SPEC/BODY)
- [ ] Excepciones propias y mensajes claros
- [ ] Un solo punto de salida por función
- [ ] BULK COLLECT/ FORALL en lotes
- [ ] Planes con DBMS_XPLAN; trazas con TKPROF

## Ejemplos

```sql
-- Paginación moderna (12c+)
SELECT id, nombre
FROM clientes
ORDER BY id
OFFSET :1 ROWS FETCH FIRST :2 ROWS ONLY;
```

```plsql
-- Paquete con un solo punto de salida
CREATE OR REPLACE PACKAGE pkg_clientes AS
  TYPE t_cliente IS RECORD(id clientes.id%TYPE, nombre clientes.nombre%TYPE);
  FUNCTION obtener(p_id IN clientes.id%TYPE) RETURN t_cliente;
END pkg_clientes;
/
CREATE OR REPLACE PACKAGE BODY pkg_clientes AS
  FUNCTION obtener(p_id IN clientes.id%TYPE) RETURN t_cliente IS
    v_out t_cliente;
  BEGIN
    SELECT id, nombre INTO v_out.id, v_out.nombre FROM clientes WHERE id = p_id;
    RETURN v_out; -- único return al final
  EXCEPTION
    WHEN NO_DATA_FOUND THEN
      raise_application_error(-20001, 'Cliente no encontrado');
  END obtener;
END pkg_clientes;
/
```

```sql
-- Índice recomendado
CREATE INDEX idx_pedidos_cliente_fecha ON pedidos(cliente_id, fecha DESC);
```

## Prompt completo (Oracle SQL/PLSQL)

````markdown
```INICIO DEL PROMPT PARA COPIAR (Oracle SQL/PLSQL)
Eres un asistente experto en Oracle. Genera SQL/PLSQL seguro, medible y mantenible.

REQUISITOS
- SQL con bind variables (:1, :name), sin concatenación de entradas.
- Índices en FK/filtros; sugiere índices compuestos cuando apliquen; evita funciones en predicados indexados.
- Paginación (FETCH FIRST / OFFSET) o keyset cuando convenga.
- PL/SQL en paquetes (SPEC/BODY), excepciones propias (-20000..-20999), raise_application_error.
- Un solo punto de salida por función; bulk (FORALL/BULK COLLECT) en lotes.
- Entrega plan con DBMS_XPLAN cuando pidas optimización.

ENTREGA
- SQL/PLSQL listo para ejecutar, más `CREATE INDEX` si aplica.
- Explica brevemente decisiones y trade-offs.
```
```FIN DEL PROMPT PARA COPIAR (Oracle SQL/PLSQL)
````

## Prompt corto (Oracle SQL/PLSQL)

````markdown
```INICIO DEL PROMPT CORTO (Oracle SQL/PLSQL)
Oracle: binds, índices (FK/filtros), paginación moderna o keyset, paquetes SPEC/BODY, excepciones -20000..-20999, raise_application_error, un solo return, FORALL/BULK COLLECT; incluye DBMS_XPLAN si optimización.
```
```FIN DEL PROMPT CORTO (Oracle SQL/PLSQL)
````

## Referencias

- Oracle Docs (SQL/PLSQL): <https://docs.oracle.com/en/database/>
- DBMS_XPLAN: <https://docs.oracle.com/en/database/oracle/oracle-database/>
- TKPROF y tracing: <https://docs.oracle.com/en/database/>

---

## 📘 Sobre esta guía

Esta guía forma parte del repositorio **Buenas Prácticas y Prompts para IAs de Programación**.

👉 **Ver más guías**: [Repositorio completo](../../README.md)

---

**Autor**: [David Bueno Vallejo](https://davidbuenov.com/) | [LinkedIn](https://www.linkedin.com/in/davidbueno/) | [GitHub](https://github.com/davidbuenov)

**Licencia**: MIT

# Guía de Buenas Prácticas C# para Unity 6.1

Patrones modernos y rendimiento para Unity 6.1 con C#, pensados para colaboración con IAs generativas.

## Tabla de contenidos

- [Alcance y versiones](#alcance-y-versiones)
- [Convenciones Unity](#convenciones-unity)
- [MonoBehaviour y ciclo de vida](#monobehaviour-y-ciclo-de-vida)
- [ScriptableObjects para datos](#scriptableobjects-para-datos)
- [Jobs, Burst y colecciones nativas](#jobs-burst-y-colecciones-nativas)
- [ECS (Entities)](#ecs-entities)
- [Serialización y configuración](#serialización-y-configuración)
- [Logging y debugging](#logging-y-debugging)
- [Checklist Unity 6.1](#checklist-unity-61)
- [Ejemplo mínimo](#ejemplo-mínimo)
- [Prompt completo (Unity 6.1 C#)](#prompt-completo-unity-61-c)
- [Prompt corto (Unity 6.1 C#)](#prompt-corto-unity-61-c)
- [Referencias](#referencias)

## Alcance y versiones

- Unity 6.1, C# con soporte moderno (records limitado; nullable reference types opcional).
- Jobs/Burst/ECS para rendimiento; MonoBehaviour para lógica de gameplay tradicional.

## Convenciones Unity

- Serializa campos privados con `[SerializeField]`; evita públicos innecesarios.
- Nombres claros: PascalCase para métodos/clases, camelCase para campos privados.
- Un solo punto de salida por método (acumula resultado y retorna al final).
- XML docs en componentes/sistemas reutilizables.

## MonoBehaviour y ciclo de vida

- `Awake` para inicialización interna; `Start` para setup con otros objetos.
- Evita `Update` pesado; usa eventos, coroutines o Jobs cuando convenga.
- Usa `GetComponent` con cache; evita llamadas repetidas en Update.

```csharp
using UnityEngine;

/// <summary>
/// Controla la salud de un jugador.
/// </summary>
public class HealthController : MonoBehaviour
{
    [SerializeField] private float maxHealth = 100f;
    private float currentHealth;

    private void Awake()
    {
        currentHealth = maxHealth;
    }

    /// <summary>
    /// Aplica daño al jugador.
    /// </summary>
    /// <param name="amount">Cantidad de daño.</param>
    public void TakeDamage(float amount)
    {
        float newHealth = currentHealth;
        if (amount > 0f)
        {
            newHealth = Mathf.Max(0f, currentHealth - amount);
        }
        currentHealth = newHealth; // un solo return implícito; assignment pattern
        if (currentHealth <= 0f)
        {
            Debug.Log("Player defeated");
        }
    }
}
```

## ScriptableObjects para datos

- Usa `ScriptableObject` para configuraciones, stats, eventos compartidos.
- Ventaja: editable en editor, referenciable, sin duplicados en prefabs.

```csharp
using UnityEngine;

[CreateAssetMenu(fileName = "EnemyData", menuName = "Game/EnemyData")]
public class EnemyData : ScriptableObject
{
    public float speed = 3f;
    public float health = 50f;
}
```

## Jobs, Burst y colecciones nativas

- Para operaciones paralelas masivas (física, IA, pathfinding), usa Jobs + Burst.
- Colecciones nativas: `NativeArray`, `NativeList`; dispone manualmente o con `Allocator.TempJob`.
- `[BurstCompile]` en jobs para compilación nativa; evita managed references dentro.

```csharp
using Unity.Burst;
using Unity.Collections;
using Unity.Jobs;
using UnityEngine;

[BurstCompile]
public struct VelocityJob : IJobParallelFor
{
    public NativeArray<Vector3> positions;
    public NativeArray<Vector3> velocities;
    public float deltaTime;

    public void Execute(int index)
    {
        positions[index] += velocities[index] * deltaTime;
    }
}
```

## ECS (Entities)

- Para rendimiento extremo y arquitectura data-oriented, considera Entities (DOTS).
- Componentes: struct con `IComponentData`; sistemas: `SystemBase` o `ISystem`.
- Evita mezclar GameObject/MonoBehaviour y ECS salvo en híbrido controlado.

```csharp
using Unity.Entities;
using Unity.Mathematics;

public struct Velocity : IComponentData
{
    public float3 Value;
}

public partial class MoveSystem : SystemBase
{
    protected override void OnUpdate()
    {
        float dt = SystemAPI.Time.DeltaTime;
        Entities.ForEach((ref LocalTransform transform, in Velocity vel) =>
        {
            transform.Position += vel.Value * dt;
        }).ScheduleParallel();
    }
}
```

## Serialización y configuración

- Unity serializa tipos básicos, arrays, List; para custom, usa `[System.Serializable]`.
- Para configuración avanzada, ScriptableObject o JSON externo.

## Logging y debugging

- `Debug.Log`/`LogWarning`/`LogError`; evita en loops intensos (usa condicionales o herramientas de profiling).
- Gizmos/DrawRay para debug visual en Scene.

## Checklist Unity 6.1

- [ ] `[SerializeField]` para privados; evita públicos innecesarios
- [ ] Cache `GetComponent`; evita Update pesado
- [ ] ScriptableObjects para configuraciones/stats
- [ ] Jobs + Burst para operaciones masivas
- [ ] ECS (Entities) para rendimiento extremo
- [ ] Un solo punto de salida por método
- [ ] XML docs en componentes reutilizables
- [ ] Debug.Log controlado (no en loops críticos)

## Ejemplo mínimo

```csharp
using UnityEngine;

/// <summary>
/// Mueve un objeto con velocidad constante.
/// </summary>
public class Mover : MonoBehaviour
{
    [SerializeField] private float speed = 5f;

    private void Update()
    {
        float deltaZ = speed * Time.deltaTime;
        Vector3 newPos = transform.position;
        newPos.z += deltaZ;
        transform.position = newPos; // un solo assignment
    }
}
```

## Prompt completo (Unity 6.1 C#)

````markdown
```INICIO DEL PROMPT PARA COPIAR (Unity 6.1 C#)
Eres un asistente experto en Unity 6.1 C#. Genera código limpio, performante y mantenible.

REQUISITOS
- `[SerializeField]` para campos privados; evita públicos innecesarios.
- Cache `GetComponent`; evita Update pesado (usa eventos/coroutines/Jobs).
- ScriptableObjects para configuraciones/stats compartidos.
- Jobs + Burst para operaciones masivas (física, IA); NativeArray con dispose.
- ECS (Entities) para rendimiento extremo cuando convenga.
- Un solo punto de salida por método.
- XML docs en componentes/sistemas reutilizables.

ENTREGA
- Código completo con using mínimos; MonoBehaviour/ScriptableObject/Job/System según aplique.
- Explica decisiones de rendimiento brevemente.
```
```FIN DEL PROMPT PARA COPIAR (Unity 6.1 C#)
````

## Prompt corto (Unity 6.1 C#)

````markdown
```INICIO DEL PROMPT CORTO (Unity 6.1 C#)
Unity 6.1: [SerializeField], cache GetComponent, ScriptableObjects, Jobs+Burst, ECS cuando convenga, un solo return, XML docs; evita Update pesado y Debug.Log en loops críticos.
```
```FIN DEL PROMPT CORTO (Unity 6.1 C#)
````

## Referencias

- Unity Docs: <https://docs.unity3d.com/>
- Jobs System: <https://docs.unity3d.com/Manual/JobSystem.html>
- Burst: <https://docs.unity3d.com/Packages/com.unity.burst@latest/>
- ECS (Entities): <https://docs.unity3d.com/Packages/com.unity.entities@latest/>

---

## 📘 Sobre esta guía

Esta guía forma parte del repositorio **Buenas Prácticas y Prompts para IAs de Programación**.

👉 **Ver más guías**: [Repositorio completo](../../README.md)

---

**Autor**: [David Bueno Vallejo](https://davidbuenov.com/) | [LinkedIn](https://www.linkedin.com/in/davidbueno/) | [GitHub](https://github.com/davidbuenov)

**Licencia**: MIT

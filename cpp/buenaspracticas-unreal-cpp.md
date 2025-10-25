# Guía de Buenas Prácticas C++ para Unreal Engine 5.6

Esta guía resume patrones y recomendaciones para escribir C++ idiomático en Unreal Engine 5.6, con foco en seguridad, rendimiento, integraciones con Blueprints y redes, y mantenibilidad.

## Tabla de contenidos

- [Objetivos y alcance](#objetivos-y-alcance)
- [Requisitos](#requisitos)
- [Convenciones de Unreal](#convenciones-de-unreal)
- [Ciclo de vida y memoria](#ciclo-de-vida-y-memoria)
- [Propiedades, funciones y metadatos](#propiedades-funciones-y-metadatos)
- [Componentes y diseño](#componentes-y-diseño)
- [Colecciones y tipos fundamentales](#colecciones-y-tipos-fundamentales)
- [Logging, aserciones y errores](#logging-aserciones-y-errores)
- [Async, timers y tareas](#async-timers-y-tareas)
- [Red y replicación](#red-y-replicación)
- [Módulos y Build.cs](#módulos-y-buildcs)
- [Checklist rápida UE 5.6](#checklist-rápida-ue-56)
- [Ejemplo mínimo (UE 5.6)](#ejemplo-mínimo-ue-56)
- [Prompt completo (UE 5.6 C++)](#prompt-completo-ue-56-c)
- [Prompt corto (UE 5.6 C++)](#prompt-corto-ue-56-c)
- [Referencias](#referencias)

## Objetivos y alcance

- C++ idiomático de Unreal 5.6: uso de macros UCLASS/UPROPERTY/UFUNCTION, GC, sistemas de componentes y subsistemas.
- Integración con Blueprints y redes con replicación básica.
- Reglas de estilo, patrones de seguridad y rendimiento.

## Requisitos

- Unreal Engine 5.6 (Editor y toolchain correspondientes)
- Visual Studio 2022 (o equivalente en su plataforma)
- Conocimiento básico de Gameplay Framework (AActor, UActorComponent, APlayerController, GameMode)

## Convenciones de Unreal

- Prefijos de tipos: U (UObject), A (Actor), F (struct/clase plain), I (interface), E (enum), T (plantillas/containers de UE: TArray, TMap, etc.).
- Estilo de nombres: PascalCase para clases y métodos, camelCase para variables. Categorías y nombres de propiedades legibles en el editor.
- Evita new/delete manual. Usa:
        - `NewObject<T>(Outer)` para UObjects creados en runtime.
        - `CreateDefaultSubobject<T>(TEXT("Name"))` dentro del constructor de actores/componentes para subobjetos.
        - `MakeShared`/`MakeUnique` para tipos no-UObject cuando proceda.
- Un solo punto de salida por función (sin returns intermedios).

## Ciclo de vida y memoria

- UObjects están manejados por el Garbage Collector. Cualquier referencia a UObjects que deba mantenerse viva debe ser UPROPERTY.
- Para referencias débiles a UObjects, usa `TWeakObjectPtr<T>` o `TSoftObjectPtr<T>` (para assets que cargan de forma diferida).
- Para tipos no-UObject, usa TSharedPtr/TUniquePtr según ownership. Evita raw new/delete.
- En actores, crea componentes con CreateDefaultSubobject en el constructor y adjúntalos en el árbol de componentes.

Ejemplo:

```cpp
UCLASS()
class AHealthPickup : public AActor
{
    GENERATED_BODY()

public:
    AHealthPickup();

protected:
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category="Components")
    UStaticMeshComponent* Mesh = nullptr;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Pickup")
    float HealAmount = 50.f;
};

AHealthPickup::AHealthPickup()
{
    Mesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Mesh"));
    RootComponent = Mesh;
}
```

## Propiedades, funciones y metadatos

- UPROPERTY: define visibilidad en editor/Blueprint y participación en GC/replicación.
  - EditAnywhere, VisibleAnywhere, BlueprintReadWrite/ReadOnly, Replicated, ReplicatedUsing=OnRep_X.
- UFUNCTION: expone a Blueprints, RPCs y eventos.
  - BlueprintCallable, BlueprintPure, Server/Reliable, Client, NetMulticast.
- Usa meta = (AllowPrivateAccess = "true") cuando quieras mantener variables privadas visibles al editor.

Ejemplo con OnRep y RPC:

```cpp
UCLASS()
class ANetworkedDoor : public AActor
{
    GENERATED_BODY()

public:
    ANetworkedDoor();

    UFUNCTION(BlueprintCallable, Category="Door")
    void Toggle();

protected:
    UPROPERTY(ReplicatedUsing=OnRep_IsOpen, EditAnywhere, BlueprintReadOnly, Category="Door")
    bool bIsOpen = false;

    UFUNCTION()
    void OnRep_IsOpen();

    UFUNCTION(Server, Reliable)
    void Server_SetOpen(bool bOpen);

    virtual void GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const override;
};

void ANetworkedDoor::Toggle()
{
    const bool bServer = HasAuthority();
    const bool bNewOpen = !bIsOpen;

    if (bServer)
    {
        bIsOpen = bNewOpen;
        OnRep_IsOpen();
    }
    else
    {
        Server_SetOpen(bNewOpen);
    }
}

void ANetworkedDoor::OnRep_IsOpen()
{
    // Actualiza visuales/animación
}

void ANetworkedDoor::Server_SetOpen_Implementation(bool bOpen)
{
    bIsOpen = bOpen;
    OnRep_IsOpen();
}

void ANetworkedDoor::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
    super::GetLifetimeReplicatedProps(OutLifetimeProps);
    DOREPLIFETIME(ANetworkedDoor, bIsOpen);
}
```

## Componentes y diseño

- Prefiere composición con UActorComponent/USceneComponent a herencias profundas.
- Mantén componentes coesos (una responsabilidad). Evita componentes "Dios".
- Evita Tick pesado. Usa timers, eventos, OnRep o notificaciones.
- Subsystems: usa UGameInstanceSubsystem/UWorldSubsystem para servicios globales.

## Colecciones y tipos fundamentales

- Usa TArray, TSet, TMap. Para búsquedas rápidas, TMap/TSet; para orden, TArray con UE::Algo::Sort.
- Cadenas: FString (mutable), FName (identificadores hash), FText (texto localizado).
- Views: TConstArrayView para parámetros de sólo lectura.
- Math: FVector, FRotator, FTransform; evita crear temporales innecesarios.

Ejemplo con UE::Algo:

```cpp
#include "Algo/Sort.h"

void SortScores(TArray<int32>& Scores)
{
    UE::Algo::Sort(Scores);
}
```

## Logging, aserciones y errores

- Define categorías de log propias.
- Usa UE_LOG para trazas; check/checkf para invariantes; ensure para condiciones recuperables.

```cpp
DEFINE_LOG_CATEGORY_STATIC(LogInventory, Log, All);

void UInventoryComponent::AddItem(const FItem& Item)
{
    checkf(Item.Id.IsValid(), TEXT("Item Id inválido"));
    Items.Add(Item);
    UE_LOG(LogInventory, Verbose, TEXT("Añadido %s"), *Item.Id.ToString());
}
```

## Async, timers y tareas

- Timers: GetWorld()->GetTimerManager().SetTimer(...)
- Tasks a GameThread: AsyncTask(ENamedThreads::GameThread, []{ ... });
- Carga asíncrona de assets: TSoftObjectPtr + StreamableManager.

```cpp
void ASpawner::BeginPlay()
{
    super::BeginPlay();
    GetWorldTimerManager().SetTimer(SpawnTimer, this, &ASpawner::SpawnOne, 1.0f, true);
}
```

## Red y replicación

- Marca propiedades con UPROPERTY(Replicated/ReplicatedUsing) y regístralas en GetLifetimeReplicatedProps.
- RPCs: Server (desde cliente al server), Client (del server a un cliente), NetMulticast (a todos).
- Usa Reliable sólo para eventos críticos; evita abuso.

## Módulos y Build.cs

- Declara dependencias en .Build.cs: Core, CoreUObject, Engine, InputCore, etc.
- Usa Private/PublicDependencyModuleNames según el ámbito.

```csharp
PublicDependencyModuleNames.AddRange(new[] { "Core", "CoreUObject", "Engine", "InputCore" });
PrivateDependencyModuleNames.AddRange(new[] { "UMG", "Slate", "SlateCore" });
```

## Checklist rápida UE 5.6

- [ ] Sin new/delete manual; usa NewObject/CreateDefaultSubobject/MakeShared.
- [ ] UPROPERTY para todas las referencias a UObjects que deban vivir.
- [ ] Un solo punto de salida por función; sin returns intermedios.
- [ ] Sin Ticks pesados; usa timers/eventos/OnRep.
- [ ] Logs con categorías; check/ensure donde corresponda.
- [ ] RPCs y replicación correctamente declarados y registrados.
- [ ] Componentes cohesionados; evita clases gigantes.
- [ ] Build.cs con dependencias mínimas necesarias.

### Comentarios y documentación

- Usa comentarios en formato Doxygen para clases y métodos clave (brief, detalles, notas, parámetros).
- Explica por qué una propiedad es UPROPERTY (GC/replicación/exposición a editor/Blueprints).
- Documenta las razones de decisiones de replicación (Reliable, OnRep, RPCs) y de rendimiento (evitar Tick, timers, etc.).

Por qué importa:

- Contrato claro entre C++ y Blueprints/red: reduce errores de uso.
- Onboarding y mantenimiento: rápida comprensión sin inspeccionar implementación.
- Sinergia con IAs: el contexto explícito guía mejores sugerencias.

Plantilla mínima (UCLASS/UFUNCTION/UPROPERTY):

```cpp
/// @brief Actor/Componente que <responsabilidad principal>.
/// @details Colabora con <X>. No hace <Y>. Expone <Z> a Blueprints.
UCLASS()
class YOURMODULE_API AMiActor : public AActor {
    GENERATED_BODY()
public:
    /// @brief Alterna el estado.
    /// @details Si se llama en cliente, invoca RPC al servidor.
    /// @note Un solo punto de salida. No usa Tick.
    UFUNCTION(BlueprintCallable, Category="Demo")
    void Toggle();

protected:
    /// @brief Estado replicado; usado por OnRep para actualizar visuales.
    /// @note UPROPERTY para GC y replicación.
    UPROPERTY(ReplicatedUsing=OnRep_State, EditAnywhere, BlueprintReadOnly, Category="Demo")
    bool bState = false;

    /// @brief Notificación de replicación para aplicar cambios visuales.
    UFUNCTION()
    void OnRep_State();
};
```

Tags útiles:

- @brief, @details: resumen y contexto.
- @param, @return: contrato de entradas/salidas.
- @note, @warning: uso correcto y riesgos.
- @see: referencias a símbolos relacionados.

## Ejemplo mínimo (UE 5.6)

- Actor con componente y replicación:
  - `cpp/ejemplos-unreal/MyReplicatedActor.h`
  - `cpp/ejemplos-unreal/MyReplicatedActor.cpp`
- Componente de salud replicado:
  - `cpp/ejemplos-unreal/HealthComponent.h`
  - `cpp/ejemplos-unreal/HealthComponent.cpp`
- Character replicado con highlight visual:
  - `cpp/ejemplos-unreal/ReplicatedCharacter.h`
  - `cpp/ejemplos-unreal/ReplicatedCharacter.cpp`

Notas:

- Reemplaza `YOURMODULE_API` por el nombre de tu módulo (por ejemplo, `MYPROJECT_API`).
- Asegura dependencias mínimas en tu `.Build.cs`:

```csharp
PublicDependencyModuleNames.AddRange(new[] { "Core", "CoreUObject", "Engine" });
```

- Habilita replicación en instancias que lo requieran y configura pruebas en PIE con múltiples jugadores si corresponde.

## Prompt completo (UE 5.6 C++)

````markdown
```INICIO DEL PROMPT PARA COPIAR (UE 5.6 C++)
Eres un asistente experto en C++ para Unreal Engine 5.6. Genera código que compile y siga estas reglas:

REQUISITOS DE UNREAL 5.6
- Usa UCLASS/UPROPERTY/UFUNCTION correctamente.
- NO uses new/delete manual para UObjects. Usa NewObject<T>() en runtime y CreateDefaultSubobject<T>() en constructores de actores/componentes. Para tipos no-UObject usa MakeUnique/MakeShared cuando proceda.
- Cualquier referencia a UObject que deba mantenerse viva debe ser UPROPERTY. Usa TWeakObjectPtr/TSoftObjectPtr cuando corresponda.
- Prefiere componentes (UActorComponent/USceneComponent) sobre herencia profunda. Evita Ticks pesados; usa timers, eventos o OnRep.
- Integra con Blueprints cuando tenga sentido: BlueprintReadWrite/ReadOnly, BlueprintCallable, BlueprintPure.
- Para red: marca propiedades Replicated/ReplicatedUsing y regístralas en GetLifetimeReplicatedProps; usa RPCs Server/Client/NetMulticast con Reliable sólo si es imprescindible.

ESTILO Y SEGURIDAD
- Un solo punto de salida por función (sin returns intermedios). Añade logs al entrar/salir de operaciones críticas.
- Usa categorías de log (DEFINE_LOG_CATEGORY_STATIC) y UE_LOG. Usa check/checkf para invariantes y ensure para condiciones recuperables.
- Usa tipos de UE: FString/FName/FText, TArray/TMap/TSet, UE::Algo.
- Evita pasar colecciones copiando; usa referencias const o TConstArrayView según aplique.

CONTRATO DE ENTREGA
- Proporciona: encabezados y .cpp completos y consistentes si procede; incluye includes mínimos necesarios.
- Incluye la declaración de categoría de log si usas logs.
- Si hay replicación, incluye GetLifetimeReplicatedProps y ejemplos de OnRep y RPCs.
- Constructor de actores: usa CreateDefaultSubobject y configura RootComponent si aplica.
- Documenta en comentarios breves por qué se aplican ciertas decisiones (GC, replicación, timers, etc.).

A PARTIR DE AQUÍ, IMPLEMENTA EL PEDIDO DEL USUARIO RESPETANDO TODO LO ANTERIOR.
```
```FIN DEL PROMPT PARA COPIAR (UE 5.6 C++)
````

## Prompt corto (UE 5.6 C++)

````markdown
```INICIO DEL PROMPT CORTO (UE 5.6 C++)
C++ Unreal 5.6 guidelines: UCLASS/UPROPERTY/UFUNCTION correctos; sin new/delete para UObjects (usa NewObject/CreateDefaultSubobject); referencias a UObject siempre UPROPERTY; componentes sobre herencia; evita Tick pesado (timers/eventos/OnRep); Blueprints expuestos cuando convenga; replicación con Replicated/ReplicatedUsing + GetLifetimeReplicatedProps; RPCs Server/Client/NetMulticast (Reliable sólo si crítico); un solo punto de salida por función (sin returns intermedios); logs con categorías + check/ensure; tipos UE (FString/FName/FText, TArray/TMap/TSet, UE::Algo). Entrega headers + .cpp completos, constructor con CreateDefaultSubobject, logs y replicación si aplica.
```
```FIN DEL PROMPT CORTO (UE 5.6 C++)
````

## Referencias

- Unreal Engine 5.6 Documentation: <https://docs.unrealengine.com/>
- Coding Standard (Epic style): <https://docs.unrealengine.com/5.3/en-US/epic-cpp-coding-standard-for-unreal-engine/>
- UE Containers: <https://docs.unrealengine.com/5.3/en-US/unreal-engine-containers/>
- Networking: <https://docs.unrealengine.com/5.3/en-US/networking-in-unreal-engine/>

---

## 📘 Sobre esta guía

Esta guía forma parte del repositorio **Buenas Prácticas y Prompts para IAs de Programación**.

👉 **Ver más guías**: [Repositorio completo](../../README.md)

---

**Autor**: [David Bueno Vallejo](https://davidbuenov.com/) | [LinkedIn](https://www.linkedin.com/in/davidbueno/) | [GitHub](https://github.com/davidbuenov)

**Licencia**: MIT

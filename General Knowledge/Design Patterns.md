[Home](../README.md) > [General Knowledge](README.md) > [Design Patterns](Design%20Patterns.md)
# Design Patterns
Unreal Engine way of implementing design patterns and explains their usage.

[Structural Patterns](#structural-patterns)  
&nbsp;&nbsp;▸ [Proxy Pattern](#proxy-pattern) : Control access to expensive resources.  
&nbsp;&nbsp;▸ [Composite Pattern](#composite-pattern) : Tree-like structures with uniform usage.  
&nbsp;&nbsp;▸ [Decorator Pattern](#decorator-pattern) : Extend features with conditions and functionalities.   
&nbsp;&nbsp;▸ [Facade Pattern](#facade-pattern) : UBlueprintFunctionLibrary or system to simplify usage of multiple systems.  
&nbsp;&nbsp;▸ [Flyweight Pattern](#flyweight-pattern) : Share common data to save memory.  
&nbsp;&nbsp;▸ [Bridge & Adapter Pattern](#bridge--adapter-pattern) : Provide a unified interface for different implementation.

[Behavioral Patterns](#behavioral-patterns)  
&nbsp;&nbsp;▸ [Command Pattern](#command-pattern) : Encapsulate actions as objects for undo/redo.  
&nbsp;&nbsp;▸ [Chain of Responsibility Pattern](#chain-of-responsibility-pattern) : Process object through handlers.  
&nbsp;&nbsp;▸ [Strategy Pattern](#strategy-pattern) : Select algorithms at runtime.  
&nbsp;&nbsp;▸ [Memento Pattern](#memento-pattern-state-snapshot) : Serialize and deserialize stateful objects.  
&nbsp;&nbsp;▸ [State Pattern](#state-pattern) : Manage state transitions and encapsulate states in separate classes.  
&nbsp;&nbsp;▸ [Mediator Pattern](#mediator-pattern) : Centralize communication between systems.  
&nbsp;&nbsp;▸ [Observer Pattern](#observer-pattern) : Event delegate for state changes.   
&nbsp;&nbsp;▸ [Iterator Pattern](#iterator-pattern) : Simplify collection iteration.  
&nbsp;&nbsp;▸ [Visitor Pattern](#visitor-pattern) : Run distinct algorithms for base class types.

[Creational Patterns](#creational-patterns)  
&nbsp;&nbsp;▸ [Singleton Pattern](#singleton-pattern) : Use subsystem to ensure one instance.  
&nbsp;&nbsp;▸ [Builder Pattern](#builder-pattern) : Use hierarchical DataTables for modular object creation.  
&nbsp;&nbsp;▸ [Factory & Prototype Pattern](#factory--prototype-pattern) : Create instance from DataTable row.  

Reference for standard design patterns : https://refactoring.guru/design-patterns/catalog

## Structural Patterns
### Proxy Pattern
Used to control access to another object and add functionality before or after executing the request.
1. Lazy Initialization: Delays object creation until it’s actually needed, helping to optimize memory usage by creating objects only when required.
2. Access & API Control: Adds validation, security checks, or restrictions before accessing the object. It can also limit or customize the exposed API based on the context, ensuring only valid interactions with the object.
3. Performance Optimization : Caches results or breaks down expensive operations into smaller tasks to improve performance, reducing the time and resources needed for expensive operations.
4. Handle success or failure of requests in a separate layer.
```cpp
// For world actors that rarely use an ability, this proxy dynamically spawn the AbilitySystemComponent when requested.
UCLASS(Transient)
class UAbilitySystemComponentProxy : public UObject
{
public:
    UAbilitySystemComponentProxy(AActor* InOwner) : Owner(InOwner), AbilitySystemComponent(nullptr) {}

    void TriggerAbility() override
    {
        if (TryCreateAbilitySystemComponent())
        {
            AbilitySystemComponent->TryActivateAbilityByClass(YourAbilityClass);
        }
    }

private:
    bool TryCreateAbilitySystemComponent()
    {
        if (!AbilitySystemComponent)
        {
            AbilitySystemComponent = NewObject<UAbilitySystemComponent>(Owner);
            AbilitySystemComponent->RegisterComponent();
            AbilitySystemComponent->AttachToComponent(Owner->GetRootComponent(), FAttachmentTransformRules::SnapToTargetNotIncludingScale);
        }
        return AbilitySystemComponent != nullptr;
    }

    AActor* Owner;
    UAbilitySystemComponent* AbilitySystemComponent;
};
```

### Composite Pattern
Used for tree-like structures where individual objects and groups of objects are treated uniformly. Unreal's AI BehaviorTree use composite for selector and sequence nodes and is combined with Decorator Pattern to control the flow of execution.
```cpp
class INode
{
public:
    virtual ~INode() = default;
    virtual void DoSomething() const = 0;
};

class FLeaf : public INode
{
public:
    FLeaf(const FString& InName) : Name(InName) {}

    virtual void DoSomething() const override
    {
    }

private:
    FString Name;
};

class FComposite : public INode
{
public:
    FComposite(const FString& InName) : Name(InName) {}

    void AddNode(TSharedPtr<INode> Node)
    {
        Nodes.Add(Node);
    }

    virtual void PrintName() const override
    {
    }

private:
    FString Name;
    TArray<TSharedPtr<INode>> Nodes;
};
```

### Decorator Pattern
Allows behavior to be added to an object dynamically, providing a flexible alternative to subclassing for extending functionality.
Unreal's AI BehaviorTree Decorator is used for controlling the flow of execution with condition check based on blackboard data.
```cpp
class IAIBehavior
{
public:
    virtual ~IAIBehavior() = default;
    virtual void Begin(AICharacter* AICharacter) = 0;
    virtual void End(AICharacter* AICharacter) = 0;
    virtual void SetActive(AICharacter* AICharacter, bool bIsActive) = 0;
};

class FPatrolBehavior : public IAIBehavior
{
public:
    virtual void Begin(AICharacter* AICharacter) override {}
    virtual void End(AICharacter* AICharacter) override {}
    virtual void SetActive(AICharacter* AICharacter, bool bIsActive) override {}
};

class FStealthDecorator : public IAIBehavior
{
public:
    FStealthDecorator(TSharedPtr<IAIBehavior> InBaseBehavior) : BaseBehavior(InBaseBehavior) {}

    virtual void Begin(AICharacter* AICharacter) override
    {
        AICharacter->SetStealthMode(true);
        BaseBehavior->Begin(AICharacter);
    }
    virtual void End(AICharacter* AICharacter) override
    {
        BaseBehavior->End(AICharacter);
        AICharacter->SetStealthMode(false);
    }

private:
    TSharedPtr<IAIBehavior> BaseBehavior;
};

TUniquePtr<IAIBehavior> Behavior = MakeUnique<FPatrolBehavior>();
if (bEnableStealth)
{
    Behavior = MakeUnique<FStealthDecorator>(MoveTemp(Behavior));
}
Behavior->Begin(AICharacter);
```

### Facade Pattern
Using a UBlueprintFunctionLibrary can be a great way to simplify interaction with multiple systems.
```cpp
UCLASS()
class UGameAudioFacade : public UBlueprintFunctionLibrary
{
    GENERATED_BODY()

public:
    UFUNCTION(BlueprintCallable, Category = "Game Audio", meta = (WorldContext = "WorldContextObject"))
    static void PlaySoundEffect(UObject* WorldContextObject, USoundBase* Sound)
    {
        if (Sound && WorldContextObject)
        {
            UGameplayStatics::PlaySound2D(WorldContextObject, Sound);
        }
    }

    UFUNCTION(BlueprintCallable, Category = "Game Audio", meta = (WorldContext = "WorldContextObject"))
    static void SetMasterVolume(UObject* WorldContextObject, float Volume)
    {
        UWorld* World = WorldContextObject ? WorldContextObject->GetWorld() : nullptr;
        UGameInstance* GameInstance = World ? World->GetGameInstance() : nullptr;
        USoundSubsystem* SoundSubsystem = GameInstance ? GameInstance->GetSubsystem<USoundSubsystem>() : nullptr;

        if (SoundSubsystem)
        {
            SoundSubsystem->SetMasterVolume(Volume);
        }
    }
};
```

### Flyweight Pattern
Separate state into immutable and mutable data to save of memory. Shared immutable data will be reused across multiple object while mutable data is per instance.
```cpp
USTRUCT()
struct FBulletSharedData // Flyweight
{
    GENERATED_BODY()

    UStaticMesh* Mesh;
    UMaterialInterface* Material;
};

UCLASS()
class ABullet : public AActor
{
    GENERATED_BODY()

    const FBulletSharedData* SharedData;
    FVector BulletVelocity;
};
```

### Bridge & Adapter Pattern
Create a unified interface for different implementation. View [interfaces](Native%20Programming.md#interfaces).
- Adapter Pattern : Used to adapt incompatible interfaces, such as 3rd-party APIs.
- Bridge Pattern : Used to separate implementations into multiple classes.

## Behavioral Patterns

### Command Pattern
Used for encapsulating requested actions, allowing undo/redo functionalities. Useful for editor tools.

In Unreal Engine, calling SetFlags(RF_Transactional) when starting object modification, then FScopedTransaction with Object->Modify() at every modification is a great way to bind to the editor's undo system.
```cpp
class IEditorCommand
{
public:
    virtual ~IEditorCommand() = default;
    virtual void Execute() = 0;
    virtual void Undo() = 0;
};

class FMoveActorCommand : public IEditorCommand
{
public:
    FMoveActorCommand(AActor* InActor, const FVector& InDeltaLocation) : TargetActor(InActor), DeltaLocation(InDeltaLocation) {}

    virtual void Execute() override
    {
        if (TargetActor)
        {
            TargetActor->AddActorWorldOffset(DeltaLocation);
        }
    }

    virtual void Undo() override
    {
        if (TargetActor)
        {
            TargetActor->AddActorWorldOffset(-DeltaLocation);
        }
    }

private:
    AActor* TargetActor;
    FVector DeltaLocation;
};

class FEditorCommandManager
{
public:
    static constexpr int32 MaxCommandHistory = 50;

    FEditorCommandManager() : CommandHistory(MaxCommandHistory) {}

    void ExecuteCommand(TUniquePtr<IEditorCommand> Command)
    {
        if (Command)
        {
            Command->Execute();
            CommandHistory.Add(MoveTemp(Command));
        }
        else
        {
            UE_LOG(LogCommandManager, Error, TEXT("Failed to execute command"));
        }
    }

    void UndoLastCommand()
    {
        if (CommandHistory.IsEmpty())
        {
            UE_LOG(LogCommandManager, Warning, TEXT("No commands to undo"));
            return;
        }

        TUniquePtr<IEditorCommand>& LastCommand = CommandHistory.Last();
        if (LastCommand.IsValid())
        {
            LastCommand->Undo();
            CommandHistory.RemoveLast();
        }
        else
        {
            UE_LOG(LogCommandManager, Error, TEXT("Invalid command in history"));
        }
    }

private:
    TCircularBuffer<TUniquePtr<IEditorCommand>> CommandHistory;
};

TUniquePtr<IEditorCommand> MoveCommand = MakeUnique<FMoveActorCommand>(Actor, DeltaLocation);
CommandManager.ExecuteCommand(MoveTemp(MoveCommand));
CommandManager.UndoLastCommand();
```

### Chain of Responsibility Pattern
Passes a request through a chain of handlers for processing or forwarding.
```cpp
class FMeshProcessor
{
protected:
    TUniquePtr<FMeshProcessor> NextProcessor;

public:
    FMeshProcessor(TUniquePtr<FMeshProcessor> Next = nullptr) : NextProcessor(MoveTemp(Next)) { }

    virtual void ProcessMesh(UStaticMesh* ModifiedMesh)
    {
        if (NextProcessor)
        {
            NextProcessor->ProcessMesh(ModifiedMesh);
        }
    }

    virtual ~FMeshProcessor() = default;
};

class FMeshProcess_RemoveDuplicateVertices : public FMeshProcessor
{
    ...
};

class FMeshProcess_ComputeNormals : public FMeshProcessor
{
    ...
};

TUniquePtr<FMeshProcessor> Step2_RemoveDuplicateVertices = MakeUnique<FMeshProcess_RemoveDuplicateVertices>();
TUniquePtr<FMeshProcessor> Step1_ComputeNormals = MakeUnique<FMeshProcess_ComputeNormals>(MoveTemp(Step2_RemoveDuplicateVertices));
Step1_ComputeNormals->ProcessMesh(Mesh);
```


### Strategy Pattern
Encapsulates algorithms (behaviors) and allows them to be interchangeable at runtime. Can be used in combination with Command where it stores the request to execute the desired strategy.
```cpp
class IVertexCleanupStrategy
{
public:
    virtual ~IVertexCleanupStrategy() = default;

    virtual void Cleanup(UMeshDescription& MeshDescription) = 0;
};

class FMergeVerticesByPositionStrategy : public IVertexCleanupStrategy
{
public:
    virtual void Cleanup(UMeshDescription& MeshDescription) override {}
};

class FMergeVerticesByDistanceStrategy : public IVertexCleanupStrategy
{
public:
    FMergeVerticesByDistanceStrategy(float InMergeDistance) : MergeDistance(InMergeDistance) {}
    virtual void Cleanup(UMeshDescription& MeshDescription) override {}
private:
    float MergeDistance;
};
```


### Memento Pattern (State Snapshot)
Capture and externalize an object's internal state without violating encapsulation, so that the object can be restored to this state later. Particularly useful for state history and rollback.
```cpp
USTRUCT()
struct FFrameState // Memento
{
    // NetSerialize is handled by USTRUCT()
};

USTRUCT(Transient)
struct FActorState : public FFrameState // Concrete Memento
{
    FVector Location;
    FRotator Rotation;
    float Health;

    DECLARE_DELEGATE(FOnStateChanged); // Single cpp listener for originator
    FOnStateChanged OnStateRestored;

    bool NetSerialize(FArchive& Ar, UPackageMap* Map)
    {
        Ar << Location;
        Ar << Rotation;
        Ar << Health;

        if (Ar.IsLoading())
        {
            OnStateRestored.ExecuteIfBound();
        }

        return true;
    }
};


class AMyActor : public AActor // Originator
{
public:
    void BeginPlay()
    {
        State = NewObject<UActorState>(this);
        State->OnStateRestored.BindUObject(this, &AMyActor::OnStateRestored);

        if (UOneStateRollbackSystem* RollbackSystem = GetWorld()->GetWorldSubsystem<UOneStateRollbackSystem>())
        {
            RollbackSystem->RegisterState(State);
        }
    }

    void OnStateRestored()
    {
        SetActorLocation(State->Location);
        SetActorRotation(State->Rotation);
    }

private:
    UPROPERTY(Transient) // Hard ref, this actor owns the state.
    UActorState* State;
};

UCLASS()
class UOneStateRollbackSystem : public UWorldSubsystem // Caretaker
{
public:
    static constexpr int32 MaxStateHistory = 50;

    void SaveState(TWeakObjectPtr<FFrameState> FrameState)
    {
        FrameToSerialize = FrameState;

        if (FrameToSerialize.IsValid())
        {
	        FBitWriter Archive;
	        FrameToSerialize->NetSerialize(Archive, GetWorld());
            StateHistory.Add(Archive.GetData());
        }
    }

    void RestoreState(AActor* Actor)
    {
        if (StateHistory.IsEmpty())
        {
            UE_LOG(LogRollback, Warning, TEXT("No state to restore"));
            return;
        }

        if (FrameToSerialize.IsValid())
        {
            FBitReader ReadArchive(StateHistory.Last());
            FrameToSerialize->NetSerialize(ReadArchive, GetWorld());
            StateHistory.RemoveLast();
        }
    }

private:
    // Weak ref, will become invalid when the actor is destroyed.
    TWeakObjectPtr<FFrameState> FrameToSerialize;
    TCircularBuffer<uint8*> StateHistory;
};
```

### State Pattern
Used to manage state transitions and simplify complex logic by encapsulating states in separate classes
```cpp
class FGameplayAbility
{
public:
    FGameplayAbility(UAbilitySystemComponent* InAbilitySystemComponent) : AbilitySystemComponent(InAbilitySystemComponent)
    {
        // Or instantiate when transitioning to the state.
        StartState = MakeShared<FStartState>(this);
        ActiveState = MakeShared<FActiveState>(this);
        EndState = MakeShared<FEndState>(this);
        CurrentState = StartState;
    }

    void ChangeState(TSharedPtr<IAbilityState> NewState)
    {
        CurrentState = NewState;
        CurrentState->Handle();
    }

    void ApplyEffect(FGameplayTag EffectGameplayTag) { ... }

private:
    UAbilitySystemComponent* AbilitySystemComponent;
    TSharedPtr<IAbilityState> CurrentState;

    TSharedPtr<FStartState> StartState;
    TSharedPtr<FActiveState> ActiveState;
    TSharedPtr<FEndState> EndState;
};

class IAbilityState
{
public:
    virtual void Handle() = 0;
    virtual void TransitionToNextState() = 0;
};

class FStartState : public IAbilityState
{
public:
    FStartState(FGameplayAbility* InAbility) : Ability(InAbility) {}
    void Handle() override { ApplyEffect(UMyGameplayTags::GE_HealthBoost); TransitionToNextState(); }
    void TransitionToNextState() override { Ability->ChangeState(Ability->ActiveState); }

private:
    FGameplayAbility* Ability;
};

class FActiveState : public IAbilityState
{
public:
    FActiveState(FGameplayAbility* InAbility) : Ability(InAbility) {}
    void Handle() override { ApplyEffect(UMyGameplayTags::GE_DamageOverTime); TransitionToNextState(); }
    void TransitionToNextState() override { Ability->ChangeState(Ability->EndState); }

private:
    FGameplayAbility* Ability;
};

class FEndState : public IAbilityState
{
public:
    FEndState(FGameplayAbility* InAbility) : Ability(InAbility) {}
    void Handle() override { ApplyEffect(UMyGameplayTags::GE_EndAbility); }
    void TransitionToNextState() override { }

private:
    FGameplayAbility* Ability;
};
```

### Mediator Pattern
Use event delegates to separate two systems. You can use AGameState, or attach a component on it, to have one system trigger a game event and an another system listen on it. That way, one system listens on the other without ever knowing it exists.

Alternatively, AHUD can be used for communication between multiple UWidget, APlayerState can be used for player-specific communication or you can use a custom mediator subsystem for specific events.

### Observer Pattern
Use event driven logic with [event delegates](Native%20Programming.md#events):
```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnHealthChanged, int, NewHealth);

UPROPERTY(BlueprintAssignable)
FOnHealthChanged OnHealthChanged;

void SetHealth(int NewHealth)
{
    Health = NewHealth;
    OnHealthChanged.Broadcast(Health);
}
```

### Iterator Pattern
Provide a way to access elements of a collection sequentially without exposing its underlying representation. Can simplify iteration for arrays with implicit order.
```cpp
class FTriangleIterator
{
public:
    FTriangleIterator(const TArrayView<int32>& InIndices) : IndicesArrayView(InIndices), CurrentIndex(0) {}

    bool HasNext() const
    {
        return CurrentIndex + 2 < IndicesArrayView.Num();
    }

    void Next(TArray<int32>& OutTriangle)
    {
        if (HasNext())
        {
            OutTriangle.Add(IndicesArrayView[CurrentIndex]);
            OutTriangle.Add(IndicesArrayView[CurrentIndex + 1]);
            OutTriangle.Add(IndicesArrayView[CurrentIndex + 2]);
            CurrentIndex += 3;
        }
        else
        {
            UE_LOG(LogTemp, Warning, TEXT("No more triangles to iterate."));
        }
    }

private:
    const TArrayView<int32>& IndicesArrayView;
    int32 CurrentIndex;
};
```

### Visitor pattern
Apply a behavior with different algorithms depending on the class, while keeping the algorithms separate from the classes to avoid making them overly complex. Instead of casting to each separate class, then calling different functions, we can create two interfaces as such:
```cpp
class IExportNode {
public:
    virtual void export(class IExportVisitor& visitor) = 0;
};

// Concrete classes
class City : public IExportNode {
public:
    void Export(IExportVisitor& visitor) override  { visitor.ExportCity(*this); }
};

class Industry : public IExportNode {
public:
    void Export(IExportVisitor& visitor) override { visitor.ExportIndustry(*this); }
};

// Abstract Visitor class
class IExportVisitor {
public:
    virtual void ExportCity(City& city) = 0;
    virtual void ExportIndustry(Industry& industry) = 0;
};

class ExportVisitor : public IExportVisitor {
public:
    void ExportAll(const TArray<IExportNode*>& Nodes)
    {
        for (IExportNode* Node : Nodes)
        {
            // Instead of casting to City and Industry, it visits their class and calls the correct export function below.
            Node->Export(*this);
        }
    }

    void ExportCity(City& city) override {
    }

    void ExportIndustry(Industry& industry) override {
    }
};
```

## Creational Patterns
### Singleton Pattern
Prefer using subsystems : https://dev.epicgames.com/documentation/en-us/unreal-engine/programming-subsystems-in-unreal-engine.
| **Subsystem**            | **Lifetime**           | **Usage** |
|--------------------------|------------------------|-----------|
| **GameInstanceSubsystem** | Entire game session    | Stores game-wide data, services, or managers that persist across levels (e.g., matchmaking, analytics). |
| **EngineSubsystem**       | Entire engine session  | Global services that exist beyond a single game instance, such as debug tools or global asset management. |
| **WorldSubsystem**        | Per UWorld instance    | Manages level-specific systems like weather, AI managers, or procedural generation. Resets on level load. |
| **LocalPlayerSubsystem**  | Per local player       | Handles player-specific systems like UI, settings, or input bindings. |

### Builder Pattern
Constructs complex objects step by step, allowing customization without exposing the object’s internal details.

Using DataTables is a great way to achieve designer-friendly builder pattern. Each entity (Enemy, Ability, Weapon, etc) is defined in a distinct DataTable using an FGameplayTag or FName as a unique key. This enables fast lookups and hierarchical relationships without hardcoded dependencies. Enemy can reference multiple ability, behaviors, decorators and weapons to be fully modular.

### Factory & Prototype Pattern
Using a datatable is a great approach by using a FGameplayTag to instantiate a specific class (factory) and to clone data (prototype). It is designer friendly and is easy to switch blueprint class for an enemy type to iterate on design. Use native FGameplayTag to access enemy class both in cpp and blueprint.
```cpp
USTRUCT(BlueprintType)
struct FCardData
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FName CardName;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    int32 ManaCost;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    TSubclassOf<UCardEffect> EffectClass;
};

USTRUCT(BlueprintType)
struct FCardPrototypeRow : public FTableRowBase
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    TSubclassOf<UCardInstanceBase> CardInstanceClass;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FCardData CardData;
};

UCLASS(Blueprintable)
class UCardInstanceBase : public UObject
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FCardData CardData;

    void InitializeCard(const FCardData& Data)
    {
        CardData = Data; // Deep copy.
    }
};

UCLASS()
class UCardManager : public UObject
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    UDataTable* CardPrototypeTable;

    UCardInstanceBase* CreateCardInstance(FName CardID)
    {
        if (!CardPrototypeTable) return nullptr;

        const FCardPrototypeRow* PrototypeRow = CardPrototypeTable->FindRow<FCardPrototypeRow>(CardID, TEXT("CreateCardInstance"));
        if (PrototypeRow)
        {
            UCardInstanceBase* NewCardInstance = NewObject<UCardInstanceBase>(this, PrototypeRow->CardInstanceClass->StaticClass());

            NewCardInstance->InitializeCard(PrototypeRow->CardData);

            return NewCardInstance;
        }

        return nullptr;
    }
};
```

© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). 
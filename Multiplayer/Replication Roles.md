[Home](../README.md) > [Multiplayer](README.md) > [Replication Roles](Replication%20Roles.md)
# Replication Roles
Using Unreal Engine 5's Lyra sample game, these are all of the most common network role values.

**ROLE_Authority** : The instance that has full control over the actor (usually the server). It can modify and replicate state.  
**ROLE_AutonomousProxy** : A client-owned actor that receives network updates but also has local prediction authority (e.g., the player's own character).  
**ROLE_SimulatedProxy** : A client-side actor that only receives updates from the server and does not have local control (e.g., other players in a multiplayer game).  
**ROLE_None** : The actor has no assigned network role, meaning it is not replicated over the network.

## Actor Functions
### Dedicated Server

|                     | Role           | RemoteRole           | NetMode         | NetOwner         | IsNetStartupActor |
|---------------------|----------------|----------------------|-----------------|------------------|-------------------|
| GameMode            | ROLE_Authority | ROLE_None            | DedicatedServer | None             | False             |
| PlayerController    | ROLE_Authority | ROLE_AutonomousProxy | DedicatedServer | PlayerController | False             |
| AIController        | ROLE_Authority | ROLE_None            | DedicatedServer | None             | False             |
| Player Pawn         | ROLE_Authority | ROLE_AutonomousProxy | DedicatedServer | Pawn             | False             |
| AI Pawn             | ROLE_Authority | ROLE_SimulatedProxy  | DedicatedServer | Pawn             | False             |
| Weapon              | ROLE_Authority | ROLE_SimulatedProxy  | DedicatedServer | Pawn             | False             |
| PlayerCameraManager | ROLE_Authority | ROLE_None            | DedicatedServer | PlayerController | False             |
 
### Listen Server
|                                      | Role           | RemoteRole           | NetMode      | NetOwner         | IsNetStartupActor |
|--------------------------------------|----------------|----------------------|--------------|------------------|-------------------|
| GameMode                             | ROLE_Authority | ROLE_None            | ListenServer | None             | False             |
| Server’s Controlled PlayerController | ROLE_Authority | ROLE_SimulatedProxy  | ListenServer | PlayerController | False             |
| PlayerController                     | ROLE_Authority | ROLE_AutonomousProxy | ListenServer | PlayerController | False             |
| AIController                         | ROLE_Authority | ROLE_None            | ListenServer | None             | False             |
| Server’s Controlled Player Pawn      | ROLE_Authority | ROLE_AutonomousProxy | ListenServer | Pawn             | False             |
| Player Pawn                          | ROLE_Authority | ROLE_AutonomousProxy | ListenServer | Pawn             | False             |
| AI Pawn                              | ROLE_Authority | ROLE_SimulatedProxy  | ListenServer | Pawn             | False             |
| Weapon                               | ROLE_Authority | ROLE_SimulatedProxy  | ListenServer | Pawn             | False             |
| PlayerCameraManager                  | ROLE_Authority | ROLE_None            | ListenServer | PlayerController | False             |

### Client
|                                    | Role                 | RemoteRole     | NetMode | NetOwner         | IsNetStartupActor |
|------------------------------------|----------------------|----------------|---------|------------------|-------------------|
| Controlled PlayerController        | ROLE_AutonomousProxy | ROLE_Authority | Client  | PlayerController | False             |
| Controlled Player Pawn             | ROLE_AutonomousProxy | ROLE_Authority | Client  | Pawn             | False             |
| AI and uncontrolled pawn           | ROLE_SimulatedProxy  | ROLE_Authority | Client  | Pawn             | False             |
| Spectator Pawn                     | ROLE_Authority       | ROLE_None      | Client  | Spectator Pawn   | False             |
| Controlled and uncontrolled Weapon | ROLE_SimulatedProxy  | ROLE_Authority | Client  | Pawn             | False             |
| PlayerCameraManager                | ROLE_Authority       | ROLE_None      | Client  | PlayerController | False             |
| Client-side Cosmetics              | ROLE_Authority       | ROLE_None      | Client  | None             | False             |

### Standalone
|                                    | Role           | RemoteRole          | NetMode    | NetOwner         | IsNetStartupActor |
|------------------------------------|----------------|---------------------|------------|------------------|-------------------|
| GameMode                           | ROLE_Authority | ROLE_None           | Standalone | None             | False             |
| Controlled PlayerController        | ROLE_Authority | ROLE_SimulatedProxy | Standalone | PlayerController | False             |
| AIController                       | ROLE_Authority | ROLE_None           | Standalone | None             | False             |
| Controlled Player Pawn             | ROLE_Authority | ROLE_SimulatedProxy | Standalone | Pawn             | False             |
| AI Pawn                            | ROLE_Authority | ROLE_SimulatedProxy | Standalone | Pawn             | False             |
| Controlled and uncontrolled Weapon | ROLE_Authority | ROLE_SimulatedProxy | Standalone | Pawn             | False             |
| PlayerCameraManager                | ROLE_Authority | ROLE_None           | Standalone | PlayerController | False             |
| Client-side Cosmetics              | ROLE_Authority | ROLE_None           | Standalone | None             | False             |

## Pawn Functions
### Dedicated Server
|             | IsPawnControlled | IsLocallyControlled | IsBotControlled | IsPlayerControlled | IsLocallyViewed | IsLocalPlayerControllerViewingAPawn |
|-------------|------------------|---------------------|-----------------|--------------------|-----------------|-------------------------------------|
| Player Pawn | true             | false               | false           | true               | false           | false                               |
| AI Pawn     | true             | true                | true            | false              | false           | false                               |

### Listen Server
|                                 | IsPawnControlled | IsLocallyControlled | IsBotControlled | IsPlayerControlled | IsLocallyViewed | IsLocalPlayerControllerViewingAPawn |
|---------------------------------|------------------|---------------------|-----------------|--------------------|-----------------|-------------------------------------|
| Server’s Controlled Player Pawn | true             | true                | false           | true               | true            | true                                |
| Player Pawn                     | true             | false               | false           | true               | false           | true                                |
| AI Pawn                         | true             | true                | true            | false              | false           | true                                |

### Client
|                        | IsPawnControlled | IsLocallyControlled | IsBotControlled | IsPlayerControlled | IsLocallyViewed | IsLocalPlayerControllerViewingAPawn |
|------------------------|------------------|---------------------|-----------------|--------------------|-----------------|-------------------------------------|
| Controlled Player Pawn | true             | true                | false           | true               | true            | true                                |
| Uncontrolled Pawn      | false            | false               | false           | true               | false           | true                                |
| AI Pawn                | false            | false               | true            | false              | false           | true                                |
| Spectator Pawn         | true             | true                | false           | false              | true            | true                                |

### Standalone
|                        | IsPawnControlled | IsLocallyControlled | IsBotControlled | IsPlayerControlled | IsLocallyViewed | IsLocalPlayerControllerViewingAPawn |
|------------------------|------------------|---------------------|-----------------|--------------------|-----------------|-------------------------------------|
| Controlled Player Pawn | true             | true                | false           | true               | true            | true                                |
| AI Pawn                | true             | true                | true            | false              | false           | true                                |

## Controller Functions
### Dedicated Server
|                  | IsLocalController | IsLocalPlayerController |
|------------------|-------------------|-------------------------|
| PlayerController | false             | false                   |
| AIController     | true              | false                   |

### Listen Server
|                                      | IsLocalController | IsLocalPlayerController |
|--------------------------------------|-------------------|-------------------------|
| Server’s Controlled PlayerController | true              | true                    |
| PlayerController                     | false             | false                   |
| AIController                         | true              | false                   |

### Client
|                        | IsLocalController | IsLocalPlayerController |
|------------------------|-------------------|-------------------------|
| Controlled Player Pawn | true              | true                    |

### Standalone
|                  | IsLocalController | IsLocalPlayerController |
|------------------|-------------------|-------------------------|
| PlayerController | true              | true                    |
| AIController     | true              | false                   |


## References
These values were acquired with the following code : 
```
FString NetModeToString(const ENetMode InMode)
{
	switch(InMode)
	{
	case ENetMode::NM_Client:
		return FString("Client");
	case ENetMode::NM_DedicatedServer:
		return FString("DedicatedServer");
	case ENetMode::NM_ListenServer:
		return FString("ListenServer");
	case ENetMode::NM_Standalone:
		return FString("Standalone");
	default:
		break;
	}
	return FString("INVALID NETMODE");
}
void ALyraGameState::PrintAllNetworkingRoles(const UObject* WorldContext)
{
	TArray<AActor*> AllActors;
	UGameplayStatics::GetAllActorsOfClass(WorldContext, AActor::StaticClass(), AllActors);
	for (AActor* Actor : AllActors)
	{
		const USceneComponent* RootComponent = Actor ? Actor->GetRootComponent() : nullptr;
		const bool IsMovable = RootComponent && RootComponent->Mobility == EComponentMobility::Movable;
		if (IsMovable || Cast<APawn>(Actor) || Cast<AController>(Actor))
		{
			const FString Role = StaticEnum<ENetRole>()->GetValueAsString(Actor->GetLocalRole());
			const FString RemoteRole = StaticEnum<ENetRole>()->GetValueAsString(Actor->GetRemoteRole());
			const FString NetMode = NetModeToString(Actor->GetNetMode());
			const AActor* NetOwner = Actor->GetNetOwner();
			const FString NetOwnerName = NetOwner ? NetOwner->GetActorNameOrLabel() : "No Net Owner";
			const FString IsNetStartupActor = UKismetStringLibrary::Conv_BoolToString(Actor->IsNetStartupActor());
			const FString ActorInfo = FString::Printf(TEXT("Role=%s | RemoteRole=%s | NetMode=%s | NetOwnerName=%s | IsNetStartupActor=%s"),
															*Role, *RemoteRole, *NetMode, *NetOwnerName, *IsNetStartupActor);
			const APawn* ActorAsPawn = Cast<APawn>(Actor);
			const FString IsPawnControlled = ActorAsPawn ? UKismetStringLibrary::Conv_BoolToString(ActorAsPawn->IsPawnControlled()) : "Not a Pawn";
			const FString IsLocallyControlled = ActorAsPawn ? UKismetStringLibrary::Conv_BoolToString(ActorAsPawn->IsLocallyControlled()) : "Not a Pawn";
			const FString IsBotControlled = ActorAsPawn ? UKismetStringLibrary::Conv_BoolToString(ActorAsPawn->IsBotControlled()) : "Not a Pawn";
			const FString IsPlayerControlled = ActorAsPawn ? UKismetStringLibrary::Conv_BoolToString(ActorAsPawn->IsPlayerControlled()) : "Not a Pawn";
			const FString IsLocallyViewed = ActorAsPawn ? UKismetStringLibrary::Conv_BoolToString(ActorAsPawn->IsLocallyViewed()) : "Not a Pawn";
			const FString IsLocalPlayerControllerViewingAPawn = ActorAsPawn ? UKismetStringLibrary::Conv_BoolToString(ActorAsPawn->IsLocalPlayerControllerViewingAPawn()) : "Not a Pawn";
			const FString PawnInfo = FString::Printf(TEXT("IsPawnControlled=%s | IsLocallyControlled=%s | IsBotControlled=%s | IsPlayerControlled=%s | IsLocallyViewed=%s | IsLocalPlayerControllerViewingAPawn=%s"),
															*IsPawnControlled, *IsLocallyControlled, *IsBotControlled, *IsPlayerControlled, *IsLocallyViewed, *IsLocalPlayerControllerViewingAPawn);
			const AController* Controller = Cast<AController>(Actor);
			const FString IsLocalController = Controller ? UKismetStringLibrary::Conv_BoolToString(Controller->IsLocalController()) : "No Controller";
			const FString IsLocalPlayerController = Controller ? UKismetStringLibrary::Conv_BoolToString(Controller->IsLocalPlayerController()) : "No Controller";
			const FString ControllerInfo = FString::Printf(TEXT("IsLocalController=%s | IsLocalPlayerController=%s"),
															*IsLocalController, *IsLocalPlayerController);
			const FString Name = Actor->GetActorNameOrLabel();
			const FString Message = FString::Printf(TEXT("%s : Actor=[%s], Pawn=[%s], Controller=[%s]"),
															*Name, *ActorInfo, *PawnInfo, *ControllerInfo);
			UKismetSystemLibrary::PrintString(WorldContext, Message);
		}
	}
}
```

© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). 
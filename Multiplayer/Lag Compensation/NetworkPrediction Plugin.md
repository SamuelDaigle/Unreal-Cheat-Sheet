# NetworkPrediction Plugin

## Setting up data
### Input Command
Client's input state and AI state
```cpp
struct FMovementInputCmd
{
	FVector_NetQuantize10 MovementInput;
	FRotator RotationInput;

	FMovementInputCmd() : MovementInput(ForceInitToZero), RotationInput(ForceInitToZero) {}

	void NetSerialize(const FNetSerializeParams& P)
	{
		P.Ar << MovementInput;
        RotationInput.SerializeCompressed(P.Ar);
	}
};
```

### SyncState
State we are evolving frame to frame and keeping in sync.
```cpp
struct FMovementSyncState
{
	FVector_NetQuantize10 Location;
	FVector_NetQuantize10 Velocity;
	FRotator Rotation;

	FMovementSyncState() : Location(ForceInitToZero), Velocity(ForceInitToZero), Rotation(ForceInitToZero) {}

	bool ShouldReconcile(const FMovementSyncState& AuthorityState) const
    {
        UE_NP_TRACE_RECONCILE(!AuthorityState.Location.Equals(Location, YourCVars::ErrorTolerance), "Loc:");
        return false;
    }

	void NetSerialize(const FNetSerializeParams& P)
	{
		P.Ar << Location;
		P.Ar << Velocity;
		Rotation.SerializeCompressed(P.Ar);
	}

	void Interpolate(const FMovementSyncState* From, const FMovementSyncState* To, float PCT)
	{
		static constexpr float TeleportThreshold = 1000.f * 1000.f;
		if (FVector::DistSquared(From->Location, To->Location) > TeleportThreshold)
		{
			*this = *To;
		}
		else
		{
			Location = FMath::Lerp(From->Location, To->Location, PCT);
			Velocity = FMath::Lerp(From->Velocity, To->Velocity, PCT);
			Rotation = FMath::Lerp(From->Rotation, To->Rotation, PCT);
		}
	}
};
```

### Auxiliary state
Inputs of the simulation that rarely or never change.
```cpp
struct FMovementAuxState
{	
	float MaxSpeed = 1200.f;
	float TurningBoost = 8.f;
	float Deceleration = 8000.f;
	float Acceleration = 4000.f;

	bool ShouldReconcile(const FMovementAuxState& AuthorityState) const
    {
        // If values do change, trigger a reconcile here.
        return false;
    }

	void NetSerialize(const FNetSerializeParams& P)
	{
		P.Ar << MaxSpeed;
		P.Ar << TurningBoost;
		P.Ar << Deceleration;
		P.Ar << Acceleration;
	}

	void Interpolate(const FMovementAuxState* From, const FMovementAuxState* To, float PCT)
	{
		// Since values change very rarely here, snap to target instead of lerping.
		MaxSpeed = To->MaxSpeed;
		TurningBoost = To->TurningBoost;
		Deceleration = To->Deceleration;
		Acceleration = To->Acceleration;
	}
};
```
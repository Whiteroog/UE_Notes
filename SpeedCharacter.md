# Скорость персонажа

Скорость персонажа регулируется переопределением метода GetMaxSpeed():

```c++
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Character movement: sprint", meta = (ClampMin = 0.0f, UIMin = 0.0f))
	float SprintSpeed = 1200.0f;

	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Character movement: fatigue", meta = (ClampMin = 0.0f, UIMin = 0.0f))
	float FatigueSpeed = 100.0f;

float UFPBaseCharacterMovementComponent::GetMaxSpeed() const
{
	float Result = Super::GetMaxSpeed();

	if(bIsSprinting)
	{
		Result = SprintSpeed;
	}

	if(bIsOutOfStamina)
	{
		Result = FatigueSpeed;
	}

	return Result;
}
```

А сама скорость регулируется флагами:

```c++
	bool bIsSprinting = false;
	bool bIsOutOfStamina = false;

void UFPBaseCharacterMovementComponent::StartSprint()
{
	bIsSprinting = true;
	bForceMaxAccel = 1;
}

void UFPBaseCharacterMovementComponent::StopSprint()
{
	bIsSprinting = false;
	bForceMaxAccel = 0;
}

void UFPBaseCharacterMovementComponent::SetIsOutOfStamina(bool bIsOutOfStamina_In)
{
	bIsOutOfStamina = bIsOutOfStamina_In;

	if(bIsOutOfStamina)
	{
		StopSprint();
	}
}
```

Этими методами регулируются в классе объекта
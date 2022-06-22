# BasePawnMovementComponent

Основная логика передвижения игрока находится в методе Tick:

```c++
void UFPBasePawnMovementComponent::TickComponent(float DeltaTime, ELevelTick TickType,
	FActorComponentTickFunction* ThisTickFunction)
{
	if (ShouldSkipUpdate(DeltaTime)) // оптимизация. Проверка на смещение персонажа
	{
		return;
	}

	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);

	FVector PendingInput = GetPendingInputVector().GetClampedToMaxSize(1.0f); // вводимый вектор перемещения. Если одно из координат больше 1, то ей ставят границу
	Velocity = PendingInput * MaxSpeed; // вектор скорости
	ConsumeInputVector(); // читска вводимого вектора перемещения 

	if (bIsEnableGravity) // мой флаг на гравитацию
	{
		FHitResult HitResult; // вектор точки столкновения
		FVector StartPoint = UpdatedComponent->GetComponentLocation(); точка, середина объекта
		float TraceDepth = 1.0f; // длина луча
		float SphereRadius = 50.0f; // погрешность высоты середины данного объекта
		FVector EndPoint = StartPoint - TraceDepth * FVector::UpVector; // точка, чуть ниже объекта
		FCollisionQueryParams CollisionParams;
		CollisionParams.AddIgnoredActor(GetOwner()); // игнорировать текущий Actor

		bool bWasFalling = bIsFalling; // кэш прошлого падения
		FCollisionShape Sphere = FCollisionShape::MakeSphere(SphereRadius); // сферическая проверяющая коллизия
		bIsFalling = !GetWorld()->SweepSingleByChannel(HitResult, StartPoint, EndPoint, FQuat::Identity, ECC_Visibility, Sphere, CollisionParams); // выпуск луча со сферой и проверка на столкновение

		if (bIsFalling)
		{
			VerticalVelocity += GetGravityZ() * FVector::UpVector * DeltaTime; // если падаем, увеличиваем скорость падения
		}
		else if (bWasFalling && VerticalVelocity.Z < 0.0f) // если прошлым кадром приземлились и есть скорость падения
		{
			VerticalVelocity = FVector::ZeroVector; // занулить скорость падения
		}
	}

	Velocity += VerticalVelocity; // к скорости перемещения по поверности добавить скорость падения
	FVector Delta = Velocity * DeltaTime; // преобразование скорости к времени обновления кадра
	if (!Delta.IsNearlyZero(1e-6f)) // сместились немного - не считается
	{
		FQuat Rot = UpdatedComponent->GetComponentQuat(); // текущий поворот объекта
		FHitResult Hit(1.f); // [«Время» удара вдоль направления трассы (в диапазоне от 0,0 до 1,0), если есть попадание, указывающее время между началом трассы и концом трассы.]
		SafeMoveUpdatedComponent(Delta, Rot, true, Hit); // безопасное перемещения объекта

		if (Hit.IsValidBlockingHit()) // если было столкновение
		{
			HandleImpact(Hit, DeltaTime, Delta); // [Если в процессе падения встречается препятствие, HandleImpact будет выполняться первым, чтобы придать силу объекту, которого коснулись.]
			// Try to slide the remaining distance along the surface.
			SlideAlongSurface(Delta, 1.f - Hit.Time, Hit.Normal, Hit, true); // скольжение персонажа
		}
	}

	UpdateComponentVelocity(); // обновление Velocity объекта
}
```

Логика Jump нет в Pawn. Ее нужно самим реализовать

```c++
UPROPERTY(EditAnywhere)
float InitialJumpVelocity = 500.0f;

FVector VerticalVelocity = FVector::ZeroVector;

void UFPBasePawnMovementComponent::JumpStart()
{
	VerticalVelocity = InitialJumpVelocity * FVector::UpVector;
}
```
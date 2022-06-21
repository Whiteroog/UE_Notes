# TimeLine

TimeLine можно создать на основе Curve (кривой).

Curve - файл, хранящий значения, которые относятся к времени (зависят от времени).

```c++
UPROPERTY(EditAnywhere, BlueprintReadOnly)
UCurveFloat* TimelineCurve;
```

Создание и привязка метода к TimeLine

```c++
FTimeline PlatformTimeline;

if (IsValid(TimelineCurve))
{
	FOnTimelineFloatStatic PlatformMovementTimelineUpdate;
	PlatformMovementTimelineUpdate.BindUObject(this, &ABasePlatform::PlatformTimelineUpdate); // привязка метода
	PlatformTimeline.AddInterpFloat(TimelineCurve, PlatformMovementTimelineUpdate); // применение Curve к методу
}
```

`IsValid(TimelineCurve)` - проверка на NULL значения

Привязанный метод, обязательно должен иметь один float аргумент, в который будет передаваться значение из Curve

```c++
void ABasePlatform::PlatformTimelineUpdate(const float Alpha)
{
	const FVector PlatformTargetLocation = FMath::Lerp(StartLocation, EndLocation, Alpha);
	PlatformMesh->SetRelativeLocation(PlatformTargetLocation);
}
```

Передаваемый аргумент расчитывается в методе Tick, где методу TickTimeline передается разность обновления кадра (мс)

```c++
void ABasePlatform::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	PlatformTimeline.TickTimeline(DeltaTime);
}
```

### Операции с TimeLine

```c++
PlatformTimeline.Play(); // старт проигрывания Curve

PlatformTimeline.Reverse(); // обратный ход Curve

PlatformTimeline.SetLooping(true); // зацикленный Curve
```


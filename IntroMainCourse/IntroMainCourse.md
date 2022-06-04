# Механика платформы

Создание платформы

```c++
UPROPERTY(EditAnywhere, BlueprintReadOnly)
UStaticMeshComponent* PlatformMesh;

ABasePlatform::ABasePlatform()
{
	PrimaryActorTick.bCanEverTick = true;
	USceneComponent* DefaultPlatformRoot = CreateDefaultSubobject<USceneComponent>(TEXT("Platform root"));
	RootComponent = DefaultPlatformRoot;

	PlatformMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Platform"));
	PlatformMesh->SetupAttachment(DefaultPlatformRoot);
}
```

`CreateDefaultSubobject<...>` - создает объект (подобие создание экземпляра класса)

`USceneComponent` - компонент "сцена". Объект не имеющий физическое и визуальное представления.
Создан для того, чтобы дочерний элемент **платформы** использовал свои координаты, а не мира.
Так проще разрабатывать точку перемещения.

`RootComponent` - компонент, которые имеется у каждого Actor

`SetupAttachment(RootComponent)` - метод привязки

----

Определение платформы

```c++
FTimeline PlatformTimeline;

void ABasePlatform::BeginPlay()
{
	Super::BeginPlay();

	StartLocation = PlatformMesh->GetRelativeLocation();

	if (IsValid(TimelineCurve))
	{
		FOnTimelineFloatStatic PlatformMovementTimelineUpdate;
		PlatformMovementTimelineUpdate.BindUObject(this, &ABasePlatform::PlatformTimelineUpdate);
		PlatformTimeline.AddInterpFloat(TimelineCurve, PlatformMovementTimelineUpdate);
	}

	PlatformTimeline.SetLooping(PlatformBehavior == EPlatformBehavior::Loop);

	if (IsValid(PlatformInvocator))
		PlatformInvocator->OnInvoctorActivated.AddUObject(this, &ABasePlatform::OnPlatformInvoked);

	//PlatformTimeline.Play();
}
```

`GetRelativeLocation()` - метод возвращающий относительную позицию данного компонента. Так в данном примере, этот компонент дочерний то он будет всегда возращать (если не перемещать) нулевую координату.

`IsValid(Value)` - ue функция проверки на null.

`FTimeline` - класс таймлайн. Таймлайн - это отрезок из которого получают значение между началом и концом в зависимости от времени

`FOnTimelineFloatStatic` - timeline делегат

Подписка делегата на функцию

```c++
PlatformMovementTimelineUpdate.BindUObject(this, &ABasePlatform::PlatformTimelineUpdate);
```

`AddInterpFloat(TimelineCurve, PlatformMovementTimelineUpdate);` - добавление отрезка ( `UCurveFloat* TimelineCurve;` )

`SetLooping(true);` - зацикливание таймлайна

Подписка на собственный делегат, который срабатывает если войти в `Overlap` область

```c++
PlatformInvocator->OnInvoctorActivated.AddUObject(this, &ABasePlatform::OnPlatformInvoked);
```

----

Метод `Tick`, в котором подписанный таймлайн должен работать в каждом кадре. Он работает за счет добавления в таймлайн `DeltaTime`

```c++
void ABasePlatform::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	PlatformTimeline.TickTimeline(DeltaTime);
}
```

----

Метод таймлайна

```c++
void ABasePlatform::PlatformTimelineUpdate(const float Alpha)
{
	const FVector PlatformTargetLocation = FMath::Lerp(StartLocation, EndLocation, Alpha);
	PlatformMesh->SetRelativeLocation(PlatformTargetLocation);
}
```

`FMath::Lerp(StartLocation, EndLocation, Alpha);` - линейная интерполяция, которое дает значение между первым и вторым значением, в зависимости от alpha

`SetRelativeLocation(PlatformTargetLocation);` - метод задающий относительную позицию данного компонента.

----

Метод вызывающийся собственным делегатом

```c++
void ABasePlatform::OnPlatformInvoked() // старт
{
	PlatformTimeline.Play();

	if(bIsAutoReverse && PlatformBehavior == EPlatformBehavior::OnDemand)
		AutoReverse();
}
```

`Play()` - запускает таймлайн. По завершению таймлайна, платформа останется в конце. Если еще раз запустить, то платформа телепортнется в начало.

----

Метод в котором реализован таймер. Данный метод запускается сразу, как только платформа стартует, поэтому чтобы платформа стояла по выставленному времени задержки, был учтен общий временной отрезок

```c++
FTimerHandle ReverseTimerPlatform;

UPROPERTY(EditAnywhere, BlueprintReadOnly)
float MaxIdlePlatformTime = 4.0f;


void ABasePlatform::AutoReverse()
{
	float MaxDelay = PlatformTimeline.GetTimelineLength() + MaxIdlePlatformTime; // Получение задержки с учетом пройденного пути
	GetWorld()->GetTimerManager().SetTimer(ReverseTimerPlatform, this, &ABasePlatform::ReversePlatform, MaxDelay, false); // таймер задержки
}
```

`PlatformTimeline.GetTimelineLength()` - из таймлайна можно получить общий временной отрезок (весь по времени отрезок)

```c++
FTimerHandle ReverseTimerPlatform;
GetWorld()->GetTimerManager().SetTimer(ReverseTimerPlatform, this, &ABasePlatform::ReversePlatform, MaxDelay, false);
```

Таймер, по истечению времени запускает следующий метод

```c++
void ABasePlatform::ReversePlatform() // возвращение
{
	PlatformTimeline.Reverse();
}
```

`Reverse()` - запускает обратный таймлайн. С конца отрезка.

----

Объявление собственного делегата без параметров и на множество подписок

```c++
DECLARE_MULTICAST_DELEGATE(FOnInvocatorActivated)

FOnInvocatorActivated OnInvoctorActivated;
```

Данный метод был подписан на делегат в blueprint редакторе

```c++
void APlatformInvocator::Invoke()
{
	if (OnInvoctorActivated.IsBound())
		OnInvoctorActivated.Broadcast();
}
```

`IsBound()` - проверка делегата на null

`Broadcast()` - вызов всех подписанных методов
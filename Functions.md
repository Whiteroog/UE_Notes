# Основные функции UE

## Функция выводит на экран сообщение в редакторе

```c++
GEngine->AddOnScreenDebugMessage(-1, 1.0f, FColor::Green, TEXT("Capsule hit!"));
```

`-1` -уникальный ключ для предотвращения повторного добавления одного и того же сообщения

`1.0f` - количество секунд отображения

----

## Функция получения строки, используя параметры

```c++
FString::Printf(TEXT("%f > %f", SurfaceNormal.Z, GetCharacterMovement()))
```

----

## Функция создания объекта

```c++
CreateDefaultSubobject<USphereComponent>(TEXT("Collision"));
```

`<...>` - в скобках указывается тип объекта

`(FName SubobjectName)` - в скобках указывается название компонента

----

## Функция создающая custom компонент на базовом классе

```c++
CreateDefaultSubobject<UPawnMovementComponent, UGCBasePawnMovementComponent>(TEXT("Movement component"));
```

`UPawnMovementComponent` - базовый класс

`UGCBasePawnMovementComponent` - custom компонент

----

## Вывод в Log

```c++
UE_LOG(LogTemp, Log, TEXT("AGameCodeBasePawn::OnBlendComplete() Current view actor: %s"), *CurrentViewActor->GetName());
```

`LogTemp` - категория, куда сообщение будет высылаться. Можно создать свой собственный

`Log` - уровень сообщения (Log, Error, Warning)

`TEXT` - сообщение

`%s` - подставляющий параметр типа string. Таких может быть сколько-угодно

`CurrentViewActor->GetName()` - название компонента

### Создание своей категории

```c++
DECLARE_LOG_CATEGORY_EXTERN(LogCameras, Log, All)
```

`LogCameras` - навзание категории

`Log` - уровень сообщения (Fatal, Error, Warning, Display, Log)

`All` - фильтрация сообщений. [Какие есть другие незнаю]

----

## TimeLine

Кривая для плавного перехода. Обязателен в таймере.

```c++
UPROPERTY(EditAnywhere, BlueprintReadOnly)
UCurveFloat* TimelineCurve;
```

Таймлайн и таймер

```c++
FTimeline PlatformTimeline;
FTimerHandle ReverseTimerPlatform;
```

Зацикленный таймлайн

```c++
PlatformTimeline.SetLooping(true);
```

Запуск

```c++
PlatformTimeline.Play(); // От начала до конца
PlatformTimeline.Reverse(); // От конца до начала
```

Инициализация таймлайна

```c++
if (IsValid(TimelineCurve))
{
	FOnTimelineFloatStatic PlatformMovementTimelineUpdate;
	PlatformMovementTimelineUpdate.BindUObject(this, &ABasePlatform::PlatformTimelineUpdate);
	PlatformTimeline.AddInterpFloat(TimelineCurve, PlatformMovementTimelineUpdate);
}
```

Update таймлайна

```c++
void ABasePlatform::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);
	PlatformTimeline.TickTimeline(DeltaTime);
}
```

Запуск таймера

```c++
GetWorld()->GetTimerManager().SetTimer(ReverseTimerPlatform, this, &ABasePlatform::ReversePlatform, MaxDelay, false);
```

----

## Inverse kinematics

### Character

Название сокетов - задаются в редакторе

```c++
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Character | IK settings")
	FName RightFootSocketName;
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Character | IK settings")
	FName LeftFootSocketName;
```

Дополнительные свойства: скорость смены положения кости и дополнительная дистанция для проверки ног

```c++
UPROPERTY(EditInstanceOnly, BlueprintReadOnly, Category = "Character | IK settings", meta = (ClampMin = 0.0f, UIMin =0.0f))
	float IKInterpSpeed = 20.0f;
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Character | IK settings", meta = (ClampMin = 0.0f, UIMin = 00f))
	float IKTraceExtendDistance = 30.0f;
```

Свойства которые передаются в AnimInstance

```c++
float IKRightFootOffset = 0.0f;
float IKLeftFootOffset = 0.0f;
```

Инициализация свойств:

IKTraceDistance - дистанция между точкой половины капсулы игрока и полом

IKScale - масштаб объекта на сцене

```c++
float IKTraceDistance = 0.0f;
float IKScale = 0.0f;

AFPBaseCharacter::AFPBaseCharacter(const FObjectInitializer& ObjectInitializer)
	: Super(ObjectInitializer.SetDefaultSubobjectClass<UFPBaseCharacterMovementComponent>(ACharacter::CharacterMovementComponentName))
{
	FPBaseCharacterMovementComponent = StaticCast<UFPBaseCharacterMovementComponent*>(GetCharacterMovement());

	IKScale = GetActorScale3D().Z;
	IKTraceDistance = GetCapsuleComponent()->GetScaledCapsuleHalfHeight();
}
```

Получение отклонения из передавемого сокета

```c++
float GetIKOffsetForASocket(const FName& SocketName) const;

float AFPBaseCharacter::GetIKOffsetForASocket(const FName& SocketName) const
{
	float Result = 0.0f; // если нет преграды, то отклонение 0

	if(SocketName == "") // если нет передаваемого отклонения
		return Result;

	FVector SocketLocation = GetMesh()->GetSocketLocation(SocketName); // получение координат сокета из мирового пространства

	FVector TraceStart(SocketLocation.X, SocketLocation.Y, GetActorLocation().Z); // начало координаты - это положение сокета на уровне центра персонажа
	FVector TraceEnd = TraceStart - IKTraceDistance * FVector::UpVector; // координаты пола 

	FHitResult HitResult;
	ETraceTypeQuery TraceType = UEngineTypes::ConvertToTraceType(ECC_Visibility); // Trace типа Visibility

	if (UKismetSystemLibrary::LineTraceSingle(GetWorld(), TraceStart, TraceEnd, TraceType, true, TArray<AActor*>(), EDrawDebugTrace::ForOneFrame, HitResult, true))
	{
		Result = (TraceEnd.Z - HitResult.Location.Z) / IKScale; // дистанция между полом персонажа и преградой. Отрицательная величина
	}
	else if (UKismetSystemLibrary::LineTraceSingle(GetWorld(), TraceEnd, TraceEnd - IKTraceExtendDistance * FVector::UpVector, TraceType, true, TArray<AActor*>(), EDrawDebugTrace::ForOneFrame, HitResult, true))
	{
		Result = (TraceEnd.Z - HitResult.Location.Z) / IKScale; // Положительная величина
	}

	return Result;
}
```

Постоянное обновление положение ног

```c++
void AFPBaseCharacter::Tick(float DeltaSeconds)
{
	Super::Tick(DeltaSeconds);

	//...

	IKRightFootOffset = FMath::FInterpTo(IKRightFootOffset, GetIKOffsetForASocket(RightFootSocketName), DeltaSeconds, IKInterpSpeed);
	IKLeftFootOffset = FMath::FInterpTo(IKLeftFootOffset, GetIKOffsetForASocket(LeftFootSocketName), DeltaSeconds, IKInterpSpeed);
}
```

### Character Animation Instance

Vector effector - для blueprint Two Bone IK, которое менает положение кости относительно координат родителя

```c++
UPROPERTY(VisibleAnywhere, Transient, BlueprintReadOnly, Category = "Character | IK settings")
	FVector RightFootEffectorLocation = FVector::ZeroVector;
UPROPERTY(VisibleAnywhere, Transient, BlueprintReadOnly, Category = "Character | IK settings")
	FVector LeftFootEffectorLocation = FVector::ZeroVector;
```

Получение Character

```c++
TWeakObjectPtr<class AFPBaseCharacter> CachedBaseCharacter;

void UFPBaseCharacterAnimInstance::NativeBeginPlay()
{
	Super::NativeBeginPlay();

	checkf(TryGetPawnOwner()->IsA<AFPBaseCharacter>(), TEXT("UFPBaseCharacterAnimInstance::NativeBeginPlay() UFPBaseCharacterAnimInstance can be used only with AFPBaseCharacter"))
	CachedBaseCharacter = StaticCast<AFPBaseCharacter*>(TryGetPawnOwner());
}
```

Установка Vector effector относительного смещения кости

```c++
void UFPBaseCharacterAnimInstance::NativeUpdateAnimation(float DeltaSeconds)
{
	Super::NativeUpdateAnimation(DeltaSeconds);

	if(!CachedBaseCharacter.IsValid())
		return;

	// ...

	RightFootEffectorLocation = FVector(CachedBaseCharacter->GetIKRightFootOffset(), 0.0f, 0.0f);
	LeftFootEffectorLocation = FVector(CachedBaseCharacter->GetIKLeftFootOffset() * -1, 0.0f, 0.0f); // Левая нога имеет противоположное направление
}
```

Многие кости поменяли направление координат: для ног, направление-Z - это направление-X
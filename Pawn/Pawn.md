# Pawn

### Инициализация pawn с коллизией и компонентом передвижения

```c++
AFirstProjectBasePawn::AFirstProjectBasePawn()
{
 	// Set this pawn to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	CollisionComponent = CreateDefaultSubobject<USphereComponent>(TEXT("Collision"));
	CollisionComponent->SetSphereRadius(50.0f);
	CollisionComponent->SetCollisionProfileName(UCollisionProfile::Pawn_ProfileName);
	RootComponent = CollisionComponent;

	//MovementComponent = CreateDefaultSubobject<UPawnMovementComponent, UFloatingPawnMovement>(TEXT("Movement component"));
	MovementComponent = CreateDefaultSubobject<UPawnMovementComponent, UGCBasePawnMovementComponent>(TEXT("Movement component"));
	MovementComponent->SetUpdatedComponent(CollisionComponent);
}
```

`SetCollisionProfileName(UCollisionProfile::Pawn_ProfileName);` - задает параметры коллизии из существующих имен.
В данном примере - все блокируется, кроме visibility

`SetUpdatedComponent(CollisionComponent);` - здесь указывается, какой компонент будет двигаться. В данном случает тот, что и root

----

### Подписка кнопок на функции их выполнения

```c++
void AFirstProjectBasePawn::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	PlayerInputComponent->BindAxis("Turn", this, &APawn::AddControllerYawInput);
	PlayerInputComponent->BindAxis("LookUp", this, &APawn::AddControllerPitchInput);
	PlayerInputComponent->BindAxis("MoveForward", this, &AFirstProjectBasePawn::MoveForward);
	PlayerInputComponent->BindAxis("MoveRight", this, &AFirstProjectBasePawn::MoveRight);
	PlayerInputComponent->BindAction("Jump", EInputEvent::IE_Pressed, this, &AFirstProjectBasePawn::Jump);
}
```

`BindAxis` - задает отрезок [-1; 1] или тот который был задан в редакторе

`"Turn"` - навзавние клавиши, которое было задано в редакторе

`&APawn::AddControllerYawInput` - функция привязки

`BindAction` - задает характер нажатой клавиши

`EInputEvent::IE_Pressed` - характер клавиши

----

## Базовое перемещение по плоскости

```c++
AActor* CurrentViewActor;

void AFirstProjectBasePawn::MoveForward(float Value)
{
	if(Value != 0)
		AddMovementInput(CurrentViewActor->GetActorForwardVector(), Value);
}

void AFirstProjectBasePawn::MoveRight(float Value)
{
	if(Value != 0)
		AddMovementInput(CurrentViewActor->GetActorRightVector(), Value);
}
```

`CurrentViewActor` можно опустить, т.к. здесь он используется для упрощения перемещения если камера pawn поменяется (т.е. если на сцене есть камера на которую игрок переключется, данный pawn будет подстраиваться под камеру).

----

## Прыжок из custom movement

```c++
void AFirstProjectBasePawn::Jump()
{
	checkf(MovementComponent->IsA<UGCBasePawnMovementComponent>(), TEXT("AFirstProjectBasePawn::Jump() can work only with UGCBasePawnMovementComponent"));
	UGCBasePawnMovementComponent* BaseMovement = StaticCast<UGCBasePawnMovementComponent*>(MovementComponent);
	BaseMovement->JumpStart();
}
```

`checkf` - функция, которая вызывает ошибку в редакторе, если условие не выполняется. В данном случае проверка `IsA` - проверка на схожесть класса. Второй параметр текст для Log

`StaticCast<UGCBasePawnMovementComponent*>(MovementComponent);` - приведение нужного типа. [Так как MovementComponent создавался как custom объект, он создался как базовый, но имеет наследника в которого может превратиться]

----

## Работа с камерой

### Определение камеры на pawn

```c++
void AFirstProjectBasePawn::BeginPlay()
{
	Super::BeginPlay();

	APlayerCameraManager* CameraManager = UGameplayStatics::GetPlayerCameraManager(GetWorld(), 0);
	CurrentViewActor = CameraManager->GetViewTarget();
	CameraManager->OnBlendComplete().AddUFunction(this, FName("OnBlendComplete"));
}
```

`UGameplayStatics::GetPlayerCameraManager(GetWorld(), 0);` - стандартное получение camera meneger. Первый параметр всегда мир, второй параметр  - индекс игрока (в однопользовательской игре это один игрок [0], в многоп. - это **не** индекс игрока по сети [???])

`GetViewTarget()` - возращает actor на котором висит действующая камера (камера через которую видит игрок)

`OnBlendComplete()` - делегат, который срабатывает, когда анимация перехода камеры завершается

`AddUFunction(this, FName("OnBlendComplete"))` - подписка на делегат, передается this класса и название функции

```c++
void AFirstProjectBasePawn::OnBlendComplete()
{
	CurrentViewActor = GetController()->GetViewTarget();
}
```

`GetController()->GetViewTarget()` - получение actor через controller, вызвав функцию полчения actor, которая показывает камера

### pawn камера, на которую переключается игрок, когда подходит близко

```c++
AInteractiveCameraActor::AInteractiveCameraActor()
{
	BoxComponent = CreateDefaultSubobject<UBoxComponent>(TEXT("Camera interaction volume"));
	BoxComponent->SetBoxExtent(FVector(500.0f, 500.0f, 500.0f));
	BoxComponent->SetCollisionObjectType(ECC_WorldDynamic);
	BoxComponent->SetCollisionResponseToAllChannels(ECR_Ignore);
	BoxComponent->SetCollisionResponseToChannel(ECC_Pawn, ECR_Overlap);
	BoxComponent->SetupAttachment(RootComponent);
}
```

`SetBoxExtent(FVector())` - задается размер box

`SetCollisionObjectType(ECC_WorldDynamic)` - установка тип данной коллизии (коллизия динамическая)

`SetCollisionResponseToAllChannels(ECR_Ignore)` - установка все параметры коллизии ignore

`SetCollisionResponseToChannel(ECC_Pawn, ECR_Overlap)` - установка одному параметру: реагирование на коллизии pawn - overlap (пересечение коллизии)

----

### Подписка делегатов

```c++
BoxComponent->OnComponentBeginOverlap.AddDynamic(this, &AInteractiveCameraActor::OnBeginOverlap);
BoxComponent->OnComponentEndOverlap.AddDynamic(this, &AInteractiveCameraActor::OnEndOverlap);
```

`OnComponentBeginOverlap` - первое пересечение коллизии

`OnComponentEndOverlap` - выход коллизии из другой

----

### Функция при пересечении коллизии

```c++
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Transition settings")
	float TransitionToCameraTime = 2.0f;

void AInteractiveCameraActor::OnBeginOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor,
	UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	APlayerController* PlayerController = UGameplayStatics::GetPlayerController(GetWorld(), 0);

	FViewTargetTransitionParams TransitionToCameraParams;
	TransitionToCameraParams.BlendTime = TransitionToCameraTime;

	PlayerController->SetViewTarget(this, TransitionToCameraParams);

}
```

`BlendTime` - скорость перехода камеры

`SetViewTarget(this, TransitionToCameraParams)` - смема активности камеры на данный класс с камерой. Передача параметров перехода камеры

----

### Функция при выходе коллизии

```c++
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Transition settings")
	float TransitionToPawnTime = 2.0f;

void AInteractiveCameraActor::OnEndOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor,
	UPrimitiveComponent* OtherComp, int32 OtherBodyIndex)
{
	APlayerController* PlayerController = UGameplayStatics::GetPlayerController(GetWorld(), 0);
	APawn* Pawn = PlayerController->GetPawn();

	FViewTargetTransitionParams TransitionToPawnParams;
	TransitionToPawnParams.BlendTime = TransitionToPawnTime;
	TransitionToPawnParams.bLockOutgoing = true;
	PlayerController->SetViewTarget(Pawn, TransitionToPawnParams);
}
```

`PlayerController->GetPawn()` - в данной ситуации, player controller привязан к pawn, поэтому для возвращении камеры берется pawn

`bLockOutgoing = true` - чтобы переход следовал за pawn

`SetViewTarget(Pawn, TransitionToPawnParams)` - переход камеры на pawn с камерой

----

## Собственный MovementComponent

```c++
UCLASS()
class FIRSTPROJECT_API UGCBasePawnMovementComponent : public UPawnMovementComponent
{
	GENERATED_BODY()
	
	virtual void TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction) override;

public:
	void JumpStart();

protected:
	UPROPERTY(EditAnywhere)
	float MaxSpeed = 1200.0f;

	UPROPERTY(EditAnywhere)
	float InitialJumpVelocity = 500.0f;

	UPROPERTY(EditAnywhere)
	bool bIsEnableGravity;

private:
	FVector VerticalVelocity = FVector::ZeroVector;
	bool bIsFalling = false;
};

void UGCBasePawnMovementComponent::TickComponent(float DeltaTime, ELevelTick TickType,
	FActorComponentTickFunction* ThisTickFunction)
{
	if (ShouldSkipUpdate(DeltaTime))
	{
		return;
	}

	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);

	FVector PendingInput = GetPendingInputVector().GetClampedToMaxSize(1.0f);
	Velocity = PendingInput * MaxSpeed;
	ConsumeInputVector();

	if (bIsEnableGravity)
	{
		FHitResult HitResult;
		FVector StartPoint = UpdatedComponent->GetComponentLocation();
		float TraceDepth = 1.0f;
		float SphereRadius = 50.0f;
		FVector EndPoint = StartPoint - TraceDepth * FVector::UpVector;
		FCollisionQueryParams CollisionParams;
		CollisionParams.AddIgnoredActor(GetOwner());

		bool bWasFalling = bIsFalling;
		FCollisionShape Sphere = FCollisionShape::MakeSphere(SphereRadius);
		bIsFalling = !GetWorld()->SweepSingleByChannel(HitResult, StartPoint, EndPoint, FQuat::Identity, ECC_Visibility, Sphere, CollisionParams);
		if (bIsFalling)
		{
			VerticalVelocity += GetGravityZ() * FVector::UpVector * DeltaTime;
		}
		else if (bWasFalling && VerticalVelocity.Z < 0.0f)
		{
			VerticalVelocity = FVector::ZeroVector;
		}
	}

	Velocity += VerticalVelocity;
	FVector Delta = Velocity * DeltaTime;
	if (!Delta.IsNearlyZero(1e-6f))
	{
		FQuat Rot = UpdatedComponent->GetComponentQuat();
		FHitResult Hit(1.f);
		SafeMoveUpdatedComponent(Delta, Rot, true, Hit);

		if (Hit.IsValidBlockingHit())
		{
			HandleImpact(Hit, DeltaTime, Delta);
			// Try to slide the remaining distance along the surface.
			SlideAlongSurface(Delta, 1.f - Hit.Time, Hit.Normal, Hit, true);
		}
	}

	UpdateComponentVelocity();
}
```

```c++
virtual void TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction) override;
```

В `PawnMovementComponent` переопределяется метод Tick, который дает движение объекту

`ShouldSkipUpdate(DeltaTime)` - функция возвращает true, если объект не двигается. Используется для оптимизации

`GetPendingInputVector()` - вектор который ввел пользователь. Т.е. если игрок идет вперед - вправо, то вектор будет
1, 1, 0

`GetClampedToMaxSize(1.0f)` - создает копию вектора с максимальной величиной. 0 -> 1 превращаться не будет

`ConsumeInputVector()` - очищает вводимый вектор из `GetPendingInputVector()`

----

### Гравитация

```c++
	if (bIsEnableGravity)
	{
		FHitResult HitResult; // Vector пересечения луча с объектом
		FVector StartPoint = UpdatedComponent->GetComponentLocation(); // получение vector actor
		float TraceDepth = 1.0f; // длина луча
		float SphereRadius = 50.0f; // радиус в см сферы луча
		FVector EndPoint = StartPoint - TraceDepth * FVector::UpVector; // vector куда направится луч
		FCollisionQueryParams CollisionParams; // параметры коллизии луча
		CollisionParams.AddIgnoredActor(GetOwner()); // луч игнорирует owner

		bool bWasFalling = bIsFalling; // сохранение прошлого состояния падения
		FCollisionShape Sphere = FCollisionShape::MakeSphere(SphereRadius); // создание сферы луча
		bIsFalling = !GetWorld()->SweepSingleByChannel(HitResult, StartPoint, EndPoint, FQuat::Identity, ECC_Visibility, Sphere, CollisionParams);
		if (bIsFalling)
		{
			VerticalVelocity += GetGravityZ() * FVector::UpVector * DeltaTime;
		}
		else if (bWasFalling && VerticalVelocity.Z < 0.0f) // с кэшировано прошлое падение. И прошлое падение обнулило VerticalVelocity. Оптимизация
		{
			VerticalVelocity = FVector::ZeroVector;
		}
	}
```

`SweepSingleByChannel(HitResult, StartPoint, EndPoint, FQuat::Identity, ECC_Visibility, Sphere, CollisionParams)` - создание луча коллизии

`GetGravityZ()` - константа из world settings

----

### Перемещение со скольжением по поверхности

```c++
    Velocity = PendingInput * MaxSpeed;

    ...

	Velocity += VerticalVelocity; // добавление к текущему перемещению XY перемещение Z
	FVector Delta = Velocity * DeltaTime; // нормализация скорости временем
	if (!Delta.IsNearlyZero(1e-6f)) // если почти стоим - мы стоим
	{
		FQuat Rot = UpdatedComponent->GetComponentQuat(); // текущее вращение
		FHitResult Hit(1.f);
		SafeMoveUpdatedComponent(Delta, Rot, true, Hit); // безопасное перемещение объекта с учетом столкновениями. bSweep = true - это учитывание ограниченного пространства капсулой

		if (Hit.IsValidBlockingHit()) // если было блокирующее попадание
		{
			HandleImpact(Hit, DeltaTime, Delta); // метод предотвращает проникновения. Блокирует перемещение
			SlideAlongSurface(Delta, 1.f - Hit.Time, Hit.Normal, Hit, true); // метод дающий скольжение об объекты
		}
	}

	UpdateComponentVelocity();
```

`UpdateComponentVelocity()` -  это должно вызываться в конце обновления всякий раз, когда скорость изменяется. Нужно вызывать этот метод, чтобы изменить Velocity UpdatedComponent после изменения переменной Velocity. Однако в pawn я не обнаружил, что использую ускорение velocity [???].

----

## Инверсионная кинематика для паука

Определение положения ножки паука определена в следующем методе

```c++
float ASpiderPawn::GetIKOffsetForASocket(const FName& SocketName)
{
	float Result = 0.0f; // число отклонения кости. По умолчанию нет отклонения

	FVector SocketLocation = SkeletalMeshComponent->GetSocketLocation(SocketName);  // местополение сокета в мировом пространстве

	FVector TraceStart(SocketLocation.X, SocketLocation.Y, GetActorLocation().Z); // точка на уровни центра pawn сферы, но онсительно XY лапки
	FVector TraceEnd = TraceStart - IKTraceDistance * FVector::UpVector; // точка пола

	FHitResult HitResult;
	ETraceTypeQuery TraceType = UEngineTypes::ConvertToTraceType(ECC_Visibility); //  Свой trace type не определили, но можно сконвертить из collision channel
	if(UKismetSystemLibrary::LineTraceSingle(GetWorld(), TraceStart, TraceEnd, TraceType, true, TArray<AActor*>(), EDrawDebugTrace::ForOneFrame, HitResult, true)) // проверка до пола
	{
		Result = (TraceEnd.Z - HitResult.Location.Z) / IKScale; // расстояние на которое ножка должна оттодвинуться в относительных координатах через сокет. От пола на отрицательную величину
	}
	else if (UKismetSystemLibrary::LineTraceSingle(GetWorld(), TraceEnd, TraceEnd - IKTraceExtendDistance * FVector::UpVector, TraceType, true, TArray<AActor*>(), EDrawDebugTrace::ForOneFrame, HitResult, true)) // проверка ниже пола
	{
		Result = (TraceEnd.Z - HitResult.Location.Z) / IKScale; // от пола на положительную вел.
	}

	return Result;
}
```

Данный метод выводит глобальное местополение ножки по координате-Z

Методы анимации

```c++
void USpiderPawnAnimInstance::NativeBeginPlay()
{
	Super::NativeBeginPlay();

	checkf(TryGetPawnOwner()->IsA<ASpiderPawn>(), TEXT("USpiderPawnAnimInstance::NativeBeginPlay() SpiderPawnAnimInstance can bbe used with spider pawn only"));
	CachedSpiderPawnOwner = StaticCast<ASpiderPawn*>(TryGetPawnOwner());
}

void USpiderPawnAnimInstance::NativeUpdateAnimation(float DeltaSeconds)
{
	Super::NativeUpdateAnimation(DeltaSeconds);

	if(!CachedSpiderPawnOwner.IsValid())
		return;

	RightFrontFootEffectorLocation = FVector(CachedSpiderPawnOwner->GetIKRightFrontFootOffset(), 0.0f, 0.0f);
	RightRearFootEffectorLocation = FVector(CachedSpiderPawnOwner->GetIKRightRearFootOffset(), 0.0f, 0.0f);
	LeftFrontFootEffectorLocation = FVector(CachedSpiderPawnOwner->GetIKLeftFrontFootOffset(), 0.0f, 0.0f);
	LeftRearFootEffectorLocation = FVector(CachedSpiderPawnOwner->GetIKLeftRearFootOffset(), 0.0f, 0.0f);
}
```

Получение от класса родителя APawn -> ASpiderPawn

```c++
void USpiderPawnAnimInstance::NativeBeginPlay()
{
	Super::NativeBeginPlay();

	checkf(TryGetPawnOwner()->IsA<ASpiderPawn>(), TEXT("USpiderPawnAnimInstance::NativeBeginPlay() SpiderPawnAnimInstance can bbe used with spider pawn only"));
	CachedSpiderPawnOwner = StaticCast<ASpiderPawn*>(TryGetPawnOwner());
}
```

Конечная точка для перемещения ножки в относительных координатах. Вектор для Two Bone IK

```c++
void USpiderPawnAnimInstance::NativeUpdateAnimation(float DeltaSeconds)
{
	Super::NativeUpdateAnimation(DeltaSeconds);

	if(!CachedSpiderPawnOwner.IsValid())
		return;

	RightFrontFootEffectorLocation = FVector(CachedSpiderPawnOwner->GetIKRightFrontFootOffset(), 0.0f, 0.0f);
	RightRearFootEffectorLocation = FVector(CachedSpiderPawnOwner->GetIKRightRearFootOffset(), 0.0f, 0.0f);
	LeftFrontFootEffectorLocation = FVector(CachedSpiderPawnOwner->GetIKLeftFrontFootOffset(), 0.0f, 0.0f);
	LeftRearFootEffectorLocation = FVector(CachedSpiderPawnOwner->GetIKLeftRearFootOffset(), 0.0f, 0.0f);
}
```

Установка относительных координат сокета (который настраивает Two Bone IK) от родителя конца ножки. Данная ножка повернута X в сторону Z, поэтому изменяется X

----
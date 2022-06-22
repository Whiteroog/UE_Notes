# Стандартный набор компонентов для Pawn

```c++
AFirstProjectBasePawn::AFirstProjectBasePawn()
{
	CollisionComponent = CreateDefaultSubobject<USphereComponent>(TEXT("Collision")); // коллизия сферы
	CollisionComponent->SetSphereRadius(CollisionSphereRadius);
	CollisionComponent->SetCollisionProfileName(UCollisionProfile::Pawn_ProfileName); // установка зарезервированного набора параметров коллизии (Pawn)
	RootComponent = CollisionComponent;

	//MovementComponent = CreateDefaultSubobject<UPawnMovementComponent, UFloatingPawnMovement>(TEXT("Movement component")); // установка 
	MovementComponent = CreateDefaultSubobject<UPawnMovementComponent, UFPBasePawnMovementComponent>(TEXT("Movement component")); // установка собственного Movement Component
	MovementComponent->SetUpdatedComponent(CollisionComponent);

	SpringArmComponent = CreateDefaultSubobject<USpringArmComponent>(TEXT("SpringArm")); // штатив для камеры
	SpringArmComponent->bUsePawnControlRotation = 1; // возможность направлять движение камерой
	SpringArmComponent->SetupAttachment(RootComponent);

	CameraComponent = CreateDefaultSubobject<UCameraComponent>(TEXT("Camera"));
	CameraComponent->SetupAttachment(SpringArmComponent);

#if WITH_EDITORONLY_DATA
	ArrowComponent = CreateDefaultSubobject<UArrowComponent>(TEXT("Arrow")); // debug информация - forward направление объекта
	ArrowComponent->SetupAttachment(RootComponent);
#endif

}
```

`CreateDefaultSubobject<БазовоыйКомпонент, ПереопределенныйКомпонентОтБазового>(TEXT("Movement component"))`

# Привязка клавишь к методам перемещения

Привязка клавишь к осям управления. Ось управления - выходное значение в пределах диапазона (стандарт: [-1; 1])

```c++
void AFirstProjectBasePawn::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	PlayerInputComponent->BindAxis("Turn", this, &APawn::AddControllerYawInput);

	PlayerInputComponent->BindAxis("MoveForward", this, &AFirstProjectBasePawn::MoveForward);
}
```

`"Turn"` - название привязки клавишь в редакторе.

`&APawn::AddControllerYawInput` - метод класса `APawn`, которое управляет вращением объекта.

Для собственных методов передвижения, для Axis, метод должен иметь обязательно float параметр.

```c++
void AFirstProjectBasePawn::MoveForward(float Value)
{
	if(Value != 0.0f)
		AddMovementInput(GetActorForwardVector(), Value);
}
```

`GetActorForwardVector()` - метод возращающий Forward вектор направления игрока

`AddMovementInput` - для Movement Component передает значение передвижения

# Jump

Метода Jump для Pawn нет, поэтому его реализуют в собственном Movement Component

```c++
	UPROPERTY(VisibleAnywhere)
	UPawnMovementComponent* MovementComponent;

// ...

	MovementComponent = CreateDefaultSubobject<UPawnMovementComponent, UFPBasePawnMovementComponent>(TEXT("Movement component"));

// ...

void AFirstProjectBasePawn::Jump()
{
	checkf(MovementComponent->IsA<UFPBasePawnMovementComponent>(), TEXT("AFirstProjectBasePawn::Jump() can work only with UFPBasePawnMovementComponent"));
	UFPBasePawnMovementComponent* BaseMovement = StaticCast<UFPBasePawnMovementComponent*>(MovementComponent);
	BaseMovement->JumpStart();
}
```

`checkf` - функция, которая используется для проверки подходящих классов

`IsA` - проверяет на соответсвие классов

Если, используется другой класс, то вызывается критическая ошибка с сообщением

`StaticCast` - не динамический каст. Преобразует только передавемый тип в другой тип. Не преобразует тип динамически, если тип поменялся после каста.

# Работа с интерактивной камерой

Для управления персонажем от лица другой камеры, чтобы персонаж управлялся в зависимости, куда смотрит камера, то нужно использовать за основу этого Actor.

Получение текущего Actor, к которому привязана камера.

```c++
void AFirstProjectBasePawn::BeginPlay()
{
	Super::BeginPlay();
	APlayerCameraManager* CameraManager = UGameplayStatics::GetPlayerCameraManager(GetWorld(), 0);
	CurrentViewActor = CameraManager->GetViewTarget(); // получение Actor в котором используется активная камера
	CameraManager->OnBlendComplete().AddUFunction(this, FName("OnBlendComplete")); // данный event, нужет для того, чтобы, после переключения камеры, получть текущего Actor с активной камерой
}
```

Получние Actor после заверешения перехода камеры:

```c++
void AFirstProjectBasePawn::OnBlendComplete()
{
	CurrentViewActor = GetController()->GetViewTarget();
}
```

Чтобы управлять от лица текущий камеры, нужно брать направление куда смотрит камера:

```c++
void AFirstProjectBasePawn::MoveForward(float Value)
{
	InputForward = Value;
	if(Value != 0.0f)
		AddMovementInput(CurrentViewActor->GetActorForwardVector(), Value);
}
```
# Создание компонентов в Actor

Создание компонентов и настойка иерархии происходит в конструкторе класса. На пример платформа:

```c++
ABasePlatform::ABasePlatform()
{
	PrimaryActorTick.bCanEverTick = true;
	USceneComponent* DefaultPlatformRoot = CreateDefaultSubobject<USceneComponent>(TEXT("Platform root"));
	RootComponent = DefaultPlatformRoot;

	PlatformMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Platform"));
	PlatformMesh->SetupAttachment(DefaultPlatformRoot);
}

```

`CreateDefaultSubobject<USceneComponent>(TEXT("Platform root"));` - создание компонента `USceneComponent` с названием "Platform root"

`USceneComponent` - пустой компонент. Для платформы нужен как корневой компонент из которого можно получать относительные координаты [???]

----

`CreateDefaultSubobject<КлассКомпонента>(TEXT("Название"))` - метод создания компонента

`КакойОбъект->SetupAttachment(В_КакойОбъект)` - присоединение к компоненту

----

На примере Интерактивной камеры, там создается компонент коллизии:

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

`UBoxComponent` - компонент коллизии в виде Box

`SetBoxExtent(FVector(500.0f, 500.0f, 500.0f))` - размеры (не масштаб (scale)) box

`SetCollisionObjectType(ECC_WorldDynamic)` - тип коллизии box

`SetCollisionResponseToAllChannels(ECR_Ignore)` - установка всем типам коллизии параметр Ignore

`SetCollisionResponseToChannel(ECC_Pawn, ECR_Overlap)` - установка коллизии Pawn параметр Overlap (пересечение коллизий)

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
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
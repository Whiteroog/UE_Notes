# Компонент коллизии

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
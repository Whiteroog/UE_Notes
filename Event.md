# Event

Для примитивов компонентов, можно добавлять собственные event. На пример, BoxComponent можно подписать событие на BeginOverlap или EndOverlap:

```c++
void AInteractiveCameraActor::BeginPlay()
{
	Super::BeginPlay();

	BoxComponent->OnComponentBeginOverlap.AddDynamic(this, &AInteractiveCameraActor::OnBeginOverlap);
	BoxComponent->OnComponentEndOverlap.AddDynamic(this, &AInteractiveCameraActor::OnEndOverlap);
}
```

Для подписи таких событий, нужно правильно собрать метод с нужными параметрами:

```c++
UFUNCTION()
void OnBeginOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);

UFUNCTION()
void OnEndOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex);
```

Далее, описать сами методы

```c++
void AInteractiveCameraActor::OnBeginOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor,
	UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{

}
```
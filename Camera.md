# Камера

Для переключения камеры используется: `PlayerController->SetViewTarget(НужныйActor_С_Камерой, ПараметрыПереключенияКамеры)` для действующего Controller игрока.

`SetViewTarget` - переключается на камеру выбранного Actor. Если в Actor есть camera, то метод находит автоматически, если не находит, то создается новая camera и устанавливается в центре Actor

На примере Интерактивной камеры: при пересечении коллизии box, камера игрока переключается на камеру этого Actor:

```c++
void AInteractiveCameraActor::OnBeginOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor,
	UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	APlayerController* PlayerController = UGameplayStatics::GetPlayerController(GetWorld(), 0); // получаем действующий контроллер игрока

	FViewTargetTransitionParams TransitionToCameraParams; // класс с параметрами для переключения
	TransitionToCameraParams.BlendTime = TransitionToCameraTime; // установка время переключения (сглаженное переключение)

	PlayerController->SetViewTarget(this, TransitionToCameraParams); // переключение камеры контроллера на данный объект
}
```

При выходе из коллизии box,  камера возвращается к pawn:

```c++
void AInteractiveCameraActor::OnEndOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor,
	UPrimitiveComponent* OtherComp, int32 OtherBodyIndex)
{
	APlayerController* PlayerController = UGameplayStatics::GetPlayerController(GetWorld(), 0); // получаем действующий контроллер игрока
	APawn* Pawn = PlayerController->GetPawn(); // контроллер привязан к pawn

	FViewTargetTransitionParams TransitionToPawnParams;
	TransitionToPawnParams.BlendTime = TransitionToPawnTime;
	TransitionToPawnParams.bLockOutgoing = true; // флаг на то, что, при переключении, камера будет следовать к объекту
	PlayerController->SetViewTarget(Pawn, TransitionToPawnParams); // обратное переключение к Pawn
}
```

Контроллер не всегда будет привязан к Pawn.
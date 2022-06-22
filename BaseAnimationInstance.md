# Базовый класс анимации

В этом классе создаются переменные для Animation Blueprint, а их значения обновляются в каждом кадре в методе `NativeUpdateAnimation(float DeltaSeconds)`

Пример переменных:

```c++
UPROPERTY(BlueprintReadOnly, Transient, Category = "Base pawn animationinstance")
float InputForward;

UPROPERTY(BlueprintReadOnly, Transient, Category = "Base pawn animationinstance")
float InputRight;

UPROPERTY(BlueprintReadOnly, Transient, Category = "Base pawn animationinstance")
bool bIsInAir;
```

Для получения данных из других классов - их кэшируют:

```c++
TWeakObjectPtr<class AFirstProjectBasePawn> CachedBasePawn;
```

`TWeakObjectPtr` - указатель на класс, который может очиститься при удалении объекта Garbage Collector

Получение класса текущего объекта:

```c++
void UFPBasePawnAnimInstance::NativeBeginPlay()
{
	Super::NativeBeginPlay();

	checkf(TryGetPawnOwner()->IsA<AFirstProjectBasePawn>(), TEXT("UFPBasePawnAnimInstance::NativeBeginPlay() only FirstProjectBasePawn can work with UFPBasePawnAnimInstance"));
	CachedBasePawn = StaticCast<AFirstProjectBasePawn*>(TryGetPawnOwner());
}
```

`TryGetPawnOwner()` - получение Pawn, из текущего объекта

Пример получения данных:

```c++
void UFPBasePawnAnimInstance::NativeUpdateAnimation(float DeltaSeconds)
{
	Super::NativeUpdateAnimation(DeltaSeconds);

	if(!CachedBasePawn.IsValid()) // кэш не NULL
		return;

	InputForward = CachedBasePawn->GetInputForward();
	InputRight = CachedBasePawn->GetInputRight();

	bIsInAir = CachedBasePawn->GetMovementComponent()->IsFalling();
}
```

Все эти методы получения данных пишутся в public собственного класса

```c++
UFUNCTION(BlueprintCallable, BlueprintPure)
float GetInputForward() { return InputForward; }

UFUNCTION(BlueprintCallable, BlueprintPure)
float GetInputRight() { return InputRight; }
```
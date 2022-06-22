# Инверсная кинематика паука

Инверсная кинематика лап паука по координате Z

## Класс объекта

Для инверсной кинематики, сначала создаются Socket в костях объекта, а затем в классе этого объекта кэшируются названия Socket:

```c++
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Spider bot|IKsettings")
FName RightFrontFootSocketName;

UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Spider bot|IKsettings")
FName RightRearFootSocketName;

UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Spider bot|IKsettings")
FName LeftFrontFootSocketName;

UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Spider bot|IKsettings")
FName LeftRearFootSocketName;
```

Для смещения кости создают переменную, которая отклонять кость в нужное направлнение. Смещение кости происходит в относительных координатах родителя кости:

```c++
float IKRightFrontFootOffset = 0.0f;
float IKRightRearFootOffset = 0.0f;
float IKLeftFrontFootOffset = 0.0f;
float IKLeftRearFootOffset = 0.0f;
```

Для получения относительных координат кости - кэшируется Skeletal Mesh объекта

```c++
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Spider bot")
class USkeletalMeshComponent* SkeletalMeshComponent;
```

Однако следует учитывать возможный изменненный масштаб объекта:

```c++
float IKScale = 0.0f;

ASpiderPawn::ASpiderPawn()
{
	SkeletalMeshComponent = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("Spider mesh"));
	SkeletalMeshComponent->SetupAttachment(RootComponent);

	IKScale = GetActorScale3D().Z;
	IKTraceDistance = CollisionSphereRadius * IKScale;
}
```

Метод нахождения относительного отклонения кости:

```c++
float ASpiderPawn::GetIKOffsetForASocket(const FName& SocketName)
{
	float Result = 0.0f; // нулевой отклонение

	FVector SocketLocation = SkeletalMeshComponent->GetSocketLocation(SocketName); // получние относительные координаты сокета, расположенного на кости

	FVector TraceStart(SocketLocation.X, SocketLocation.Y, GetActorLocation().Z); // берутся глобальная высота сокета лапы
	FVector TraceEnd = TraceStart - IKTraceDistance * FVector::UpVector; // пол объекта

	FHitResult HitResult;
	ETraceTypeQuery TraceType = UEngineTypes::ConvertToTraceType(ECC_Visibility); // тип луча для поиска видимых преград
	if(UKismetSystemLibrary::LineTraceSingle(GetWorld(), TraceStart, TraceEnd, TraceType, true, TArray<AActor*>(), EDrawDebugTrace::ForOneFrame, HitResult, true)) // выпуск луча до земли, вместе с отрисовкой луча
	{
		Result = (TraceEnd.Z - HitResult.Location.Z) / IKScale; // плучение отклонения
	}
	else if (UKismetSystemLibrary::LineTraceSingle(GetWorld(), TraceEnd, TraceEnd - IKTraceExtendDistance * FVector::UpVector, TraceType, true, TArray<AActor*>(), EDrawDebugTrace::ForOneFrame, HitResult, true)) // выпуск луча дальше земли
	{
		Result = (TraceEnd.Z - HitResult.Location.Z) / IKScale;
	}

	return Result;
}
```

Чтобы кость быстро не перемещалась используется интерполяция. Здесь же обновляется смещение кости:

```c++
void ASpiderPawn::Tick(float DeltaSeconds)
{
	Super::Tick(DeltaSeconds);

	IKRightFrontFootOffset = FMath::FInterpTo(IKRightFrontFootOffset, GetIKOffsetForASocket(RightFrontFootSocketName), DeltaSeconds, IKInterpSpeed);
	IKRightRearFootOffset = FMath::FInterpTo(IKRightRearFootOffset, GetIKOffsetForASocket(RightRearFootSocketName), DeltaSeconds, IKInterpSpeed);
	IKLeftFrontFootOffset = FMath::FInterpTo(IKLeftFrontFootOffset, GetIKOffsetForASocket(LeftFrontFootSocketName), DeltaSeconds, IKInterpSpeed);
	IKLeftRearFootOffset = FMath::FInterpTo(IKLeftRearFootOffset, GetIKOffsetForASocket(LeftRearFootSocketName), DeltaSeconds, IKInterpSpeed);
}
```

## Анимационный класс

Здесь задаются координаты отклонения Effector для Two Bone IK

```c++
protected:
	UPROPERTY(VisibleAnywhere, Transient, BlueprintReadOnly, Category = "Spider bot|IK settings")
	FVector RightFrontFootEffectorLocation = FVector::ZeroVector;

	UPROPERTY(VisibleAnywhere, Transient, BlueprintReadOnly, Category = "Spider bot|IK settings")
	FVector RightRearFootEffectorLocation = FVector::ZeroVector;

	UPROPERTY(VisibleAnywhere, Transient, BlueprintReadOnly, Category = "Spider bot|IK settings")
	FVector LeftFrontFootEffectorLocation = FVector::ZeroVector;

	UPROPERTY(VisibleAnywhere, Transient, BlueprintReadOnly, Category = "Spider bot|IK settings")
	FVector LeftRearFootEffectorLocation = FVector::ZeroVector;

private:
	TWeakObjectPtr<class ASpiderPawn> CachedSpiderPawnOwner;

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

	RightFrontFootEffectorLocation = FVector(CachedSpiderPawnOwner->GetIKRightFrontFootOffset(), 0.0f, 0.0f); // отклоняется лапа паука по координате X, из-за особенностей модели
	RightRearFootEffectorLocation = FVector(CachedSpiderPawnOwner->GetIKRightRearFootOffset(), 0.0f, 0.0f);
	LeftFrontFootEffectorLocation = FVector(CachedSpiderPawnOwner->GetIKLeftFrontFootOffset(), 0.0f, 0.0f);
	LeftRearFootEffectorLocation = FVector(CachedSpiderPawnOwner->GetIKLeftRearFootOffset(), 0.0f, 0.0f);
}
```
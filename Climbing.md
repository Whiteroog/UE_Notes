# Climbing

Для определения столкновений препятствий с заранее определенными параметрами коллизии, создают в редакторе новый trace channel. Но данный trace не будет создан с таким же именнем в коде, но он будет ссчитаться зарезервированным для редактора. Это первый созданный пользовательский trace channel, поэтому он будет иметь имя `ECC_GameTraceChannel1`.

Чтобы не использовать данное имя, а каким-нибудь другим, для этого создается отдельный файл `.h`, который будет хранить подобные типы. В `.h` файле используют конструкцию `define`, который заменяет в всем коде одно значение на другое.

```c++
#define ECC_Climbing ECC_GameTraceChannel1
```

# Ledge Detector Component

Компонент, который определяет место и возможность взобраться на препятствие.

```c++
  UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Detection settings", meta = (UIMin = 0.0f, ClampMin = 0.0f))
 float MinimumLedgeHeight = 40.0f;

 UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Detection settings", meta = (UIMin = 0.0f, ClampMin = 0.0f))
 float MaximumLedgeHeight = 200.0f;

 UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Detection settings", meta = (UIMin = 0.0f, ClampMin = 0.0f))
 float ForwardCheckDistance = 100.0f;
```

```c++
bool ULedgeDetectorComponent::DetectLedge(FLedgeDescription& LedgeDescription)
{
 UCapsuleComponent* CapsuleComponent = CachedCharacterOwner->GetCapsuleComponent();

 FCollisionQueryParams QueryParams;
 QueryParams.bTraceComplex = true;
 QueryParams.AddIgnoredActor(GetOwner());

#if ENABLE_DRAW_DEBUG
 UDebugSubsystem* DebugSubsystem = UGameplayStatics::GetGameInstance(GetWorld())->GetSubsystem<UDebugSubsystem>();
 bool bIsDebugEnabled = DebugSubsystem->IsCategoryEnabled(DebugCategoryLedgeDetection);
#else
 bIsDebugEnabled = false;
#endif 

 float BottomZOffset = 2.0f;
 FVector CharacterBottom = CachedCharacterOwner->GetActorLocation() - (CapsuleComponent->GetScaledCapsuleHalfHeight()
  - BottomZOffset) * FVector::UpVector;

 // Forward check

 float ForwardCheckCapsuleRadius = CapsuleComponent->GetScaledCapsuleRadius();
 float ForwardCheckHalfHeight = (MaximumLedgeHeight - MinimumLedgeHeight) * 0.5f;

 FVector ForwardStartLocation = CharacterBottom + (MinimumLedgeHeight + ForwardCheckHalfHeight) * FVector::UpVector;
 FVector ForwardEndLocation = ForwardStartLocation + CachedCharacterOwner->GetActorForwardVector() *
  ForwardCheckDistance;

 FHitResult ForwardCheckHitResult;

 float DrawTime = 2.0f;

 if (!FPTraceUtils::SweepCapsuleSingleByChannel(GetWorld(), ForwardCheckHitResult, ForwardStartLocation,
                                                ForwardEndLocation, ForwardCheckCapsuleRadius,
                                                ForwardCheckHalfHeight, FQuat::Identity, ECC_Climbing, QueryParams,
                                                FCollisionResponseParams::DefaultResponseParam, bIsDebugEnabled,
                                                DrawTime))
 {
  return false;
 }

 // Downward check

 FHitResult DownwardCheckHitResult;
 float DownwardSphereCheckRadius = ForwardCheckCapsuleRadius;

 float DownwardCheckDepthOffset = 10.0f;
 FVector DownwardStartLocation = ForwardCheckHitResult.ImpactPoint - ForwardCheckHitResult.ImpactNormal *
  DownwardCheckDepthOffset;
 DownwardStartLocation.Z = CharacterBottom.Z + MaximumLedgeHeight + DownwardSphereCheckRadius;

 FVector DownwardEndLocation(DownwardStartLocation.X, DownwardStartLocation.Y, CharacterBottom.Z);

 if (!FPTraceUtils::SweepShapeSingleByChannel(GetWorld(), DownwardCheckHitResult, DownwardStartLocation,
                                              DownwardEndLocation, DownwardSphereCheckRadius, ECC_Climbing,
                                              QueryParams, FCollisionResponseParams::DefaultResponseParam,
                                              bIsDebugEnabled, DrawTime))
 {
  return false;
 }

 // Overlap check

 float OverlapCapsuleRadius = CapsuleComponent->GetScaledCapsuleRadius();
 float OverlapCapsuleHalfHeight = CapsuleComponent->GetScaledCapsuleHalfHeight();

 float OverlapCapsuleFloorOffset = 2.0f;
 FVector OverlapLocation = DownwardCheckHitResult.ImpactPoint + (OverlapCapsuleHalfHeight +
  OverlapCapsuleFloorOffset) * FVector::UpVector;

 if (FPTraceUtils::OverlapCapsuleAnyByProfile(GetWorld(), OverlapLocation, OverlapCapsuleRadius,
                                              OverlapCapsuleHalfHeight, FQuat::Identity, CollisionProfilePawn, QueryParams,
                                              bIsDebugEnabled, DrawTime))
 {
  return false;
 }

 LedgeDescription.Location = OverlapLocation;
 LedgeDescription.Rotation = (ForwardCheckHitResult.ImpactNormal * FVector(-1.0f, -1.0f, 0.0f)).
  ToOrientationRotator();

 return true;
}
```

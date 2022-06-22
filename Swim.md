# Механика плавания

Скоростью персонажа регулируется специальная Volume-среда.

Для методов плавания задаются свои клавиши, чтобы не было путаницы в методах.

Для движения в воде вперед задается угол, куда смотрит камера, он же, вращение контроллера

```c++
void APlayerCharacter::SwimForward(float Value)
{
	Super::SwimForward(Value);

	if (GetCharacterMovement()->IsSwimming() && !FMath::IsNearlyZero(Value, 1e-6f))
	{
		FRotator YawRotator(GetControlRotation().Pitch, GetControlRotation().Yaw, 0.0f);
		FVector ForwardVector = YawRotator.RotateVector(FVector::ForwardVector);

		AddMovementInput(ForwardVector, Value);
	}
}
```
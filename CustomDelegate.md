# Собственный делегат

Объявление делегата происходит за классом, с помощью макроса. Пример, созданный делегат с множеством возможных подписок на него и без параметров:

```c++
DECLARE_MULTICAST_DELEGATE(FOnInvocatorActivated)
```

Создание делегата происходит в пределах класса

```c++
FOnInvocatorActivated OnInvoctorActivated;
```

Вызов подписок:

```c++
void APlatformInvocator::Invoke()
{
	if (OnInvoctorActivated.IsBound())
		OnInvoctorActivated.Broadcast();
}
```

`IsBound()` - проверка, есть ли подписи

`Broadcast()` - вызов, всех подписок

### Подписка на свой делегат

Объявляется класс, где он хранится

```c++
UPROPERTY(EditInstanceOnly, BlueprintReadOnly)
APlatformInvocator* PlatformInvocator;
```

Подписка происходит через метод `AddUObject`:

```c++
if (IsValid(PlatformInvocator))
	PlatformInvocator->OnInvoctorActivated.AddUObject(this, &ABasePlatform::OnPlatformInvoked);
```
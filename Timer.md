# Timer

Timer можно задать через TimerManager следующим образом:

```c++
FTimerHandle ReverseTimerPlatform;

void ABasePlatform::AutoReverse()
{
	float MaxDelay = PlatformTimeline.GetTimelineLength() + MaxIdlePlatformTime; // Получение задержки с учетом пройденного пути
	GetWorld()->GetTimerManager().SetTimer(ReverseTimerPlatform, this, &ABasePlatform::ReversePlatform, MaxDelay, false); // таймер задержки
}
```

`ReverseTimerPlatform` - переменная, хранящая Timer (проще - уникальный идентификатор для TimerManager)

`MaxDelay` - время, через которое вызавится передаваемый метод

`false` - флаг на зацикливание timer
# Debug

## Вывод на консоль

Вывод на консоль с параметрами

```c++
int intVar = 5;
float floatVar = 3.7f;
FString fstringVar = "an fstring variable";
UE_LOG(LogTemp, Warning, TEXT("Text, %d %f %s"), intVar, floatVar, *fstringVar);
```

`%d` - decimal number

`%f` - float number

`%s` - string number

`LogTemp` - категория, куда выводится

`Warning` - тип вывода, как выводится

`TEXT()` - макрос, хранящий текст

## Вывод на экран

Вывод на экран с параметрами

```c++
GEngine->AddOnScreenDebugMessage(1, 1.0f, FColor::Yellow, FString::Printf(TEXT("Stamina: %.2f"), CurrentStamina));
```

`1` - ключ вывода информации, указывающий приоритет вывода. Чем меньше, тем выше приоритет и выводится выше. Если ключи совпадают, то выводится последенее [???]

`1.0f` - время удержания информации

`FString::Printf(TEXT(), args..)` - создает строку с параметрами

## Собственное логирование

В файле .h с названием проекта (FirstProject.h) создается своя категория логирования, с праметрами

```c++
DECLARE_LOG_CATEGORY_EXTERN(LogCameras, Log, All)
```

В файле .cpp с навзанием проекта (FirstProject.cpp) объявляется эта категория логирования

```c++
DEFINE_LOG_CATEGORY(LogCameras)
```

Потом, эту категорию можно применять
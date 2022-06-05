# Основные функции UE

### Функция выводит на экран сообщение в редакторе

```c++
GEngine->AddOnScreenDebugMessage(-1, 1.0f, FColor::Green, TEXT("Capsule hit!"));
```

`-1` -уникальный ключ для предотвращения повторного добавления одного и того же сообщения

`1.0f` - количество секунд отображения

----

### Функция создания объекта

```c++
CreateDefaultSubobject<USphereComponent>(TEXT("Collision"));
```

`<...>` - в скобках указывается тип объекта

`(FName SubobjectName)` - в скобках указывается название компонента

----

### Функция создающая custom компонент на базовом классе

```c++
CreateDefaultSubobject<UPawnMovementComponent, UGCBasePawnMovementComponent>(TEXT("Movement component"));
```

`UPawnMovementComponent` - базовый класс

`UGCBasePawnMovementComponent` - custom компонент

----

## Вывод в Log

```c++
UE_LOG(LogTemp, Log, TEXT("AGameCodeBasePawn::OnBlendComplete() Current view actor: %s"), *CurrentViewActor->GetName());
```

`LogTemp` - категория, куда сообщение будет высылаться. Можно создать свой собственный

`Log` - уровень сообщения (Log, Error, Warning)

`TEXT` - сообщение

`%s` - подставляющий параметр типа string. Таких может быть сколько-угодно

`CurrentViewActor->GetName()` - название компонента

### Создание своей категории

```c++
DECLARE_LOG_CATEGORY_EXTERN(LogCameras, Log, All)
```

`LogCameras` - навзание категории

`Log` - уровень сообщения (Fatal, Error, Warning, Display, Log)

`All` - фильтрация сообщений. [Какие есть другие незнаю]

----


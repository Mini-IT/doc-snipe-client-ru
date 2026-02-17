#miniit #snipe #snipe-api
# Snipe 9 - инструкция по переводу проекта

## Обновить пакеты
Они совместимы с версиями 7-9
- config 4.2.0
- analytics.basic 4.3.0
- analytics.amplitude 5.0.1

## Обновить SnipeTools
SnipeTools 1.9.0

Будут ошибки компиляции из-за строго кода в проекте. Нужно перезапустить Unity. Ошибки никуда не пропадут, но SnipeTools обновится

## Скачать новый SnipeApi
`Snipe` -> `Download SnipeApi ...`

убедиться, что возле кнопки Download указана 9 версия снайпа
```
Snipe API Service Version: 9 (local source generator)
```

## Правки кода

Основное отличие - в том, что `ISnipeManager` теперь является единой "точкой входа" для работы со снайпом. Больше никаких отдельных синглтонов и статических классов - всё через `ISnipeManager`, всё в соответствии с DI паттернами.

## DI регистрация

```cs
using MiniIT.Snipe;  
using MiniIT.Snipe.Unity;

builder.RegisterSingleton<ISnipeManager>(() => new SnipeManager(new UnitySnipeServicesFactory()));
```

## Конфигурирование

### SnipeOptions

`SnipeConfig` переименован в `SnipeOptions` (`SnipeConfigBuilder` - в `SnipeOptionBuilder`), чтобы избежать путаницы с `RemoteConfig` и привести нейминг к общепринятым стандартам

### TablesOptions

Аналогично `TablesConfig` переименован в `TablesOptions`. И теперь это не статический класс, а поле в менеджере.

пример:

```cs
_snipe.TablesOptions.Versioning = TablesOptions.VersionsResolution.ForceBuiltIn;
```

Обращаться к `_snipe.TableOptions` можно только после вызова `_snipe.Initialize(...)`
## Initialization

```cs
private readonly ISnipeManager _snipe;

var builder = new SnipeOptionsBuilder();
_appConfig.InitializeSnipeOptions(builder);

var contextFactory = new SnipeApiContextFactory(_snipe, builder);
var tablesFactory = new SnipeApiTablesFactory(_snipe.Services, builder);

_snipe.Initialize(contextFactory, tablesFactory);
```

## Пробросить ISnipeServices

`SnipeServices` больше не является статическим классом. Инстанс можно получить через `ISnipeManager.Services`.
Т.к. это больше не статический класс, то некоторым объектам нужно явно передать ссылку.

### В биндинги

```cs
snipeContext.Auth.RegisterBinding(new DeviceIdBinding(_snipe.Services));
snipeContext.Auth.RegisterBinding(new AdvertisingIdBinding(_snipe.Services));
snipeContext.Auth.RegisterBinding(new FacebookBinding(_snipe.Services));
snipeContext.Auth.RegisterBinding(new AmazonBinding(_snipe.Services));
```

### В Аналитику !!!!!! ВАЖНО !!!!!!!!
Раньше пакет аналитики сам мог подцепиться к статическому классу. Теперь нужно указать треккер вручную:
```cs
public class AnalyticsService : BasicAnalyticsTracker
{
  public AnalyticsService(ISnipeManager snipe,...)
  {
    //...
    snipe.Services.Analytics.SetTracker(this);
    //...
  }
}
```

### В конфиг

```cs
public class AppConfig : Config
{
  public AppConfig(ISnipeManager snipe)
    : base(new SnipeConfigProvider(PROJECT_ID, snipe.Services), TimeSpan.FromSeconds(15)) { }
  // ...
}
```

## SnipeApi
Сокращённые названия событий переименованы на полные (см. [Changes v.9](Changes%20v.9.md)). При необходимости - поправить обращения в коде проекта

## ProfileManager

`ProfileManager` теперь изолирован и создаётся через собственный конструктор:
```cs
var profileManager = new ProfileManager(context.GetApi(), context.Communicator, context.Auth, _snipe.Services.SharedPrefs);
```

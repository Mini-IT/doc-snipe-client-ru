#miniit #snipe #snipe-api
# Snipe 9 - инструкция по переводу проекта

Основное отличие - в том, что `ISnipeManager` теперь является единой "точкой входа" для работы со снайпом. Больше никаких отдельных синглтонов и статических классов - всё через `ISnipeManager`, всё в соответствии с DI паттернами.

## DI регистрации

```cs
builder.RegisterSingleton<ISnipeManager>(() => new SnipeManager(new UnitySnipeServicesFactory()));
```

## Конфигурирование

### SnipeOptions

`SnipeConfig` переименован в `SnipeOptions`, чтобы избежать путаницы с `RemoteConfig` и привести нейминг к общепринятым стандартам
`SnipeConfigBuilder` - в `SnipeOptionBuilder`

### TablesOptions

Аналогично `TablesConfig` переименован в `TablesOptions`. И теперь это не статический класс, а поле в менеджере.

пример:

```cs
_snipe.TablesOptions.Versioning = TablesOptions.VersionsResolution.ForceBuiltIn;
```

## Initialization

```cs
private readonly ISnipeManager _snipe;

var builder = new SnipeOptionsBuilder();
var contextFactory = new SnipeApiContextFactory(_snipe, builder);
var tablesFactory = new SnipeApiTablesFactory(_snipe.Services, builder);

_appConfig.InitializeSnipeOptions(builder);
_snipe.Initialize(contextFactory, tablesFactory);
```

## Пробросить ISnipeServices

`SnipeServices` больше не является синглтоном. Инстанс можно получить через `ISnipeManager.Services`.
Т.к. это больше не синглтон, то некоторым объектам нужно явно передать ссылку.  

### В биндинги

```cs
snipeContext.Auth.RegisterBinding(new DeviceIdBinding(_snipe.Services));
snipeContext.Auth.RegisterBinding(new AdvertisingIdBinding(_snipe.Services));
snipeContext.Auth.RegisterBinding(new FacebookBinding(_snipe.Services));
snipeContext.Auth.RegisterBinding(new AmazonBinding(_snipe.Services));
```

### В Аналитику !!!!!! ВАЖНО !!!!!!!!

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
```

## SnipeApi

Кастомные сокращения вырезаны - названия теперь формируются по единым стандартам

было
```
api.Chat.Msg ...
api.Clan.Msg ...
api.Room.Left ...
api.Room.Dead ...
// api.Room.Join // даже не существовало
// api.Room.Broadcast // даже не существовало
```

стало
```
api.Chat.OnChatMsg ...
api.Clan.OnClanMsg ...
api.Room.OnRoomJoin ...
api.Room.OnRoomLeft ...
api.Room.OnRoomDead ...
api.Room.OnRoomBroadcast ...
```

## Tables.Load Cancellation

в Tables.Load() теперь можно передать CancellationToken и прервать операцию

## ProfileManager

`ProfileManager` всё ещё является частью пакета, но теперь он полностью изолирован. Поэтому треубется явно прокинуть ссылки в конструкторе:
```cs
var profileManager = new ProfileManager(context.GetApi(), context.Communicator, context.Auth, _snipe.Services.SharedPrefs);
```

## AuthBinding.AvailableForRegistration

Можно создавать кастомные биндинги, которые будут автоматически добавляться в запрос `registerAndLogin`, если у них свойство `AvailableForRegistration == true`

------

## SnipeTools

- SnipeApi v9
- Preload tables - автоматически создаёт папку StreamingAssets. Раньше был фейл, если её не существовало.

---------

## Packages affected

config
analytics.basic
analytics.amplitude

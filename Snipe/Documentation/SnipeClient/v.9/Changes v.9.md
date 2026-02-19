#miniit #snipe 

Основное отличие - в том, что `ISnipeManager` теперь является единой "точкой входа" для работы со снайпом. Больше никаких отдельных синглтонов и статических классов - всё через `ISnipeManager`, всё в соответствии с DI паттернами.
`SnipeServices` больше не является статическим классом. Инстанс можно получить через `ISnipeManager.Services`.
Т.к. это больше не статический класс, то некоторым объектам нужно явно передать ссылку.

### SnipeOptions и TablesOptions

`SnipeConfig` переименован в `SnipeOptions` (`SnipeConfigBuilder` - в `SnipeOptionBuilder`), чтобы избежать путаницы с `RemoteConfig` и привести нейминг к общепринятым стандартам

Аналогично `TablesConfig` переименован в `TablesOptions`. И теперь это не статический класс, а поле в менеджере.

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

в `Tables.Load()` теперь можно передать CancellationToken и прервать операцию

## ProfileManager

`ProfileManager` изолирован и вынесен в отдельный пакет `unity-snipe-profilemanager`. Поэтому требуется явно прокинуть ссылки в конструкторе:
```cs
var profileManager = new ProfileManager(context.GetApi(), context.Communicator, context.Auth, _snipe.Services.SharedPrefs);
```

## AuthBinding.AvailableForRegistration

Можно создавать кастомные биндинги, которые будут автоматически добавляться в запрос `registerAndLogin`, если у них свойство `AvailableForRegistration == true`

------

## SnipeTools 1.9.0

- SnipeApi v9
- Preload tables - автоматически создаёт папку StreamingAssets. Раньше падал эксепшен, если её не существовало.

---------

## Packages affected

- config 4.2.0
- analytics.basic 4.3.0
- analytics.amplitude 5.0.1


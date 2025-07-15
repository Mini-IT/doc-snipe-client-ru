- обновить пакет `SnipeTools` . Если есть compilation errors, перезапустить Unity, чтобы обновились скрипты
- обновить пакет `SnipeClient`
- обновить пакет `framework.config`
- обновить пакет `framework.valuesyncer`
- обновить пакет `framework.iap.serververification` `3.1.0`
- при необходимости, обновить пакеты аналитики и рекламы
-  `using MiniIT.Framework.Configuration.ConfigProvider;` заменить на
  `using MiniIT.Framework.Configuration.Providers;`

- если в проекте используются `ISnipeContextHolder`, `ISnipeContextMaster`, `class ServerContext : ISnipeContextMaster`, то вырезаем их.
  > [!warning]
  > Это проектный код и тут могут использоваться какая-то специфическая логика инициализации и заполнения конфига - не потеряйте её
  
- Регистрируем `SnipeManager` в DI `AppRootContainer`
```cs
	using MiniIT.Snipe;
	
	builder.RegisterSingleton<ISnipeManager, SnipeManager>()
	    .As<ISnipeContextProvider>()
	    .As<ISnipeTablesProvider>();
```
- `snipeContext.Api.XXX` заменяем на `snipeContext.GetApi().XXX`
- таблицы отделены от контекста, обращаться к ним теперь нужно через `SnipeManager`:
```cs
	using MiniIT.Snipe;     // interface ISnipeManager
	using MiniIT.Snipe.Api; // Extension method ISnipeManager.GetTables();
	
	ISnipeManager snipe;
	snipe.GetTables();
```
- использование в коде интерфейса `ISnipeContextHolder` можно заменить на `ISnipeManager` (или на более низкоуровневые `ISnipeContextProvider`, `ISnipeTablesProvider`, в зависимости от конкретного сценария использования, но это будет менее удобно)
- к `SnipeConfig` теперь нельзя обращаться из проектного кода - он задается через фабрику перед коннектом
	- к полю `Config.ProjectName` можно обратиться через контекст `context.ProjectName`
	```cs
	// было
	if (_snipe.Context != null)
	{
	    return (_snipe.Context.Config.ProjectName) + ":" + _snipe.Context.Auth.UserID;
	}

	// стало
	if (_snipe.TryGetContext(out var context))
	{
	    return (context.ProjectName) + ":" + context.Auth.UserID;
	}
	```
- Facebook JSON blob
	- было: ~~`string json = SnipeObject.ConvertToJSONString(FacebookProvider.Instance.PlayerProfile);`~~
	- стало: `string json = JsonUtility.ToJson(FacebookProvider.Instance.PlayerProfile.ToObject());`
- OnLogin
	- было: 
	```cs
	server.LoggedIn += SetUserId;
	```
	- стало:
	```cs
	snipe.GetContextWhenReady(context =>
	{  
		context.Auth.LoginSucceeded += uid => SetUserId(uid.ToString());
	});
	```
- обратите внимание (в продолжение предыдущего пункта), в OnLoginSucceeded добавился параметр `int userId`:
```cs
	// Было
	context.Auth.LoginSucceeded += () =>
	{
	    SetUserId(context.Auth.UserID.ToString());
	};
	
	// стало
	context.Auth.LoginSucceeded += (userId) =>
	{
	    SetUserId(userId.ToString());
	};
```
- Инициализация
```cs
	// было
	_snipe.InitializeSnipeContext(_appConfig);
	
	// стало
	var snipeConfigBuilder = new SnipeConfigBuilder();
	// инициализация данными из внешнего конфига
	_appConfig.InitializeSnipeConfig(snipeConfigBuilder);
	// или вместо предыдущей строки можно зафорсить дев
	// snipeConfigBuilder.InitializeDefault(new SnipeProjectInfo()  
	// {  
	//  Mode = SnipeProjectMode.Dev,  
	//  ProjectID = "study1",  
	//  ClientKey = "client-XXXXXXXX"  
	// });
	var factory = new SnipeApiContextFactory(_snipe, snipeConfigBuilder);
	_snipe.Initialize(factory);
```
- `TablesLoadingSegment`
```cs
	private readonly ISnipeManager _snipe;
	
	public override async UniTask Init()  
	{  
	    // await UniTask.WaitUntil(() => _snipe.Initialized);
	    await UniTask.WaitUntil(() => _snipe.TryGetContext(out _));
	    await _snipe.GetTables().Load();
	}
```

### События коннекта
Для устранения неоднозначностей, события `SnipeCommunicator` реорганизованы
```cs
// Соединение установлено
ConnectionEstablished;  
  
// Соединение полностью потеряно. Автоматических попыток реконнекта больше не будет
ConnectionClosed;  
  
// Соединение потеряно или его не удалось установить. Если не закончились попытки, то будет автореконнект
ConnectionDisrupted;  
  
// Автореконнект инициирован. После кулдауна будет попытка восстановить соединение
ReconnectionScheduled;
```

### Bindings
Инициализация биндингов переделана. См. Подробнее тут: [Bindings](Bindings.md)

### Подмена конфига
иногда требуется подменить конфиг и переконнектиться.
```cs
var snipeConfigBuilder = new SnipeConfigBuilder();
// инициализация данными из внешнего конфига
_appConfig.InitializeSnipeConfig(snipeConfigBuilder);
var factory = new SnipeApiContextFactory(_snipe, snipeConfigBuilder);
// применить новый конфиг
factory.Reconfigure(context);
```
При этом не пересоздаётся ни контекст, ни коммуникатор, ни таблицы. Поэтому заново подписываться на события не требуется. Старые ValueSyncer-ы тоже должны работать - пересоздавать не нужно.


### Troubleshooting
Если видите одну из этих ошибок
```
error CS1061: 'ISnipeManager' does not contain a definition for 'GetTables' ...
error CS1061: 'ISnipeManager' does not contain a definition for 'GetApi' ...
```
просто добавьте вначале файла
```cs
using MiniIT.Snipe.Api;
```

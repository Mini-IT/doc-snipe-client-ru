- обновить пакет `SnipeTools` . Если есть compilation errors, перезапустить Unity, чтобы обновились скрипты
- обновить пакет `SnipeClient`
- обновить пакет `framework.config`
- обновить пакет `framework.valuesyncer`
-  `using MiniIT.Framework.Configuration.ConfigProvider;` заменить на
  `using MiniIT.Framework.Configuration.Providers;`
- если в проекте используются `ISnipeContextHolder`, `ISnipeContextMaster`, `class ServerContext : ISnipeContextMaster`, то вырезаем их. (Это проектный код и тут могут использоваться какая-то специфическая логика инициализации и заполнения конфига - не потеряйте ее)
- Регистрируем в DI `AppRootContainer`
```cs
using MiniIT.Snipe;

builder.Register(() => SnipeManager.Instance)  
    .As<ISnipeManager>()  
    .As<ISnipeContextProvider>()  
    .As<ISnipeTablesProvider>();
```
- `snipeContext.Api.XXX` заменяем на `snipeContext.GetApi().XXX`
- таблицы отделены от контекста, но через extention сделан вспомогательный метод `context.GetTables()`. Либо обращаться напрямую к `SnipeManager`-у:
```cs
ISnipeManager snipe;

// Вариант 1. Вернёт нетипизированные таблицы
snipe.GetTables();
// Вариант 2. Расширение, которое вернёт типизированные таблицы данного проекта
snipe.GetProjectTables();

// Вариант 2. Расширение контекста, которое вернёт типизированные таблицы данного проекта
if (snipe.TryGetContext(out var context))
{
	context.GetTables(); // то же самое, что snipe.GetProjectTables();
}
```
- использование в коде интерфейса `ISnipeContextHolder` можно заменить на `ISnipeContextProvider` или на более высокоуровневый `ISnipeManager`
- к `SnipeConfig` теперь нельзя обращаться из проектного кода - он задается через фабрику перед коннектом
	- к полю `Config.ProjectName` можно обратиться через контекст `context.ProjectName`
- Facebook JSON blob
	- было: ~~`string json = SnipeObject.ConvertToJSONString(FacebookProvider.Instance.PlayerProfile);`~~
	- стало: `string json = JsonUtility.ToJson(FacebookProvider.Instance.PlayerProfile.ToObject());`
- OnLogin
	- было: `server.LoggedIn += SetUserId;`
	- стало:
		```cs
		snipe.GetContextWhenReady(context =>  
		{  
		    context.Auth.LoginSucceeded += uid => SetUserId(uid.ToString());  
		});
		```
- Инициализация
```cs
var factory = new SnipeApiContextFactory(snipeConfigBuilder);  
_snipe.Initialize(factory, factory);
```
- `TablesLoadingSegment`
```cs
private readonly ISnipeManager _snipe;

public override async UniTask Init()  
{  
    await UniTask.WaitUntil(() => _snipe.Initialized);  
    await _snipe.GetTables().Load();  
}
```


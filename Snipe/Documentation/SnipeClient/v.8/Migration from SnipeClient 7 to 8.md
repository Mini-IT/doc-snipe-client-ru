- обновить пакет `SnipeTools` . Если есть compilation errors, перезапустить Unity, чтобы обновились скрипты
- обновить пакет `SnipeClient`
- обновить пакет `framework.config`
- обновить пакет `framework.valuesyncer` `1.4.0`
- обновить пакет `framework.iap.serververification` `3.1.0`
- при необходимости, обновить пакеты аналитики и рекламы
-  `using MiniIT.Framework.Configuration.ConfigProvider;` заменить на
  `using MiniIT.Framework.Configuration.Providers;`

- если в проекте используются `ISnipeContextHolder`, `ISnipeContextMaster`, `class ServerContext : ISnipeContextMaster`, то вырезаем их.
> [!warning]
  >  Это проектный код и тут могут использоваться какая-то специфическая логика инициализации и заполнения конфига - не потеряйте её
  
- Регистрируем `SnipeManager` в DI `AppRootContainer`
```cs
	using MiniIT.Snipe;
	
	builder.RegisterSingleton<ISnipeManager, SnipeManager>()
	    .As<ISnipeContextProvider>() // optional
	    .As<ISnipeTablesProvider>(); // optional
```
- `snipeContext.Api.XXX` заменяем на `snipeContext.GetApi().XXX`
- таблицы отделены от контекста, обращаться к ним теперь нужно через `SnipeManager`:
```cs
	using MiniIT.Snipe;     // тут объявлен interface ISnipeManager
	using MiniIT.Snipe.Api; // тут раширение, реализующее метод ISnipeManager.GetTables();
	
	ISnipeManager snipe;
	snipe.GetTables();
```
- использование в коде интерфейса `ISnipeContextHolder` можно заменить на `ISnipeManager` (или на более низкоуровневые `ISnipeContextProvider`, `ISnipeTablesProvider`, в зависимости от конкретного сценария использования, но это будет менее удобно)
- к `SnipeConfig` теперь нельзя обращаться из проектного кода - он задается через фабрику при инициализации
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
  ```cs
	// было
	string json = SnipeObject.ConvertToJSONString(FacebookProvider.Instance.PlayerProfile);

	// стало
	string json = JsonUtility.ToJson(FacebookProvider.Instance.PlayerProfile.ToObject());
	```
- OnLogin
	```cs
	// было 
	server.LoggedIn += SetUserId;
	
	// стало
	snipe.GetContextWhenReady(context =>
	{  
		context.Auth.LoginSucceeded += uid => SetUserId(uid.ToString());
	});
	```
- обратите внимание (в продолжение предыдущего пункта), в `OnLoginSucceeded` добавился параметр `int userId`:
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
Основная статья: [Connection Events Flow](Connection%20Events%20Flow.md)
Для устранения неоднозначностей, события `SnipeCommunicator` реорганизованы.
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

### Загрузка внешнего конфига
>[!WARNING]
> Важно!

Загрузка конфига (т.е. инициализация `_appConfig` из примера выше) не начнётся пока не будет вызван `SnipeServices.Initialize`. Если не вызывать его вручную, то он автоматически вызовется со стандартными параметрами при первой попытке создать контекст. Поэтому, если задавать параметры соединения вручную через `snipeConfigBuilder.InitializeDefault()`, то вызывать `SnipeServices.Initialize` нет необходимости. Однако на практике часто требуется сначала загрузить конфиг, чтобы проинициализировать SnipeManager и только потом создавать контексты. Для этого нужно заранее проинициализировать SnipeServices. Например так:
```csharp
public class ServerService
{
    public ServerService(...)
    {
        // ...

        InitializeSnipeContext();
    }

    private void InitializeSnipeContext()
    {
        if (!SnipeServices.IsInitialized)
        {
            SnipeServices.Initialize(new UnitySnipeServicesFactory());
        }

        _contextCancellation = new CancellationTokenSource();
        WaitConfigAndStart(_contextCancellation.Token).Forget();
    }

    private async UniTaskVoid WaitConfigAndStart(CancellationToken cancellationToken)
    {
        await UniTask.WaitUntil(() => _appConfig.Initialized, cancellationToken: cancellationToken)
            .SuppressCancellationThrow();

    //...
```

### Подмена конфига
иногда требуется подменить конфиг и переконнектиться.
```cs
// создаем новый билдер конфига
var snipeConfigBuilder = new SnipeConfigBuilder();
// инициализируем билдер данными из внешнего конфига
_appConfig.InitializeSnipeConfig(snipeConfigBuilder);
// создаем новую фабрику
var factory = new SnipeApiContextFactory(_snipe, snipeConfigBuilder);
// примененяем новый конфиг к старому контексту
factory.Reconfigure(context);
```
При этом не пересоздаётся ни контекст, ни коммуникатор, ни таблицы. Поэтому заново подписываться на события не требуется. Старые ValueSyncer-ы тоже должны работать - пересоздавать не нужно.

>[!NOTE] 
> На текущее соединение изменения не повлияют. Это только для того, чтобы переконнетиться с другими параметрами.


### Troubleshooting
Если видите одну из этих ошибок
```
error CS1061: 'ISnipeManager' does not contain a definition for 'GetTables' ...
error CS1061: 'ISnipeManager' does not contain a definition for 'GetApi' ...
```
просто добавьте вначале файла
```cs
using MiniIT.Snipe.Api; // тут объявлены расширения с методами GetApi и GetTables
```

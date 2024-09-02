#miniit #snipe 

>[!info]
>Настоятельно рекомендуется сначала прочитать [Общие заметки](Общие%20заметки.md)

### Инициализаця

Пример

```csharp
public class ServerService
{
	//...
	private readonly AppConfig _appConfig;
	
	private readonly SnipeApiContext _snipeContext;
	public SnipeApiContext Snipe => _snipeContext;
	
	private CancellationTokenSource _cancellationTokenSource;

	public ServerService(AppConfig appConfig,...)
	{
		_appConfig = appConfig;
		//...

		_snipeContext = SnipeApiContext.Default;
		
		//...
		_cancellationTokenSource = new CancellationTokenSource();
		StartServer(_cancellationTokenSource.Token);
	}
	
	private async void StartServer(CancellationToken cancellationToken)
	{
		while (!_appConfig.Initialized)
		{
			await Task.Delay(100, cancellationToken);
		}
		if (cancellationToken.IsCancellationRequested)
		{
			return;
		}

		_snipeContext.Config.Init(_appConfig.GetSnipeConfig());

		_snipeContext.Communicator.ConnectionSucceeded += OnConnectionSucceeded;
		_snipeContext.Communicator.ConnectionFailed += OnConnectionFailed;
		_snipeContext.Auth.LoginRequested += OnLoginRequested;
		_snipeContext.Auth.LoginSucceeded += OnLoginSucceeded;
		_snipeContext.Auth.AutomaticallyBindCollisions = true;

		_snipeContext.Communicator.StartCommunicator();
	}
	
	//...
}
```

### SnipeConfig

`SnipeConfig` перемещен в `SnipeContext`:

```csharp
// было
~~SnipeConfig.Init(...);~~
// стало
_snipeContext.Config.Init(...);
```

### SnipeApi

`SnipeApi` заменен на `SnipeApiService` и помещен в `SnipeApiContext`.

Нужно скачать `SnipeApiService`.

В проектах с DI рекомендуется удалить старый `SnipeApi.cs` и использовать DI для передачи единой ссылки на инстанс `ServerService`, через свойство которого осуществляется доступ к инстансу `SnipeApiContext`.

В проектах без DI вместо старого `SnipeApi` будет сгенерирован одноименный класс-обертка, который позволит обращаться к Api по-старому, за счет использования `SnipeApiContext.Default`.

подробнее в доке [Общие заметки](Общие%20заметки.md) 

### Старый SnipeApi

Если в проекте имеется файл `SnipeApi.cs`, то при скачивании нового файла, на его месте будет сгенерирован класс-обертка с тем же названием `SnipeApi`, в котором, помимо старых полей, добавлены свойства:

```csharp
SnipeApi.Context; // instance of SnipeApiContext
SnipeApi.Service; // instance of SnipeApiService
```

Инициализация больше не требуется:

```csharp
// было
~~SnipeApi.Initialize();~~
```

### Таблицы

Доступ к таблицам теперь также осуществляестя через контекст:

```csharp
var context = SnipeApiContext.Default;
if (!context.Tables.Loaded)
{
	context.Tables.Load();
}
```

### Persistent

<aside>
💡 Необходимо использовать `Persistent` классы из пэкэджа версии не ниже 1.0.1. Старый код будет кидать эксепшены

</aside>

<aside>
💡 Текущая реализация `Persistent` не поддреживает одновременного использования более одного инстанса `SnipeContext`.

</aside>

Пример инициализации

```csharp
public class AppRootContainer : RootContainer
{
	//.. 

	protected override void Resolve()
	{
		//..
		var analyticsTracker = DIContainer.Resolve<AnalyticsTracker>();
		var serverService = DIContainer.Resolve<ServerService>();
		
		_ = PersistentValuesService.CreateWithSnipeApiService(serverService.Snipe.Api, analyticsTracker);
	}
}
```

### SnipeCommunicator.Instance

Так как теперь может одновременно быть больше одного активного коммуникатора, то дефолтной ссылки на единый инстанс больше нет. Вместо этого к нему нужно обращаться через `SnipeContext`:

```csharp
// было
~~SnipeCommunicator.Instance.StartCommunicator();~~
// стало
_snipeContext.Communicator.Start();
// или для конкретного контекста
SnipeApiContext.GetInstance("TicTacToe").Communicator.Start();
```

Также `SnipeCommunicator.Instance.Auth` теперь тоже находится в контексте:

```csharp
_snipeContext.Auth.AutomaticallyBindCollisions = true;
_snipeContext.Auth.LoginSucceeded += OnLoginSucceeded;
```

### Dispose

```csharp
// было
~~SnipeCommunicator.DestroyInstance();~~
// стало
_snipeContext.Dispose();
```

### Получение существующего инстанса

Часто встречаются задачи, когда что-то нужно сделать, только если инстанс существует, а не создавать его.

```csharp
// было
~~if (SnipeCommunicator.InstanceInitialized)~~
// стало
var instance = SnipeContext.GetInstance(initialize: false);
// var instance = SnipeContext.GetInstance(null, false);
// var instance = SnipeContext.GetInstance("Player1", false);
if (instance != null)
```

### LogReporter

`LogReporter` перемещен в `SnipeContext`, поэтому иницаиализация больеше не требуется:

```csharp
// было
~~MiniIT.LogReporter.InitInstance();~~
```

Отправка логов теперь вызывается через `SnipeContext`:

```csharp
// было
~~MiniIT.LogReporter.SendAsync();~~
// стало
var snipeContext = MiniIT.Snipe.SnipeContext.GetInstance(initialize: false);
snipeContext?.LogReporter.SendAsync();
```

Кроме того, `LogReporter` теперь динамически вычисляет размер порции данных для отправки. Раньше лог разбивался на равное количество записей, но т.к. размер записей мог очень сильно разниться, то иногда он не помещался в лимит. Теперь же записи лога добавляются в порцию до тех пор, пока размер не превысит 200К символов.

### Привязка аккаунтов

В v.4 автоматически добавлялись привязки для аккаунтов ….

Чтобы их выключить, нужно было объявить константу компилятора

```csharp
// было
~~#define BINDING_DISABLED~~
```

В v.5 для этого вынесено свойство `UseDefaultBindings`. По умолчанию значение `true`.

```csharp
// запретим дефолтные привязки
_snipeContext.Auth.UseDefaultBindings = false;
// будут использованы только заданые явно
_ = _snipeContext.Auth.GetBinding<FacebookBinding>();
```

### Аналитика

Интерфейс `MiniIT.Snipe.IAnalyticsTracker` переименован в `MiniIT.Snipe.ISnipeCommunicatorAnalyticsTracker`.
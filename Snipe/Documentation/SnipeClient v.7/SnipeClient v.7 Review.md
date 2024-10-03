#miniit  #snipe 
# Changelog v 7.0
- `AlterTask` (для поддержки однопоточных тасков в WebGL)
- UWP system info extractor (для отправки в adjust через сервер)
- `MessagePack` exception added - для проброса в аналитику ошибок парсинга сообщений от сервера
- Semaphore exception fixed
- в `AppInfo` передается название платформы с суффиксом `Amazon/RuStore/Nutaku`
- Везде явно прописана `InvariantCulture` - больше нет строгой необходимости в коде проекта прописывать
  `System.Threading.Thread.CurrentThread.CurrentCulture = CultureInfo.InvariantCulture;`
- исправлена поддержка `ZString` в сервисе отправки логов на сервер
- возможность [управлять батчингом attr.set](Batching%20attr.set.md)
- [подтверждение получения сообщений от сервера](https://app.asana.com/0/1185654335370062/1208400518366715/f)
- копия логов для отправки в графану больше не копится в памяти, а пишется в фоне во временный файл
- рефакторинг отдельных классов и разные багфиксы
- переделан `SnipeContext`:
	- избавились от рефлексии при создании контекста
	- избавились от торчащих наружу синглтонов. `SnipeContext.Default` больше не существует, поэтому требуется переделать код, который к нему обращался
	- `id` контекста теперь `int` (был string)

## Повлияло на следующие пакеты
- `com.miniit.altertask` - новый пакет
- `com.miniit.framework.config` - требуется версия 3.4.0
- `com.miniit.framework.iap` - требуется версия 2.0.0
- `com.miniit.framework.iap-serververification` - требуется версия 3.0.0
- `Unity-Log-Viewer` - см. ниже
- `UWPNotificationsService` (CYSI) - нужно переделать обращения к `SnipeContext.Default`
- `SnipeTools` 1.6.0

## SnipeApi generator changes (server side)
1. заменить
`using System.Threading.Tasks;`
на
`using MiniIT.Threading;`

2. заменить все обращения к классу `Task` на `AlterTask` (просто заменой по целому слову)

constructor - поменялся второй аргумент
```csharp
public SnipeApiService(SnipeCommunicator communicator, AuthSubsystem auth)
	: base(communicator, auth)
{
	// тут всё без изменений
}
```

```csharp
public class SnipeApiContext : BaseSnipeApiContext
{
	public SnipeApiService Api { get; }
	public SnipeTables Tables { get; }

	public SnipeApiContext(int id, SnipeConfig config, SnipeCommunicator communicator, AuthSubsystem auth, LogReporter logReporter, SnipeApiService api, SnipeTables tables, TimeZoneInfo serverTimeZone)
		: base(id, config, communicator, auth, logReporter, api, tables, serverTimeZone)
	{
		Api = api;
		Tables = tables;
	}
}

public class SnipeApiContextFactory : AbstractSnipeApiContextFactory
{
	protected override SnipeContext InternalCreateContext(int id, SnipeConfig config, SnipeCommunicator communicator, AuthSubsystem auth, LogReporter logReporter)
	{
		var api = new SnipeApiService(communicator, auth);
		var tables = new SnipeTables();
		var serverTimeZone = TimeZoneInfo.CreateCustomTimeZone("server time", TimeSpan.FromHours(3), "server time", "server time");
		return new SnipeApiContext(id, config, communicator, auth, logReporter, api, tables, serverTimeZone);
	}
}
```

## Правки кода
В коде проекта вместо обращения к `SnipeContext.Default`, создаем инстанс контекста:
```csharp
_snipeContext = new SnipeApiContextFactory().CreateContext(0) as SnipeApiContext;
```

и обращаемся везде уже к нашему инстансу
### interface ISnipeContextReference
В пэкэдже объявлен интерфейс `MiniIT.Snipe.ISnipeContextReference`, который используется вместо старых
`SnipeContext.Default` и `SnipeContext.GetInstance`.
Его нужно реализовать для того, чтобы код в пэкэджах мог достучаться до снайпа. Требуется не во всех проектах.

#### Старые проекты без DI
Например CYSI, StG
```csharp
public class Server : MonoSingleton<Server>, ISnipeContextReference
{
	public SnipeApiContext Snipe => _snipeContext ??= new SnipeApiContextFactory().CreateContext(0) as SnipeApiContext;
	private SnipeApiContext _snipeContext;

	public bool TryGetSnipeContext(out SnipeContext context)
	{
	    context = _snipeContext;
	    return context != null;
	}
	
	//...
}

class AnyOtherClass
{
	private void SomeMethod()
	{
		var snipe = Server.Instance.Snipe;
		var progress = snipe.Api.UserAttr.UserAttributes.dailyProgress.GetValue();
	}
}
```

#### Проекты с DI + ISnipeContextMaster, ISnipeContextHolder
Наследуем и реализуем `ISnipeContextReference` и меняем строчку создания инстанса SnipeContext:
```csharp
public interface ISnipeContextHolder : ISnipeContextReference
{
//...
}

public class ServerContext : ISnipeContextMaster
{
	public SnipeApiContext Context
	{
		get
		{
			if (_snipeContext == null || _snipeContext.IsDisposed)
			{
				_initialized = false;
				_snipeContext = new SnipeApiContextFactory().CreateContext(0) as SnipeApiContext;
			}
			return _snipeContext;
		}
	}
	
	public bool TryGetSnipeContext(out SnipeContext context)
	{
		if (TryGetContext(out var apicontext))
		{
			context = apicontext;
			return true;
		}
	
		context = null;
		return false;
	}
	//...
```

#### Проекты с DI без ISnipeContextHolder
Реализуем в зависимости от структуры проекта

## Unity Log Viewer / Reporter
В LogViewer-е был добавлен костыль, который отправляет лог на сервер в графану. Этот костыль использовал обращение к синглтону `SnipeContext.Default`, которого больше не существует.
```csharp
private void SaveLogsToDevice()
{
	//...
	
	MiniIT.Snipe.SnipeContext.GetInstance(null, false)?.LogReporter.SendAsync();
}
```
Поэтому нужно реализовать инверсию управления. Просто создаем событие и слушаем его в проектном сервисе:
```csharp
public event Action LogSaved;

private void SaveLogsToDevice()
{
	//...
	
	LogSaved?.Invoke();
}
```
```csharp

public class ServerService
{
	private async UniTaskVoid WaitConfigAndStart(CancellationToken cancellationToken)
	{
		//...

		var reporter = UnityEngine.Object.FindObjectOfType<Reporter>();
		if (reporter != null)
		{
			reporter.LogSaved -= OnReporterLogSaved;
			reporter.LogSaved += OnReporterLogSaved;
		}
	}
	
	private void OnReporterLogSaved()
	{
		_ = _snipeContext.LogReporter.SendAsync();
	}
}
```

## Обращения из вьюшек
Некоторые монобехи (view-шки) тоже обращались к синглтону `SnipeContext.Default`, что является нарушением нашей MVC-подобной структуры кода. Это должно выполняться в контроллерах.
Пример из `color block` / `docked blocks`
```csharp
class OptionsPopupView : MonoBehaviour
{
	private void Start()
	{
		if (_userIdText != null)
		{
			SnipeContext snipeContext = SnipeContext.Default;
			_userIdText.text = (snipeContext.Auth.UserID != 0) ? $"ID: {snipeContext.Auth.UserID}" : "";
		}
	}
}
```
Переделываем следующим образом:
```csharp
class OptionsPopupView : MonoBehaviour
{
	public void SetUserID(string uid)
	{
		if (_userIdText != null)
		{
			_userIdText.text = uid;
		}
	}
}

class OptionsPopup
{
	// можно сохранить ссылку на весь ISnipeContextHolder,
	// но в данном примере это бессмысленно, т.к.
	// нам от него нужен только user id
	private readonly string _userID;
	
	public OptionsPopup(//...,
            ISnipeContextHolder snipe)
            : base()
	{
		//...
		
		int uid = snipe.Context.Auth.UserID;
		_userID = (uid != 0) ? $"ID: {uid}" : "";
	}

	public override void Init(IPopupView view, UIPopup popup, params object[] args)
	{
		//...
		
		optionsPopupView.SetUserID(_userID);
	}
}
```


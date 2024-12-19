#miniit #snipe 

## Пакеты
Требуется установить пакет `com.miniit.altertask` и обновить другие связанные пакеты. См. [SnipeClient v.7 Review](SnipeClient%20v.7%20Review.md)

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


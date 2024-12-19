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
- Детектор дисконнекта переделан на таймер, зависящий от FPS
- Добавлен запрос `auth.connect`, который можно использовать вместо биндинга для привязки одного FB аккаунта к нескольким устройствам
- В юнити редакторе при выходе из плеймода генерируется отчет о полученных от сервера сообщениях с errorCode != ok

## Повлияло на следующие пакеты
- `com.miniit.altertask` - новый пакет
- `com.miniit.framework.config` - требуется версия 3.4.0
- `com.miniit.framework.iap` - требуется версия 2.0.0
- `com.miniit.framework.iap-serververification` - требуется версия 3.0.0
- [`com.miniit.unity-web-cookies`](https://github.com/Mini-IT/unity-web-cookies) требуется для WebGL сборок
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

## Инструкция по обновлению
[SnipeClient 6 → 7](SnipeClient%206%20→%207.md)

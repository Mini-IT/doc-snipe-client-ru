# SnipeClient 6.0 → 6.1

добавить пакет `com.miniit.framework.utilities`
[`https://ghp_....@github.com/Mini-IT/unity-framework-utilities.git`](https://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@github.com/Mini-IT/unity-framework-utilities.git)

(надо бы его сделать публичным)

обновить пакет `SnipeTools` 1.4.0
[`https://github.com/Mini-IT/SnipeToolsUnityPackage.git`](https://github.com/Mini-IT/SnipeToolsUnityPackage.git#feature/config)

обновить пакет `SnipeClient` 6.1.0
[`https://github.com/Mini-IT/SnipeUnityPackage.git`](https://github.com/Mini-IT/SnipeToolsUnityPackage.git#feature/config)

## SnipeProjectInfo

В метод `snipeContext.Config.Initialize` первым параметром теперь нужно передать инстанс структуры `SnipeProjectInfo`.

> [!note]
> Далее следует инструкция к старой верcии пакета конфига.
> В новой версии реализован вспомогательный метод `InitializeSnipeConfig`, который позволит сделать всё то же самое более удобным образом. См. [[Перевод на Snipe Remote Config 3.2.0]]

предлагается в проектном коде в классе `AppConfig` добавить следующий код

```csharp
using MiniIT.Snipe;

public const string PROJECT_ID = "...";
private const string CLIENT_KEY_LIVE = "client-...";
private const string CLIENT_KEY_DEV = "client-...";

public SnipeProjectInfo GetSnipeProjectInfo()
{
	// имя параметра или вообще всё условие в вашем проекте, вероятно, будет другим
	bool devMode = GetConfigValue("snipeDev", false);
	return new SnipeProjectInfo()
	{
		ProjectID = PROJECT_ID,
		ClientKey = devMode ? CLIENT_KEY_DEV : CLIENT_KEY_LIVE,
		Mode = devMode ? SnipeProjectMode.Dev : SnipeProjectMode.Live,
	};
}
```

`ServerService`

```csharp
var projectInfo = _appConfig.GetSnipeProjectInfo();
var snipeConfig = _appConfig.GetSnipeConfig();
_snipe.InitializeSnipeContext(projectInfo, snipeConfig);
```

```csharp
public void InitializeSnipeContext(SnipeProjectInfo projectInfo, IDictionary<string, object> snipeConfig)
{
	//...
	
	var snipeContext = this.Context;
	snipeContext.Config.Initialize(projectInfo, snipeConfig);
	//...
```

Вам понадобятся `Project ID` и `Client Key`. Посмотреть их можно в дашборде снайпа. 

Подробности тут:

[Project ID](Project%20ID.md)

Либо в старом дефолтном конфиге
 `Assets/StreamingAssets/remote_config_defaults.json`.
поле `snipe_project` содержит все необходимые значения
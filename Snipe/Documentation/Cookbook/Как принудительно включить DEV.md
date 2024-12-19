#snipe #dev #config 
## Как принудительно включить DEV

```csharp
#if UNITY_EDITOR
    snipeContext.Config.InitializeDefault(new SnipeProjectInfo()
    {
        Mode = SnipeProjectMode.Dev,
        ProjectID = "___",
        ClientKey = "client-______"
    });
    //snipeContext.Config.ServerUdpUrls.Clear();  // если надо отключить UDP
    //snipeContext.Config.ServerWebSocketUrls.Clear(); // если надо отключить WebSocket
    //snipeContext.Config.HttpHeartbeatInterval = TimeSpan.FromSeconds(10); // если нужны HTTP пинги
#else
    _appConfig.InitializeSnipeConfig(snipeContext.Config);
#endif
```
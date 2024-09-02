#miniit #snipe 

## Обновление пакетов и библиотек

1. в Unity 2021+ нужно перевести в `ProjectSettings
Api compatibility level` = `.NET Standard 2.1`
2. Установить `UniTask` если его еще нет в проекте
[`https://github.com/Cysharp/UniTask.git?path=src/UniTask/Assets/Plugins/UniTask`](https://github.com/Cysharp/UniTask.git?path=src/UniTask/Assets/Plugins/UniTask)
3. Установить пакет `StreamingAssetsReader`
Если в проекте до сих пор используется `BetterStreamingAssets`, то заменить его на наш аналог
[`https://github.com/Mini-IT/unity-framework-streamingassetsreader.git`](https://github.com/Mini-IT/unity-framework-streamingassetsreader.git)
(строго говоря, удалять BSA нет необходимости, но этот пакет нужно поставить)
4. Установить пакет `Logger`
[`https://github.com/Mini-IT/unity-logger.git`](https://github.com/Mini-IT/unity-logger.git)
5. Установить пакет `NuGetForUnity` 
[`https://github.com/GlitchEnzo/NuGetForUnity.git?path=/src/NuGetForUnity`](https://github.com/GlitchEnzo/NuGetForUnity.git?path=/src/NuGetForUnity)
    1. если в проекте есть майкрософтовские dll-ки (в `Assets/Plugins` и/или `Assets/Plugins/ZString`), удалить их и поставить через NuGet (может потребоваться перезагрузить юнити, чтобы NuGet manager обнаружил, что dll-ки были удалены)
    2. Необходимые библиотеки для SnipeClient:
        1. [System.Buffers](https://www.nuget.org/packages/System.Buffers/4.5.1) (для unity 2020) - интегрирована в SnipeClient v.5, поэтому добавить её можно будет только после обновления пакета снайпа
        2. [System.Memory](https://www.nuget.org/packages/System.Memory/4.5.5) (для unity 2020)
        3. [Microsoft.Extensions.Logging.Abstractions](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Abstractions/7.0.1)
    3. Для ZString:
        1. System.Buffers (для unity 2020)
        2. System.Memory (для unity 2020)
        3. System.Runtime.CompilerServices.Unsafe
6. Обновить пакет `SnipeClient`
7. в Unity 2020 добавить `System.Buffers` (см. п. 5b)
8. Установить пакет `Social`
[`https://ghp_v72...@github.com/Mini-IT/unity-framework-social-core.git`](https://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@github.com/Mini-IT/unity-framework-social-core.git)
9. обновить пакет `Analytics - Core` до 2.3.0
10. обновить пакет `Analytics - Basic Tracker` до 2.3.0

## Правка кода

`TablesConfig.PersistentDataPath` больше не существует - вырезать все обращения
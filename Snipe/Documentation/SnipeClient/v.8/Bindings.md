Метод `GetBinding<T>()` выполнял больше одной функции, поэтому был разбит на 2 отдельных метода:
- `T RegisterBinding<T>(T binding)`
- `bool TryGetBinding<T>(bool searchBaseClasses = true, out T binding)`

Флаг `UseDefaultBindings` был старым костылём. Теперь он вырезан и проектный код явно и наглядно контролирует, какие биндинги будут использованы.
>[!warning] 
> ВАЖНО!!!
> Теперь в каждом проекте нужно явно указывать биндинги
>

```cs
// Для дефолтных биндингов можно использовать вспомогательный метод
snipeContext.Auth.RegisterDefaultBindings();

// или регистрировть каждый биндинг индивидуально
snipeContext.Auth.RegisterBinding(new DeviceIdBinding());  
snipeContext.Auth.RegisterBinding(new AdvertisingIdBinding());  
snipeContext.Auth.RegisterBinding(new FacebookBinding());  
snipeContext.Auth.RegisterBinding(new AmazonBinding());  
snipeContext.Auth.RegisterBinding(new NutakuBinding());
```

## ContextID prefix
>[!warning]
>НУЖНО протестировать биндинги и восстановления аккаунтов, т.к. переделана логика приписывания ID контекста ко внешнему ID юзера




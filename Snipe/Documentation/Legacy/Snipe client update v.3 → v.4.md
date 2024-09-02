#miniit #snipe 

## Что изменилось

1. Snipe client v.4 это бывшая ветка `autoregister`

поэтому если проект уже использовал `autoregister`, никаких переделок не требуется.

1. `flagAutoJoinRoom` отправляется автоматически. Если проект уже использует этот флаг, можно вырезать его явное указание (но не обязательно).

>[!warning] Важно!
>Если в проекте используется матчмейкинг, то сообщение `matchmaking.start` больше приходить не будет! подробнее в [тикете](https://trello.com/c/gegiW9Vp/1226-%D1%84%D0%BB%D0%B0%D0%B3-%D0%B0%D0%B2%D1%82%D0%BE-%D0%B4%D0%B6%D0%BE%D0%B9%D0%BD-%D0%BA-%D1%80%D1%83%D0%BC%D0%B5)

## Переделка на autoregister

классов `AuthProvider` и `BindProvider` (и их наследников) больше нет. они разделены на `AuthBinding` и `AuthIdFetcher`.

Из-за этого
было:

```csharp
~~void OnAccountBindingCollision(BindProvider provider, string user_name)~~
```

стало:

```csharp
void OnAccountBindingCollision(AuthBinding binding, string user_name)
```

этого больше нет. вырезаем:

```csharp
~~SnipeCommunicator.Instance.Auth.ClearAuthDataAndSetCurrentProvider(provider);~~
~~SnipeCommunicator.Instance.Auth.RebindAllProvidersAfterAuthorization();~~
```

Однако коллизии можно вообще вручную не разруливать. Добавился новый флаг
`SnipeCommunicator.Instance.Auth.AutomaticallyBindCollisions`
если он == `true`, то вообще не будет генерироваться `AccountBindingCollision`, а просто автоматически будет перепривязывать id к текущему аккаунту

AuthProvider-ы

```csharp
~~SnipeCommunicator.Instance.Auth.AddAuthProvider<xxx>();~~
```

по умолчанию уже добавлена авторизация через `DeviceId`, `AdvertisingId`, `Facebook`, `Amazon`.

для привязки амазона (если в проекте этого не было, то добавлять не нужно)

```csharp
SnipeCommunicator.Instance.Auth.GetBinding<AmazonBinding>().SetUserId(id);
```

### Перепривязка аккаунта

было:

```csharp
~~SnipeCommunicator.Instance.Auth.BindAllProviders(true);~~
```

стало:

```csharp
SnipeCommunicator.Instance.Auth.BindAll();
```

### Запрос атрибутов пользователя без авторизации

было:

```csharp
~~SnipeCommunicator.Instance.Auth.GetUserAttribute(providerId, userId, "level",~~
	(error_code, name, key, val) =>
	{
		//...
	});
```

стало:

```csharp
var attrGetter = new UnauthorizedUserAttributeGetter(SnipeCommunicator.Instance);
attrGetter.GetUserAttribute(providerId, userId, "level",
	(error_code, name, key, val) =>
	{
		//...
	});
```

### Отслеживание регистрации

т.к. отдельного запроса на регистрацию больше нет, то и события `AccountRegisterResponse` тоже больше нет. Можно подписаться на событие логина и проверять значение флага `SnipeCommunicator.Instance.Auth.JustRegistered`

было:

```csharp
~~SnipeCommunicator.Instance.Auth.AccountRegisterResponse += (errorCode, userId) =>~~
{
	if (errorCode == "ok")
	{
		// new account registered
	}
	else
	{
		// error
	}
};
```

стало:

```csharp
SnipeCommunicator.Instance.LoginSucceeded += () =>
{
	if (SnipeCommunicator.Instance.Auth.JustRegistered)
	{
		// new account registered
	}
};
```
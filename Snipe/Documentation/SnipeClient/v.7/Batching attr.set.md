#miniit #snipe #snipe-api #user-attr #request #batching

# Батчинг attr.set

Присвоения значений атрибутам, выполненные через врапперы `SnipeApi.UserAttr.UserAttributes` не будут моментально отправлены на сервер. Они будут накапливаться в течение небольшого периода времени и затем отправятся единым запросом `attr.setMulti`. 

По умолчанию этот период составляет 900 мс, но это значение можно изменить следующим образом:
```csharp
SnipeApiUserAttribute.RequestDelay = TimeSpan.FromMilliseconds(600);
```

Запросы, сформированные без использования `SnipeApi.UserAttr.UserAttributes`, батчиться не будут и сразу отправляются на сервер.

### Примеры
Старый "прямой" механизм (не батчится):
```csharp
snipeContext.Api.UserAttr.Set("facebookBlob", "xxx");
```

Как надо чтобы батчилось:
```csharp
snipeContext.Api.UserAttr.UserAttributes.facebookBlob.SetValue("xxx");
```

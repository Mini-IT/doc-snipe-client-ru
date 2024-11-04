#miniit #snipe 

### TargetInvocationException или ошибка “MessageReceived invokation error - ….”

Означает, что в обработчике сообщения случился эксепшен. Код вызова обернут в `try-catch`:

```csharp
try
{
	MessageReceived?.Invoke(...);
}
catch (Exception e)
{
	DebugLogger.LogError($”MessageReceived invokation error - {e}”);
}
```

Аналогично `TargetInvocationException` может быть вызыван при логине, если в коллбеке возникает эксепшен.

Пример сообщения в логе:
```
[AuthSubsystem] RaiseEvent - Error in the handler OnLoginSucceeded: System.Reflection.TargetInvocationException: Exception has been thrown by the target of an invocation. ---> System.NullReferenceException: Object reference not set to an instance of an object
  at MiniIT.Framework.ValueSyncer.ValueResolution+<>c__1`1[T].<Max>...
  ...
   at App.ServerContext.OnLoginSucceeded ()...
```

#miniit #snipe 

### Ошибка “MessageReceived invokation error - ….”

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
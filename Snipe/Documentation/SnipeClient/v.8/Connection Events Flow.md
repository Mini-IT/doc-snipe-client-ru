#miniit #snipe #connection #events

# События жизненного цикла соединения

Класс `SnipeCommunicator` генерирует события при измении статуса соединения. 
 - `ConnectionEstablished` - Соединение установлено
 - `ConnectionDisrupted` - Соединение прервано (временно). Будет попытка автореконнекта (если попытки не закончились)
 - `ConnectionClosed` - Окончательный дисконнект. Автореконнектов больше не будет
 - `MessageReceived` - Получено сообщение от сервера

# Псевдокод 
Для ясности проиллюстрируем последовательность вызова данных событий псевдокодом:
```csharp
event Action ConnectionEstablished;  // Соединение установлено
event Action ConnectionDisrupted;    // Соединение прервано (временно)
event Action ConnectionClosed;       // Соединение закрыто
event MessageReceivedHandler MessageReceived;

// Разрешенное количество попыток автореконнекта
public int RestoreConnectionAttempts = 10;

// количество выполненных попыток автореконнекта
private int _restoreConnectionAttempt = 0;

void NetworkLoop()
{
    // Пытаемся установить соединение
    StartConnection();

    // Если успешно приконнектились
    if (Connected)
    {
        // Уведомляем слушателей об успешном коннекте
        ConnectionEstablished();
        _restoreConnectionAttempt = 0;
    }

    // Пока соединение не разорвано
    while (Connected)
    {
        // Проверяем входящие сообщения
        if (_receivedMessagesQueue.Count > 0)
        {
            // Уведомляем слушателей о получении нового сообщения
            var message = _receivedMessagesQueue.Dequeue();
            MessageReceived(message);
        }
    }

    // Соединение разорвано
    OnConnectionFailed();
}

void OnConnectionFailed()
{
    // Уведомляем слушателей о разрыве соединения
    ConnectionDisrupted();

    // Если ещё не все попытки реконнекта исчерпались
    if (_restoreConnectionAttempt < RestoreConnectionAttempts)
    {
        // Уведомляем слушателей о попытке реконнекта
        ReconnectionScheduled();

        // Увеличиваем счетчик попыток
        _restoreConnectionAttempt++;

        // запускаем процедуру реконнекта (начинается с кулдауна)
        AttemptToRestoreConnection();
        return;
    }

    // Все попытки закончились. Автореконнектов больше не будет.
    // Уведомляем слушателей об окончательном дисконнекте.
    ConnectionClosed();
}
```
Кратко основные отличия:
- `SnipeObject` больше не существует - используется стандартный `IDictionary<string, object>`
- `SnipeManager` - единая точка для получения доступа к контекстам и таблицам
- Таблицы отделены от контекста
- `ConnectionFailed` разделено на 3 отдельных события без аргументов ([Connection Events Flow](Connection%20Events%20Flow.md)): 
	- `ConnectionClosed` - Connection is completely lost. No reties left
	- `ConnectionDisrupted` - Connection failed or lost
	- `ReconnectionScheduled` - Automatic connection recovery routine initiated (вместо старого флага `will_reconnect`)
- Отслеживание состояния `common -> matchmaking -> room` и интенсификация HTTP-пингов
- Инициализация биндингов переделана. См. [Bindings](Bindings.md)

## Relogin + ValueSyncer
При перелогинивании больше не нужно уничтожать контекст и, соответственно, в `UserDataService` не требуется пересоздавать `ValueSyncer`-ы. Однако, нужно иметь в виду, что после логина в другой аккаунт, значения атрибутов изменятся и `ValueSyncer` может их неправильно перезаполнить из-за того, что у него хранится значение от предыдущего аккаунта.

## Подробнее
См. [Migration from SnipeClient 7 to 8](Migration%20from%20SnipeClient%207%20to%208.md)

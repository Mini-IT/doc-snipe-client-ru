#miniit #snipe #iap 

Быстрый старт по созданию магазина на стороне Yookassa:
[https://yookassa.ru/developers/payment-acceptance/getting-started/quick-start](https://yookassa.ru/developers/payment-acceptance/getting-started/quick-start)

Создание тестового магазина и тестирование платежей:
[https://yookassa.ru/developers/payment-acceptance/testing-and-going-live/testing#test-options](https://yookassa.ru/developers/payment-acceptance/testing-and-going-live/testing#test-options)

На сервере настроено, что на дев кластере всегда работает тестовый магазин, если он определен. В нем можно использовать тестовые номера карт для бесплатных платежей. На лайве всегда обычный магазин.

В редакторе нужно:

1. Настроить внутренние переменные
- `yookassa.shopID` - идентификатор указан в разделе Настройки —> Магазин.
- `yookassa.secret` - секретный ключ нужно сгенерировать и активировать паролем из смс в разделе Интеграция —> Ключи API.
- `yookassa.payment.backendKey` - ключ для работы ссылки начала платежа

2. Для работы тестового магазина аналогично заполнить
- `yookassa.dev.shopID`
- `yookassa.dev.secret`

3. Заполнить все Payments (provider = yook)

Клиенту нужно:

1. Обработать `SnipeApi.PaymentYookassa.Completed` - игрок совершил покупку и получил награду.
2. Загружать список всех продуктов из `SnipeApi.Context.Tables.PaymentItems.Items` при инициализации.
3. Задать `YOKASSA_KEY` - отправляется вместе с запросом покупки (в редакторе это yookassa.payment.backendKey, спросить у кого есть доступ до внутренних переменных).
4. Задать URL запроса покупки, который будет открываться через `Application.OpenURL(url)`.
```csharp
int userID = SnipeApi.Context.Auth.UserID;
string itemID = _processingProduct.productId;
bool isDev = _appConfig.GetConfigValue<bool>("snipeDev");

string baseUrl = $"https://kit.snipe.dev/api/v1/yookassa/start/{YOKASSA_KEY}";
string url = $"{baseUrl}?userID={userID}&itemID={itemID}&isDev={isDev.ToString().ToLower()}";
```

Для удобств хранения данных о продуктах, лучше создать кастомный класс:
```csharp
public class YookassaProduct
{
    public enum ProductType
    {
        NON_CONSUMABLE,
        CONSUMABLE,
        SUBSCRIPTION
    }

    public YookassaProduct(string productId, ProductType productType, int price, string currency)
    {
        this.productId = productId;
        this.productType = productType;
        this.price = price;
        this.currency = currency;
    }

    public string productId;
    public ProductType productType;
    public int price;
    public string currency;
}
````

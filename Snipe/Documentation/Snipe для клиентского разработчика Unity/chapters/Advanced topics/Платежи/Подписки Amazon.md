#miniit #snipe #iap #iap-subscription #amazon #android 

1. подключить при старте сессии привязку Amazon User ID (userID получается таким образом - [https://docs.unity3d.com/Packages/com.unity.purchasing@4.4/manual/UnityIAPAmazonExtendedFunctionality.html](https://docs.unity3d.com/Packages/com.unity.purchasing@4.4/manual/UnityIAPAmazonExtendedFunctionality.html))
2. при инициализации и завершении платежа за подписку **НЕ** отправлять на сервер обычный `PaymentAmazon.Notify`, сервис Амазона уведомит его сам
3. Обработать `PaymentAmazon.Subscribed` - игрок подписался и получил награды по списку, формат наград ниже
4. Обработать `PaymentAmazon.Unsubscribed` - игрок отменил подписку или она истекла, это нотифай на клиент

>[!warning] ВАЖНО
>При поиске в какой БД находится игрок сперва проверяется лайв. Поэтому если привязка есть и в деве и в лайве, то подписка сработает и выдаст награду в лайве. Чтобы этого избежать во время теста, надо удалять из лайв-версии проекта привязку тестера к Amazon.


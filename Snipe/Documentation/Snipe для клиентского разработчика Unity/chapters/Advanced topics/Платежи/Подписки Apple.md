#miniit #snipe #iap #iap-subscription #ios

>[!warning] ВАЖНО
>Проблема при тестировании: при первой транзакции эппл (и бд проекта) привязывается через originalTransactionId к конкретному юзеру. последующие с2с уведомления (автоматический RENEW раз в 4 минуты) ищут транзакцию по этому иду. обычно это юзер на дев сервере.


В V2 версии API есть appAccountToken, уникальный ид юзера, который можно добавить в транзакцию:

A UUID that associates the transaction with a user on your own service. If your app doesn’t provide an appAccountToken, this string is empty. For more information, see appAccountToken(_:).

[https://developer.apple.com/documentation/appstoreservernotifications/jwstransactiondecodedpayload](https://developer.apple.com/documentation/appstoreservernotifications/jwstransactiondecodedpayload)

[https://developer.apple.com/documentation/storekit/product/purchaseoption/3749440-appaccounttoken](https://developer.apple.com/documentation/storekit/product/purchaseoption/3749440-appaccounttoken)

Но в Unity еще не сделали поддержку StoreKit 2. Ждем пока сделают…:

[https://forum.unity.com/threads/apple-storekit-2.1230108/](https://forum.unity.com/threads/apple-storekit-2.1230108/)
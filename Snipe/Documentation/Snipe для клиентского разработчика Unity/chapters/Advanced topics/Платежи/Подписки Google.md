#miniit #snipe #iap #iap-subscription #android

В редакторе нужно:

1. Настроить внутреннюю переменную googlePlay.payment.backendKey
2. Сохранить урл колбека в гугловой админке - [https://kit.snipe.dev/api/v1/google/notify/](https://kit.snipe.dev/api/v1/google/notify/)<ключ>

Клиенту нужно:

1. Обработать PaymentGoogle.Subscribed - игрок подписался и получил награды по списку, формат наград ниже
2. Обработать PaymentGoogle.Unsubscribed - игрок отменил подписку или она истекла, это нотифай на клиент
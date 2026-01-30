#miniit #snipe #iap #iap-subscription #android

Платежи Xsolla также являются платежами в Crazygames.

Доки:

1) https://docs.crazygames.com/sdk/in-game-purchases/ - интеграция в CrazyGames
2) https://developers.xsolla.com/xps/game-keys/setup-selling/set-up-webhooks/ - как сетапить серверные вебхуки в Xsolla

Заметки и что нужно сделать:

1) В редакторе нужно внутренние переменные: `xsolla.payment.backendKey`, `xsolla.payment.secret`
2) Создать платежные айтемы с провайдером `xsol`
3) Cохранить урл серверного вебхука в админке Xsolla (в Crazygames доках написано что дадут доступ) - `https://kit.snipe.dev/api/v1/xsolla/notify/<xsolla.payment.backendKey>`
4) При создании платежа в CrazyGames надо в user external_id писать crazygames id (это их требование из https://docs.crazygames.com/sdk/in-game-purchases/
5) При этом в `custom_parameters:object` мы пишем обьект с данными снайпа: `{ projectID:string, userID:int }` (судя по их докам именно обьект, а не JSON-строку!). Здесь projectID полный, например `iracing_dev` или `iracing_live`. По нему мы определяем где искать юзера с этим идом.
6) После успешного платежа сервер Xsolla запустит вебхук и уведомит Снайп о платеже. Снайп отправит сообщение клиенту: `type=payment/generic.completed, params={ id:string, itemID:string, results:[]PaymentResult, provider=xsol, providerParams:{ orderID:string } }`

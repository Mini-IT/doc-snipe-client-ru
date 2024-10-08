#miniit #snipe #snipe-binding #login 

При регистрации на сервере игроку присваивается личный `id`. Чтобы игрок его не потерял и смог вернуться в игру даже после удаления и переустановки приложения, к внутреннему `id` добавляются другие идентификаторы, позволяющие определить, что это тот же самый пользователь. В первую очередь привязываются `DeviceID` и `AdvertisingID` (если доступен). Также пользователям предлагается авторизоваться через Facebook, чтобы можно было привязаться еще и к этому аккаунту.

Привязки аккаунтов осуществляются в автоматическом режиме без участия клиентского программиста (на данный момент за исключением Amazon-а). Однако в некоторых случаях могут возникнуть коллизии, которые нужно разрешить в соответствии с требованиями конкретного проекта.

Например, пользователь установил приложение, поиграл какое-то время и привязал свой Facebook аккаунт. Затем купил новый смартфон и туда тоже установил это же приложение. В результате на сервере будет создан новый аккаунт, в котором пользователь может тоже получить какой-то игровой прогресс. Далее, когда на новом девайсе пользователь авторизуется через Facebook, на сервере найдется старый аккаунт. В этом случае вызовется событие

`snipeContext.Auth.AccountBindingCollision`

В зависимости от требований проекта, нужно либо предложить пользователю выбор аккаунта, либо каким-то образом автоматически “склеить” аккаунты (перенести значения из старого в новый).
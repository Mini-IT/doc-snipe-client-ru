#miniit #snipe #config 

Перед соединением с сервером запрашивается ремоут конфиг (наша замена Фаербейзу):

`POST` `https://config.snipe.dev/api/v1/config` 

`project`: `string` - строковый ид дашборд версии проекта (то есть без `_dev` или `_live`)

`deviceID`: `string`

`identifier`: `string`

`version`: `string`

`packageVersion`: `string`

`platform`: `string`

Ответ:

`errorCode`: `string`

`data` - обьект конфига в соотв с его описанием в редакторе

Ошибки с хттп статусом:

400 - wrongMessage, noSuchProject

500 - serviceError
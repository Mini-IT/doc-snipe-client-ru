#miniit #snipe #iap #iap-subscription #android

В редакторе нужно:

1. Настроить внутреннюю переменную googlePlay.payment.backendKey

ПМ-у нужно:
1. Открыть console.cloud.google.com
2. Выбрать проект <img width="2846" height="1220" alt="image" src="https://github.com/user-attachments/assets/350a3a03-5db7-4900-bf29-acd9117ee770" />
3. В поиске найти pub/sub <img width="2852" height="1144" alt="image" src="https://github.com/user-attachments/assets/fb79ae5f-8f3a-4c5d-a517-9f8770c0d7a8" />
4. Выбрать  Create topic <img width="1420" height="516" alt="image" src="https://github.com/user-attachments/assets/ba7c23bf-0228-4843-82a9-8940c2d66920" />
5. Создаём топик подписки и переходим в раздел Subscriptions <img width="1421" height="588" alt="image" src="https://github.com/user-attachments/assets/0145f4ed-dda3-4463-a93d-866369128fdc" />
6. В разделе подписки переходим к редактированию <img width="1418" height="657" alt="image" src="https://github.com/user-attachments/assets/8531e9f6-45e3-457d-8be2-33e0ccf78934" />
7. Выбираме Delivery type - Push, в поле Endpoint url вставляем адрес googlePlay.payment.backendKey из Snipe проекта <img width="2850" height="1188" alt="image" src="https://github.com/user-attachments/assets/fbb2f0cf-753c-4419-94ba-c0c4236fc572" />

8. Выставляем Expiry period - Never expire, Acknowledgement deadline - 10 sec <img width="2848" height="1104" alt="image" src="https://github.com/user-attachments/assets/41c58577-59f1-4888-b61c-9bdebca9d5ec" />

9. Остальные пункты не трогаем, сохраняем изменения.


Клиенту нужно:

1. Обработать PaymentGoogle.Subscribed - игрок подписался и получил награды по списку, формат наград ниже
2. Обработать PaymentGoogle.Unsubscribed - игрок отменил подписку или она истекла, это нотифай на клиент

**Клиенту НЕ НУЖНО отправлять на сервер колбэк PaymentGoogle.Notify в случае подписок.**

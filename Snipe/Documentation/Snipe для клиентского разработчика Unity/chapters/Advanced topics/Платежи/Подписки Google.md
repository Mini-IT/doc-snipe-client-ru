#miniit #snipe #iap #iap-subscription #android

В редакторе нужно:

1. Настроить внутреннюю переменную googlePlay.payment.backendKey

ПМ-у нужно:
1. Открыть console.cloud.google.com
2. Выбрать проект <img width="2846" height="1220" alt="image" src="https://github.com/user-attachments/assets/388225ba-d946-4380-a826-a425f69049f2" />
3. В поиске найти pub/sub <img width="2852" height="1144" alt="image" src="https://github.com/user-attachments/assets/fb79ae5f-8f3a-4c5d-a517-9f8770c0d7a8" />
4. Выбрать  Create topic <img width="1420" height="516" alt="image" src="https://github.com/user-attachments/assets/ba7c23bf-0228-4843-82a9-8940c2d66920" />
5. Создаём топик подписки и переходим в раздел Subscriptions <img width="1421" height="588" alt="image" src="https://github.com/user-attachments/assets/0145f4ed-dda3-4463-a93d-866369128fdc" />
6. В разделе подписки переходим к редактированию <img width="1395" height="525" alt="image" src="https://github.com/user-attachments/assets/6d7e375b-d66b-48b8-a31e-37f8581e1b45" />
7. Выбираме Delivery type - Push, в поле Endpoint url вставляем адрес googlePlay.payment.backendKey из Snipe проекта <img width="2850" height="1188" alt="image" src="https://github.com/user-attachments/assets/fbb2f0cf-753c-4419-94ba-c0c4236fc572" />

8. Выставляем Expiry period - Never expire, Acknowledgement deadline - 10 sec <img width="2848" height="1104" alt="image" src="https://github.com/user-attachments/assets/41c58577-59f1-4888-b61c-9bdebca9d5ec" />
9. Попросить Влада добавить учётную запись службы google-play-developer-notifications@system.gserviceaccount.com с ролью Pub/Sub Publisher <img width="1420" height="652" alt="image" src="https://github.com/user-attachments/assets/a0f36fcd-0d0a-406b-8c80-cf7e4b2c2365" />
10. В google console в настройках монетизации нужно включить уведомления в режиме реального времени (Real-time developer notifications) и скопировать в поле Название темы полное наименование из Sub/Pub, сохранить изменения и проверить тестовую отправку сообщения, если возникает ошибка с отправкой - нужно вернуться к настройке доступа к чётной записи <img width="2814" height="1090" alt="image" src="https://github.com/user-attachments/assets/6be44cb1-f5c6-4a35-899e-fb098f900ec7" />

    скопировать полное название темы можно в настройках
    <img width="755" height="489" alt="image" src="https://github.com/user-attachments/assets/85b18974-db27-4266-b197-9c5e1ce8b942" />


Клиенту нужно:

1. Обработать PaymentGoogle.Subscribed - игрок подписался и получил награды по списку, формат наград ниже
2. Обработать PaymentGoogle.Unsubscribed - игрок отменил подписку или она истекла, это нотифай на клиент

**Клиенту НЕ НУЖНО отправлять на сервер колбэк PaymentGoogle.Notify в случае подписок.**

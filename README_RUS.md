# whatsapp-api-client library for javascript
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/green-api/whatsapp-api-client/blob/master/LICENSE)
[![GitHub release](https://img.shields.io/github/v/release/green-api/whatsapp-api-client.svg)](https://github.com/green-api/whatsapp-api-client/releases)

Javascript библиотека для интеграции с мессенджером WhatsAPP через API сервиса [green-api.com](https://green-api.com). Чтобы воспользоваться библиотекой нужно получить регистрационный токен и id инстанса через сервер [green-api.com](https://green-api.com).

## API

Документация к REST API находится по [ссылке](https://green-api.com/documents/green-api.html#82fcbe04-233f-492d-baf1-098f340bc0dc). Библиотека является оберткой к REST API, поэтому документация по ссылке выше применима и к самой библиотеке.

## Установка

Билиотека работает как на node >=10, так и на современных версиях браузеров chrome, firefox и др. Для приложений на webpack и npm установка выполняется через команду:
```
npm i @green-api/whatsapp-api-client
```
Для чистого html js сайта либу можно подключить в index.html
``` html
<script type="text/javascript" src="https://unpkg.com/@green-api/whatsapp-api-client/lib/whatsapp-api-client.min.js"></script>
```

## Авторизация 

Чтобы отправить сообщение или выполнить другой метод API, аккаунт WhatsApp в приложении телефона должен быть в авторизованном состоянии. 

Это можно сделать двумя способами:
1. Через веб-интерфейс с помощью считывания QR кода по ссылке https://api.green-api.com/waInstance{{idInstance}}/{{apiTokenInstance}}, где ``idInstance`` и ``apiTokenInstance`` это параметры, полученные при регистрации на [green-api.com](https://green-api.com)

2. Программно. Выполняется с помощью считывания QR кода через websocket отдельным методом API scanqrcode, который пока не реализован в библиотеке. Описание этого метода доступно по ссылке [ instance.scanqrcode (websocket)](https://documenter.getpostman.com/view/11185176/Szme3xf1?version=latest#048e8f7c-5bf1-4655-a719-c2d2ee78c676) 

## Примеры

### Отправка сообщения на номер WhatsApp
Используя common js
``` js
const whatsAppClient = require('@green-api/whatsapp-api-client')

const restAPI = whatsAppClient.restAPI(({
    idInstance: YOUR_ID_INSTANCE,
    apiTokenInstance: YOUR_API_TOKEN_INSTANCE
}))

restAPI.message.sendMessage(null, 79999999999, "hello world")
.then((data) => {
    console.log(data);
}) ;

```
или используя js script
``` html
<script src="https://unpkg.com/@green-api/whatsapp-api-client/lib/whatsapp-api-client.min.js"></script>
<script>
    const restAPI = whatsAppClient.restAPI(({
        idInstance: "YOUR_ID_INSTANCE",
        apiTokenInstance: "YOUR_API_TOKEN_INSTANCE"
    }))
    restAPI.message.sendMessage(null, 79999999999, "hello world")
    .then((data) => {
        console.log(data);
    }).catch((reason) =>{
        console.error(reason);
    });
</script>
```
Или можно воспользоваться синтаксисом ES6. Для того, чтобы этот синтаксис сработал в приложении на nodejs, нужно в package.json прописать ключ ``"type": "module"``. Далее все примеры будут на ES6 синтаксисе.

``` js
import whatsAppClient from '@green-api/whatsapp-api-client'

(async () => {
    const restAPI = whatsAppClient.restAPI(({
        idInstance: YOUR_ID_INSTANCE, 
        apiTokenInstance: YOUR_API_TOKEN_INSTANCE
    }))
    const response = await restAPI.message.sendMessage(null, 79999999999, "hello world");
})();
```
Пример кода здесь: [SendWhatsAppMessage.js](examples/SendWhatsAppMessage.js)

### Отправка файла на номер WhatsApp
``` js
import whatsAppClient from '@green-api/whatsapp-api-client'

(async () => {
    const restAPI = whatsAppClient.restAPI(({
        idInstance: YOUR_ID_INSTANCE,
        apiTokenInstance: YOUR_API_TOKEN_INSTANCE
    }))
    const response = await restAPI.file.sendFileByUrl(null, 79999999999, 'https://avatars.mds.yandex.net/get-pdb/477388/77f64197-87d2-42cf-9305-14f49c65f1da/s375', 'horse.png', 'horse');
})();
```
Пример кода здесь: [SendWhatsAppFile.js](examples/SendWhatsAppFile.js)

### Пример использования вебхука

Вебхуки работают только в node js с на базе express

``` js
import whatsAppClient from '@green-api/whatsapp-api-client'
import express from "express";
import bodyParser from 'body-parser';

(async () => {
    try {

        // Устанавливаем http url ссылку, куда будут отправляться вебхуки. 
        // Ссылка должна иметь публичный адрес.
        await restAPI.settings.setSettings({
            webhookUrl: 'MY_HTTP_SERVER:3000/webhooks'
        });

        const app = express();
        app.use(bodyParser.json());
        const webHookAPI = whatsAppClient.webhookAPI(app, '/webhooks')

        // Подписываемся на событие вебхука при получении сообщения
        webHookAPI.onIncomingMessageReceivedHookText((data, idInstance, idMessage, sender, typeMessage, textMessage) => {
            console.log(`outgoingMessageStatus data ${data.toString()}`)
        });

        // Запускаем веб сервер, имеющий публичный адрес
        app.listen(3000, async () => {
            console.log(`Started. App listening on port 3000!`)

            const restAPI = whatsAppClient.restAPI(({
                idInstance: MY_ID_INSTANCE,
                apiTokenInstance: MY_API_TOKEN_INSTANCE
            }));
            // Отправляем тестовое сообщение, для того чтобы сработали события вебхуков
            const response = await restAPI.message.sendMessage(null, 79999999999, "hello world");
    
        });
    } catch (error) {
        console.error(error);
        process.exit(1);
    }
})();

```
Пример кода здесь: [ReceiveWebhook.js](examples/ReceiveWebhook.js)

## Разворачивание окружения разработки

Помощь в доработке и в исправлении ошибок приветствуется. Шаги для разворачивания:

1. Склонируйте репозиторий через git clone
2. Установите зависимости через npm install
3. Установите глобально библиотеку ``rollup`` для сборки.
4. Для вебхуков добавьте express как новую зависимость через npm
5. Создайте файл .env в рутовом каталоге и пропишите переменные окружения. Образец переменных в файле [env.example](env.example)

## Сборка
Скомпилировать как browser, так и node/webpack версии либы можно одной командой
```
npm run build
```

## Сторонние продукты

* [axios](https://github.com/axios/axios) - для http запросов
* [express](https://www.npmjs.com/package/express) - сервер приложений для вебхуков

## Лицензия

Лицензировано на условиях MIT. Смотрите файл [LICENSE](LICENSE)

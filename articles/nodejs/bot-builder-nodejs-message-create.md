---
title: Creación de mensajes | Microsoft Docs
description: Obtenga información sobre cómo crear mensajes con Bot Framework SDK para Node.js.
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/7/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 7e2f48a3450154de9e2465f9d0d992ace4f3996f
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299805"
---
# <a name="create-messages"></a>Creación de mensajes

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

La comunicación entre el bot y el usuario se realiza a través de mensajes. El bot enviará actividades de mensajes para comunicar información a los usuarios y, del mismo modo, recibirá actividades de mensajes de los usuarios. Algunos mensajes pueden constar simplemente de texto sin formato, mientras que otros pueden incluir contenido más enriquecido, como texto que se va a decir, acciones sugeridas, datos adjuntos multimedia, tarjetas enriquecidas y datos específicos del canal.

En este artículo se describen algunos de los métodos de mensaje usados con frecuencia que puede utilizar para mejorar la experiencia del usuario.

## <a name="default-message-handler"></a>Controlador de mensajes predeterminado

Bot Framework SDK para Node.js incluye un controlador de mensajes predeterminado integrado en el objeto [`session`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html). Este controlador de mensajes permite enviar y recibir mensajes de texto entre el bot y el usuario.

### <a name="send-a-text-message"></a>Enviar un mensaje de texto

El envío de un mensaje de texto mediante el controlador de mensajes predeterminado es sencillo, simplemente llame a `session.send` y pase una **cadena**.

En este ejemplo se muestra cómo se puede enviar un mensaje de texto para saludar al usuario.
```javascript
session.send("Good morning.");
```

En este ejemplo se muestra cómo se puede enviar un mensaje de texto mediante la plantilla de cadena de JavaScript.
```javascript
var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
session.send(msg); //msg: "You ordered: Potato Salad for a total of $5.99."
```

### <a name="receive-a-text-message"></a>Recibir un mensaje de texto

Cuando un usuario envía un mensaje al bot, el bot recibe el mensaje a través de la propiedad `session.message`.

En este ejemplo se muestra cómo acceder al mensaje del usuario.
```javascript
var userMessage = session.message.text;
```

## <a name="customizing-a-message"></a>Personalización de un mensaje

Para tener más control sobre el formato del texto de los mensajes, puede crear un objeto [`message`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html) personalizado y establecer las propiedades necesarias antes de enviarlo al usuario.

En este ejemplo se muestra cómo crear un objeto `message` personalizado y establecer las propiedades `text`, `textFormat` y `textLocale`.

```javascript
var customMessage = new builder.Message(session)
    .text("Hello!")
    .textFormat("plain")
    .textLocale("en-us");
session.send(customMessage);
```

En casos donde el objeto `session` no está en el ámbito, se puede usar el método `bot.send` para enviar un mensaje con formato al usuario.

La propiedad `textFormat` de un mensaje se puede usar para especificar el formato del texto. La propiedad `textFormat` se puede establecer en **plain**, **markdown** o **xml**. El valor predeterminado de `textFormat` es **markdown**. 

## <a name="message-property"></a>Propiedad de mensajes

El objeto [`Message`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html) tiene una propiedad **data** interna que usa para administrar el mensaje que se envía. El resto de propiedades se establecen a través de los distintos métodos que expone este objeto. 

## <a name="message-methods"></a>Métodos de mensaje

Las propiedades de mensaje se establecen y recuperan a través de los métodos del objeto. En la tabla siguiente se proporciona una lista de los métodos que se pueden llamar para establecer y obtener las diferentes propiedades **Message**.

| Método | DESCRIPCIÓN |
| ---- | ---- | 
| [`addAttachment(attachment:AttachmentType)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#addattachment) | Agrega datos adjuntos a un mensaje|
| [`addEntity(obj:Object)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#addentity) | Agrega una entidad al mensaje. |
| [`address(adr:IAddress)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#address) | Información de enrutamiento de direcciones para el mensaje. Para enviar un mensaje automático al usuario, guarde la dirección del mensaje en el contenedor userData. |
| [`attachmentLayout(style:string)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#attachmentlayout) | Sugerencia sobre cómo deben diseñar los clientes varios datos adjuntos. El valor predeterminado es "list". |
| [`attachments(list:AttachmentType)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#attachments) | Una lista de tarjetas o imágenes para enviar al usuario. |
| [`compose(prompts:string[], ...args:any[])`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#compose) | Crea una respuesta compleja y aleatoria para el usuario. |
| [`entities(list:Object[])`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#entities) | Objetos estructurados que se pasan al bot o al usuario. |
| [`inputHint(hint:string)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#inputhint) | Sugerencia enviada al usuario para informarle de si el bot está esperando una entrada adicional o no. Las solicitudes integradas rellenarán automáticamente este valor para los mensajes salientes. |
| [`localTimeStamp((optional)time:string)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#localtimestamp) | Hora local del envío del mensaje (establecida por el cliente o el bot, por ejemplo: 2016-09-23T13:07:49.4714686-07:00.) |
| [`originalEvent(event:any)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#originalevent) | Mensaje en formato original o nativo del canal para los mensajes entrantes. |
| [`sourceEvent(map:ISourceEventMap)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#sourceevent) | Para los mensajes salientes se puede usar para pasar datos de eventos específicos de origen como datos adjuntos personalizados. |
| [`speak(ssml:TextType, ...args:any[])`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#speak) | Establece el campo de voz del mensaje como *Lenguaje de marcado de síntesis de voz (SSML)* . Esto se proporcionará de forma hablada al usuario en los dispositivos compatibles. |
| [`suggestedActions(suggestions:ISuggestedActions `&#124;` IIsSuggestedActions)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#suggestedactions) | Acciones sugeridas opcionales para enviar al usuario. Las acciones sugeridas solo se mostrarán en los canales que admiten acciones sugeridas. |
| [`summary(text:TextType, ...argus:any[])`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#summary) | El texto se mostrará como reserva y como una descripción breve del contenido del mensaje (por ejemplo: lista de conversaciones recientes). |
| [`text(text:TextType, ...args:any[])`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#text) | Establece el texto del mensaje. |
| [`textFormat(style:string)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#textformat) | Establece el formato de texto. El formato predeterminado es **markdown**. |
| [`textLocale(locale:string)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#textlocale) | Establece el idioma de destino del mensaje. |
| [`toMessage()`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#tomessage) | Obtiene el código JSON para el mensaje. |
| [`composePrompt(session:Session, prompts:string[], args?:any[])`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#composeprompt-1) | Combina una matriz de mensajes en un único mensaje localizado y, después, rellena las ranuras de plantilla de mensajes de forma opcional con los argumentos que se pasan. |
| [`randomPrompt(prompts:TextType)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#randomprompt) | Obtiene un mensaje aleatorio de la matriz de **prompts* que se pasa. |

## <a name="next-step"></a>Paso siguiente

> [!div class="nextstepaction"]
> [Envío y recepción de archivos adjuntos](bot-builder-nodejs-send-receive-attachments.md)


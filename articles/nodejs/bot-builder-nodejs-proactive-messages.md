---
title: Envío de mensajes automáticos | Microsoft Docs
description: Aprenda a interrumpir el flujo de la conversación actual con un mensaje automático mediante Bot Framework SDK para Node.js
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8ca8043c5680a993fa27e2febb9740206691884c
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225580"
---
# <a name="send-proactive-messages"></a>Envío de mensajes automáticos
[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-proactive-messages.md)
> - [Node.js](../nodejs/bot-builder-nodejs-proactive-messages.md)

[!INCLUDE [Introduction to proactive messages - part 1](../includes/snippet-proactive-messages-intro-1.md)]

## <a name="types-of-proactive-messages"></a>Tipos de mensajes automáticos

[!INCLUDE [Introduction to proactive messages - part 2](../includes/snippet-proactive-messages-intro-2.md)]

## <a name="send-an-ad-hoc-proactive-message"></a>Envío de un mensaje automático ad hoc

Los ejemplos de código siguientes muestran cómo enviar un mensaje automático ad hoc con Bot Framework SDK para Node.js.

Para poder enviar un mensaje ad hoc a un usuario, en primer lugar, el bot debe recopilar y almacenar información sobre el usuario de la conversación actual. La propiedad **dirección** del mensaje incluye toda la información que el bot necesitará para enviar un mensaje ad hoc al usuario más adelante. 

```javascript
bot.dialog('adhocDialog', function(session, args) {
    var savedAddress = session.message.address;

    // (Save this information somewhere that it can be accessed later, such as in a database, or session.userData)
    session.userData.savedAddress = savedAddress;

    var message = 'Hello user, good to meet you! I now know your address and can send you notifications in the future.';
    session.send(message);
})
```

> [!NOTE]
> El bot puede almacenar los datos de usuario de cualquier manera, siempre que pueda acceder a ellos más adelante.

Una vez que el bot ha recopilado la información sobre el usuario, puede enviar un mensaje automático ad hoc al usuario en cualquier momento. Para ello, simplemente recupera los datos de usuario que almacenó previamente, construye el mensaje y lo envía.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

var bot = new builder.UniversalBot(connector)
                .set('storage', inMemoryStorage); // Register in-memory storage 

function sendProactiveMessage(address) {
    var msg = new builder.Message().address(address);
    msg.text('Hello, this is a notification');
    msg.textLocale('en-US');
    bot.send(msg);
}
```

> [!TIP]
> Un mensaje ad hoc automático se puede iniciar del mismo modo que desde activadores asincrónicos, como solicitudes HTTP, temporizadores, colas o desde cualquier otro lugar que el desarrollador elija.

## <a name="send-a-dialog-based-proactive-message"></a>Envío de un mensaje automático basado en diálogos

Los ejemplos de código siguientes muestran cómo enviar un mensaje automático basado en diálogos con Bot Framework SDK para Node.js. Puede encontrar el ejemplo completo en la carpeta [Microsoft/BotBuilder-Samples/nodo/core-proactiveMessages/startNewDialog](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages/startNewDialog).

Para poder enviar un mensaje basado en diálogos a un usuario, en primer lugar el bot debe recopilar y guardar información de la conversación actual. El objeto `session.message.address` incluye toda la información que el bot necesitará para enviar un mensaje automático basado en diálogos al usuario. 

```javascript
// proactiveDialog dialog
bot.dialog('proactiveDialog', function (session, args) {

    savedAddress = session.message.address;

    var message = 'Hey there, I\'m going to interrupt our conversation and start a survey in five seconds...';
    session.send(message);

    message = `You can also make me send a message by accessing: http://localhost:${server.address().port}/api/CustomWebApi`;
    session.send(message);

    setTimeout(() => {
        startProactiveDialog(savedAddress);
    }, 5000);
});
```

Cuando llega el momento de enviar el mensaje, el bot crea un diálogo y lo agrega a la parte superior de la pila del diálogo. El nuevo diálogo controla la conversación, entrega el mensaje automático, lo cierra y luego devuelve el control al diálogo anterior de la pila. 

```javascript
// initiate a dialog proactively 
function startProactiveDialog(address) {
    bot.beginDialog(address, "*:survey");
}
```

> [!NOTE]
> El ejemplo de código anterior requiere un archivo personalizado, **botadapter.js**, que puede [descargar desde GitHub](https://github.com/Microsoft/BotBuilder-Samples/blob/master/Node/core-proactiveMessages/startNewDialog/botadapter.js).

El diálogo de encuesta controla la conversación hasta que termina. A continuación, se cierra (mediante una llamada a `session.endDialog()`), con lo que el control se devuelve al diálogo anterior. 


```javascript
// handle the proactive initiated dialog
bot.dialog('survey', function (session, args, next) {
  if (session.message.text === "done") {
    session.send("Great, back to the original conversation");
    session.endDialog();
  } else {
    session.send('Hello, I\'m the survey dialog. I\'m interrupting your conversation to ask you a question. Type "done" to resume');
  }
});
```

## <a name="sample-code"></a>Código de ejemplo

Para obtener un ejemplo completo en el que se muestre cómo enviar mensajes automáticos con Bot Framework SDK para Node.js, consulte <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages" target="_blank">Ejemplo de mensajes automáticos</a> en GitHub. En el ejemplo de mensajes automáticos, <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages/simpleSendMessage" target="_blank">simpleSendMessage</a> muestra cómo enviar un mensaje automático ad hoc y <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages/startNewDialog" target="_blank">startNewDialog</a> muestra cómo enviar un mensaje automático basado en diálogos.

## <a name="additional-resources"></a>Recursos adicionales

- [Diseño del flujo de conversación](../bot-service-design-conversation-flow.md)

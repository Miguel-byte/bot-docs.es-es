---
title: Envío de un indicador de escritura | Microsoft Docs
description: Aprenda a agregar un indicador "Espere" para indicar a un usuario que un bot está procesando una solicitud mediante Bot Framework SDK para Node.js
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: b7bef8e17584d94d6821e8b936abae9ef20088e2
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299705"
---
# <a name="send-a-typing-indicator"></a>Envío de un indicador de escritura 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Los usuarios esperan una respuesta a tiempo a sus mensajes. Si el bot realiza alguna tarea de larga ejecución como llamar a un servidor o ejecutar una consulta sin proporcionar al usuario ninguna indicación de que el bot le ha escuchado, el usuario podría impacientarse o simplemente suponer que el bot no funciona.
Muchos canales admiten el envío de una indicación de escritura para mostrar al usuario que el mensaje se recibió y que se está procesando.


## <a name="typing-indicator-example"></a>Ejemplo de indicador de escritura

En el ejemplo siguiente se muestra cómo enviar una indicación de escritura mediante [session.sendTyping()][SendTyping].  Puede probar esto con el emulador de Bot Framework.


```javascript

// Create bot and default message handler
var bot = new builder.UniversalBot(connector, function (session) {
    session.sendTyping();
    setTimeout(function () {
        session.send("Hello there...");
    }, 3000);
});
```

Los indicadores de escritura también son útiles cuando se inserta un retraso de mensaje para evitar que los mensajes que contienen imágenes se envíen desordenados.

Para más información, consulte [How to send a rich card](bot-builder-nodejs-send-rich-cards.md) (Envío de una tarjeta enriquecida).


## <a name="additional-resources"></a>Recursos adicionales

* [sendTyping][SendTyping]


[SendTyping]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping
[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

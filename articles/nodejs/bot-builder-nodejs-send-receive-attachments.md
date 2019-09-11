---
title: Envío y recepción de datos adjuntos | Microsoft Docs
description: Obtenga información sobre cómo enviar y recibir mensajes que contengan datos adjuntos con Bot Framework SDK para Node.js.
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: c830ed61a70b3b92451988ea73f223da3e55bbc7
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299725"
---
# <a name="send-and-receive-attachments"></a>Envío y recepción de datos adjuntos

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

Un intercambio de mensajes entre el usuario y el bot puede contener datos adjuntos con elementos multimedia, como imágenes, vídeo, audio y archivos. Los tipos de datos adjuntos que se pueden enviar varía según el canal, pero estos son los tipos básicos:

* **Elementos multimedia y archivos**: puede enviar archivos, como imágenes, audio y vídeos, estableciendo **contentType** en el tipo MIME del [objeto IAttachment][IAttachment] y, a continuación, pasando un vínculo al archivo en **contentUrl**.
* **Tarjetas**: puede enviar un amplio conjunto de tarjetas visuales <!-- and custom keyboards --> estableciendo **contentType** en el tipo de tarjeta deseado y, a continuación, pasando el código JSON de la tarjeta. Si usa una de las clases de generador de tarjetas enriquecidas, como **HeroCard**, los adjuntos se rellenan automáticamente. Consulte [envío de una tarjeta enriquecida](bot-builder-nodejs-send-rich-cards.md) para obtener un ejemplo de esto.

## <a name="add-a-media-attachment"></a>Incorporación de datos adjuntos con elementos multimedia
Se espera que el objeto del mensaje sea una instancia de [IMessage][IMessage] y lo más útil resulta enviar al usuario un mensaje como objeto cuando desea incluir adjuntos, como una imagen. Use el método [session.send()][SessionSend] para enviar mensajes en forma de objeto JSON. 

## <a name="example"></a>Ejemplo

En el ejemplo siguiente se comprueba si el usuario ha enviado un adjunto, y si lo hizo, devolverá la imagen contenida en el adjunto. Puede probar esto con Bot Framework Emulator; para ello, envíe una imagen al bot.

```javascript
// Create your bot with a function to receive messages from the user
var bot = new builder.UniversalBot(connector, function (session) {
    var msg = session.message;
    if (msg.attachments && msg.attachments.length > 0) {
     // Echo back attachment
     var attachment = msg.attachments[0];
        session.send({
            text: "You sent:",
            attachments: [
                {
                    contentType: attachment.contentType,
                    contentUrl: attachment.contentUrl,
                    name: attachment.name
                }
            ]
        });
    } else {
        // Echo back users text
        session.send("You said: %s", session.message.text);
    }
});
```
## <a name="additional-resources"></a>Recursos adicionales

* [Preview features with the Channel Inspector][inspector] (Vista previa de las características con el Inspector de canales)
* [IMessage][IMessage]
* [Envío de una tarjeta enriquecida][SendRichCard]
* [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[SendRichCard]: bot-builder-nodejs-send-rich-cards.md
[SessionSend]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send
[IAttachment]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iattachment.html
[inspector]: ../bot-service-channel-inspector.md

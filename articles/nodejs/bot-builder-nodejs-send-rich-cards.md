---
title: Incorporación de datos adjuntos de tarjetas enriquecidas a mensajes | Microsoft Docs
description: Obtenga información sobre cómo enviar tarjetas enriquecidas interactivas y atractivas mediante Bot Framework SDK para Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: cba67dc4da5a0b505b4f91f9cbf7fbc0a47b8974
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404802"
---
# <a name="add-rich-card-attachments-to-messages"></a>Incorporación de datos adjuntos de tarjetas enriquecidas a mensajes

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]


> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

Varios canales, como Skype y Facebook, admiten el envío de tarjetas gráficas enriquecidas a los usuarios con botones interactivos en los cuales el usuario hace clic para iniciar una acción. El SDK proporciona varias clases de generadores de tarjetas y mensajes que se pueden usar para crear y enviar tarjetas. El servicio Bot Framework Connector representará estas tarjetas con un esquema nativo para el canal, compatible con la comunicación multiplataforma. Si el canal no es compatible con las tarjetas, como SMS, Bot Framework hará todo lo posible para representar una experiencia razonable a los usuarios. 

## <a name="types-of-rich-cards"></a>Tipos de tarjetas enriquecidas 
Bot Framework admite actualmente ocho tipos de tarjetas enriquecidas: 

| Tipo de tarjeta | DESCRIPCIÓN |
|------|------|
| <a href="/adaptive-cards/get-started/bots">Tarjeta adaptable</a> | Una tarjeta personalizable que puede contener cualquier combinación de texto, voz, imágenes, botones y campos de entrada.  Consulte la [compatibilidad por canal](/adaptive-cards/get-started/bots#channel-status). |
| [Tarjeta de animación][animationCard] | Una tarjeta que puede reproducir archivos GIF animados o vídeos cortos. |
| [Tarjeta de audio][audioCard] | Una tarjeta que se puede reproducir un archivo de audio. |
| [Tarjeta de héroe][heroCard] | Una tarjeta que normalmente contiene una sola imagen grande, uno o varios botones y texto. |
| [Tarjeta de miniatura][thumbnailCard] | Una tarjeta que normalmente contiene una sola imagen miniatura, uno o varios botones, y texto.|
| [Tarjeta de recepción][receiptCard] | Una tarjeta que permite que un bot proporcione un recibo al usuario. Normalmente, contiene la lista de elementos que se incluyen en el recibo, la información de impuestos y del total, y texto adicional. |
| [Tarjeta de inicio de sesión][signinCard] | Una tarjeta que permite al bot solicitar que un usuario inicie sesión. Normalmente contiene texto y uno o varios botones en los que el usuario puede hacer clic para iniciar el proceso de inicio de sesión. |
| [Tarjeta de videollamada][videoCard] | Una tarjeta que puede reproducir vídeos. |

## <a name="send-a-carousel-of-hero-cards"></a>Enviar un carrusel de tarjetas de imagen prominente
En el ejemplo siguiente se muestra un bot para una compañía ficticia de camisetas y cómo enviar un carrusel de tarjetas en respuesta a la solicitud del usuario para "mostrar camisetas". 

```javascript
// Create your bot with a function to receive messages from the user
// Create bot and default message handler
var bot = new builder.UniversalBot(connector, function (session) {
    session.send("Hi... We sell shirts. Say 'show shirts' to see our products.");
});

// Add dialog to return list of shirts available
bot.dialog('showShirts', function (session) {
    var msg = new builder.Message(session);
    msg.attachmentLayout(builder.AttachmentLayout.carousel)
    msg.attachments([
        new builder.HeroCard(session)
            .title("Classic White T-Shirt")
            .subtitle("100% Soft and Luxurious Cotton")
            .text("Price is $25 and carried in sizes (S, M, L, and XL)")
            .images([builder.CardImage.create(session, 'http://petersapparel.parseapp.com/img/whiteshirt.png')])
            .buttons([
                builder.CardAction.imBack(session, "buy classic white t-shirt", "Buy")
            ]),
        new builder.HeroCard(session)
            .title("Classic Gray T-Shirt")
            .subtitle("100% Soft and Luxurious Cotton")
            .text("Price is $25 and carried in sizes (S, M, L, and XL)")
            .images([builder.CardImage.create(session, 'http://petersapparel.parseapp.com/img/grayshirt.png')])
            .buttons([
                builder.CardAction.imBack(session, "buy classic gray t-shirt", "Buy")
            ])
    ]);
    session.send(msg).endDialog();
}).triggerAction({ matches: /^(show|list)/i });
```
En este ejemplo se usa la clase [Message][Message] para compilar un carrusel.  
El carrusel está formado por una lista de clases [HeroCard][heroCard] que contienen una imagen, texto y un solo botón que desencadena la compra del elemento.  
Al hacer clic en el botón **Comprar**, se desencadena el envío de un mensaje, por lo que se debe agregar un segundo diálogo para detectar el clic del botón. 

## <a name="handle-button-input"></a>Controlar la entrada del botón

El diálogo `buyButtonClick` se desencadenará cada vez que se reciba un mensaje que comience por "comprar" o "agregar" y, después, contenga algún texto que incluya la palabra "camiseta". El diálogo se inicia mediante el uso de un par de expresiones regulares para buscar la camisa de tamaño opcional y el color que haya pedido el usuario.
Esta flexibilidad añadida le permite admitir los clics de botón y los mensajes de lenguaje natural del usuario, como "agrega una camisa gris de tamaño grande a mi carro de la compra".
Si el color es válido, pero se desconoce el tamaño, el bot pide al usuario que elija un tamaño de una lista antes de agregar el artículo al carro de la compra. Una vez que el bot tiene toda la información que necesita, coloca el artículo en un carro de la compra que se ha conservado con **session.userData** y, a continuación, envía al usuario un mensaje de confirmación.

```javascript
// Add dialog to handle 'Buy' button click
bot.dialog('buyButtonClick', [
    function (session, args, next) {
        // Get color and optional size from users utterance
        var utterance = args.intent.matched[0];
        var color = /(white|gray)/i.exec(utterance);
        var size = /\b(Extra Large|Large|Medium|Small)\b/i.exec(utterance);
        if (color) {
            // Initialize cart item
            var item = session.dialogData.item = { 
                product: "classic " + color[0].toLowerCase() + " t-shirt",
                size: size ? size[0].toLowerCase() : null,
                price: 25.0,
                qty: 1
            };
            if (!item.size) {
                // Prompt for size
                builder.Prompts.choice(session, "What size would you like?", "Small|Medium|Large|Extra Large");
            } else {
                //Skip to next waterfall step
                next();
            }
        } else {
            // Invalid product
            session.send("I'm sorry... That product wasn't found.").endDialog();
        }   
    },
    function (session, results) {
        // Save size if prompted
        var item = session.dialogData.item;
        if (results.response) {
            item.size = results.response.entity.toLowerCase();
        }

        // Add to cart
        if (!session.userData.cart) {
            session.userData.cart = [];
        }
        session.userData.cart.push(item);

        // Send confirmation to users
        session.send("A '%(size)s %(product)s' has been added to your cart.", item).endDialog();
    }
]).triggerAction({ matches: /(buy|add)\s.*shirt/i });
```

<!-- 

> [!NOTE]
> When sending a message that contains images, keep in mind that some channels download images before displaying a message to the user.   
> As a result, a message containing an image followed immediately by a message without images may sometimes be flipped in the user's feed.
> For information on how to avoid messages being sent out of order, see [Message ordering][MessageOrder].  

-->
## <a name="add-a-message-delay-for-image-downloads"></a>Agregar un retraso de mensaje para descargas de imágenes
Algunos canales tienden a descargar imágenes antes de mostrar un mensaje al usuario, de modo que si envía un mensaje que contiene una imagen inmediatamente seguida de un mensaje sin imágenes, a veces verá los mensajes volteados en la fuente del usuario. Para minimizar la posibilidad de que esto ocurra, puede intentar asegurarse de que sus imágenes procedan de redes de entrega de contenido (CDN) y evitar el uso de imágenes demasiado grandes. En casos extremos incluso es posible que necesite insertar un retraso de 1 a 2 segundos entre el mensaje con la imagen y el siguiente. Puede hacer que este retraso parezca un poco más natural para el usuario mediante una llamada a **session.sendTyping()** para enviar un indicador de escritura antes de iniciar el retraso. 

<!-- 
To learn more about sending a typing indicator, see [How to send a typing indicator](bot-builder-nodejs-send-typing-indicator.md).
-->

Bot Framework implementa un procesamiento por lotes para intentar evitar varios mensajes del bot que muestren que no está en funcionamiento. <!-- Unfortunately, not all channels can guarantee this. --> Cuando su bot envía varias respuestas al usuario, los mensajes individuales se agrupan automáticamente en un lote y se entregan al usuario como un conjunto en un esfuerzo por conservar el orden original de los mensajes. Este procesamiento por lotes automático espera un tiempo predeterminado de 250 ms después de cada llamada a **session.send()** antes de iniciar la siguiente llamada a **send()** .

El retraso de procesamiento por lotes del mensaje se puede configurar. Para deshabilitar la lógica del procesamiento por lotes automático del SDK, establezca el retraso predeterminado en un número grande y, a continuación, llame manualmente a **sendBatch()** con una devolución de llamada para realizar la invocación una vez que se haya entregado el lote.

## <a name="send-an-adaptive-card"></a>Enviar una tarjeta adaptable

Una tarjeta adaptable puede contener cualquier combinación de texto, voz, imágenes, botones y campos de entrada. Las tarjetas adaptables se crean con el formato JSON especificado en <a href="http://adaptivecards.io" target="_blank">Adaptive Cards</a> (Tarjetas adaptables), que proporciona control total sobre el contenido y el formato de la tarjeta. 

Para crear una tarjeta adaptable con Node.js, utilice la información del sitio <a href="http://adaptivecards.io" target="_blank">Adaptive Cards</a> (Tarjetas adaptables) a fin de comprender el esquema de la tarjeta adaptable, explorar los elementos de la tarjeta adaptable y ver ejemplos de JSON que puede usar para crear tarjetas de diferente composición y complejidad. Además, puede usar el visualizador interactivo para diseñar cargas útiles de la tarjeta adaptable y obtener una vista previa del resultado de la tarjeta.

En este ejemplo de código se muestra cómo crear un mensaje que contiene una tarjeta adaptable para un recordatorio del calendario: 

[!code-javascript[Add Adaptive Card attachment](../includes/code/node-send-card-buttons.js#addAdaptiveCardAttachment)]

La tarjeta resultante contiene tres bloques de texto, un campo de entrada (lista de opciones) y tres botones:

![Recordatorio del calendario de tarjeta adaptable](../media/adaptive-card-reminder.png)

## <a name="additional-resources"></a>Recursos adicionales

* [Preview features with the Channel Inspector][inspector] (Vista previa de las características con el Inspector de canales)
* <a href="http://adaptivecards.io" target="_blank">Tarjetas adaptables</a>
* [AnimationCard][animationCard]
* [AudioCard][audioCard]
* [HeroCard][heroCard]
* [ThumbnailCard][thumbnailCard]
* [ReceiptCard][receiptCard]
* [SigninCard][signinCard]
* [VideoCard][videoCard]
* [Mensaje][Message]
* [How to send attachments](bot-builder-nodejs-send-receive-attachments.md) (Cómo enviar datos adjuntos)

[MessageOrder]: bot-builder-nodejs-manage-conversation-flow.md#message-ordering
[Message]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message
[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[animationCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.animationcard.html 

[audioCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.audiocard.html 

[heroCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.herocard.html

[thumbnailCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.thumbnailcard.html 

[receiptCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.receiptcard.html 

[signinCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.signincard.html 

[videoCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.videocard.html

[inspector]: ../bot-service-channel-inspector.md

---
title: Incorporación de voz a los mensajes | Microsoft Docs
description: Obtenga información sobre cómo agregar voz a los mensajes mediante Bot Builder SDK para Node.js.
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3c17097197ba4b6ed0523d84a81974d9cc9fe3b5
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999138"
---
# <a name="add-speech-to-messages"></a>Incorporación de voz a los mensajes

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.js](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

Si está creando un bot para un canal habilitado para voz, como Cortana, puede construir mensajes que especifiquen el texto que dirá el bot. Para intentar influir en el estado del micrófono del cliente, puede especificar una [sugerencia de entrada](bot-builder-nodejs-send-input-hints.md) para indicar si el bot acepta, espera o ignora la entrada del usuario.

## <a name="specify-text-to-be-spoken-by-your-bot"></a>Especificación del texto que dirá el bot

Mediante Bot Builder SDK para Node.js, hay varias maneras de especificar el texto que dirá el bot en un canal habilitado para voz. Puede establecer la propiedad `IMessage.speak` y enviar el mensaje con el método `session.send()`, enviar el mensaje con el método `session.say()` (pasando los parámetros que especifican el texto para mostrar, el texto para hablar y las opciones) o enviar uno de los avisos integrados (especificando las opciones `speak` y `retrySpeak`).

### <a id="message-speak"></a> IMessage.speak 

Si está creando un mensaje que se va a enviar mediante el método `session.send()`, establezca la propiedad `speak` para especificar el texto que dirá el bot. El siguiente ejemplo de código crea un mensaje que especifica el texto que se dirá e indica que el bot [espera la entrada del usuario](bot-builder-nodejs-send-input-hints.md).

[!code-javascript[IMessage.speak](../includes/code/node-text-to-speech.js#IMessageSpeak)]

### <a id="session-say"></a> session.say()

Como alternativa al uso de `session.send()`, puede llamar al método `session.say()` para crear y enviar un mensaje que especifica el texto que se dirá, además del texto que se mostrará y otras opciones. El método se define como sigue:

`session.say(displayText: string, speechText: string, options?: object)`

| Parámetro | DESCRIPCIÓN |
|----|----|
| `displayText` | Texto para mostrar. |
| `speechText` | El texto (en texto sin formato o en formato <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">SSML</a>) que el bot dirá. |
| `options` | Un objeto [IMessage][IMessage] que puede contener datos adjuntos o una [sugerencia de entrada](bot-builder-nodejs-send-input-hints.md). |

El siguiente ejemplo de código envía un mensaje que especifica el texto que se mostrará y el texto que se dirá e indica que el bot [ignora la entrada del usuario](bot-builder-nodejs-send-input-hints.md).

[!code-javascript[Session.say()](../includes/code/node-text-to-speech.js#SessionSay)]

### <a id="prompt-options"></a> Opciones de aviso

Con cualquiera de los avisos integrados, puede establecer las opciones `speak` y `retrySpeak` para especificar el texto que dirá el bot. En el ejemplo de código siguiente se crea un aviso que especifica el texto que se mostrará, el texto que se dirá inicialmente y el texto que se dirá después de esperar un tiempo los datos de entrada del usuario. Indica que el bot [espera la entrada del usuario](bot-builder-nodejs-send-input-hints.md) y usa el formato [SSML](#ssml) para especificar que la palabra "sure" (seguro) se debe decir con un énfasis moderado.

[!code-javascript[Prompt](../includes/code/node-text-to-speech.js#Prompt)]

## <a id="ssml"></a> Lenguaje de marcado de síntesis de voz (SSML)

Para especificar el texto que dirá el bot, puede usar una cadena de texto sin formato o en una cadena con formato de lenguaje de marcado de síntesis de voz (SSML), un lenguaje de marcado basado en XML que le permite controlar diversas características de la voz del bot, como la voz, la velocidad, el volumen, la pronunciación, el tono y mucho más. Para más información acerca de SSML, consulte <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">Referencia del lenguaje de marcado de síntesis de voz</a>.

> [!TIP]
> Use una <a href="https://www.npmjs.com/search?q=ssml" target="_blank">biblioteca de SSML</a> para crear SSML con el formato correcto.

## <a name="input-hints"></a>Sugerencias de entrada

Cuando envía un mensaje en un canal habilitado para voz, puede intentar influir en el estado del micrófono del cliente e incluir una sugerencia de entrada para indicar si el bot acepta, espera o ignora la entrada del usuario. Para más información, consulte [Incorporación de sugerencias de entrada a los mensajes](bot-builder-nodejs-send-input-hints.md).

## <a name="sample-code"></a>Código de ejemplo 

Para un ejemplo completo en el que se muestra cómo crear un bot habilitado para voz mediante Bot Builder SDK para. NET, consulte el ejemplo <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill" target="_blank">Roller</a> en GitHub.

## <a name="additional-resources"></a>Recursos adicionales

- <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">Lenguaje de marcado de síntesis de voz (SSML)</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill" target="_blank">Ejemplo Roller (GitHub)</a>
- [Referencia de Bot Builder SDK para Node.js][SDKReference]

[SDKReference]: https://docs.botframework.com/en-us/node/builder/chat-reference/modules/_botbuilder_d_.html

[Message]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

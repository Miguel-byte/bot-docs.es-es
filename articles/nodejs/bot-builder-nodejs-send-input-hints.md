---
title: Incorporación de sugerencias de entrada a los mensajes | Microsoft Docs
description: Aprenda a agregar sugerencias de entrada a los mensajes mediante Bot Framework SDK para .NET.
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 513a5aa5b71bf8ec051a26027a56e67c01b1784a
ms.sourcegitcommit: 4c5c08e7c7eaa5f74c6ac35d8478954b998625f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2019
ms.locfileid: "64906291"
---
# <a name="add-input-hints-to-messages"></a>Incorporación de sugerencias de entrada a los mensajes

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-input-hints.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-input-hints.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-input-hints.md)

Al especificar una sugerencia de entrada para un mensaje, puede indicar si el bot acepta, espera o ignora la entrada del usuario después de que el mensaje se haya entregado al cliente. En muchos canales, esto permite a los clientes establecer en consecuencia el estado de los controles de entrada de usuario. Por ejemplo, si la sugerencia de entrada de un mensaje indica que el bot ignora la entrada de usuario, el cliente podría desactivar el micrófono y deshabilitar el cuadro de entrada para impedir que el usuario proporcione una entrada.

## <a name="accepting-input"></a>Aceptar entradas

Para indicar que el bot está preparado pasivamente para la entrada, pero que no espera ninguna respuesta del usuario, establezca la sugerencia de entrada del mensaje en `builder.InputHint.acceptingInput`. En muchos canales, esto hará que se habilite el cuadro de entrada del cliente y que el micrófono se desactive, sin dejar de estar accesible para el usuario. Por ejemplo, Cortana activará el micrófono para aceptar la entrada del usuario si este mantiene presionado el botón de micrófono. En el ejemplo de código siguiente se crea un mensaje que indica que el bot acepta la entrada del usuario.

[!code-javascript[IMessage.speak](../includes/code/node-input-hints.js#InputHintAcceptingInput)]

## <a name="expecting-input"></a>Esperar entradas

Para indicar que el bot espera una respuesta del usuario, establezca la sugerencia de entrada del mensaje en `builder.InputHint.expectingInput`. En muchos canales, esto hará que se habilite el cuadro de entrada del cliente y que el micrófono se active. En el ejemplo de código siguiente se crea un aviso que indica que el bot espera los datos de entrada del usuario.

[!code-javascript[Prompt](../includes/code/node-input-hints.js#InputHintExpectingInput)]

## <a name="ignoring-input"></a>Ignorar entradas

Para indicar que el bot no está preparado para recibir la entrada del usuario, establezca la sugerencia de entrada del mensaje en `builder.InputHint.ignoringInput`. En muchos canales, esto hará que se deshabilite el cuadro de entrada del cliente y que el micrófono se desactive. El ejemplo de código siguiente usa el método `session.say()` para enviar un mensaje que indica que el bot ignora la entrada del usuario.

[!code-javascript[Session.say()](../includes/code/node-input-hints.js#InputHintIgnoringInput)]

## <a name="default-values-for-input-hint"></a>Valores predeterminados para las sugerencias de entrada

Si no establece la sugerencia de entrada para un mensaje, Bot Framework SDK la establecerá automáticamente mediante esta lógica: 

- Si el bot envía un aviso, en la sugerencia de datos de entrada del mensaje se especificará que el bot **espera datos de entrada**.</li>
- Si el bot envía un solo mensaje, en la sugerencia de entrada del mensaje se especificará que el bot **acepta entradas**.</li>
- Si el bot envía una serie de mensajes consecutivos, la sugerencia de entrada de todos los mensajes excepto el último de la serie especificará que el bot **ignora las entradas**, mientras que la sugerencia de entrada del último mensaje de la serie especificará que el bot **acepta entradas**.

## <a name="additional-resources"></a>Recursos adicionales

- [Incorporación de voz a los mensajes](bot-builder-nodejs-text-to-speech.md)

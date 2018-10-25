---
title: Agregar acciones sugeridas a los mensajes | Microsoft Docs
description: Obtenga información sobre cómo agregar acciones sugeridas a los mensajes mediante el SDK de Bot Builder para Node.js.
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 06/06/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1be5423de3859b32416b5acf9ba7b9d8e92ecb14
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997252"
---
# <a name="add-suggested-actions-to-messages"></a>Incorporación de acciones sugeridas a mensajes

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

> [!TIP]
> Use el [Inspector de canales][channelInspector] para ver la apariencia de las acciones sugeridas y cómo funcionan en varios canales.

## <a name="suggested-actions-example"></a>Ejemplo de acciones sugeridas

Para agregar acciones sugeridas a un mensaje, establezca la propiedad `suggestedActions` del mensaje en una lista de [acciones de tarjeta][ICardAction] que representa los botones que se presentarán al usuario.

Este ejemplo de código muestra cómo enviar un mensaje que presenta tres acciones sugeridas al usuario:

[!code-javascript[Send suggested actions](../includes/code/node-send-suggested-actions.js#sendSuggestedActions)]

Cuando el usuario pulsa en una de las acciones sugeridas, el bot recibirá un mensaje del usuario que contiene el elemento `value` de la acción correspondiente.

Tenga en cuenta que el método `imBack` registrará el elemento `value` en la ventana de chat del canal que use. Si este no es el efecto deseado, puede usar el método `postBack`, que seguirá publicando la selección en su bot, pero no mostrará la selección en la ventana de chat. Sin embargo, algunos canales no admiten `postBack` y, en esos casos, el método se comportará como `imBack`.

## <a name="additional-resources"></a>Recursos adicionales

* [Preview features with the Channel Inspector][inspector] (Vista previa de las características con el Inspector de canales)
* [IMessage][IMessage]
* [ICardAction][ICardAction]
* [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[SessionSend]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send

[ICardAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.icardaction.html

[inspector]: ../bot-service-channel-inspector.md

[channelInspector]: ../bot-service-channel-inspector.md

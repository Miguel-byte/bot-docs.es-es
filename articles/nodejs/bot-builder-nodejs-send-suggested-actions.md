---
title: Incorporación de acciones sugeridas a mensajes | Microsoft Docs
description: Aprenda a enviar acciones sugeridas dentro de mensajes mediante Bot Framework SDK para Node.js.
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/19/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 73193b82c8f03a1a49df1927ced15684bd7af955
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "64564153"
---
# <a name="add-suggested-actions-to-messages"></a>Incorporación de acciones sugeridas a mensajes

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

## <a name="suggested-actions-example"></a>Ejemplo de acciones sugeridas

Para agregar acciones sugeridas a un mensaje, establezca la propiedad `suggestedActions` del mensaje en una lista de [acciones de tarjeta][ICardAction] que representa los botones que se presentarán al usuario.

Este ejemplo de código muestra cómo enviar un mensaje que presenta tres acciones sugeridas al usuario:

[!code-javascript[Send suggested actions](../includes/code/node-send-suggested-actions.js#sendSuggestedActions)]

Cuando el usuario pulsa en una de las acciones sugeridas, el bot recibirá un mensaje del usuario que contiene el elemento `value` de la acción correspondiente.

Tenga en cuenta que el método `imBack` registrará el elemento `value` en la ventana de chat del canal que use. Si este no es el efecto deseado, puede usar el método `postBack`, que seguirá publicando la selección en su bot, pero no mostrará la selección en la ventana de chat. Sin embargo, algunos canales no admiten `postBack` y, en esos casos, el método se comportará como `imBack`.

## <a name="additional-resources"></a>Recursos adicionales

- [Ejemplos][samples]
- [IMessage][IMessage]
- [ICardAction][ICardAction]
- [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[SessionSend]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send

[ICardAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.icardaction.html

<!-- The inspector is no longer supported: we're redirecting to the samples for now. -->
[samples]: https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples

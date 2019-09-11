---
title: Control de eventos de usuario y conversación | Microsoft Docs
description: Aprenda a controlar eventos como la unión de un usuario a una conversación mediante Bot Framework SDK para Node.js.
author: DucVo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 074868cfe6cd8bb2317d8c3cae1a814d9906a0c0
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299848"
---
# <a name="handle-user-and-conversation-events"></a>Control de eventos de usuario y conversación

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

En este artículo se muestra cómo el bot puede controlar eventos, como la unión de un usuario a una conversación, agregar un bot a una lista de contactos o decir **Adiós** cuando se quita un bot de una conversación.


## <a name="greet-a-user-on-conversation-join"></a>Saludo a un usuario al unirse a la conversación
Bot Framework proporciona el evento [conversationUpdate][conversationUpdate] para notificar a su bot cada vez que un miembro se une a una conversación o la abandona. Un miembro de la conversación puede ser un usuario o un bot.

El siguiente fragmento de código permite al bot saludar a los nuevos miembros de una conversación o decir **Adiós** si se quitan de la conversación.

[!INCLUDE [conversationUpdate sample Node.js](../includes/snippet-code-node-conversationupdate-1.md)]

## <a name="acknowledge-add-to-contacts-list"></a>Confirmación de incorporación a la lista de contactos

El evento [contactRelationUpdate][contactRelationUpdate] notifica al bot que un usuario le ha agregado a su lista de contactos.

[!INCLUDE [contactRelationUpdate sample Node.js](../includes/snippet-code-node-contactrelationupdate-1.md)]

## <a name="add-a-first-run-dialog"></a>Incorporación de un primer diálogo de ejecución

Dado que los eventos **conversationUpdate** y **contactRelationUpdate** no se admiten en todos los canales, una manera universal de saludar a un usuario que se une a una conversación es agregar un primer diálogo de ejecución.

En el ejemplo siguiente, agregamos una función que desencadena el diálogo cada vez que nunca hemos visto antes a un usuario. Puede personalizar la manera en que se desencadena una acción si proporciona un controlador [onFindAction][onFindAction] para la acción. 

[!INCLUDE [first-run sample Node.js](../includes/snippet-code-node-first-run-dialog-1.md)]

También puede personalizar lo que hace una acción después de que se ha desencadenado si proporciona un controlador [onSelectAction][onSelectAction]. Para desencadenar acciones, puede proporcionar un controlador [onInterrupted][onInterrupted] para interceptar una interrupción antes de que ocurra. Para más información, consulte [Control de acciones del usuario](bot-builder-nodejs-dialog-actions.md).

## <a name="additional-resources"></a>Recursos adicionales

* [conversationUpdate][conversationUpdate]
* [contactRelationUpdate][contactRelationUpdate]

[conversationUpdate]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iconversationupdate.html
[contactRelationUpdate]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.icontactrelationupdate.html

[onFindAction]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#onfindaction
[onSelectAction]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#onselectaction
[onInterrupted]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#oninterrupted

[SendTyping]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping
[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[ChatConnector]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[session_userData]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata

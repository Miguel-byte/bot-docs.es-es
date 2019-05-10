---
title: Incorporación de acciones sugeridas a mensajes | Microsoft Docs
description: Obtenga información sobre cómo agregar acciones sugeridas a mensajes mediante el servicio Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 346c522a1e1cd3d5904b74ad6e6092a1457a001c
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032591"
---
# <a name="add-suggested-actions-to-messages"></a>Incorporación de acciones sugeridas a mensajes
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

## <a name="send-suggested-actions"></a>Envío de acciones sugeridas

Para agregar acciones sugeridas a un mensaje, establezca la propiedad `suggestedActions` de la [actividad][Activity] para especificar la lista de objetos [CardAction][CardAction] que representan los botones que se presentarán al usuario. 

La siguiente solicitud envía un mensaje que presenta tres acciones sugeridas al usuario. En esta solicitud de ejemplo, `https://smba.trafficmanager.net/apis` representa el URI base; los URI base para las solicitudes que emite el bot pueden ser diferentes. Para más información sobre cómo establecer el URI base, vea [Referencia de la API](bot-framework-rest-connector-api-reference.md#base-uri).

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/5d5cdc723
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "I have colors in mind, but need your help to choose the best one.",
    "inputHint": "expectingInput",
    "suggestedActions": {
        "actions": [
            {
                "type": "imBack",
                "title": "Blue",
                "value": "Blue"
            },
            {
                "type": "imBack",
                "title": "Red",
                "value": "Red"
            },
            {
                "type": "imBack",
                "title": "Green",
                "value": "Green"
            }
        ]
    },
    "replyToId": "5d5cdc723"
}
```

Cuando el usuario pulsa en una de las acciones sugeridas, el bot recibirá un mensaje del usuario que contiene el elemento `value` de la acción correspondiente.

## <a name="additional-resources"></a>Recursos adicionales

- [Creación de mensajes](bot-framework-rest-connector-create-messages.md)
- [Envío y recepción de mensajes](bot-framework-rest-connector-send-and-receive-messages.md)

[channelInspector]: ../bot-service-channel-inspector.md

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object

[CardAction]: bot-framework-rest-connector-api-reference.md#cardaction-object

[SuggestedAction]: bot-framework-rest-connector-api-reference.md#suggestedactions-object

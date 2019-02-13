---
title: Finalización de una conversación | Microsoft Docs
description: Aprenda a finalizar una conversación utilizando Direct Line API v3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 438558995f83ade38404856d61ba66ee77480a27
ms.sourcegitcommit: 32615b88e4758004c8c99e9d564658a700c7d61f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/04/2019
ms.locfileid: "55711939"
---
# <a name="end-a-conversation"></a>Finalización de una conversación

El valor **endOfConversation** de [activity](bot-framework-rest-connector-activities.md) significa que el canal o el bot han finalizado la conversación. 

> [!NOTE] 
> Aunque solo unos pocos canales envían el evento **endOfConversation**, el canal Cortana es el único que lo acepta. Otros canales, como Direct Line, no implementan esta funcionalidad y, en su lugar, terminan o reenvían la actividad; cada canal determina cómo debe reaccionar a una actividad endOfConversation. Si está diseñando un cliente de DirectLine, debe actualizar el cliente para que se comporte de forma adecuada, como generar un error si el bot envía una actividad a una conversación que ya ha finalizado.

## <a name="send-an-endofconversation-activity"></a>Envío de una actividad endOfConversation

Una actividad **endOfConversation** finaliza la comunicación entre bot y cliente. Después de que se haya enviado una actividad **endOfConversation**, el cliente aún puede [recuperar mensajes](bot-framework-rest-direct-line-3-0-receive-activities.md#http-get) mediante `HTTP GET`, pero ni el cliente ni el bot pueden enviar ningún mensaje adicional a la conversación. 

Para finalizar una conversación, solo tiene que emitir una solicitud POST para enviar una actividad **endOfConversation**.

### <a name="request"></a>Solicitud

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
[other headers]
```

```json
{
    "type": "endOfConversation",
    "from": {
        "id": "user1"
    }
}
```

### <a name="response"></a>Response

Si la solicitud es correcta, la respuesta contendrá un identificador para la actividad que se envió.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "id": "0004"
}
```

## <a name="additional-resources"></a>Recursos adicionales

- [Conceptos clave](bot-framework-rest-direct-line-3-0-concepts.md)
- [Autenticación](bot-framework-rest-direct-line-3-0-authentication.md)
- [Envío de una actividad al bot](bot-framework-rest-direct-line-3-0-send-activity.md)

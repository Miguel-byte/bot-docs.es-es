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
ms.openlocfilehash: f0985f28fd1744bcfb6bf5cea1c2230254670e01
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000212"
---
# <a name="end-a-conversation"></a>Finalización de una conversación

Un cliente o un bot pueden indicar el final de una conversación de Direct Line enviando una [actividad](bot-framework-rest-connector-activities.md) **endOfConversation**. 

> [!NOTE] 
> El evento endOfConversation solo se admite en el canal de Cortana; otros canales no implementan esta funcionalidad. Cada canal determina cómo debe reaccionar ante una actividad endOfConversation. Si está diseñando un cliente de DirectLine, debe actualizar el cliente para que se comporte de forma adecuada, como generar un error si el bot envía una actividad a una conversación que ya ha finalizado.

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

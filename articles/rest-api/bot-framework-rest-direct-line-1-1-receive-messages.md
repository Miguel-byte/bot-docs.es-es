---
title: Recepción de mensajes del bot | Microsoft Docs
description: Aprenda a recibir mensajes del bot mediante Direct Line API v1.1.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 6f9a132b538a278b0990271864a70e77ea7dc56c
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299624"
---
# <a name="receive-messages-from-the-bot"></a>Recepción de mensajes del bot

> [!IMPORTANT]
> En este artículo se describe cómo recibir mensajes del bot mediante Direct Line API v1.1. Si va a crear una nueva conexión entre la aplicación cliente y el bot, use [Direct Line API 3.0](bot-framework-rest-direct-line-3-0-receive-activities.md) en su lugar.

Mediante el protocolo Direct Line 1.1, los clientes deben sondear una interfaz `HTTP GET` para recibir mensajes. 

## <a name="retrieve-messages-with-http-get"></a>Recuperación de mensajes con HTTP GET

Para recuperar mensajes de una conversación específica, emita una solicitud `GET` al punto de conexión `api/conversations/{conversationId}/messages` y, si lo desea, especifique el parámetro `watermark` para indicar el mensaje más reciente visto por el cliente. Se devolverá un valor `watermark` en la respuesta JSON, incluso si no se incluyen mensajes.

Los fragmentos de código siguientes proporcionan un ejemplo de la solicitud Get Messages y la respuesta. Contiene la respuesta de Get Messages `watermark` como una propiedad de [MessageSet](bot-framework-rest-direct-line-1-1-api-reference.md#messageset-object). Los clientes deben desplazarse por los mensajes disponibles haciendo avanzar el valor `watermark` hasta que no se devuelvan mensajes. 

### <a name="request"></a>Solicitud

```http
GET https://directline.botframework.com/api/conversations/abc123/messages?watermark=0001a-94
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>Response

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "messages": [
        {
            "conversation": "abc123",
            "id": "abc123|0000",
            "text": "hello",
            "from": "user1"
        }, 
        {
            "conversation": "abc123",
            "id": "abc123|0001",
            "text": "Nice to see you, user1!",
            "from": "bot1"
        }
    ],
    "watermark": "0001a-95"
}
```

## <a name="timing-considerations"></a>Consideraciones de tiempo

Aunque Direct Line es un protocolo de varias partes con posibles diferencias temporales, el protocolo y el servicio están diseñados para que sea fácil crear un cliente confiable. La propiedad `watermark` que se envía en la respuesta de Get Messages es confiable. Un cliente no perderá ningún mensaje siempre que se reproduzca la marca de agua textualmente.

Los clientes deben elegir un intervalo de sondeo que coincida con su uso previsto.

- Las aplicaciones de servicio a servicio suelen usar un intervalo de sondeo de 5 o 10 s.

- Las aplicaciones orientadas al cliente a menudo usan un intervalo de sondeo de 1 segundo y emiten una solicitud adicional ~ 300 ms después de todos los mensajes que envía el cliente (para recuperar rápidamente la respuesta de un bot). Este retraso de 300 ms se debe ajustar en función del tiempo de tránsito y la velocidad del bot.

## <a name="additional-resources"></a>Recursos adicionales

- [Conceptos clave](bot-framework-rest-direct-line-1-1-concepts.md)
- [Autenticación](bot-framework-rest-direct-line-1-1-authentication.md)
- [Inicio de una conversación](bot-framework-rest-direct-line-1-1-start-conversation.md)
- [Envío de un mensaje al bot](bot-framework-rest-direct-line-1-1-send-message.md)
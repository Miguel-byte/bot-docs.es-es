---
title: Recepción de actividades del bot | Microsoft Docs
description: Aprenda a recibir actividades del bot mediante Direct Line API v3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: dd5e81ba3feaba09e60011c138dcbe1537144b5a
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/12/2019
ms.locfileid: "59541011"
---
# <a name="receive-activities-from-the-bot"></a>Recepción de actividades del bot

Con el protocolo Direct Line 3.0, los clientes pueden recibir actividades mediante una secuencia de `WebSocket` o recuperar actividades mediante la emisión de solicitudes `HTTP GET`. 

## <a name="websocket-vs-http-get"></a>WebSocket frente a HTTP GET

Un protocolo WebSocket de streaming inserta mensajes en los clientes, siempre y cuando la interfaz GET permita que los clientes soliciten mensajes explícitamente. Aunque con frecuencia se prefiere el mecanismo WebSocket dada su eficacia, el mecanismo GET puede ser útil para los clientes que no pueden usar WebSockets. 

No todos los [tipos de actividad](bot-framework-rest-connector-activities.md) están disponibles mediante WebSocket y HTTP GET. En la tabla siguiente se describe la disponibilidad de los distintos tipos de actividad para los clientes que usan el protocolo Direct Line.

| Tipo de actividad | Disponibilidad | 
|----|----|
| Mensaje | HTTP GET y WebSocket |
| typing | Solo WebSocket |
| conversationUpdate | No enviados o recibidos mediante el cliente |
| contactRelationUpdate | No se admite en Direct Line |
| endOfConversation | HTTP GET y WebSocket |
| todos los demás tipos de actividad | HTTP GET y WebSocket |

## <a id="connect-via-websocket"></a> Recepción de actividades mediante una secuencia de WebSocket

Cuando un cliente envía una solicitud [Iniciar conversación](bot-framework-rest-direct-line-3-0-start-conversation.md) para abrir una conversación con un bot, la respuesta del servicio incluye una propiedad `streamUrl` que el cliente puede usar posteriormente para conectarse por medio de WebSocket. La dirección URL de la secuencia se autoriza previamente y, por tanto, la solicitud del cliente para conectarse mediante WebSocket NO requiere un encabezado `Authorization`.

En el ejemplo siguiente se muestra una solicitud que usa `streamUrl` para conectarse por medio de WebSocket.

```http
-- connect to wss://directline.botframework.com --
GET /v3/directline/conversations/abc123/stream?t=RCurR_XV9ZA.cwA..."
Upgrade: websocket
Connection: upgrade
[other headers]
```

El servicio responde con el código de estado HTTP 101 ("Cambiando protocolos").

```http
HTTP/1.1 101 Switching Protocols
[other headers]
```

### <a name="receive-messages"></a>Recepción de mensajes

Después de conectarse mediante WebSocket, un cliente puede recibir estos tipos de mensajes del servicio Direct Line:

- Un mensaje que contiene un elemento [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object) que incluye una o varias actividades y una marca de agua (se describe a continuación).
- Un mensaje vacío, que usa el servicio Direct Line para garantizar que la conexión sigue siendo válida.
- Tipos adicionales, que se definirán más adelante. Estos tipos se identifican mediante las propiedades de la raíz JSON.

Un elemento `ActivitySet` contiene los mensajes que envían el bot y todos los usuarios de la conversación. En el ejemplo siguiente se muestra un elemento `ActivitySet` que contiene un único mensaje.

```json
{
    "activities": [
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0000",
            "from": {
                "id": "user1"
            },
            "text": "hello"
        }
    ],
    "watermark": "0000a-42"
}
```

### <a name="process-messages"></a>Procesamiento de mensajes

Un cliente debe realizar un seguimiento del valor `watermark` que recibe en cada [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object), de modo que pueda usar la marca de agua para garantizar que no se pierde ningún mensaje en caso de que se pierda la conexión y sea necesario [volver a conectarse a la conversación](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md). Si el cliente recibe un elemento `ActivitySet` donde la propiedad `watermark` falta o es `null`, debe omitir ese valor y no sobrescribir la marca de agua anterior que recibió.

Un cliente debe omitir los mensajes vacíos que recibe del servicio Direct Line.

Un cliente puede enviar mensajes vacíos al servicio Direct Line para comprobar la conectividad. El servicio Direct Line pasará por alto los mensajes vacíos que recibe del cliente.

El servicio Direct Line puede forzar el cierre de la conexión de WebSocket en determinadas condiciones. Si el cliente no ha recibido una actividad `endOfConversation`, es posible que [genere una nueva dirección URL de secuencia de WebSocket](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md) que puede usar para volver a conectarse a la conversación. 

La secuencia de WebSocket contiene actualizaciones directas y mensajes muy recientes (desde que se emitió la llamada para conectarse mediante WebSocket) pero no incluye los mensajes enviados antes de la solicitud `POST` más reciente a `/v3/directline/conversations/{id}`. Para recuperar los mensajes enviados anteriormente en la conversación, use `HTTP GET` tal como se describe a continuación.

## <a id="http-get"></a> Recuperación de actividades con HTTP GET

Los clientes que no pueden usar WebSockets pueden recuperar las actividades mediante `HTTP GET`.

Para recuperar mensajes de una conversación específica, emita una solicitud `GET` al punto de conexión `/v3/directline/conversations/{conversationId}/activities` y, si lo desea, especifique el parámetro `watermark` para indicar el mensaje más reciente visto por el cliente. 

Los fragmentos de código siguientes proporcionan un ejemplo de la solicitud y respuesta de Obtener actividades de conversación. La respuesta Obtener actividades de conversación contiene `watermark` como propiedad de [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object). Los clientes deben desplazarse por las actividades disponibles haciendo avanzar el valor `watermark` hasta que no se devuelvan actividades.

### <a name="request"></a>Solicitud

```http
GET https://directline.botframework.com/v3/directline/conversations/abc123/activities?watermark=0001a-94
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>Response

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "activities": [
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0000",
            "from": {
                "id": "user1"
            },
            "text": "hello"
        }, 
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0001",
            "from": {
                "id": "bot1"
            },
            "text": "Nice to see you, user1!"
        }
    ],
    "watermark": "0001a-95"
}
```

## <a name="timing-considerations"></a>Consideraciones de tiempo

La mayoría de los clientes desean conservar un historial completo de mensajes. Aunque Direct Line es un protocolo de varias partes con posibles diferencias temporales, el protocolo y el servicio están diseñados para que sea fácil crear un cliente confiable.

- La propiedad `watermark` que se envía en la secuencia de WebSocket y en la respuesta de Obtener actividades de conversación es confiable. Un cliente no perderá ningún mensaje siempre que se reproduzca la marca de agua textualmente.

- Cuando un cliente inicia una conversación y se conecta a la secuencia de WebSocket, las actividades que se envían después de POST, pero antes de que se abra el socket, se reproducen antes que las nuevas actividades.

- Cuando un cliente emite una solicitud Obtener actividades de conversación (para actualizar el historial) mientras se conecta a la secuencia de WebSocket, las actividades pueden estar duplicadas en ambos canales. Los clientes deben mantener el seguimiento de todos los identificadores de actividad conocidos para que puedan rechazar las actividades duplicadas, en caso de producirse.

Los clientes que sondean con `HTTP GET` deben elegir un intervalo de sondeo que coincida con su uso previsto.

- Las aplicaciones de servicio a servicio suelen usar un intervalo de sondeo de 5 o 10 s.

- Las aplicaciones orientadas al cliente a menudo usan un intervalo de sondeo de 1 segundo y emiten una única solicitud adicional poco después de cada mensaje que envía el cliente (para recuperar rápidamente la respuesta de un bot). La duración mínima de este retraso puede ser de 300 ms, pero se debe ajustar según el tiempo de tránsito y la velocidad del bot. El sondeo no debe tener una frecuencia superior a una vez por segundo durante cualquier período de tiempo prolongado.

## <a name="additional-resources"></a>Recursos adicionales

- [Conceptos clave](bot-framework-rest-direct-line-3-0-concepts.md)
- [Autenticación](bot-framework-rest-direct-line-3-0-authentication.md)
- [Inicio de una conversación](bot-framework-rest-direct-line-3-0-start-conversation.md)
- [Volver a conectarse a una conversación](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)
- [Envío de una actividad al bot](bot-framework-rest-direct-line-3-0-send-activity.md)
- [Finalización de una conversación](bot-framework-rest-direct-line-3-0-end-conversation.md)

---
title: Volver a conectarse a una conversación | Microsoft Docs
description: Obtenga información sobre cómo volver a conectarse a una conversación mediante Direct Line API v3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/09/2019
ms.openlocfilehash: 3197b4f0e3d8d2cc07cea967f4ddf0738e3e27e9
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299550"
---
# <a name="reconnect-to-a-conversation"></a>Volver a conectarse a una conversación

Si un cliente usa la [interfaz WebSocket](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket) para recibir mensajes pero pierde su conexión, es posible que tenga que volver a conectarse. En este caso, el cliente debe generar una nueva URL de secuencia de WebSocket que pueda usar para volver a conectarse a la conversación.

## <a name="generate-a-new-websocket-stream-url"></a>Generar una nueva dirección URL de secuencia de WebSocket

Para generar una nueva URL de secuencia de WebSocket que se pueda usar para volver a conectarse a una conversación existente, emita esta solicitud: 

```http
GET https://directline.botframework.com/v3/directline/conversations/{conversationId}?watermark={watermark_value}
Authorization: Bearer SECRET_OR_TOKEN
```

En este URI de solicitud, reemplace **{conversationId}** con el id. de conversación y reemplace **{watermark_value}** con el valor de la marca de agua (si el parámetro `watermark` se ha proporcionado). El `watermark` es opcional. Si el parámetro `watermark` se especifica en el URI de solicitud, la conversación se reproduce desde la marca de agua, lo que garantiza que ningún mensaje se pierda. Si se omite el parámetro `watermark` en el URI de la solicitud, se reproducen únicamente los mensajes recibidos tras la solicitud de reconexión.

Los fragmentos de código siguientes proporcionan un ejemplo de la solicitud de reconexión y la respuesta.

### <a name="request"></a>Solicitud

```http
GET https://directline.botframework.com/v3/directline/conversations/abc123?watermark=0000a-42
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Response

Si la solicitud es correcta, la respuesta contendrá un identificador para la conversación, un token y una nueva dirección URL de secuencia de WebSocket.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "streamUrl": "https://directline.botframework.com/v3/directline/conversations/abc123/stream?watermark=000a-4&amp;t=RCurR_XV9ZA.cwA..."
}
```

## <a name="reconnect-to-the-conversation"></a>Volver a conectarse a una conversación

El cliente debe usar la nueva dirección URL de secuencia de WebSocket para [volver a conectarse a la conversación](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket) en 60 segundos. Si no se puede establecer la conexión durante este tiempo, el cliente debe emitir otra solicitud de reconexión para generar una nueva dirección URL de secuencia.

Si tiene "Opción de autenticación mejorada" habilitada en la configuración de Direct Line, podría recibir un error 400 "MissingProperty" que indica que no se especificó ningún identificador de usuario.

## <a name="additional-resources"></a>Recursos adicionales

- [Conceptos clave](bot-framework-rest-direct-line-3-0-concepts.md)
- [Autenticación](bot-framework-rest-direct-line-3-0-authentication.md)
- [Receive activities via WebSocket stream](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket) (Recepción de actividades mediante una secuencia de WebSocket)

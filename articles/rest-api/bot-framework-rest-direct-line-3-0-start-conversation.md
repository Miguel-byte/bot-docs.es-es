---
title: Inicio de una conversación | Microsoft Docs
description: Aprenda a iniciar una conversación mediante Direct Line API v3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 7cc77133b6b2a212b7fde2e0b9d8c854e2bbab9c
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299543"
---
# <a name="start-a-conversation"></a>Inicio de una conversación

Los clientes pueden abrir las conversaciones de Direct Line explícitamente y estas se pueden ejecutar siempre y cuando el cliente y el bot participen y tengan credenciales válidas. Mientras la conversación está abierta, el bot y el cliente pueden enviar mensajes. Más de un cliente puede conectarse a una conversación determinada y cada cliente puede participar en nombre de varios usuarios.

## <a name="open-a-new-conversation"></a>Apertura de una nueva conversación

Para abrir una nueva conversación con un bot, emita esta solicitud:

```http
POST https://directline.botframework.com/v3/directline/conversations
Authorization: Bearer SECRET_OR_TOKEN
```

Los fragmentos de código siguientes proporcionan un ejemplo de la solicitud y respuesta de Iniciar conversación.

### <a name="request"></a>Solicitud

```http
POST https://directline.botframework.com/v3/directline/conversations
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Response

Si la solicitud es correcta, la respuesta contendrá un identificador para la conversación, un token, un valor que indica el número de segundos hasta que expire el token y una dirección URL de secuencia que el cliente puede usar para [recibir actividades mediante la secuencia de WebSocket](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket).

```http
HTTP/1.1 201 Created
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "expires_in": 1800,
  "streamUrl": "https://directline.botframework.com/v3/directline/conversations/abc123/stream?t=RCurR_XV9ZA.cwA..."
}
```

Normalmente, una solicitud Iniciar conversación se usa para abrir una nueva conversación y, si la nueva conversación se inicia correctamente, se devuelve un código de estado **HTTP 201**. Sin embargo, si un cliente envía una solicitud Iniciar conversación con un token de Direct Line en el encabezado `Authorization` que se ha usado anteriormente para iniciar una conversación mediante la operación Iniciar conversación, se devuelve un código de estado **HTTP 200** para indicar que la solicitud era aceptable, pero no se ha creado ninguna conversación (porque ya existía).

> [!TIP]
> Tendrá 60 segundos para conectarse a la dirección URL de secuencia de WebSocket. Si no se puede establecer la conexión durante este tiempo, puede [volver a conectarse a la conversación](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md) para generar una nueva dirección URL de secuencia.

## <a name="start-conversation-versus-generate-token"></a>Iniciar conversación frente a Generar token

La operación Iniciar conversación (`POST /v3/directline/conversations`) se parece a la operación [Generar token](bot-framework-rest-direct-line-3-0-authentication.md#generate-token) (`POST /v3/directline/tokens/generate`) en que ambas operaciones devuelven un `token` que puede usarse para acceder a una única conversación. Sin embargo, la operación Iniciar conversación también inicia la conversación, se pone en contacto con el bot y crea una dirección URL de secuencia de WebSocket, mientras que la operación Generar token no hace ninguna de estas cosas. 

Si quiere iniciar la conversación inmediatamente, use la operación Iniciar conversación. Si va a distribuir el token a los clientes y quiere que inicien la conversación, use en su lugar la operación [Generar token](bot-framework-rest-direct-line-3-0-authentication.md#generate-token). 

## <a name="additional-resources"></a>Recursos adicionales

- [Conceptos clave](bot-framework-rest-direct-line-3-0-concepts.md)
- [Autenticación](bot-framework-rest-direct-line-3-0-authentication.md)
- [Receive activities via WebSocket stream](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket) (Recepción de actividades mediante una secuencia de WebSocket)
- [Volver a conectarse a una conversación](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)
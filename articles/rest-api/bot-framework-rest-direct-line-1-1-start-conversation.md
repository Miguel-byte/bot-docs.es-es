---
title: Inicio de una conversación | Microsoft Docs
description: Obtenga información sobre cómo iniciar una conversación mediante Direct Line API v1.1.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 16ad7b817ccde2139b703e858e57bee2abf36fb8
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299602"
---
# <a name="start-a-conversation"></a>Inicio de una conversación

> [!IMPORTANT]
> En este artículo se describe cómo iniciar una conversación mediante Direct Line API 1.1. Si va a crear una nueva conexión entre la aplicación cliente y el bot, use [Direct Line API 3.0](bot-framework-rest-direct-line-3-0-start-conversation.md) en su lugar.

Los clientes pueden abrir las conversaciones de Direct Line explícitamente y estas se pueden ejecutar siempre y cuando el cliente y el bot participen y tengan credenciales válidas. Mientras la conversación está abierta, el bot y el cliente pueden enviar mensajes. Más de un cliente puede conectarse a una conversación determinada y cada cliente puede participar en nombre de varios usuarios.

## <a name="open-a-new-conversation"></a>Apertura de una nueva conversación

Para abrir una nueva conversación con un bot, emita esta solicitud:

```http
POST https://directline.botframework.com/api/conversations
Authorization: Bearer SECRET_OR_TOKEN
```

Los fragmentos de código siguientes proporcionan un ejemplo de la solicitud y respuesta de Iniciar conversación.

### <a name="request"></a>Solicitud

```http
POST https://directline.botframework.com/api/conversations
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Response

Si la solicitud es correcta, la respuesta contendrá un identificadorpara la conversación, un token y un valor que indica el número de segundos hasta que el token expira.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "expires_in": 1800
}
```

## <a name="start-conversation-versus-generate-token"></a>Iniciar conversación frente a Generar token

La operación Iniciar conversación (`POST /api/conversations`) se parece a la operación [Generar token](bot-framework-rest-direct-line-1-1-authentication.md#generate-token) (`POST /api/tokens/conversation`) en que ambas operaciones devuelven un `token` que puede usarse para acceder a una única conversación. Sin embargo, la operación Iniciar conversación también inicia la conversación y se pone en contacto con el bot, mientras que la operación Generar token no hace ninguna de estas cosas. 

Si quiere iniciar la conversación inmediatamente, use la operación Iniciar conversación. Si va a distribuir el token a los clientes y quiere que inicien la conversación, use en su lugar la operación [Generar token](bot-framework-rest-direct-line-1-1-authentication.md#generate-token). 

## <a name="additional-resources"></a>Recursos adicionales

- [Conceptos clave](bot-framework-rest-direct-line-1-1-concepts.md)
- [Autenticación](bot-framework-rest-direct-line-1-1-authentication.md)
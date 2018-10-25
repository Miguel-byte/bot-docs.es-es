---
title: Autenticación | Microsoft Docs
description: Aprenda a autenticar solicitudes de API en Direct Line API v1.1.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 4f607050fd891eefe2129973a46d830aa0bab6c7
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997622"
---
# <a name="authentication"></a>Autenticación

> [!IMPORTANT]
> En este artículo se describe la autenticación en Direct Line API 1.1. Si va a crear una nueva conexión entre la aplicación cliente y el bot, use [Direct Line API 3.0](bot-framework-rest-direct-line-3-0-authentication.md) en su lugar.

Un cliente puede autenticar solicitudes a Direct Line API 1.1 mediante un **secreto** que se [obtiene de la página de configuración del canal de Direct Line](../bot-service-channel-connect-directline.md) en el portal de Bot Framework o mediante un **token** que se obtiene en tiempo de ejecución.

El secreto o token debe especificarse en el encabezado `Authorization` de cada solicitud, mediante los esquemas "Bearer" o "BotConnector". 

**Esquema Bearer**:
```http
Authorization: Bearer SECRET_OR_TOKEN
```

**Esquema BotConnector**:
```http
Authorization: BotConnector SECRET_OR_TOKEN
```

## <a name="secrets-and-tokens"></a>Secretos y tokens

Un **secreto** de Direct Line es una clave maestra que se puede usar para acceder a cualquier conversación que pertenezca al bot asociado. Un **secreto** se puede usar también para obtener un **token**. Los secretos no expiran. 

Un **token** de Direct Line es una clave que se puede usar para acceder a una única conversación. Un token expira, pero se puede actualizar. 

Si va a crear una aplicación de servicio a servicio, puede que el enfoque más sencillo sea especificar el **secreto** en el encabezado `Authorization` de las solicitudes de Direct Line API. Si va a escribir una aplicación en la que el cliente se ejecuta en un explorador web o una aplicación móvil, puede que quiera intercambiar el secreto por un token (que solo sirve para una única conversación y expira a menos que se actualice) y especificar el **token** en el encabezado `Authorization` de las solicitudes de Direct Line API. Elija el modelo de seguridad que mejor funcione en su caso.

> [!NOTE]
> Las credenciales de cliente de Direct Line son diferentes de las credenciales del bot. Esto le permite revisar las claves de forma independiente y compartir los tokens de cliente sin revelar la contraseña del bot. 

## <a name="get-a-direct-line-secret"></a>Obtención de un secreto de Direct Line

También puede [obtener un secreto de Direct Line](../bot-service-channel-connect-directline.md) a través de la página de configuración del canal Direct Line de su bot en el <a href="https://dev.botframework.com/" target="_blank">portal de Bot Framework</a>:

![Configuración de Direct Line](../media/direct-line-configure.png)

## <a id="generate-token"></a> Generación de un token de Direct Line

Para generar un token de Direct Line que pueda usarse para acceder a una única conversación, primero obtenga el secreto de Direct Line de la página de configuración del canal Direct Line en el <a href="https://dev.botframework.com/" target="_blank">portal de Bot Framework</a>. A continuación, emita esta solicitud para intercambiar el secreto de Direct Line por un token de Direct Line:

```http
POST https://directline.botframework.com/api/tokens/conversation
Authorization: Bearer SECRET
```

En el encabezado `Authorization` de esta solicitud, reemplace **SECRET** por el valor del secreto de Direct Line.

Los fragmentos de código siguientes proporcionan un ejemplo de la solicitud y respuesta de Generar un token.

### <a name="request"></a>Solicitud

```http
POST https://directline.botframework.com/api/tokens/conversation
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>Response

Si la solicitud es correcta, la respuesta contiene un token válido para una conversación. El token expira en 30 minutos. Para que el token siga siendo útil, debe [actualizarlo](#refresh-token) antes de que expire.

```http
HTTP/1.1 200 OK
[other headers]

"RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn"
```

### <a name="generate-token-versus-start-conversation"></a>Generar token frente a Iniciar conversación

La operación Generar token (`POST /api/tokens/conversation`) es parecida a la operación [Iniciar conversación](bot-framework-rest-direct-line-1-1-start-conversation.md) (`POST /api/conversations`) en el sentido de que ambas operaciones devuelven un `token` que se puede usar para acceder a una única conversación. Sin embargo, a diferencia de la operación Iniciar conversación, la operación Generar token no inicia la conversación o el contacto con el bot. 

Si va a distribuir el token a los clientes y quiere que inicien la conversación, use la operación Generar token. Si tiene previsto iniciar inmediatamente la conversación, use en su lugar la operación [Iniciar conversación](bot-framework-rest-direct-line-1-1-start-conversation.md).

## <a id="refresh-token"></a> Actualización de un token de Direct Line

Un token de Direct Line es válido durante 30 minutos desde el momento en que se genera y se puede actualizar una cantidad ilimitada de veces, siempre y cuando no haya expirado. No se puede actualizar un token expirado. Para actualizar un token de Direct Line, emita esta solicitud:

```http
POST https://directline.botframework.com/api/tokens/{conversationId}/renew
Authorization: Bearer TOKEN_TO_BE_REFRESHED
```

En el URI de esta solicitud, reemplace **{conversationId}** por el identificador de la conversación para la que el token es válido, y en el encabezado `Authorization` de esta solicitud, reemplace **TOKEN_TO_BE_REFRESHED** por el token de Direct Line que quiere actualizar.

Los fragmentos de código siguientes proporcionan un ejemplo de la solicitud y respuesta de Actualizar token.

### <a name="request"></a>Solicitud

```http
POST https://directline.botframework.com/api/tokens/abc123/renew
Authorization: Bearer CurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Response

Si la solicitud es correcta, la respuesta contiene un nuevo token que es válido para la misma conversación que el token anterior. El token nuevo expirará en 30 minutos. Para que el nuevo token siga siendo útil, debe [actualizarlo](#refresh-token) antes de que expire.

```http
HTTP/1.1 200 OK
[other headers]

"RCurR_XV9ZA.cwA.BKA.y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xniaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0"
```

## <a name="additional-resources"></a>Recursos adicionales

- [Conceptos clave](bot-framework-rest-direct-line-1-1-concepts.md)
- [Conectar un bot con Direct Line](../bot-service-channel-connect-directline.md)
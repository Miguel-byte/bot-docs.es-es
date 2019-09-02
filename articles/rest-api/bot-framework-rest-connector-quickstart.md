---
title: Crear un bot con el servicio Bot Connector | Microsoft Docs
description: Cree un bot con el servicio Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 751a5d8430bb675e8ad5e10d02f94ee5642672cb
ms.sourcegitcommit: c200cc2db62dbb46c2a089fb76017cc55bdf26b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2019
ms.locfileid: "70037477"
---
# <a name="create-a-bot-with-the-bot-connector-service"></a>Crear un bot con el servicio Bot Connector
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.js](../nodejs/bot-builder-nodejs-quickstart.md)
> - [Bot Service](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

El servicio Bot Connector permite que su bot intercambie mensajes con los canales configurados en <a href="https://dev.botframework.com/" target="_blank">Bot Framework Portal</a>, mediante los estándares del sector JSON y REST a través de HTTPS. Este tutorial le guiará a través del proceso para obtener un token de acceso de Bot Framework y utilizar el servicio Bot Connector para intercambiar mensajes con el usuario.

## <a id="get-token"></a> Obtener un token de acceso

> [!IMPORTANT]
> Si aún no lo ha hecho, deberá [registrar el bot](../bot-service-quickstart-registration.md) con Bot Framework para obtener su identificador de aplicación y la contraseña. Necesitará el identificador de la aplicación del bot y la contraseña para obtener un token de acceso.

Para comunicarse con el servicio Bot Connector, debe especificar un token de acceso en el encabezado `Authorization` de cada solicitud de API, con este formato: 

```http
Authorization: Bearer ACCESS_TOKEN
```

Puede obtener el token de acceso para el bot si emite una solicitud de API.

### <a name="request"></a>Solicitud

Para solicitar un token de acceso que se pueda usar para autenticar las solicitudes al servicio Bot Connector, emita la siguiente solicitud, reemplace **MICROSOFT-APP-ID** y **MICROSOFT-APP-PASSWORD** con el identificador de aplicación y la contraseña que obtuvo cuando [registró](../bot-service-quickstart-registration.md) su bot con Bot Framework.

```http
POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=MICROSOFT-APP-ID&client_secret=MICROSOFT-APP-PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default
```

### <a name="response"></a>Response

Si la solicitud se realiza correctamente, recibirá una respuesta HTTP 200 que especifica el token de acceso e información sobre la fecha de expiración. 

```json
{
    "token_type":"Bearer",
    "expires_in":3600,
    "ext_expires_in":3600,
    "access_token":"eyJhbGciOiJIUzI1Ni..."
}
```

> [!TIP]
> Para obtener más información sobre la autenticación en el servicio Bot Connector, consulte [Autenticación](bot-framework-rest-connector-authentication.md).

## <a name="exchange-messages-with-the-user"></a>Intercambiar mensajes con el usuario

Una conversación es una serie de mensajes que se intercambian un usuario y su bot. 

### <a name="receive-a-message-from-the-user"></a>Recibir un mensaje del usuario

Cuando el usuario envía un mensaje, Bot Framework Connector envía una solicitud al punto de conexión que se especificó cuando se [registró](../bot-service-quickstart-registration.md) el bot. El cuerpo de la solicitud es un objeto [Actividad][]. En el siguiente ejemplo se muestra el cuerpo de la solicitud que recibe un bot cuando el usuario le envía un mensaje simple. 

```json
{
    "type": "message",
    "id": "bf3cc9a2f5de...",
    "timestamp": "2016-10-19T20:17:52.2891902Z",
    "serviceUrl": "https://smba.trafficmanager.net/apis",
    "channelId": "channel's name/id",
    "from": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
    "recipient": {
        "id": "12345678",
        "name": "bot's name"
    },
    "text": "Haircut on Saturday"
}
```

### <a name="reply-to-the-users-message"></a>Responder al mensaje del usuario

Cuando el punto de conexión del bot recibe una solicitud `POST` que representa un mensaje del usuario (es decir, `type` = **mensaje**), use la información de esa solicitud para crear el objeto [Actividad][] de la respuesta.

1. Establezca la propiedad **conversation** en el contenido de la propiedad **conversation** del mensaje del usuario.
2. Establezca la propiedad **from** en el contenido de la propiedad **recipient** del mensaje del usuario.
3. Establezca la propiedad **recipient** en el contenido de la propiedad **from** del mensaje del usuario.
4. Establezca las propiedades **text** y **attachments** según corresponda.

Use la propiedad `serviceUrl` de la solicitud entrante para [identificar el URI base](bot-framework-rest-connector-api-reference.md#base-uri) que el bot debería usar para emitir su respuesta. 

Para enviar la respuesta, envíe mediante `POST` el objeto `Activity` a `/v3/conversations/{conversationId}/activities/{activityId}`, tal como se muestra en el siguiente ejemplo. El cuerpo de esta solicitud es un objeto `Activity` que solicita al usuario que seleccione una hora de cita disponible.

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "text": "I have these times available:",
    "replyToId": "bf3cc9a2f5de..."
}
```

En la solicitud del ejemplo, `https://smba.trafficmanager.net/apis` representa el URI base; el URI base para solicitudes que su bot emita puede ser distinto. Para obtener más información sobre cómo establecer el URI base, consulte [Referencia de la API](bot-framework-rest-connector-api-reference.md#base-uri). 

> [!IMPORTANT]
> Tal como se muestra en este ejemplo, el encabezado `Authorization` de cada solicitud de API que envíe debe contener la palabra **Bearer** (portador) seguida del token de acceso que [obtuvo en Bot Framework](#get-token).

Para enviar otro mensaje que permita al usuario seleccionar una hora de cita disponible al hacer clic en un botón, `POST` otra solicitud para el mismo punto de conexión:

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "attachmentLayout": "list",
    "attachments": [
      {
        "contentType": "application/vnd.microsoft.card.thumbnail",
        "content": {
          "buttons": [
            {
              "type": "imBack",
              "title": "10:30",
              "value": "10:30"
            },
            {
              "type": "imBack",
              "title": "11:30",
              "value": "11:30"
            },
            {
              "type": "openUrl",
              "title": "See more",
              "value": "http://www.contososalon.com/scheduling"
            }
          ]
        }
      }
    ],
    "replyToId": "bf3cc9a2f5de..."
}
```   

## <a name="next-steps"></a>Pasos siguientes

En este tutorial obtuvo un token de acceso de Bot Framework y usó el servicio Bot Connector para intercambiar mensajes con el usuario. Puede usar [Bot Framework Emulator](../bot-service-debug-emulator.md) para probar y depurar el bot. Si desea compartir su bot con otros usuarios, deberá [configurarlo](../bot-service-manage-channels.md) para que se ejecute en uno o más canales.

[Actividad]: bot-framework-rest-connector-api-reference.md#activity-object

---
title: Referencia de API | Microsoft Docs
description: Obtenga información sobre los encabezados, operaciones, objetos y errores de los servicios Bot Connector y Bot State.
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 10/25/2018
ms.openlocfilehash: f8f04c8b0cbd2b43f29676f0315739f4cc7716b3
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2019
ms.locfileid: "68757169"
---
# <a name="api-reference"></a>Referencia de API

> [!NOTE]
> La API de REST no es equivalente al SDK. La API de REST se proporciona para permitir la comunicación de REST estándar; no obstante, el método preferido para interactuar con Bot Framework es el SDK. 

En Bot Framework, el servicio Bot Connector permite que el bot intercambie mensajes con los usuarios en los canales configurados en el portal de Bot Framework. El servicio usa los estándares del sector REST y JSON sobre HTTPS.

## <a name="base-uri"></a>URI base

Cuando un usuario envía un mensaje al bot, la solicitud entrante contiene un objeto [Actividad](#bot-framework-activity-schema) con una propiedad `serviceUrl` que especifica el punto de conexión al que el bot debe enviar su respuesta. Para acceder al servicio Bot Connector, use el valor `serviceUrl` como el identificador URI de base para las solicitudes de API. 

Por ejemplo, suponga que su bot recibe la siguiente actividad cuando el usuario envía un mensaje al bot.

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

La propiedad `serviceUrl` del mensaje del usuario indica que el bot debe enviar su respuesta al punto de conexión `https://smba.trafficmanager.net/apis`; será el URI base para todas las solicitudes posteriores que el bot envíe en el contexto de esta conversación. Si el bot necesita enviar un mensaje automático al usuario, asegúrese de guardar el valor de `serviceUrl`.

En el ejemplo siguiente se muestra la solicitud que el bot envía para responder al mensaje del usuario. 

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
    "text": "I have several times available on Saturday!",
    "replyToId": "bf3cc9a2f5de..."
}
```

## <a name="headers"></a>encabezados

### <a name="request-headers"></a>Encabezados de solicitud

Además de los encabezados de solicitud HTTP estándares, todas las solicitudes de API que envíe deben incluir un encabezado `Authorization` que especifique un token de acceso para autenticar el bot. Especifique el encabezado `Authorization` con este formato:

```http
Authorization: Bearer ACCESS_TOKEN
```

Para consultar información sobre cómo obtener un token de acceso para el bot, vea [Authenticate requests from your bot to the Bot Connector service](bot-framework-rest-connector-authentication.md#bot-to-connector) (Autenticación de solicitudes del bot en el servicio Bot Connector).

### <a name="response-headers"></a>Encabezados de respuesta

Además de los encabezados de respuesta HTTP estándares, cada respuesta contendrá un encabezado `X-Correlating-OperationId`. El valor de este encabezado es un identificador que corresponde a la entrada de registro de Bot Framework que contiene detalles acerca de la solicitud. Siempre que reciba una respuesta de error, debe capturar el valor de este encabezado. Si no puede resolver el problema de forma independiente, incluya este valor en la información que proporcione al equipo de soporte técnico al informar del problema.

## <a name="http-status-codes"></a>Códigos de estado HTTP

El <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank">código de estado HTTP</a> que se devuelve con cada respuesta indica el resultado de la solicitud correspondiente. 

| Código de estado HTTP | Significado |
|----|----|
| 200 | La solicitud finalizó correctamente. |
| 201 | La solicitud finalizó correctamente. |
| 202 | La solicitud se ha aceptado para procesamiento. |
| 204 | La solicitud se realizó correctamente, pero no se devolvió ningún contenido. |
| 400 | La solicitud tenía formato incorrecto o era incorrecta por otro motivo. |
| 401 | El bot no está autorizado para realizar solicitudes. |
| 403 | No se permite el bot para llevar a cabo la operación solicitada. |
| 404 | No se encontró el recurso solicitado. |
| 500 | Se ha producido un error del servidor interno. |
| 503 | El servicio no está disponible. |

### <a name="errors"></a>Errors

Cualquier respuesta que especifique un código de estado HTTP en el rango de 4xx o 5xx incluirá un objeto [ErrorResponse](#bot-framework-activity-schema) en el cuerpo de la respuesta que proporciona información sobre el error. Si recibe una respuesta de error en el rango de 4xx, inspeccione el objeto **ErrorResponse** para identificar la causa del error y resolver el problema antes de volver a enviar la solicitud.

## <a name="conversation-operations"></a>Operaciones con conversaciones 
Use estas operaciones para crear conversaciones, enviar mensajes (actividades) y administrar el contenido de las conversaciones.

| Operación | DESCRIPCIÓN |
|----|----|
| [Crear conversación](#create-conversation) | Crea una conversación. |
| [Enviar a conversación](#send-to-conversation) | Envía una actividad (mensaje) al final de la conversación especificada. |
| [Responder a actividad](#reply-to-activity) | Envía una actividad (mensaje) a la conversación especificada, como una respuesta a la actividad especificada. |
| [Obtener conversaciones](#get-conversations) | Obtiene una lista de las conversaciones en las que ha participado el bot. |
| [Obtener miembros de la conversación](#get-conversation-members) | Obtiene los miembros de la conversación especificada. |
| [Obtener los miembros de la conversación paginados](#get-conversation-paged-members) | Obtiene los miembros de la conversación especificada una página a la vez. |
| [Obtener miembros de la actividad](#get-activity-members) | Obtiene los miembros de la actividad especificada dentro de la conversación especificada. |
| [Actualizar actividad](#update-activity) | Actualiza una actividad existente. |
| [Eliminar actividad](#delete-activity) | Elimina una actividad existente. |
| [Eliminar miembro de la conversación](#delete-conversation-member) | Quita un miembro de una conversación. |
| [Enviar historial de la conversación](#send-conversation-history) | Carga una transcripción de actividades pasadas a la conversación. |
| [Cargar datos adjuntos al canal](#upload-attachment-to-channel) | Carga un archivo adjunto directamente al almacenamiento de blobs de un canal. |

### <a name="create-conversation"></a>Crear conversación
Crea una conversación.
```http 
POST /v3/conversations
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Un objeto `ConversationParameters` |
| **Devuelve** | Un objeto `ConversationResourceResponse` |

### <a name="send-to-conversation"></a>Enviar a conversación
Envía una actividad (mensaje) a la conversación especificada. La actividad se anexará al final de la conversación según la marca de tiempo o la semántica del canal. Para responder a un mensaje específico dentro de la conversación, en su lugar, use [Responder a actividad](#reply-to-activity).
```http
POST /v3/conversations/{conversationId}/activities
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Un objeto [Activity](#bot-framework-activity-schema) |
| **Devuelve** | Un objeto [Identification](#bot-framework-activity-schema) | 

### <a name="reply-to-activity"></a>Responder a actividad
Envía una actividad (mensaje) a la conversación especificada, como una respuesta a la actividad especificada. La actividad se agregará como respuesta a otra actividad, si lo admite el canal. Si el canal no admite respuestas anidadas, esta operación se comporta como [Enviar a conversación](#send-to-conversation).
```http
POST /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Un objeto [Activity](#bot-framework-activity-schema) |
| **Devuelve** | Un objeto [Identification](#bot-framework-activity-schema) | 

### <a name="get-conversations"></a>Obtener conversaciones
Obtiene una lista de las conversaciones en las que ha participado el bot.
```http
GET /v3/conversations?continuationToken={continuationToken}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Un objeto [ConversationsResult](#bot-framework-activity-schema) | 

### <a name="get-conversation-members"></a>Obtener miembros de la conversación
Obtiene los miembros de la conversación especificada.
```http
GET /v3/conversations/{conversationId}/members
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Una matriz de objetos [ChannelAccount](#bot-framework-activity-schema) | 

### <a name="get-conversation-paged-members"></a>Obtener los miembros de la conversación paginados
Obtiene los miembros de la conversación especificada una página a la vez.
```http
GET /v3/conversations/{conversationId}/pagedmembers?pageSize={pageSize}&continuationToken={continuationToken}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Una matriz de objetos [ChannelAccount](#bot-framework-activity-schema) y un token de continuación que se puede utilizar para obtener más valores. |

### <a name="get-activity-members"></a>Obtener miembros de la actividad
Obtiene los miembros de la actividad especificada dentro de la conversación especificada.
```http
GET /v3/conversations/{conversationId}/activities/{activityId}/members
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Una matriz de objetos [ChannelAccount](#bot-framework-activity-schema) | 

### <a name="update-activity"></a>Actualizar actividad
Algunos canales le permiten modificar una actividad existente para reflejar el nuevo estado de una conversación de bot. Por ejemplo, puede quitar los botones de un mensaje de la conversación después de que el usuario haya hecho clic en uno de los botones. Si se realiza correctamente, esta operación actualiza la actividad especificada dentro de la conversación establecida. 
```http
PUT /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Un objeto [Activity](#bot-framework-activity-schema) |
| **Devuelve** | Un objeto [Identification](#bot-framework-activity-schema) | 

### <a name="delete-activity"></a>Eliminar actividad
Algunos canales le permiten eliminar una actividad existente. Si se realiza correctamente, esta operación elimina la actividad especificada de la conversación establecida.
```http
DELETE /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Un código de estado HTTP que indica el resultado de la operación. No se especifica nada en el cuerpo de la respuesta. | 

### <a name="delete-conversation-member"></a>Eliminar miembro de la conversación
Quita un miembro de una conversación. Si ese miembro era el último miembro de la conversación, también se eliminará la conversación.
```http
DELETE /v3/conversations/{conversationId}/members/{memberId}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Un código de estado HTTP que indica el resultado de la operación. No se especifica nada en el cuerpo de la respuesta. | 

### <a name="send-conversation-history"></a>Enviar historial de la conversación
Carga una transcripción de actividades pasadas a la conversación para que el cliente pueda representarlas.
```http
POST /v3/conversations/{conversationId}/activities/history
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Un objeto [Transcript](#bot-framework-activity-schema). |
| **Devuelve** | Un objeto [ResourceResponse](#bot-framework-activity-schema). | 

### <a name="upload-attachment-to-channel"></a>Cargar datos adjuntos al canal
Carga un archivo adjunto para la conversación especificada directamente en el almacenamiento de blobs de un canal. Esto le permite almacenar datos en un almacén compatible.
```http 
POST /v3/conversations/{conversationId}/attachments
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Un objeto [AttachmentUpload](#bot-framework-activity-schema). |
| **Devuelve** | Un objeto [ResourceResponse](#bot-framework-activity-schema). La propiedad **Id.** especifica el identificador de datos adjuntos que se puede usar con las operaciones [Obtener información de datos adjuntos](#get-attachment-info) y [Obtener datos adjuntos](#get-attachment). | 

## <a name="attachment-operations"></a>Operaciones con datos adjuntos 
Utilice estas operaciones para recuperar información sobre los datos adjuntos y los datos binarios para el propio archivo.

| Operación | DESCRIPCIÓN |
|----|----|
| [Obtener información de datos adjuntos](#get-attachment-info) | Obtiene información sobre los datos adjuntos especificados, incluido el nombre de archivo, el tipo de archivo y las vistas disponibles (p. ej., original o miniatura). |
| [Obtener datos adjuntos](#get-attachment) | Obtiene la vista especificada de los datos adjuntos determinados como contenido binario. | 

### <a name="get-attachment-info"></a>Obtener información de datos adjuntos 
Obtiene información sobre los datos adjuntos especificados, incluido el nombre de archivo, el tipo y las vistas disponibles (p. ej., original o miniatura).
```http
GET /v3/attachments/{attachmentId}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Un objeto [AttachmentInfo](#bot-framework-activity-schema) | 

### <a name="get-attachment"></a>Obtener datos adjuntos
Obtiene la vista especificada de los datos adjuntos determinados como contenido binario.
```http
GET /v3/attachments/{attachmentId}/views/{viewId}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Contenido binario que representa la vista especificada de los datos adjuntos establecidos | 

## <a name="state-operations"></a>Operaciones de estado
El servicio de estado de Microsoft bot Framework se retirará a partir del 30 de marzo de 2018. Anteriormente, los bots creados en Azure Bot Service o en el SDK Bot Builder tenían una conexión predeterminada con este servicio hospedado por Microsoft para almacenar los datos de estado del bot. Los bots se deberán actualizar para usar su propio almacenamiento del estado.

| Operación | DESCRIPCIÓN |
|----|----|
| `Set User Data` | Almacena datos de estado de un usuario específico en un canal. |
| `Set Conversation Data` | Almacena datos de estado de una conversación específica en un canal. |
| `Set Private Conversation Data` | Almacena datos de estado de un usuario específico en el contexto de una conversación determinada en un canal. |
| `Get User Data` | Recupera datos de estado que previamente se almacenaron para un usuario específico de todas las conversaciones en un canal. |
| `Get Conversation Data` | Recupera datos de estado que previamente se almacenaron para una conversación específica en un canal. |
| `Get Private Conversation Data` | Recupera datos de estado que previamente se almacenaron para un usuario específico en el contexto de una conversación determinada en un canal. |
| `Delete State For User` | Elimina los datos de estado que se han almacenado previamente para un usuario. |

### <a name="set-user-data"></a>Establecer datos de usuario
Almacena datos de estado del usuario especificado en el canal establecido.
```http
POST /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Un objeto `BotData` |
| **Devuelve** | Un objeto `BotData` | 

### <a name="set-conversation-data"></a>Establecer datos de conversación
Almacena datos de estado de la conversación especificada en el canal establecido.
```http
POST /v3/botstate/{channelId}/conversations/{conversationId}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Un objeto `BotData` |
| **Devuelve** | Un objeto `BotData` | 

### <a name="set-private-conversation-data"></a>Establecer datos de conversación privados
Almacena datos de estado del usuario especificado en el contexto de la conversación definida en el canal establecido.
```http
POST /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId} 
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Un objeto `BotData` |
| **Devuelve** | Un objeto `BotData` | 

### <a name="get-user-data"></a>Obtener datos de usuario
Recupera datos de estado que previamente se almacenaron para un usuario específico de todas las conversaciones en un canal determinado.
```http
GET /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Un objeto `BotData` | 

### <a name="get-conversation-data"></a>Obtener datos de conversación
Recupera datos de estado que previamente se almacenaron para una conversación específica en un canal determinado.
```http
GET /v3/botstate/{channelId}/conversations/{conversationId} 
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Un objeto `BotData` | 

### <a name="get-private-conversation-data"></a>Obtener datos de conversación privados
Recupera datos de estado que previamente se almacenaron para el usuario especificado en el contexto de la conversación establecida en un canal determinado.
```http
GET /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId} 
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Un objeto [`BotData` | 

### <a name="delete-state-for-user"></a>Eliminar estado de usuario
Elimina los datos de estado almacenados anteriormente de un usuario específico en el canal establecido con el uso de las operaciones [Establecer datos de usuario](#set-user-data) o [Establecer datos de conversación privados](#set-private-conversation-data).
```http
DELETE /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Una matriz de cadenas (ID) | 

## <a name="bot-framework-activity-schema"></a>Esquema de actividad de Bot Framework

Consulte el [Esquema de actividad de Bot Framework](https://aka.ms/botSpecs-activitySchema) para ver los objetos y las propiedades que puede usar el bot para comunicarse con un usuario.

---
title: Referencia de API | Microsoft Docs
description: Obtenga información sobre los encabezados, operaciones, objetos y errores de los servicios Bot Connector y Bot State.
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/25/2018
ms.openlocfilehash: 2600b69fff24f6d952853c7b1ed764577b4cb270
ms.sourcegitcommit: f3fda6791f48ab178721b72d4f4a77c373573e38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2019
ms.locfileid: "68671492"
---
# <a name="api-reference"></a>Referencia de API

> [!NOTE]
> La API de REST no es equivalente al SDK. La API de REST se proporciona para permitir la comunicación de REST estándar; no obstante, el método preferido para interactuar con Bot Framework es el SDK. 

En Bot Framework, el servicio Bot Connector permite al bot intercambiar mensajes con usuarios en canales configurados en el portal de Bot Framework, y el servicio Bot State permite al bot almacenar y recuperar datos de estado relacionados con las conversaciones que el bot mantiene mediante el servicio Bot Connector. Ambos servicios usan los estándares del sector REST y JSON a través de HTTPS.

> [!IMPORTANT]
> No se recomienda la API del servicio de estado de Bot Framework para entornos de producción y es posible que se encuentre en desuso en una versión futura. Se recomienda actualizar el código del bot para que use el almacenamiento en memoria para realizar pruebas o usar una de las **extensiones de Azure** para bots de producción. Para más información, consulte el tema **Administración de datos de estado** para la implementación de [.NET](~/dotnet/bot-builder-dotnet-state.md) o [Node](~/nodejs/bot-builder-nodejs-state.md).

## <a name="base-uri"></a>URI base

Cuando un usuario envía un mensaje al bot, la solicitud entrante contiene un objeto [Actividad](#activity-object) con una propiedad `serviceUrl` que especifica el punto de conexión al que el bot debe enviar su respuesta. Para acceder al servicio Bot Connector o Bot State, use el valor `serviceUrl` como el URI base para las solicitudes de API. 

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

Cualquier respuesta que especifique un código de estado HTTP en el rango de 4xx o 5xx incluirá un objeto [ErrorResponse](#errorresponse-object) en el cuerpo de la respuesta que proporciona información sobre el error. Si recibe una respuesta de error en el rango de 4xx, inspeccione el objeto **ErrorResponse** para identificar la causa del error y resolver el problema antes de volver a enviar la solicitud.

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
| **Cuerpo de la solicitud** | Un objeto [ConversationParameters](#conversationparameters-object) |
| **Devuelve** | Un objeto [ConversationResourceResponse](#conversationresourceresponse-object) |

### <a name="send-to-conversation"></a>Enviar a conversación
Envía una actividad (mensaje) a la conversación especificada. La actividad se anexará al final de la conversación según la marca de tiempo o la semántica del canal. Para responder a un mensaje específico dentro de la conversación, en su lugar, use [Responder a actividad](#reply-to-activity).
```http
POST /v3/conversations/{conversationId}/activities
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Un objeto [Activity](#activity-object) |
| **Devuelve** | Un objeto [Identification](#identification-object) | 

### <a name="reply-to-activity"></a>Responder a actividad
Envía una actividad (mensaje) a la conversación especificada, como una respuesta a la actividad especificada. La actividad se agregará como respuesta a otra actividad, si lo admite el canal. Si el canal no admite respuestas anidadas, esta operación se comporta como [Enviar a conversación](#send-to-conversation).
```http
POST /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Un objeto [Activity](#activity-object) |
| **Devuelve** | Un objeto [Identification](#identification-object) | 

### <a name="get-conversations"></a>Obtener conversaciones
Obtiene una lista de las conversaciones en las que ha participado el bot.
```http
GET /v3/conversations?continuationToken={continuationToken}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Un objeto [ConversationsResult](#conversationsresult-object) | 

### <a name="get-conversation-members"></a>Obtener miembros de la conversación
Obtiene los miembros de la conversación especificada.
```http
GET /v3/conversations/{conversationId}/members
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Una matriz de objetos [ChannelAccount](#channelaccount-object) | 

### <a name="get-conversation-paged-members"></a>Obtener los miembros de la conversación paginados
Obtiene los miembros de la conversación especificada una página a la vez.
```http
GET /v3/conversations/{conversationId}/pagedmembers?pageSize={pageSize}&continuationToken={continuationToken}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Una matriz de objetos [ChannelAccount](#channelaccount-object) y un token de continuación que se puede utilizar para obtener más valores. |

### <a name="get-activity-members"></a>Obtener miembros de la actividad
Obtiene los miembros de la actividad especificada dentro de la conversación especificada.
```http
GET /v3/conversations/{conversationId}/activities/{activityId}/members
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Una matriz de objetos [ChannelAccount](#channelaccount-object) | 

### <a name="update-activity"></a>Actualizar actividad
Algunos canales le permiten modificar una actividad existente para reflejar el nuevo estado de una conversación de bot. Por ejemplo, puede quitar los botones de un mensaje de la conversación después de que el usuario haya hecho clic en uno de los botones. Si se realiza correctamente, esta operación actualiza la actividad especificada dentro de la conversación establecida. 
```http
PUT /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Un objeto [Activity](#activity-object) |
| **Devuelve** | Un objeto [Identification](#identification-object) | 

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
| **Cuerpo de la solicitud** | Un objeto [Transcript](#transcript-object). |
| **Devuelve** | Un objeto [ResourceResponse](#resourceresponse-object). | 

### <a name="upload-attachment-to-channel"></a>Cargar datos adjuntos al canal
Carga un archivo adjunto para la conversación especificada directamente en el almacenamiento de blobs de un canal. Esto le permite almacenar datos en un almacén compatible.
```http 
POST /v3/conversations/{conversationId}/attachments
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Un objeto [AttachmentUpload](#attachmentupload-object). |
| **Devuelve** | Un objeto [ResourceResponse](#resourceresponse-object). La propiedad **Id.** especifica el identificador de datos adjuntos que se puede usar con las operaciones [Obtener información de datos adjuntos](#get-attachment-info) y [Obtener datos adjuntos](#get-attachment). | 

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
| **Devuelve** | Un objeto [AttachmentInfo](#attachmentinfo-object) | 

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
Use estas operaciones para almacenar y recuperar datos de estado.

| Operación | DESCRIPCIÓN |
|----|----|
| [Establecer datos de usuario](#set-user-data) | Almacena datos de estado de un usuario específico en un canal. | 
| [Establecer datos de conversación](#set-conversation-data) | Almacena datos de estado de una conversación específica en un canal. | 
| [Establecer datos de conversación privados](#set-private-conversation-data) | Almacena datos de estado de un usuario específico en el contexto de una conversación determinada en un canal. | 
| [Obtener datos de usuario](#get-user-data) | Recupera datos de estado que previamente se almacenaron para un usuario específico de todas las conversaciones en un canal. | 
| [Obtener datos de conversación](#get-conversation-data) | Recupera datos de estado que previamente se almacenaron para una conversación específica en un canal. | 
| [Obtener datos de conversación privados](#get-private-conversation-data) | Recupera datos de estado que previamente se almacenaron para un usuario específico en el contexto de una conversación determinada en un canal. | 
| [Eliminar estado de usuario](#delete-state-for-user) | Elimina los datos de estado almacenados anteriormente de un usuario mediante las operaciones [Establecer datos de usuario](#set-user-data) o [Establecer datos de conversación privados](#set-private-conversation-data). | 

### <a name="set-user-data"></a>Establecer datos de usuario
Almacena datos de estado del usuario especificado en el canal establecido.
```http
POST /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Un objeto [BotData](#botdata-object) |
| **Devuelve** | Un objeto [BotData](#botdata-object) | 

### <a name="set-conversation-data"></a>Establecer datos de conversación
Almacena datos de estado de la conversación especificada en el canal establecido.
```http
POST /v3/botstate/{channelId}/conversations/{conversationId}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Un objeto [BotData](#botdata-object) |
| **Devuelve** | Un objeto [BotData](#botdata-object) | 

### <a name="set-private-conversation-data"></a>Establecer datos de conversación privados
Almacena datos de estado del usuario especificado en el contexto de la conversación definida en el canal establecido.
```http
POST /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId} 
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Un objeto [BotData](#botdata-object) |
| **Devuelve** | Un objeto [BotData](#botdata-object) | 

### <a name="get-user-data"></a>Obtener datos de usuario
Recupera datos de estado que previamente se almacenaron para un usuario específico de todas las conversaciones en un canal determinado.
```http
GET /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Un objeto [BotData](#botdata-object) | 

### <a name="get-conversation-data"></a>Obtener datos de conversación
Recupera datos de estado que previamente se almacenaron para una conversación específica en un canal determinado.
```http
GET /v3/botstate/{channelId}/conversations/{conversationId} 
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Un objeto [BotData](#botdata-object) | 

### <a name="get-private-conversation-data"></a>Obtener datos de conversación privados
Recupera datos de estado que previamente se almacenaron para el usuario especificado en el contexto de la conversación establecida en un canal determinado.
```http
GET /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId} 
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Un objeto [BotData](#botdata-object) | 

### <a name="delete-state-for-user"></a>Eliminar estado de usuario
Elimina los datos de estado almacenados anteriormente de un usuario específico en el canal establecido con el uso de las operaciones [Establecer datos de usuario](#set-user-data) o [Establecer datos de conversación privados](#set-private-conversation-data).
```http
DELETE /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Una matriz de cadenas (ID) | 

## <a id="objects"></a> Esquema

El esquema define el objeto y sus propiedades que el bot puede usar para comunicarse con un usuario. 

| Object | DESCRIPCIÓN |
| ---- | ---- |
| [Objeto Activity](#activity-object) | Define un mensaje que se intercambia entre el bot y el usuario. |
| [Objeto AnimationCard](#animationcard-object) | Define una tarjeta que puede reproducir archivos GIF animados o vídeos cortos. |
| [Objeto Attachment](#attachment-object) | Define información adicional que se va a incluir en el mensaje. Los datos adjuntos pueden ser un archivo multimedia (por ejemplo, audio, vídeo, imagen o archivo) o una tarjeta enriquecida. |
| [Objeto AttachmentData](#attachmentdata-object) | Describe los datos adjuntos. |
| [Objeto AttachmentInfo](#attachmentinfo-object) | Describe los datos adjuntos. |
| [Objeto AttachmentView](#attachmentview-object) | Define una vista de datos adjuntos. |
| [Objeto AttachmentUpload](#attachmentupload-object) | Define los datos adjuntos que se van a cargar. |
| [Objeto AudioCard](#audiocard-object) | Define una tarjeta que puede reproducir un archivo de audio. |
| [Objeto BotData](#botdata-object) | Define los datos de estado de un usuario, una conversación o un usuario en el contexto de una conversación concreta que se almacena con el servicio Bot State. |
| [Objeto CardAction](#cardaction-object) | Define una acción que se va a realizar. |
| [Objeto CardImage](#cardimage-object) | Define una imagen para mostrarla en una tarjeta. |
| [Objeto ChannelAccount](#channelaccount-object) | Define un bot o cuenta de usuario en el canal. |
| [Objeto ConversationAccount](#conversationaccount-object) | Define una conversación en un canal. |
| [Objeto ConversationMembers](#conversationmembers-object) | Define los miembros de una conversación. |
| [Objeto ConversationParameters](#conversationparameters-object) | Define los parámetros para crear una conversación. |
| [Objeto ConversationReference](#conversationreference-object) | Define un punto concreto de una conversación. |
| [Objeto ConversationResourceResponse](#conversationresourceresponse-object) | Define una respuesta para [Crear conversación](#create-conversation). |
| [Objeto ConversationsResult](#conversationsresult-object) | Define el resultado de una llamada a [Obtener conversaciones](#get-conversations). |
| [Objeto Entity](#entity-object) | Define un objeto entidad. |
| [Objeto Error](#error-object) | Define un error. |
| [Objeto ErrorResponse](#errorresponse-object) | Define una respuesta de la API HTTP. |
| [Objeto Fact](#fact-object) | Define un par clave-valor que contiene un hecho. |
| [Objeto Geocoordinates](#geocoordinates-object) | Define una ubicación geográfica mediante las coordenadas del sistema geodésico mundial (WSG84). |
| [Objeto HeroCard](#herocard-object) | Define una tarjeta con una imagen grande, título, texto y botones de acción. |
| [Objeto Identification](#identification-object) | Identifica un recurso. |
| [Objeto MediaEventValue](#mediaeventvalue-object) | Parámetros complementarios para eventos multimedia. |
| [Objeto MediaUrl](#mediaurl-object) | Define la dirección URL al origen de un archivo multimedia. |
| [Objeto Mention](#mention-object) | Define un usuario o un bot que se mencionó en la conversación. |
| [Objeto MessageReaction](#messagereaction-object) | Define una reacción a un mensaje. |
| [Objeto Place](#place-object) | Define un lugar mencionado en la conversación. |
| [Objeto ReceiptCard](#receiptcard-object) | Define una tarjeta que contiene una recepción de una compra. |
| [Objeto ReceiptItem](#receiptitem-object) | Define un elemento de línea dentro de una confirmación. |
| [Objeto ResourceResponse](#resourceresponse-object) | Define un recurso. |
| [Objeto SemanticAction](#semanticaction-object) | Define una referencia a una acción mediante programación. |
| [Objeto SignInCard](#signincard-object) | Define una tarjeta que permite a un usuario iniciar sesión en un servicio. |
| [Objeto SuggestedActions](#suggestedactions-object) | Define las opciones que puede elegir un usuario. |
| [Objeto ThumbnailCard](#thumbnailcard-object) | Define una tarjeta con una imagen en miniatura, título, texto y botones de acción. |
| [Objeto ThumbnailUrl](#thumbnailurl-object) | Define la dirección URL al origen de una imagen. |
| [Objeto Transcript](#transcript-object) | Una colección de actividades que se cargará mediante [Enviar historial de la conversación](#send-conversation-history). |
| [Objeto VideoCard](#videocard-object) | Define una tarjeta que puede reproducir vídeos. |

### <a name="activity-object"></a>Objeto Activity
Define un mensaje que se intercambia entre el bot y el usuario.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **action** | string | La acción que se va a aplicar o que se aplicó. Use la propiedad **type** para determinar el contexto de la acción. Por ejemplo, si **type** es **contactRelationUpdate**, el valor de la propiedad **action** sería **add** si el usuario agregó el bot a su lista de contactos, o **remove** si quitó el bot de su lista de contactos. |
| **attachments** | [Attachment](#attachment-object)[] | Matriz de objetos **Attachment** que define información adicional que se va a incluir en el mensaje. Cada dato adjunto puede ser un archivo multimedia (por ejemplo, audio, vídeo, imagen o archivo) o una tarjeta enriquecida. |
| **attachmentLayout** | string | Diseño de los **datos adjuntos** de la tarjeta enriquecida que el mensaje incluye. Uno de estos valores: **carousel**, **list**. Para más información sobre los datos adjuntos de tarjetas enriquecidas, vea [Incorporación de datos adjuntos de tarjetas enriquecidas a mensajes](bot-framework-rest-connector-add-rich-cards.md). |
| **channelData** | objeto | Objeto que incluye contenido específico de canal. Algunos canales proporcionan características que requieren información adicional que no se puede representar mediante el esquema de datos adjuntos. Para esos casos, establezca esta propiedad en el contenido específico de canal tal como se define en la documentación del canal. Para más información, vea [Implementación de una funcionalidad específica de canal](bot-framework-rest-connector-channeldata.md). |
| **channelId** | string | Identificador que distingue de manera única el canal. Se establece mediante el canal. | 
| **conversation** | [ConversationAccount](#conversationaccount-object) | Un objeto **ConversationAccount** que define la conversación a la que pertenece la actividad. |
| **code** | string | Código que indica por qué la conversación ha finalizado. |
| **entities** | object[] | Matriz de objetos que representa las entidades que se han mencionado en el mensaje. Los objetos de esta matriz pueden ser cualquier objeto <a href="http://schema.org/" target="_blank">Schema.org</a>. Por ejemplo, la matriz puede incluir objetos [Mention](#mention-object) que identifican a alguien a quien se ha mencionado en la conversación y objetos [Place](#place-object) que identifican un lugar mencionado en la conversación. |
| **from** | [ChannelAccount](#channelaccount-object) | Un objeto **ChannelAccount** que especifica el remitente del mensaje. |
| **historyDisclosed** | boolean | Marca que indica si se revela o no el historial. El valor predeterminado es **false**. |
| **id** | string | Identificador que distingue de manera única la actividad en el canal. | 
| **inputHint** | string | Valor que indica si el bot acepta, espera o ignora la entrada del usuario después de que el mensaje se haya entregado al cliente. Uno de estos valores: **acceptingInput**, **expectingInput**, **ignoringInput**. |
| **locale** | string | Configuración regional del idioma que se debe usar para mostrar el texto del mensaje, en el formato `<language>-<country>`. El canal utiliza esta propiedad para indicar el idioma del usuario, para que su bot pueda especificar las cadenas para mostrar en ese idioma. El valor predeterminado es **en-US**. |
| **localTimestamp** | string | Fecha y hora en que se envió el mensaje en la zona horaria local, expresada en el formato <a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a>. |
| **membersAdded** | [ChannelAccount](#channelaccount-object)[] | Matriz de objetos **ChannelAccount** que representa la lista de usuarios que se unieron a la conversación. Presente solo si la actividad **type** es "conversationUpdate" y si los usuarios se unieron a la conversación. | 
| **membersRemoved** | [ChannelAccount](#channelaccount-object)[] | Matriz de objetos **ChannelAccount** que representa la lista de usuarios que abandonaron la conversación. Presente solo si la actividad **type** es "conversationUpdate" y si los usuarios abandonaron la conversación. | 
| **name** | string | Nombre de la operación para invocar o el nombre del evento. |
| **recipient** | [ChannelAccount](#channelaccount-object) | Un objeto **ChannelAccount** que especifica el destinatario del mensaje. |
| **relatesTo** | [ConversationReference](#conversationreference-object) | Un objeto **ConversationReference** que define un punto concreto en una conversación. |
| **replyToId** | string | Identificador del mensaje al que responde este mensaje. Para responder a un mensaje que el usuario envía, establezca esta propiedad con el identificador del mensaje del usuario. No todos los canales admiten respuestas encadenadas. En estos casos, el canal ignorará esta propiedad y usará la semántica ordenada por tiempo (marca de tiempo) para anexar el mensaje a la conversación. | 
| **serviceUrl** | string | Dirección URL que especifica el punto de conexión de servicio del canal. Se establece mediante el canal. | 
| **speak** | string | Texto que pronuncia el bot en un canal habilitado para voz. Para controlar diversas características de voz del bot como voz, velocidad, volumen, pronunciación y tono, especifique esta propiedad con el formato de <a href="https://msdn.microsoft.com/library/hh378377(v=office.14).aspx" target="_blank">lenguaje de marcado de síntesis de voz (SSML)</a>. |
| **suggestedActions** | [SuggestedActions](#suggestedactions-object) | Un objeto **SuggestedActions** que define las opciones entre las que el usuario puede elegir. |
| **summary** | string | Resumen de la información que contiene el mensaje. Por ejemplo, en un mensaje que se envía en un canal de correo electrónico, esta propiedad puede especificar los 50 primeros caracteres del mensaje de correo electrónico. |
| **text** | string | Texto del mensaje que se envía del usuario al bot o del bot al usuario. Consulte la documentación del canal para conocer los límites impuestos en el contenido de esta propiedad. |
| **textFormat** | string | Formato del **texto** del mensaje. Uno de estos valores: **markdown**, **plain**, **xml**. Para más información sobre el formato del texto, vea [Creación de mensajes](bot-framework-rest-connector-create-messages.md). |
| **timestamp** | string | Fecha y hora en que se envió el mensaje en la zona horaria UTC, expresada en el formato <a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a>. |
| **topicName** | string | Tema de la conversación a la que pertenece la actividad. |
| **type** | string | Tipo de actividad. Uno de estos valores: **contactRelationUpdate**, **conversationUpdate**, **deleteUserData**, **message**, **typing**, **event** y **endOfConversation**. Para obtener detalles sobre los tipos de actividad, vea [Introducción a las actividades](bot-framework-rest-connector-activities.md). |
| **value** | objeto | Valor de final abierto. |
| **semanticAction** |[SemanticAction](#semanticaction-object) | Un objeto **SemanticAction** que representa una referencia a una acción mediante programación. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="animationcard-object"></a>Objeto AnimationCard
Define una tarjeta que puede reproducir archivos GIF animados o vídeos cortos.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **autoloop** | boolean | Marca que indica si se debe reproducir la lista de archivos GIF animados cuando finaliza la última. Establezca esta propiedad en **true** para reproducir el archivo animado automáticamente; de lo contrario, establézcala en **false**. El valor predeterminado es **true**. |
| **autostart** | boolean | Marca que indica si se debe reproducir automáticamente el archivo de animación cuando se muestra la tarjeta. Establezca esta propiedad en **true** para reproducir el archivo animado automáticamente; de lo contrario, establézcala en **false**. El valor predeterminado es **true**. |
| **buttons** | [CardAction](#cardaction-object)[] | Matriz de objetos **CardAction** que permite al usuario realizar una o varias acciones. El canal determina el número de botones que se pueden especificar. |
| **duration** | string | La longitud del contenido multimedia, en el [formato de duración ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html). |
| **image** | [ThumbnailUrl](#thumbnailurl-object) | Un objeto **ThumbnailUrl** que especifica la imagen que se mostrará en la tarjeta. |
| **media** | [MediaUrl](#mediaurl-object)[] | Matriz de objetos **MediaUrl** que especifica la lista de archivos GIF animados para reproducir. |
| **shareable** | boolean | Marca que indica si la animación puede compartirse con otros usuarios. Establezca esta propiedad en **true** si la animación puede compartirse; de lo contrario, en **false**. El valor predeterminado es **true**. |
| **subtitle** | string | Subtítulo que se mostrará debajo del título de la tarjeta. |
| **text** | string | Descripción o mensaje para mostrar debajo del título o subtítulo de la tarjeta. |
| **title** | string | Título de la tarjeta. |
| **value** | objeto | Parámetro complementario de esta tarjeta. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="attachment-object"></a>Objeto Attachment
Define información adicional que se va a incluir en el mensaje. Los datos adjuntos pueden ser un archivo multimedia (por ejemplo, audio, vídeo, imagen o archivo) o una tarjeta enriquecida.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **contentType** | string | Tipo de elemento multimedia del contenido de los datos adjuntos. Para los archivos multimedia, establezca esta propiedad en los tipos de elementos multimedia conocidos como **image/png**, **audio/wav** y **video/mp4**. Para tarjetas enriquecidas, establezca esta propiedad en uno de estos tipos específicos del proveedor:<ul><li>**application/vnd.microsoft.card.adaptive**: tarjeta enriquecida que puede contener cualquier combinación de texto, voz, imágenes, botones y campos de entrada. Establezca la propiedad **content** en un objeto <a href="http://adaptivecards.io/documentation/#create-cardschema" target="_blank">AdaptiveCard</a>.</li><li>**application/vnd.microsoft.card.animation**: tarjeta enriquecida que reproduce una animación. Establezca la propiedad **content** en un objeto [AnimationCard](#animationcard-object).</li><li>**application/vnd.microsoft.card.audio**: tarjeta enriquecida que reproduce archivos de audio. Establezca la propiedad **content** en un objeto [AudioCard](#audiocard-object).</li><li>**application/vnd.microsoft.card.video**: tarjeta enriquecida que reproduce vídeos. Establezca la propiedad **content** en un objeto [VideoCard](#videocard-object).</li><li>**application/vnd.microsoft.card.hero**: tarjeta de imagen prominente. Establezca la propiedad **content** en un objeto [HeroCard](#herocard-object).</li><li>**application/vnd.microsoft.card.thumbnail**: tarjeta de miniatura. Establezca la propiedad **content** en un objeto [ThumbnailCard](#thumbnailcard-object).</li><li>**application/vnd.microsoft.com.card.receipt**: tarjeta de recepción. Establezca la propiedad **content** en un objeto [ReceiptCard](#receiptcard-object).</li><li>**application/vnd.microsoft.com.card.signin**: tarjeta de inicio de sesión del usuario. Establezca la propiedad **content** en un objeto [SignInCard](#signincard-object).</li></ul> |
| **contentUrl** | string | Dirección URL del contenido de los datos adjuntos. Por ejemplo, si los datos adjuntos son una imagen, establezca **contentUrl** en la dirección URL que representa la ubicación de la imagen. Los protocolos admitidos son: HTTP, HTTPS, File y Data. |
| **content** | objeto | Contenido de los datos adjuntos. Si los datos adjuntos son una tarjeta enriquecida, establezca esta propiedad en el objeto de tarjeta enriquecida. Esta propiedad y la propiedad **contentUrl** son mutuamente excluyentes. |
| **name** | string | Nombre de los datos adjuntos. |
| **thumbnailUrl** | string | Dirección URL de una imagen en miniatura que el canal puede utilizar si admite el uso de una forma alternativa más pequeña de **content** o **contentUrl**. Por ejemplo, si establece **contentType** en **application/word** y **contentUrl** en la ubicación del documento de Word, puede incluir una imagen en miniatura que representa el documento. El canal puede mostrar la imagen en miniatura en lugar del documento. Cuando el usuario hace clic en la imagen, el canal abre el documento. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="attachmentdata-object"></a>Objeto AttachmentData 
Describe los datos adjuntos.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **name** | string | Nombre de los datos adjuntos. |
| **originalBase64** | string | Contenido de los datos adjuntos. |
| **thumbnailBase64** | string | Contenido de los datos adjuntos en miniatura. |
| **type** | string | Tipo de contenido de los datos adjuntos. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="attachmentinfo-object"></a>Objeto AttachmentInfo
Describe los datos adjuntos.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **name** | string | Nombre de los datos adjuntos. |
| **type** | string | Tipo de contenido de los datos adjuntos. |
| **vistas** | [AttachmentView](#attachmentview-object)[] | Matriz de objetos **AttachmentView** que representan las vistas disponibles para los datos adjuntos. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="attachmentview-object"></a>Objeto AttachmentView
Define una vista de datos adjuntos.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **viewId** | string | Identificador de la vista. |
| **Tamaño** | número | Tamaño del archivo. |

<a href="#objects">Volver a la tabla de esquema</a>

<!-- TODO - can't find in swagger file -->
### <a name="attachmentupload-object"></a>Objeto AttachmentUpload
Define los datos adjuntos que se van a cargar.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **type** | string | Tipo de contenido de los datos adjuntos. | 
| **name** | string | Nombre de los datos adjuntos. | 
| **originalBase64** | string | Datos binarios que representan el contenido de la versión original del archivo. |
| **thumbnailBase64** | string | Datos binarios que representan el contenido de la versión en miniatura del archivo. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="audiocard-object"></a>Objeto AudioCard
Define una tarjeta que puede reproducir un archivo de audio.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **aspect** | string | Relación de aspecto de la miniatura que se especifica en la propiedad **image**. Los valores válidos son **16:9** y **9:16**. |
| **autoloop** | boolean | Marca que indica si se debe reproducir la lista de archivos de audio cuando finaliza la última. Establezca esta propiedad en **true** para reproducir los archivos de audio automáticamente; de lo contrario, establézcala en **false**. El valor predeterminado es **true**. |
| **autostart** | boolean | Marca que indica si se debe reproducir automáticamente el archivo de audio cuando se muestra la tarjeta. Establezca esta propiedad en **true** para reproducir el audio automáticamente; de lo contrario, establézcala en **false**. El valor predeterminado es **true**. |
| **buttons** | [CardAction](#cardaction-object)[] | Matriz de objetos **CardAction** que permite al usuario realizar una o varias acciones. El canal determina el número de botones que se pueden especificar. |
| **duration** | string | La longitud del contenido multimedia, en el [formato de duración ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html). |
| **image** | [ThumbnailUrl](#thumbnailurl-object) | Un objeto **ThumbnailUrl** que especifica la imagen que se mostrará en la tarjeta. |
| **media** | [MediaUrl](#mediaurl-object)[] | Matriz de objetos **MediaUrl** que especifica la lista de archivos de audio para reproducir. |
| **shareable** | boolean | Marca que indica si los archivos de audio compartirse con otros usuarios. Establezca esta propiedad en **true** si el audio puede compartirse; de lo contrario, en **false**. El valor predeterminado es **true**. |
| **subtitle** | string | Subtítulo que se mostrará debajo del título de la tarjeta. |
| **text** | string | Descripción o mensaje para mostrar debajo del título o subtítulo de la tarjeta. |
| **title** | string | Título de la tarjeta. |
| **value** | objeto | Parámetro complementario de esta tarjeta. |

<a href="#objects">Volver a la tabla de esquema</a>

<!-- TODO - can't find in swagger file -->
### <a name="botdata-object"></a>Objeto BotData
Define los datos de estado de un usuario, una conversación o un usuario en el contexto de una conversación concreta que se almacena con el servicio Bot State.<br/><br/>

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **Datos** | objeto | En una solicitud, un objeto JSON que especifica las propiedades y los valores para almacenar mediante el servicio Bot State. En una respuesta, un objeto JSON que especifica las propiedades y los valores que se han almacenado con el servicio Bot State. | 
| **eTag** | string | El valor de etiqueta de entidad que puede usar para controlar la simultaneidad de los datos que se almacenan con el servicio Bot State. Para más información, vea [Administración de datos de estado](bot-framework-rest-state.md). | 

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="cardaction-object"></a>Objeto CardAction
Define una acción que se va a realizar.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **image** | string | Dirección URL de una imagen para mostrarla en | 
| **text** | string | Texto para la acción |
| **title** | string | Texto del botón. Solo es aplicable para la acción de un botón. |
. Solo es aplicable para la acción de un botón. |
| **type** | string | Tipo de acción que se va a realizar. Para obtener una lista de valores válidos, vea [Incorporación de tarjetas enriquecidas a mensajes](bot-framework-rest-connector-add-rich-cards.md). |
| **value** | objeto | Parámetro complementario de la acción. El valor de esta propiedad variará según el objeto **type** de la acción. Para más información, vea [Incorporación de datos adjuntos de tarjetas enriquecidas a mensajes](bot-framework-rest-connector-add-rich-cards.md). |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="cardimage-object"></a>Objeto CardImage
Define una imagen para mostrarla en una tarjeta.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **alt** | string | Descripción de la imagen. Debe incluir la descripción para admitir la accesibilidad. |
| **tap** | [CardAction](#cardaction-object) | Un objeto **CardAction** que especifica la acción que se realizará si el usuario pulsa o hace clic en la imagen. |
| **url** | string | Dirección URL del origen de la imagen o del binario Base64 de la imagen (por ejemplo, `data:image/png;base64,iVBORw0KGgo...`). |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="channelaccount-object"></a>Objeto ChannelAccount
Define un bot o cuenta de usuario en el canal.<br/><br/>

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **id** | string | Identificador único del usuario o bot en este canal. |
| **name** | string | Nombre descriptivo del bot o el usuario. |
| **aadObjectId** | string | Identificador de objeto de esta cuenta dentro de Azure Active Directory. |
| **role** | string enum | Rol de la entidad que se encuentra detrás de la cuenta. `user` o `bot`. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="conversationaccount-object"></a>Objeto ConversationAccount
Define una conversación en un canal.<br/><br/>

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **id** | string | Identificador de la conversación. El identificador es único para cada canal. Si el canal inicia la conversación, establece este identificador; de lo contrario, el bot establece esta propiedad en el identificador que obtiene con la respuesta cuando inicia la conversación (vea Inicio de una conversación). |
| **isGroup** | boolean | Marca que indica si la conversación contiene más de dos participantes en el momento en que se generó la actividad. Se establece en **true** si se trata de una conversación de grupo; en caso contrario, en **false**. El valor predeterminado es **false**. |
| **name** | string | Un nombre para mostrar que se puede usar para identificar la conversación. |
| **conversationType** | string | Indica el tipo de conversación en canales que distinguen entre tipos de conversación (por ejemplo, grupo o personal). |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="conversationmembers-object"></a>Objeto ConversationMembers
Define los miembros de una conversación.<br/><br/>

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **id** | string | El identificador de la conversación. |
| **members** | array | Una matriz de objetos [ChannelAccount](#channelaccount-object). |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="conversationparameters-object"></a>Objeto ConversationParameters
Define los parámetros para crear una conversación.<br/><br/>

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **isGroup** | boolean | Indica si se trata de una conversación de grupo. |
| **bot** | [ChannelAccount](#channelaccount-object) | Información de la cuenta de canal necesaria para enrutar un mensaje al bot. |
| **members** | Matriz [ChannelAccount](#channelaccount-object) | Información de la cuenta de canal necesaria para enrutar un mensaje a cada usuario. |
| **topicName** | string | Opcional, tema de la conversación. Esta propiedad solo se usa si un canal la admite. |
| **tennantId** | string | Opcional; el identificador del inquilino en el que se debe crear la conversación. |
| **activity** | [Actividad](#activity-object) | Opcional; el mensaje inicial que se envía a la conversación cuando se crea. |
| **channelData** | objeto | Carga útil específica del canal para crear la conversación. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="conversationreference-object"></a>Objeto ConversationReference
Define un punto concreto de una conversación.<br/><br/>

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **activityId** | string | Identificador que identifica de forma única la actividad a la que hace referencia este objeto. | 
| **bot** | [ChannelAccount](#channelaccount-object) | Un objeto **ChannelAccount** que identifica el bot en la conversación a la que este objeto hace referencia. |
| **channelId** | string | Un identificador que identifica de forma única el canal en la conversación a la que este objeto hace referencia. | 
| **conversation** | [ConversationAccount](#conversationaccount-object) | Un objeto **ConversationAccount** que define la conversación a la que este objeto hace referencia. |
| **serviceUrl** | string | Dirección URL que especifica el punto de conexión de servicio del canal de la conversación a la que este objeto hace referencia. | 
| **user** | [ChannelAccount](#channelaccount-object) | Un objeto **ChannelAccount** que identifica al usuario en la conversación a la que este objeto hace referencia. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="conversationresourceresponse-object"></a>Objeto ConversationResourceResponse
Define una respuesta para [Crear conversación](#create-conversation).<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **activityId** | string | Identificador de la actividad, si se envía. |
| **id** | string | Identificador del recurso. |
| **serviceUrl** | string | Punto de conexión de servicio en el que se pueden realizar las operaciones relacionadas con la conversación. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="conversationsresult-object"></a>Objeto ConversationsResult
Define el resultado de [Obtener conversaciones](#get-conversations).<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **continuationToken** | string | El token de continuación que se puede usar en las llamadas subsiguientes a [Obtener conversaciones](#get-conversations). |
| **conversaciones** | array | Una matriz de objetos [ConversationMembers](#conversationmembers-object) |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="error-object"></a>Objeto Error
Define un error.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **code** | string | Código de error. |
| **message** | string | Descripción del error. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="entity-object"></a>Objeto Entity
Define un objeto entidad.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **type** | string | Tipo de entidad. Normalmente contiene tipos de schema.org. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="errorresponse-object"></a>Objeto ErrorResponse
Define una respuesta de la API HTTP.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **error** | [Error](#error-object) | Un objeto **Error** que contiene información sobre el error. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="fact-object"></a>Objeto Fact
Define un par clave-valor que contiene un hecho.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **key** | string | Nombre del hecho. Por ejemplo, **inserción en el repositorio**. La clave se usa como una etiqueta al mostrar el valor del hecho. |
| **value** | string | Valor del hecho. Por ejemplo, **10 de octubre de 2016**. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="geocoordinates-object"></a>Objeto Geocoordinates
Define una ubicación geográfica mediante las coordenadas del sistema geodésico mundial (WSG84).<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **elevation** | número | Elevación de la ubicación. |
| **name** | string | Nombre de la ubicación. |
| **latitude** | número | Latitud de la ubicación. |
| **longitude** | número | Longitud de la ubicación. |
| **type** | string | El tipo de este objeto. Siempre se establece en **GeoCoordinates**. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="herocard-object"></a>Objeto HeroCard
Define una tarjeta con una imagen grande, título, texto y botones de acción.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | Matriz de objetos **CardAction** que permite al usuario realizar una o varias acciones. El canal determina el número de botones que se pueden especificar. |
| **images** | [CardImage](#cardimage-object)[] | Matriz de objetos **CardImage** que especifica la imagen que se mostrará en la tarjeta. Una tarjeta Hero contiene solo una imagen. |
| **subtitle** | string | Subtítulo que se mostrará debajo del título de la tarjeta. |
| **tap** | [CardAction](#cardaction-object) | Un objeto **CardAction** que especifica la acción que se realizará si el usuario pulsa o hace clic en la tarjeta. Puede ser la misma acción que uno de los botones o una acción diferente. |
| **text** | string | Descripción o mensaje para mostrar debajo del título o subtítulo de la tarjeta. |
| **title** | string | Título de la tarjeta. |

<a href="#objects">Volver a la tabla de esquema</a>

<!--TODO can't find-->
### <a name="identification-object"></a>Objeto Identification
Identifica un recurso.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **id** | string | Identificador que distingue de manera única el recurso. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="mediaeventvalue-object"></a>Objeto MediaEventValue 
Parámetros complementarios para eventos multimedia.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **cardValue** | objeto | Parámetro de devolución de llamada especificado en el campo **Valor** de la tarjeta multimedia que originó este evento. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="mediaurl-object"></a>Objeto MediaUrl
Define la dirección URL al origen de un archivo multimedia.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **profile** | string | Sugerencia que describe el contenido del elemento multimedia. |
| **url** | string | Dirección URL del origen del archivo multimedia. |

<a href="#objects">Volver a la tabla de esquema</a>

<!--TODO can't find-->
### <a name="mention-object"></a>Objeto Mention
Define un usuario o un bot que se mencionó en la conversación.<br/><br/> 


|          Propiedad          |                   Escriba                   |                                                                                                                                                                                                                           DESCRIPCIÓN                                                                                                                                                                                                                            |
|----------------------------|------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <strong>mentioned</strong> | [ChannelAccount](#channelaccount-object) | Un objeto <strong>ChannelAccount</strong> que especifica el usuario o el bot que se ha mencionado. Tenga en cuenta que algunos canales como Slack asignan nombres por conversación, por lo que es posible que el nombre mencionado del bot (en la propiedad <strong>recipient</strong> del mensaje) difiera del identificador especificado cuando se [registró](../bot-service-quickstart-registration.md) el bot. Sin embargo, los identificadores de cuenta podrían ser los mismos. |
|   <strong>text</strong>    |                  string                  |                                                                                                                         El usuario o bot como se mencionaron en la conversación. Por ejemplo, si el mensaje es "@ColorBot, elígeme un color nuevo", esta propiedad se establecería en <strong>@ColorBot</strong>. No todos los canales establecen esta propiedad.                                                                                                                          |
|   <strong>type</strong>    |                  string                  |                                                                                                                                                                                                   Tipo de este objeto. Siempre se establece en <strong>Mention</strong>.                                                                                                                                                                                                    |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="messagereaction-object"></a>Objeto MessageReaction
Define una reacción a un mensaje.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **type** | string | Tipo de reacción. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="place-object"></a>Objeto Place
Define un lugar mencionado en la conversación.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **address** | objeto |  Dirección de un lugar. Esta propiedad puede ser `string` o un objeto complejo de tipo `PostalAddress`. |
| **geo** | [GeoCoordinates](#geocoordinates-object) | Un objeto **GeoCoordinates** que especifica las coordenadas geográficas de un lugar. |
| **hasMap** | objeto | Mapa del lugar. Esta propiedad puede ser `string` (URL) o un objeto complejo de tipo `Map`. |
| **name** | string | Nombre del lugar. |
| **type** | string | Tipo de este objeto. Siempre se establece en **Place**. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="receiptcard-object"></a>Objeto ReceiptCard
Define una tarjeta que contiene una recepción de una compra.<br/><br/>

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | Matriz de objetos **CardAction** que permite al usuario realizar una o varias acciones. El canal determina el número de botones que se pueden especificar. |
| **facts** | [Fact](#fact-object)[] | Matriz de objetos **Fact** que especifican información sobre la compra. Por ejemplo, la lista de hechos de la factura de la estancia en un hotel puede incluir la fecha de entrada y la fecha de salida. El canal determina el número de hechos que se pueden especificar. |
| **items** | [ReceiptItem](#receiptitem-object)[] | Matriz de objetos **ReceiptItem** que especifican los elementos comprados. |
| **tap** | [CardAction](#cardaction-object) | Un objeto **CardAction** que especifica la acción que se realizará si el usuario pulsa o hace clic en la tarjeta. Puede ser la misma acción que uno de los botones o una acción diferente. |
| **tax** | string | Una cadena con formato de moneda que especifica la cuantía de los impuestos aplicados a la compra. |
| **title** | string | Título que se muestra en la parte superior de la factura. |
| **total** | string | Una cadena con formato de moneda que especifica el precio total de la compra, incluidos todos los impuestos aplicables. |
| **vat** | string | Una cadena con formato de moneda que especifica la cuantía del impuesto sobre el valor añadido (IVA) aplicado al precio de compra. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="receiptitem-object"></a>Objeto ReceiptItem
Define un elemento de línea dentro de una confirmación.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **image** | [CardImage](#cardimage-object) | Un objeto **CardImage** que especifica la imagen en miniatura que se mostrará junto al elemento de línea.  |
| **price** | string | Una cadena con formato de moneda que especifica el precio total de todas las unidades compradas. |
| **quantity** | string | Una cadena numérica que especifica el número de unidades compradas. |
| **subtitle** | string | Subtítulo para mostrar debajo del título del elemento de línea. |
| **tap** | [CardAction](#cardaction-object) | Un objeto **CardAction** que especifica la acción que se realizará si el usuario pulsa o hace clic en el elemento de línea. |
| **text** | string | Descripción del elemento de línea. |
| **title** | string | Título del elemento de línea. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="resourceresponse-object"></a>Objeto ResourceResponse
Define una respuesta que contiene un identificador de recurso.<br/><br/>

|      Propiedad       |  Escriba  |                DESCRIPCIÓN                |
|---------------------|--------|-------------------------------------------|
| <strong>id</strong> | string | Identificador que distingue de manera única el recurso. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="semanticaction-object"></a>Objeto SemanticAction
Define una referencia a una acción mediante programación.<br/><br/>

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **id** | string | Identificación de esta acción |
| **entities** | [Entidad](#entity-object) | Entidades asociadas a esta acción |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="signincard-object"></a>Objeto SignInCard
Define una tarjeta que permite a un usuario iniciar sesión en un servicio.<br/><br/>

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | Matriz de objetos **CardAction** que permite al usuario iniciar sesión en un servicio. El canal determina el número de botones que se pueden especificar. |
| **text** | string | Descripción o mensaje para incluir en la tarjeta de inicio de sesión. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="suggestedactions-object"></a>Objeto SuggestedActions
Define las opciones que puede elegir un usuario.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **actions** | [CardAction](#cardaction-object)[] | Matriz de objetos **CardAction** que definen las acciones sugeridas. |
| **to** | string[] | Matriz de cadenas que contiene los identificadores de los destinatarios a los que se mostrarán las acciones sugeridas. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="thumbnailcard-object"></a>Objeto ThumbnailCard
Define una tarjeta con una imagen en miniatura, título, texto y botones de acción.<br/><br/>

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | Matriz de objetos **CardAction** que permite al usuario realizar una o varias acciones. El canal determina el número de botones que se pueden especificar. |
| **images** | [CardImage](#cardimage-object)[] | Matriz de objetos **CardImage** que especifican las imágenes en miniatura que se mostrarán en la tarjeta. El canal determina el número de imágenes en miniatura que se pueden especificar. |
| **subtitle** | string | Subtítulo que se mostrará debajo del título de la tarjeta. |
| **tap** | [CardAction](#cardaction-object) | Un objeto **CardAction** que especifica la acción que se realizará si el usuario pulsa o hace clic en la tarjeta. Puede ser la misma acción que uno de los botones o una acción diferente. |
| **text** | string | Descripción o mensaje para mostrar debajo del título o subtítulo de la tarjeta. |
| **title** | string | Título de la tarjeta. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="thumbnailurl-object"></a>Objeto ThumbnailUrl
Define la dirección URL al origen de una imagen.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **alt** | string | Descripción de la imagen. Debe incluir la descripción para admitir la accesibilidad. |
| **url** | string | Dirección URL del origen de la imagen o del binario Base64 de la imagen (por ejemplo, `data:image/png;base64,iVBORw0KGgo...`). |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="transcript-object"></a>Objeto Transcript
Una colección de actividades que se cargará mediante [Enviar historial de la conversación](#send-conversation-history).<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **activities** | array | Una matriz de objetos [Activity](#activity-object). Deben tener un identificador único y una marca de tiempo. |

<a href="#objects">Volver a la tabla de esquema</a>

### <a name="videocard-object"></a>Objeto VideoCard
Define una tarjeta que puede reproducir vídeos.<br/><br/>

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **aspect** | string | Relación de aspecto de un vídeo (p. ej.: 16:9, 4:3).|
| **autoloop** | boolean | Marca que indica si se debe reproducir la lista de vídeos cuando finaliza la última. Establezca esta propiedad en **true** para reproducir los vídeos automáticamente; de lo contrario, establézcala en **false**. El valor predeterminado es **true**. |
| **autostart** | boolean | Marca que indica si se deben reproducir automáticamente los vídeos cuando se muestra la tarjeta. Establezca esta propiedad en **true** para reproducir los vídeos automáticamente; de lo contrario, establézcala en **false**. El valor predeterminado es **true**. |
| **buttons** | [CardAction](#cardaction-object)[] | Matriz de objetos **CardAction** que permite al usuario realizar una o varias acciones. El canal determina el número de botones que se pueden especificar. |
| **duration** | string | La longitud del contenido multimedia, en el [formato de duración ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html). |
| **image** | [ThumbnailUrl](#thumbnailurl-object) | Un objeto **ThumbnailUrl** que especifica la imagen que se mostrará en la tarjeta. |
| **media** | [MediaUrl](#mediaurl-object)[] | Matriz de objetos **MediaUrl** que especifica la lista de vídeos para mostrar. |
| **shareable** | boolean | Marca que indica si los vídeos pueden compartirse con otros usuarios. Establezca esta propiedad en **true** si los vídeos pueden compartirse; de lo contrario, en **false**. El valor predeterminado es **true**. |
| **subtitle** | string | Subtítulo que se mostrará debajo del título de la tarjeta. |
| **text** | string | Descripción o mensaje para mostrar debajo del título o subtítulo de la tarjeta. |
| **title** | string | Título de la tarjeta. |
| **value** | objeto | Parámetro complementario de esta tarjeta.|

<a href="#objects">Volver a la tabla de esquema</a>

---
title: 'Referencia de API: Direct Line API 3.0 | Microsoft Docs'
description: Obtenga información sobre encabezados, códigos de estado HTTP, esquemas, operaciones y objetos en Direct Line API 3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 36a95ec60af117a9a13843dc078bd9bd5cfde635
ms.sourcegitcommit: 6ed90a4c90add925a0a865be1127041b7775fd3d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/30/2018
ms.locfileid: "50234478"
---
# <a name="api-reference---direct-line-api-30"></a>Referencia de API: Direct Line API 3.0

Para permitir que su aplicación cliente se comunique con su bot, puede usar Direct Line API 3.0. Direct Line API 3.0 usa REST y JSON estándares a través de HTTPS.

## <a name="base-uri"></a>URI base

Para obtener acceso a Direct Line API 3.0, use este URI base para todas las solicitudes de API:

`https://directline.botframework.com`

## <a name="headers"></a>encabezados

Además de los encabezados de solicitud HTTP estándar, una solicitud de Direct Line API debe incluir un encabezado `Authorization` que especifique un secreto o un token para autenticar al cliente que emite la solicitud. Especifique el encabezado `Authorization` con este formato:

```http
Authorization: Bearer SECRET_OR_TOKEN
```

Para más información sobre cómo obtener un secreto o un token que el cliente pueda usar para autenticar sus solicitudes de Direct Line API, consulte [Autenticación](bot-framework-rest-direct-line-3-0-authentication.md).

## <a name="http-status-codes"></a>Códigos de estado HTTP

El <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank">código de estado HTTP</a> que se devuelve con cada respuesta indica el resultado de la solicitud correspondiente. 

| Código de estado HTTP | Significado |
|----|----|
| 200 | La solicitud finalizó correctamente. |
| 201 | La solicitud finalizó correctamente. |
| 202 | La solicitud se ha aceptado para procesamiento. |
| 204 | La solicitud se realizó correctamente, pero no se devolvió ningún contenido. |
| 400 | La solicitud tenía formato incorrecto o era incorrecta por otro motivo. |
| 401 | El cliente no está autorizado a realizar solicitudes. A menudo, este código de estado se produce porque falta el encabezado `Authorization` o tiene un formato incorrecto. |
| 403 | El cliente no tiene permitido llevar a cabo la operación solicitada. Si la solicitud especificaba un token que previamente era válido, pero ha expirado, la propiedad `code` del [Error](bot-framework-rest-connector-api-reference.md#error-object) que se devuelve dentro del objeto [ErrorResponse](bot-framework-rest-connector-api-reference.md#errorresponse-object) se establece en `TokenExpired`. |
| 404 | No se encontró el recurso solicitado. Normalmente, este código de estado indica un URI de solicitud no válido. |
| 500 | Se ha producido un error interno del servidor en el servicio de Direct Line. |
| 502 | El bot no está disponible o devolvió un error. **Se trata de un código de error común.** |

> [!NOTE]
> El código de estado HTTP 101 se usa en la ruta de acceso de conexión de WebSocket, aunque es probable que el cliente de WebSocket controle esto.

### <a name="errors"></a>Errors

Cualquier respuesta que especifique un código de estado HTTP en el rango de 4xx o 5xx incluirá un objeto [ErrorResponse](bot-framework-rest-connector-api-reference.md#errorresponse-object) en el cuerpo de la respuesta que proporciona información sobre el error. Si recibe una respuesta de error en el rango de 4xx, inspeccione el objeto **ErrorResponse** para identificar la causa del error y resolver el problema antes de volver a enviar la solicitud.

> [!NOTE]
> Los códigos de estado HTTP y los valores especificados en la propiedad `code` dentro del objeto **ErrorResponse** son estables. Los valores especificados en la propiedad `message` dentro del objeto **ErrorResponse** pueden cambiar con el tiempo.

Los fragmentos de código siguientes muestran un ejemplo de solicitud y la respuesta de error resultante.

#### <a name="request"></a>Solicitud

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
[detail omitted]
```

#### <a name="response"></a>Response
```http
HTTP/1.1 502 Bad Gateway
[other headers]
```
```json
{
    "error": {
        "code": "BotRejectedActivity",
        "message": "Failed to send activity: bot returned an error"
    }
}
```

## <a name="token-operations"></a>Operaciones con tokens 
Use estas operaciones para crear o actualizar un token que un cliente pueda utilizar para acceder a una conversación única.

| Operación | DESCRIPCIÓN |
|----|----|
| [Generar token](#generate-token) | Genera un token para una conversación nueva. | 
| [Actualizar token](#refresh-token) | Actualiza un token. | 

### <a name="generate-token"></a>Generar token
Genera un token válido para una conversación. 
```http 
POST /v3/directline/tokens/generate
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Un objeto [Conversation](#conversation-object) | 

### <a name="refresh-token"></a>Actualizar token
Actualiza el token. 
```http 
POST /v3/directline/tokens/refresh
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Un objeto [Conversation](#conversation-object) | 

## <a name="conversation-operations"></a>Operaciones con conversaciones 
Use estas operaciones para abrir una conversación con sus bot e intercambie actividades entre el cliente y el bot.

| Operación | DESCRIPCIÓN |
|----|----|
| [Iniciar conversación](#start-conversation) | Abre una nueva conversación con el bot. | 
| [Obtener información de la conversación](#get-conversation-information) | Le permite obtener información sobre una conversación existente. Esta operación genera una nueva dirección URL de flujo de WebSocket que un cliente puede utilizar para [reconectarse](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md) a una conversación. |
| [Obtener actividades](#get-activities) | Recupera actividades del bot. |
| [Enviar una actividad](#send-an-activity) | Envía una actividad al bot. | 
| [Cargar y enviar archivos](#upload-send-files) | Carga y envía archivos como adjuntos. |

### <a name="start-conversation"></a>Iniciar conversación
Abre una nueva conversación con el bot. 
```http 
POST /v3/directline/conversations
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Un objeto [Conversation](#conversation-object) | 

### <a name="get-conversation-information"></a>Obtener información de la conversación
Obtiene información sobre una conversación existente y también genera una nueva dirección URL de flujo de WebSocket que un cliente puede utilizar para [reconectarse](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md) a una conversación. Como alternativa, se puede proporcionar el parámetro `watermark` en el URI de la solicitud para indicar el mensaje más reciente visto por el cliente.
```http 
GET /v3/directline/conversations/{conversationId}?watermark={watermark_value}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Un objeto [Conversation](#conversation-object) | 

### <a name="get-activities"></a>Obtener actividades
Recupera las actividades del bot para la conversación especificada. Como alternativa, se puede proporcionar el parámetro `watermark` en el URI de la solicitud para indicar el mensaje más reciente visto por el cliente. 

```http 
GET /v3/directline/conversations/{conversationId}/activities?watermark={watermark_value}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Un objeto [ActivitySet](#activityset-object). La respuesta contiene `watermark` como propiedad del objeto `ActivitySet`. Los clientes deben desplazarse por las actividades disponibles haciendo avanzar el valor `watermark` hasta que no se devuelvan actividades. | 

### <a name="send-an-activity"></a>Enviar una actividad
Envía una actividad al bot. 
```http 
POST /v3/directline/conversations/{conversationId}/activities
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Un objeto [Activity](bot-framework-rest-connector-api-reference.md#activity-object) |
| **Devuelve** | Un objeto [ResourceResponse](bot-framework-rest-connector-api-reference.md#resourceresponse-object) que contiene una propiedad `id` que especifica el id. de la actividad que se envió al bot. | 

### <a id="upload-send-files"></a> Cargar y enviar archivos
Carga y envía archivos como adjuntos. Establecer el parámetro `userId` en el URI de la solicitud para especificar el id. del usuario que envía los adjuntos.
```http 
POST /v3/directline/conversations/{conversationId}/upload?userId={userId}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Para un único dato adjunto, rellene el cuerpo de la solicitud con el contenido del archivo. Para varios archivos adjuntos, cree un cuerpo de solicitud de varias partes que contenga una parte para cada archivo adjunto y, además (de manera opcional), una parte para el objeto [Activity](bot-framework-rest-connector-api-reference.md#activity-object) que debe actuar como contenedor para los archivos adjuntos especificados. Para obtener más información, consulte [Envío de una actividad al bot](bot-framework-rest-direct-line-3-0-send-activity.md). |
| **Devuelve** | Un objeto [ResourceResponse](bot-framework-rest-connector-api-reference.md#resourceresponse-object) que contiene una propiedad `id` que especifica el id. de la actividad que se envió al bot. | 

> [!NOTE]
> Los archivos cargados se eliminan después de 24 horas.

## <a name="schema"></a>Esquema

El esquema de Direct Line 3.0 incluye todos los objetos definidos por el [esquema de Bot Framework v3](bot-framework-rest-connector-api-reference.md#objects), así como el objeto `ActivitySet` y el objeto `Conversation`.

### <a name="activityset-object"></a>Objeto ActivitySet 
Define un conjunto de actividades.<br/><br/>

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **activities** | [Actividad](bot-framework-rest-connector-api-reference.md#activity-object)[] | Matriz de objetos **Activity**. |
| **watermark** | string | Marca de agua máxima de actividades dentro del conjunto. Un cliente puede utilizar el valor `watermark` para indicar el mensaje más reciente que ha visto al [recuperar actividades del bot](bot-framework-rest-direct-line-3-0-receive-activities.md#http-get) o al [generar una nueva dirección URL del flujo de WebSocket](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md). |

### <a name="conversation-object"></a>Objeto Conversation
Define una conversación de Direct Line.<br/><br/>

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **conversationId** | string | Identificador que identifica de forma única la conversación para la cual el token especificado es válido. |
| **expires_in** | número | Número de segundos hasta que expira el token. |
| **streamUrl** | string | Dirección URL del flujo de mensajes de la conversación. |
| **token** | string | Token válido para la conversación especificada. |

### <a name="activities"></a>Actividades

Para cada [Activity](bot-framework-rest-connector-api-reference.md#activity-object) que un cliente recibe de un bot a través de Direct Line:

- Se conservan las tarjetas de archivos adjuntos.
- Las direcciones URL para los archivos adjuntos cargados se ocultan con un vínculo privado.
- La propiedad `channelData` se conserva sin modificaciones. 

Es posible que los clientes [reciban](bot-framework-rest-direct-line-3-0-receive-activities.md) varias actividades del bot como parte de un objeto [ActivitySet](#activityset-object). 

Cuando un cliente envía un objeto [Activity](bot-framework-rest-connector-api-reference.md#activity-object) a un bot a través de Direct Line:

- La propiedad `type` especifica el tipo de actividad que envía (normalmente **mensaje**).
- La propiedad `from` se debe rellenar con un id. de usuario elegido por el cliente.
- Los archivos adjuntos pueden contener direcciones URL a los recursos existentes o direcciones URL cargadas a través del punto de conexión de los archivos adjuntos de Direct Line.
- La propiedad `channelData` se conserva sin modificaciones.
- El tamaño total de la actividad, cuando se serializa en JSON, no debe superar los 300 K caracteres.

Es posible que los clientes [envíen](bot-framework-rest-direct-line-3-0-send-activity.md) una única actividad por solicitud. 

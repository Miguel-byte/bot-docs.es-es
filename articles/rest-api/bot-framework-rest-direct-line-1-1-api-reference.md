---
title: 'Referencia de API: Direct Line API 1.1 | Microsoft Docs'
description: Obtenga información sobre encabezados, códigos de estado HTTP, esquemas, operaciones y objetos en Direct Line API 1.1.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 3607957cd5cb8738e8268ece6eba4417250bc596
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997967"
---
# <a name="api-reference---direct-line-api-11"></a>Referencia de API: Direct Line API 1.1

> [!IMPORTANT]
> Este artículo contiene información de referencia para Direct Line API 1.1. Si va a crear una nueva conexión entre la aplicación cliente y el bot, use [Direct Line API 3.0](bot-framework-rest-direct-line-3-0-api-reference.md) en su lugar.

Para permitir que su aplicación cliente se comunique con su bot, puede usar Direct Line API 1.1. Direct Line API 1.1 usa REST y JSON estándar del sector mediante HTTPS.

## <a name="base-uri"></a>URI base

Para acceder a Direct Line API 1.1, use este URI base para todas las solicitudes de API:

`https://directline.botframework.com`

## <a name="headers"></a>encabezados

Además de los encabezados de solicitud HTTP estándar, una solicitud de Direct Line API debe incluir un encabezado `Authorization` que especifique un secreto o un token para autenticar al cliente que emite la solicitud. Puede especificar el encabezado `Authorization` mediante el esquema "Bearer" o "BotConnector". 

**Esquema Bearer**:
```http
Authorization: Bearer SECRET_OR_TOKEN
```

**Esquema BotConnector**:
```http
Authorization: BotConnector SECRET_OR_TOKEN
```

Para más información sobre cómo obtener un secreto o un token que el cliente pueda usar para autenticar sus solicitudes de Direct Line API, consulte [Autenticación](bot-framework-rest-direct-line-1-1-authentication.md).

## <a name="http-status-codes"></a>Códigos de estado HTTP

El <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank">código de estado HTTP</a> que se devuelve con cada respuesta indica el resultado de la solicitud correspondiente. 

| Código de estado HTTP | Significado |
|----|----|
| 200 | La solicitud finalizó correctamente. |
| 204 | La solicitud se realizó correctamente, pero no se devolvió ningún contenido. |
| 400 | La solicitud tenía formato incorrecto o era incorrecta por otro motivo. |
| 401 | El cliente no está autorizado a realizar solicitudes. A menudo, este código de estado se produce porque falta el encabezado `Authorization` o tiene un formato incorrecto. |
| 403 | El cliente no tiene permitido llevar a cabo la operación solicitada. A menudo, este código de estado se produce porque el encabezado `Authorization` especifica un token o un secreto no válidos. |
| 404 | No se encontró el recurso solicitado. Normalmente, este código de estado indica un URI de solicitud no válido. |
| 500 | Se ha producido un error interno del servidor en el servicio de Direct Line |
| 502 | Se produjo un error en el bot; el bot no está disponible o devolvió un error.  **Se trata de un código de error común.** |

## <a name="token-operations"></a>Operaciones con tokens 
Use estas operaciones para crear o actualizar un token que un cliente pueda utilizar para acceder a una conversación única.

| Operación | DESCRIPCIÓN |
|----|----|
| [Generar token](#generate-token) | Genera un token para una conversación nueva. | 
| [Actualizar token](#refresh-token) | Actualiza un token. | 

### <a name="generate-token"></a>Generar token
Genera un token válido para una conversación. 
```http 
POST /api/tokens/conversation
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Una cadena que representa el token | 

### <a name="refresh-token"></a>Actualizar token
Actualiza el token. 
```http 
GET /api/tokens/{conversationId}/renew
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Una cadena que representa el nuevo token | 

## <a name="conversation-operations"></a>Operaciones con conversaciones 
Use estas operaciones para abrir una conversación con el bot e intercambiar mensajes entre el cliente y el bot.

| Operación | DESCRIPCIÓN |
|----|----|
| [Iniciar conversación](#start-conversation) | Abre una nueva conversación con el bot. | 
| [Get Messages](#get-messages) | Recibe mensajes del bot. |
| [Enviar un mensaje](#send-a-message) | Envía un mensaje al bot. | 
| [Cargar y enviar archivos](#upload-send-files) | Carga y envía archivos como adjuntos. |

### <a name="start-conversation"></a>Iniciar conversación
Abre una nueva conversación con el bot. 
```http 
POST /api/conversations
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Un objeto [Conversation](#conversation-object) | 

### <a name="get-messages"></a>Obtener mensajes
Recupera los mensajes del bot para la conversación especificada. Establezca el parámetro `watermark` en el URI de la solicitud para indicar el mensaje más reciente visto por el cliente. 

```http
GET /api/conversations/{conversationId}/messages?watermark={watermark_value}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | N/D |
| **Devuelve** | Un objeto [MessageSet](#messageset-object). La respuesta contiene `watermark` como propiedad del objeto `MessageSet`. Los clientes deben desplazarse por los mensajes disponibles haciendo avanzar el valor `watermark` hasta que no se devuelvan mensajes. | 

### <a name="send-a-message"></a>Enviar un mensaje
Envía un mensaje al bot. 
```http 
POST /api/conversations/{conversationId}/messages
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Un objeto [Message](#message-object) |
| **Devuelve** | No se devuelven datos en el cuerpo de la respuesta. El servicio responde con un código de estado HTTP 204, si el mensaje se envió correctamente. El cliente puede obtener su mensaje enviado (junto con los mensajes que el bot ha enviado al cliente) mediante la operación [Obtener mensajes](#get-messages). |

### <a id="upload-send-files"></a> Cargar y enviar archivos
Carga y envía archivos como adjuntos. Establezca el parámetro `userId` en el URI de la solicitud para especificar el identificador del usuario que envía los datos adjuntos.
```http 
POST /api/conversations/{conversationId}/upload?userId={userId}
```

| | |
|----|----|
| **Cuerpo de la solicitud** | Para un único dato adjunto, rellene el cuerpo de la solicitud con el contenido del archivo. Para varios datos adjuntos, cree un cuerpo de solicitud de varias partes que contenga una parte para cada dato adjunto y, además (de manera opcional), una parte para el objeto [Message](#message-object) que debe actuar como contenedor para los datos adjuntos especificados. Para más información, consulte [Envío de una actividad al bot](bot-framework-rest-direct-line-1-1-send-message.md). |
| **Devuelve** | No se devuelven datos en el cuerpo de la respuesta. El servicio responde con un código de estado HTTP 204, si el mensaje se envió correctamente. El cliente puede obtener su mensaje enviado (junto con los mensajes que el bot ha enviado al cliente) mediante la operación [Obtener mensajes](#get-messages). | 

> [!NOTE]
> Los archivos cargados se eliminan después de 24 horas.

## <a name="schema"></a>Esquema

El esquema de Direct Line 1.1 es una copia simplificada del esquema de Bot Framework v1 que incluye los siguientes objetos.

### <a name="message-object"></a>Objeto de mensaje

Define un mensaje que un cliente envía a un bot o recibe de un bot.

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **id** | string | Identificador que identifica de forma única el mensaje (asignado por Direct Line). | 
| **conversationId** | string | Identificador que identifica la conversación.  | 
| **created** | string | Fecha y hora en que se creó el mensaje, expresado en el formato <a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a>. | 
| **from** | string | Identificador que identifica el usuario que es el remitente del mensaje. Al crear un mensaje, los clientes deben establecer esta propiedad en un identificador de usuario estable. Aunque Direct Line asignará un identificador de usuario si no se proporciona ninguno, esto da lugar normalmente a un comportamiento inesperado. | 
| **text** | string | Texto del mensaje que se envía del usuario al bot o del bot al usuario. | 
| **channelData** | objeto | Objeto que incluye contenido específico de canal. Algunos canales proporcionan características que requieren información adicional que no se puede representar mediante el esquema de datos adjuntos. Para esos casos, establezca esta propiedad en el contenido específico de canal tal como se define en la documentación del canal. Estos datos se envían sin modificar entre el cliente y el bot. Esta propiedad se debe establecer en un objeto complejo o dejarse vacía. No la establezca en una cadena, número u otro tipo simple. | 
| **images** | string[] | Matriz de cadenas que contiene las direcciones URL para las imágenes que contiene el mensaje. En algunos casos, las cadenas de esta matriz pueden ser direcciones URL relativas. Si cualquier cadena de esta matriz no comienza con "http" o "https", anteponga `https://directline.botframework.com` a la cadena para formar la dirección URL completa. | 
| **attachments** | [Attachment](#attachment-object)[] | Matriz de objetos **Attachment** que representan los datos adjuntos que no son imágenes que contiene el mensaje. Cada objeto de la matriz contiene una propiedad `url` y una propiedad `contentType`. En los mensajes que recibe un cliente de un bot, la propiedad `url` puede especificar a veces una dirección URL relativa. Para cualquier valor de propiedad `url` que no comience por "http" o "https", anteponga `https://directline.botframework.com` a la cadena para formar la dirección URL completa. | 

En el ejemplo siguiente se muestra un objeto Message que contiene todas las propiedades posibles. En la mayoría de los casos al crear un mensaje, el cliente solo tiene que proporcionar la propiedad `from` y al menos una propiedad de contenido (por ejemplo, `text`, `images`, `attachments` o `channelData`).

```json
{
    "id": "CuvLPID4kDb|000000000000000004",
    "conversationId": "CuvLPID4kDb",
    "created": "2016-10-28T21:19:51.0357965Z",
    "from": "examplebot",
    "text": "Hello!",
    "channelData": {
        "examplefield": "abc123"
    },
    "images": [
        "/attachments/CuvLPID4kDb/0.jpg?..."
    ],
    "attachments": [
        {
            "url": "https://example.com/example.docx",
            "contentType": "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
        }, 
        {
            "url": "https://example.com/example.doc",
            "contentType": "application/msword"
        }
    ]
}
```

### <a name="messageset-object"></a>Objeto MessageSet 
Define un conjunto de mensajes.<br/><br/>

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **messages** | [Message](#message-object)[] | Matriz de objetos **Message**. |
| **watermark** | string | Marca de agua máxima de mensajes dentro del conjunto. Un cliente puede usar el valor `watermark` para indicar el mensaje más reciente que ha visto al [recuperar mensajes del bot](bot-framework-rest-direct-line-1-1-receive-messages.md). |

### <a name="attachment-object"></a>Objeto Attachment
Define un dato adjunto que no es una imagen.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **contentType** | string | Tipo de elemento multimedia del contenido de los datos adjuntos. |
| **url** | string | Dirección URL del contenido de los datos adjuntos. |

### <a name="conversation-object"></a>Objeto Conversation
Define una conversación de Direct Line.<br/><br/>

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **conversationId** | string | Identificador que identifica de forma única la conversación para la cual el token especificado es válido. |
| **token** | string | Token válido para la conversación especificada. |
| **expires_in** | número | Número de segundos hasta que expira el token. |

### <a name="error-object"></a>Objeto Error
Define un error.<br/><br/> 

| Propiedad | Escriba | DESCRIPCIÓN |
|----|----|----|
| **code** | string | Código de error. Uno de estos valores: **MissingProperty**, **MalformedData**, **NotFound**, **ServiceError**, **Internal**, **InvalidRange**, **NotSupported**, **NotAllowed**, **BadCertificate**. |
| **message** | string | Descripción del error. |
| **statusCode** | número | Código de estado. |

### <a name="errormessage-object"></a>Objeto ErrorMessage
Una carga estandarizada de errores de mensajes.<br/><br/> 


|        Propiedad        |          Escriba          |                                 DESCRIPCIÓN                                 |
|------------------------|------------------------|-----------------------------------------------------------------------------|
| <strong>error</strong> | [Error](#error-object) | Un objeto <strong>Error</strong> que contiene información sobre el error. |


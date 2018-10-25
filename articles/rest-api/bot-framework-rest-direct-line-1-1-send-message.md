---
title: Envío de un mensaje al bot | Microsoft Docs
description: Obtenga información sobre cómo enviar un mensaje al bot mediante Direct Line API v1.1.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 360ec3a6a6c9a3be16370aaf445f24a237a702e3
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998022"
---
# <a name="send-a-message-to-the-bot"></a>Envío de un mensaje al bot

> [!IMPORTANT]
> En este artículo se describe cómo enviar un mensaje al bot mediante Direct Line API v1.1. Si va a crear una conexión entre la aplicación cliente y el bot, use [Direct Line API 3.0](bot-framework-rest-direct-line-3-0-send-activity.md) en su lugar.

Mediante el protocolo de Direct Line 1.1, los clientes pueden intercambiar mensajes con los bots. Estos mensajes se convierten en el esquema que es compatible con el bot (Bot Framework v1 o Bot Framework v3). Un cliente puede enviar un único mensaje por solicitud. 

## <a name="send-a-message"></a>Envío de un mensaje

Para enviar un mensaje al bot, el cliente debe crear un objeto [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) para definir el mensaje y, a continuación, emitir una solicitud `POST` a `https://directline.botframework.com/api/conversations/{conversationId}/messages`, especificando el objeto Mensaje en el cuerpo de la solicitud.

Los fragmentos de código siguientes proporcionan un ejemplo de la solicitud de envío del mensaje y la respuesta.

### <a name="request"></a>Solicitud

```http
POST https://directline.botframework.com/api/conversations/abc123/messages
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
[other headers]
```

```json
{
  "text": "hello",
  "from": "user1"
}
```

### <a name="response"></a>Response

Cuando se entrega el mensaje al bot, el servicio responde con un código de estado HTTP que refleja el código de estado del bot. Si el bot genera un error, se devuelve una respuesta HTTP 500 ("Error interno del servidor") al cliente en respuesta a su solicitud de envío de mensaje. Si la PUBLICACIÓN se realiza correctamente, el servicio devuelve un código de estado HTTP 204. En el cuerpo de la respuesta no se devuelve ningún dato. El mensaje del cliente y los mensajes enviados por el bot pueden obtenerse mediante [sondeo](bot-framework-rest-direct-line-1-1-receive-messages.md). 

```http
HTTP/1.1 204 No Content
[other headers]
```

### <a name="total-time-for-the-send-message-requestresponse"></a>Tiempo total para la solicitud de envío de mensaje/respuesta

El tiempo total para PUBLICAR un mensaje en una conversación de Direct Line es la suma de lo siguiente:

- Tiempo de tránsito de la solicitud HTTP para desplazarse desde el cliente al servicio Direct Line
- Tiempo de procesamiento interno en Direct Line (normalmente menos de 120 ms)
- Tiempo de tránsito desde el servicio Direct Line al bot
- Tiempo de procesamiento en el bot
- Tiempo de tránsito de la respuesta HTTP hasta el cliente

## <a name="send-attachments-to-the-bot"></a>Envío de datos adjuntos al bot

En algunas situaciones, un cliente debe enviar datos adjuntos al bot, como imágenes o documentos. Un cliente puede enviar datos adjuntos al bot ya sea [especificando las direcciones URL](#send-by-url) de los datos adjuntos en el objeto [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) que envía mediante `POST /api/conversations/{conversationId}/messages` o [cargando datos adjuntos](#upload-attachments) mediante `POST /api/conversations/{conversationId}/upload`.

## <a id="send-by-url"></a> Envío de datos adjuntos por dirección URL

Para enviar uno o varios datos adjuntos como pate del objeto [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) con el uso de `POST /api/conversations/{conversationId}/messages`, especifique las direcciones URL de los datos adjuntos en las matrices `images` o `attachments` del mensaje.

## <a id="upload-attachments"></a> Envío de datos adjuntos mediante carga

A menudo, un cliente puede tener imágenes o documentos en un dispositivo que desea enviar al bot, pero no hay direcciones URL correspondientes a esos archivos. En esta situación, un cliente puede emitir una solicitud `POST /api/conversations/{conversationId}/upload` para enviar datos adjuntos al bot mediante carga. El formato y el contenido de la solicitud dependerán de si el cliente [envía un único dato adjunto](#upload-one-attachment) o [envía varios datos adjuntos](#upload-multiple-attachments).

### <a id="upload-one-attachment"></a> Envío de un único dato adjunto mediante carga

Para enviar un único dato adjunto mediante carga, emita esta solicitud: 

```http
POST https://directline.botframework.com/api/conversations/{conversationId}/upload?userId={userId}
Authorization: Bearer SECRET_OR_TOKEN
Content-Type: TYPE_OF_ATTACHMENT
Content-Disposition: ATTACHMENT_INFO
[other headers]

[file content]
```

En este URI de solicitud, reemplace **{conversationId}** con el identificador de la conversación y **{userId}** con el identificador del usuario que envía el mensaje. En los encabezados de solicitud, establezca `Content-Type` para especificar el tipo de datos adjuntos y defina `Content-Disposition` para especificar el nombre de archivo de los datos adjuntos.

Los fragmentos de código siguientes proporcionan un ejemplo de la solicitud de envío de un único dato adjunto y la respuesta.

#### <a name="request"></a>Solicitud

```http
POST https://directline.botframework.com/api/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: image/jpeg
Content-Disposition: name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]
```

#### <a name="response"></a>Response

Si la solicitud es correcta, se envía un mensaje al bot cuando se completa la carga y el servicio devuelve un código de estado HTTP 204.

```http
HTTP/1.1 204 No Content
[other headers]
```

### <a id="upload-multiple-attachments"></a> Envío de varios datos adjuntos mediante carga

Para enviar varios datos adjuntos mediante carga, emita una solicitud `POST` de varias partes al punto de conexión `/api/conversations/{conversationId}/upload`. Establezca el encabezado `Content-Type` de la solicitud en `multipart/form-data` e incluya el encabezado `Content-Type` y el encabezado `Content-Disposition` para que cada parte especifique el tipo y el nombre de archivo de los datos adjuntos. En el URI de solicitud, establezca el parámetro `userId` como el identificador del usuario que envía el mensaje. 

Puede incluir un objeto [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) en la solicitud con la adición de una parte que especifica como encabezado `Content-Type` el valor `application/vnd.microsoft.bot.message`. Esto permite al cliente personalizar el mensaje que contiene los datos adjuntos. Si la solicitud incluye un mensaje, los datos adjuntos especificados por otras partes de la carga se agregan como datos adjuntos a ese mensaje antes de enviarlo. 

Los fragmentos de código siguientes proporcionan un ejemplo de la solicitud de envío de varios datos adjuntos y la respuesta. En este ejemplo, la solicitud envía un mensaje que contiene texto y un único adjunto de imagen. Se pueden agregar partes adicionales a la solicitud para incluir varios datos adjuntos en este mensaje.

#### <a name="request"></a>Solicitud

```http
POST https://directline.botframework.com/api/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: multipart/form-data; boundary=----DD4E5147-E865-4652-B662-F223701A8A89
[other headers]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: image/jpeg
Content-Disposition: form-data; name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: application/vnd.microsoft.bot.message
[other headers]

{
  "text": "Hey I just IM'd you\n\nand this is crazy\n\nbut here's my webhook\n\nso POST me maybe",
  "from": "user1"
}

----DD4E5147-E865-4652-B662-F223701A8A89
```

#### <a name="response"></a>Response

Si la solicitud es correcta, se envía un mensaje al bot cuando se completa la carga y el servicio devuelve un código de estado HTTP 204.

```http
HTTP/1.1 204 No Content
[other headers]
```

## <a name="additional-resources"></a>Recursos adicionales

- [Conceptos clave](bot-framework-rest-direct-line-1-1-concepts.md)
- [Autenticación](bot-framework-rest-direct-line-1-1-authentication.md)
- [Inicio de una conversación](bot-framework-rest-direct-line-1-1-start-conversation.md)
- [Recepción de mensajes del bot](bot-framework-rest-direct-line-1-1-receive-messages.md)
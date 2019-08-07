---
title: Envío de una actividad al bot | Microsoft Docs
description: Obtenga información sobre cómo enviar una actividad al bot con Direct Line API v3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: a9a98419a8ac65c7e1093e1281e03917fa4eca11
ms.sourcegitcommit: f3fda6791f48ab178721b72d4f4a77c373573e38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2019
ms.locfileid: "68671501"
---
# <a name="send-an-activity-to-the-bot"></a>Envío de una actividad al bot

Mediante el protocolo Direct Line 3.0, los clientes y los bots pueden intercambiar varios tipos diferentes de [actividades](bot-framework-rest-connector-activities.md), incluidas las actividades de **mensaje**, las de **escritura** y las actividades personalizadas que admita el bot. Un cliente puede enviar una sola actividad por solicitud. 

## <a name="send-an-activity"></a>Envío de una actividad

Para enviar una actividad al bot, el cliente debe crear un objeto [Activity](bot-framework-rest-connector-api-reference.md#activity-object) para definir la actividad y, a continuación, emitir una solicitud `POST` a `https://directline.botframework.com/v3/directline/conversations/{conversationId}/activities`, especificando el objeto Activity en el cuerpo de la solicitud.

Los fragmentos de código siguientes proporcionan un ejemplo de la solicitud de envío de la actividad y la respuesta.

### <a name="request"></a>Solicitud

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: application/json
[other headers]
```

```json
{
    "type": "message",
    "from": {
        "id": "user1"
    },
    "text": "hello"
}
```

### <a name="response"></a>Response

Cuando se entrega la actividad al bot, el servicio responde con un código de estado HTTP que refleja el código de estado del bot. Si el bot genera un error, se devuelve una respuesta HTTP 502 ("Puerta de enlace incorrecta") al cliente en respuesta a su solicitud de envío de actividad.

> [!NOTE]
> Esto puede deberse al hecho de que no se ha usado un token correcto. Para enviar una actividad solo se puede usar el token que se recibió al *iniciar la conversación*.

Si la solicitud POST se realiza correctamente, la respuesta contiene una carga JSON que especifica el identificador de la actividad que se envió al bot.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "id": "0001"
}
```

### <a name="total-time-for-the-send-activity-requestresponse"></a>Tiempo total para la solicitud y respuesta del envío de la actividad

El tiempo total para el envío POST de un mensaje en una conversación de Direct Line es la suma de lo siguiente:

- Tiempo de tránsito de la solicitud HTTP para desplazarse desde el cliente al servicio Direct Line
- Tiempo de procesamiento interno en Direct Line (normalmente menos de 120 ms)
- Tiempo de tránsito desde el servicio Direct Line al bot
- Tiempo de procesamiento en el bot
- Tiempo de tránsito de la respuesta HTTP hasta el cliente

## <a name="send-attachments-to-the-bot"></a>Envío de datos adjuntos al bot

En algunas situaciones, un cliente debe enviar datos adjuntos al bot, como imágenes o documentos. Un cliente puede enviar datos adjuntos al bot ya sea [especificando las direcciones URL](#send-by-url) de los datos adjuntos en el objeto [Activity](bot-framework-rest-connector-api-reference.md#activity-object) que envía mediante `POST /v3/directline/conversations/{conversationId}/activities` o [cargando los datos adjuntos](#upload-attachments) mediante `POST /v3/directline/conversations/{conversationId}/upload`.

## <a id="send-by-url"></a> Envío de datos adjuntos por dirección URL

Para enviar uno o varios archivos adjuntos como parte del objeto [Activity](bot-framework-rest-connector-api-reference.md#activity-object) mediante `POST /v3/directline/conversations/{conversationId}/activities`, basta con incluir uno o varios objetos [Attachment](bot-framework-rest-connector-api-reference.md#attachment-object) dentro del objeto Activity y establecer la propiedad `contentUrl` de cada objeto Attachment para especificar la dirección HTTP o HTTPS o el identificador URI `data` de los datos adjuntos.

## <a id="upload-attachments"></a> Envío de datos adjuntos mediante carga

A menudo, un cliente puede tener imágenes o documentos en un dispositivo que desea enviar al bot, pero no hay direcciones URL correspondientes a esos archivos. En esta situación, un cliente puede emitir una solicitud `POST /v3/directline/conversations/{conversationId}/upload` para enviar datos adjuntos al bot mediante carga. El formato y el contenido de la solicitud dependerán de si el cliente [envía un único dato adjunto](#upload-one-attachment) o [envía varios datos adjuntos](#upload-multiple-attachments).

### <a id="upload-one-attachment"></a> Envío de un único dato adjunto mediante carga

Para enviar un único dato adjunto mediante carga, emita esta solicitud: 

```http
POST https://directline.botframework.com/v3/directline/conversations/{conversationId}/upload?userId={userId}
Authorization: Bearer SECRET_OR_TOKEN
Content-Type: TYPE_OF_ATTACHMENT
Content-Disposition: ATTACHMENT_INFO
[other headers]

[file content]
```

En este identificador URI de solicitud, reemplace **{conversationId}** por el identificador de la conversación y **{userId}** por el identificador del usuario que envía el mensaje. El parámetro `userId` es obligatorio. En los encabezados de solicitud, establezca `Content-Type` para especificar el tipo de datos adjuntos y establezca `Content-Disposition` para especificar el nombre de archivo de los datos adjuntos.

Los fragmentos de código siguientes proporcionan un ejemplo de la solicitud de envío de un único dato adjunto y la respuesta.

#### <a name="request"></a>Solicitud

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: image/jpeg
Content-Disposition: name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]
```

#### <a name="response"></a>Response

Si la solicitud es correcta, se envía un **mensaje** Activity al bot cuando finaliza la carga y la respuesta que recibe el cliente incluirá el identificador de la actividad que se envió.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "id": "0003"
}
```

### <a id="upload-multiple-attachments"></a> Envío de varios datos adjuntos mediante carga

Para enviar varios datos adjuntos mediante carga, emita una solicitud `POST` de varias partes al punto de conexión `/v3/directline/conversations/{conversationId}/upload`. Establezca el encabezado `Content-Type` de la solicitud en `multipart/form-data` e incluya el encabezado `Content-Type` y el encabezado `Content-Disposition` para que cada parte especifique el tipo y el nombre de archivo de los datos adjuntos. En el identificador URI de la solicitud, establezca el parámetro `userId` en el identificador del usuario que envía el mensaje. 

Puede incluir un objeto [Activity](bot-framework-rest-connector-api-reference.md#activity-object) en la solicitud con la adición de una parte que especifica como encabezado `Content-Type` el valor `application/vnd.microsoft.activity`. Si la solicitud incluye una actividad, los datos adjuntos especificados por otras partes de la carga se agregan como datos adjuntos a esa actividad antes de enviarla. Si la solicitud no incluye una actividad, se crea una actividad vacía para servir como el contenedor en el que se envían los datos adjuntos especificados.

Los fragmentos de código siguientes proporcionan un ejemplo de la solicitud de envío de varios datos adjuntos y la respuesta. En este ejemplo, la solicitud envía un mensaje que contiene texto y un único adjunto de imagen. Se pueden agregar partes adicionales a la solicitud para incluir varios datos adjuntos en este mensaje.

#### <a name="request"></a>Solicitud

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: multipart/form-data; boundary=----DD4E5147-E865-4652-B662-F223701A8A89
[other headers]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: image/jpeg
Content-Disposition: form-data; name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: application/vnd.microsoft.activity
[other headers]

{
  "type": "message",
  "from": {
    "id": "user1"
  },
  "text": "Hey I just IM'd you\n\nand this is crazy\n\nbut here's my webhook\n\nso POST me maybe"
}

----DD4E5147-E865-4652-B662-F223701A8A89
```

#### <a name="response"></a>Response

Si la solicitud es correcta, se envía un mensaje Activity al bot cuando finaliza la carga y la respuesta que recibe el cliente incluirá el identificador de la actividad que se envió.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "id": "0004"
}
```

## <a name="additional-resources"></a>Recursos adicionales

- [Conceptos clave](bot-framework-rest-direct-line-3-0-concepts.md)
- [Autenticación](bot-framework-rest-direct-line-3-0-authentication.md)
- [Inicio de una conversación](bot-framework-rest-direct-line-3-0-start-conversation.md)
- [Volver a conectarse a una conversación](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)
- [Recepción de actividades del bot](bot-framework-rest-direct-line-3-0-receive-activities.md)
- [Finalización de una conversación](bot-framework-rest-direct-line-3-0-end-conversation.md)

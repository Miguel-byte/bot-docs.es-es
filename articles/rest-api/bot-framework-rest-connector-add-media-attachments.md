---
title: Incorporación de datos adjuntos de elementos multimedia a mensajes | Microsoft Docs
description: Aprenda a agregar datos adjuntos de elementos multimedia a mensajes mediante el servicio Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/25/2018
ms.openlocfilehash: eb08ca3b560f5058526387c71eca4b790d033a1e
ms.sourcegitcommit: 15f7fa40b7e0a05507cdc66adf75bcfc9533e781
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916812"
---
# <a name="add-media-attachments-to-messages"></a>Incorporación de datos adjuntos con elementos multimedia a mensajes
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

Los bots y los canales suelen intercambiar cadenas de texto, pero algunos canales también admiten el intercambio de datos adjuntos, lo que permite a un bot enviar mensajes más completos a los usuarios. Por ejemplo, el bot puede enviar datos adjuntos de elementos multimedia (como imágenes, vídeos, audio, archivos) y [tarjetas enriquecidas](bot-framework-rest-connector-add-rich-cards.md). En este artículo se describe cómo agregar datos adjuntos de elementos multimedia a mensajes mediante el servicio Bot Connector.

> [!TIP]
> Para determinar el tipo y el número de datos adjuntos que admite un canal, y cómo el canal representa los datos adjuntos, consulte [Channel Inspector][ChannelInspector].

## <a name="add-a-media-attachment"></a>Incorporación de datos adjuntos con elementos multimedia  

Para agregar datos adjuntos de elementos multimedia a un mensaje, cree un objeto [Attachment][Attachment], establezca la propiedad `name`, establezca la propiedad `contentUrl` en la dirección URL del archivo multimedia y establezca la propiedad `contentType` en el tipo de elemento multimedia adecuado (por ejemplo, **imagen/png**, **audio/wav**, **vídeo/mp4**). A continuación, en el objeto [Activity][Activity] que representa el mensaje, especifique el objeto [Attachment][Attachment] dentro de la matriz `attachments`. 

En el ejemplo siguiente se muestra una solicitud que envía un mensaje que contiene texto y datos adjuntos con una sola imagen. En esta solicitud de ejemplo, `https://smba.trafficmanager.net/apis` representa el URI base; los URI base para las solicitudes que emite el bot pueden ser diferentes. Para más información sobre cómo establecer el URI base, consulte [Referencia de la API](bot-framework-rest-connector-api-reference.md#base-uri).

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/5d5cdc723
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "Here's a picture of the duck I was telling you about.",
    "attachments": [
        {
            "contentType": "image/png",
            "contentUrl": "http://aka.ms/Fo983c",
            "name": "duck-on-a-rock.jpg"
        }
    ],
    "replyToId": "5d5cdc723"
}
```

En el caso de canales que admiten datos binarios en línea de una imagen, puede establecer la propiedad `contentUrl` del objeto `Attachment` en un archivo binario de base64 de la imagen (por ejemplo, **datos:imagen/png;base64, iVBORw0KGgo...** ). El canal mostrará la imagen o la dirección URL de la imagen junto a la cadena de texto del mensaje.

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "Here's a picture of the duck I was telling you about.",
    "attachments": [
        {
            "contentType": "image/png",
            "contentUrl": "data:image/png;base64,iVBORw0KGgo…",
            "name": "duck-on-a-rock.jpg"
        }
    ],
    "replyToId": "5d5cdc723"
}
```

Puede adjuntar un archivo de vídeo o de audio a un mensaje siguiendo el mismo proceso descrito anteriormente para un archivo de imagen. Dependiendo del canal, el vídeo y el audio pueden reproducirse en línea o se pueden mostrar como un vínculo.

> [!NOTE] 
> Su bot también puede recibir mensajes que contengan datos adjuntos de elementos multimedia.
> Por ejemplo, un mensaje que recibe el bot puede contener datos adjuntos si el canal permite al usuario cargar una foto para analizar o un documento para almacenar.

## <a name="add-an-audiocard-attachment"></a>Incorporación de datos adjuntos de tarjeta de audio

Agregar datos adjuntos de [tarjeta de audio](bot-framework-rest-connector-api-reference.md#audiocard-object) o [tarjeta de vídeo](bot-framework-rest-connector-api-reference.md#videocard-object) es lo mismo que agregar datos adjuntos de elementos multimedia. Por ejemplo, el siguiente código JSON muestra cómo agregar una tarjeta de audio en los datos adjuntos de elementos multimedia.

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "attachments": [
    {
      "contentType": "application/vnd.microsoft.card.audio",
      "content": {
        "title": "Allegro in C Major",
        "subtitle": "Allegro Duet",
        "text": "No Image, No Buttons, Autoloop, Autostart, Sharable",
        "duration": "PT2M55S",
        "media": [
          {
            "url": "https://contoso.com/media/AllegrofromDuetinCMajor.mp3"
          }
        ],
        "shareable": true,
        "autoloop": true,
        "autostart": true,
        "value": {
            // Supplementary parameter for this card
        }
      }
    }],
    "replyToId": "5d5cdc723"
}
```

Una vez que el canal recibe estos datos adjuntos, comenzará a reproducirse el archivo de audio. Si un usuario interactúa con el audio y hace clic en el botón **Pausar**, por ejemplo, el canal enviará una devolución de llamada al bot con un código JSON similar al siguiente:

```json
{
    ...
    "type": "event",
    "name": "media/pause",
    "value": {
        "url": // URL for media
        "cardValue": {
            // Supplementary parameter for this card
        }
    }
}
```

El nombre del evento multimedia **media/pause** aparecerá en el campo `activity.name`. Tome como referencia la tabla siguiente para obtener una lista de todos los nombres de eventos multimedia.

| Evento | DESCRIPCIÓN |
| ---- | ---- |
| **media/next** | El cliente saltó al siguiente elemento multimedia |
| **media/pause** | El cliente pausó la reproducción del elemento multimedia |
| **media/play** | El cliente inició la reproducción del elemento multimedia |
| **media/previous** | El cliente saltó al elemento multimedia anterior |
| **media/resume** | El cliente reanudó la reproducción del elemento multimedia |
| **media/stop** | El cliente detuvo la reproducción del elemento multimedia |

## <a name="additional-resources"></a>Recursos adicionales

- [Creación de mensajes](bot-framework-rest-connector-create-messages.md)
- [Envío y recepción de mensajes](bot-framework-rest-connector-send-and-receive-messages.md)
- [Incorporación de tarjetas enriquecidas a mensajes](bot-framework-rest-connector-add-rich-cards.md)
- [Esquema de tarjeta de Bot Framework](https://aka.ms/botSpecs-cardSchema)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[Attachment]: bot-framework-rest-connector-api-reference.md#attachment-object

---
title: Incorporación de datos adjuntos de tarjetas enriquecidas a mensajes | Microsoft Docs
description: Aprenda a agregar tarjetas enriquecidas a los mensajes mediante el servicio Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 10246fda94932feb96e5faa0cdd8ca489c98c855
ms.sourcegitcommit: c200cc2db62dbb46c2a089fb76017cc55bdf26b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2019
ms.locfileid: "70037476"
---
# <a name="add-rich-card-attachments-to-messages"></a>Incorporación de datos adjuntos de tarjetas enriquecidas a mensajes
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

Los bots y los canales suelen intercambiar cadenas de texto, pero algunos canales también admiten el intercambio de datos adjuntos, lo que permite a un bot enviar mensajes más completos a los usuarios. Por ejemplo, el bot puede enviar datos adjuntos de elementos multimedia y tarjetas enriquecidas (como imágenes, vídeos, audio, archivos). En este artículo se describe cómo agregar datos adjuntos de tarjetas enriquecidas a mensajes mediante el servicio Bot Connector.

> [!NOTE]
> Para información sobre cómo agregar datos adjuntos con elementos multimedia a los mensajes, consulte [Incorporación de datos adjuntos con elementos multimedia a mensajes](bot-framework-rest-connector-add-media-attachments.md).

## <a id="types-of-cards"></a> Tipos de tarjetas enriquecidas

Una tarjeta enriquecida consta de un título, una descripción, un vínculo e imágenes. Un mensaje puede contener varias tarjetas enriquecidas, que se muestran en formato de lista o de carrusel.
Bot Framework admite actualmente ocho tipos de tarjetas enriquecidas: 

| Tipo de tarjeta | DESCRIPCIÓN |
|----|----|
| <a href="/adaptive-cards/get-started/bots">AdaptiveCard</a> | Una tarjeta personalizable que puede contener cualquier combinación de texto, voz, imágenes, botones y campos de entrada. Consulte la [compatibilidad por canal](/adaptive-cards/get-started/bots#channel-status).  |
| [AnimationCard][] | Una tarjeta que puede reproducir archivos GIF animados o vídeos cortos. |
| [AudioCard][] | Una tarjeta que se puede reproducir un archivo de audio. |
| [HeroCard][] | Una tarjeta que normalmente contiene una sola imagen grande, uno o varios botones y texto. |
| [ThumbnailCard][] | Una tarjeta que normalmente contiene una sola imagen miniatura, uno o varios botones, y texto. |
| [ReceiptCard][] | Una tarjeta que permite que un bot proporcione un recibo al usuario. Normalmente, contiene la lista de elementos que se incluyen en el recibo, la información de impuestos y del total, y texto adicional. |
| [SignInCard][] | Una tarjeta que permite al bot solicitar que un usuario inicie sesión. Normalmente contiene texto y uno o varios botones en los que el usuario puede hacer clic para iniciar el proceso de inicio de sesión. |
| [VideoCard][] | Una tarjeta que puede reproducir vídeos. |

> [!TIP]
> Para determinar el tipo de tarjetas enriquecidas que admite un canal y ver cómo el canal presenta las tarjeta enriquecidas, consulte el [Inspector de canales][ChannelInspector]. Consulte la documentación del canal para obtener información sobre las limitaciones del contenido de las tarjetas (por ejemplo, el número máximo de botones o la longitud máxima del título).

## <a name="process-events-within-rich-cards"></a>Procesamiento de eventos dentro de tarjetas enriquecidas

Para procesar eventos dentro de tarjetas enriquecidas, use los objetos [CardAction][] para especificar qué debe ocurrir cuando el usuario hace clic en un botón o pulsa una sección de la tarjeta. Cada objeto `CardAction` contiene estas propiedades:

| Propiedad | Escriba | DESCRIPCIÓN | 
|----|----|----|
| Tipo | string | tipo de acción (uno de los valores especificados en la tabla siguiente) |
| título | string | título del botón |
| imagen | string | dirección URL de la imagen del botón |
| value | string | valor necesario para realizar el tipo de acción especificado |

> [!NOTE]
> Los botones dentro de las tarjetas adaptables no se crean mediante el uso de objetos `CardAction`, sino que mediante el esquema que definen las <a href="http://adaptivecards.io" target="_blank">tarjetas adaptables</a>. Consulte [Incorporación de una tarjeta adaptable a un mensaje](#adaptive-card) para ver un ejemplo que muestra cómo agregar botones a una tarjeta adaptable.

En esta tabla se enumeran los valores válidos para la propiedad `type` de un objeto `CardAction` y se describe el contenido esperado de la propiedad `value` de cada tipo:

| Tipo | value | 
|----|----|
| openUrl | Dirección URL que se abrirá en el explorador integrado |
| imBack | Texto del mensaje para enviar al bot (desde el usuario que hizo clic en el botón o que pulsó la tarjeta). Este mensaje (del usuario al bot) será visible para todos los participantes de la conversación mediante la aplicación cliente que hospeda la conversación. |
| postBack | Texto del mensaje para enviar al bot (desde el usuario que hizo clic en el botón o que pulsó la tarjeta). Algunas aplicaciones cliente pueden mostrar este texto en la fuente del mensaje, donde estará visible para todos los participantes de la conversación. |
| llamada | Destino de una llamada telefónica con este formato: **tel:123123123123** |
| playAudio | Dirección URL del audio que se va a reproducir |
| playVideo | Dirección URL del vídeo que se va a reproducir |
| showImage | Dirección URL de la imagen que se va a mostrar |
| DownloadFile | Dirección URL del archivo que se va a descargar |
| signin | Dirección URL del flujo de OAuth que se va a iniciar |

## <a name="add-a-hero-card-to-a-message"></a>Incorporación de una tarjeta de imagen prominente a un mensaje

Para agregar un archivo adjunto de tarjeta enriquecido a un mensaje, cree primero un objeto que corresponda al [tipo de tarjeta](#types-of-cards) que desee agregar al mensaje. A continuación, cree un objeto [Datos adjuntos][], establezca su propiedad `contentType` en el tipo de elemento multimedia de la tarjeta y su propiedad `content` como el objeto que ha creado para representar la tarjeta. Especifique su objeto `Attachment` dentro de la matriz `attachments` del mensaje.

> [!TIP]
> Los mensajes que contienen datos adjuntos de tarjeta enriquecida normalmente no especifican `text`.

Algunos canales permiten agregar varias tarjetas enriquecidas a la matriz `attachments` dentro de un mensaje. Esta funcionalidad puede ser útil en escenarios donde desee proporcionar al usuario varias opciones. Por ejemplo, si el bot permite a los usuarios reservar habitaciones de hotel, podría presentar al usuario una lista de tarjetas enriquecidas que muestre los tipos de habitaciones disponibles. Cada tarjeta podría contener una imagen y una lista de servicios correspondientes al tipo de habitación y el usuario podría seleccionar una punteando en una tarjeta o haciendo clic en un botón.

> [!TIP]
> Para mostrar varias tarjetas enriquecidas en formato de lista, establezca la propiedad `attachmentLayout` del objeto [Actividad][] en "list" (lista). Para mostrar varias tarjetas enriquecidas en formato de carrusel, establezca la propiedad `attachmentLayout` del objeto `Activity` en "carousel" (carrusel). Si el canal no admite el formato de carrusel, mostrará las tarjetas enriquecidas en el formato de lista, incluso si la propiedad `attachmentLayout` especifica "carousel" (carrusel).

En el ejemplo siguiente se muestra una solicitud que envía un mensaje que contiene datos adjuntos con una sola tarjeta de imagen prominente. En esta solicitud de ejemplo, `https://smba.trafficmanager.net/apis` representa el URI base; los URI base para las solicitudes que emite el bot pueden ser diferentes. Para más información sobre cómo establecer el URI base, consulte [Referencia de la API](bot-framework-rest-connector-api-reference.md#base-uri).

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
    "attachments": [
        {
            "contentType": "application/vnd.microsoft.card.hero",
            "content": {
                "title": "title goes here",
                "subtitle": "subtitle goes here",
                "text": "descriptive text goes here",
                "images": [
                    {
                        "url": "https://aka.ms/DuckOnARock",
                        "alt": "picture of a duck",
                        "tap": {
                            "type": "playAudio",
                            "value": "url to an audio track of a duck call goes here"
                        }
                    }
                ],
                "buttons": [
                    {
                        "type": "playAudio",
                        "title": "Duck Call",
                        "value": "url to an audio track of a duck call goes here"
                    },
                    {
                        "type": "openUrl",
                        "title": "Watch Video",
                        "image": "https://aka.ms/DuckOnARock",
                        "value": "url goes here of the duck in flight"
                    }
                ]
            }
        }
    ],
    "replyToId": "5d5cdc723"
}
```

## <a id="adaptive-card"></a> Incorporación de una tarjeta adaptable a un mensaje

Una tarjeta adaptable puede contener cualquier combinación de texto, voz, imágenes, botones y campos de entrada. Las tarjetas adaptables se crean con el formato JSON especificado en <a href="http://adaptivecards.io" target="_blank">Adaptive Cards</a> (Tarjetas adaptables), que proporciona control total sobre el contenido y el formato de la tarjeta. 

Use la información del sitio <a href="http://adaptivecards.io" target="_blank">Adaptive Cards</a> (Tarjetas adaptables) a fin de comprender el esquema de la tarjeta adaptable, explorar los elementos de la tarjeta adaptable y ver ejemplos de JSON que puede usar para crear tarjetas de diferente composición y complejidad. Además, puede usar el visualizador interactivo para diseñar cargas útiles de la tarjeta adaptable y obtener una vista previa de la salida de la tarjeta.

En el ejemplo siguiente se muestra una solicitud que envía un mensaje que contiene una tarjeta adaptable para un recordatorio de calendario. En esta solicitud de ejemplo, `https://smba.trafficmanager.net/apis` representa el URI base; los URI base para las solicitudes que emite el bot pueden ser diferentes. Para más información sobre cómo establecer el URI base, consulte [Referencia de la API](bot-framework-rest-connector-api-reference.md#base-uri).

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
    "attachments": [
        {
            "contentType": "application/vnd.microsoft.card.adaptive",
            "content": {
                "type": "AdaptiveCard",
                "body": [
                    {
                        "type": "TextBlock",
                        "text": "Adaptive Card design session",
                        "size": "large",
                        "weight": "bolder"
                    },
                    {
                        "type": "TextBlock",
                        "text": "Conf Room 112/3377 (10)"
                    },
                    {
                        "type": "TextBlock",
                        "text": "12:30 PM - 1:30 PM"
                    },
                    {
                        "type": "TextBlock",
                        "text": "Snooze for"
                    },
                    {
                        "type": "Input.ChoiceSet",
                        "id": "snooze",
                        "style": "compact",
                        "choices": [
                            {
                                "title": "5 minutes",
                                "value": "5",
                                "isSelected": true
                            },
                            {
                                "title": "15 minutes",
                                "value": "15"
                            },
                            {
                                "title": "30 minutes",
                                "value": "30"
                            }
                        ]
                    }
                ],
                "actions": [
                    {
                        "type": "Action.Http",
                        "method": "POST",
                        "url": "http://foo.com",
                        "title": "Snooze"
                    },
                    {
                        "type": "Action.Http",
                        "method": "POST",
                        "url": "http://foo.com",
                        "title": "I'll be late"
                    },
                    {
                        "type": "Action.Http",
                        "method": "POST",
                        "url": "http://foo.com",
                        "title": "Dismiss"
                    }
                ]
            }
        }
    ],
    "replyToId": "5d5cdc723"
}
```

La tarjeta resultante contiene tres bloques de texto, un campo de entrada (lista de opciones) y tres botones:

![Recordatorio del calendario de tarjeta adaptable](../media/adaptive-card-reminder.png)


## <a name="additional-resources"></a>Recursos adicionales

- [Creación de mensajes](bot-framework-rest-connector-create-messages.md)
- [Envío y recepción de mensajes](bot-framework-rest-connector-send-and-receive-messages.md)
- [Incorporación de datos adjuntos con elementos multimedia a mensajes](bot-framework-rest-connector-add-media-attachments.md)
- [Esquema Activity de Bot Framework](https://aka.ms/botSpecs-activitySchema)
- [Channel Inspector][ChannelInspector]
- <a href="http://adaptivecards.io" target="_blank">Tarjetas adaptables</a>

[ChannelInspector]: ../bot-service-channel-inspector.md

[Actividad]: bot-framework-rest-connector-api-reference.md#activity-object
[Datos adjuntos]: bot-framework-rest-connector-api-reference.md#attachment-object
[CardAction]: bot-framework-rest-connector-api-reference.md#cardaction-object

[AnnimationCard]: bot-framework-rest-connector-api-reference.md#annimationcard-object
[AudioCard]: bot-framework-rest-connector-api-reference.md#audiocard-object
[HeroCard]: bot-framework-rest-connector-api-reference.md#herocard-object
[ThumbnailCard]: bot-framework-rest-connector-api-reference.md#thumbnailcard-object
[ReceiptCard]: bot-framework-rest-connector-api-reference.md#receiptcard-object
[SigninCard]: bot-framework-rest-connector-api-reference.md#signincard-object
[VideoCard]: bot-framework-rest-connector-api-reference.md#videocard-object

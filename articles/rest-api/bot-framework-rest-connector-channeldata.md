---
title: Implementar una funcionalidad específica de canal | Microsoft Docs
description: Aprenda a implementar una funcionalidad específica de canal mediante Bot Connector API.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 95ec59da7b2b64391a599fa690bf3e8410c3cd53
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2019
ms.locfileid: "68757193"
---
# <a name="implement-channel-specific-functionality"></a>Implementar una funcionalidad específica de canal

Algunos canales proporcionan características que no se puede implementar mediante el uso de [texto y datos adjuntos del mensaje](bot-framework-rest-connector-create-messages.md) únicamente. Para implementar la funcionalidad específica del canal, puede pasar metadatos nativo a un canal en la propiedad `channelData` del objeto `Activity`. Por ejemplo, el bot puede usar la propiedad `channelData` para indicar a Telegram que envíe un adhesivo o para indicar a Office 365 que envíe un correo electrónico.

En este artículo se describe cómo usar la propiedad `channelData` de la actividad de un mensaje para implementar esta funcionalidad específica de canal:

| Canal | Funcionalidad |
|----|----|
| Email | Envío y recepción de un correo electrónico que contiene metadatos de importancia, cuerpo y asunto. |
| Slack | Envío de mensajes de Slack de plena fidelidad. |
| Facebook | Envío de notificaciones de Facebook de forma nativa. |
| Telegram | Acciones específicas de Telegram, como compartir una nota de voz o un adhesivo. |
| Kik | Envío y recepción mensajes nativos de Kik | 

> [!NOTE]
> El valor de la propiedad `channelData` de un objeto `Activity` es un objeto JSON. La estructura del objeto JSON variará según el canal y la funcionalidad que se implementa, tal como se describe a continuación. 

## <a name="create-a-custom-email-message"></a>Crear un mensaje de correo electrónico personalizado

Para crear un mensaje de correo electrónico, establezca la propiedad `channelData` del objeto `Activity` en un objeto JSON que contenga estas propiedades:

[!INCLUDE [Email channelData table](~/includes/snippet-channelData-email.md)]

En este fragmento de código se muestra un ejemplo de la propiedad `channelData` para un mensaje de correo electrónico personalizado.

```json
"channelData":
{
    "htmlBody": "<html><body style = /"font-family: Calibri; font-size: 11pt;/" >This is more than awesome.</body></html>",
    "subject": "Super awesome message subject",
    "importance": "high",
    "ccRecipients": "Yasemin@adatum.com;Temel@adventure-works.com"
}
```

## <a name="create-a-full-fidelity-slack-message"></a>Creación de un mensaje de Slack de plena fidelidad

Para crear un mensaje de Slack de plena fidelidad, establezca la propiedad `channelData` del objeto `Activity` en un objeto JSON que especifique <a href="https://api.slack.com/docs/messages" target="_blank">mensajes de Slack</a>, <a href="https://api.slack.com/docs/message-attachments" target="_blank">adjuntos de Slack</a> o <a href="https://api.slack.com/docs/message-buttons" target="_blank">botones de Slack</a>. 

> [!NOTE]
> Para admitir botones en los mensajes de Slack, debe habilitar **Mensajes interactivos** cuando [conecta el bot](../bot-service-manage-channels.md) al canal de Slack.

En este fragmento de código se muestra un ejemplo de la propiedad `channelData` para un mensaje de Slack personalizado.

```json
"channelData": {
   "text": "Now back in stock! :tada:",
   "attachments": [
        {
            "title": "The Further Adventures of Slackbot",
            "author_name": "Stanford S. Strickland",
            "author_icon": "https://api.slack.com/img/api/homepage_custom_integrations-2x.png",
            "image_url": "http://i.imgur.com/OJkaVOI.jpg?1"
        },
        {
            "fields": [
                {
                    "title": "Volume",
                    "value": "1",
                    "short": true
                },
                {
                    "title": "Issue",
                    "value": "3",
                    "short": true
                }
            ]
        },
        {
            "title": "Synopsis",
            "text": "After @episod pushed exciting changes to a devious new branch back in Issue 1, Slackbot notifies @don about an unexpected deploy..."
        },
        {
            "fallback": "Would you recommend it to customers?",
            "title": "Would you recommend it to customers?",
            "callback_id": "comic_1234_xyz",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "actions": [
                {
                    "name": "recommend",
                    "text": "Recommend",
                    "type": "button",
                    "value": "recommend"
                },
                {
                    "name": "no",
                    "text": "No",
                    "type": "button",
                    "value": "bad"
                }
            ]
        }
    ]
}
```

Cuando un usuario hace clic en un botón dentro de un mensaje de Slack, el bot recibirá un mensaje de respuesta en el que la propiedad `channelData` se rellena con un objeto JSON `payload`. El objeto `payload` especifica el contenido del mensaje original, identifica el botón donde se hizo clic e identifica al usuario que hizo clic en el botón. 

En este fragmento de código se muestra un ejemplo de la propiedad `channelData` en el mensaje que recibe un bot cuando un usuario hace clic en un botón en el mensaje de Slack.

```json
"channelData": {
    "payload": {
        "actions": [
            {
                "name": "recommend",
                "value": "yes"
            }
        ],
        . . .
        "original_message": "{…}",
        "response_url": "https://hooks.slack.com/actions/..."
    }
}
```

El bot puede responder a este mensaje de la [forma normal](bot-framework-rest-connector-send-and-receive-messages.md#create-reply) o puede registrar su respuesta directamente en punto de conexión que haya especificado la propiedad `response_url` del objeto `payload`. Para obtener información sobre cuándo y cómo publicar una respuesta en `response_url`, consulte <a href="https://api.slack.com/docs/message-buttons" target="_blank">Botones de Slack</a>. 

## <a name="create-a-facebook-notification"></a>Creación de una notificación de Facebook

Para crear una notificación de Facebook, establezca la propiedad `channelData` del objeto `Activity` en un objeto JSON que especifique estas propiedades: 

| Propiedad | DESCRIPCIÓN |
|----|----|
| notification_type | El tipo de notificación (por ejemplo, **REGULAR**, **SILENT_PUSH**, **NO_PUSH**).
| attachment | Un elemento adjunto que especifica una imagen, un vídeo u otro tipo de elemento multimedia, o un adjunto con plantilla como, por ejemplo, un recibo. |

> [!NOTE]
> Para obtener más información sobre el formato y el contenido de la propiedad `notification_type` y la propiedad `attachment`, consulte la <a href="https://developers.facebook.com/docs/messenger-platform/send-api-reference#guidelines" target="_blank">documentación de la API de Facebook</a>. 

En este fragmento de código se muestra un ejemplo de la propiedad `channelData` para un elemento adjunto con recibo de Facebook.

```json
"channelData": {
    "notification_type": "NO_PUSH",
    "attachment": {
        "type": "template"
        "payload": {
            "template_type": "receipt",
            . . .
        }
    }
}
```

## <a name="create-a-telegram-message"></a>Creación de un mensaje de Telegram

Para crear un mensaje que implemente las acciones específicas de Telegram, como compartir una nota de voz o un sticker, establezca la propiedad `channelData` del objeto `Activity` en un objeto JSON que especifique estas propiedades: 

| Propiedad | DESCRIPCIÓN |
|----|----|
| method | El método de Telegram Bot API al que se llamará. |
| parameters | Los parámetros del método especificado. |

Se admiten los métodos de Telegram siguientes: 

- answerInlineQuery
- editMessageCaption
- editMessageReplyMarkup
- editMessageText
- forwardMessage
- kickChatMember
- sendAudio
- sendChatAction
- sendContact
- sendDocument
- sendLocation
- sendMessage
- sendPhoto
- sendSticker
- sendVenue
- sendVideo
- sendVoice
- unbanChateMember

Para obtener más información sobre estos métodos de Telegram y sus parámetros, consulte la <a href="https://core.telegram.org/bots/api#available-methods" target="_blank">documentación de Telegram Bot API</a>.

> [!NOTE]
> <ul><li>El parámetro <code>chat_id</code> es común a todos los métodos de Telegram. Si no especifica <code>chat_id</code> como parámetro, el marco proporcionará el identificador automáticamente.</li>
> <li>En lugar de pasar el contenido del archivo insertado, especifique el archivo mediante una dirección URL y el tipo de medio, tal como se muestra en el ejemplo siguiente.</li>
> <li>Dentro de cada mensaje que recibe su bot del canal de Telegram, la propiedad <code>channelData</code> incluirá el mensaje que su bot envió anteriormente.</li></ul>

En este fragmento de código se muestra un ejemplo de una propiedad `channelData` que especifica un único método de Telegram.

```json
"channelData": {
    "method": "sendSticker",
    "parameters": {
        "sticker": {
            "url": "https://domain.com/path/gif",
            "mediaType": "image/gif",
        }
    }
}
```

En este fragmento de código se muestra un ejemplo de una propiedad `channelData` que especifica una matriz de métodos de Telegram.

```json
"channelData": [
    {
        "method": "sendSticker",
        "parameters": {
            "sticker": {
                "url": "https://domain.com/path/gif",
                "mediaType": "image/gif",
            }
        }
    },
    {
        "method": "sendMessage",
        "parameters": {
            "text": "<b>This message is HTML formatted.</b>",
            "parse_mode": "HTML"
        }
    }
]
```

## <a name="create-a-native-kik-message"></a>Creación de un mensaje de Kik nativo

Para crear un mensaje de Kik nativo, establezca la propiedad `channelData` del objeto `Activity` en un objeto JSON que especifique esta propiedad: 

| Propiedad | DESCRIPCIÓN |
|----|----|
| messages | Una matriz de mensajes de Kik. Para obtener más información sobre el formato de los mensajes de Kik, consulte los <a href="https://dev.kik.com/#/docs/messaging#message-formats" target="_blank">formatos de mensaje de Kik</a>. |

En este fragmento de código se muestra un ejemplo de la propiedad `channelData` para un mensaje de Kik nativo.

```json
"channelData": {
    "messages": [
        {
            "chatId": "c6dd8165…",
            "type": "link",
            "to": "kikhandle",
            "title": "My Webpage",
            "text": "Some text to display",
            "url": "http://botframework.com",
            "picUrl": "http://lorempixel.com/400/200/",
            "attribution": {
                "name": "My App",
                "iconUrl": "http://lorempixel.com/50/50/"
            },
            "noForward": true,
            "kikJsData": {
                    "key": "value"
                }
        }
    ]
}
```

## <a name="additional-resources"></a>Recursos adicionales

- [Activities overview](bot-framework-rest-connector-activities.md) (Introducción a las actividades)
- [Creación de mensajes](bot-framework-rest-connector-create-messages.md)
- [Envío y recepción de mensajes](bot-framework-rest-connector-send-and-receive-messages.md)
- [Esquema Activity de Bot Framework](https://aka.ms/botSpecs-activitySchema)
- [Preview features with the Channel Inspector](../bot-service-channel-inspector.md) (Vista previa de las características con el Inspector de canales)

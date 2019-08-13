---
title: Envío y recepción de mensajes | Microsoft Docs
description: Obtenga información sobre cómo enviar y recibir mensajes mediante el servicio Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 8ffb3d0572eafccc89bcc2cf6487aeef475bd6b1
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2019
ms.locfileid: "68757139"
---
# <a name="send-and-receive-messages"></a>Envío y recepción de mensajes

El servicio Bot Connector le permite a un bot comunicarse a través de varios canales, como Skype, correo electrónico, Slack y mucho más. Facilita la comunicación entre el bot y el usuario mediante la retransmisión de [actividades](bot-framework-rest-connector-activities.md) desde el bot hacia el canal y viceversa. Cada actividad contiene información utilizada para enrutar el mensaje al destino adecuado junto con información sobre quién creó el mensaje, cuál es su contexto y quién es el destinatario del mismo. En este artículo se describe cómo usar el servicio Bot Connector para intercambiar actividades de **mensaje** entre el bot y el usuario en un canal. 

## <a id="create-reply"></a> Respuestas para un mensaje

### <a name="create-a-reply"></a>Creación de una respuesta 

Cuando el usuario envía un mensaje al bot, el bot recibirá el mensaje como un objeto `Activity` de tipo **mensaje**. Para crear una respuesta para el mensaje de un usuario, cree un nuevo objeto `Activity` y comience por establecer estas propiedades:

| Propiedad | Valor |
|----|----|
| conversation | Establezca esta propiedad como el contenido de la propiedad `conversation` en el mensaje del usuario. |
| De | Establezca esta propiedad como el contenido de la propiedad `recipient` en el mensaje del usuario. |
| locale | Establezca esta propiedad como el contenido de la propiedad `locale` en el mensaje del usuario, si está especificada. |
| recipient | Establezca esta propiedad como el contenido de la propiedad `from` en el mensaje del usuario. |
| replyToId | Establezca esta propiedad como el contenido de la propiedad `id` en el mensaje del usuario. |
| Tipo | Establezca esta propiedad como **message**. |

A continuación, establezca las propiedades que especifican la información que desea comunicarle al usuario. Por ejemplo, puede establecer la propiedad `text` para especificar el texto que se mostrará en el mensaje, la propiedad `speak` para especificar el texto que su bot dirá en voz alta y la propiedad `attachments` para especificar los adjuntos multimedia o las tarjetas enriquecidas que se incluirán en el mensaje. Para obtener información detallada acerca de las propiedades de mensaje más usadas, consulte [Create messages](bot-framework-rest-connector-create-messages.md) (Creación de mensajes).

### <a name="send-the-reply"></a>Envío de la respuesta

Use la propiedad `serviceUrl` de la solicitud entrante para [identificar la URI base](bot-framework-rest-connector-api-reference.md#base-uri) que el bot debería usar para emitir su respuesta. 

Para enviar una respuesta, emita esta solicitud: 

```http
POST /v3/conversations/{conversationId}/activities/{activityId}
```

En este URI de solicitud, reemplace **{conversationId}** por el valor de la propiedad `id` del objeto `conversation` dentro de la actividad (respuesta) y reemplace **{activityId}** por el valor de la propiedad `replyToId` dentro de la actividad (respuesta). Establezca el cuerpo de la solicitud en el objeto `Activity` que creó para representar el mensaje de respuesta.

En el ejemplo siguiente se muestra una solicitud que envía una respuesta sencilla de solo texto al mensaje de un usuario. En la solicitud del ejemplo, `https://smba.trafficmanager.net/apis` representa el URI base; el URI base para solicitudes que su bot emita puede ser distinto. Para obtener más información sobre cómo establecer el URI base, consulte [API Reference](bot-framework-rest-connector-api-reference.md#base-uri) (Referencia de la API).

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
        "name": "Pepper's News Feed"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "Convo1"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "SteveW"
    },
    "text": "My bot's reply",
    "replyToId": "5d5cdc723"
}
```

## <a id="send-message"></a> Envío de un mensaje (sin respuesta)

La mayoría de los mensajes que el bot envíe serán una respuesta a los mensajes que recibe del usuario. Sin embargo, puede haber ocasiones en las que su bot requiera enviar a la conversación un mensaje que no es una respuesta directa a algún mensaje del usuario. Por ejemplo, es posible que su bot requiera iniciar un nuevo tema de conversación o enviar un mensaje de despedida al final de la conversación. 

Para enviar a la conversación un mensaje que no sea una respuesta directa a algún mensaje del usuario, emita esta solicitud: 

```http
POST /v3/conversations/{conversationId}/activities
```

En este URI de solicitud, reemplace **{conversationId}** por el identificador de la conversación. 
    
Establezca el cuerpo de la solicitud en el objeto `Activity` que creó para representar el mensaje de respuesta.

> [!NOTE]
> Bot Framework no impone ninguna restricción en el número de mensajes que puede enviar un bot. Sin embargo, la mayoría de los canales aplican valores de limitación para impedir que los bots envíen un gran número de mensajes en un breve período de tiempo. Además, si el bot envía varios mensajes en una sucesión rápida, puede que el canal no siempre procese los mensajes en el orden correcto.

## <a name="start-a-conversation"></a>Inicio de una conversación

Puede haber ocasiones en las que su bot debe iniciar una conversación con uno o varios usuarios. Para iniciar una conversación con algún usuario en un canal, su bot debe conocer la información de su cuenta y la información de la cuenta del usuario en dicho canal. 

> [!TIP]
> Si es posible que su bot deba iniciar conversaciones con sus usuarios en el futuro, copie en caché la información de la cuenta de usuario, además de otra información pertinente, como las preferencias del usuario, la configuración regional y la dirección URL del servicio (para usarla como la URI base en la solicitud de inicio de la conversación). 

Para iniciar una conversación, emita esta solicitud: 

```http
POST /v3/conversations
```

Establezca el cuerpo de la solicitud en un objeto `ConversationParameters` que especifica la información de la cuenta del bot y la información de la cuenta de los usuarios que se van a incluir en la conversación.

> [!NOTE]
> No todos los canales admiten conversaciones de grupo. Consulte la documentación del canal para determinar si un canal es compatible con las conversaciones de grupo e identificar el número máximo de participantes que un canal permite en una conversación.

En el ejemplo siguiente se muestra una solicitud que inicia una conversación. En la solicitud del ejemplo, `https://smba.trafficmanager.net/apis` representa el URI base; el URI base para solicitudes que su bot emita puede ser distinto. Para obtener más información sobre cómo establecer el URI base, consulte [API Reference](bot-framework-rest-connector-api-reference.md#base-uri) (Referencia de la API).

```http
POST https://smba.trafficmanager.net/apis/v3/conversations 
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "bot": {
        "id": "12345678",
        "name": "bot's name"
    },
    "isGroup": false,
    "members": [
        {
            "id": "1234abcd",
            "name": "recipient's name"
        }
    ],
    "topicName": "News Alert"
}
```

Si se establece la conversación con los usuarios especificados, la respuesta contendrá un identificador para identificar la conversación. 

```json
{
    "id": "abcd1234"
}
```

Por lo tanto, el bot puede usar este identificador de conversación para [enviar un mensaje](#send-message) a los usuarios dentro de la conversación.

## <a name="additional-resources"></a>Recursos adicionales

- [Activities overview](bot-framework-rest-connector-activities.md) (Introducción a las actividades)
- [Creación de mensajes](bot-framework-rest-connector-create-messages.md)
- [Esquema Activity de Bot Framework](https://aka.ms/botSpecs-activitySchema)
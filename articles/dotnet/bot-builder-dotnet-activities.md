---
title: Introducción a las actividades | Microsoft Docs
description: Conozca sobre los diferentes tipos de actividades disponibles en Bot Framework SDK para. NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 076e460f393c5db524cfade81e5c007484fe2cca
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225670"
---
# <a name="activities-overview"></a>Introducción a las actividades

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Activity concept overview](../includes/snippet-dotnet-concept-activity.md)]

## <a name="activity-types-in-the-bot-framework-sdk-for-net"></a>Tipos de actividades en Bot Framework SDK para .NET

Bot Framework SDK para. NET admite los siguientes tipos de actividades.

| Tipo de actividad | Interfaz | DESCRIPCIÓN |
|------|------|------|
| [message](#message) | IMessageActivity | Representa una comunicación entre el bot y el usuario. |
| [conversationUpdate](#conversationupdate) | IConversationUpdateActivity | Indica que el bot se agregó a una conversación, que otros miembros se agregaron o se quitaron de la conversación, o bien que los metadatos de la conversación han cambiado. |
| [contactRelationUpdate](#contactrelationupdate) | IContactRelationUpdateActivity | Indica que el bot se agregó o quitó de la lista de contactos de un usuario. |
| [typing](#typing) | ITypingActivity | Indica que el usuario o el bot en el otro extremo de la conversación está redactando una respuesta. | 
| [deleteUserData](#deleteuserdata) | N/D | Indica a un bot que un usuario ha solicitado que el bot elimine todos los datos de usuario que haya podido almacenar. |
| [endOfConversation](#endofconversation) | IEndOfConversationActivity | Indica el final de una conversación. |
| [event](#event) | IEventActivity | Representa una comunicación enviada a un bot que no es visible para el usuario. |
| [invoke](#invoke) | IInvokeActivity | Representa una comunicación enviada a un bot para solicitarle que realice una operación específica. Este tipo de actividad está reservado para uso interno de Microsoft Bot Framework. |
| [messageReaction](#messagereaction) | IMessageReactionActivity | Indica que un usuario ha reaccionado a una actividad existente. Por ejemplo, un usuario hace clic en el botón "Me gusta" de un mensaje. |

## <a name="message"></a>Mensaje

El bot enviará actividades de **mensaje** para comunicar información a los usuarios y recibir actividades de **mensaje** de ellos. Algunos mensajes pueden constar simplemente de texto sin formato, mientras que otros pueden incluir contenido más enriquecido, como [texto que se va a decir](bot-builder-dotnet-text-to-speech.md), [acciones sugeridas](bot-builder-dotnet-add-suggested-actions.md), [datos adjuntos multimedia](bot-builder-dotnet-add-media-attachments.md), [tarjetas enriquecidas](bot-builder-dotnet-add-rich-card-attachments.md) y [datos específicos del canal](bot-builder-dotnet-channeldata.md). Para información sobre las propiedades de mensaje más usadas, consulte [Creación de mensajes](bot-builder-dotnet-create-messages.md).

## <a name="conversationupdate"></a>conversationUpdate

Un bot recibe una actividad **conversationUpdate** cada vez que se ha agregado a una conversación, se han agregado otros miembros a la conversación (o se han eliminado de ella), o bien han cambiado los metadatos de la conversación. 

Si se han agregado miembros a la conversación, la propiedad `MembersAdded` de los miembros de la actividad contendrá una matriz de objetos `ChannelAccount` para identificar a los miembros nuevos. 

Para determinar si el bot se ha agregado a la conversación (es decir, es uno de los miembros nuevos), evalúe si el valor `Recipient.Id` para la actividad (es decir, el identificador del bot) coincide con la propiedad `Id` de cualquiera de las cuentas de la matriz `MembersAdded`.

Si se han quitado miembros de la conversación, la propiedad `MembersRemoved` contendrá una matriz de objetos `ChannelAccount` para identificar a los miembros que se han eliminado. 

> [!TIP]
> Si el bot recibe una actividad **conversationUpdate** en la que se indica que un usuario se ha unido a la conversación, puede elegir que le responda enviando un mensaje de bienvenida a ese usuario. 

## <a name="contactrelationupdate"></a>contactRelationUpdate

Un bot recibe una actividad **contactRelationUpdate** siempre que se agrega o se quita de la lista de contactos de un usuario. El valor de la propiedad `Action` de la actividad (add | remove) indica si el bot se ha agregado o quitado de la lista de contactos del usuario.

## <a name="typing"></a>typing

Un bot recibe una actividad **typing** para indicar que el usuario está escribiendo una respuesta. Un bot puede enviar una actividad **tyiping** para indicar al usuario que está trabajando para satisfacer una solicitud o compilar una respuesta. 

## <a name="deleteuserdata"></a>deleteUserData

Un bot recibe una actividad **deleteUserData** cuando un usuario solicita la eliminación de cualquier dato que el bot haya conservado previamente para el usuario. Si el bot recibe este tipo de actividad, debe eliminar cualquier información de identificación personal (PII) que haya almacenado previamente para el usuario que ha realizado la solicitud.

## <a name="endofconversation"></a>endOfConversation 

Un bot recibe una actividad **endOfConversation** para indicar que el usuario ha puesto fin a la conversación. Un bot podría enviar una actividad **endOfConversation** para indicar al usuario que la conversación está llegando a su fin. 

## <a name="event"></a>event

El bot puede recibir una actividad **event** de un proceso externo o un servicio que quiere transmitir información al bot sin que sea visible para los usuarios. Normalmente, el remitente de una actividad **event** no espera que el bot acuse recibo de ninguna manera.

## <a name="invoke"></a>invoke

El bot puede recibir una actividad **invoke** que representa una solicitud para que realice una operación específica. El remitente de una actividad **invoke** normalmente espera que el bot confirme la recepción mediante la respuesta HTTP. Este tipo de actividad está reservado para uso interno de Microsoft Bot Framework.

## <a name="messagereaction"></a>messageReaction

Algunos canales enviarán actividades **messageReaction** al bot cuando un usuario reacciona a una actividad existente. Por ejemplo, un usuario hace clic en el botón "Me gusta" de un mensaje. La propiedad **ReplyToId** indicará a qué actividad ha reaccionado al usuario.

La actividad **messageReaction** puede corresponder a cualquier número de **messageReactionTypes** definidos por el canal. Por ejemplo, "Like" o "PlusOne" son tipos de reacción que un canal puede enviar. 

## <a name="additional-resources"></a>Recursos adicionales

- [Envío y recepción de actividades](bot-builder-dotnet-connector.md)
- [Creación de mensajes](bot-builder-dotnet-create-messages.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Clase Activity</a>

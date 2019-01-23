---
title: Introducción a las actividades | Microsoft Docs
description: Conozca sobre los diferentes tipos de actividades disponibles en Bot Connector Service.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: db09888c5f782b065ff6bd108ce93379cd42527a
ms.sourcegitcommit: b94361234816e6b95459f142add936732fc40344
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/15/2019
ms.locfileid: "54317685"
---
# <a name="activities-overview"></a>Introducción a las actividades

Bot Connector Service intercambia información entre el bot y el canal (usuario) pasando un objeto [Activity][Activity]. El tipo de actividad más común es **message**, pero hay otros tipos de actividades que se pueden usar para comunicar distintos tipos de información a un bot o canal. 

## <a name="activity-types-in-the-bot-connector-service"></a>Tipos de actividad en Bot Connector Service

Bot Connector Service admite los siguientes tipos de actividades.

| Tipo de actividad | DESCRIPCIÓN |
|------|------|------|
| Mensaje | Representa una comunicación entre el bot y el usuario. |
| conversationUpdate | Indica que el bot se agregó a una conversación, que otros miembros se agregaron o se quitaron de la conversación, o bien que los metadatos de la conversación han cambiado. |
| contactRelationUpdate | Indica que el bot se agregó o quitó de la lista de contactos de un usuario. |
| typing | Indica que el usuario o el bot en el otro extremo de la conversación está redactando una respuesta. | 
| deleteUserData | Indica a un bot que un usuario ha solicitado que el bot elimine todos los datos de usuario que haya podido almacenar. |
| endOfConversation | Indica el final de una conversación. |

## <a name="message"></a>Mensaje

El bot enviará actividades de **mensaje** para comunicar información a los usuarios y recibir actividades de **mensaje** de ellos. Algunos mensajes pueden ser de texto sin formato, mientras que otros pueden incluir contenido más enriquecido como [elementos multimedia adjuntos](bot-framework-rest-connector-add-media-attachments.md), [botones y tarjetas](bot-framework-rest-connector-add-rich-cards.md), o [datos específicos del canal](bot-framework-rest-connector-channeldata.md). Para información sobre las propiedades de mensaje más usadas, consulte [Creación de mensajes](bot-framework-rest-connector-create-messages.md). Para más información acerca de cómo enviar y recibir mensajes, consulte [Envío y recepción de mensajes](bot-framework-rest-connector-send-and-receive-messages.md). 

## <a name="conversationupdate"></a>conversationUpdate

Un bot recibe una actividad **conversationUpdate** cada vez que se ha agregado a una conversación, se han agregado otros miembros a la conversación (o se han eliminado de ella), o bien han cambiado los metadatos de la conversación. 

Si se han agregado miembros a la conversación, la propiedad `addedMembers` de la actividad identificará a los miembros nuevos. Si se han eliminado miembros de la conversación, la propiedad `removedMembers` identificará a los miembros eliminados. 

> [!TIP]
> Si el bot recibe una actividad **conversationUpdate** en la que se indica que un usuario se ha unido a la conversación, puede elegir que le responda enviando un mensaje de bienvenida a ese usuario. 

## <a name="contactrelationupdate"></a>contactRelationUpdate

Un bot recibe una actividad **contactRelationUpdate** siempre que se agrega o se quita de la lista de contactos de un usuario. El valor de la propiedad `action` de la actividad (add | remove) indica si el bot se ha agregado o quitado de la lista de contactos del usuario.

## <a name="typing"></a>typing

Un bot recibe una actividad **typing** para indicar que el usuario está escribiendo una respuesta. Un bot puede enviar una actividad **tyiping** para indicar al usuario que está trabajando para satisfacer una solicitud o compilar una respuesta. 

## <a name="deleteuserdata"></a>deleteUserData

Un bot recibe una actividad **deleteUserData** cuando un usuario solicita la eliminación de cualquier dato que el bot haya conservado previamente para el usuario. Si el bot recibe este tipo de actividad, debe eliminar cualquier información de identificación personal (PII) que haya almacenado previamente para el usuario que ha realizado la solicitud.

## <a name="endofconversation"></a>endOfConversation 

Un bot recibe una actividad **endOfConversation** para indicar que el usuario ha puesto fin a la conversación. Un bot podría enviar una actividad **endOfConversation** para indicar al usuario que la conversación está llegando a su fin. 

## <a name="additional-resources"></a>Recursos adicionales

- [Creación de mensajes](bot-framework-rest-connector-create-messages.md)
- [Envío y recepción de mensajes](bot-framework-rest-connector-send-and-receive-messages.md)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object

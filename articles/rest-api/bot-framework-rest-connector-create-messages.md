---
title: Crear mensajes con la API de Bot Connector | Microsoft Docs
description: Obtenga información sobre las propiedades de mensaje usadas habitualmente en la API de Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 727a319ef479a1f8954151d0e7dcaefaf3d249a1
ms.sourcegitcommit: e815e786413296deea0bd78e5a495df329a9a7cb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2019
ms.locfileid: "70876160"
---
# <a name="create-messages"></a>Creación de mensajes

El bot enviará objetos [Actividad][] de tipo **mensaje** para comunicar información a los usuarios y, del mismo modo, recibirá actividades de **mensajes** de los usuarios. Algunos mensajes pueden constar simplemente de texto sin formato, mientras que otros pueden incluir contenido más enriquecido, como [texto que se va a decir](bot-framework-rest-connector-text-to-speech.md), [acciones sugeridas](bot-framework-rest-connector-add-suggested-actions.md), [datos adjuntos multimedia](bot-framework-rest-connector-add-media-attachments.md), [tarjetas enriquecidas](bot-framework-rest-connector-add-rich-cards.md) y [datos específicos del canal](bot-framework-rest-connector-channeldata.md). En este artículo se describen algunas propiedades de mensaje usadas habitualmente.

## <a name="message-text-and-formatting"></a>Texto y formato del mensaje

Se puede aplicar formato al texto del mensaje mediante **plain**, **markdown** o **xml**. El formato predeterminado de la propiedad `textFormat` es **markdown** e interpreta el texto mediante los estándares de formato de Markdown. El nivel de compatibilidad con el formato de texto según el canal. Para ver si una característica que quiere usar es compatible con el canal de destino, realice una vista previa de la característica mediante [Channel Inspector][ChannelInspector]. 

La propiedad `textFormat` de un objeto [Actividad][] se puede usar para especificar el formato del texto. Por ejemplo, para crear un mensaje básico que contenga solo texto sin formato, establezca la propiedad `textFormat` del objeto `Activity` en **plain**, establezca la propiedad `text` en el contenido del mensaje y establezca la propiedad `locale` en la configuración regional del remitente. 

## <a name="attachments"></a>Datos adjuntos

La propiedad `attachments` de un objeto [Actividad][] puede usarse para enviar datos adjuntos multimedia sencillos (imagen, audio, vídeo, archivo) y tarjetas enriquecidas. Para más información, vea [Add media attachments to messages](bot-framework-rest-connector-add-media-attachments.md) (Agregar datos adjuntos multimedia a mensajes) y [Add rich cards to messages](bot-framework-rest-connector-add-rich-cards.md) (Agregar tarjetas enriquecidas a mensajes).

## <a name="entities"></a>Entidades

La propiedad `entities` de un objeto [Actividad][] es una matriz de objetos <a href="http://schema.org/" target="_blank">schema.org</a> de extremo abierto que permite el intercambio de metadatos contextuales comunes entre el canal y el bot.

### <a name="mention-entities"></a>Entidades de mención

Muchos canales ofrecen la posibilidad de que un bot o usuario "mencione" a alguien dentro del contexto de una conversación. Para mencionar a un usuario en un mensaje, rellene la propiedad `entities` del mensaje con un objeto [Mention][]. 

### <a name="place-entities"></a>Entidades de lugar

Para transmitir <a href="https://schema.org/Place" target="_blank">información relacionada con la ubicación</a> dentro de un mensaje, rellene la propiedad `entities` del mensaje con un objeto [Place][]. 

## <a name="channel-data"></a>Channel data

La propiedad `channelData` de un objeto [Actividad][] puede usarse para implementar la funcionalidad específica del canal. Para más información, vea [Implementación de una funcionalidad específica de canal](bot-framework-rest-connector-channeldata.md).

## <a name="text-to-speech"></a>Texto a voz

La propiedad `speak` del objeto [Actividad][] puede usarse para especificar el texto que va a decir el bot en un canal habilitado para voz y la propiedad `inputHint` del objeto `Activity` se puede usar para influir en el estado del micrófono del cliente. Para más información, vea [Incorporación de voz a mensajes](bot-framework-rest-connector-text-to-speech.md) e [Incorporación de sugerencias de entrada a mensajes](bot-framework-rest-connector-add-input-hints.md).

## <a name="suggested-actions"></a>Acciones sugeridas

La propiedad `suggestedActions` de un objeto [Actividad][] puede usarse para presentar los botones que el usuario puede pulsar para proporcionar la entrada. A diferencia de los botones que aparecen en las tarjetas enriquecidas (que permanecen visibles y accesibles para el usuario incluso después de que se pulsen), los botones que aparecen en el panel de acciones sugeridas desaparecerán una vez que el usuario haya hecho una selección. Para más información, vea [Incorporación de acciones sugeridas a mensajes](bot-framework-rest-connector-add-suggested-actions.md).

## <a name="additional-resources"></a>Recursos adicionales

- [Preview features with the Channel Inspector][ChannelInspector] (Vista previa de las características con el Inspector de canales)
- [Activities overview](https://aka.ms/botSpecs-activitySchema) (Introducción a las actividades)
- [Envío y recepción de mensajes](bot-framework-rest-connector-send-and-receive-messages.md)
- [Incorporación de datos adjuntos con elementos multimedia a mensajes](bot-framework-rest-connector-add-media-attachments.md)
- [Incorporación de tarjetas enriquecidas a mensajes](bot-framework-rest-connector-add-rich-cards.md)
- [Incorporación de voz a mensajes](bot-framework-rest-connector-text-to-speech.md)
- [Incorporación de sugerencias de entrada a mensajes](bot-framework-rest-connector-add-input-hints.md)
- [Incorporación de acciones sugeridas a mensajes](bot-framework-rest-connector-add-suggested-actions.md)
- [Implementación de una funcionalidad específica de canal](bot-framework-rest-connector-channeldata.md)

[ChannelInspector]: ../bot-service-channel-inspector.md
[textFormating]: ../bot-service-channel-inspector.md#text-formatting

[Actividad]: bot-framework-rest-connector-api-reference.md#activity-object
[Mention]: bot-framework-rest-connector-api-reference.md#mention-object
[Place]: bot-framework-rest-connector-api-reference.md#place-object

---
title: Creación de mensajes con Bot Framework SDK para .NET | Microsoft Docs
description: Obtenga información sobre las propiedades de mensaje usadas habitualmente en Bot Framework SDK para. NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 96a0c63575b1e77418262a22050013413f39141f
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225920"
---
# <a name="create-messages"></a>Creación de mensajes

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

El bot enviará [actividades](bot-builder-dotnet-activities.md) de **mensajes** para comunicar información a los usuarios y, del mismo modo, recibirá actividades de **mensajes** de los usuarios. Algunos mensajes pueden constar simplemente de texto sin formato, mientras que otros pueden incluir contenido más enriquecido, como [texto que se va a decir](bot-builder-dotnet-text-to-speech.md), [acciones sugeridas](bot-builder-dotnet-add-suggested-actions.md), [datos adjuntos multimedia](bot-builder-dotnet-add-media-attachments.md), [tarjetas enriquecidas](bot-builder-dotnet-add-rich-card-attachments.md) y [datos específicos del canal](bot-builder-dotnet-channeldata.md). 

En este artículo se describen algunas propiedades de mensaje usadas habitualmente.

## <a name="customizing-a-message"></a>Personalización de un mensaje

Para tener más control sobre el formato del texto de los mensajes, puede crear un mensaje personalizado mediante el objeto [Activity](https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html) y establecer las propiedades necesarias antes de enviarlo al usuario.

En este ejemplo se muestra cómo crear un objeto `message` personalizado y establecer las propiedades `Text`, `TextFormat` y `Local`.

[!code-csharp[Set message properties](../includes/code/dotnet-create-messages.cs#setBasicProperties)]

La propiedad `TextFormat` de un mensaje puede usarse para especificar el formato del texto. La propiedad `TextFormat` se puede establecer en **plain**, **markdown** o **xml**. El valor predeterminado de `TextFormat` es **markdown**. 

## <a name="attachments"></a>Datos adjuntos

La propiedad `Attachments` de una actividad de mensajes puede usarse para enviar y recibir datos adjuntos multimedia sencillos (imagen, audio, vídeo, archivo) y tarjetas enriquecidas. Para más información, vea [Add media attachments to messages](bot-builder-dotnet-add-media-attachments.md) (Agregar datos adjuntos multimedia a mensajes) y [Add rich cards to messages](bot-builder-dotnet-add-rich-card-attachments.md) (Agregar tarjetas enriquecidas a mensajes).

## <a name="entities"></a>Entidades

La propiedad `Entities` de un mensaje es una matriz de objetos <a href="http://schema.org/" target="_blank">schema.org</a> de extremo abierto que permite el intercambio de metadatos contextuales comunes entre el canal y el bot.

### <a name="mention-entities"></a>Entidades de mención

Muchos canales ofrecen la posibilidad de que un bot o usuario "mencione" a alguien dentro del contexto de una conversación. Para mencionar a un usuario en un mensaje, rellene la propiedad `Entities` del mensaje con un objeto `Mention`. El objeto `Mention` contiene estas propiedades: 

| Propiedad | DESCRIPCIÓN | 
|----|----|
| Escriba | Tipo de la entidad ("mention") | 
| Mentioned | Objeto `ChannelAccount` que indica a qué usuario se ha mencionado | 
| Texto | Texto de la propiedad `Activity.Text` que representa la mención en sí misma (puede estar vacío o ser nulo) |

En este ejemplo de código se muestra cómo se agrega una entidad `Mention` a la colección `Entities`.

[!code-csharp[set Mention](../includes/code/dotnet-create-messages.cs#setMention)]

> [!TIP]
> Al intentar determinar la intención del usuario, puede que el bot omita la parte del mensaje en la que se menciona. Llame al método `GetMentions` y evalúe los objetos `Mention` devueltos en la respuesta.

### <a name="place-objects"></a>Objetos Place

La <a href="https://schema.org/Place" target="_blank">información relacionada con la ubicación</a> se puede transmitir dentro de un mensaje si se rellena la propiedad `Entities` del mensaje con un objeto `Place` o `GeoCoordinates`. 

El objeto `Place` contiene estas propiedades:

| Propiedad | DESCRIPCIÓN | 
|----|----|
| Escriba | Tipo de la entidad ("Place") |
| Dirección | Descripción u objeto `PostalAddress` (en un futuro) | 
| Geoárea | GeoCoordinates | 
| HasMap | Dirección URL de un mapa u objeto `Map` (en un futuro) |
| NOMBRE | Nombre del lugar |

El objeto `GeoCoordinates` contiene estas propiedades:

| Propiedad | DESCRIPCIÓN | 
|----|----|
| Escriba | Tipo de la entidad ("GeoCoordinates") |
| NOMBRE | Nombre del lugar |
| Longitud | Longitud de la ubicación (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) | 
| Latitud | Latitud de la ubicación (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) | 
| Elevation | Altitud de la ubicación (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) | 

En este ejemplo de código se muestra cómo se agrega una entidad `Place` a la colección `Entities`:

[!code-csharp[set GeoCoordinates](../includes/code/dotnet-create-messages.cs#setGeoCoord)]

### <a name="consume-entities"></a>Consumir entidades

Para consumir entidades, use la palabra clave `dynamic` o clases fuertemente tipadas.

En este ejemplo de código se muestra cómo usar la palabra clave `dynamic` para procesar una entidad en la propiedad `Entities` de un mensaje:

[!code-csharp[examine entity using dynamic keyword](../includes/code/dotnet-create-messages.cs#examineEntity1)]

En este ejemplo de código se muestra cómo usar una clase fuertemente tipada para procesar una entidad en la propiedad `Entities` de un mensaje:

[!code-csharp[examine entity using typed class](../includes/code/dotnet-create-messages.cs#examineEntity2)]

## <a name="channel-data"></a>Datos de canal

La propiedad `ChannelData` de una actividad de mensaje puede usarse para implementar la funcionalidad específica del canal. Para más información, vea [Implementación de una funcionalidad específica de canal](bot-builder-dotnet-channeldata.md).

## <a name="text-to-speech"></a>Texto a voz

La propiedad `Speak` de una actividad de mensaje puede usarse para especificar el texto que va a decir el bot en un canal habilitado para voz. La propiedad `InputHint` de una actividad de mensajes puede usarse para controlar el estado del micrófono y del cuadro de entrada del cliente (si los hay). Para más información, vea [Incorporación de voz a mensajes](bot-builder-dotnet-text-to-speech.md).

## <a name="suggested-actions"></a>Acciones sugeridas

La propiedad `SuggestedActions` de una actividad de mensajes puede usarse para presentar los botones que el usuario puede pulsar para proporcionar la entrada. A diferencia de los botones que aparecen en las tarjetas enriquecidas (que permanecen visibles y accesibles para el usuario incluso después de que se pulsen), los botones que aparecen en el panel de acciones sugeridas desaparecerán una vez que el usuario haya hecho una selección. Para más información, vea [Incorporación de acciones sugeridas a mensajes](bot-builder-dotnet-add-suggested-actions.md).

## <a name="next-steps"></a>Pasos siguientes

Un bot y un usuario pueden enviarse mensajes entre sí. Cuando el mensaje es más complejo, el bot puede enviar una tarjeta enriquecida en un mensaje al usuario. Las tarjetas enriquecidas abarcan muchos de los escenarios de presentación e interacción que suelen ser necesarios en la mayoría de los bots.

> [!div class="nextstepaction"]
> [Enviar una tarjeta enriquecida en un mensaje](bot-builder-dotnet-add-rich-card-attachments.md)

## <a name="additional-resources"></a>Recursos adicionales

- [Activities overview](bot-builder-dotnet-activities.md) (Introducción a las actividades)
- [Envío y recepción de actividades](bot-builder-dotnet-connector.md)
- [Incorporación de datos adjuntos con elementos multimedia a mensajes](bot-builder-dotnet-add-media-attachments.md)
- [Incorporación de tarjetas enriquecidas a mensajes](bot-builder-dotnet-add-rich-card-attachments.md)
- [Incorporación de voz a mensajes](bot-builder-dotnet-text-to-speech.md)
- [Incorporación de acciones sugeridas a mensajes](bot-builder-dotnet-add-suggested-actions.md)
- [Implementación de una funcionalidad específica de canal](bot-builder-dotnet-channeldata.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Clase Activity</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">Interfaz IMessageActivity</a>


---
title: Entidades y tipos de actividad | Microsoft Docs
description: Entidades y tipos de actividad.
keywords: entidades de mención, tipos de actividad, consumir entidades
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/01/2018
ms.openlocfilehash: d329fcbe5b4a34cb3e9c1fbf0160c5248020a508
ms.sourcegitcommit: cb0b70d7cf1081b08eaf1fddb69f7db3b95b1b09
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/09/2018
ms.locfileid: "51332969"
---
# <a name="entities-and-activity-types"></a>Entidades y tipos de actividad

Las entidades son una parte de una actividad y proporcionan información adicional sobre la actividad o la conversación.

[!include[Entity boilerplate](includes/snippet-entity-boilerplate.md)]

## <a name="entities"></a>Entidades

La propiedad *entities* de un mensaje es una matriz de objetos <a href="http://schema.org/" target="_blank">schema.org</a> de extremo abierto que permite el intercambio de metadatos contextuales comunes entre el canal y el bot.

### <a name="mention-entities"></a>Entidades de mención

Muchos canales ofrecen la posibilidad de que un bot o usuario "mencione" a alguien dentro del contexto de una conversación.
Para mencionar a un usuario en un mensaje, rellene la propiedad entities del mensaje con un objeto *mention*.
El objeto mention contiene estas propiedades:

| Propiedad | DESCRIPCIÓN |
|----|----|
| Escriba | Tipo de la entidad ("mention") |
| Mentioned | Objeto de cuenta de canal que indica a qué usuario se ha mencionado | 
| Texto | Texto de la propiedad *activity.text* que representa la mención en sí misma (puede estar vacío o ser NULL) |

En este ejemplo de código se muestra cómo se agrega una entidad mention a la colección entities:

# <a name="ctabcs"></a>[C#](#tab/cs)
[!code-csharp[set Mention](includes/code/dotnet-create-messages.cs#setMention)]

> [!TIP]
> Al intentar determinar la intención del usuario, puede que el bot omita la parte del mensaje en la que se menciona. Llame al método `GetMentions` y evalúe los objetos `Mention` devueltos en la respuesta.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
var entity = context.activity.entities;

const mention = {
    type: "Mention",
    text: "@johndoe",
    mentioned: {
        name: "John Doe",
        id: "UV341235"
    }
}

entity = [mention];
```

---

### <a name="place-objects"></a>Objetos Place

La <a href="https://schema.org/Place" target="_blank">información relacionada con la ubicación</a> se puede transmitir dentro de un mensaje si se rellena la propiedad entities del mensaje con un objeto *Place* o *GeoCoordinates*.

El objeto place contiene estas propiedades:

| Propiedad | DESCRIPCIÓN |
|----|----|
| Escriba | Tipo de la entidad ("Place") |
| Dirección | Descripción u objeto de dirección postal (en un futuro) |
| Geoárea | GeoCoordinates |
| HasMap | Dirección URL de un mapa u objeto de mapa (en un futuro) |
| NOMBRE | Nombre del lugar |

El objeto geoCoordinates contiene estas propiedades:

| Propiedad | DESCRIPCIÓN |
|----|----|
| Escriba | Tipo de la entidad ("GeoCoordinates") |
| NOMBRE | Nombre del lugar |
| Longitud | Longitud de la ubicación (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |
| Longitud | Latitud de la ubicación (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |
| Elevation | Altitud de la ubicación (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |

En este ejemplo de código se muestra cómo se agrega una entidad place a la colección entities:

# <a name="ctabcs"></a>[C#](#tab/cs)
[!code-csharp[set GeoCoordinates](includes/code/dotnet-create-messages.cs#setGeoCoord)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
var entity = context.activity.entities;

const place = {
    elavation: 100,
    type: "GeoCoordinates",
    name : "myPlace",
    latitude: 123,
    longitude: 234
};

entity = [place];

```

---

### <a name="consume-entities"></a>Consumir entidades

# <a name="ctabcs"></a>[C#](#tab/cs)

Para consumir entidades, use la palabra clave `dynamic` o clases fuertemente tipadas.

En este ejemplo de código se muestra cómo usar la palabra clave `dynamic` para procesar una entidad en la propiedad `Entities` de un mensaje:

[!code-csharp[examine entity using dynamic keyword](includes/code/dotnet-create-messages.cs#examineEntity1)]

En este ejemplo de código se muestra cómo usar una clase fuertemente tipada para procesar una entidad en la propiedad `Entities` de un mensaje:

[!code-csharp[examine entity using typed class](includes/code/dotnet-create-messages.cs#examineEntity2)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

En este ejemplo de código se muestra cómo procesar una entidad en la propiedad `entity` de un mensaje:

```javascript
if (entity[0].type === "GeoCoordinates" && entity[0].latitude > 34) {
    // do something
}
```

---

## <a name="activity-types"></a>Tipos de actividad

Este ejemplo de código se muestra cómo procesar una actividad de tipo **message**:

# <a name="ctabcs"></a>[C#](#tab/cs)

```cs
if (context.Activity.Type == ActivityTypes.Message){
    // do something
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```js
if(context.activity.type === 'message'){
    // do something
}
```

---

Las actividades pueden ser de otros tipos además de **message** (el más común). Hay varios tipos de actividad:

| Tipo de actividad | Interfaz | DESCRIPCIÓN |
|-----|-----|-----|
| [message](#message) | IMessageActivity (C#) <br> Activity (JS) | Representa una comunicación entre el bot y el usuario. |
| [contactRelationUpdate](#contactrelationupdate) | IContactRelationUpdateActivity (C#) <br> Activity (JS) | Indica que el bot se agregó o quitó de la lista de contactos de un usuario. |
| [conversationUpdate](#conversationupdate) | IConversationUpdateActivity (C#) <br> Activity (JS) | Indica que el bot se agregó a una conversación, que otros miembros se agregaron o se quitaron de la conversación, o bien que los metadatos de la conversación han cambiado. |
| [deleteUserData](#deleteuserdata) | N/D | Indica a un bot que un usuario ha solicitado que el bot elimine todos los datos de usuario que haya podido almacenar. |
| [endOfConversation](#endofconversation) | IEndOfConversationActivity (C#) <br> Activity (JS) | Indica el final de una conversación. |
| [event](#event) | IEventActivity (C#) <br> Activity (JS) | Representa una comunicación enviada a un bot que no es visible para el usuario. |
| [installationUpdate](#installationupdate) | IInstallationUpdateActivity (C#) <br> Activity (JS) | Representa una instalación o desinstalación de un bot dentro de una unidad organizativa (por ejemplo, un inquilino de cliente o "equipo") de un canal. |
| [invoke](#invoke) | IInvokeActivity (C#) <br> Activity (JS) | Representa una comunicación enviada a un bot para solicitarle que realice una operación específica. Este tipo de actividad está reservado para uso interno de Microsoft Bot Framework. |
| [messageReaction](#messagereaction) | IMessageReactionActivity (C#) <br> Activity (JS) | Indica que un usuario ha reaccionado a una actividad existente. Por ejemplo, un usuario hace clic en el botón "Me gusta" de un mensaje. |
| [typing](#typing) | ITypingActivity (C#) <br> Activity (JS) | Indica que el usuario o el bot en el otro extremo de la conversación está redactando una respuesta. |

## <a name="message"></a>Mensaje

<!-- Only the last link is different. -->

::: moniker range="azure-bot-service-3.0"

El bot enviará actividades de mensaje para comunicar información a los usuarios y recibir actividades de mensaje de ellos.
Algunos mensajes pueden constar simplemente de texto sin formato, mientras que otros pueden incluir contenido más enriquecido, como texto hablado, [acciones sugeridas](v4sdk/bot-builder-howto-add-suggested-actions.md), [datos adjuntos multimedia](v4sdk/bot-builder-howto-add-media-attachments.md), [tarjetas enriquecidas](v4sdk/bot-builder-howto-add-media-attachments.md#send-a-hero-card) y [datos específicos del canal](~/dotnet/bot-builder-dotnet-channeldata.md).

::: moniker-end

::: moniker range="azure-bot-service-4.0"

El bot enviará actividades de mensaje para comunicar información a los usuarios y recibir actividades de mensaje de ellos.
Algunos mensajes pueden constar simplemente de texto sin formato, mientras que otros pueden incluir contenido más enriquecido, como texto hablado, [acciones sugeridas](v4sdk/bot-builder-howto-add-suggested-actions.md), [datos adjuntos multimedia](v4sdk/bot-builder-howto-add-media-attachments.md), [tarjetas enriquecidas](v4sdk/bot-builder-howto-add-media-attachments.md#send-a-hero-card) y [datos específicos del canal](~/v4sdk/bot-builder-channeldata.md).

::: moniker-end

## <a name="contactrelationupdate"></a>contactRelationUpdate

Un bot recibe una actividad de actualización de relación de contacto siempre que se agrega o se quita de la lista de contactos de un usuario. El valor de la propiedad de acción de la actividad (add | remove) indica si el bot se ha agregado o quitado de la lista de contactos del usuario.

## <a name="conversationupdate"></a>conversationUpdate

Un bot recibe una actividad de actualización de la conversación cada vez que se ha agregado a una conversación, se han agregado otros miembros a la conversación (o se han eliminado de ella), o bien han cambiado los metadatos de la conversación.

Si se han agregado miembros a la conversación, la propiedad added de los miembros de la actividad contendrá una matriz de objetos de cuenta de canal para identificar a los miembros nuevos.

Para determinar si el bot se ha agregado a la conversación (es decir, es uno de los miembros nuevos), evalúe si el valor de identificador de destinatario para la actividad (es decir, el identificador del bot) coincide con la propiedad Id de cualquiera de las cuentas de la matriz members added.

Si se han quitado miembros de la conversación, la propiedad removed de los miembros contendrá una matriz de objetos de cuenta de canal para identificar a los miembros que se han eliminado.

> [!TIP]
> Si el bot recibe una actividad de actualización de la conversación en la que se indica que un usuario se ha unido a la conversación, puede elegir que le responda enviando un mensaje de bienvenida a ese usuario.

## <a name="deleteuserdata"></a>deleteUserData

Un bot recibe una actividad de eliminación de datos de usuario cuando un usuario solicita la eliminación de cualquier dato que el bot haya conservado previamente para el usuario. Si el bot recibe este tipo de actividad, debe eliminar cualquier información de identificación personal (PII) que haya almacenado previamente para el usuario que ha realizado la solicitud.

## <a name="endofconversation"></a>endOfConversation

Un bot recibe una actividad de finalización de la conversación para indicar que el usuario ha puesto fin a la conversación. Un bot podría enviar una actividad de finalización de la conversación para indicar al usuario que la conversación está llegando a su fin.

## <a name="event"></a>event

El bot puede recibir una actividad de evento de un proceso externo o un servicio que quiere transmitir información al bot sin que sea visible para los usuarios. Normalmente, el remitente de una actividad de evento no espera que el bot acuse recibo de ninguna manera.

## <a name="installationupdate"></a>installationUpdate

Las actividades de actualización de la instalación representan una instalación o desinstalación de un bot dentro de una unidad organizativa (por ejemplo, un inquilino de cliente o "equipo") de un canal. Por lo general, las actividades de actualización de la instalación no representan la adición o eliminación de un canal. Los canales pueden enviar actividades de instalación cuando se agrega o se quita un bot de un inquilino, equipo u otra unidad organizativa dentro del canal. Los canales no deberían enviar actividades de instalación cuando el bot se instala o se quita de un canal.

## <a name="invoke"></a>invoke

El bot puede recibir una actividad invoke que representa una solicitud para que realice una operación específica.
El remitente de una actividad invoke normalmente espera que el bot confirme la recepción a través de la respuesta HTTP.
Este tipo de actividad está reservado para uso interno de Microsoft Bot Framework.

## <a name="messagereaction"></a>messageReaction

Algunos canales enviarán actividades de reacción de mensaje al bot cuando un usuario reacciona a una actividad existente. Por ejemplo, un usuario hace clic en el botón "Me gusta" de un mensaje. La propiedad ReplyToId indicará a qué actividad ha reaccionado al usuario.

La actividad de reacción de mensaje puede corresponder a cualquier número de tipos de reacción de mensaje definidos por el canal. Por ejemplo, "Like" o "PlusOne" son tipos de reacción que un canal puede enviar.

## <a name="typing"></a>typing

Un bot recibe una actividad de escritura para indicar que el usuario está escribiendo una respuesta.
Un bot puede enviar una actividad de escritura para indicar al usuario que está trabajando para satisfacer una solicitud o compilar una respuesta.

::: moniker range="azure-bot-service-3.0"

## <a name="additional-resources"></a>Recursos adicionales

- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Clase Activity</a>
::: moniker-end

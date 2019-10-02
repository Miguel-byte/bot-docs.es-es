---
title: Entidades y tipos de actividad | Microsoft Docs
description: Entidades y tipos de actividad.
keywords: entidades de mención, tipos de actividad, consumir entidades
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 03/01/2018
ms.openlocfilehash: e1eae45702a1eee94714f96425050948310c7520
ms.sourcegitcommit: e9cd857ee11945ef0b98a1ffb4792494dfaeb126
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/01/2019
ms.locfileid: "71693116"
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
| type | Tipo de la entidad ("mention") |
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
| type | Tipo de la entidad ("Place") |
| Dirección | Descripción u objeto de dirección postal (en un futuro) |
| Geoárea | GeoCoordinates |
| HasMap | Dirección URL de un mapa u objeto de mapa (en un futuro) |
| NOMBRE | Nombre del lugar |

El objeto geoCoordinates contiene estas propiedades:

| Propiedad | DESCRIPCIÓN |
|----|----|
| type | Tipo de la entidad ("GeoCoordinates") |
| NOMBRE | Nombre del lugar |
| Longitud | Longitud de la ubicación (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |
| Latitud | Latitud de la ubicación (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |
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
<!-- 
This code example show how to process an activity of type **message**:

# [C#](#tab/cs)

```cs
if (context.Activity.Type == ActivityTypes.Message){
    // do something
}
```

# [JavaScript](#tab/js)

```js
if(context.activity.type === 'message'){
    // do something
}
```

--- -->

Las actividades pueden ser de otros tipos además de **message** (el más común). Puede encontrar más explicaciones y detalles sobre los diferentes tipos de actividades en el [esquema de actividad de Bot Framework](https://aka.ms/botSpecs-activitySchema).

::: moniker range="azure-bot-service-3.0"

## <a name="additional-resources"></a>Recursos adicionales

- <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Clase Activity</a>
::: moniker-end

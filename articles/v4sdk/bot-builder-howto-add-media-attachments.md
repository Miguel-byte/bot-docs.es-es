---
title: Incorporación de elementos multimedia a los mensajes | Microsoft Docs
description: Obtenga información sobre cómo agregar elementos multimedia a los mensajes mediante Bot Framework SDK.
keywords: multimedia, mensajes, imágenes, audio, vídeo, archivos, MessageFactory, tarjetas enriquecidas, mensajes, tarjetas adaptables, tarjeta de imagen prominente, acciones sugeridas
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: cef8c3eba77e2cf42cf63e698f4dcca9beaa41dd
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2019
ms.locfileid: "59905098"
---
# <a name="add-media-to-messages"></a>Incorporación de elementos multimedia a los mensajes

[!INCLUDE[applies-to](../includes/applies-to.md)]

Un intercambio de mensajes entre el usuario y el bot puede contener datos adjuntos con elementos multimedia, como imágenes, vídeo, audio y archivos. Bot Framework SDK es compatible con la tarea de enviar mensajes enriquecidos al usuario. Para determinar el tipo de mensajes enriquecidos que admite un canal (Facebook, Skype, Slack, etc), consulte la documentación del canal para más información sobre las limitaciones.

Consulte cómo [diseñar la experiencia del usuario](../bot-service-design-user-experience.md) para obtener una lista de las tarjetas disponibles.

## <a name="send-attachments"></a>Envío de datos adjuntos

Para enviar el contenido de usuario como una imagen o un vídeo, puede agregar datos adjuntos o una lista de datos adjuntos a un mensaje.

Consulte cómo [diseñar la experiencia del usuario](../bot-service-design-user-experience.md) para obtener una lista de las tarjetas disponibles.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

La propiedad `Attachments` del objeto `Activity` contiene una matriz de objetos `Attachment` que representan los datos adjuntos con elementos multimedia y las tarjetas enriquecidas adjuntas al mensaje. Para agregar datos adjuntos con elementos multimedia a un mensaje, cree un objeto `Attachment` para la actividad `message` y establezca las propiedades `ContentType`, `ContentUrl` y `Name`.
El código fuente que se muestra a continuación se basa en el ejemplo [Handling Attachments](https://aka.ms/bot-attachments-sample-code) (sobre control de datos adjuntos). 

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

// Create an attachment.
var attachment = new Attachment
    {
        ContentUrl = "imageUrl.png",
        ContentType = "image/png",
        Name = "imageName",
    };

// Add the attachment to our reply.
reply.Attachments = new List<Attachment>() { attachment };

// Send the activity to the user.
await turnContext.SendActivityAsync(reply, cancellationToken: cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El código fuente que se muestra a continuación se basa en el ejemplo [JS Handling Attachments](https://aka.ms/bot-attachments-sample-code-js) (sobre control de datos adjuntos de JS).
Para enviar al usuario un único fragmento de contenido, como una imagen o un vídeo, puede enviar elementos multimedia contenidos en una dirección URL:

```javascript
const { ActionTypes, ActivityTypes, CardFactory } = require('botbuilder');

// Call function to get an attachment.
const reply = { type: ActivityTypes.Message };
reply.attachments = [this.getInternetAttachment()];
reply.text = 'This is an internet attachment.';
// Send the activity to the user.
await turnContext.sendActivity(reply);

/* function getInternetAttachment - Returns an attachment to be sent to the user from a HTTPS URL */
getInternetAttachment() {
        return {
            name: 'imageName.png',
            contentType: 'image/png',
            contentUrl: 'imageUrl.png'}
}
```

---

Si los datos adjuntos son una imagen, un audio o un vídeo, el servicio del conector comunicará los datos adjuntos al canal de una manera que permita que el [canal](bot-builder-channeldata.md) presente esos datos adjuntos dentro de la conversación. Si los datos adjuntos son un archivo, la dirección URL del archivo se presentará como un hipervínculo dentro de la conversación.

## <a name="send-a-hero-card"></a>Envío de una tarjeta de imagen prominente

Además de los datos adjuntos de imagen o vídeo sencillos, puede adjuntar una **tarjeta de imagen prominente** que le permite combinar las imágenes y los botones en un objeto y los envían al usuario. Markdown es compatible con la mayoría de los campos de texto, pero la compatibilidad puede variar según el canal.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Para redactar un mensaje con un botón y una tarjeta de imagen prominente, puede adjuntar `HeroCard` a un mensaje. El código fuente que se muestra a continuación se basa en el ejemplo [Handling Attachments](https://aka.ms/bot-attachments-sample-code) (sobre control de datos adjuntos).

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

// Create a HeroCard with options for the user to choose to interact with the bot.
var card = new HeroCard
{
    Text = "You can upload an image or select one of the following choices",
    Buttons = new List<CardAction>()
    {
        new CardAction(ActionTypes.ImBack, title: "1. Inline Attachment", value: "1"),
        new CardAction(ActionTypes.ImBack, title: "2. Internet Attachment", value: "2"),
        new CardAction(ActionTypes.ImBack, title: "3. Uploaded Attachment", value: "3"),
    },
};

// Add the card to our reply.
reply.Attachments = new List<Attachment>() { card.ToAttachment() };

await turnContext.SendActivityAsync(reply, cancellationToken: cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Para redactar un mensaje con un botón y una tarjeta de imagen prominente, puede adjuntar `HeroCard` a un mensaje. El código fuente que se muestra a continuación se basa en el ejemplo [JS Handling Attachments](https://aka.ms/bot-attachments-sample-code-js) (sobre control de datos adjuntos de JS):

```javascript
const { ActionTypes, ActivityTypes, CardFactory } = require('botbuilder');
// build buttons to display.
const buttons = [
            { type: ActionTypes.ImBack, title: '1. Inline Attachment', value: '1' },
            { type: ActionTypes.ImBack, title: '2. Internet Attachment', value: '2' },
            { type: ActionTypes.ImBack, title: '3. Uploaded Attachment', value: '3' }
];

// construct hero card.
const card = CardFactory.heroCard('', undefined,
buttons, { text: 'You can upload an image or select one of the following choices.' });

// add card to Activity.
const reply = { type: ActivityTypes.Message };
reply.attachments = [card];

// Send hero card to the user.
await turnContext.sendActivity(reply);
```

---

## <a name="process-events-within-rich-cards"></a>Procesamiento de eventos dentro de tarjetas enriquecidas

Para procesar eventos dentro de tarjetas enriquecidas, use los objetos de _acción de tarjeta_ para especificar qué debe ocurrir cuando el usuario hace clic en un botón o pulsa una sección de la tarjeta. Cada acción de la tarjeta tiene un _tipo_ y un _valor_.

Para que el funcionamiento sea correcto, asigne un tipo de acción a cada elemento en el que se puede hacer clic en la tarjeta. Esta tabla enumera y describe los tipos de acciones disponibles y lo que debería estar en la propiedad de valor asociada.

| Type | DESCRIPCIÓN | Valor |
| :---- | :---- | :---- |
| openUrl | Se abre una dirección URL en el explorador integrado. | Dirección URL que se va a abrir. |
| imBack | Envía un mensaje al bot y publica una respuesta visible en el chat. | Texto del mensaje para enviar. |
| postBack | Envía un mensaje al bot y puede no publicar una respuesta visible en el chat. | Texto del mensaje para enviar. |
| llamada | Inicia una llamada de teléfono. | Destino de una llamada telefónica con este formato: `tel:123123123123`. |
| playAudio | Reproduce audio. | La dirección URL del audio que se va a reproducir. |
| playVideo | Reproduce un vídeo. | La dirección URL del vídeo que se va a reproducir. |
| showImage | Muestra una imagen. | La dirección URL de la imagen que se va a mostrar. |
| DownloadFile | Descarga un archivo. | La dirección URL del archivo que se va a descargar. |
| signin | Inicia un proceso de inicio de sesión de OAuth. | La dirección URL del flujo de OAuth que se va a iniciar. |

## <a name="hero-card-using-various-event-types"></a>Tarjeta de imagen prominente a través de varios tipos de evento

El código siguiente muestra ejemplos que usan varios eventos de tarjeta enriquecida.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

var card = new HeroCard
{
    Buttons = new List<CardAction>()
    {
        new CardAction(title: "Much Quieter", type: ActionTypes.PostBack, value: "Shh! My Bot friend hears me."),
        new CardAction(ActionTypes.OpenUrl, title: "Azure Bot Service", value: "https://azure.microsoft.com/en-us/services/bot-service/"),
    },
};

```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {ActionTypes} = require("botbuilder");

const hero = MessageFactory.attachment(
    CardFactory.heroCard(
        'Holler Back Buttons',
        ['https://example.com/whiteShirt.jpg'],
        [{
            type: ActionTypes.ImBack,
            title: 'ImBack',
            value: 'You can ALL hear me! Shout Out Loud'
        },
        {
            type: ActionTypes.PostBack,
            title: 'PostBack',
            value: 'Shh! My Bot friend hears me. Much Quieter'
        },
        {
            type: ActionTypes.OpenUrl,
            title: 'OpenUrl',
            value: 'https://en.wikipedia.org/wiki/{cardContent.Key}'
        }]
    )
);

await context.sendActivity(hero);

```

---

## <a name="send-an-adaptive-card"></a>Envío de una tarjeta adaptable
Las tarjetas adaptables y MessageFactory se utilizan para enviar mensajes enriquecidos, incluidos textos, imágenes, vídeo, audio y archivos para comunicarse con los usuarios. Sin embargo, hay algunas diferencias entre ellos. 

En primer lugar, solo algunos canales admiten tarjetas adaptables y aquellos canales que las admiten, podrían hacerlo de un modo parcial. Por ejemplo, si envía una tarjeta adaptable en Facebook, los botones no funcionarán, pero los textos y las imágenes funcionan bien. MessageFactory es simplemente una clase asistente en Bot Framework SDK para automatizar los pasos de creación y es compatible con la mayoría de los canales. 

En segundo lugar, una tarjeta adaptable entrega los mensajes en formato de tarjeta y el canal determina el diseño de la tarjeta. El formato de los mensajes que entrega MessageFactory depende del canal y no es necesariamente en formato de tarjeta, a menos que la tarjeta adaptable sea parte de los datos adjuntos. 

Para encontrar la información más reciente sobre la compatibilidad de canales de tarjetas adaptables, consulte <a href="http://adaptivecards.io/designer/">Diseñador de tarjetas adaptables</a>.

Para usar las tarjetas adaptables, no olvide agregar el paquete `AdaptiveCards` de NuGet. 


> [!NOTE]
> Debe probar esta característica con los canales que el bot usará para determinar si esos canales admiten las tarjetas adaptables.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

El código fuente que se muestra a continuación se basa en el ejemplo [Using Adaptive Cards](https://aka.ms/bot-adaptive-cards-sample-code) (Uso de tarjetas adaptables):

```csharp
using AdaptiveCards;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using Newtonsoft.Json;

// Creates an attachment that contains an adaptive card
// filePath is the path to JSON file
private static Attachment CreateAdaptiveCardAttachment(string filePath)
{
    var adaptiveCardJson = File.ReadAllText(filePath);
    var adaptiveCardAttachment = new Attachment()
    {
        ContentType = "application/vnd.microsoft.card.adaptive",
        Content = JsonConvert.DeserializeObject(adaptiveCardJson),
    };
    return adaptiveCardAttachment;
}

// Create adaptive card and attach it to the message 
var cardAttachment = CreateAdaptiveCardAttachment(adaptiveCardJsonFilePath);
var reply = turnContext.Activity.CreateReply();
reply.Attachments = new List<Attachment>() { cardAttachment };

await turnContext.SendActivityAsync(reply, cancellationToken: cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El código fuente que se muestra a continuación se basa en el ejemplo [JS Using Adaptive Cards](https://aka.ms/bot-adaptive-cards-js-sample-code) (Uso de tarjetas adaptables de JS):

```javascript
const { BotFrameworkAdapter } = require('botbuilder');

// Import AdaptiveCard content.
const FlightItineraryCard = require('./resources/FlightItineraryCard.json');
const ImageGalleryCard = require('./resources/ImageGalleryCard.json');
const LargeWeatherCard = require('./resources/LargeWeatherCard.json');
const RestaurantCard = require('./resources/RestaurantCard.json');
const SolitaireCard = require('./resources/SolitaireCard.json');

// Create array of AdaptiveCard content, this will be used to send a random card to the user.
const CARDS = [
    FlightItineraryCard,
    ImageGalleryCard,
    LargeWeatherCard,
    RestaurantCard,
    SolitaireCard
];
// Select a random card to send.
const randomlySelectedCard = CARDS[Math.floor((Math.random() * CARDS.length - 1) + 1)];
// Send adaptive card.
await context.sendActivity({
      text: 'Here is an Adaptive Card:',
       attachments: [CardFactory.adaptiveCard(randomlySelectedCard)]
});
```

---

## <a name="send-a-carousel-of-cards"></a>Envío de un carrusel de cartas

Los mensajes también pueden incluir varios datos adjuntos en un diseño de carrusel, que coloca los datos adjuntos en paralelo y permite que el usuario se desplace por ellos.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

// Create the activity and attach a set of Hero cards.
var activity = MessageFactory.Carousel(
    new Attachment[]
    {
        new HeroCard(
            title: "title1",
            images: new CardImage[] { new CardImage(url: "imageUrl1.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button1", type: ActionTypes.ImBack, value: "item1")
            })
        .ToAttachment(),
        new HeroCard(
            title: "title2",
            images: new CardImage[] { new CardImage(url: "imageUrl2.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button2", type: ActionTypes.ImBack, value: "item2")
            })
        .ToAttachment(),
        new HeroCard(
            title: "title3",
            images: new CardImage[] { new CardImage(url: "imageUrl3.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button3", type: ActionTypes.ImBack, value: "item3")
            })
        .ToAttachment()
    });

// Send the activity as a reply to the user.
await turnContext.SendActivityAsync(reply, cancellationToken: cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// require MessageFactory and CardFactory from botbuilder.
const {MessageFactory, CardFactory} = require('botbuilder');

//  init message object
let messageWithCarouselOfCards = MessageFactory.carousel([
    CardFactory.heroCard('title1', ['imageUrl1'], ['button1']),
    CardFactory.heroCard('title2', ['imageUrl2'], ['button2']),
    CardFactory.heroCard('title3', ['imageUrl3'], ['button3'])
]);

await context.sendActivity(messageWithCarouselOfCards);
```

---

<!-- TODO: Add a media card, such as video or audion. Revisit which examples we put here and link to the 06 through 08 samples. -->

## <a name="additional-resources"></a>Recursos adicionales

Consulte cómo [diseñar la experiencia del usuario](../bot-service-design-user-experience.md) para obtener una lista de las tarjetas disponibles.

Para más información sobre el esquema, consulte el [esquema de la tarjeta de Bot Framework](https://aka.ms/botSpecs-cardSchema) y la [sección sobre la actividad de mensajes](https://aka.ms/botSpecs-activitySchema#message-activity) del esquema de actividad de Bot Framework.

El ejemplo de código se puede encontrar aquí para las tarjetas: [C#](https://aka.ms/bot-cards-sample-code)/[JS](https://aka.ms/bot-cards-js-sample-code), tarjetas adaptables: [C#](https://aka.ms/bot-adaptive-cards-sample-code)/[JS](https://aka.ms/bot-adaptive-cards-js-sample-code), datos adjuntos: [C#](https://aka.ms/bot-attachments-sample-code)/[JS](https://aka.ms/bot-attachments-sample-code-js), y acciones sugeridas: [C#](https://aka.ms/SuggestedActionsCSharp)/[JS](https://aka.ms/SuggestedActionsJS).
Consulte el repositorio de ejemplos de Bot Builder en [GitHub](https://aka.ms/bot-samples-readme) para obtener ejemplos adicionales.

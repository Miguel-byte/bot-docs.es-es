---
title: Referencia rápida de la migración de la versión v3 a la v4 de JavaScript | Microsoft Docs
description: Esquema de las principales diferencias del SDK de Bot Framework en las versiones v3 y v4 de JavaScript.
keywords: JavaScript, bot migration, dialogs, v3 bot
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2935c187b9c9687d4fbf207c2d78a135d3c8959f
ms.sourcegitcommit: 312a4593177840433dfee405335100ce59aac347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/12/2019
ms.locfileid: "73933779"
---
# <a name="javascript-migration-quick-reference"></a>Referencia rápida de la migración de JavaScript

El SDK BotBuilder para JavaScript v4 introduce varios cambios fundamentales que afectan a cómo se crean los bots. El propósito de esta guía es proporcionar una referencia rápida para resaltar las diferencias comunes al realizar una tarea en los SDK de las versiones v3 y v4.

- La forma en que la información pasa entre un bot y los canales ha cambiado.
    En v3, ha utilizado los objetos _conector_ y _sesión_.
    En v4, se reemplazan por los objetos _adaptador_ y _contexto de turno_.

- Además, los diálogos y las instancias de los bot se han desacoplado aún más.
    En v3, se registraron los diálogos directamente en el constructor del bot.
    En v4, ahora puede pasar diálogos a instancias de los bot como argumentos, lo que proporciona una mayor flexibilidad de composición.

- Además, la versión v4 proporciona una clase `ActivityHandler`, que ayuda a automatizar el control de diferentes tipos de actividades, tales como _mensajes_, _actualización de conversación_ y actividades de _eventos_.

Estas mejoras dan como resultado cambios en la sintaxis para el desarrollo de bots en JavaScript, especialmente en la creación de objetos de bot, la definición de diálogos y la codificación de la lógica del control de eventos.

En el resto de este tema se comparan las construcciones de la versión v3 de SDK de Bot Framework para JavaScript con su equivalente de la versión v4.

## <a name="to-listen-for-incoming-requests"></a>Para escuchar las solicitudes entrantes

### <a name="v3"></a>v3

```javascript
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

server.post('/api/messages', connector.listen());
```

### <a name="v4"></a>v4

```javascript
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        await bot.run(context);
    });
});
```

## <a name="to-create-a-bot-instance"></a>Para crear una instancia del bot

### <a name="v3"></a>v3

```javascript
var bot = new builder.UniversalBot(connector, [ ...DIALOG_STEPS ]);
```

### <a name="v4"></a>v4

```javascript
// Define the bot class as extending ActivityHandler.
const { ActivityHandler } = require('botbuilder');

class MyBot extends ActivityHandler {
    // ...
}

// Instantiate a new bot instance.
const bot = new MyBot(conversationState, dialog);
```

## <a name="to-send-a-message-to-a-user"></a>Para enviar un mensaje a un usuario

### <a name="v3"></a>v3

```javascript
session.send('Hello and welcome to the help desk bot.');
```

### <a name="v4"></a>v4

```javascript
await context.sendActivity('Hello and welcome to the help desk bot.');
```

## <a name="to-define-a-default-dialog"></a>Para definir un diálogo predeterminado

### <a name="v3"></a>v3

```javascript
var bot = new builder.UniversalBot(connector, [
    // Add default dialog waterfall steps...
]);
```

### <a name="v4"></a>v4

```javascript
// In the main dialog class, define a run method.
async run(turnContext, accessor) {
    const dialogSet = new DialogSet(accessor);
    dialogSet.add(this);

    const dialogContext = await dialogSet.createContext(turnContext);
    const results = await dialogContext.continueDialog();
    if (results.status === DialogTurnStatus.empty) {
        await dialogContext.beginDialog(this.id);
    }
}

// Pass conversation state management and a main dialog objects to the bot (in index.js).
const bot = new MyBot(conversationState, dialog);

// Inside the bot's constructor, add the dialog as a member property and define a DialogState property accessor.
this.dialog = dialog;
this.dialogState = this.conversationState.createProperty('DialogState');

// Inside the bot's message handler, invoke the dialog's run method, passing in the turn context and DialogState accessor.
this.onMessage(async (context, next) => {
    // Run the Dialog with the new message Activity.
    await this.dialog.run(context, this.dialogState);
    await next();
});
```

## <a name="to-start-a-child-dialog"></a>Para iniciar un diálogo secundario

El diálogo principal se reanuda después de que finalice el diálogo secundario.

### <a name="v3"></a>v3

```javascript
session.beginDialog(DIALOG_ID);
```

### <a name="v4"></a>v4

```javascript
await context.beginDialog(DIALOG_ID);
```

## <a name="to-replace-a-dialog"></a>Para reemplazar un diálogo

El diálogo actual se reemplaza en la pila por el diálogo nuevo.

### <a name="v3"></a>v3

```javascript
session.replaceDialog(DIALOG_ID);
```

### <a name="v4"></a>v4

```javascript
await context.replaceDialog(DIALOG_ID);
```

## <a name="to-end-a-dialog"></a>Para finalizar un diálogo

### <a name="v3"></a>v3

```javascript
session.endDialog();
```

### <a name="v4"></a>v4

Puede incluir un valor devuelto opcional.

```javascript
await context.endDialog(returnValue);
```

## <a name="to-register-a-dialog-with-a-bot-instance"></a>Para registrar un diálogo con una instancia de bot

### <a name="v3"></a>v3

```javascript
// Create the bot.
var bot = new builder.UniversalBot(connector, [
    // ...
]};

// Add the dialog to the bot.
bot.dialog('myDialog', require('./mydialog'));
```

### <a name="v4"></a>v4

```javascript
// In the main dialog class constructor.
this.addDialog(new MyDialog(DIALOG_ID));

// In the app entry point file (index.js)
const { MainDialog } = require('./dialogs/main');

// Create the base dialog and bot, passing the main dialog as an argument.
const dialog = new MainDialog();
const reservationBot = new ReservationBot(conversationState, dialog);
```

## <a name="to-prompt-a-user-for-input"></a>Para solicitar una entrada del usuario

### <a name="v3"></a>v3

```javascript
var builder = require('botbuilder');
builder.Prompts.text(session, 'Please enter your destination');
```

### <a name="v4"></a>v4

```javascript
const { TextPrompt } = require('botbuilder-dialogs');

// In the dialog's constructor, register the prompt.
this.addDialog(new TextPrompt('initialPrompt'));

// In the dialog step, invoke the prompt.
return await stepContext.prompt(
    'initialPrompt', {
        prompt: 'Please enter your destination'
    }
);
```

## <a name="to-retrieve-the-users-response-to-a-prompt"></a>Para recuperar la respuesta del usuario en una petición de consentimiento

### <a name="v3"></a>v3

```javascript
function (session, result) {
    var destination = result.response;
    // ...
}
```

### <a name="v4"></a>v4

```javascript
// In the dialog step after the prompt step, retrieve the result from the waterfall step context.
async nextStep(stepContext) {
    const destination = stepContext.result;
    // ...
}
```

## <a name="to-save-information-to-dialog-state"></a>Para guardar la información en el estado del diálogo

### <a name="v3"></a>v3

```javascript
session.dialogData.destination = results.response;
```

### <a name="v4"></a>v4

```javascript
// In a dialog, set the value in the waterfall step context.
stepContext.values.destination = destination;
```

## <a name="to-write-changes-in-state-to-the-persistance-layer"></a>Para escribir los cambios de estado en la capa de persistencia

### <a name="v3"></a>v3

```javascript
// State data is saved by default at the end of the turn.
// Do this to save it manually.
session.save();
```

### <a name="v4"></a>v4

```javascript
// State changes are not saved automatically at the end of the turn.
await this.conversationState.saveChanges(context, false);
await this.userState.saveChanges(context, false);
```

## <a name="to-create-and-register-state-storage"></a>Para crear y registrar el almacenamiento de estado

### <a name="v3"></a>v3

```javascript
var builder = require('botbuilder');

// Create conversation state with in-memory storage provider.
var inMemoryStorage = new builder.MemoryBotStorage();

// Create the base dialog and bot
var bot = new builder.UniversalBot(connector, [ /*...*/ ]).set('storage', inMemoryStorage);
```

### <a name="v4"></a>v4

```javascript
const { MemoryStorage } = require('botbuilder');

// Create the memory layer object.
const memoryStorage = new MemoryStorage();

// Create the conversation state management object, using the storage provider.
const conversationState = new ConversationState(memoryStorage);

// Create the base dialog and bot.
const dialog = new MainDialog();
const reservationBot = new ReservationBot(conversationState, dialog);
```

## <a name="to-catch-an-error-thrown-from-a-dialog"></a>Para detectar un error producido en un diálogo

### <a name="v3"></a>v3

```javascript
// Set up the error handler.
session.on('error', function (err) {
    session.send('Failed with message:', err.message);
    session.endDialog();
});

// Throw an error.
session.error('An error has occurred.');
```

### <a name="v4"></a>v4

```javascript
// Set up the error handler, using the adapter.
adapter.onTurnError = async (context, error) => {
    const errorMsg = error.message

    // Clear out conversation state to avoid keeping the conversation in a corrupted bot state.
    await conversationState.delete(context);

    // Send a message to the user.
    await context.sendActivity(errorMsg);
};

// Throw an error.
async () => {
    throw new Error('An error has occurred.');
}
```

## <a name="to-register-activity-handlers"></a>Para registrar los controladores de actividad

### <a name="v3"></a>v3

```javascript
bot.on('conversationUpdate', function (message) {
    // ...
}

bot.on('contactRelationUpdate', function (message) {
    // ...
}
```

### <a name="v4"></a>v4

```javascript
// In the bot's constructor, add the handlers.
this.onMessage(async (context, next) => {
    // ...
}
  
this.onDialog(async (context, next) => {
    // ...
}

this.onMembersAdded(async (context, next) => {
    // ...
}
```

## <a name="to-send-attachments"></a>Para enviar datos adjuntos

### <a name="v3"></a>v3

```javascript
var message = new builder.Message()
    .attachments(hotelHeroCards
    .attachmentLayout(builder.AttachmentLayout.carousel));
```

### <a name="v4"></a>v4

```javascript
await context.sendActivity({
    attachments: hotelHeroCards,
    attachmentLayout: AttachmentLayoutTypes.Carousel
});
```

## <a name="to-use-natural-language-recognition-luis"></a>Para usar el reconocimiento de lenguaje natural (LUIS)

### <a name="v3"></a>v3

```javascript
// The LUIS recognizer was part of the 'botbuilder' library
var builder = require('botbuilder');

var recognizer = new builder.LuisRecognizer(LUIS_MODEL_URL);
bot.recognizer(recognizer);
```

### <a name="v4"></a>v4

```javascript
// The LUIS recognizer is now part of the 'botbuilder-ai' library
const { LuisRecognizer } = require('botbuilder-ai');

const luisApp = { applicationId: LuisAppId, endpointKey: LuisAPIKey, endpoint: `https://${ LuisAPIHostName }` };
const recognizer = new LuisRecognizer(luisApp);

const recognizerResult = await recognizer.recognize(context);
const intent = LuisRecognizer.topIntent(recognizerResult);
```

## <a name="v3-intent-dialog-and-v4-equivalent"></a>Diálogo de intención de la versión v3 y equivalente de la versión v4

### <a name="v3"></a>v3

```javascript
// Create a 'greetings' RegExpRecognizer that can be turned off
var greetings = new builder.RegExpRecognizer('Greetings', /hello|hi|hey|greetings/i)
    .onEnabled(function (session, callback) {
        // Check to see if this recognizer should be enabled
        if (session.conversationData.useGreetings) {
            callback(null, true);
        } else {
            callback(null, false);
        }
    });

// Create our IntentDialog and add recognizers
var intents = new builder.IntentDialog({ recognizers: [greetings] });

bot.dialog('/', intents);

// If no intent is recognized, direct user to Recognizer Menu
intents.onDefault('RecognizerMenu');

// Match our "Greetings" and "Farewell" intents with their dialogs
intents.matches('Greetings', 'Greetings');

// Add a greetings dialog
bot.dialog('Greetings', [
    function (session) {
        session.endDialog('Greetings!');
    }
]);
```

### <a name="v4"></a>v4

```javascript
this.onMessage(async (context, next) => {

    const recognizerResult = {
        text: context.activity.text,
        intents: []
    };

    const greetingRegex = RegExp(/hello|hi|hey|greetings/i);

    if (greetingRegex.test(context.activity.text)) {
      // greeting intent identified
      recognizerResult.intents.push('Greeting');
    }

    if (recognizerResult.intents.includes('Greeting')) {
        // Run the 'Greeting' dialog
        await context.beginDialog(GREETING_DIALOG_ID);
    }

    await next();
});
```
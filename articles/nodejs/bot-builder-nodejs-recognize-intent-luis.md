---
title: Reconocimiento de intenciones y entidades con LUIS | Microsoft Docs
description: Integre un bot con LUIS para detectar la intención del usuario y responder según corresponda mediante el desencadenamiento de diálogos con Bot Framework SDK para Node.js.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/28/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: acdc6053f7d666c2f086dca554efafc93c8af769
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225290"
---
# <a name="recognize-intents-and-entities-with-luis"></a>Reconocimiento de intenciones y entidades con LUIS 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Este artículo usa el ejemplo de un bot para tomar notas, a fin de demostrar cómo Language Understanding ([LUIS][LUIS]) ayuda a que el bot responda correctamente a la entrada de lenguaje natural. Un bot detecta lo que un usuario desea hacer mediante la identificación de su **intención**. Esta intención se determina a partir de la entrada de texto, voz o **grabaciones de voz**. La intención asigna grabaciones de voz a acciones que realiza el bot, como invocar un diálogo. Puede que un bot también necesite extraer **entidades**, que son palabras importantes de las grabaciones de voz. A veces, las entidades deben cumplir una intención. En el ejemplo de un bot de toma de notas, la entidad `Notes.Title` identifica el título de cada nota.

## <a name="create-a-language-understanding-bot-with-bot-service"></a>Crear un bot de Language Understanding con Bot Service

1. En [Azure Portal](https://portal.azure.com), haga clic en **Crear nuevo recurso** en la hoja de menú y, después, en **Ver todo**.

    ![Crear recurso](../media/bot-builder-nodejs-use-luis/bot-service-creation.png)

2. En el cuadro de búsqueda, busque **Bot de aplicación web**. 

    ![Crear recurso](../media/bot-builder-nodejs-use-luis/bot-service-selection.png)

3. En la hoja **Servicio de bots**, proporcione la información necesaria y haga clic en **Crear**. Esto crea e implementa el servicio de bots y la aplicación de LUIS en Azure. 
   * Establezca **Nombre de la aplicación** en el nombre del bot. El nombre se usa como el subdominio cuando el bot se implementa en la nube (por ejemplo, mynotesbot.azurewebsites.net). Este nombre también se utiliza como el nombre de la aplicación de LUIS asociada con el bot. Cópielo para usarlo más adelante, para encontrar la aplicación de LUIS asociada con el bot.
   * Seleccione la suscripción, el [grupo de recursos](/azure/azure-resource-manager/resource-group-overview), el plan de App Service y la [ubicación](https://azure.microsoft.com/en-us/regions/).
   * Seleccione la plantilla **Language Understanding (Node.js)** en el campo **Bot template** (Plantilla de bot).

     ![Hoja Servicio de bots](../media/bot-builder-nodejs-use-luis/bot-service-setting-callout-template.png)

   * Active la casilla para confirmar los términos del servicio.

4. Confirme que se ha implementado el servicio de bots.
    * Haga clic en Notificaciones (el icono de la campana que se encuentra en el borde superior de Azure Portal). La notificación cambiará de **Implementación iniciada** a **Implementación correcta**.
    * Después de que la notificación cambie a **Implementación correcta**, haga clic en **Ir al recurso** en esa notificación.

## <a name="try-the-bot"></a>Probar el bot

Para confirmar que el bot se ha implementado, active las **Notificaciones**. Las notificaciones cambiarán de **Implementación en curso...** a **Implementación correcta**. Haga clic en el botón **Ir al recurso** para abrir la hoja de recursos del bot.

Una vez registrado el bot, haga clic en **Test in Web Chat** (Probar en Chat en web) para abrir el panel Chat en web. Escriba "hello" ("hola") en el Chat en web.

  ![Probar el bot en Chat en web](../media/bot-builder-nodejs-use-luis/bot-service-web-chat.png)

El bot responde "You have reached Greeting. You said: hello" ("Se ha puesto en contacto con Saludo. Ha dicho: hola"). Esto confirma que el bot ha recibido el mensaje y lo ha pasado a una aplicación de LUIS predeterminada que ha creado. Esta aplicación de LUIS predeterminada ha detectado una intención Saludo.

## <a name="modify-the-luis-app"></a>Modificación de la aplicación de LUIS

Inicie sesión en [https://www.luis.ai](https://www.luis.ai) con la misma cuenta que usa para iniciar sesión en Azure. Haga clic en **Mis aplicaciones**. En la lista de aplicaciones, busque la aplicación que comienza con el nombre especificado en **Nombre de la aplicación** en la hoja **Servicio de bots** cuando creó el servicio de bots. 

La aplicación de LUIS se inicia con 4 intenciones: Cancelar: Greeting, Help y None. <!-- picture -->

Los siguientes pasos agregan las intenciones Note.Create, Note.ReadAloud y Note.Delete: 

1. Haga clic en **Dominios creados previamente** en la parte inferior izquierda de la página. Busque el dominio **Nota** y haga clic en **Agregar dominio**.

2. En este tutorial no se usan todas las intenciones incluidas en el dominio **Nota** creado previamente. En la página **Intenciones**, haga clic en cada uno de los siguientes nombres de intenciones y luego haga clic en el botón **Eliminar intención**.
   * Note.ShowNext
   * Note.DeleteNoteItem
   * Note.Confirm
   * Note.Clear
   * Note.CheckOffItem
   * Note.AddToNote

   Las únicas intenciones que deben permanecer en la aplicación de LUIS son las siguientes: 
   * Note.ReadAloud
   * Note.Create
   * Note.Delete
   * None
   * Ayuda
   * Greeting
   * Cancelar

     ![intenciones mostradas en la aplicación de LUIS](../media/bot-builder-nodejs-use-luis/luis-intent-list.png)


3.  Haga clic en el botón **Entrenar** en la esquina superior derecha para entrenar la aplicación.
4.  Haga clic en **PUBLICAR** en la barra de navegación superior para abrir la página **Publicar**. Haga clic en el botón **Publish to production slot** (Publicar en el espacio de producción). Tras una publicación correcta, una aplicación de LUIS se implementa en la dirección URL mostrada en la columna **Punto de conexión** de la página **Publicar aplicación**, en la fila que empieza con el nombre de recurso Starter_Key. La dirección URL tiene un formato similar a este ejemplo: `https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx?subscription-key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&timezoneOffset=0&verbose=true&q=`. El identificador de la aplicación y la clave de suscripción de esta dirección URL son las mismas que LuisAppId y LuisAPIKey en ** Configuración de App Service > ApplicationSettings > Configuración de la aplicación **


## <a name="modify-the-bot-code"></a>Modificar el código del bot

Haga clic en **Compilar** y después en **Open online code editor** (Abrir el editor de código en línea).

   ![Abrir el editor de código en línea](../media/bot-builder-nodejs-use-luis/bot-service-build.png)

En el editor de código, abra `app.js`. Contiene el siguiente código:

```javascript
var restify = require('restify');
var builder = require('botbuilder');
var botbuilder_azure = require("botbuilder-azure");

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log('%s listening to %s', server.name, server.url); 
});
  
// Create chat connector for communicating with the Bot Framework Service
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword,
    openIdMetadata: process.env.BotOpenIdMetadata 
});

// Listen for messages from users 
server.post('/api/messages', connector.listen());

/*----------------------------------------------------------------------------------------
* Bot Storage: This is a great spot to register the private state storage for your bot. 
* We provide adapters for Azure Table, CosmosDb, SQL Azure, or you can implement your own!
* For samples and documentation, see: https://github.com/Microsoft/BotBuilder-Azure
* ---------------------------------------------------------------------------------------- */

var tableName = 'botdata';
var azureTableClient = new botbuilder_azure.AzureTableClient(tableName, process.env['AzureWebJobsStorage']);
var tableStorage = new botbuilder_azure.AzureBotStorage({ gzipData: false }, azureTableClient);

// Create your bot with a function to receive messages from the user
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send('You reached the default message handler. You said \'%s\'.', session.message.text);
});

bot.set('storage', tableStorage);

// Make sure you add code to validate these fields
var luisAppId = process.env.LuisAppId;
var luisAPIKey = process.env.LuisAPIKey;
var luisAPIHostName = process.env.LuisAPIHostName || 'westus.api.cognitive.microsoft.com';

const LuisModelUrl = 'https://' + luisAPIHostName + '/luis/v2.0/apps/' + luisAppId + '?subscription-key=' + luisAPIKey;

// Create a recognizer that gets intents from LUIS, and add it to the bot
var recognizer = new builder.LuisRecognizer(LuisModelUrl);
bot.recognizer(recognizer);

// Add a dialog for each intent that the LUIS app recognizes.
// See https://docs.microsoft.com/en-us/bot-framework/nodejs/bot-builder-nodejs-recognize-intent-luis 
bot.dialog('GreetingDialog',
    (session) => {
        session.send('You reached the Greeting intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Greeting'
})

bot.dialog('HelpDialog',
    (session) => {
        session.send('You reached the Help intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Help'
})

bot.dialog('CancelDialog',
    (session) => {
        session.send('You reached the Cancel intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Cancel'
})

```


> [!TIP] 
> También puede encontrar el código de ejemplo descrito en este artículo en el [ejemplo del bot de notas][NotesSample].



## <a name="edit-the-default-message-handler"></a>Edición del controlador de mensajes predeterminado
El bot tiene un controlador de mensajes predeterminado. Edítelo para que coincida con lo siguiente: 
```javascript
// Create your bot with a function to receive messages from the user.
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send("Hi... I'm the note bot sample. I can create new notes, read saved notes to you and delete notes.");

   // If the object for storing notes in session.userData doesn't exist yet, initialize it
   if (!session.userData.notes) {
       session.userData.notes = {};
       console.log("initializing userData.notes in default message handler");
   }
});
```

## <a name="handle-the-notecreate-intent"></a>Control de la intención Note.Create

Copie el código siguiente y péguelo en el extremo de app.js:

```javascript
// CreateNote dialog
bot.dialog('CreateNote', [
    function (session, args, next) {
        // Resolve and store any Note.Title entity passed from LUIS.
        var intent = args.intent;
        var title = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');

        var note = session.dialogData.note = {
          title: title ? title.entity : null,
        };
        
        // Prompt for title
        if (!note.title) {
            builder.Prompts.text(session, 'What would you like to call your note?');
        } else {
            next();
        }
    },
    function (session, results, next) {
        var note = session.dialogData.note;
        if (results.response) {
            note.title = results.response;
        }

        // Prompt for the text of the note
        if (!note.text) {
            builder.Prompts.text(session, 'What would you like to say in your note?');
        } else {
            next();
        }
    },
    function (session, results) {
        var note = session.dialogData.note;
        if (results.response) {
            note.text = results.response;
        }
        
        // If the object for storing notes in session.userData doesn't exist yet, initialize it
        if (!session.userData.notes) {
            session.userData.notes = {};
            console.log("initializing session.userData.notes in CreateNote dialog");
        }
        // Save notes in the notes object
        session.userData.notes[note.title] = note;

        // Send confirmation to user
        session.endDialog('Creating note named "%s" with text "%s"',
            note.title, note.text);
    }
]).triggerAction({ 
    matches: 'Note.Create',
    confirmPrompt: "This will cancel the creation of the note you started. Are you sure?" 
}).cancelAction('cancelCreateNote', "Note canceled.", {
    matches: /^(cancel|nevermind)/i,
    confirmPrompt: "Are you sure?"
});
```

Todas las entidades de la grabación de voz se pasan al diálogo mediante el parámetro `args`. El primer paso de la [cascada][waterfall] llama a [EntityRecognizer.findEntity][EntityRecognizer_findEntity] para obtener el título de la nota de cualquier entidad `Note.Title` de la respuesta de LUIS. Si la aplicación de LUIS no detectó una entidad `Note.Title`, el bot pide al usuario el nombre de la nota. El segundo paso de la cascada pide el texto que se va a incluir en la nota. Una vez que el bot tiene el texto de la nota, el tercer paso usa [session.userData][session_userData] para guardar la nota en un objeto `notes`, para lo que usa el título como clave. Para más información sobre `session.UserData`, vea [Administración de datos de estado](./bot-builder-nodejs-state.md). 



## <a name="handle-the-notedelete-intent"></a>Control de la intención Note.Delete
Igual que para la intención `Note.Create`, el bot examina el parámetro `args` para un título. Si no se detecta ningún título, el bot pregunta al usuario. El título se utiliza para buscar la nota para eliminarla de `session.userData.notes`. 



Copie el código siguiente y péguelo en el extremo de app.js:
```javascript
// Delete note dialog
bot.dialog('DeleteNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify that the title is in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to delete?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to delete.");
        }
    },
    function (session, results) {
        delete session.userData.notes[results.response.entity];        
        session.endDialog("Deleted the '%s' note.", results.response.entity);
    }
]).triggerAction({
    matches: 'Note.Delete'
}).cancelAction('cancelDeleteNote', "Ok - canceled note deletion.", {
    matches: /^(cancel|nevermind)/i
});
```

El código que controla `Note.Delete` usa la función `noteCount` para determinar si el objeto `notes` contiene las notas. 

Pegue la función de la aplicación auxiliar `noteCount` al final de `app.js`.

[!code-js[Add a helper function that returns the number of notes (JavaScript)](../includes/code/node-basicNote.js#CountNotesHelper)]

## <a name="handle-the-notereadaloud-intent"></a>Control de la intención Note.ReadAloud

Copie el código siguiente y péguelo en `app.js` después del controlador para `Note.Delete`:

```javascript
// Read note dialog
bot.dialog('ReadNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
           
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify it's in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to read?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to read.");
        }
    },
    function (session, results) {        
        session.endDialog("Here's the '%s' note: '%s'.", results.response.entity, session.userData.notes[results.response.entity].text);
    }
]).triggerAction({
    matches: 'Note.ReadAloud'
}).cancelAction('cancelReadNote', "Ok.", {
    matches: /^(cancel|nevermind)/i
});

```

El objeto `session.userData.notes` se pasa como el tercer argumento a `builder.Prompts.choice`, para que el mensaje muestre una lista de notas al usuario.

Ahora que ha agregado controladores para las nuevas intenciones, el código completo para `app.js` contiene lo siguiente:

```javascript
var restify = require('restify');
var builder = require('botbuilder');
var botbuilder_azure = require("botbuilder-azure");

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log('%s listening to %s', server.name, server.url); 
});
  
// Create chat connector for communicating with the Bot Framework Service
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword,
    openIdMetadata: process.env.BotOpenIdMetadata 
});

// Listen for messages from users 
server.post('/api/messages', connector.listen());

/*----------------------------------------------------------------------------------------
* Bot Storage: This is a great spot to register the private state storage for your bot. 
* We provide adapters for Azure Table, CosmosDb, SQL Azure, or you can implement your own!
* For samples and documentation, see: https://github.com/Microsoft/BotBuilder-Azure
* ---------------------------------------------------------------------------------------- */

var tableName = 'botdata';
var azureTableClient = new botbuilder_azure.AzureTableClient(tableName, process.env['AzureWebJobsStorage']);
var tableStorage = new botbuilder_azure.AzureBotStorage({ gzipData: false }, azureTableClient);

// Create your bot with a function to receive messages from the user.
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send("Hi... I'm the note bot sample. I can create new notes, read saved notes to you and delete notes.");

   // If the object for storing notes in session.userData doesn't exist yet, initialize it
   if (!session.userData.notes) {
       session.userData.notes = {};
       console.log("initializing userData.notes in default message handler");
   }
});

bot.set('storage', tableStorage);

// Make sure you add code to validate these fields
var luisAppId = process.env.LuisAppId;
var luisAPIKey = process.env.LuisAPIKey;
var luisAPIHostName = process.env.LuisAPIHostName || 'westus.api.cognitive.microsoft.com';

const LuisModelUrl = 'https://' + luisAPIHostName + '/luis/v2.0/apps/' + luisAppId + '?subscription-key=' + luisAPIKey;

// Create a recognizer that gets intents from LUIS, and add it to the bot
var recognizer = new builder.LuisRecognizer(LuisModelUrl);
bot.recognizer(recognizer);

// CreateNote dialog
bot.dialog('CreateNote', [
    function (session, args, next) {
        // Resolve and store any Note.Title entity passed from LUIS.
        var intent = args.intent;
        var title = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');

        var note = session.dialogData.note = {
          title: title ? title.entity : null,
        };
        
        // Prompt for title
        if (!note.title) {
            builder.Prompts.text(session, 'What would you like to call your note?');
        } else {
            next();
        }
    },
    function (session, results, next) {
        var note = session.dialogData.note;
        if (results.response) {
            note.title = results.response;
        }

        // Prompt for the text of the note
        if (!note.text) {
            builder.Prompts.text(session, 'What would you like to say in your note?');
        } else {
            next();
        }
    },
    function (session, results) {
        var note = session.dialogData.note;
        if (results.response) {
            note.text = results.response;
        }
        
        // If the object for storing notes in session.userData doesn't exist yet, initialize it
        if (!session.userData.notes) {
            session.userData.notes = {};
            console.log("initializing session.userData.notes in CreateNote dialog");
        }
        // Save notes in the notes object
        session.userData.notes[note.title] = note;

        // Send confirmation to user
        session.endDialog('Creating note named "%s" with text "%s"',
            note.title, note.text);
    }
]).triggerAction({ 
    matches: 'Note.Create',
    confirmPrompt: "This will cancel the creation of the note you started. Are you sure?" 
}).cancelAction('cancelCreateNote', "Note canceled.", {
    matches: /^(cancel|nevermind)/i,
    confirmPrompt: "Are you sure?"
});

// Delete note dialog
bot.dialog('DeleteNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify that the title is in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to delete?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to delete.");
        }
    },
    function (session, results) {
        delete session.userData.notes[results.response.entity];        
        session.endDialog("Deleted the '%s' note.", results.response.entity);
    }
]).triggerAction({
    matches: 'Note.Delete'
}).cancelAction('cancelDeleteNote', "Ok - canceled note deletion.", {
    matches: /^(cancel|nevermind)/i
});


// Read note dialog
bot.dialog('ReadNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
           
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify it's in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to read?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to read.");
        }
    },
    function (session, results) {        
        session.endDialog("Here's the '%s' note: '%s'.", results.response.entity, session.userData.notes[results.response.entity].text);
    }
]).triggerAction({
    matches: 'Note.ReadAloud'
}).cancelAction('cancelReadNote', "Ok.", {
    matches: /^(cancel|nevermind)/i
});


// Helper function to count the number of notes stored in session.userData.notes
function noteCount(notes) {

    var i = 0;
    for (var name in notes) {
        i++;
    }
    return i;
}
```

## <a name="test-the-bot"></a>Probar el bot

En Azure Portal, haga clic en **Test in Web Chat** (Probar en Chat en web) para probar el bot. Intente escribir mensajes como "Crear una nota", "Leer mis notas" y "Eliminar notas" para invocar las intenciones agregadas.
   ![Probar el bot de notas en Chat en web](../media/bot-builder-nodejs-use-luis/bot-service-test-notebot.png)

> [!TIP]
> Si ve que el bot no reconoce siempre la intención o las entidades correctas, proporcione a la aplicación de LUIS más ejemplos de expresiones para entrenarla y mejorar su rendimiento. Puede volver a entrenar la aplicación de LUIS sin modificar el código del bot. Vea [Add example utterances](/azure/cognitive-services/LUIS/add-example-utterances) (Agregar expresiones de ejemplo) y [Train and test your LUIS app](/azure/cognitive-services/LUIS/train-test) (Entrenar y probar la aplicación de LUIS).


## <a name="next-steps"></a>Pasos siguientes

Al probar el bot, puede ver que el reconocedor puede desencadenar la interrupción del diálogo activo actualmente. Permitir y controlar interrupciones es un diseño flexible que representa lo que el usuario hace realmente. Más información sobre las diversas acciones que puede asociar con una intención reconocida.

> [!div class="nextstepaction"]
> [Control de acciones del usuario](bot-builder-nodejs-dialog-actions.md)


[LUIS]: https://www.luis.ai/

[intentDialog]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.intentdialog.html

[intentDialog_matches]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.intentdialog.html#matches 

[NotesSample]: https://github.com/Microsoft/BotFramework-Samples/tree/master/docs-samples/Node/basics-naturalLanguage

[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[confirmPrompt]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#confirmprompt

[waterfall]: bot-builder-nodejs-dialog-manage-conversation-flow.md#manage-conversation-flow-with-a-waterfall

[session_userData]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata

[EntityRecognizer_findEntity]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#findentity

[matches]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#matches

[LUISAzureDocs]: /azure/cognitive-services/LUIS/Home

[Dialog]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html

[IntentRecognizerSetOptions]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html

[LuisRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer

[LUISConcepts]: https://docs.botframework.com/en-us/node/builder/guides/understanding-natural-language/

[DisambiguationSample]: https://aka.ms/v3-js-onDisambiguateRoute

[IDisambiguateRouteHandler]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idisambiguateroutehandler.html

[RegExpRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.regexprecognizer.html

[AlarmBot]: https://aka.ms/v3-js-luisSample

[UniversalBot]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html

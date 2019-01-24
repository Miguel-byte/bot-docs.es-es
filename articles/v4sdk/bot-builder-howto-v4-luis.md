---
title: Incorporación de reconocimiento de lenguaje natural al bot | Microsoft Docs
description: Obtenga información sobre cómo usar LUIS para el reconocimiento del lenguaje natural con Bot Framework SDK.
keywords: Language Understanding, LUIS, reconocimiento de intenciones, entidades, software intermedio
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 11/28/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4c43426f508d629c325889da6a9f7b06cac7e846
ms.sourcegitcommit: c6ce4c42fc56ce1e12b45358d2c747fb77eb74e2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/22/2019
ms.locfileid: "54453900"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>Incorporación de reconocimiento de lenguaje natural al bot

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

La tarea de entender qué quiere decir el usuario conversacionalmente y contextualmente puede ser difícil, pero hace que la conversación del bot parezca más natural. Language Understanding, denominado LUIS, le permite hacer precisamente esto, de modo que el bot pueda reconocer la intención de los mensajes de usuario, permitir al usuario emplear un lenguaje más natural y dirigir mejor el flujo de conversación. Este tema le guía por el proceso de configurar un bot sencillo que usa LUIS para reconocer algunas intenciones diferentes. 
## <a name="prerequisites"></a>Requisitos previos
- Cuenta de [luis.ai](https://www.luis.ai)
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)
- El código de este artículo se basa en el ejemplo de **NLP con LUIS**. Necesitará una copia del ejemplo en [C# ](https://aka.ms/cs-luis-sample) o en [JS](https://aka.ms/js-luis-sample). 
- Conocimientos sobre los [conceptos básicos de bots](bot-builder-basics.md), el [procesamiento de lenguaje natural](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/what-is-luis) y el archivo [.bot](bot-file-basics.md).

## <a name="create-a-luis-app-in-the-luis-portal"></a>Creación de una aplicación de LUIS en el portal de LUIS
Inicie sesión en el portal LUIS para crear su propia versión de la aplicación LUIS de ejemplo. Las aplicaciones se pueden crear y administrar en **My Apps** (Mis aplicaciones). 

1. Seleccione **Import new app** (Importar aplicación nueva). 
1. Haga clic en **Choose App file (JSON format)...** (Elegir archivo de aplicación [formato JSON]). 
1. Seleccione el archivo `reminders.json` que se encuentra en la carpeta `CognitiveModels` del ejemplo. En el campo de **nombre opcional**, escriba **LuisBot**. Este archivo contiene tres intenciones: Calendar_Add, Calendar_Find y None. Usaremos estas intenciones para entender qué quiso decir el usuario cuando envió un mensaje al bot. Si desea incluir entidades, consulte la [sección opcional](#optional---extract-entities) al final de este artículo.
1. [Entrene](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train) la aplicación.
1. [Publique](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp) la aplicación en el entorno de *producción*.

### <a name="obtain-values-to-connect-to-your-luis-app"></a>Obtención de valores para conectarse a la aplicación de LUIS

Una vez que la aplicación de LUIS se ha publicado, puede acceder a ella desde su bot. Para acceder a su aplicación de LUIS desde el bot, deberá registrar varios valores. Para recuperar esa información, puede usar el portal de LUIS.

#### <a name="retrieve-application-information-from-the-luisai-portal"></a>Recuperación de información de la aplicación desde el portal de LUIS.ai
El archivo .bot actúa como el lugar para reunir todas las referencias de servicio en un solo lugar. La información que recupere se agregará al archivo .bot en la siguiente sección. 
1. Seleccione la aplicación de LUIS publicada en [luis.ai](https://www.luis.ai).
1. Con la aplicación de LUIS publicada abierta, seleccione la pestaña **MANAGE** (ADMINISTRAR).
1. Seleccione la pestaña **Application Information** (Información de la aplicación) a la izquierda y registre el valor mostrado para _Application ID_ (Id. de aplicación) como <SU_ID_DE_APLICACIÓN>.
1. Seleccione la pestaña **Keys and Endpoints** (Claves y puntos de conexión) a la izquierda y registre el valor mostrado para _Authoring Key_ (Clave de creación) como <SU_CLAVE_DE_CREACIÓN>. Tenga en cuenta que *la clave de suscripción* es la misma que *la clave de creación*. 
1. Desplácese hasta el final de la página, registre el valor que se muestra para _región_ como <SU_REGIÓN>.
1. Registre el valor que se muestra en _Endpoint_ (Punto de conexión) como <SU_PUNTO_DE_CONEXIÓN>.

#### <a name="update-the-bot-file"></a>Actualización del archivo bot
Agregue la información necesaria para acceder a la aplicación de LUIS, como el identificador de la aplicación, la clave de creación, la clave de suscripción, el punto de conexión y la región al archivo `nlp-with-luis.bot`. Estos son los valores que guardó anteriormente de la aplicación de LUIS publicada.

```json
{
    "name": "LuisBot",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "166"
        },
        {
            "type": "luis",
            "name": "LuisBot",
            "appId": "<luis appid>",
            "version": "0.1",
            "authoringKey": "<luis authoring key>",
            "subscriptionKey": "<luis subscription key>",
            "region": "<luis region>",
            "id": "158"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```
# <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="configure-your-bot-to-use-your-luis-app"></a>Configuración del bot para usar la aplicación de LUIS

A continuación, se inicializa una nueva instancia de la clase BotService en `BotServices.cs`, que toma la información anterior de su archivo `.bot`. El servicio externo se configura mediante la clase `BotConfiguration`.

```csharp
public class BotServices
{
    // Initializes a new instance of the BotServices class
    public BotServices(BotConfiguration botConfiguration)
    {
        foreach (var service in botConfiguration.Services)
        {
            switch (service.Type)
            {
                case ServiceTypes.Luis:
                {
                    var luis = (LuisService)service;
                    if (luis == null)
                    {
                        throw new InvalidOperationException("The LUIS service is not configured correctly in your '.bot' file.");
                    }

                    var app = new LuisApplication(luis.AppId, luis.AuthoringKey, luis.GetEndpoint());
                    var recognizer = new LuisRecognizer(app);
                    this.LuisServices.Add(luis.Name, recognizer);
                    break;
                    }
                }
            }
        }

    // Gets the set of LUIS Services used. LuisServices is represented as a dictionary.  
    public Dictionary<string, LuisRecognizer> LuisServices { get; } = new Dictionary<string, LuisRecognizer>();
}
```

A continuación, se registra la aplicación de LUIS como un singleton en el archivo `Startup.cs` mediante el código siguiente dentro del método `ConfigureServices`.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\nlp-with-luis.bot", secretKey);
    services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // Initialize Bot Connected Services clients.
    var connectedServices = new BotServices(botConfig);
    services.AddSingleton(sp => connectedServices);
    services.AddSingleton(sp => botConfig);

    services.AddBot<LuisBot>(options =>
    {
        // Retrieve current endpoint.
        var environment = _isProduction ? "production" : "development";
        var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
        if (!(service is EndpointService endpointService))
        {
            throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
        }

        options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);

        // ...
    });
}
```

A continuación, en el archivo `LuisBot.cs`, el bot obtiene esta instancia de LUIS.

```csharp
public class LuisBot : IBot
{
    public static readonly string LuisKey = "LuisBot";
    private const string WelcomeText = "This bot will introduce you to natural language processing with LUIS. Type an utterance to get started";

    // Services configured from the ".bot" file.
    private readonly BotServices _services;

    // Initializes a new instance of the LuisBot class.
    public LuisBot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        if (!_services.LuisServices.ContainsKey(LuisKey))
        {
            throw new System.ArgumentException($"Invalid configuration....");
        }
    }
    // ...
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

En nuestro ejemplo, el código de inicio está en un archivo **index.js**, el código para la lógica del bot se muestra en un archivo **bot.js** y la información de configuración adicional, en **nlp-with-luis.bot**.

En el archivo **bot.js**, se lee la información de configuración para generar el servicio LUIS e inicializar el bot.
Actualice el valor de `LUIS_CONFIGURATION` con el nombre de la aplicación de LUIS, tal y como aparece en el archivo de configuración.

```javascript
// Language Understanding (LUIS) service name as defined in the .bot file.YOUR_LUIS_APP_NAME is "LuisBot" in the JavaScript code.
const LUIS_CONFIGURATION = '<YOUR_LUIS_APP_NAME>';

// Get endpoint and LUIS configurations by service name.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
const luisConfig = botConfig.findServiceByNameOrId(LUIS_CONFIGURATION);

// Map the contents to the required format for `LuisRecognizer`.
const luisApplication = {
    applicationId: luisConfig.appId,
    endpointKey: luisConfig.subscriptionKey || luisConfig.authoringKey,
    azureRegion: luisConfig.region
};

// Create configuration for LuisRecognizer's runtime behavior.
const luisPredictionOptions = {
    includeAllIntents: true,
    log: true,
    staging: false
};

// Create adapter...

// Create the LuisBot.
let bot;
try {
    bot = new LuisBot(luisApplication, luisPredictionOptions);
} catch (err) {
    console.error(`[botInitializationError]: ${ err }`);
    process.exit();
}
```

A continuación, se crea el servidor HTTP y se escuchan las solicitudes entrantes, lo cual genera llamadas a la lógica del bot.

```javascript
// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open nlp-with-luis.bot file in the emulator.`);
});

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async(turnContext) => {
        await bot.onTurn(turnContext);
    });
});
```

---

LUIS ya está configurado para el bot. A continuación, echemos un vistazo a cómo obtener la intención de LUIS.

### <a name="get-the-intent-by-calling-luis"></a>Obtención de la intención mediante una llamada a LUIS

El bot obtiene los resultados de LUIS mediante una llamada el reconocedor de LUIS.

# <a name="ctabcs"></a>[C#](#tab/cs)

Para que el bot simplemente envíe una respuesta basada en la intención que la aplicación de LUIS ha detectado, llame a `LuisRecognizer` para obtener un valor `RecognizerResult`. Esto puede realizarse en el código cada vez que necesite obtener la intención de LUIS.

```cs
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))

{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Check LUIS model
        var recognizerResult = await _services.LuisServices[LuisKey].RecognizeAsync(turnContext, cancellationToken);
        var topIntent = recognizerResult?.GetTopScoringIntent();
        if (topIntent != null && topIntent.HasValue && topIntent.Value.intent != "None")
        {
            await turnContext.SendActivityAsync($"==>LUIS Top Scoring Intent: {topIntent.Value.intent}, Score: {topIntent.Value.score}\n");
        }
        else
        {
            var msg = @"No LUIS intents were found.
                        This sample is about identifying two user intents:
                        'Calendar.Add'
                        'Calendar.Find'
                        Try typing 'Add Event' or 'Show me tomorrow'.";
            await turnContext.SendActivityAsync(msg);
        }
        }
        else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
        {
            // Send a welcome message to the user and tell them what actions they may perform to use this bot
            await SendWelcomeMessageAsync(turnContext, cancellationToken);
        }
        else
        {
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected", cancellationToken: cancellationToken);
        }
}
```

Todas las intenciones reconocidas en la expresión se devolverán como una asignación de nombres de intenciones a las puntuaciones, y se puede acceder a ellas desde `recognizerResult.Intents`. Se proporciona un método `recognizerResult?.GetTopScoringIntent()` estático para ayudar a simplificar la búsqueda de la intención con la puntuación más alta de un conjunto de resultados.

Todas las entidades reconocidas se devolverán como una asignación de nombres de entidades a valores, y se accederá a ellas mediante `recognizerResults.entities`. Es posible devolver metadatos de entidad adicionales si se pasa un valor `verbose=true` al crear la clase LuisRecognizer. Después, se puede acceder a los metadatos agregados mediante `recognizerResults.entities.$instance`.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

En el archivo **bot.js**, se pasa la entrada del usuario al método `recognize` del reconocedor de LUIS desde la aplicación de LUIS.

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

/**
 * A simple bot that responds to utterances with answers from the Language Understanding (LUIS) service.
 * If an answer is not found for an utterance, the bot responds with help.
 */
class LuisBot {
    /**
     * The LuisBot constructor requires one argument (`application`) which is used to create an instance of `LuisRecognizer`.
     * @param {LuisApplication} luisApplication The basic configuration needed to call LUIS. In this sample the configuration is retrieved from the .bot file.
     * @param {LuisPredictionOptions} luisPredictionOptions (Optional) Contains additional settings for configuring calls to LUIS.
     */
    constructor(application, luisPredictionOptions, includeApiResults) {
        this.luisRecognizer = new LuisRecognizer(application, luisPredictionOptions, true);
    }

    /**
     * Every conversation turn calls this method.
     * @param {TurnContext} turnContext Contains all the data needed for processing the conversation turn.
     */
    async onTurn(turnContext) {
        // By checking the incoming Activity type, the bot only calls LUIS in appropriate cases.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Perform a call to LUIS to retrieve results for the user's message.
            const results = await this.luisRecognizer.recognize(turnContext);

            // Since the LuisRecognizer was configured to include the raw results, get the `topScoringIntent` as specified by LUIS.
            const topIntent = results.luisResult.topScoringIntent;

            if (topIntent.intent !== 'None') {
                await turnContext.sendActivity(`LUIS Top Scoring Intent: ${ topIntent.intent }, Score: ${ topIntent.score }`);
            } else {
                // If the top scoring intent was "None" tell the user no valid intents were found and provide help.
                await turnContext.sendActivity(`No LUIS intents were found.
                                                \nThis sample is about identifying two user intents:
                                                \n - 'Calendar.Add'
                                                \n - 'Calendar.Find'
                                                \nTry typing 'Add Event' or 'Show me tomorrow'.`);
            }
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate &&
            turnContext.activity.recipient.id !== turnContext.activity.membersAdded[0].id) {
            // If the Activity is a ConversationUpdate, send a greeting message to the user.
            await turnContext.sendActivity('Welcome to the NLP with LUIS sample! Send me a message and I will try to predict your intent.');
        } else if (turnContext.activity.type !== ActivityTypes.ConversationUpdate) {
            // Respond to all other Activity types.
            await turnContext.sendActivity(`[${ turnContext.activity.type }]-type activity detected.`);
        }
    }
}

module.exports.LuisBot = LuisBot;
```

El reconocedor de LUIS devuelve información sobre el grado de coincidencia de la expresión con las intenciones disponibles. La propiedad `luisResult.intents` del objeto del resultado contiene una matriz de las intenciones puntuadas. La propiedad `luisResult.topScoringIntent` del objeto del resultado contiene la intención con la máxima puntuación y su puntuación.

---

### <a name="test-the-bot"></a>Probar el bot

1. Ejecute el ejemplo localmente en la máquina. Si necesita instrucciones, consulte el archivo Léame en el ejemplo de [C#](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/12.nlp-with-luis/README.md) o [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/12.nlp-with-luis/README.md).

1. En el emulador, escriba un mensaje como se muestra a continuación. 

![entrada de ejemplo de procesamiento de lenguaje natural de prueba](./media/nlp-luis-sample-message.png)

El bot responderá con la intención de máxima puntuación, que en este caso es la intención "Calendar_Add". Recuerde que el archivo `reminders.json` importado en el portal de luis.ai define las intenciones "Calendar_Add", "Calendar_Find", y "None". 

![respuesta del ejemplo de procesamiento de lenguaje natural de prueba](./media/nlp-luis-sample-response.png) 

Una puntuación de predicción indica el grado de confianza que tiene LUIS en los resultados de la predicción. Una puntuación de predicción se encuentra entre cero (0) y uno (1). Un ejemplo de una puntuación de LUIS de gran confianza es 0,99. Un ejemplo de una puntuación de confianza baja es 0,01. 

## <a name="optional---extract-entities"></a>Opcional: extracción de entidades

Además de reconocer la intención del usuario, una aplicación LUIS también puede devolver entidades. Las entidades son palabras importantes relacionadas con la intención y a veces pueden ser esenciales para satisfacer la solicitud de un usuario o permitir que el bot se comporte de forma más inteligente. 

### <a name="why-use-entities"></a>Por qué usar entidades

Las entidades de LUIS permiten que el bot entienda inteligentemente ciertas cosas o eventos que son diferentes a las intenciones estándar. Esto le permite recopilar información adicional del usuario, lo que le permite al bot responder de forma más inteligente o, posiblemente, saltarse ciertas preguntas en las que se le pide al usuario esa información. Por ejemplo, en un bot de meteorología, la aplicación LUIS puede utilizar una entidad _Ubicación_ para extraer la ubicación del informe meteorológico solicitado en el mensaje del usuario. Esto le permitiría al bot omitir la pregunta de dónde se encuentra el usuario y le proporcionaría una conversación más inteligente y concisa.

### <a name="prerequisites"></a>Requisitos previos

Para utilizar entidades con este ejemplo, necesitará crear una aplicación LUIS que incluya entidades. Siga los pasos de la sección anterior para [crear la aplicación LUIS](#create-a-luis-app-in-the-luis-portal), pero en lugar de utilizar el archivo `reminders.json`, utilice el archivo [reminders-with-entities.json](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/nlp-with-luis) para crear la aplicación LUIS. Este archivo proporciona las mismas intenciones además de tres entidades adicionales: Appointment, Meeting y Schedule. Estas entidades ayudan a LUIS a determinar la intención del mensaje del usuario. 

### <a name="extract-and-display-entities"></a>Extracción y muestra de entidades
El siguiente código opcional puede agregarse a esta aplicación de ejemplo para extraer y mostrar la información de la entidad cuando esta la utiliza LUIS para ayudar a identificar la intención del usuario. 

# <a name="ctabcs"></a>[C#](#tab/cs)

La siguiente función auxiliar se puede agregar al bot para obtener entidades del `RecognizerResult` de LUIS. Requerirá el uso de la biblioteca `Newtonsoft.Json.Linq`, que tendrá que agregar a sus instrucciones **using**. Si se encuentra información de entidad al analizar el JSON devuelto por LUIS, la función Newtonsoft _DeserializeObject_ convertirá este JSON en un objeto dinámico, al proporcionar acceso a la información de la entidad.

```cs
using Newtonsoft.Json.Linq;

private string ParseLuisForEntities(RecognizerResult recognizerResult)
{
   var result = string.Empty;

   // recognizerResult.Entities returns type JObject.
   foreach (var entity in recognizerResult.Entities)
   {
       // Parse JObject for a known entity types: Appointment, Meeting, and Schedule.
       var appointFound = JObject.Parse(entity.Value.ToString())["Appointment"];
       var meetingFound = JObject.Parse(entity.Value.ToString())["Meeting"];
       var schedFound = JObject.Parse(entity.Value.ToString())["Schedule"];

       // We will return info on the first entity found.
       if (appointFound != null)
       {
           // use JsonConvert to convert entity.Value to a dynamic object.
           dynamic o = JsonConvert.DeserializeObject<dynamic>(entity.Value.ToString());
           if (o.Appointment[0] != null)
           {
              // Find and return the entity type and score.
              var entType = o.Appointment[0].type;
              var entScore = o.Appointment[0].score;
              result = "Entity: " + entType + ", Score: " + entScore + ".";
              
              return result;
            }
        }

        if (meetingFound != null)
        {
            // use JsonConvert to convert entity.Value to a dynamic object.
            dynamic o = JsonConvert.DeserializeObject<dynamic>(entity.Value.ToString());
            if (o.Meeting[0] != null)
            {
                // Find and return the entity type and score.
                var entType = o.Meeting[0].type;
                var entScore = o.Meeting[0].score;
                result = "Entity: " + entType + ", Score: " + entScore + ".";
                
                return result;
            }
        }

        if (schedFound != null)
        {
            // use JsonConvert to convert entity.Value to a dynamic object.
            dynamic o = JsonConvert.DeserializeObject<dynamic>(entity.Value.ToString());
            if (o.Schedule[0] != null)
            {
                // Find and return the entity type and score.
                var entType = o.Schedule[0].type;
                var entScore = o.Schedule[0].score;
                result = "Entity: " + entType + ", Score: " + entScore + ".";
                
                return result;
            }
        }
    }

    // No entities were found, empty string returned.
    return result;
}
```

Esta información de la entidad detectada se puede mostrar junto con la intención del usuario identificado. Para mostrar esta información, agregue las siguientes líneas de código a la tarea _OnTurnAsync_ del código de ejemplo, justo después de que se haya mostrado la información de la intención.

```cs
// See if LUIS found and used an entity to determine user intent.
var entityFound = ParseLuisForEntities(recognizerResult);

// Inform the user if LUIS used an entity.
if (entityFound.ToString() != string.Empty)
{
   await turnContext.SendActivityAsync($"==>LUIS Entity Found: {entityFound}\n");
}
else
{
   await turnContext.SendActivityAsync($"==>No LUIS Entities Found.\n");
}
```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

El siguiente código se puede agregar al bot para extraer la información de la entidad del resultado `luisRecognizer` devuelto por LUIS. Dentro del procesamiento `onTurn` del archivo de ejemplo de código bot.js agregue la siguiente línea justo después de la declaración de constante _topIntent_. Esto capturará cualquier información de la entidad devuelta: 

```javascript
// Since the LuisRecognizer was configured to include the raw results, get returned entity data.
var entityData = results.luisResult.entities;

```

Para mostrar al usuario cualquier información de entidad devuelta, agregue las siguientes líneas de código justo después de la llamada _sendActivity_ que se utiliza dentro del código de ejemplo para informar al usuario cuando se ha encontrado un topIntent.

```javascript
// See if LUIS found and used an entity to determine user intent.
if (entityData.length > 0)
{
   if ((entityData[0].type == "Appointment") || (entityData[0].type == "Meeting") || (entityData[0].type == "Schedule") )
   {
      // inform user if LUIS used an entity.
      await turnContext.sendActivity(`LUIS Entity Found: Entity: ${entityData[0].entity}, Score: ${entityData[0].score}.`);
   }
}
else{
       await turnContext.sendActivity(`No LUIS Entities Found.`);
}
```

Este código primero comprueba si LUIS ha devuelto alguna información de la entidad dentro del resultado devuelto y, si lo hizo, muestra información concerniente a la primera entidad detectada.

---

### <a name="test-bot-with-entities"></a>Bot de prueba con entidades

1. Para probar el bot que incluye entidades, ejecute el ejemplo localmente, tal como se explica [arriba](#test-the-bot).

1. En el emulador, escriba el mensaje que se muestra a continuación. 

![entrada de ejemplo de procesamiento de lenguaje natural de prueba](./media/nlp-luis-sample-message.png)

El bot ahora responde con la intención de puntuación más alta "Calendar_Add" más la entidad "Meetings" que usó LUIS para determinar la intención del usuario.

![respuesta del ejemplo de procesamiento de lenguaje natural de prueba](./media/nlp-luis-sample-entity-response.png) 

La detección de entidades puede ayudar a mejorar el rendimiento general del bot. Por ejemplo, detectar la entidad "Meeting" (mostrada arriba) podría permitir que la aplicación llame ahora a una función especializada diseñada para crear una nueva reunión en el calendario del usuario.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Uso de QnA Maker para responder a preguntas](./bot-builder-howto-qna.md)

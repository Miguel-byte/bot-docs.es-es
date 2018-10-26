---
title: Uso de LUIS para la comprensión del lenguaje | Microsoft Docs
description: Obtenga información sobre cómo usar LUIS para la comprensión del lenguaje natural con Bot Builder SDK.
keywords: Language Understanding, LUIS, reconocimiento de intenciones, entidades, software intermedio
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 10/12/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 78654c78282c0a8e73dd17a093d27800f9fc8cb1
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326512"
---
# <a name="using-luis-for-language-understanding"></a>Uso de LUIS para la comprensión del lenguaje

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

La tarea de entender qué quiere decir el usuario conversacionalmente y contextualmente puede ser difícil, pero hace que la conversación del bot parezca más natural. Language Understanding, denominado LUIS, le permite hacer precisamente esto, de modo que el bot pueda reconocer la intención de los mensajes de usuario, permitir al usuario emplear un lenguaje más natural y dirigir mejor el flujo de conversación. Si necesita más información sobre cómo se integra LUIS con un bot, vea [Language understanding for bots](./bot-builder-concept-LUIS.md) (Language Understanding para bots).

## <a name="prerequisites"></a>Requisitos previos
Este tema le guía por el proceso de configurar un bot sencillo que usa LUIS para reconocer algunas intenciones diferentes. El código de este artículo se basa en el ejemplo de NLP con LUIS en [C#](https://aka.ms/cs-luis-sample) y [JavaScript](https://aka.ms/js-luis-sample).

## <a name="create-a-luis-app-in-the-luis-portal"></a>Creación de una aplicación de LUIS en el portal de LUIS

En primer lugar, regístrese para obtener una cuenta en [luis.ai](https://www.luis.ai) y cree una aplicación de LUIS en el portal de LUIS mediante las instrucciones indicadas [aquí](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-how-to-start-new-app). Si quiere crear su propia versión de la aplicación de LUIS de ejemplo usada en este artículo, en el portal de LUIS [importe](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app) este archivo `LUIS.Reminders.json` ([C#](https://github.com/Microsoft/BotBuilder-Samples/blob/v4/samples/csharp_dotnetcore/12.nlp-with-luis/CognitiveModels/LUIS-Reminders.json) | [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/12.nlp-with-luis/cognitiveModels/reminders.json)) para compilar la aplicación de LUIS y, luego, [entrénela](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train) y [publíquela](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp).

### <a name="obtain-values-to-connect-to-your-luis-app"></a>Obtención de valores para conectarse a la aplicación de LUIS

Una vez que la aplicación de LUIS se ha publicado, puede acceder a ella desde su bot. Para acceder a su aplicación de LUIS desde el bot, deberá registrar varios valores. Para recuperar esa información, puede usar el portal de LUIS o las herramientas de la CLI.

#### <a name="using-luis-portal"></a>Uso del portal de LUIS
- Seleccione la aplicación de LUIS publicada en [luis.ai](https://www.luis.ai).
- Con la aplicación de LUIS publicada abierta, seleccione la pestaña **MANAGE** (ADMINISTRAR).
- Seleccione la pestaña **Application Information** (Información de la aplicación) a la izquierda y registre el valor mostrado para _Application ID_ (Id. de aplicación) como <SU_ID_DE_APLICACIÓN>.
- Seleccione la pestaña **Keys and Endpoints** (Claves y puntos de conexión) a la izquierda y registre el valor mostrado para _Authoring Key_ (Clave de creación) como <SU_CLAVE_DE_CREACIÓN>. Tenga en cuenta que <SU_CLAVE_DE_SUSCRIPCIÓN> es la misma que <SU_CLAVE_DE_CREACIÓN>. Desplácese hasta el final de la página, registre el valor que se muestra para _Region_ (Región) como <SU_REGIÓN> y registre el valor mostrado para _Endpoint_ (Punto de conexión) como <SU_PUNTO_DE_CONEXIÓN>.

#### <a name="using-cli-tools"></a>Mediante herramientas de la CLI

Puede usar las herramientas [luis](https://aka.ms/botbuilder-tools-luis) y [msbot](https://aka.ms/botbuilder-tools-msbot-readme) de la CLI de BotBuilder para obtener metadatos sobre la aplicación de LUIS y agregarlos al archivo **.bot**.

1. Abra un terminal o un símbolo del sistema y vaya al directorio raíz del proyecto del bot.
2. Asegúrese de que las herramientas `luis` y `msbot` estén instaladas.

    ```shell
    npm install luis msbot
    ```

3. Ejecute `luis init` para crear un archivo de recurso de LUIS (**.luisrc**). Cuando se le pida, proporcione su clave de creación y su región de LUIS. No es necesario que escriba su identificador de aplicación en este momento.
4. Ejecute el siguiente comando para descargar los metadatos y agregarlos al archivo de configuración del bot.
    Si ha cifrado el archivo de configuración, deberá proporcionar la clave secreta para actualizar el archivo.

    ```shell
    luis get application --appId <your-app-id> --msbot | msbot connect luis --stdin [--secret <YOUR-SECRET>]
    ```

## <a name="configure-your-bot-to-use-your-luis-app"></a>Configuración del bot para usar la aplicación de LUIS

Primero se agrega una referencia a la aplicación de LUIS al inicializar el bot. Luego, se invoca dentro de la lógica de nuestro bot.

### <a name="prerequisite"></a>Requisito previo

Antes de comenzar a crear el código, asegúrese de que tiene los paquetes necesarios para la aplicación de LUIS.

# <a name="ctabcs"></a>[C#](#tab/cs)

Agregue el siguiente [paquete NuGet](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui) al bot.

* `Microsoft.Bot.Builder.AI.Luis`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Las características de LUIS están en el paquete `botbuilder-ai`. Puede agregar este paquete al proyecto a través de npm:

```shell
npm install --save botbuilder-ai
```

---

# <a name="ctabcs"></a>[C#](#tab/cs)

Descargue y abra el [código de ejemplo de NLP LUIS](https://aka.ms/cs-luis-sample) que se encuentra aquí. Se modificará el código según sea necesario. 

Primero, agregue la información necesaria para acceder a la aplicación de LUIS, como el identificador de la aplicación, la clave de creación, la clave de suscripción, el punto de conexión y la región al archivo `BotConfiguration.bot`. Estos son los valores que guardó anteriormente de la aplicación de LUIS publicada.

```csharp
{
  "name": "LuisBot",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "appPassword": "",
      "id": "1"
    },
    {
      "type": "luis",
      "name": "LuisBot",
      "AppId": "<YOUR_APP_ID>",
      "SubscriptionKey": "<YOUR_SUBSCRIPTION_KEY>",
      "AuthoringKey": "<YOUR_AUTHORING_KEY>",
      "GetEndpoint": "<YOUR_ENDPOINT>",
      "Region": "<YOUR_REGION>"
    }
  ],
  "padlock": "",
  "version": "2.0"
}
```

A continuación, se inicializa una nueva instancia de la clase `BotServices.cs` de BotService, que toma la información anterior de su archivo `.bot`. Agregue el siguiente código al archivo `BotServices.cs`.

```csharp
public class BotServices
{
    /// Initializes a new instance of the BotServices class
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

    /// Gets the set of LUIS Services used.
    /// LuisServices is represented as a dictionary.  
    public Dictionary<string, LuisRecognizer> LuisServices { get; } = new Dictionary<string, LuisRecognizer>();
}
```

A continuación, se registra la aplicación de LUIS como un singleton en el archivo `Startup.cs` mediante la incorporación del código siguiente dentro de `ConfigureServices`.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
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

        // Creates a logger for the application to use.
        ILogger logger = _loggerFactory.CreateLogger<LuisBot>();

        // Catches any errors that occur during a conversation turn and logs them.
        options.OnTurnError = async (context, exception) =>
        {
            logger.LogError($"Exception caught : {exception}");
            await context.SendActivityAsync("Sorry, it looks like something went wrong.");
        };
        /// ...
    });
}
```

A continuación, tenemos que darle al bot esta instancia de LUIS. Abra `LuisBot.cs` y agregue el siguiente código en la parte superior del archivo.

```csharp
public class LuisBot : IBot
{
    public static readonly string LuisKey = "LuisBot";
    private const string WelcomeText = "This bot will introduce you to natural language processing with LUIS. Type an utterance to get started";

    /// Services configured from the ".bot" file.
    private readonly BotServices _services;

    /// Initializes a new instance of the LuisBot class.
    public LuisBot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        if (!_services.LuisServices.ContainsKey(LuisKey))
        {
            throw new System.ArgumentException($"Invalid configuration. Please check your '.bot' file for a LUIS service named '{LuisKey}'.");
        }
    }
    /// ...
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

En nuestro ejemplo, el código de inicio está en un archivo **index.js**, el código para la lógica del bot se muestra en un archivo **bot.js** y la información de configuración adicional, en **nlp-with-luis.bot**.

Después de seguir las instrucciones para crear una aplicación de LUIS y actualizar su archivo **.bot**, el archivo **nlp-with-luis.bot** debe incluir una entrada de servicio para la aplicación de LUIS.

```json
{
    "name": "YOUR_LUIS_APP_NAME",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "35"
        },
        {
            "type": "luis",
            "name": "YOUR_LUIS_APP_NAME",
            "appId": "<YOUR_APP_ID>",
            "version": "0.1",
            "authoringKey": "<YOUR_AUTHORING_KEY>",
            "subscriptionKey": "<YOUR_SUBSCRIPTION_KEY>>",
            "region": "<YOUR_REGION>",
            "id": "83"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

En el archivo **index.js**, se lee la información de configuración para generar el servicio LUIS e inicializar el bot.
Actualice el valor de `LUIS_CONFIGURATION` con el nombre de la aplicación de LUIS, tal y como aparece en el archivo de configuración.

```javascript
// Language Understanding (LUIS) service name as defined in the .bot file.YOUR_LUIS_APP_NAME is "LuisBot" in the C# code.
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

## <a name="get-the-intent-by-calling-luis"></a>Obtención de la intención mediante una llamada a LUIS

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

## <a name="extract-entities"></a>Extraer entidades

Además de reconocer la intención, una aplicación de LUIS puede extraer entidades (es decir, palabras importantes) para cumplir la solicitud de un usuario. Por ejemplo, para un bot del tiempo, la aplicación de LUIS podría extraer a partir mensaje del usuario la ubicación para proporcionar el informe meteorológico.

Una manera habitual de estructurar la conversación consiste en identificar todas las entidades del mensaje del usuario y solicitarle las entidades necesarias que no se encuentren. Después, los pasos siguientes controlan la respuesta a la solicitud.

<!--Snip
# [C#](#tab/cs)

Let's say the message from the user was "What's the weather in Seattle"? The [LuisRecognizer](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.luisrecognizer) gives you a [RecognizerResult](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult) with an [`Entities` property](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult#properties-) that has this structure:

```json
{
"$instance": {
    "Weather_Location": [
        {
            "startIndex": 22,
            "endIndex": 29,
            "text": "seattle",
            "score": 0.8073087
        }
    ]
},
"Weather_Location": [
        "seattle"
    ]
}
```

The following helper function can be added to your bot to get entities out of the `RecognizerResult` from LUIS. It will require the use of the `Newtonsoft.Json.Linq` library, which you'll have to add to your **using** statements.

```cs
// Get entities from LUIS result
private T GetEntity<T>(RecognizerResult luisResult, string entityKey)
{
    var data = luisResult.Entities as IDictionary<string, JToken>;
    if (data.TryGetValue(entityKey, out JToken value))
    {
        return value.First.Value<T>();
    }
    return default(T);
}
```

When gathering information like entities from multiple steps in a conversation, it can be helpful to save the information you need in your state. If an entity is found, it can be added to the appropriate state field. In your conversation if the current step already has the associated field completed, the step to prompt for that information can be skipped.

# [JavaScript](#tab/js)

Let's say the message from the user was "What's the weather in Seattle"? The [LuisRecognizer](https://docs.microsoft.com/en-us/javascript/api/botbuilder-ai/luisrecognizer) gives you a [RecognizerResult](https://docs.microsoft.com/en-us/javascript/api/botbuilder-core-extensions/recognizerresult) with an `entities` property that has this structure:

```json
{
"$instance": {
    "Weather_Location": [
        {
            "startIndex": 22,
            "endIndex": 29,
            "text": "seattle",
            "score": 0.8073087
        }
    ]
},
"Weather_Location": [
        "seattle"
    ]
}
```

This `findEntities` function looks for any entities recognized by the LUIS app that match the incoming `entityName`.

```javascript
// Helper function for finding a specified entity
// entityResults are the results from LuisRecognizer.get(context)
function findEntities(entityName, entityResults) {
    let entities = []
    if (entityName in entityResults) {
        entityResults[entityName].forEach(entity => {
            entities.push(entity);
        });
    }
    return entities.length > 0 ? entities : undefined;
}
```
/Snip-->

Al recopilar información como entidades a partir de varios pasos de una conversación, puede ser útil guardar la información que necesita en su estado. Si se encuentra una entidad, se puede agregar al campo de estado adecuado. En la conversación, si el paso actual ya tiene el campo asociado completado, se puede omitir el paso para solicitar esa información.

## <a name="additional-resources"></a>Recursos adicionales

Para obtener un ejemplo de uso de LUIS, consulte los proyectos para [[C#](https://aka.ms/cs-luis-sample)] o [[JavaScript](https://aka.ms/js-luis-sample)].

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Combinación de LUIS y servicios QnA con la herramienta de distribución](./bot-builder-tutorial-dispatch.md)

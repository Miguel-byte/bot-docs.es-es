---
title: Actividades del bot en Bot Builder SDK | Microsoft Docs
description: Describe cómo funcionan las actividades y HTTP en Bot Builder SDK.
keywords: flujo de conversación, turno, conversación del bot, diálogos, avisos, cascadas, conjunto de diálogos
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 9/26/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: fde88929c688c25d473ce8242ebfd5d44dc3a22f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998132"
---
# <a name="understanding-how-bots-work"></a>Cómo funcionan los bots

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Un bot es una aplicación con la que los usuarios interactúan de forma conversacional mediante texto, gráficos (tarjetas o imágenes) o voz. Cada interacción entre el usuario y el bot genera una *actividad*. Bot Service envía información entre la aplicación del usuario conectada al bot (como Facebook, Skype o Slack, a lo que llamamos el *canal*) y el bot. Cada canal puede incluir información adicional en las actividades que envían. Antes de crear bots, es importante entender cómo utiliza el bot los objetos de actividad para comunicarse con los usuarios. En primer lugar, echemos un vistazo a las actividades que se intercambian cuando ejecutamos un bot de eco sencillo.

![Diagrama de actividades](media/bot-builder-activity.png)

Se muestran dos tipos de actividad: *actualización de conversación* y *mensaje*.

Bot Framework Service puede enviar una actualización de conversación cuando una entidad se une a la conversación. Por ejemplo, al iniciar una conversación con Bot Framework Emulator, verá dos actividades de actualización de conversación (una cuando el usuario se une a la conversación y otra cuando se une el bot). Para distinguir entre estas actividades de actualización de conversación, compruebe si la propiedad *miembros agregados* incluye un miembro que no sea el bot. 

La actividad de mensaje lleva información de la conversación entre las partes. En un ejemplo de bot de eco, las actividades de mensaje llevan texto simple y el canal representará este texto. Como alternativa, la actividad de mensaje podría llevar texto hablado, acciones sugeridas o tarjetas para mostrar.

En este ejemplo, el bot crea y envía una actividad de mensaje como respuesta a la actividad de mensaje entrante que ha recibido. Sin embargo, un bot puede responder de otras maneras a una actividad de mensaje recibida; no es raro que un bot responda a una actividad de actualización de conversación enviando algún texto de bienvenida en una actividad de mensaje. Puede encontrar más información en [dar la bienvenida al usuario](bot-builder-welcome-user.md).

### <a name="http-details"></a>Detalles HTTP

Las actividades llegan al bot desde Bot Framework Service mediante una solicitud POST HTTP. El bot responde a la solicitud POST entrante con un código de estado HTTP 200. Las actividades que se envían desde el bot al canal se envían en una solicitud POST HTTP independiente a Bot Framework Service. Esta se confirma de vuelta con un código de estado HTTP 200.

El protocolo no especifica el orden en el que se realizan estas solicitudes POST y sus confirmaciones. Sin embargo, para ajustarse a los marcos de servicios HTTP comunes, normalmente estas solicitudes se anidan, lo que significa que el bot realiza la solicitud HTTP de salida en el ámbito de la solicitud HTTP de entrada. Este proceso se ilustra en el diagrama anterior. Puesto que hay dos conexiones HTTP distintas consecutivas, se debe proporcionar un modelo de seguridad para ambas.

### <a name="defining-a-turn"></a>Definir un turno

En lo que respecta a los bots, un turno se usa para describir todo el procesamiento asociado con la llegada de una actividad. 

El objeto de *contexto de turno* proporciona información acerca de la actividad, como el remitente y receptor, el canal y otros datos necesarios para procesar la actividad. También permite la adición de información durante el turno en las distintas capas del bot.

El contexto de turno es una de las abstracciones más importantes en el SDK. No solo lleva la actividad de entrada a todos los componentes de middleware y la lógica de la aplicación, sino que también proporciona el mecanismo mediante el cual los componentes de middleware y la lógica de la aplicación pueden enviar actividades de salida.

## <a name="the-activity-processing-stack"></a>Pila de procesamiento de actividades

Vamos a profundizar en el diagrama anterior con el foco puesto en la llegada de una actividad de mensaje.

![Pila de procesamiento de actividades](media/bot-builder-activity-processing-stack.png)

En el ejemplo anterior, el bot respondió a la actividad de mensaje con otra actividad de mensaje que contenía el mismo mensaje de texto. El procesamiento comienza con la solicitud POST HTTP, con la información de la actividad en forma de una carga JSON que llega al servidor web. En C#, normalmente será un proyecto de ASP.NET y en un proyecto JavaScript de Node.js es probable que sea uno de los marcos más populares, como Express o Restify.

El *adaptador*, un componente integrado del SDK, actúa como el director del marco. El servicio usa la información de la actividad para crear un objeto de actividad y, a continuación, llama al método *procesar actividad* del adaptador y pasa el objeto de actividad y la información de autenticación (esta llamada se encapsula en las bibliotecas para C#, pero la verá en JavaScript). Tras la recepción de la actividad, el adaptador crea un objeto de contexto de turno y llama al [middleware](#middleware). Después del middleware, el procesamiento continúa hacia la lógica del bot, la canalización finaliza y el adaptador elimina el objeto de contexto de turno.

El *controlador de turnos* del bot, que lleva a cabo la mayor parte de la lógica de la aplicación, toma un contexto de turno como argumento. El controlador de turnos suele procesar el contenido de la actividad de entrada y generar una o varias actividades en respuesta, que envía mediante el método *enviar actividad* del contexto de turno. Una llamada al método enviar actividad enviará una actividad al canal del usuario, a menos que el procesamiento se interrumpa. La actividad incorporará todos los [controladores de eventos](#response-event-handlers) registrados antes del envío al canal.

## <a name="middleware"></a>Software intermedio

El middleware es un conjunto lineal de componentes que se agregan y ejecutan en orden y que tienen la posibilidad de operar sobre la actividad antes y después del controlador de turnos, además de tener acceso al contexto de turno de esa actividad. A menos que el middleware [sufra un cortocircuito](~/v4sdk/bot-builder-concept-middleware.md#short-circuiting), la fase final de la canalización de middleware es una devolución de llamada para invocar al controlador de turnos del bot antes de volver a la pila. Para información más detallada sobre el middleware, consulte el [tema sobre el middleware](~/v4sdk/bot-builder-concept-middleware.md).

## <a name="generating-responses"></a>Generación de respuestas

El contexto de turno proporciona métodos de respuesta de actividad para permitir que el código responda a una actividad:

* Los métodos _send activity_ y _send activities_ envían una o más actividades a la conversación.
* Si el canal lo admite, el método _update activity_ actualiza una actividad de la conversación.
* Si el canal lo admite, el método _delete activity_ elimina una actividad de la conversación.

Cada método de respuesta se ejecuta en un proceso asincrónico. Cuando se llama al método de respuesta de actividad, este clona la lista de [controladores de eventos](#response-event-handlers) asociados antes de empezar a llamar a los controladores, lo que significa que contendrá todos los controladores agregados hasta ese momento, pero no contendrá nada de lo que se agregue una vez iniciado el proceso.

Esto también significa que no se garantiza el orden de las respuestas a las distintas llamadas de actividad, en particular cuando una tarea es más compleja que otra. Si el bot puede generar varias respuestas para una actividad entrante, asegúrese de que tienen sentido en cualquier orden en el que las reciba el usuario. La única excepción a esto es el método *enviar actividades*, que le permite enviar un conjunto ordenado de actividades.

> [!IMPORTANT]
> El subproceso que administra el turno de bot principal se ocupa de desechar el objeto de contexto cuando termina. **Asegúrese de usar `await` para las llamadas de actividad**, a fin de que el subproceso principal espere la actividad generada antes de finalizar su procesamiento y desechar el contexto de turno. De otro modo, si una respuesta (incluidos sus controladores) tarda mucho e intenta actuar sobre el objeto de contexto, es posible que reciba un error `Context was disposed`. 

## <a name="response-event-handlers"></a>Controladores de eventos de respuesta

Además de la lógica de la aplicación y del middleware, se pueden agregar controladores de respuesta (a veces llamados controladores de eventos o controladores de eventos de actividad) al objeto de contexto. Se llama a estos controladores cuando la [respuesta](#generating-responses) asociada se produce en el objeto de contexto actual, antes de ejecutar la respuesta real. Estos controladores son útiles si sabe que querrá hacer algo, ya sea antes o después del evento real, para todas las actividades de ese tipo durante el resto de la respuesta actual.

> [!WARNING]
> Tenga cuidado de no llamar a un método de respuesta de actividad desde su controlador de eventos de respuesta correspondiente, por ejemplo, mediante una llamada al método de actividad de envío desde un controlador _on send activity_. Si lo hace, puede generar un bucle infinito.

Recuerde que cada actividad nueva obtiene un nuevo subproceso en el que se ejecuta. Cuando se crea el subproceso para procesar la actividad, la lista de controladores de esa actividad se copia en ese subproceso nuevo. No se ejecutará para ese evento de actividad específico ningún controlador agregado después de ese punto.

El adaptador administra los controladores registrados en un objeto de contexto de forma muy similar al modo en que administra la [canalización de middleware](~/v4sdk/bot-builder-concept-middleware.md#the-bot-middleware-pipeline). Es decir, se llama a los controladores en el orden en que se agregan, y al llamar al delegado _next_ se pasa el control al siguiente controlador de eventos registrado. Si un controlador no llama al delegado next, no se llama a ninguno de los controladores de eventos posteriores; se produce un [cortocircuito](~/v4sdk/bot-builder-concept-middleware.md#short-circuiting) en el evento y el adaptador no envía la respuesta al canal.

## <a name="bot-structure"></a>Estructura del bot

Echemos un vistazo al ejemplo de bot de eco con contador [[C#](https://aka.ms/EchoBotWithStateCSharp) | [JS](https://aka.ms/EchoBotWithStateJS)] para examinar los elementos principales del bot.

# <a name="ctabcs"></a>[C#](#tab/cs)

Un bot es un tipo de aplicación web de [ASP.NET Core](https://docs.microsoft.com/aspnet/core/?view=aspnetcore-2.1). Si consulta los conceptos básicos de [ASP.NET](https://docs.microsoft.com/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x), verá código similar en archivos como Program.cs y Startup.cs. Estos archivos son necesarios para todas las aplicaciones web y no son específicos del bot. El código de algunos de estos archivos no se tratará aquí, pero puede consultar el ejemplo de bot de eco con contador.

### <a name="echowithcounterbotcs"></a>EchoWithCounterBot.cs

La lógica principal del bot se define en la clase `EchoWithCounterBot` que deriva de la interfaz `IBot`. `IBot` define un único método `OnTurnAsync`. La aplicación debe implementar este método. `OnTurnAsync` tiene un objeto turnContext que proporciona información acerca de la actividad de entrada. La actividad de entrada se corresponde a la solicitud HTTP entrante. Las actividades pueden ser de diversos tipos, por lo que primero comprobamos si el bot ha recibido un mensaje. Si es un mensaje, se obtiene el estado de la conversación del contexto de turno, se incrementa el contador de turnos y, a continuación, se conserva el nuevo valor del contador de turnos en el estado de la conversación. Y, a continuación, se envía un mensaje al usuario mediante la llamada a SendActivityAsync. La actividad saliente corresponde a la solicitud HTTP saliente.

```cs
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context.
        var oldState = await _accessors.CounterState.GetAsync(turnContext, () => new CounterState());

        // Bump the turn count for this conversation.
        var newState = new CounterState { TurnCount = oldState.TurnCount + 1 };

        // Set the property using the accessor.
        await _accessors.CounterState.SetAsync(turnContext, newState);

        // Save the new turn count into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        // Echo back to the user whatever they typed.
        var responseMessage = $"Turn {newState.TurnCount}: You sent '{turnContext.Activity.Text}'\n";
        await turnContext.SendActivityAsync(responseMessage);
    }
    else
    {
        await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
    }
}
```

### <a name="startupcs"></a>Startup.cs

El método `ConfigureServices` carga los servicios conectados del archivo [.bot](bot-builder-basics.md#the-bot-file), detecta los errores que se producen durante un turno de conversación y los registra, configura el proveedor de credenciales y crea un objeto de estado de la conversación para almacenar los datos de la conversación en la memoria.

```csharp
services.AddBot<EchoWithCounterBot>(options =>
{
    // Creates a logger for the application to use.
    ILogger logger = _loggerFactory.CreateLogger<EchoWithCounterBot>();

    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    BotConfiguration botConfig = null;
    try
    {
        botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
    }
    catch
    {
        //...
    }

    services.AddSingleton(sp => botConfig);

    // Retrieve current endpoint.
    var environment = _isProduction ? "production" : "development";
    var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
    if (!(service is EndpointService endpointService))
    {
        throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
    }

    options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);

    // Catches any errors that occur during a conversation turn and logs them.
    options.OnTurnError = async (context, exception) =>
    {
        logger.LogError($"Exception caught : {exception}");
        await context.SendActivityAsync("Sorry, it looks like something went wrong.");
    };

    // The Memory Storage used here is for local bot debugging only. When the bot
    // is restarted, everything stored in memory will be gone.
    IStorage dataStore = new MemoryStorage();

    // ...

    // Create Conversation State object.
    // The Conversation State object is where we persist anything at the conversation-scope.
    var conversationState = new ConversationState(dataStore);

    options.State.Add(conversationState);
});
```

También crea y registra `EchoBotAccessors`, que se definen en el archivo **EchoBotStateAccessors.cs** y se pasan en el constructor `EchoWithCounterBot` público mediante el marco de inserción de dependencias de ASP.NET Core.

```csharp
// Create and register state accessors.
// Accessors created here are passed into the IBot-derived class on every turn.
services.AddSingleton<EchoBotAccessors>(sp =>
{
    var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
    // ...
    var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
    // ...

    // Create the custom state accessor.
    // State accessors enable other components to read and write individual properties of state.
    var accessors = new EchoBotAccessors(conversationState)
    {
        CounterState = conversationState.CreateProperty<CounterState>(EchoBotAccessors.CounterStateName),
    };

    return accessors;
});
```

El método `Configure` finaliza la configuración de la aplicación, ya que especifica que la aplicación usa Bot Framework y algunos otros archivos. Todos los bots que utilizan Bot Framework necesitarán esa llamada de configuración. El entorno de ejecución llama a `ConfigureServices` y `Configure` cuando se inicia la aplicación.

### <a name="counterstatecs"></a>CounterState.cs

Este archivo contiene una clase simple que el bot utiliza para mantener el estado actual. Contiene solo un valor `int` que se usa para incrementar el contador.

```cs
public class CounterState
{
    public int TurnCount { get; set; } = 0;
}
```

### <a name="echobotaccessorscs"></a>EchoBotAccessors.cs

La clase `EchoBotAccessors` se crea como singleton en la clase `Startup` y se pasa a la clase derivada IBot. En este caso, `public class EchoWithCounterBot : IBot`. El bot usa el descriptor de acceso para conservar los datos de la conversación. El constructor de `EchoBotAccessors` se pasa en un objeto de conversación que se crea en el archivo Startup.cs.

```cs
public class EchoBotAccessors
{
    public EchoBotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string CounterStateName { get; } = $"{nameof(EchoBotAccessors)}.CounterState";

    public IStatePropertyAccessor<CounterState> CounterState { get; set; }

    public ConversationState ConversationState { get; }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

La sección del sistema contiene principalmente los archivos **package.json**, **.env**, **index.js** y **README.md**. El código de algunos de estos archivos no se copiará aquí, pero lo verá al ejecutar el bot.

### <a name="packagejson"></a>package.json

**package.json** especifica las dependencias del bot y sus versiones asociadas. Esto viene configurado por la plantilla y el sistema.

### <a name="env-file"></a>Archivo .env

El archivo **.env** especifica la información de configuración del bot, como el número de puerto, el identificador de aplicación y la contraseña, entre otras cosas. Si utiliza determinadas tecnologías o si usa este bot en producción, deberá agregar las claves o dirección URL específicas a esta configuración. Para este bot de eco, no obstante, no necesita hacer nada aquí ahora mismo; el identificador de aplicación y la contraseña se pueden dejar sin definir en este momento.

Para usar el archivo de configuración **.env** la plantilla necesita incluir un paquete adicional.  En primer lugar, obtenga el paquete `dotenv` de npm:

`npm install dotenv`

### <a name="indexjs"></a>index.js

El archivo `index.js` configura el bot y el servicio de hospedaje que reenviará las actividades a la lógica del bot.

#### <a name="required-libraries"></a>Bibliotecas necesarias

En la parte superior del archivo `index.js` encontrará una serie de módulos o bibliotecas que van a ser necesarios. Estos módulos le darán acceso a un conjunto de funciones que tal vez desee incluir en la aplicación.

```javascript
// Import required packages
const path = require('path');
const restify = require('restify');

// Import required bot services. See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter, ConversationState, MemoryStorage } = require('botbuilder');
// Import required bot configuration.
const { BotConfiguration } = require('botframework-config');

const { EchoBot } = require('./bot');

// Read botFilePath and botFileSecret from .env file
// Note: Ensure you have a .env file and include botFilePath and botFileSecret.
const ENV_FILE = path.join(__dirname, '.env');
const env = require('dotenv').config({ path: ENV_FILE });
```

#### <a name="bot-configuration"></a>Configuración del bot

La siguiente parte carga información del archivo de configuración de bot.

```javascript
// Get the .bot file path
// See https://aka.ms/about-bot-file to learn more about .bot file its use and bot configuration.
const BOT_FILE = path.join(__dirname, (process.env.botFilePath || ''));
let botConfig;
try {
    // Read bot configuration from .bot file.
    botConfig = BotConfiguration.loadSync(BOT_FILE, process.env.botFileSecret);
} catch (err) {
    console.error(`\nError reading bot file. Please ensure you have valid botFilePath and botFileSecret set for your environment.`);
    console.error(`\n - The botFileSecret is available under appsettings for your Azure Bot Service bot.`);
    console.error(`\n - If you are running this bot locally, consider adding a .env file with botFilePath and botFileSecret.`);
    console.error(`\n - See https://aka.ms/about-bot-file to learn more about .bot file its use and bot configuration.\n\n`);
    process.exit();
}

// For local development configuration as defined in .bot file
const DEV_ENVIRONMENT = 'development';

// Define name of the endpoint configuration section from the .bot file
const BOT_CONFIGURATION = (process.env.NODE_ENV || DEV_ENVIRONMENT);

// Get bot endpoint configuration by service name
// Bot configuration as defined in .bot file
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
```

#### <a name="bot-adapter-http-server-and-bot-state"></a>Adaptador del bot, servidor HTTP y estado del bot

Las partes siguientes configuran el servidor y el adaptador que permiten al bot comunicarse con el usuario y enviar respuestas. El servidor escuchará en el puerto especificado en el archivo de configuración **BotConfiguration.bot** o volverá al _3978_ para la conexión con el emulador. El adaptador actúa como el director del bot, dirige la comunicación entrante y saliente y la autenticación, entre otras.

También se crea un objeto de estado que usa `MemoryStorage` como proveedor de almacenamiento. Este estado se define como `ConversationState`, lo que significa simplemente que mantiene el estado de la conversación. `ConversationState` almacenará en memoria la información que le interesa, que en este caso es simplemente un contador de turnos.

```javascript
// Create bot adapter.
// See https://aka.ms/about-bot-adapter to learn more about bot adapter.
const adapter = new BotFrameworkAdapter({
    appId: endpointConfig.appId || process.env.microsoftAppID,
    appPassword: endpointConfig.appPassword || process.env.microsoftAppPassword
});

// Catch-all for any unhandled errors in your bot.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${ error }`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
    // Clear out state
    await conversationState.clear(context);
    // Save state changes.
    await conversationState.saveChanges(context);
};

// Define a state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state store to persist the dialog and user state between messages.
let conversationState;

// For local development, in-memory storage is used.
// CAUTION: The Memory Storage used here is for local bot debugging only. When the bot
// is restarted, anything stored in memory will be gone.
const memoryStorage = new MemoryStorage();
conversationState = new ConversationState(memoryStorage);

// Create the main dialog.
const bot = new EchoBot(conversationState);

// Create HTTP server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator`);
    console.log(`\nTo talk to your bot, open echoBot-with-counter.bot file in the Emulator`);
});
```

#### <a name="bot-logic"></a>Lógica del bot

El elemento `processActivity` del adaptador envía las actividades entrantes a la lógica del bot.
El tercer parámetro en `processActivity` es un controlador de función al que se llamará para realizar la lógica del bot después de que la [actividad](#the-activity-processing-stack) recibida haya sido previamente procesada por el adaptador y enrutada a algún middleware. La variable de contexto de turno, que se pasa como un argumento al controlador de función, se puede usar para proporcionar información sobre la actividad entrante, el remitente y el receptor, el canal o la conversación, entre otros. El procesamiento de la actividad se enruta al elemento `onTurn` del bot de eco.

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // Route to main dialog.
        await bot.onTurn(context);
    });
});
```

### <a name="echobot"></a>Bot de eco

Todo el procesamiento de la actividad se enruta al controlador `onTurn` de esta clase. Cuando se crea la clase, se pasa un objeto de estado. Con este objeto de estado, el constructor crea un descriptor de acceso `this.countProperty` para conservar el contador de turnos de este bot.

En primer lugar, se comprueba en cada turno si el bot ha recibido un mensaje. Si el bot no ha recibido un mensaje, se devuelve el tipo de actividad recibida. A continuación, creamos una variable de estado que contiene la información de la conversación del bot. Si la variable de contador es `undefined`, se establece en 1 (lo que ocurrirá cuando se inicia por primera vez el bot) o se incrementa con cada nuevo mensaje. Se devuelve al usuario el contador junto con el mensaje que envió. Por último, se establece el contador y se guardan los cambios en el estado.

```javascript
const { ActivityTypes } = require('botbuilder');

// Turn counter property
const TURN_COUNTER_PROPERTY = 'turnCounterProperty';

class EchoBot {

    constructor(conversationState) {
        // Creates a new state accessor property.
        // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors
        this.countProperty = conversationState.createProperty(TURN_COUNTER_PROPERTY);
        this.conversationState = conversationState;
    }

    async onTurn(turnContext) {
        // Handle message activity type. User's responses via text or speech or card interactions flow back to the bot as Message activity.
        // Message activities may contain text, speech, interactive cards, and binary or unknown attachments.
        // see https://aka.ms/about-bot-activity-message to learn more about the message and other activity types
        if (turnContext.activity.type === ActivityTypes.Message) {
            // read from state.
            let count = await this.countProperty.get(turnContext);
            count = count === undefined ? 1 : ++count;
            await turnContext.sendActivity(`${ count }: You said "${ turnContext.activity.text }"`);
            // increment and set turn counter.
            await this.countProperty.set(turnContext, count);
        } else {
            // Generic handler for all other activity types.
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
        // Save state changes
        await this.conversationState.saveChanges(turnContext);
    }
}

exports.EchoBot = EchoBot;
```

---

### <a name="the-bot-file"></a>Archivo .bot

El archivo **.bot** contiene información, incluidos el punto de conexión, el identificador de aplicación, la contraseña y las referencias a los servicios que el bot utiliza. Este archivo se crea automáticamente al iniciar la creación de un bot desde una plantilla, pero puede crear el suyo propio con el emulador u otras herramientas. Puede especificar el archivo .bot que se va a utilizar al probar el bot con el [emulador](../bot-service-debug-emulator.md).

```json
{
    "name": "echobot-with-counter",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "1"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

## <a name="additional-resources"></a>Recursos adicionales

Para más información acerca de la administración de estados, consulte [Administración del estado de la conversación y el usuario](bot-builder-howto-v4-state.md)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de un bot](~/bot-service-quickstart.md)

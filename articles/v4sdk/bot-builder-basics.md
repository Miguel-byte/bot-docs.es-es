---
title: Funcionamiento de los bots | Microsoft Docs
description: Describe cómo funcionan las actividades y HTTP en Bot Framework SDK.
keywords: flujo de conversación, turno, conversación del bot, diálogos, avisos, cascadas, conjunto de diálogos
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0e8a8275a7ede599b3d25576abcd3c1160873db7
ms.sourcegitcommit: eacf1522d648338eebefe2cc5686c1f7866ec6a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2019
ms.locfileid: "70167199"
---
# <a name="how-bots-work"></a>Funcionamiento de los bots

[!INCLUDE[applies-to](../includes/applies-to.md)]

Un bot es una aplicación con la que los usuarios interactúan de forma conversacional mediante texto, gráficos (tarjetas o imágenes) o voz. Cada interacción entre el usuario y el bot genera una *actividad*. Bot Framework Service, que es un componente de Azure Bot Service, envía información entre la aplicación del usuario conectada al bot (como Facebook, Skype o Slack, a lo que llamamos el *canal*) y el bot. Cada canal puede incluir información adicional en las actividades que envían. Antes de crear bots, es importante entender cómo utiliza el bot los objetos de actividad para comunicarse con los usuarios. En primer lugar, echemos un vistazo a las actividades que se intercambian cuando ejecutamos un bot de eco sencillo. 

![Diagrama de actividades](media/bot-builder-activity.png)

Se muestran dos tipos de actividad: *actualización de conversación* y *mensaje*.

Bot Framework Service puede enviar una actualización de conversación cuando una entidad se une a la conversación. Por ejemplo, al iniciar una conversación con Bot Framework Emulator, verá dos actividades de actualización de conversación (una cuando el usuario se une a la conversación y otra cuando se une el bot). Para distinguir entre estas actividades de actualización de conversación, compruebe si la propiedad *miembros agregados* incluye un miembro que no sea el bot. 

La actividad de mensaje lleva información de la conversación entre las partes. En un ejemplo de bot de eco, las actividades de mensaje llevan texto simple y el canal representará este texto. Como alternativa, la actividad de mensaje podría llevar texto hablado, acciones sugeridas o tarjetas para mostrar.

En este ejemplo, el bot crea y envía una actividad de mensaje como respuesta a la actividad de mensaje entrante que ha recibido. Sin embargo, un bot puede responder de otras maneras a una actividad de mensaje recibida; no es raro que un bot responda a una actividad de actualización de conversación enviando algún texto de bienvenida en una actividad de mensaje. Puede encontrar más información en [dar la bienvenida al usuario](bot-builder-welcome-user.md).

### <a name="http-details"></a>Detalles HTTP

Las actividades llegan al bot desde Bot Framework Service mediante una solicitud POST HTTP. El bot responde a la solicitud POST entrante con un código de estado HTTP 200. Las actividades que se envían desde el bot al canal se envían en una solicitud POST HTTP independiente a Bot Framework Service. Esta se confirma de vuelta con un código de estado HTTP 200.

El protocolo no especifica el orden en el que se realizan estas solicitudes POST y sus confirmaciones. Sin embargo, para ajustarse a los marcos de servicios HTTP comunes, normalmente estas solicitudes se anidan, lo que significa que el bot realiza la solicitud HTTP de salida en el ámbito de la solicitud HTTP de entrada. Este proceso se ilustra en el diagrama anterior. Puesto que hay dos conexiones HTTP distintas consecutivas, se debe proporcionar un modelo de seguridad para ambas.

### <a name="defining-a-turn"></a>Definir un turno

En una conversación, la gente a menudo habla de uno en uno, haciendo turnos para hablar. Con un bot, por lo general reacciona a las entradas del usuario. En Bot Framework SDK, un _turno_ consiste en la actividad de la entrada del usuario en el bot y en cualquier actividad que el bot devuelve al usuario como respuesta inmediata. Se puede pensar en un turno como el procesamiento asociado con la llegada de una actividad determinada.

El objeto de *contexto de turno* proporciona información acerca de la actividad, como el remitente y receptor, el canal y otros datos necesarios para procesar la actividad. También permite la adición de información durante el turno en las distintas capas del bot.

El contexto de turno es una de las abstracciones más importantes en el SDK. No solo lleva la actividad de entrada a todos los componentes de middleware y la lógica de la aplicación, sino que también proporciona el mecanismo mediante el cual los componentes de middleware y la lógica de la aplicación pueden enviar actividades de salida.

## <a name="the-activity-processing-stack"></a>Pila de procesamiento de actividades

Vamos a profundizar en el diagrama anterior con el foco puesto en la llegada de una actividad de mensaje.

![Pila de procesamiento de actividades](media/bot-builder-activity-processing-stack.png)

En el ejemplo anterior, el bot respondió a la actividad de mensaje con otra actividad de mensaje que contenía el mismo mensaje de texto. El procesamiento comienza con la solicitud POST HTTP, con la información de la actividad en forma de una carga JSON que llega al servidor web. En C#, normalmente será un proyecto de ASP.NET y en un proyecto JavaScript de Node.js es probable que sea uno de los marcos más populares, como Express o Restify.

El *adaptador*, un componente integrado del SDK, es el núcleo del entorno de ejecución del SDK. La actividad se realiza como JSON en el cuerpo de la solicitud HTTP POST. Este código JSON se deserializa para crear el objeto de actividad que, a continuación, se pasa al adaptador con una llamada al método *process activity*. Tras la recepción de la actividad, el adaptador crea un *contexto de turno* y llama al middleware. 

Como se ha indicado anteriormente, el contexto de turno proporciona el mecanismo para que el bot envíe las actividades de salida, muy frecuentemente en respuesta a una actividad de entrada. Para ello, el contexto de turno proporciona métodos de respuesta para _enviar, actualizar y eliminar una actividad_. Cada método de respuesta se ejecuta en un proceso asincrónico. 

[!INCLUDE [alert-await-send-activity](../includes/alert-await-send-activity.md)]

## <a name="activity-handlers"></a>Controladores de actividad

Cuando el bot recibe una actividad, la pasa a sus *controladores de actividad*. En segundo plano, hay un controlador base denominado *controlador de turnos*. Todas las actividades se enrutan a través de él. Luego, dicho controlador llama al controlador de la actividad individual sea cual sea la actividad que haya recibido.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Por ejemplo, si el bot recibe una actividad de mensaje, el controlador de turnos vería dicha actividad entrante y la enviaría al controlador de actividad `OnMessageActivityAsync`. 

Al crear el bot, la lógica del bot para controlar los mensajes y responderlos a mensajes se incluirá en dicho `OnMessageActivityAsync`controlador. Del mismo modo, la lógica para controlar la incorporación de miembros se incluirá en el controlador `OnMembersAddedAsync`, al que se llama cada vez que se agrega un miembro a la conversación.

Para implementar la lógica de estos controladores, deberá reemplazar estos métodos en el bot como se muestra en la sección [Lógica del bot](#bot-logic), que encontrará más adelante. En estos controladores no hay implementación base, por lo que se puede agregar la lógica que se desee en la invalidación.

Hay ciertas situaciones en las que puede desear invalidar el controlador de base, como por ejemplo cuando se [guarda el estado](bot-builder-concept-state.md) al final de un turno. Al hacerlo, asegúrese de llamar en primer lugar a `await base.OnTurnAsync(turnContext, cancellationToken);` para asegurarse de que la implementación base de `OnTurnAsync` se ejecuta antes que el código adicional. Dicha implementación base es, entre otras cosas, la encargada de llamar al resto de los controladores de actividad como `OnMessageActivityAsync`.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Por ejemplo, si el bot recibe una actividad de mensaje, el controlador de turnos vería dicha actividad entrante y la enviaría al controlador de actividad `onMessage`.

Al crear el bot, la lógica del bot para controlar los mensajes y responderlos a mensajes se incluirá en dicho `onMessage`controlador. Del mismo modo, la lógica para controlar la incorporación de miembros se incluirá en el controlador `onMembersAdded`, al que se llama cada vez que se agrega un miembro a la conversación.

Para implementar la lógica de estos controladores, deberá reemplazar estos métodos en el bot como se muestra en la sección [Lógica del bot](#bot-logic), que encontrará más adelante. Defina la lógica del bot de cada uno de estos controladores y, después, **no olvide llamar a `next()` al final**. Al llamar a `next()`, asegúrese de que se ejecuta el siguiente controlador.

No hay situaciones comunes en las que vaya a desear invalidar el controlador de turnos base, así que tenga cuidado si intenta hacerlo. Para operaciones, como [guardar el estado](bot-builder-concept-state.md), que desee realizar al final de un turno, hay un controlador especial llamado `onDialog`. El controlador `onDialog` se ejecuta al final, después de que se hayan ejecutado el resto de los controladores, y no está asociado a un tipo de actividad determinado. Como con todos los controladores anteriores, asegúrese de llamar a `next()` para asegurarse de que concluye el resto del proceso.

---

## <a name="middleware"></a>Software intermedio

El middleware es muy similar a cualquier otro middleware de mensajería, incluye un conjunto lineal de componentes que se ejecutan en orden, dando a cada uno la oportunidad de operar en la actividad. La etapa final de la canalización del middleware es una devolución de llamada al controlador de turnos de la clase del bot que la aplicación ha registrado con el método *process activity* del adaptador. Por lo general, el controlador de turnos está escrito `OnTurnAsync` en C# y `onTurn` en JavaScript.

El controlador de turnos toma un contexto de turno como argumento, normalmente la lógica de la aplicación que se ejecuta dentro de la función del controlador de turnos procesará el contenido de la actividad de entrada y generará una o varias actividades en respuesta, las cuales se envían con la función *send activity* en el contexto de turno. Una llamada a *send activity* en el contexto de turno hará que se invoque a los componentes de middleware en las actividades de salida. Los componentes de middleware se ejecutan antes y después de la función del controlador de turnos del bot. La ejecución es anidada de forma inherente y, por lo tanto, a veces se la compara con una Matrioshka. Para información más detallada sobre el middleware, consulte el [tema sobre el middleware](~/v4sdk/bot-builder-concept-middleware.md).

## <a name="bot-structure"></a>Estructura del bot

En las secciones siguientes, se examinan las _piezas clave_ de un EchoBot que se puede crear fácilmente mediante las plantillas proporcionadas para [ **CSharp**](../dotnet/bot-builder-dotnet-sdk-quickstart.md) o [**JavaScript**](../javascript/bot-builder-javascript-quickstart.md).

<!--Need to add section calling out the controller in code, and explaining it further-->

Un bot es una aplicación web y se proporcionan plantillas para cada idioma.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

La plantilla VSIX genera una aplicación web [ASP.NET MVC Core](https://dotnet.microsoft.com/apps/aspnet/mvc). Si consulta los conceptos básicos de [ASP.NET](https://docs.microsoft.com/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x), verá código similar en archivos como **Program.cs** y **Startup.cs**. Estos archivos son necesarios para todas las aplicaciones web y no son específicos del bot.

### <a name="appsettingsjson-file"></a>Archivo appsettings.json

El archivo **appsettings.json** especifica la información de configuración del bot, como el identificador de la aplicación y la contraseña, entre otras cosas. Si utiliza determinadas tecnologías o si usa este bot en producción, deberá agregar las claves o dirección URL específicas a esta configuración. Para este bot de eco, no obstante, no necesita hacer nada aquí ahora mismo; el identificador de aplicación y la contraseña se pueden dejar sin definir en este momento.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

<!-- TODO: Update this aka link to point to samples/javascript_nodejs/02.echobot (instead of samples/javascript_nodejs/02.a.echobot) once work-in-progress is merged into master. -->

El generador Yeoman crea una aplicación web de tipo [restify](http://restify.com/). Si observa la guía de inicio rápido de restify en su documentación, verá una aplicación similar al archivo **index.js** generado. Se describen algunos de los archivos de clave que genera la plantilla. El código de algunos archivos no se copiará, pero lo verá cuando ejecute el bot y puede consultar el ejemplo del [bot de eco de Node.js](https://aka.ms/js-echobot-sample).

### <a name="packagejson"></a>package.json

**package.json** especifica las dependencias del bot y sus versiones asociadas. Esto viene configurado por la plantilla y el sistema.

### <a name="env-file"></a>Archivo .env

El archivo **.env** especifica la información de configuración del bot, como el número de puerto, el identificador de aplicación y la contraseña, entre otras cosas. Si utiliza determinadas tecnologías o si usa este bot en producción, deberá agregar las claves o dirección URL específicas a esta configuración. Para este bot de eco, no obstante, no necesita hacer nada aquí ahora mismo; el identificador de aplicación y la contraseña se pueden dejar sin definir en este momento.

Para usar el archivo de configuración **.env** la plantilla necesita incluir un paquete adicional.  En primer lugar, obtenga el paquete `dotenv` de npm:

`npm install dotenv`

---

### <a name="bot-logic"></a>Lógica del bot

La lógica del bot procesa las actividades entrantes de uno o varios canales y genera actividades salientes como respuesta.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

La lógica principal del bot se define en el código del bot, que aquí se denomina `Bots/EchoBot.cs`. `EchoBot` deriva de `ActivityHandler`, que a su vez deriva de la interfaz de `IBot`. `ActivityHandler` define varios controladores para diferentes tipos de actividades, como los dos que se definen aquí: `OnMessageActivityAsync` y `OnMembersAddedAsync`. Estos métodos están protegidos, pero se puede sobrescribir, ya que se derivan de `ActivityHandler`.

Los controladores definidos en `ActivityHandler` son:

| Evento | Controlador | DESCRIPCIÓN |
| :-- | :-- | :-- |
| Cualquier tipo de actividad recibida | `OnTurnAsync` | Llama a uno de los otros controladores, en función del tipo de actividad que reciba. |
| Actividad de mensaje recibida | `OnMessageActivityAsync` | Se invalida para controlar un actividad `message`. |
| Actividad de actualización de conversación recibida | `OnConversationUpdateActivityAsync` | En una actividad `conversationUpdate`, llama a un controlador si cualquiera de los miembros, que no sea el bot, se une a la conversación, o la abandona. |
| Miembros que no son el bot se han unido a la conversación | `OnMembersAddedAsync` | Se invalida para controlar a los miembros que se unen a una conversación. |
| Miembros que no son el bot ha abandonado la conversación | `OnMembersRemovedAsync` | Se invalida para controlar a los miembros que abandonan una conversación. |
| Actividad de evento recibida | `OnEventActivityAsync` | En una actividad `event`, llama a un controlador específico del tipo de evento. |
| Actividad de evento token-respuesta recibida | `OnTokenResponseEventAsync` | Se invalida para controlar los eventos de respuesta del token. |
| Actividad de evento no token-respuesta recibida | `OnEventAsync` | Se invalida para controlar otros tipos de eventos. |
| Actividad de reacción de mensajes recibida | `OnMessageReactionActivityAsync` | En una actividad `messageReaction`, llama a un controlador si se han agregado o quitado una o más reacciones de un mensaje. |
| Reacciones de mensajes agregadas a un mensaje | `OnReactionsAddedAsync` | Invalide esto para controlar las reacciones agregadas a un mensaje. |
| Reacciones de mensajes eliminadas de un mensaje | `OnReactionsRemovedAsync` | Invalide esto para controlar las reacciones eliminadas de un mensaje. |
| Otro tipo de actividad recibida | `OnUnrecognizedActivityTypeAsync` | Se invalida para controlar cualquier tipo de actividad que no se controle de otra forma. |

Estos controladores tienen un objeto `turnContext` que proporciona información acerca de la actividad entrante, que corresponde a la solicitud HTTP entrante. Las actividades pueden ser de diversos tipos, por lo que cada controlador proporciona una actividad fuertemente tipada en su parámetro de contexto de turno; en la mayoría de los casos, `OnMessageActivityAsync` siempre se controlará y, por lo general, es el más común.

Al igual que en versiones anteriores de 4.x de este marco, también existe la opción de implementar el método público `OnTurnAsync`. Actualmente, la implementación base de este método controla la comprobación de errores y, después, llama a cada uno de los controladores específicos (por ejemplo, los dos que se definen en este ejemplo) en función del tipo de actividad entrante. En la mayoría de los casos, puede olvidar el método y usar los controladores individuales, pero si la situación requiere una implementación personalizada de `OnTurnAsync`, sigue siendo una opción.

> [!IMPORTANT]
> Si invalida el método `OnTurnAsync`, deberá llamar a `base.OnTurnAsync` para obtener la implementación base para llamar a los restantes controladores de `On<activity>Async`, o bien llamar a los controladores personalmente. De lo contrario, no se llamará a dichos controladores y el código no se ejecutará.

En este ejemplo, se da la bienvenida a un usuario nuevo o se devuelve el mensaje que el usuario envió mediante la llamada `SendActivityAsync`. La actividad saliente corresponde a la solicitud HTTP POST saliente.

```cs
public class MyBot : ActivityHandler
{
    protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
    {
        await turnContext.SendActivityAsync(MessageFactory.Text($"Echo: {turnContext.Activity.Text}"), cancellationToken);
    }

    protected override async Task OnMembersAddedAsync(IList<ChannelAccount> membersAdded, ITurnContext<IConversationUpdateActivity> turnContext, CancellationToken cancellationToken)
    {
        foreach (var member in membersAdded)
        {
            await turnContext.SendActivityAsync(MessageFactory.Text($"welcome {member.Name}"), cancellationToken);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

La lógica principal del bot se define en el código del bot, que aquí se denomina `bots\echoBot.js`. `EchoBot` deriva de `ActivityHandler`. `ActivityHandler` define varios controladores para diferentes tipos de actividades y se puede modificar el comportamiento del bot proporcionando lógica adicional, como con `onMessage` y `onConversationUpdate` aquí.

Los controladores definidos en `ActivityHandler` son:

| Evento | Controlador | DESCRIPCIÓN |
| :-- | :-- | :-- |
| Cualquier tipo de actividad recibida | `onTurn` | Se llama cuando se recibe cualquier actividad. |
| Actividad de mensaje recibida | `onMessage` | Se llama cuando se recibe una actividad `message`. |
| Actividad de actualización de conversación recibida | `onConversationUpdate` | Se llama cuando se recibe cualquier actividad `conversationUpdate`. |
| Miembros se unen a la conversación | `onMembersAdded` | Se llama cuando algún miembro se une a la conversación, incluido el bot. |
| Miembros abandonan la conversación | `onMembersRemoved` | Se llama cuando algún miembro abandona la conversación, incluido el bot. |
| Actividad de reacción de mensajes recibida | `onMessageReaction` | Se llama cuando se recibe cualquier actividad `messageReaction`. |
| Reacciones de mensajes agregadas a un mensaje | `onReactionsAdded` | Se llama cuando se agregan reacciones a un mensaje. |
| Reacciones de mensajes eliminadas de un mensaje | `onReactionsRemoved` | Se llama cuando se eliminan reacciones de un mensaje. |
| Actividad de evento recibida | `onEvent` | Se llama cuando se recibe cualquier actividad `event`. |
| Actividad de evento token-respuesta recibida | `onTokenResponseEvent` | Se llama cuando se recibe un evento `tokens/response`. |
| Otro tipo de actividad recibida | `onUnrecognizedActivityType` | Se llama cuando no hay definido un controlador para el tipo de actividad específico. |
| Los controladores de actividad se han completado | `onDialog` | Se llama después de que se hayan completado los controladores correspondientes. |

Llame al parámetro de función `next` desde cada controlador para permitir que continúe el procesamiento. Si no se llama a `next`, el procesamiento de la actividad finaliza.

En primer lugar, se comprueba en cada turno si el bot ha recibido un mensaje. Cuando recibimos un mensaje del usuario, devolvemos el mensaje que ha enviado.

```javascript
const { ActivityHandler } = require('botbuilder');

class MyBot extends ActivityHandler {
    constructor() {
        super();
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        this.onMessage(async (context, next) => {
            await context.sendActivity(`You said '${ context.activity.text }'`);
            // By calling next() you ensure that the next BotHandler is run.
            await next();
        });
        this.onConversationUpdate(async (context, next) => {
            await context.sendActivity('[conversationUpdate event detected]');
            // By calling next() you ensure that the next BotHandler is run.
            await next();
        });
    }
}

module.exports.MyBot = MyBot;
```

---

### <a name="access-the-bot-from-your-app"></a>Acceso al bot desde una aplicación

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

#### <a name="set-up-services"></a>Configuración de los servicios

El método `ConfigureServices` del archivo `Startup.cs` carga los servicios conectados, así como sus claves de `appsettings.json` o de Azure Key Vault (si hay), conecta el estado, y así sucesivamente. En este caso, vamos a agregar MVC y a establecer la versión de compatibilidad en los servicios y, a continuación, a configurar el adaptador y el bot para que estén disponibles a través de la inserción de dependencias en el controlador del bot.

<!-- want to explain the singleton vs transient here?-->

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    // Create the credential provider to be used with the Bot Framework Adapter.
    services.AddSingleton<ICredentialProvider, ConfigurationCredentialProvider>();

    // Create the Bot Framework Adapter.
    services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkHttpAdapter>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```

Para finalizar la configuración de la aplicación, el método `Configure` que especifica que la aplicación use MVC y otros archivos. Todos los bots que usen Bot Framework necesitarán dicha llamada de configuración, sin embargo estará ya definida en los ejemplos o en la plantilla de VSIX al compilar el bot. El entorno de ejecución llama a `ConfigureServices` y `Configure` cuando se inicia la aplicación.

#### <a name="bot-controller"></a>Controlador del bot

El controlador, que sigue la estructura de MVC, le permite determinar el enrutamiento de mensajes y de las solicitudes HTTP POST. En el caso de nuestro bot, pasamos la solicitud entrante al método *process async activity* como se explica en la sección [Pila de procesamiento de actividades](#the-activity-processing-stack) anterior. En la llamada, especificamos el bot y cualquier otra información de autorización que pueda ser necesaria.

El controlador implementa `ControllerBase`, contiene el adaptador y el bot que se estableció en `Startup.cs` (que están disponibles aquí a través de la inserción de dependencias) y pasa la información necesaria al bot cuando recibe un HTTP POST entrante.

En este caso, verá la clase seguida de los atributos de ruta y controlador, que ayudan al marco a enrutar los mensajes correctamente y a saber qué controlador se debe usar. Si cambia el valor del atributo de ruta, también cambiará el punto de conexión que el emulador u otros canales usan para acceder al bot.

```cs
// This ASP Controller is created to handle a request. Dependency Injection will provide the Adapter and IBot
// implementation at runtime. Multiple different IBot implementations running at different endpoints can be
// achieved by specifying a more specific type for the bot constructor argument.
[Route("api/messages")]
[ApiController]
public class BotController : ControllerBase
{
    private readonly IBotFrameworkHttpAdapter Adapter;
    private readonly IBot Bot;

    public BotController(IBotFrameworkHttpAdapter adapter, IBot bot)
    {
        Adapter = adapter;
        Bot = bot;
    }

    [HttpPost]
    public async Task PostAsync()
    {
        // Delegate the processing of the HTTP POST to the adapter.
        // The adapter will invoke the bot.
        await Adapter.ProcessAsync(Request, Response, Bot);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

#### <a name="indexjs"></a>index.js

El archivo `index.js` configura el bot y el servicio de hospedaje que reenviará las actividades a la lógica del bot.

#### <a name="required-libraries"></a>Bibliotecas necesarias

En la parte superior del archivo `index.js` encontrará una serie de módulos o bibliotecas que van a ser necesarios. Estos módulos le darán acceso a un conjunto de funciones que tal vez desee incluir en la aplicación.

```javascript
const dotenv = require('dotenv');
const path = require('path');
const restify = require('restify');

// Import required bot services.
// See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter } = require('botbuilder');

// This bot's main dialog.
const { MyBot } = require('./bot');

// Import required bot configuration.
const ENV_FILE = path.join(__dirname, '.env');
dotenv.config({ path: ENV_FILE });
```

#### <a name="set-up-services"></a>Configuración de los servicios

Las partes siguientes configuran el servidor y el adaptador que permiten al bot comunicarse con el usuario y enviar respuestas. El servidor escuchará el puerto especificado desde el archivo de configuración o revertirá al _3978_ para conectarse con su emulador. El adaptador actúa como el director del bot, dirige la comunicación entrante y saliente y la autenticación, entre otras.

```javascript
// Create HTTP server
const server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, () => {
    console.log(`\n${ server.name } listening to ${ server.url }`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/bot-framework-www-portal-emulator`);
    console.log(`\nTo talk to your bot, open the emulator select "Open Bot"`);
});

// Create adapter.
// See https://aka.ms/about-bot-adapter to learn more about how bots work.
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

// Catch-all for errors.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${ error }`);
    // Send a message to the user
    await context.sendActivity(`Oops. Something went wrong!`);
};

// Create the main dialog.
const myBot = new MyBot();
```

#### <a name="forwarding-requests-to-the-bot-logic"></a>Reenvío de solicitudes a la lógica del bot

El elemento `processActivity` del adaptador envía las actividades entrantes a la lógica del bot.
El tercer parámetro de `processActivity` es un controlador de funciones al que se llamará para realizar la lógica del bot después de que el enrutador haya procesado previamente la [actividad](#the-activity-processing-stack) recibida y se haya enrutado a través de algún middleware. La variable de contexto de turno, que se pasa como un argumento al controlador de función, se puede usar para proporcionar información sobre la actividad entrante, el remitente y el receptor, el canal o la conversación, entre otros. El procesamiento de la actividad se enruta al método `run` del bot. `run` se define en `ActivityHandler`; realiza algunas comprobaciones de errores y, después, llama a los controladores de eventos del bot en función del tipo de actividad que recibe.

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to main dialog.
        await myBot.run(context);
    });
});
```

---

## <a name="manage-bot-resources"></a>Administración de recursos de bot

Los recursos del bot, como el id. de la aplicación, las contraseñas, las claves o los secretos de los servicios conectados deben administrarse de manera adecuada. Para más información acerca de cómo hacerlo, consulte [Administración de recursos del bot](bot-file-basics.md).

## <a name="additional-resources"></a>Recursos adicionales

- Para conocer el rol del estado en los bots, consulte el artículo acerca de la [administración de estados](bot-builder-concept-state.md).

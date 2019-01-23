---
title: Uso de varios modelos de LUIS y QnA | Microsoft Docs
description: Obtenga información sobre cómo usar LUIS y QnA Maker en su bot.
keywords: LUIS, QnA, herramienta de distribución, varios servicios, intenciones de ruta
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c798c26f108458e1caeb16aa22c02c6e7c70fb61
ms.sourcegitcommit: 3cc768a8e676246d774a2b62fb9c688bbd677700
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/16/2019
ms.locfileid: "54323661"
---
# <a name="use-multiple-luis-and-qna-models"></a>Uso de varios modelos de LUIS y QnA

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

En este tutorial, demostramos cómo utilizar el servicio de distribución para enrutar las expresiones cuando hay varios modelos de LUIS y servicios de QnA Maker para diferentes escenarios que admite el bot. En este caso, configuramos la distribución con varios modelos de LUIS para conversaciones sobre automatización de dispositivos del hogar e información meteorológica; también vamos a configurar el servicio QnA Maker para responder a preguntas basadas en un archivo de texto de preguntas frecuentes como entrada. Este ejemplo combina los servicios siguientes.

| NOMBRE | DESCRIPCIÓN |
|------|------|
| HomeAutomation | Una aplicación LUIS que reconoce una intención de automatización de dispositivos del hogar con datos de la entidad asociada.|
| Tiempo | Una aplicación LUIS que reconoce las intenciones `Weather.GetForecast` y `Weather.GetCondition` con datos de localización.|
| Preguntas más frecuentes  | Una base de conocimiento de QnA Maker que proporciona respuestas a preguntas sencillas sobre el bot. |

## <a name="prerequisites"></a>Requisitos previos

- El código de este artículo se basa en el ejemplo **NLP con la herramienta de distribución**. Necesitará una copia del ejemplo en [C# ](https://aka.ms/dispatch-sample-cs) o en [JS](https://aka.ms/dispatch-sample-js).
- Se requieren conocimientos sobre los [conceptos básicos de bots](bot-builder-basics.md), el [procesamiento de lenguaje natural](bot-builder-howto-v4-luis.md), [QnA Maker](bot-builder-howto-qna.md) y el archivo [.bot](bot-file-basics.md).
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download) para pruebas.

## <a name="create-the-services-and-test-the-bot"></a>Creación de los servicios y prueba del bot

Puede seguir las instrucciones del archivo **LÉAME** para [C# ](https://aka.ms/dispatch-sample-readme-cs) o [JS](https://aka.ms/dispatch-sample-readme-js) para crear este bot mediante llamadas a la interfaz de la línea de comandos, o seguir estos pasos para crear manualmente el bot mediante las interfaces de usuario de Azure, LUIS y QnAMaker.

 ### <a name="create-your-bot-using-service-ui"></a>Creación del bot mediante la interfaz de usuario del servicio
 
Para empezar a crear manualmente el bot, descargue los siguientes 4 archivos que se encuentran en el repositorio de GitHub [BotFramework-Samples](https://aka.ms/botdispatchgitsamples) en una carpeta local: [home-automation.json](https://aka.ms/dispatch-home-automation-json), [weather.json ](https://aka.ms/dispatch-weather-json), [nlp-with-dispatchDispatch.json](https://aka.ms/dispatch-dispatch-json) y [QnAMaker.tsv](https://aka.ms/dispatch-qnamaker-tsv) Un método para realizar esta acción es abrir el vínculo del repositorio de GitHub anterior, hacer clic en **BotFramework-Samples** y, a continuación, hacr clic en "Clonar o descargar" el repositorio en el equipo local. Tenga en cuenta que estos archivos están en un repositorio diferente que el ejemplo mencionado en los requisitos previos.

### <a name="manually-create-luis-apps"></a>Creación manual de aplicaciones de LUIS

Inicie sesión en el [portal web de LUIS](https://www.luis.ai/). En la sección _Mis aplicaciones_, seleccione la pestaña _Import new app_ (Importar nueva aplicación). Aparecerá el siguiente cuadro de diálogo:

![Importar archivo json de LUIS](./media/tutorial-dispatch/import-new-luis-app.png)

Seleccione el botón _Choose app file_ (Elegir archivo de aplicación) y seleccione el archivo descargado "home-automation.json". Deje en blanco el campo de nombre opcional. Seleccione _Listo_.

Una vez que LUIS abra la aplicación Home Automation, seleccione el botón _Train_ (Entrenar). Esto permitirá entrenar la aplicación mediante el conjunto de expresiones que acaba de importar mediante el archivo "home-automation.json".

Cuando finalice el entrenamiento, seleccione el botón _Publish_ (Publicar). Aparecerá el siguiente cuadro de diálogo:

![Publicar aplicación de LUIS](./media/tutorial-dispatch/publish-luis-app.png)

Elija el entorno "production" (producción) y, después, seleccione el botón _Publish_ (Publicar).

Una vez que se ha publicado la nueva aplicación de LUIS, seleccione la pestaña _MANAGE_ (Administrar). En la página "Application Information" (Información de la aplicación), registre los valores `Application ID` y `Display name`. En la página "Key and Endpoints" (Clave y puntos de conexión), registre los valores `Authoring Key` y `Region`. El archivo "nlp-with-dispatch.bot" usará estos valores más adelante.

Una vez terminadas, _entrene_ y _publique_ la aplicación meteorológica y la de distribución de LUIS repitiendo estos mismos pasos para los archivos "weather.json" y "nlp-with-dispatchDispatch.json" que se descargaron localmente.

### <a name="manually-create-qna-maker-app"></a>Creación manual de una aplicación de QnA Maker

El primer paso para configurar una base de conocimiento de QnA Maker es configurar un servicio QnA Maker en Azure. Para ello, siga las instrucciones detalladas que se indican [aquí](https://aka.ms/create-qna-maker). Ahora, inicie sesión en el [portal web de QnAMaker](https://qnamaker.ai). Vaya al paso 2

![Paso 2, crear QnA](./media/tutorial-dispatch/create-qna-step-2.png)

y seleccione
1. La cuenta de Azure AD.
1. El nombre de la suscripción de Azure.
1. El nombre que ha creado para el servicio QnA Maker. (Si el servicio Azure QnA no aparece inicialmente en esta lista desplegable, pruebe a actualizar la página). 

Vaya al paso 3.

![Paso 3, crear QnA](./media/tutorial-dispatch/create-qna-step-3.png)

Proporcione un nombre para la base de conocimiento de QnA Maker. En este ejemplo vamos a usar el nombre "sample-qna".

Vaya al paso 4

![Paso 4, crear QnA](./media/tutorial-dispatch/create-qna-step-4.png)

Seleccione la opción _+ Agregar archivo_ y seleccione el archivo descargado "QnAMaker.tsv"

Hay una selección adicional para agregar una personalidad de _charla_ a la base de conocimiento, pero nuestro ejemplo no incluye esta opción.

Seleccione _Save and train_ (Guardar y entrenar) y, cuando finalice, seleccione la pestaña _PUBLISH_ (Publicar) y publique la aplicación.

Una vez que se publique la aplicación de QnA Maker, seleccione la pestaña _SETTINGS_ (Configuración) y desplácese hacia abajo hasta "Deployment details" (Detalles de la implementación). Registre los valores siguientes en la solicitud HTTP de _Postman_ de ejemplo.

```
POST /knowledgebases/<Your_Knowledgebase_Id>/generateAnswer
Host: <Your_Hostname>
Authorization: EndpointKey <Your_Endpoint_Key>
```
El archivo "nlp-with-dispatch.bot" usará estos valores más adelante.

### <a name="manually-update-your-bot-file"></a>Actualización manual del archivo .bot

Una vez creadas todas las aplicaciones de servicio, se tiene que agregar la información de cada una al archivo "nlp-with-dispatch.bot". Abra este archivo dentro del archivo de ejemplo de C# o JS que descargó anteriormente. Agregue los valores siguientes a cada sección "type": "luis" o "type": "dispatch"

```
"appId": "<Your_Recorded_App_Id>",
"authoringKey": "<Your_Recorded_Authoring_Key>",
"subscriptionKey": "<Your_Recorded_Authoring_Key>",
"version": "0.1",
"region": "<Your_Recorded_Region>",
```

Para la sección "type": "qna" Agregue los siguientes valores:

```
"type": "qna",
"name": "sample-qna",
"id": "201",
"kbId": "<Your_Recorded_Knowledgebase_Id>",
"subscriptionKey": "<Your_Azure_Subscription_Key>", // Used when creating your QnA service.
"endpointKey": "<Your_Recorded_Endpoint_Key>",
"hostname": "<Your_Recorded_Hostname>"
```

Cuando se hayan realizado todos los cambios, guarde este archivo.

### <a name="test-your-bot"></a>Prueba del bot

Ahora ejecute el ejemplo mediante el emulador. Una vez que el emulador se abra, seleccione el archivo "nlp-with-dispatch.bot".

Como referencia, estas son algunas de las preguntas y los comandos que están cubiertos por los servicios que hemos incluido:

* QnA Maker
  * `hi`, `good morning`
  * `what are you`, `what do you do`
* LUIS (automatización del hogar)
  * `turn on bedroom light`
  * `turn off bedroom light`
  * `make some coffee`
* LUIS (tiempo)
  * `whats the weather in redmond washington`
  * `what's the forecast for london`
  * `show me the forecast for nebraska`

### <a name="connecting-to-the-services-from-your-bot"></a>Conexión a los servicios desde el bot

Para conectarse a los servicios Dispatch, LUIS y QnA Maker, su bot extrae información desde el archivo **.bot**.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En **Startup.cs**, `ConfigureServices` lee en el archivo de configuración y `InitBotServices` usa esa información para inicializar los servicios. Cada vez que se crea, se inicializa el bot con el objeto `BotServices` registrado. Estas son las partes importantes de estos dos métodos.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    //...
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\nlp-with-dispatch.bot", secretKey);
    services.AddSingleton(sp => botConfig
        ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // ...
    
    var connectedServices = InitBotServices(botConfig);
    services.AddSingleton(sp => connectedServices);
    
    services.AddBot<NlpDispatchBot>(options =>
    {
          
          // The Memory Storage used here is for local bot debugging only. 
          // When the bot is restarted, everything stored in memory will be gone.

          Storage dataStore = new MemoryStorage();

          // ...

          // Create Conversation State object.
          // The Conversation State object is where we persist anything at the conversation-scope.

          var conversationState = new ConversationState(dataStore);
          options.State.Add(conversationState);
     });
}

```
El siguiente código inicializa las referencias del bot a servicios externos. Por ejemplo, aquí se crean los servicios LUIS y QnaMaker. Estos servicios externos se configuran mediante la clase `BotConfiguration` (basado en el contenido del archivo ".bot").

```csharp
private static BotServices InitBotServices(BotConfiguration config)
{
    var qnaServices = new Dictionary<string, QnAMaker>();
    var luisServices = new Dictionary<string, LuisRecognizer>();

    foreach (var service in config.Services)
    {
        switch (service.Type)
        {
            case ServiceTypes.Luis:
                {
                    // ...
                    var app = new LuisApplication(luis.AppId, luis.AuthoringKey, luis.GetEndpoint());
                    var recognizer = new LuisRecognizer(app);
                    luisServices.Add(luis.Name, recognizer);
                    break;
                }

            case ServiceTypes.Dispatch:
                // ...
                var dispatchApp = new LuisApplication(dispatch.AppId, dispatch.AuthoringKey, dispatch.GetEndpoint());

                // Since the Dispatch tool generates a LUIS model, we use the LuisRecognizer to resolve the
                // dispatching of the incoming utterance.
                var dispatchARecognizer = new LuisRecognizer(dispatchApp);
                luisServices.Add(dispatch.Name, dispatchARecognizer);
                break;

            case ServiceTypes.QnA:
                {
                    // ...
                    var qnaEndpoint = new QnAMakerEndpoint()
                    {
                        KnowledgeBaseId = qna.KbId,
                        EndpointKey = qna.EndpointKey,
                        Host = qna.Hostname,
                    };

                    var qnaMaker = new QnAMaker(qnaEndpoint);
                    qnaServices.Add(qna.Name, qnaMaker);
                    break;
                }
        }
    }

    return new BotServices(qnaServices, luisServices);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El código de ejemplo usa constantes predefinidas de nomenclatura para identificar las distintas secciones del archivo .bot. Si ha modificado cualquier nombre de sección en los nombres de ejemplo originales del archivo _nlp-with-dispatch.bot_, asegúrese de que encuentra la declaración de constantes asociada en los archivos **bot.js**, **homeAutomation.js**, **qna.js** o **weather.js** y cambie esa entrada por el nombre modificado.  
```javascript
// In file bot.js
// this is the LUIS service type entry in the .bot file.
const DISPATCH_CONFIG = 'nlp-with-dispatchDispatch';

// In file homeAutomation.js
// this is the LUIS service type entry in the .bot file.
const LUIS_CONFIGURATION = 'Home Automation';

// In file qna.js
// Name of the QnA Maker service in the .bot file.
const QNA_CONFIGURATION = 'sample-qna';

// In file weather.js
// this is the LUIS service type entry in the .bot file.
const WEATHER_LUIS_CONFIGURATION = 'Weather';
```

En **bot.js** la información contenida en el archivo de configuración _nlp-with-dispatch.bot_ se usa para conectar el bot de distribución a los diversos servicios. Cada constructor busca y usa las secciones adecuadas del archivo de configuración en función de los nombres de sección detallados anteriormente.

```javascript
class DispatchBot {
    constructor(conversationState, userState, botConfig) {
        //...
        this.homeAutomationDialog = new HomeAutomation(conversationState, userState, botConfig);
        this.weatherDialog = new Weather(botConfig);
        this.qnaDialog = new QnA(botConfig);

        this.conversationState = conversationState;
        this.userState = userState;

        // dispatch recognizer
        const dispatchConfig = botConfig.findServiceByNameOrId(DISPATCH_CONFIG);
        //...
```
---

### <a name="calling-the-services-from-your-bot"></a>Llamada a los servicios desde el bot

La lógica del bot comprueba la entrada del usuario en el modelo de distribución combinada.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En el archivo **NlpDispatchBot.cs**, el constructor del bot obtiene el objeto `BotServices` que se registrará en el inicio.

```csharp
private readonly BotServices _services;

public NlpDispatchBot(BotServices services)
{
    _services = services ?? throw new System.ArgumentNullException(nameof(services));

    //...
}
```

En el método `OnTurnAsync` del bot, comprobamos los mensajes entrantes del usuario en el modelo Dispatch.

```csharp
// Get the intent recognition result
var recognizerResult = await _services.LuisServices[DispatchKey].RecognizeAsync(context, cancellationToken);
var topIntent = recognizerResult?.GetTopScoringIntent();

if (topIntent == null)
{
    await context.SendActivityAsync("Unable to get the top intent.");
}
else
{
    await DispatchToTopIntentAsync(context, topIntent, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el método `onTurn` de **bot.js**, se comprueban los mensajes entrantes del usuario. Si se recibe el tipo _ActivityType.Message_, este mensaje se envía a través de _dispatchRecognizer_ del bot.

```javascript
if (turnContext.activity.type === ActivityTypes.Message) {
    // determine which dialog should fulfill this request
    // call the dispatch LUIS model to get results.
    const dispatchResults = await this.dispatchRecognizer.recognize(turnContext);
    const dispatchTopIntent = LuisRecognizer.topIntent(dispatchResults);
    //...
 }
```
---

### <a name="working-with-the-recognition-results"></a>Trabajo con los resultados de reconocimiento

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Cuando el modelo genera un resultado, indica qué servicio puede procesar de la manera más adecuada la declaración. El código de este bot enruta la solicitud al servicio correspondiente y, a continuación, resume la respuesta del servicio que llama.

```csharp
// Depending on the intent from Dispatch, routes to the right LUIS model or QnA service.
private async Task DispatchToTopIntentAsync(
    ITurnContext context,
    (string intent, double score)? topIntent,
    CancellationToken cancellationToken = default(CancellationToken))
{
    const string homeAutomationDispatchKey = "l_Home_Automation";
    const string weatherDispatchKey = "l_Weather";
    const string noneDispatchKey = "None";
    const string qnaDispatchKey = "q_sample-qna";

    switch (topIntent.Value.intent)
    {
        case homeAutomationDispatchKey:
            await DispatchToLuisModelAsync(context, HomeAutomationLuisKey);

            // Here, you can add code for calling the hypothetical home automation service, passing in any entity
            // information that you need.
            break;
        case weatherDispatchKey:
            await DispatchToLuisModelAsync(context, WeatherLuisKey);

            // Here, you can add code for calling the hypothetical weather service,
            // passing in any entity information that you need
            break;
        case noneDispatchKey:
            // You can provide logic here to handle the known None intent (none of the above).
            // In this example we fall through to the QnA intent.
        case qnaDispatchKey:
            await DispatchToQnAMakerAsync(context, QnAMakerKey);
            break;

        default:
            // The intent didn't match any case, so just display the recognition results.
            await context.SendActivityAsync($"Dispatch intent: {topIntent.Value.intent} ({topIntent.Value.score}).");
            break;
    }
}

// Dispatches the turn to the request QnAMaker app.
private async Task DispatchToQnAMakerAsync(
    ITurnContext context,
    string appName,
    CancellationToken cancellationToken = default(CancellationToken))
{
    if (!string.IsNullOrEmpty(context.Activity.Text))
    {
        var results = await _services.QnAServices[appName].GetAnswersAsync(context).ConfigureAwait(false);
        if (results.Any())
        {
            await context.SendActivityAsync(results.First().Answer, cancellationToken: cancellationToken);
        }
        else
        {
            await context.SendActivityAsync($"Couldn't find an answer in the {appName}.");
        }
    }
}


// Dispatches the turn to the requested LUIS model.
private async Task DispatchToLuisModelAsync(
    ITurnContext context,
    string appName,
    CancellationToken cancellationToken = default(CancellationToken))
{
    await context.SendActivityAsync($"Sending your request to the {appName} system ...");
    var result = await _services.LuisServices[appName].RecognizeAsync(context, cancellationToken);

    await context.SendActivityAsync($"Intents detected by the {appName} app:\n\n{string.Join("\n\n", result.Intents)}");

    if (result.Entities.Count > 0)
    {
        await context.SendActivityAsync($"The following entities were found in the message:\n\n{string.Join("\n\n", result.Entities)}");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Cuando el modelo genera un resultado, indica qué servicio puede procesar de la manera más adecuada la declaración. El código de este bot enruta la solicitud al servicio correspondiente.

```javascript
switch (dispatchTopIntent) {
   case HOME_AUTOMATION_INTENT:
      await this.homeAutomationDialog.onTurn(turnContext);
      break;
   case WEATHER_INTENT:
      await this.weatherDialog.onTurn(turnContext);
      break;
   case QNA_INTENT:
      await this.qnaDialog.onTurn(turnContext);
      break;
   case NONE_INTENT:
      default:
      // Unknown request
       await turnContext.sendActivity(`I do not understand that.`);
       await turnContext.sendActivity(`I can help with weather forecast, turning devices on and off and answer general questions like 'hi', 'who are you' etc.`);
 }
 
 // In homeAutomation.js
 async onTurn(turnContext) {
    // make call to LUIS recognizer to get home automation intent + entities
    const homeAutoResults = await this.luisRecognizer.recognize(turnContext);
    const topHomeAutoIntent = LuisRecognizer.topIntent(homeAutoResults);
    // depending on intent, call turn on or turn off or return unknown
    switch (topHomeAutoIntent) {
       case HOME_AUTOMATION_INTENT:
          await this.handleDeviceUpdate(homeAutoResults, turnContext);
          break;
       case NONE_INTENT:
       default:
         await turnContext.sendActivity(`HomeAutomation dialog cannot fulfill this request.`);
    }
}
    
// In weather.js
async onTurn(turnContext) {
   // Call weather LUIS model.
   const weatherResults = await this.luisRecognizer.recognize(turnContext);
   const topWeatherIntent = LuisRecognizer.topIntent(weatherResults);
   // Get location entity if available.
   const locationEntity = (LOCATION_ENTITY in weatherResults.entities) ? weatherResults.entities[LOCATION_ENTITY][0] : undefined;
   const locationPatternAnyEntity = (LOCATION_PATTERNANY_ENTITY in weatherResults.entities) ? weatherResults.entities[LOCATION_PATTERNANY_ENTITY][0] : undefined;
   // Depending on intent, call "Turn On" or "Turn Off" or return unknown.
   switch (topWeatherIntent) {
      case GET_CONDITION_INTENT:
         await turnContext.sendActivity(`You asked for current weather condition in Location = ` + (locationEntity || locationPatternAnyEntity));
         break;
      case GET_FORECAST_INTENT:
         await turnContext.sendActivity(`You asked for weather forecast in Location = ` + (locationEntity || locationPatternAnyEntity));
         break;
      case NONE_INTENT:
      default:
         wait turnContext.sendActivity(`Weather dialog cannot fulfill this request.`);
   }
}
    
// In qna.js
async onTurn(turnContext) {
   // Call QnA Maker and get results.
   const qnaResult = await this.qnaRecognizer.generateAnswer(turnContext.activity.text, QNA_TOP_N, QNA_CONFIDENCE_THRESHOLD);
   if (!qnaResult || qnaResult.length === 0 || !qnaResult[0].answer) {
       await turnContext.sendActivity(`No answer found in QnA Maker KB.`);
       return;
    }
    // respond with qna result
    await turnContext.sendActivity(qnaResult[0].answer);
}
```
---

## <a name="edit-intents-to-improve-performance"></a>Edición de intenciones para mejorar el rendimiento

Una vez que el bot se esté ejecutando, es posible mejorar su rendimiento mediante la eliminación de expresiones parecidas o superpuestas. Por ejemplo, supongamos que en la aplicación de LUIS `Home Automation` para las solicitudes como "encender las luces" se asignan a una intención "TurnOnLights", pero las solicitudes como "¿Por qué no se encienden las luces?" se asignan a una intención "None" para que se puedan pasar a QnA Maker. Al combinar la aplicación de LUIS y el servicio QnA Maker mediante la herramienta de distribución, es preciso llevar a cabo una de las acciones siguientes:

* Quitar la intención "None" de la aplicación de LUIS `Home Automation` original y, en su lugar, agregar las expresiones de esa intención a la intención "None" de la aplicación de distribuidor.
* Si no quita la intención "None" de la aplicación de LUIS original, tendrá que agregar, en su lugar, lógica en su bot para pasar los mensajes que coinciden con esa intención al servicio QnA Maker.

Cualquiera de las dos acciones anteriores reducirá el número de veces que el bot responderá a los usuarios con el mensaje "Couldn't find an answer" (No se pudo encontrar una respuesta). 

## <a name="additional-resources"></a>Recursos adicionales

**Actualizar o crear un nuevo modelo de LUIS:** Este ejemplo se basa en un modelo de LUIS preconfigurado. Puede encontrar información adicional que le ayudará a actualizar este modelo o crear un nuevo modelo de LUIS, [aquí](https://aka.ms/create-luis-model#updating-your-cognitive-models
).

**Eliminación de recursos:** en este ejemplo se crea una serie de aplicaciones y recursos que puede eliminar mediante los pasos que se indican a continuación, pero no debe eliminar los recursos en los que se basan *otras aplicaciones o servicios*. 

_Recursos de LUIS_
1. Inicie sesión en el portal [luis.ai](https://www.luis.ai).
1. Vaya a la página _My Apps_ (Mis aplicaciones).
1. Seleccione las aplicaciones creadas por este ejemplo.
   * `Home Automation`
   * `Weather`
   * `NLP-With-Dispatch-BotDispatch`
1. Haga clic en _Eliminar_ y haga clic en _Aceptar_ para confirmar.

_Recursos de QnA Maker_
1. Inicie sesión en el portal [qnamaker.ai](https://www.qnamaker.ai/).
1. Vaya a la página _My knowledge bases_ (Mis bases de conocimiento).
1. Haga clic en el botón Delete (Eliminar) de la base de conocimiento `Sample QnA` y haga clic en _Delete_ para confirmar.

**Procedimiento recomendado:** para mejorar los servicios utilizados en este ejemplo, consulte los procedimientos recomendados para [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-best-practices) y [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/concepts/best-practices).
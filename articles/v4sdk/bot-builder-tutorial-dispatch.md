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
ms.date: 11/26/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9b0ddf5cf8af61048ba78f10824c9573da82fc08
ms.sourcegitcommit: a722f960cd0a8513d46062439eb04de3a0275346
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/27/2018
ms.locfileid: "52336275"
---
# <a name="use-multiple-luis-and-qna-models"></a>Uso de varios modelos de LUIS y QnA

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

En este tutorial, demostramos cómo utilizar el servicio de distribución para enrutar las expresiones cuando hay varios modelos de LUIS y servicios de QnA Maker para diferentes escenarios que admite el bot. En este caso, configuramos la distribución con varios modelos de LUIS para conversaciones en torno a la automatización de dispositivos del hogar y la información meteorológica, además del servicio QnA Maker para responder a las preguntas basadas en un archivo de texto de preguntas frecuentes como entrada. Este ejemplo combina los servicios siguientes.

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

Siga las instrucciones del archivo **Léame** para [C#](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/14.nlp-with-dispatch/README.md) o [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/14.nlp-with-dispatch/README.md) para crear y ejecutar el ejemplo mediante el emulador. 

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

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

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

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

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

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

---

## <a name="evaluate-the-dispatchers-performance"></a>Evaluar el rendimiento del distribuidor

Algunos mensajes de usuario se proporcionan como ejemplos en las aplicaciones de LUIS y los servicios QnA Maker, y la aplicación de LUIS combinada que la herramienta de distribución genera no funcionará bien para esas entradas. Para comprobar el rendimiento de la aplicación, use la opción `eval`.

```shell
dispatch eval
```

Al ejecutar `dispatch eval` se genera un archivo **Summary.html** que proporciona estadísticas sobre el rendimiento predicho del nuevo modelo de lenguaje. Puede ejecutar `dispatch eval` en cualquier aplicación de LUIS, no solo en las que la herramienta de distribución haya creado.

### <a name="edit-intents-for-duplicates-and-overlaps"></a>Editar las intenciones para los duplicados y las superposiciones

Revise las expresiones de ejemplo marcadas como duplicados en **Summary.html** y quite los ejemplos similares o que se superponen. Por ejemplo, supongamos que en la aplicación de LUIS `Home Automation` para las solicitudes como "encender las luces" se asignan a una intención "TurnOnLights", pero las solicitudes como "¿Por qué no se encienden las luces?" se asignan a una intención "None" para que se puedan pasar a QnA Maker. Al combinar la aplicación de LUIS y el servicio QnA Maker mediante la herramienta de distribución, es preciso llevar a cabo una de las acciones siguientes:

* Quitar la intención "None" de la aplicación de LUIS `Home Automation` original y agregar las expresiones de dicha intención a la intención "None" de la aplicación de distribuidor.
* Si no quita la intención "None" de la aplicación de LUIS original, deberá agregar lógica en su bot para pasar los mensajes que coinciden con dicha intención al servicio QnA Maker.


## <a name="additional-resources"></a>Recursos adicionales 

**Eliminar recursos:** en este ejemplo se crea una serie de aplicaciones y recursos que puede eliminar mediante los pasos que se indican a continuación, pero no debe eliminar los recursos en los que se basan *otras aplicaciones o servicios*. 

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

---
title: Integración de varios servicios de LUIS y QnA con la herramienta de distribución | Microsoft Docs
description: Obtenga información sobre cómo usar LUIS y QnA Maker en su bot.
keywords: LUIS, QnA, herramienta de distribución, varios servicios
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 10/09/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d1216cf863015e48a18c2f52e7326262c3b271ff
ms.sourcegitcommit: a9270702536eb34dedf5128bf57889c1ffcd7f4c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/11/2018
ms.locfileid: "49088897"
---
# <a name="integrate-multiple-luis-and-qna-services-with-the-dispatch-tool"></a>Integración de varios servicios de LUIS y QnA con la herramienta de distribución

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

<!--TODO: Add the JS sections back in and update them once the JS sample is working.-->

En este tutorial se muestra cómo usar un modelo de LUIS generado por la herramienta de distribución, a fin de integrar el bot con varias aplicaciones de Language Understanding (LUIS) y servicios QnA Maker. Este ejemplo combina los servicios siguientes.

| Tipo de servicio | NOMBRE | DESCRIPCIÓN |
|------|------|------|
| Aplicación de LUIS | HomeAutomation | Reconoce una intención de automatización del hogar con datos de la entidad asociada.|
| Aplicación de LUIS | Tiempo | Reconoce las intenciones Weather.GetForecast y Weather.GetCondition con los datos de ubicación.|
| Servicio QnA Maker | Preguntas más frecuentes  | Proporciona respuestas a algunas preguntas sencillas sobre el bot. |

El código para este artículo se toma del ejemplo **NLP con Dispatch** [[C#](https://aka.ms/dispatch-sample-cs)<!-- | [JS](https://aka.ms/dispatch-sample-js)-->].

Consulte [reconocimiento del lenguaje](bot-builder-concept-luis.md), para obtener información general de los servicios de lenguaje. Consulte los artículos para [LUIS](bot-builder-howto-v4-luis.md) y [QnA Maker](bot-builder-howto-qna.md), para obtener instrucciones sobre cómo implementar estas opciones en un bot.

Puede seguir las instrucciones en el archivo Léame del ejemplo para configurar y probar el bot, o puede pasar a [notas sobre el código](#notes-about-the-code).

## <a name="create-the-services-and-test-the-bot"></a>Creación de los servicios y prueba del bot

Siga las instrucciones del archivo Léame para el ejemplo. Tendrá que utilizar las herramientas de la CLI para crear y publicar estos servicios y actualizar la información sobre ellos en el archivo de configuración (**.bot**).

1. Clone o extraiga los ejemplos del repositorio.
1. Instale las herramientas de la CLI de BotBuilder.
1. Configure manualmente los servicios necesarios.

### <a name="test-your-bot"></a>Prueba del bot

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Inicie el bot con Visual Studio o Visual Studio Code.

<!--
# [JavaScript](#tab/javascript)
-->

---

Conéctese al bot con Bot Framework Channel Emulator.

Se muestran algunas de las entradas cubiertas por los servicios que se han incluido:

* QnA Maker
  * `hi`, `good morning`
  * `what are you`, `what do you do`
* LUIS (automatización del hogar)
  * `turn on bedroom light`
  * `turn off bedroom light`
  * `make some coffee`
* LUIS (tiempo)
  * `whats the weather in chennai india`
  * `what's the forecast for bangalore`
  * `show me the forecast for nebraska`

## <a name="notes-about-the-code"></a>Notas sobre el código

### <a name="packages"></a>Paquetes

En este ejemplo se usan los paquetes siguientes.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Las versiones más recientes de v4 de estos [paquetes de NuGet](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui#updating-a-package).

* `Microsoft.Bot.Builder`
* `Microsoft.Bot.Builder.AI.Luis`
* `Microsoft.Bot.Builder.AI.QnA`
* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Configuration`

<!--
# [JavaScript](#tab/javascript)

Download the [LUIS Dispatch sample][DispatchBotJs].  Install the required packages, including the `botbuilder-ai` package for LUIS and QnA Maker, using npm:

* `npm install --save botbuilder`
* `npm install --save botbuilder-ai`
-->

---

### <a name="botbuilder-cli-tools"></a>Herramientas de la CLI de BotBuilder

El ejemplo usa estas [herramientas de la CLI de BotBuilder](https://aka.ms/botbuilder-tools-readme) (disponibles a través de npm) para crear, entrenar y publicar los servicios de distribución de LUIS, QnA Maker y Dispatch; y para registrar información sobre estos servicios en la configuración de su archivo bot (**.bot**).

* [Dispatch](https://aka.ms/botbuilder-tools-dispatch)
* [LUDown](https://aka.ms/botbuilder-tools-ludown-readme)
* [LUIS](https://aka.ms/botbuilder-tools-luis)
* [MSBot](https://aka.ms/botbuilder-tools-msbot-readme)
* [QnAMaker](https://aka.ms/botbuilder-tools-qnaMaker)

> [!TIP]
> Para asegurarse de que tiene la versión más recientes de npm y de estas herramientas de la CLI, ejecute el siguiente comando.
>
> ```shell
> npm i -g npm dispatch ludown luis-apis msbot qnamaker
> ```

Una vez que ha usado las herramientas para configurar los servicios, su archivo **.bot** para este ejemplo debe ser similar al siguiente. (Puede ejecutar `msbot secret -n` para cifrar los valores confidenciales de este archivo).

```json
{
    "name": "NLP-With-Dispatch-Bot",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "id": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages"
        },
        {
            "type": "luis",
            "name": "Home Automation",
            "appId": "<your-home-automation-luis-app-id>",
            "version": "0.1",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "region": "westus",
            "id": "110"
        },
        {
            "type": "luis",
            "name": "Weather",
            "appId": "<your-weather-luis-app-id>",
            "version": "0.1",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "region": "westus",
            "id": "92"
        },
        {
            "type": "qna",
            "name": "Sample QnA",
            "kbId": "<your-qna-knowledge-base-id>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "endpointKey": "<your-qna-endpoint-key>",
            "hostname": "<your-qna-host-name>",
            "id": "184"
        },
        {
            "type": "dispatch",
            "name": "NLP-With-Dispatch-BotDispatch",
            "appId": "<your-dispatch-app-id>",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "version": "Dispatch",
            "region": "westus",
            "serviceIds": [
                "110",
                "92",
                "184"
            ],
            "id": "27"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

### <a name="connecting-to-the-services-from-your-bot"></a>Conexión a los servicios desde el bot

Para conectarse a los servicios Dispatch, LUIS y QnA Maker, su bot extrae información desde el archivo **.bot**.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En **Startup.cs**, `ConfigureServices` lee en el archivo de configuración y `InitBotServices` usa esa información para inicializar los servicios. Cada vez que se crea, se inicializa el bot con el objeto `BotServices` registrado. Estas son las partes importantes de estos dos métodos.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    //...
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
    services.AddSingleton(sp => botConfig
        ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // Retrieve current endpoint.
    var environment = _isProduction ? "production" : "development";
    var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
    if (!(service is EndpointService endpointService))
    {
        throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
    }

    var connectedServices = InitBotServices(botConfig);

    services.AddSingleton(sp => connectedServices);
    //...
}
```

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
/// <summary>
/// Depending on the intent from Dispatch, routes to the right LUIS model or QnA service.
/// </summary>
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

/// <summary>
/// Dispatches the turn to the request QnAMaker app.
/// </summary>
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

/// <summary>
/// Dispatches the turn to the requested LUIS model.
/// </summary>
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

Al ejecutar `dispatch eval` se genera un archivo **Summary.html** que proporciona estadísticas sobre el rendimiento predicho del nuevo modelo de lenguaje.

> [!TIP]
> Puede ejecutar `dispatch eval` en cualquier aplicación de LUIS, no solo en las que la herramienta de distribución haya creado.

### <a name="edit-intents-for-duplicates-and-overlaps"></a>Editar las intenciones para los duplicados y las superposiciones

Revise las expresiones de ejemplo marcadas como duplicados en **Summary.html** y quite los ejemplos similares o que se superponen. Por ejemplo, supongamos que en la aplicación de LUIS `Home Automation` para las solicitudes como "encender las luces" se asignan a una intención "TurnOnLights", pero las solicitudes como "¿Por qué no se encienden las luces?" se asignan a una intención "None" para que se puedan pasar a QnA Maker. Al combinar la aplicación de LUIS y el servicio QnA Maker mediante la herramienta de distribución, es preciso llevar a cabo una de las acciones siguientes:

* Quitar la intención "None" de la aplicación de LUIS `Home Automation` original y agregar las expresiones de dicha intención a la intención "None" de la aplicación de distribuidor.
* Si no quita la intención "None" de la aplicación de LUIS original, deberá agregar lógica en su bot para pasar los mensajes que coinciden con dicha intención al servicio QnA Maker.

> [!TIP]
> Vea [Best practices for Language Understanding](./bot-builder-concept-luis.md#best-practices-for-language-understanding) (Procedimientos recomendados para Language Understanding) para obtener sugerencias sobre cómo mejorar el rendimiento del modelo de lenguaje.

## <a name="to-clean-up-resources-from-this-sample"></a>Para limpiar los recursos de este ejemplo

Este ejemplo crea varias aplicaciones y recursos. Para eliminarlos, puede seguir estas instrucciones.

### <a name="luis-resources"></a>Recursos de LUIS

1. Inicie sesión en el portal [luis.ai](https://www.luis.ai).
1. Vaya a la página **My Apps** (Mis aplicaciones).
1. Seleccione las aplicaciones creadas por este ejemplo.
   * `Home Automation`
   * `Weather`
   * `NLP-With-Dispatch-BotDispatch`
1. Haga clic en **Eliminar** y haga clic en **Aceptar** para confirmar.

### <a name="qna-maker-resources"></a>Lista de recursos de QnA Maker

1. Inicie sesión en el portal [qnamaker.ai](https://www.qnamaker.ai/).
1. Vaya a la página **My knowledge bases** (Mis bases de conocimiento).
1. Haga clic en el botón Delete (Eliminar) de la base de conocimiento `Sample QnA` y haga clic en **Delete** para confirmar.

### <a name="azure-resources"></a>Recursos de Azure

> [!WARNING]
> No elimine ninguno de estos recursos en los que otras aplicaciones o servicios se basan.

1. Inicie sesión en el [Azure Portal](https://portal.azure.com/).
1. Vaya a la página **Introducción** del recurso de Cognitive Services que creó para el ejemplo.
1. Haga clic en **Eliminar** y haga clic en **Sí** para confirmar.

## <a name="additional-resources"></a>Recursos adicionales

* [Comprensión del lenguaje](bot-builder-concept-luis.md)
* [Diseño de bots de conocimientos](../bot-service-design-pattern-knowledge-base.md)
* [Configuración de la preparación para la voz](../bot-service-manage-speech-priming.md)
* [Uso de LUIS para la comprensión del lenguaje](bot-builder-howto-v4-luis.md)
* [Uso de QnA Maker para responder a preguntas](bot-builder-howto-qna.md)

---
title: Uso de QnA Maker para responder preguntas | Microsoft Docs
description: Obtenga información sobre cómo usar QnA Maker en el bot.
keywords: pregunta y respuesta, QnA, preguntas más frecuentes, qna maker
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 11/19/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0708244b9f9e4859ba069ed463cef83a0ecdf20d
ms.sourcegitcommit: b9482670285295a2af0dfbb8f4b7e543c1c10542
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/13/2018
ms.locfileid: "53327161"
---
# <a name="use-qna-maker-to-answer-questions"></a>Uso de QnA Maker para responder preguntas

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Puede usar el servicio QnA Maker para agregar compatibilidad de preguntas y respuestas al bot. Uno de los requisitos básicos para crear su propia instancia del servicio QnA Maker es inicializarlo con preguntas y respuestas. En muchos casos, las preguntas y respuestas ya existen en el contenido como preguntas más frecuentes u otra documentación. En otras ocasiones, le interesará personalizar las respuestas a las preguntas de forma más natural y conversacional.

En este tema se creará una base de conocimiento y usarla en un bot.

## <a name="prerequisites"></a>Requisitos previos
- Cuenta de [QnA Maker](https://www.qnamaker.ai/)
- El código de este artículo se basa en el ejemplo de **QnA Maker**. Necesitará una copia del ejemplo en [ C# ](https://aka.ms/cs-qna) o [JS](https://aka.ms/js-qna-sample).
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)
- Base de conocimientos de [conceptos básicos de bots](bot-builder-basics.md), [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/overview/overview) y el archivo [.bot](bot-file-basics.md).

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>Creación de una instancia del servicio QnA Maker y publicación de una base de conocimiento
1. En primer lugar, deberá crear un [servicio QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure).
1. A continuación, va a crear una base de conocimiento con el archivo `smartLightFAQ.tsv` que se encuentra en la carpeta CognitiveModels del proyecto. Los pasos para crear, entrenar y publicar la [base de conocimiento](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base) de QnA Maker se muestran en la documentación de QnA Maker. Cuando siga estos pasos, asigne el nombre de `qna` a la base de conocimiento y utilice el archivo `smartLightFAQ.tsv` para rellenarla.

## <a name="obtain-values-to-connect-to-your-connect-your-bot-to-the-knowledge-base"></a>Obtención de valores para conectar el bot a la base de conocimiento
1. En el sitio de [QnA Maker](https://www.qnamaker.ai/), seleccione la base de conocimiento.
1. Con la base de conocimiento abierta, seleccione **Configuración**. Registre el valor que se aparece en _nombre del servicio_ como <su_base_de_conocimiento>
1. Desplácese hacia abajo hasta encontrar **Detalles de implementación** y anote los siguientes valores:
   - POST /knowledgebases/<id_de_su_base_de_conocimiento>/generateAnswer
   - Host: https://<su_nombre_de_host>.azurewebsites.net/qnamaker
   - Autorización: EndpointKey <su_clave_de_punto_de_conexión>

## <a name="update-the-bot-file"></a>Actualización del archivo .bot
En primer lugar, agregue la información necesaria para acceder a la base de conocimiento, incluido el nombre de host, la clave del punto de conexión y el identificador de la base de conocimiento (KbId) en `qnamaker.bot`. Estos son los valores que guardó en **Configuración** en la base de conocimiento de QnA Maker.

```json
{
  "name": "qnamaker",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "appPassword": ""
      "id": "25",
    
    },
    {
      "type": "qna",
      "name": "QnABot",
      "KbId": "<YOUR_KNOWLEDGE_BASE_ID>",
      "subscriptionKey": "<Your_Azure_Subscription_Key>", // Used when creating your QnA service.
      "endpointKey": "<Your_Recorded_Endpoint_Key>",
      "hostname": "<Your_Recorded_Hostname>",
      "id": "117"
    }
  ],
  "padlock": "",
   "version": "2.0"
}
```

# <a name="ctabcs"></a>[C#](#tab/cs)
A continuación, se inicializa una nueva instancia de la clase BotService en BotServices.cs, que toma la información anterior de su archivo .bot. El servicio externo se configura mediante la clase BotConfiguration.

```csharp
private static BotServices InitBotServices(BotConfiguration config)
{
    var qnaServices = new Dictionary<string, QnAMaker>();
    foreach (var service in config.Services)
    {
        switch (service.Type)
        {
            case ServiceTypes.QnA:
            {
                // Create a QnA Maker that is initialized and suitable for passing
                // into the IBot-derived class (QnABot).
                var qna = (QnAMakerService)service;
                if (qna == null)
                {
                    throw new InvalidOperationException("The QnA service is not configured correctly in your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.KbId))
                {
                    throw new InvalidOperationException("The QnA KnowledgeBaseId ('kbId') is required to run this sample. Please update your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.EndpointKey))
                {
                    throw new InvalidOperationException("The QnA EndpointKey ('endpointKey') is required to run this sample. Please update your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.Hostname))
                {
                    throw new InvalidOperationException("The QnA Host ('hostname') is required to run this sample. Please update your '.bot' file.");
                }

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
    var connectedServices = new BotServices(qnaServices);
    return connectedServices;
}
```

A continuación, en QnABot.cs, proporcionamos el bot a esta instancia de QnAMaker. Si va a acceder a su propia base de conocimiento, cambie el mensaje de _bienvenida_ que se muestra a continuación para proporcionar instrucciones iniciales útiles para los usuarios.

```csharp
public class QnABot : IBot
{
    public static readonly string QnAMakerKey = "QnABot";
    private const string WelcomeText = @"This bot will introduce you to QnA Maker.
                                         Ask a question to get started.";
    private readonly BotServices _services;
    public QnABot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        Console.WriteLine($"{_services}");
        if (!_services.QnAServices.ContainsKey(QnAMakerKey))
        {
            throw new System.ArgumentException($"Invalid configuration. Please check your '.bot' file for a QnA service named '{QnAMakerKey}'.");
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

En nuestro ejemplo, el código de inicio está en un archivo **index.js**, el código para la lógica del bot se muestra en un archivo **bot.js** y la información de configuración adicional, en **qnamaker.bot**.

En el archivo **index.js**, lea la información de configuración para generar la instancia del servicio QnA Maker e inicializar el bot.

Actualice el valor de `QNA_CONFIGURATION` con el nombre de la base de conocimiento, tal como aparece en el archivo de configuración.

```js
// QnA Maker knowledge base name as specified in .bot file.
const QNA_CONFIGURATION = '<YOUR_KB_NAME>';

// Get endpoint and QnA Maker configurations by service name.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
const qnaConfig = botConfig.findServiceByNameOrId(QNA_CONFIGURATION);

// Map the contents to the required format for `QnAMaker`.
const qnaEndpointSettings = {
    knowledgeBaseId: qnaConfig.kbId,
    endpointKey: qnaConfig.endpointKey,
    host: qnaConfig.hostname
};

// Create adapter...

// Create the QnAMakerBot.
let bot;
try {
    bot = new QnAMakerBot(qnaEndpointSettings, {});
} catch (err) {
    console.error(`[botInitializationError]: ${ err }`);
    process.exit();
}
```

A continuación, cree el servidor HTTP y escuche las solicitudes entrantes, lo cual generará llamadas a la lógica del bot.

```js
// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open qnamaker.bot file in the emulator.`);
});

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (turnContext) => {
        await bot.onTurn(turnContext);
    });
});
```

---

## <a name="calling-qna-maker-from-your-bot"></a>Llamadas a QnA Maker desde el bot

# <a name="ctabcs"></a>[C#](#tab/cs)

Cuando el bot necesite una respuesta de QnA Maker, llame a `GetAnswersAsync()` desde el código del bot para obtener la respuesta correcta en función del contexto actual. Si va a acceder a su propia base de conocimiento, cambie el mensaje de que _no hay respuestas_ que se muestra a continuación para proporcionar instrucciones útiles para los usuarios.

```csharp
// Check QnA Maker model
var response = await _services.QnAServices[QnAMakerKey].GetAnswersAsync(turnContext);
if (response != null && response.Length > 0)
{
    await turnContext.SendActivityAsync(response[0].Answer, cancellationToken: cancellationToken);
}
else
{
    var msg = @"No QnA Maker answers were found. This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs.
                To see QnA Maker in action, ask the bot questions like 'Why won't it turn on?' or 'I need help'.";
    await turnContext.SendActivityAsync(msg, cancellationToken: cancellationToken);
}

    /// ...
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

En el archivo **bot.js**, pase le entrada del usuario al método `generateAnswer` del servicio QnA Maker para obtener respuestas de la base de conocimiento. Si va a acceder a su propia base de conocimiento, cambie el mensaje de que _no hay respuestas_ y de _bienvenida_ que se muestran a continuación para proporcionar instrucciones útiles para los usuarios.

```javascript
const { ActivityTypes, TurnContext } = require('botbuilder');
const { QnAMaker, QnAMakerEndpoint, QnAMakerOptions } = require('botbuilder-ai');

/**
 * A simple bot that responds to utterances with answers from QnA Maker.
 * If an answer is not found for an utterance, the bot responds with help.
 */
class QnAMakerBot {
    /**
     * The QnAMakerBot constructor requires one argument (`endpoint`) which is used to create an instance of `QnAMaker`.
     * @param {QnAMakerEndpoint} endpoint The basic configuration needed to call QnA Maker. In this sample the configuration is retrieved from the .bot file.
     * @param {QnAMakerOptions} config An optional parameter that contains additional settings for configuring a `QnAMaker` when calling the service.
     */
    constructor(endpoint, qnaOptions) {
        this.qnaMaker = new QnAMaker(endpoint, qnaOptions);
    }

    /**
     * Every conversation turn for our QnAMakerBot will call this method.
     * @param {TurnContext} turnContext Contains all the data needed for processing the conversation turn.
     */
    async onTurn(turnContext) {
        // By checking the incoming Activity type, the bot only calls QnA Maker in appropriate cases.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Perform a call to the QnA Maker service to retrieve matching Question and Answer pairs.
            const qnaResults = await this.qnaMaker.generateAnswer(turnContext.activity.text);

            // If an answer was received from QnA Maker, send the answer back to the user.
            if (qnaResults[0]) {
                await turnContext.sendActivity(qnaResults[0].answer);

            // If no answers were returned from QnA Maker, reply with help.
            } else {
                await turnContext.sendActivity('No QnA Maker answers were found. This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. To see QnA Maker in action, ask the bot questions like "Why won\'t it turn on?" or "I need help."');
            }

        // If the Activity is a ConversationUpdate, send a greeting message to the user.
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate &&
                   turnContext.activity.recipient.id !== turnContext.activity.membersAdded[0].id) {
            await turnContext.sendActivity('Welcome to the QnA Maker sample! Ask me a question and I will try to answer it.');

        // Respond to all other Activity types.
        } else if (turnContext.activity.type !== ActivityTypes.ConversationUpdate) {
            await turnContext.sendActivity(`[${ turnContext.activity.type }]-type activity detected.`);
        }
    }
}

module.exports.QnAMakerBot = QnAMakerBot;
```

---

## <a name="test-the-bot"></a>Probar el bot

Ejecute el ejemplo localmente en la máquina. Si necesita instrucciones, consulte el archivo Léame en el ejemplo de [C#](https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/11.qnamaker) o [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/11.qnamaker/README.md).

En el emulador, envíe el mensaje al bot, tal como se muestra a continuación.

![ejemplo de prueba de qna](~/media/emulator-v4/qna-test-bot.png)


## <a name="next-steps"></a>Pasos siguientes

QnA Maker se puede combinar con otros servicios de Cognitive Services para que el bot sea aún más eficaz. La herramienta de distribución proporciona una forma de combinar QnA con Language Understanding (LUIS) en el bot.

> [!div class="nextstepaction"]
> [Combinación de LUIS y servicios QnA con la herramienta de distribución](./bot-builder-tutorial-dispatch.md)

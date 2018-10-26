---
title: Uso de QnA Maker para responder preguntas | Microsoft Docs
description: Obtenga información sobre cómo usar QnA Maker en el bot.
keywords: pregunta y respuesta, QnA, preguntas más frecuentes, software intermedio
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 10/08/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3d488cc2bb61ef460ed45707596cb7db9e6c23e8
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999090"
---
# <a name="use-qna-maker-to-answer-questions"></a>Uso de QnA Maker para responder preguntas

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Puede usar el servicio QnA Maker para agregar compatibilidad de preguntas y respuestas al bot. Uno de los requisitos básicos para crear su propia instancia del servicio QnA Maker es inicializarlo con preguntas y respuestas. En muchos casos, las preguntas y respuestas ya existen en el contenido como preguntas más frecuentes u otra documentación. En otras ocasiones, le interesará personalizar las respuestas a las preguntas de forma más natural y conversacional.

## <a name="prerequisites"></a>Requisitos previos
- Creación de una cuenta de [QnA Maker](https://www.qnamaker.ai/)
- Descarga del ejemplo de QnA Maker [[C#](https://aka.ms/cs-qna) | [JavaScript](https://aka.ms/js-qna-sample)]

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>Creación de una instancia del servicio QnA Maker y publicación de una base de conocimiento

Después de crear la cuenta de QnA Maker, siga las instrucciones para crear una instancia del [servicio QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure) y la [base de conocimiento](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base). 

Después de publicar la base de conocimiento, debe registrar los siguientes valores para conectar el bot a la base de conocimiento mediante programación.
- En el sitio de [QnA Maker](https://www.qnamaker.ai/), seleccione la base de conocimiento.
- Con la base de conocimiento abierta, seleccione **Configuración**. Registre el valor que se aparece en _nombre del servicio_ como <su_base_de_conocimiento>
- Desplácese hacia abajo hasta encontrar **Detalles de implementación** y anote los siguientes valores:
   - POST /knowledgebases/<id_de_su_base_de_conocimiento>/generateAnswer
   - Host: https://<su_nombre_de_host>.azurewebsites.net/qnamaker
   - Authorization: EndpointKey <su_clave_de_punto_de_conexión>

## <a name="installing-packages"></a>Instalar paquetes

Antes de pasar al código, asegúrese de que tiene los paquetes necesarios para QnA Maker.

# <a name="ctabcs"></a>[C#](#tab/cs)

Agregue el siguiente [paquete NuGet](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui) al bot.

* `Microsoft.Bot.Builder.AI.QnA`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Las características de QnA Maker están en el paquete `botbuilder-ai`. Puede agregar este paquete al proyecto a través de npm:

```shell
npm install --save botbuilder-ai
```

---

## <a name="using-cli-tools-to-update-your-bot-configuration"></a>Actualización de la configuración de .bot mediante las herramientas de la CLI

Un método alternativo para obtener los valores de acceso de la base de conocimiento es usar las herramientas de CLI de BotBuilder [qnamaker](https://aka.ms/botbuilder-tools-qnaMaker) y [msbot](https://aka.ms/botbuilder-tools-msbot-readme) para obtener los metadatos de la base de conocimiento y agregarlos al archivo .bot.

1. Abra un terminal o el símbolo del sistema y vaya al directorio raíz del proyecto del bot.
2. Ejecute `qnamaker init` para crear un archivo de recursos de QnA Maker (**.qnamakerrc**). Se le pedirá la clave de suscripción de QnA Maker.
3. Ejecute el siguiente comando para descargar los metadatos y agregarlos al archivo de configuración del bot.

    ```shell
    qnamaker get kb --kbId <your-kb-id> --msbot | msbot connect qna --stdin [--secret <your-secret>]
    ```
Si ha cifrado el archivo de configuración, deberá proporcionar la clave secreta para actualizar el archivo.

## <a name="using-qna-maker"></a>Uso de QnA Maker
Al inicializar el bot, primero se agrega una referencia a QnA Maker. Luego, se invoca dentro de la lógica de nuestro bot.

# <a name="ctabcs"></a>[C#](#tab/cs)
Abra el ejemplo de QnA Maker que descargó anteriormente. Puede modificar el código según proceda.
En primer lugar, agregue la información necesaria para acceder a la base de conocimiento, incluido el nombre de host, la clave del punto de conexión y el identificador de la base de conocimiento (KbId) en `BotConfiguration.bot`. Estos son los valores que guardó en **Configuración** en la base de conocimiento de QnA Maker.

```json
{
  "name": "QnABotSample",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "id": "1",
      "appPassword": ""
    },
    {
      "type": "qna",
      "name": "QnABot",
      "KbId": "<YOUR_KNOWLEDGE_BASE_ID>",
      "Hostname": "https://<YOUR_HOSTNAME>.azurewebsites.net/qnamaker",
      "EndpointKey": "<YOUR_ENDPOINT_KEY>"
    }
  ],
  "version": "2.0",
  "padlock": ""
}
```

Después, cree una instancia de QnA Maker en `Startup.cs`. De esta manera se toma la información anterior del archivo `BotConfiguration.bot`. Estas cadenas también pueden tener codificación rígida para las pruebas.

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

A continuación, esta instancia de QnA Maker se le proporciona al bot. Abra `QnABot.cs` y, en la parte superior del archivo, agregue el siguiente código. Si va a acceder a su propia base de conocimiento, cambie el mensaje de _bienvenida_ que se muestra a continuación para proporcionar instrucciones iniciales útiles para los usuarios.

```csharp
public class QnABot : IBot
{
    public static readonly string QnAMakerKey = "QnABot";
    private const string WelcomeText = @"This bot will introduce you to QnA Maker.
                                         Ask a quesiton to get started.";
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
Abra el ejemplo de QnA Maker que descargó anteriormente. Puede modificar el código según proceda.
En nuestro ejemplo, el código de inicio está en un archivo **index.js**, el código para la lógica del bot se muestra en un archivo **bot.js** y la información de configuración adicional, en **qnamaker.bot**.

Después de seguir las instrucciones para crear la base de conocimiento y actualizar el archivo **.bot**, el archivo **qnamaker.bot** debe incluir una entrada de servicio para la base de conocimiento de QnA Maker.

```json
{
    "name": "qnamaker",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "id": "1",
            "appId": "",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages"
        },
        {
            "type": "qna",
            "name": "<YOUR_KB_NAME>",
            "kbId": "<YOUR_KNOWLEDGE_BASE_ID>",
            "endpointKey": "<YOUR_ENDPOINT_KEY>",
            "hostname": "https://<YOUR_HOSTNAME>.azurewebsites.net/qnamaker",
            "id": "221"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

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

Haga preguntas al bot para ver las respuestas del servicio QnA Maker. Para más información sobre las pruebas y la publicación de la instancia del servicio QnA, consulte el artículo de QnA Maker sobre la [prueba de una base de conocimiento](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/test-knowledge-base).

## <a name="next-steps"></a>Pasos siguientes

QnA Maker se puede combinar con otros servicios de Cognitive Services para que el bot sea aún más eficaz. La herramienta de distribución proporciona una forma de combinar QnA con Language Understanding (LUIS) en el bot.

> [!div class="nextstepaction"]
> [Combinación de LUIS y servicios QnA con la herramienta de distribución](./bot-builder-tutorial-dispatch.md)

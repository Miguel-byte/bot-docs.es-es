---
title: Tutorial de Azure Bot Service para tener un bot que responde preguntas | Microsoft Docs
description: Tutorial para usar QnA Maker en el bot para responder preguntas.
keywords: QnA Maker, preguntas y respuestas, base de conocimiento
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: tutorial
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b1c34531ee60b2ce9037f42e4f5a7093501cf83a
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360968"
---
# <a name="tutorial-use-qna-maker-in-your-bot-to-answer-questions"></a>Tutorial: Uso de QnA Maker en el bot para responder preguntas

Puede usar el servicio QnA Maker para agregar compatibilidad con preguntas y respuestas al bot. Cuando se crea la base de conocimiento, se inicializa con preguntas y respuestas.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de un servicio QnA Maker y una base de conocimiento
> * Incorporación de información de la base de conocimiento al archivo .bot
> * Actualización del bot para consultar la base de conocimiento
> * Publicación de nuevo del bot

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

* El bot que creó en el [tutorial anterior](bot-builder-tutorial-basic-deploy.md). Se agregará una característica de preguntas y respuestas al bot.
* Será útil cierta familiaridad con QnA Maker. Usaremos el portal de QnA Maker para crear, entrenar y publicar la base de conocimiento que se va a usar con el bot.

Ya debería disponer de los requisitos previos para el tutorial anterior:

[!INCLUDE [deployment prerequisites snippet](~/includes/deploy/snippet-prerequisite.md)]

## <a name="sign-in-to-qna-maker-portal"></a>Inicio de sesión en el portal de QnA Maker

<!-- This and the next step are close duplicates of what's in the QnA How-To -->

Inicie sesión en el [portal de QnA Maker](https://qnamaker.ai/) con las credenciales de Azure.

## <a name="create-a-qna-maker-service-and-knowledge-base"></a>Creación de un servicio QnA Maker y una base de conocimiento

Se importará una definición de la base de conocimiento existente desde el ejemplo de QnA Maker del repositorio [Microsoft/BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples).

1. Clone o copie el repositorio de ejemplos en el equipo.
1. En el portal de QnA Maker, seleccione **Create a knowledge base** (Crear una base de conocimiento).
   1. Si es necesario, cree un servicio QnA. (Puede usar un servicio QnA Maker existente o crear uno nuevo para este tutorial). Para instrucciones más detalladas de QnA Maker, consulte [Creación de un servicio QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure) y [Creación, entrenamiento y publicación de la base de conocimiento de QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base).
   1. Conecte el servicio QnA a la base de conocimiento.
   1. Asigne un nombre a la base de conocimiento.
   1. Para rellenar la base de conocimiento, utilice el archivo **BotBuilder-Samples\samples\csharp_dotnetcore\11.qnamaker\CognitiveModels\smartLightFAQ.tsv** del repositorio de ejemplos.
   1. Haga clic en **Create your kb** (Crear la base de conocimiento) para crear la base de conocimiento.
1. Haga clic en **Save and train** (Guardar y entrenar) la base de conocimiento.
1. Haga clic en **Publish** (Publicar) la base de conocimiento.

   La base de conocimiento ahora está lista para su uso con el bot. Registre el identificador de base de conocimiento, la clave del punto de conexión y el nombre de host. Los necesitará para el siguiente paso.

## <a name="add-knowledge-base-information-to-your-bot-file"></a>Incorporación de información de la base de conocimiento al archivo .bot

Agregue al archivo .bot la información necesaria para acceder a la base de conocimiento.

1. Abra el archivo .bot en un editor.
1. Agregue un elemento `qna` a la matriz `services`.

    ```json
    {
        "type": "qna",
        "name": "<your-knowledge-base-name>",
        "kbId": "<your-knowledge-base-id>",
        "hostname": "<your-qna-service-hostname>",
        "endpointKey": "<your-knowledge-base-endpoint-key>",
        "subscriptionKey": "<your-azure-subscription-key>",
        "id": "<a-unique-id>"
    }
    ```

    | Campo | Valor |
    |:----|:----|
    | Tipo | Debe ser `qna`. Esto indica que esta entrada de servicio describe una base de conocimiento de QnA. |
    | Nombre | Nombre que asignó a la base de conocimiento. |
    | kbId | Identificador de base de conocimiento que el portal de QnA Maker genera automáticamente. |
    | hostname | Dirección URL del host que generó el portal de QnA Maker. Use la dirección URL completa, empezando por `https://` y terminando con `/qnamaker`. |
    | endpointKey | Clave del punto de conexión que el portal de QnA Maker genera automáticamente. |
    | subscriptionKey | Identificador de la suscripción que usó cuando creó el servicio QnA Maker en Azure. |
    | id | Identificador único que no se haya utilizado para alguno de los otros servicios incluidos en el archivo .bot, por ejemplo, "201". |

1. Guarde los cambios.

## <a name="update-your-bot-to-query-the-knowledge-base"></a>Actualización del bot para consultar la base de conocimiento

Actualice el código de inicialización para cargar la información de servicio para la base de conocimiento.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. Agregue el paquete de NuGet **Microsoft.Bot.Builder.AI.QnA** al proyecto.
1. Cambie el nombre de la clase que implementa **IBot** a `QnaBot`.
1. Cambie el nombre de la clase que contiene los descriptores de acceso del bot a `QnaBotAccessors`.
1. En el archivo **Startup.cs**, agregue estas referencias de espacio de nombres.
    ```csharp
    using System.Collections.Generic;
    using System.Linq;
    using Microsoft.Bot.Builder.AI.QnA;
    using Microsoft.Bot.Builder.Integration;
    ```
1. Y modifique el método **ConfigureServices** para inicializar y registrar las bases de conocimiento definidas en el archivo **.bot**. Tenga en cuenta que estas primeras líneas se han movido desde el cuerpo de la llamada `services.AddBot<QnaBot>(options =>` a antes de ella.
    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        var secretKey = Configuration.GetSection("botFileSecret")?.Value;
        var botFilePath = Configuration.GetSection("botFilePath")?.Value;

        // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
        var botConfig = BotConfiguration.Load(botFilePath ?? @".\jfEchoBot.bot", secretKey);
        services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

        // Initialize the QnA knowledge bases for the bot.
        services.AddSingleton(sp => {
            var qnaServices = new List<QnAMaker>();
            foreach (var qnaService in botConfig.Services.OfType<QnAMakerService>())
            {
                qnaServices.Add(new QnAMaker(qnaService));
            }
            return qnaServices;
        });

        services.AddBot<QnaBot>(options =>
        {
            // Retrieve current endpoint.
            // ...
        });

        // Create and register state accessors.
        // ...
    }
    ```
1. En el archivo **QnaBot.cs**, agregue estas referencias de espacio de nombres.
    ```csharp
    using System.Collections.Generic;
    using Microsoft.Bot.Builder.AI.QnA;
    ```
1. Agregue una propiedad `_qnaServices` e inicialícela en el constructor del bot.
    ```csharp
    private readonly List<QnAMaker> _qnaServices;

    /// ...
    public QnaBot(QnaBotAccessors accessors, List<QnAMaker> qnaServices, ILoggerFactory loggerFactory)
    {
        // ...
        _qnaServices = qnaServices;
    }
    ```
1. Modifique el controlador de turnos para consultar las bases de conocimiento registradas con la entrada del usuario. Cuando el bot necesite una respuesta de QnA Maker, llame a `GetAnswersAsync` desde el código del bot para obtener la respuesta correcta en función del contexto actual. Si va a acceder a su propia base de conocimiento, cambie el mensaje de que _no hay respuestas_ que se muestra a continuación para proporcionar instrucciones útiles para los usuarios.
    ```csharp
    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            foreach(var qnaService in _qnaServices)
            {
                var response = await qnaService.GetAnswersAsync(turnContext);
                if (response != null && response.Length > 0)
                {
                    await turnContext.SendActivityAsync(
                        response[0].Answer,
                        cancellationToken: cancellationToken);
                    return;
                }
            }

            var msg = "No QnA Maker answers were found. This example uses a QnA Maker knowledge base that " +
                "focuses on smart light bulbs. Ask the bot questions like 'Why won't it turn on?' or 'I need help'.";

            await turnContext.SendActivityAsync(msg, cancellationToken: cancellationToken);
        }
        else
        {
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
        }
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. Abra un terminal o un símbolo del sistema y vaya al directorio raíz del proyecto.
1. Agregue el paquete de npm **botbuilder-ai** al proyecto.
    ```shell
    npm i botbuilder-ai
    ```
1. En el archivo **index.js**, agregue esta instrucción require.
    ```javascript
    const { QnAMaker } = require('botbuilder-ai');
    ```
1. Lea la información de configuración para generar los servicios QnA Maker.
    ```javascript
    // Read bot configuration from .bot file.
    // ...

    // Initialize the QnA knowledge bases for the bot.
    // Assume each QnA entry in the .bot file is well defined.
    const qnaServices = [];
    botConfig.services.forEach(s => {
        if (s.type == 'qna') {
            const endpoint = {
                knowledgeBaseId: s.kbId,
                endpointKey: s.endpointKey,
                host: s.hostname
            };
            const options = {};
            qnaServices.push(new QnAMaker(endpoint, options));
        }
    });

    // Get bot endpoint configuration by service name
    // ...
    ```
1. Actualice la construcción de bot para pasar los servicios QnA.
    ```javascript
    // Create the bot.
    const myBot = new MyBot(qnaServices);
    ```
1. En el archivo **bot.js**, agregue un constructor.
    ```javascript
    constructor(qnaServices) {
        this.qnaServices = qnaServices;
    }
    ```
1. Además, actualice el controlador de turnos para consultar las bases de conocimiento en busca de una respuesta.
    ```javascript
    async onTurn(turnContext) {
        if (turnContext.activity.type === ActivityTypes.Message) {
            for (let i = 0; i < this.qnaServices.length; i++) {
                // Perform a call to the QnA Maker service to retrieve matching Question and Answer pairs.
                const qnaResults = await this.qnaServices[i].getAnswers(turnContext);

                // If an answer was received from QnA Maker, send the answer back to the user and exit.
                if (qnaResults[0]) {
                    await turnContext.sendActivity(qnaResults[0].answer);
                    return;
                }
            }
            // If no answers were returned from QnA Maker, reply with help.
            await turnContext.sendActivity('No QnA Maker answers were found. '
                + 'This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. '
                + `Ask the bot questions like "Why won't it turn on?" or "I need help."`);
        } else {
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
    }
    ```

---

### <a name="test-the-bot-locally"></a>Prueba local del bot

En este momento el bot debe ser capaz de responder algunas preguntas. Ejecute el bot localmente y ábralo en el emulador.

![ejemplo de prueba de qna](~/media/emulator-v4/qna-test-bot.png)

## <a name="re-publish-your-bot"></a>Publicación de nuevo del bot

Ahora podemos volver a publicar el bot.

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

### <a name="test-the-published-bot"></a>Prueba del bot publicado

Después de publicar el bot, Azure tarda un minuto o dos en actualizar e iniciar el bot.

1. Use el emulador para probar el punto de conexión de producción del bot o utilice Azure Portal para probar el bot en el chat en web.

   En cualquier caso, debería ver el mismo comportamiento que al probar localmente.

## <a name="clean-up-resources"></a>Limpieza de recursos

<!-- In the first tutorial, we should tell them to use a new resource group, so that it is easy to clean up resources. We should also mention in this step in the first tutorial not to clean up resources if they are continuing with the sequence. -->

Si no va a seguir usando esta aplicación, puede eliminar los recursos asociados mediante los siguientes pasos:

1. En Azure Portal, abra el grupo de recursos del bot.
1. Haga clic en **Eliminar grupo de recursos** para eliminar el grupo y todos los recursos que contiene.
1. En el panel de confirmación, escriba el nombre del grupo de recursos y haga clic en **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo agregar características al bot, consulte los artículos de la sección de procedimientos de desarrollo.
> [!div class="nextstepaction"]
> [Botón de pasos siguientes](bot-builder-howto-send-messages.md)

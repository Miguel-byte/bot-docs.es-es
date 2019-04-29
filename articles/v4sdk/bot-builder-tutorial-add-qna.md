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
ms.date: 04/18/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bd29aa1ee56ebf64dc5db2edc47adc3ab250e7d5
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904948"
---
# <a name="tutorial-use-qna-maker-in-your-bot-to-answer-questions"></a>Tutorial: Uso de QnA Maker en el bot para responder preguntas

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

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

También debería disponer ya de estos requisitos previos para el tutorial anterior:

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

## <a name="add-knowledge-base-information-to-your-bot"></a>Incorporación de información de la base de conocimiento al bot
A partir de bot framework 4.3, Azure ya no proporciona un archivo .bot como parte del código fuente del bot descargado. Utilice las siguientes instrucciones para conectar el bot CSharp o JavaScript a la base de conocimiento.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Agregue los valores siguientes al archivo appsetting.json:

```json
{
   "MicrosoftAppId": "",
  "MicrosoftAppPassword": "",
  "ScmType": "None",

  "kbId": "<your-knowledge-base-id>",
  "endpointKey": "<your-knowledge-base-endpoint-key>",
  "hostname": "<your-qna-service-hostname>" // This is a URL
}
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Agregue los valores siguientes al archivo .env:

```javascript
MicrosoftAppId=""
MicrosoftAppPassword=""
ScmType=None

kbId="<your-knowledge-base-id>"
endpointKey="<your-knowledge-base-endpoint-key>"
hostname="<your-qna-service-hostname>" // This is a URL

```

---

    | Campo | Valor |
    |:----|:----|
    | kbId | Identificador de base de conocimiento que el portal de QnA Maker genera automáticamente. |
    | endpointKey | Clave del punto de conexión que el portal de QnA Maker genera automáticamente. |
    | hostname | Dirección URL del host que generó el portal de QnA Maker. Use la dirección URL completa, empezando por `https://` y terminando con `/qnamaker`. |

Ahora, guarde las modificaciones.

## <a name="update-your-bot-to-query-the-knowledge-base"></a>Actualización del bot para consultar la base de conocimiento

Actualice el código de inicialización para cargar la información de servicio para la base de conocimiento.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. Agregue el paquete de NuGet **Microsoft.Bot.Builder.AI.QnA** al proyecto.
1. Agregar el paquete NuGet **Microsoft.Extensions.Configuration** al proyecto.
1. En el archivo **Startup.cs**, agregue estas referencias de espacio de nombres.

   **startup.cs**
   ```csharp
       using Microsoft.Bot.Builder.AI.QnA;
       using Microsoft.Extensions.Configuration;
   ```
1. Y, modifique el método _ConfigureServices_ para crear un objeto QnAMkaerEndpoint que se conecte a la base de conocimiento definida en el archivo **appsettings.json**.

   **startup.cs**
   ```csharp
   // Create QnAMaker endpoint as a singleton
   services.AddSingleton(new QnAMakerEndpoint
   {
      KnowledgeBaseId = Configuration.GetValue<string>($"kbId"),
      EndpointKey = Configuration.GetValue<string>($"endpointKey"),
      Host = Configuration.GetValue<string>($"hostname")
    });

   ```
1. En el archivo **EchoBot.cs**, agregue estas referencias de espacio de nombres.

   **EchoBot.cs**
   ```csharp
   using System.Linq;
   using Microsoft.Bot.Builder.AI.QnA;
   ```

1. Agregue un conector `EchoBotQnA` e inicialícela en el constructor del bot.

   **EchoBot.cs**
   ```csharp
   public QnAMaker EchoBotQnA { get; private set; }
   public EchoBot(QnAMakerEndpoint endpoint)
   {
      // connects to QnA Maker endpoint for each turn
      EchoBotQnA = new QnAMaker(endpoint);
   }
   ```
1. Debajo del método _OnMembersAddedAsync( )_ cree el método _AccessQnAMaker( )_ al agregar el siguiente código:

   **EchoBot.cs**
   ```csharp
   private async Task AccessQnAMaker(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
   {
      var results = await EchoBotQnA.GetAnswersAsync(turnContext);
      if (results.Any())
      {
         await turnContext.SendActivityAsync(MessageFactory.Text("QnA Maker Returned: " + results.First().Answer), cancellationToken);
      }
      else
      {
         await turnContext.SendActivityAsync(MessageFactory.Text("Sorry, could not find an answer in the Q and A system."), cancellationToken);
      }
   }
   ```
1. Ahora dentro de _OnMessageActivityAsync( )_ llame a su nuevo método _AccessQnAMaker( )_ como sigue:

   **EchoBot.cs**
   ```csharp
   protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
   {
      // First send the user input to your QnA Maker knowledgebase
      await AccessQnAMaker(turnContext, cancellationToken);
      ...
   }
   ```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. Abra un terminal o un símbolo del sistema y vaya al directorio raíz del proyecto.
1. Agregue el paquete de npm **botbuilder-ai** al proyecto.
   ```shell
   npm i botbuilder-ai
   ```

1. En **index.js**, después de la sección // Crear adaptador, agregue el siguiente código para leer la información de configuración del archivo .env necesaria para generar los servicios de QnA Maker.

   **index.js**
   ```javascript
   // Map knowledgebase endpoint values from .env file into the required format for `QnAMaker`.
   const configuration = {
      knowledgeBaseId: process.env.kbId,
      endpointKey: process.env.endpointKey,
      host: process.env.hostname
   };

   ```

1. Actualice la construcción del bot para que pase en la información de configuración de los servicios QnA.

   **index.js**
   ```javascript
   // Create the main dialog.
   const myBot = new MyBot(configuration, {}, logger);
   ```

1. En su archivo **bot.js**, agregue este requisito para QnAMaker

   **bot.js**
   ```javascript
   const { QnAMaker } = require('botbuilder-ai');
   ```

1. Modifique el constructor para recibir ahora los parámetros de configuración pasados necesarios para crear un conector QnAMaker y produzca un error si no se proporcionan estos parámetros.

   **bot.js**
   ```javascript
      class MyBot extends ActivityHandler {
         constructor(configuration, qnaOptions) {
            super();
            if (!configuration) throw new Error('[QnaMakerBot]: Missing parameter. configuration is required');
            // now create a qnaMaker connector.
            this.qnaMaker = new QnAMaker(configuration, qnaOptions);
   ```

1. Finalmente, agregue el siguiente código a la llamada onMessage() que pasa cada entrada de usuario a la base de conocimiento de QnA Maker y devuelve la respuesta de QnA Maker al usuario.  para consultar sus bases de conocimiento y obtener una respuesta.
 
    **bot.js**
    ```javascript
   // send user input to QnA Maker.
   const qnaResults = await this.qnaMaker.getAnswers(turnContext);

   // If an answer was received from QnA Maker, send the answer back to the user.
   if (qnaResults[0]) {
      await turnContext.sendActivity(`QnAMaker returned response: ' ${ qnaResults[0].answer}`);
   } 
   else { 
      // If no answers were returned from QnA Maker, reply with help.
      wait turnContext.sendActivity('No QnA Maker response was returned.'
           + 'This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. '
           + `Ask the bot questions like "Why won't it turn on?" or "I need help."`);
   }
   ```
---

### <a name="test-the-bot-locally"></a>Prueba local del bot

En este momento el bot debe ser capaz de responder algunas preguntas. Ejecute el bot localmente y ábralo en el emulador.

![ejemplo de prueba de qna](./media/qna-test-bot.png)

## <a name="re-publish-your-bot"></a>Publicación de nuevo del bot

Ahora podemos volver a publicar el bot en Azure.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish-js.md)]

---

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

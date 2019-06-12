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
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5c8b164aa97ff4ea74acbd6765c72ea0c3f1ad3b
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693650"
---
# <a name="tutorial-use-qna-maker-in-your-bot-to-answer-questions"></a>Tutorial: Uso de QnA Maker en el bot para responder preguntas

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

Puede usar el servicio QnA Maker para agregar compatibilidad con preguntas y respuestas al bot. Cuando se crea la base de conocimiento, se inicializa con preguntas y respuestas.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de un servicio QnA Maker y una base de conocimiento
> * Incorporación de información de la base de conocimiento al archivo de configuración
> * Actualización del bot para consultar la base de conocimiento
> * Nueva publicación del bot

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

* El bot que creó en el [tutorial anterior](bot-builder-tutorial-basic-deploy.md). Se agregará una característica de preguntas y respuestas al bot.
* Es útil conocer algo [QnA Maker](https://qnamaker.ai/). Usaremos el portal de QnA Maker para crear, entrenar y publicar la base de conocimiento que se va a usar con el bot.
* Familiaridad con la [creación de bots QnA](https://aka.ms/azure-create-qna) mediante Azure Bot Service.

También debería disponer ya de los requisitos previos del tutorial anterior.

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

Una vez que se publique la aplicación de QnA Maker, seleccione la pestaña _SETTINGS_ (Configuración) y desplácese hacia abajo hasta "Deployment details" (Detalles de la implementación). Registre los valores siguientes en la solicitud HTTP de _Postman_ de ejemplo.

```text
POST /knowledgebases/<knowledge-base-id>/generateAnswer
Host: <your-hostname>  // NOTE - this is a URL ending in /qnamaker.
Authorization: EndpointKey <qna-maker-resource-key>
```

La cadena completa de la dirección URL del nombre de host se verá como "https://< >.azure.net/qnamaker".

Estos valores se utilizarán en los archivos `appsettings.json` o `.env` en el paso siguiente.

La base de conocimiento ahora está lista para su uso con el bot.

## <a name="add-knowledge-base-information-to-your-bot"></a>Incorporación de información de la base de conocimiento al bot
A partir de Bot Framework 4.3, Azure deja de proporcionar un archivo .bot como parte del código fuente del bot descargado. Utilice las siguientes instrucciones para conectar el bot CSharp o JavaScript a la base de conocimiento.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Agregue los valores siguientes al archivo appsetting.json:

```json
{
   "MicrosoftAppId": "",
  "MicrosoftAppPassword": "",
  "ScmType": "None",
  
  "QnAKnowledgebaseId": "<knowledge-base-id>",
  "QnAAuthKey": "<qna-maker-resource-key>",
  "QnAEndpointHostName": "<your-hostname>" // This is a URL ending in /qnamaker
}
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Agregue los valores siguientes al archivo .env:

```
MicrosoftAppId=""
MicrosoftAppPassword=""
ScmType=None

QnAKnowledgebaseId="<knowledge-base-id>"
QnAAuthKey="<qna-maker-resource-key>"
QnAEndpointHostName="<your-hostname>" // This is a URL ending in /qnamaker
```

---

| Campo | Valor |
|:----|:----|
| QnAKnowledgebaseId | Identificador de base de conocimiento que el portal de QnA Maker genera automáticamente. |
| QnAAuthKey | Clave del punto de conexión que el portal de QnA Maker genera automáticamente. |
| QnAEndpointHostName | Dirección URL del host que generó el portal de QnA Maker. Use la dirección URL completa, empezando por `https://` y terminando con `/qnamaker`. La cadena completa de la dirección URL será similar a esta: "https://< >.azure.net/qnamaker". |

Ahora, guarde las modificaciones.

## <a name="update-your-bot-to-query-the-knowledge-base"></a>Actualización del bot para consultar la base de conocimiento

Actualice el código de inicialización para cargar la información de servicio para la base de conocimiento.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. Agregue el paquete de NuGet **Microsoft.Bot.Builder.AI.QnA** al proyecto.
1. Agregar el paquete NuGet **Microsoft.Extensions.Configuration** al proyecto.
1. En el archivo **Startup.cs**, agregue estas referencias de espacio de nombres.

   **Startup.cs**
   ```csharp
   using Microsoft.Bot.Builder.AI.QnA;
   using Microsoft.Extensions.Configuration;
   ```
1. Y, modifique el método _ConfigureServices_ para crear un objeto QnAMakerEndpoint que se conecte a la base de conocimiento definida en el archivo **appsettings.json**.

   **Startup.cs**
   ```csharp
   // Create QnAMaker endpoint as a singleton
   services.AddSingleton(new QnAMakerEndpoint
   {
      KnowledgeBaseId = Configuration.GetValue<string>($"QnAKnowledgebaseId"),
      EndpointKey = Configuration.GetValue<string>($"QnAAuthKey"),
      Host = Configuration.GetValue<string>($"QnAEndpointHostName")
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
      knowledgeBaseId: process.env.QnAKnowledgebaseId,
      endpointKey: process.env.QnAAuthKey,
      host: process.env.QnAEndpointHostName
   };

   ```

1. Actualice la construcción del bot para que pase en la información de configuración de los servicios QnA.

   **index.js**
   ```javascript
   // Create the main dialog.
   const myBot = new MyBot(configuration, {});
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

1. Por último, agregue el siguiente código a la llamada onMessage() que pasa cada entrada de usuario a la base de conocimiento de QnA Maker y devuelve la respuesta de QnA Maker al usuario para que consulte sus bases de conocimiento, con el fin de encontrar una respuesta.
 
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
      await turnContext.sendActivity('No QnA Maker response was returned.'
           + 'This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. '
           + `Ask the bot questions like "Why won't it turn on?" or "I need help."`);
   }
   ```
---

### <a name="test-the-bot-locally"></a>Prueba local del bot

En este momento el bot debe ser capaz de responder algunas preguntas. Ejecute el bot localmente y ábralo en el emulador.

![ejemplo de prueba de qna](./media/qna-test-bot.png)

## <a name="republish-your-bot"></a>Nueva publicación del bot

Ahora podemos volver a publicar el bot en Azure.

> [!IMPORTANT]
> Antes de comprimir los archivos del proyecto, asegúrese de que está _en_ la carpeta correcta. 
> - Para bots de C#, es la carpeta que contiene el archivo .csproj. 
> - Para bots de JS, es la carpeta que contiene el archivo app.js o index.js. 
>
> Seleccione todos los archivos y comprímalos mientras está en esa carpeta y, a continuación, ejecute el comando también en esa carpeta.
>
> Si la ubicación de la carpeta raíz es incorrecta, el **bot no podrá ejecutarse en Azure Portal**.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)
```cmd
az webapp deployment source config-zip --resource-group <resource-group-name> --name <bot-name-in-azure> --src "c:\bot\mybot.zip"
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish-js.md)]

---

### <a name="test-the-published-bot"></a>Prueba del bot publicado

Después de publicar el bot, Azure tarda un minuto o dos en actualizar e iniciar el bot.

Use el emulador para probar el punto de conexión de producción del bot o utilice Azure Portal para probar el bot en el chat en web.
En cualquier caso, debería ver el mismo comportamiento que al probar localmente.

## <a name="clean-up-resources"></a>Limpieza de recursos

<!-- In the first tutorial, we should tell them to use a new resource group, so that it is easy to clean up resources. We should also mention in this step in the first tutorial not to clean up resources if they are continuing with the sequence. -->

Si no va a seguir usando esta aplicación, puede eliminar los recursos asociados mediante los siguientes pasos:

1. En Azure Portal, abra el grupo de recursos del bot.
1. Haga clic en **Eliminar grupo de recursos** para eliminar el grupo y todos los recursos que contiene.
1. En el panel de confirmación, escriba el nombre del grupo de recursos y haga clic en **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información acerca de cómo agregar características a un bot, consulte el artículo **Envío y recepción de mensajes de texto**, así como los restantes mensajes de la sección de procedimientos de desarrollo.
> [!div class="nextstepaction"]
> [Envío y recepción de mensajes de texto](bot-builder-howto-send-messages.md)

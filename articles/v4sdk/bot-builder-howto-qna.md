---
title: Uso de QnA Maker para responder preguntas | Microsoft Docs
description: Obtenga información sobre cómo usar QnA Maker en el bot.
keywords: pregunta y respuesta, QnA, preguntas más frecuentes, qna maker
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b58307732ae973719231987a35eab6374ea704e4
ms.sourcegitcommit: 312a4593177840433dfee405335100ce59aac347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/12/2019
ms.locfileid: "73933609"
---
# <a name="use-qna-maker-to-answer-questions"></a>Uso de QnA Maker para responder preguntas

[!INCLUDE[applies-to](../includes/applies-to.md)]

QnA Maker proporciona una capa conversacional de preguntas y respuestas sobre los datos. Esto permite que el bot envíe al QnA Maker una pregunta y reciba una respuesta sin tener que analizar e interpretar la intención de la pregunta.

Uno de los requisitos básicos para crear su propia instancia del servicio QnA Maker es inicializarlo con preguntas y respuestas. En muchos casos, las preguntas y respuestas ya existen en el contenido, como las preguntas más frecuentes u otra documentación; otras veces, es posible que quiera personalizar las respuestas a las preguntas de una manera más natural y conversacional.

## <a name="prerequisites"></a>Requisitos previos

- El código de este artículo se basa en el ejemplo de QnA Maker. Necesitará una copia del ejemplo en **[C#](https://aka.ms/cs-qna) o [JavaScript](https://aka.ms/js-qna-sample)** .
- Cuenta de [QnA Maker](https://www.qnamaker.ai/)
- Base de conocimiento de [conceptos básicos de bots](bot-builder-basics.md), [QnA Maker](https://docs.microsoft.com/azure/cognitive-services/qnamaker/overview/overview) y [administración de recursos de bots](bot-file-basics.md).

## <a name="about-this-sample"></a>Acerca de este ejemplo

Para que el bot utilice QnA Maker, primero tendrá que crear una base de conocimiento en [QnA Maker](https://www.qnamaker.ai/), que trataremos en la siguiente sección. El bot puede enviarle la consulta del usuario, y le devolverá la mejor respuesta a la pregunta.

## <a name="ctabcs"></a>[C#](#tab/cs)

![Flujo de lógica de QnABot](./media/qnabot-logic-flow.png)

Se llama a `OnMessageActivityAsync` para cada entrada del usuario recibida. Cuando se le llama, accede a la información `_configuration` almacenada en el archivo `appsetting.json` del código de ejemplo para encontrar el valor para conectarse a su base de conocimiento preconfigurada de QnA Maker.

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

![Flujo de lógica de QnABot JS](./media/qnabot-js-logic-flow.png)

`OnMessage` se llama para cada entrada de usuario recibida. Cuando se le llama, accede al conector `qnamaker` que se ha preconfigurado mediante los valores proporcionados por el archivo `.env` del código de ejemplo.  El método qnamaker `getAnswers` conecta el bot a la base de conocimiento externa de QnA Maker.

---

La entrada del usuario se envía a la base de conocimiento y la mejor respuesta devuelta se muestra al usuario.

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>Creación de una instancia del servicio QnA Maker y publicación de una base de conocimiento

El primer paso es crear un servicio de QnA Maker. Siga los pasos mostrados en la [documentación](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure) de QnA Maker para crear el servicio en Azure.

A continuación, va a crear una base de conocimiento con el archivo `smartLightFAQ.tsv` que se encuentra en la carpeta CognitiveModels del proyecto de ejemplo. Los pasos para crear, entrenar y publicar la [base de conocimiento](https://docs.microsoft.com/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base) de QnA Maker se muestran en la documentación de QnA Maker. Cuando siga estos pasos, asigne el nombre de `qna` a la base de conocimiento y utilice el archivo `smartLightFAQ.tsv` para rellenarla.

> Nota. Este artículo también puede utilizarse para acceder a su propia base de conocimiento de QnA Maker desarrollada por los usuarios.

## <a name="obtain-values-to-connect-your-bot-to-the-knowledge-base"></a>Obtención de valores para conectar el bot a la base de conocimiento

1. En el sitio de [QnA Maker](https://www.qnamaker.ai/), seleccione la base de conocimiento.
1. Con la base de conocimiento abierta, seleccione **Configuración**. Anote el valor que se muestra en _nombre de servicio_. Este valor es útil para buscar la base de conocimiento apropiada en la interfaz del portal de QnA Maker. No se utiliza para conectar la aplicación de bot a esta base de conocimiento.
1. Desplácese hacia abajo hasta encontrar **Detalles de implementación** y anote los siguientes valores de la solicitud HTTP de ejemplo de Postman:
   - POST /knowledgebases/\<knowledge-base-id>/generateAnswer
   - Host: \<nombre-de-host > // dirección URL completa que termina con /qnamaker
   - Autorización: EndpointKey \<clave_de_su_punto_de_conexión>

La cadena completa de la dirección URL del nombre de host se verá como "https://< >.azure.net/qnamaker". Estos tres valores proporcionarán la información necesaria para que su aplicación pueda conectarse a la base de conocimiento de QnA Maker mediante el servicio Azure QnA.  

## <a name="update-the-settings-file"></a>Actualización del archivo de configuración

En primer lugar, agregue la información necesaria para acceder a la base de conocimiento, incluido el nombre de host, la clave del punto de conexión y el identificador de la base de conocimiento (kbId) en el archivo de configuración. Estos son los valores que guardó en la pestaña **Configuración** en la base de conocimiento de QnA Maker.

Si no lo está implementando para la producción, los campos de ID y contraseña de la aplicación pueden dejarse en blanco.

> [!NOTE]
> Si va a agregar acceso a una base de conocimiento de QnA Maker a una aplicación de bot existente, asegúrese de agregar títulos informativos a las entradas de QnA. En esta sección, el valor "name" proporciona la clave necesaria para acceder a esta información desde la aplicación.

## <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="update-your-appsettingsjson-file"></a>Actualización del archivo appsettings.json

[!code-csharp[appsettings](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/appsettings.json)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="update-your-env-file"></a>Actualización del archivo .env

[!code-javascript[.env file](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/.env)]

---

## <a name="set-up-the-qna-maker-instance"></a>Configuración de la instancia de QnA Maker

Primero, creamos un objeto para acceder a la base de conocimiento de QnA Maker.

## <a name="ctabcs"></a>[C#](#tab/cs)

Asegúrese de que el paquete NuGet **Microsoft.Bot.Builder.AI.QnA** está instalado para el proyecto.

En **QnABot.cs**, en el método `OnMessageActivityAsync`, creamos una instancia de QnAMaker. La clase `QnABot` es también donde se introducen los nombres de la información de conexión, guardada en `appsettings.json` arriba. Si ha elegido nombres diferentes para la información de conexión de la base de conocimiento en el archivo de configuración, asegúrese de actualizar los nombres aquí para que reflejen el nombre elegido.

**Bots/QnABot.cs** [!code-csharp[qna connection](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/Bots/QnABot.cs?range=32-39)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Asegúrese de que **botbuilder-ai** del paquete npm esté instalado para el proyecto.

En el ejemplo, el código para la lógica del bot se encuentra en un archivo **QnABot.js**.

En el archivo **QnABot.js**, se utiliza la información de conexión proporcionada por el archivo .env para establecer una conexión con el servicio QnA Maker: _this.qnaMaker_

**bots/QnABot.js** [!code-javascript[QnAMaker](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/bots/QnABot.js?range=12-16)]

---

## <a name="calling-qna-maker-from-your-bot"></a>Llamadas a QnA Maker desde el bot

## <a name="ctabcs"></a>[C#](#tab/cs)

Cuando el bot necesite una respuesta de QnA Maker, llame a `GetAnswersAsync()` desde el código del bot para obtener la respuesta correcta en función del contexto actual. Si va a acceder a su propia base de conocimiento, cambie el mensaje de que _no se encontraron respuestas_ que se muestra a continuación para proporcionar instrucciones útiles para los usuarios.

**Bots/QnABot.cs** [!code-csharp[qna get answers](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/Bots/QnABot.cs?range=43-52)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

En el archivo **QnABot.js**, pase le entrada del usuario al método `getAnswers` del servicio QnA Maker para obtener respuestas de la base de conocimiento. Si QnA Maker devuelve una respuesta, se muestra al usuario. De lo contrario, el usuario recibe el mensaje "No QnA Maker answers were found" (No se encontraron respuestas de QnA Maker).

**bots/QnABot.js** [!code-javascript[OnMessage](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/bots/QnABot.js?range=46-55)]

---

## <a name="test-the-bot"></a>Probar el bot

Ejecute el ejemplo localmente en la máquina. Si aún no lo hizo, instale [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download). Para obtener más instrucciones, consulte el archivo Léame para el [ejemplo en C#](https://aka.ms/cs-qna) o el [ejemplo en Javascript](https://aka.ms/js-qna-sample).

Inicie el emulador, conéctese al bot y envíe un mensaje como se muestra a continuación.

![ejemplo de prueba de qna](../media/emulator-v4/qna-test-bot.png)

## <a name="additional-information"></a>Información adicional

### <a name="multi-turn-prompts"></a>Avisos de múltiples turnos

QnA Maker admite avisos de seguimiento, también conocidos como avisos de múltiples turnos.
Si la base de conocimiento de QnA Maker requiere una respuesta adicional por parte del usuario, QnA Maker envía información de contexto que se puede usar para preguntar al usuario. Esta información también se utiliza para realizar llamadas de seguimiento al servicio QnA Maker.
En la versión 4.6, Bot Framework SDK agregó la compatibilidad con esta característica.

Para crear una base de conocimiento de este tipo, consulte la documentación de QnA Maker sobre cómo [Usar avisos de seguimiento para crear varios turnos de una conversación](https://aka.ms/qnamaker-multiturn-conversation). <!--To learn how to incorporate multi-turn support in your bot, take a look at the QnA Maker Multi-turn [[**C#**](https://aka.ms/cs-qna-multiturn) | [**JS**](https://aka.ms/js-qna-multiturn)] sample.-->

<!--TODO: Update code based on final sample 
The following code snippets come from the proof-of-concept **multi-turn QnA Maker prompts** sample for
[**C#**](https://github.com/microsoft/BotBuilder-Samples/tree/master/experimental/qnamaker-prompting/csharp_dotnetcore) and
[**JavaScript**](https://github.com/microsoft/BotBuilder-Samples/tree/master/experimental/qnamaker-prompting/javascript_nodejs).
-->
<!--
#### Sample code

This sample uses a custom QnA dialog to track state for QnA Maker and handle the user's input and QnA Maker's response. When the user sends a message to the bot, the bot treats the input as either an initial query or a response to a follow-up question. The bot starts or continues its QnA dialog, which tracks the QnA Maker context information.

1. When the dialog **starts**, it makes an initial call to the QnA Maker service. QnA Maker context information **is not** included in the call.
1. When the dialog **continues**, it makes a follow-up call to the QnA Maker service. QnA Maker context information included **is** included in the call.
1. In either case, the dialog evaluates the query results.
   - Each QnA Maker result includes either an answer or a follow-up question to the user's initial query. (This sample uses only the first result.)
   - If the result includes follow-up prompts, the dialog prompts the user, saves the context information, and stays on the dialog stack, waiting for additional information from the user.
   - Otherwise, the result represents an answer, the dialog sends the answer and ends.

This sample implements this across two dialog classes:

- The base _functional dialog_ defines the begin, continue, and state logic for the dialog, notably, the _run state machine_ method.
- The derived _QnA dialog_ defines the logic to call QnA Maker, evaluate its response, and send an answer or follow-up question to the user.

#### QnA Maker context

You need to track context information for the QnA Maker service.
The dialog funnels incoming activities through its _run state machine_ method, which:

1. Gets previous QnA Maker context, if any, from state.
1. Calls its _process_ method to call QnA Maker and generates a response for the user.
1. If the result was a follow-up question, sends the question, saves new QnA Maker context to state, and waits for more input on the next turn.
1. If the result was an answer, sends the answer and ends the dialog.

##### [C#](#tab/csharp)

**Dialogs\FunctionDialogBase.cs** defines the **RunStateMachineAsync** method.

```csharp
private async Task<DialogTurnResult> RunStateMachineAsync(DialogContext dialogContext, CancellationToken cancellationToken)
{
     // Get the Process function's current state from the dialog state
     var oldState = GetPersistedState(dialogContext.ActiveDialog);

     // Run the Process function.
     var (newState, output, result) = await ProcessAsync(oldState, dialogContext.Context.Activity).ConfigureAwait(false);

     // If we have output to send then send it.
     foreach (var activity in output)
     {
          await dialogContext.Context.SendActivityAsync(activity).ConfigureAwait(false);
     }

     // If we have new state then we must still be running.
     if (newState != null)
     {
          // Add the state returned from the Process function to the dialog state.
          dialogContext.ActiveDialog.State[FunctionStateName] = newState;

          // Return Waiting indicating this dialog is still in progress.
          return new DialogTurnResult(DialogTurnStatus.Waiting);
     }
     else
     {
          // The Process function indicates it's completed by returning null for the state.
          return await dialogContext.EndDialogAsync(result).ConfigureAwait(false);
     }
}
```

##### [JavaScript](#tab/javascript)

**dialogs/functionDialogBase.js** defines the **runStateMachine** method.

```javascript
async runStateMachine(dc) {

     var oldState = this.getPersistedState(dc.activeDialog);

     var processResult = await this.processAsync(oldState, dc.context.activity);

     var newState = processResult[0];
     var output = processResult[1];
     var result = processResult[2];

     await dc.context.sendActivity(output);

     if(newState != null){
          dc.activeDialog.state[functionStateName] = newState;
          return { status: DialogTurnStatus.waiting };
     }
     else{
          return await dc.endDialog();
     }
}
```

---

#### QnA Maker input and response

You need to provide any previous context information when you call the QnA Maker service.
The dialog handles the call to QnA Maker in its _process_ method, which:

1. Makes the call to QnA Maker, passing in the previous context, if any.
   - This sample uses a _query QnA service_ helper method to format the parameters and make the call.
   - Importantly, if this is a follow-up call to QnA Maker, the _QnA Maker options_ should include values for the _QnA request context_ and the _QnA question ID_.
1. Gets QnA Maker's response and any follow-up prompts from the top result.
1. If the result was a follow-up question:
   - Generates a hero card that contains the question and options for the user's response.
   - Includes the hero card in a message activity.
   - Returns the message activity and the new QnA Maker context.
1. If the result was an answer, returns a message activity that contains the answer.

You can format a follow-up activity in many ways. This sample uses a hero card. However, using suggested actions would be another option.

##### [C#](#tab/csharp)

**Dialogs\QnADialog.cs** defines the **RunStateMachineAsync** method.

```csharp
protected override async Task<(object newState, IEnumerable<Activity> output, object result)> ProcessAsync(object oldState, Activity inputActivity)
{
     Activity outputActivity = null;
     QnABotState newState = null;

     var query = inputActivity.Text;
     var qnaResult = await _qnaService.QueryQnAServiceAsync(query, (QnABotState)oldState);
     var qnaAnswer = qnaResult[0].Answer;
     var prompts = qnaResult[0].Context?.Prompts;

     if (prompts == null || prompts.Length < 1)
     {
          outputActivity = MessageFactory.Text(qnaAnswer);
     }
     else
     {
          // Set bot state only if prompts are found in QnA result
          newState = new QnABotState
          {
          PreviousQnaId = qnaResult[0].Id,
          PreviousUserQuery = query
          };

          outputActivity = CardHelper.GetHeroCard(qnaAnswer, prompts);
     }

     return (newState, new Activity[] { outputActivity }, null);
}
```

##### [JavaScript](#tab/javascript)

**dialogs/qnaDialog.js** defines the **runStateMachine** method.

```javascript
async processAsync(oldState, activity){

     var newState = null;
     var query = activity.text;
     var qnaResult = await QnAServiceHelper.queryQnAService(query, oldState);
     var qnaAnswer = qnaResult[0].answer;

     var prompts = null;
     if(qnaResult[0].context != null){
          prompts = qnaResult[0].context.prompts;
     }

     var outputActivity = null;
     if(prompts == null || prompts.length < 1){
          outputActivity = MessageFactory.text(qnaAnswer);
     }
     else{
          var newState = {
               PreviousQnaId: qnaResult[0].id,
               PreviousUserQuery: query
          }

          outputActivity = CardHelper.GetHeroCard(qnaAnswer, prompts);
     }

     return [newState, outputActivity , null];
}  
```

---
-->
## <a name="next-steps"></a>Pasos siguientes

QnA Maker se puede combinar con otros servicios de Cognitive Services para que el bot sea aún más eficaz. La herramienta de distribución proporciona una forma de combinar QnA con Language Understanding (LUIS) en el bot.

> [!div class="nextstepaction"]
> [Combinación de LUIS y servicios QnA con la herramienta de distribución](./bot-builder-tutorial-dispatch.md)

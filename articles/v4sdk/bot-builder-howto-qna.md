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
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 15581daa570b9e51ff8f7bec93d16deebcd71d45
ms.sourcegitcommit: 93508adfb79523f610a919b361fc34f5c8dd3eff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/02/2019
ms.locfileid: "67533384"
---
# <a name="use-qna-maker-to-answer-questions"></a>Uso de QnA Maker para responder preguntas

[!INCLUDE[applies-to](../includes/applies-to.md)]

QnA Maker proporciona una capa conversacional de preguntas y respuestas sobre los datos. Esto permite que el bot envíe al QnA Maker una pregunta y reciba una respuesta sin tener que analizar e interpretar la intención de la pregunta. 

Uno de los requisitos básicos para crear su propia instancia del servicio QnA Maker es inicializarlo con preguntas y respuestas. En muchos casos, las preguntas y respuestas ya existen en el contenido, como las preguntas más frecuentes u otra documentación; otras veces, es posible que quiera personalizar las respuestas a las preguntas de una manera más natural y conversacional. 

## <a name="prerequisites"></a>Requisitos previos

- El código de este artículo se basa en el ejemplo de QnA Maker. Necesitará una copia del ejemplo en **[CSharp](https://aka.ms/cs-qna) o [JavaScript](https://aka.ms/js-qna-sample)** .
- Cuenta de [QnA Maker](https://www.qnamaker.ai/)
- Base de conocimiento de [conceptos básicos de bots](bot-builder-basics.md), [QnA Maker](https://docs.microsoft.com/azure/cognitive-services/qnamaker/overview/overview) y [administración de recursos de bots](bot-file-basics.md).

## <a name="about-this-sample"></a>Acerca de este ejemplo

Para que el bot utilice QnA Maker, primero tendrá que crear una base de conocimiento en [QnA Maker](https://www.qnamaker.ai/), que trataremos en la siguiente sección. El bot puede enviarle la consulta del usuario, y le devolverá la mejor respuesta a la pregunta.

## <a name="ctabcs"></a>[C#](#tab/cs)
![Flujo de lógica de QnABot](./media/qnabot-logic-flow.png)

`OnMessageActivityAsync` se llama para cada entrada de usuario recibida. Cuando se le llama, accede a la información `_configuration` almacenada en el archivo `appsetting.json` del código de ejemplo para encontrar el valor para conectarse a su base de conocimiento preconfigurada de QnA Maker. 

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

```json
{
  "MicrosoftAppId": "",
  "MicrosoftAppPassword": "",
  
  "QnAKnowledgebaseId": "<knowledge-base-id>",
  "QnAAuthKey": "<your-endpoint-key>",
  "QnAEndpointHostName": "<your-hostname>"
}
```

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="update-your-env-file"></a>Actualización del archivo .env

```file
MicrosoftAppId=""
MicrosoftAppPassword=""

QnAKnowledgebaseId="<knowledge-base-id>"
QnAAuthKey="<your-endpoint-key>"
QnAEndpointHostName="<your-hostname>"
```

---

## <a name="set-up-the-qna-maker-instance"></a>Configuración de la instancia de QnA Maker

Primero, creamos un objeto para acceder a la base de conocimiento de QnA Maker.

## <a name="ctabcs"></a>[C#](#tab/cs)

Asegúrese de que el paquete NuGet **Microsoft.Bot.Builder.AI.QnA** está instalado para el proyecto.

En **QnABot.cs**, en el método `OnMessageActivityAsync`, creamos una instancia de QnAMaker. La clase `QnABot` es también donde se introducen los nombres de la información de conexión, guardada en `appsettings.json` arriba. Si ha elegido nombres diferentes para la información de conexión de la base de conocimiento en el archivo de configuración, asegúrese de actualizar los nombres aquí para que reflejen el nombre elegido.

**Bots/QnABot.cs**  
[!code-csharp[qna connection](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/Bots/QnABot.cs?range=32-37)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Asegúrese de que **botbuilder-ai** del paquete npm esté instalado para el proyecto.

En el ejemplo, el código para la lógica del bot se encuentra en un archivo **QnABot.js**.

En el archivo **QnABot.js**, se utiliza la información de conexión proporcionada por el archivo .env para establecer una conexión con el servicio QnA Maker: _this.qnaMaker_

**QnAMaker.js** [!code-javascript[QnAMaker](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/bots/QnABot.js?range=19-23)]


---

## <a name="calling-qna-maker-from-your-bot"></a>Llamadas a QnA Maker desde el bot

## <a name="ctabcs"></a>[C#](#tab/cs)

Cuando el bot necesite una respuesta de QnA Maker, llame a `GetAnswersAsync()` desde el código del bot para obtener la respuesta correcta en función del contexto actual. Si va a acceder a su propia base de conocimiento, cambie el mensaje de que _no se encontraron respuestas_ que se muestra a continuación para proporcionar instrucciones útiles para los usuarios.

**QnABot.cs**  
[!code-csharp[qna connection](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/Bots/QnABot.cs?range=43-52)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

En el archivo **QnABot.js**, pase le entrada del usuario al método `getAnswers` del servicio QnA Maker para obtener respuestas de la base de conocimiento. Si QnA Maker devuelve una respuesta, se muestra al usuario. De lo contrario, el usuario recibe el mensaje "No QnA Maker answers were found" (No se encontraron respuestas de QnA Maker). 

**QnABot.js** [!code-javascript[OnMessage](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/bots/QnABot.js?range=43-59)]

---

## <a name="test-the-bot"></a>Probar el bot

Ejecute el ejemplo localmente en la máquina. Si aún no lo hizo, instale [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download). Para obtener más instrucciones, consulte el archivo Léame para el [ejemplo en C#](https://aka.ms/cs-qna) o el [ejemplo en Javascript](https://aka.ms/js-qna-sample).

Inicie el emulador, conéctese al bot y envíe un mensaje como se muestra a continuación.

![ejemplo de prueba de qna](../media/emulator-v4/qna-test-bot.png)

## <a name="next-steps"></a>Pasos siguientes

QnA Maker se puede combinar con otros servicios de Cognitive Services para que el bot sea aún más eficaz. La herramienta de distribución proporciona una forma de combinar QnA con Language Understanding (LUIS) en el bot.

> [!div class="nextstepaction"]
> [Combinación de LUIS y servicios QnA con la herramienta de distribución](./bot-builder-tutorial-dispatch.md)

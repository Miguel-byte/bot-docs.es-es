---
title: Uso de varios modelos de LUIS y QnA | Microsoft Docs
description: Obtenga información sobre cómo usar LUIS y QnA Maker en su bot.
keywords: LUIS, QnA, herramienta de distribución, varios servicios, intenciones de ruta
author: diberry
ms.author: diberry
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c72f978e927f05f430ec94cf747016f4ebc15c5d
ms.sourcegitcommit: 0e6c49964b96c1ac8485ba7afe0daae04b671138
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/01/2019
ms.locfileid: "67492009"
---
# <a name="use-multiple-luis-and-qna-models"></a>Uso de varios modelos de LUIS y QnA

[!INCLUDE[applies-to](../includes/applies-to.md)]

Si un bot utiliza varios modelos LUIS e instancias de QnA Maker Knowledge Base de QnA Maker (Knowledge Base), puede utilizar la herramienta de distribución para determinar qué modelo LUIS o Knowledge Base de QnA Maker es el que mejor se adapta a la entrada del usuario. La herramienta de distribución lo hace mediante la creación de una única aplicación LUIS para dirigir la entrada del usuario al modelo correcto. Para más información sobre la herramienta de distribución, incluidos los comandos de la CLI, consulte el archivo [LÉAME][dispatch-readme].

## <a name="prerequisites"></a>Requisitos previos
- Conocimientos de los [conceptos básicos de bot](bot-builder-basics.md) y [LUIS][howto-luis], and [QnA Maker][howto-qna]. 
- [Herramienta de distribución](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch)
- Una copia de **NLP con distribución** del repositorio de código de [ejemplo en C#][cs-sample] or [JS Sample][js-sample].
- Una cuenta [luis.ai](https://www.luis.ai/) para publicar aplicaciones de LUIS.
- Una cuenta de [QnA Maker](https://www.qnamaker.ai/) para publicar la base de conocimiento de QnA.

## <a name="about-this-sample"></a>Acerca de este ejemplo

Este ejemplo se basa en un conjunto predefinido de aplicaciones de LUIS y QnA Maker.

## <a name="ctabcs"></a>[C#](#tab/cs)

![Flujo lógico del código de ejemplo](./media/tutorial-dispatch/dispatch-logic-flow.png)

Se llama a `OnMessageActivityAsync` para cada entrada del usuario recibida. Este módulo busca la intención del usuario con mayor puntuación y pasa el resultado a `DispatchToTopIntentAsync`. DispatchToTopIntentAsync, a su vez, llama al controlador de la aplicación adecuada.

- `ProcessSampleQnAAsync` -para las preguntas más frecuentes sobre bot.
- `ProcessWeatherAsync` -para las consultas sobre el tiempo.
- `ProcessHomeAutomationAsync` -para los comandos de iluminación doméstica.

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

![Flujo lógico del código de ejemplo](./media/tutorial-dispatch/dispatch-logic-flow-js.png)

Se llama a `onMessage` para cada entrada del usuario recibida. Este módulo busca la intención del usuario con mayor puntuación y pasa el resultado a `dispatchToTopIntentAsync`. DispatchToTopIntentAsync, a su vez, llama al controlador de la aplicación adecuada.

- `processSampleQnA` -para las preguntas más frecuentes sobre bot.
- `processWeather` -para las consultas sobre el tiempo.
- `processHomeAutomation` -para los comandos de iluminación doméstica.

---

El controlador llama al servicio LUIS o QnA Maker y devuelve el resultado generado al usuario.

## <a name="create-luis-apps-and-qna-knowledge-base"></a>Creación de aplicaciones de LUIS y Knowledge Base de QnA
Antes de que pueda crear el modelo de distribución, tendrá que crear y publicar las aplicaciones de LUIS y Knowledge Base de QnA. En este artículo, se publicarán los siguientes modelos que se incluyen con el ejemplo _NLP con distribución_ en la carpeta `\CognitiveModels`: 

| NOMBRE | DESCRIPCIÓN |
|------|------|
| HomeAutomation | Una aplicación LUIS que reconoce una intención de automatización de dispositivos del hogar con datos de la entidad asociada.|
| Tiempo | Una aplicación de LUIS que reconoce las intenciones relacionadas con el tiempo con datos de ubicación.|
| QnAMaker  | Una instancia de Knowledge Base de QnA Maker que proporciona respuestas a preguntas sencillas sobre el bot. |

### <a name="create-luis-apps"></a>Creación de aplicaciones de LUIS
1. Inicie sesión en el [portal web de LUIS](https://www.luis.ai/). En la sección _Mis aplicaciones_, seleccione la pestaña _Importar nueva aplicación_. Aparecerá el siguiente cuadro de diálogo:

    ![Importar archivo json de LUIS](./media/tutorial-dispatch/import-new-luis-app.png)

2. Seleccione el botón _Choose app file_ (Elegir archivo de aplicación), vaya a la carpeta CognitiveModel del código de ejemplo y seleccione el archivo "HomeAutomation.json". Deje en blanco el campo de nombre opcional. 

3. Seleccione _Listo_.

4. Una vez que LUIS abra la aplicación Home Automation, seleccione el botón _Train_ (Entrenar). Esto permitirá entrenar la aplicación mediante el conjunto de expresiones que acaba de importar mediante el archivo "home-automation.json".

5. Cuando finalice el entrenamiento, seleccione el botón _Publish_ (Publicar). Aparecerá el siguiente cuadro de diálogo:

    ![Publicar aplicación de LUIS](./media/tutorial-dispatch/publish-luis-app.png)

6. Elija el entorno "production" (producción) y, después, seleccione el botón _Publish_ (Publicar).

7. Una vez que se ha publicado la nueva aplicación de LUIS, seleccione la pestaña _MANAGE_ (Administrar). En la página "Application Information" (Información de aplicación), registre los valores `Application ID` como "_id-aplicación-para-aplicación_" y `Display name` como "_nombre-de-aplicación_". En la página "Key and Endpoints" (Clave y puntos de conexión), registre los valores `Authoring Key` como "_su-clave-creación-luis_" y `Region` como "_su-región_". Estos valores se utilizarán más adelante en el archivo "appsetting.json".

8. Una vez terminadas, _entrene_ y _publique_ la aplicación meteorológica y la de distribución de LUIS repitiendo los pasos anteriores para el archivo "Weather.json".

### <a name="create-qna-maker-knowledge-base"></a>Creación de una base de conocimiento de QnA Maker

El primer paso para configurar una instancia de Knowledge Base de QnA Maker es configurar un servicio QnA Maker en Azure. Para ello, siga las instrucciones detalladas que se indican [aquí](https://aka.ms/create-qna-maker).

Una vez creado el servicio QnA Maker en Azure, debe registrar la clave _Key 1_ de Cognitive Services proporcionada para el servicio QnA Maker. Esto se usará como \<azure-qna-service-key1> al agregar la aplicación QnA Maker a la aplicación de distribución. Los pasos siguientes le proporcionan esta clave:
    
![Selección de Cognitive Services](./media/tutorial-dispatch/select-qna-cognitive-service.png)

1. Desde Azure Portal, seleccione el servicio QnA Maker de Cognitive Services.

    ![Selección de las claves de Cognitive Services](./media/tutorial-dispatch/select-cognitive-service-keys.png)

1. Seleccione el icono de claves que se encuentra en la sección _Administración de recursos_ en el menú izquierdo.

    ![Selección de Cognitive Service Key1](./media/tutorial-dispatch/select-cognitive-service-key1.png)

1. Copie el valor de _Key 1_ en el Portapapeles y guárdelo en local. Esto se usará como el valor de clave (-k) \<azure-qna-service-key1> al agregar la aplicación QnA Maker a la aplicación de distribución.

1. Ahora, inicie sesión en el [portal web de QnAMaker](https://qnamaker.ai). 

1. En el paso 2, seleccione lo siguiente:

    * La cuenta de Azure AD.
    * El nombre de la suscripción de Azure.
    * El nombre que ha creado para el servicio QnA Maker. (Si el servicio Azure QnA no aparece inicialmente en esta lista desplegable, pruebe a actualizar la página).

    ![Paso 2, crear QnA](./media/tutorial-dispatch/create-qna-step-2.png) 
     

1. En el paso 3, proporcione un nombre para la instancia de Knowledge Base de QnA Maker. En este ejemplo, utilice el nombre "sample-qna".

    ![Paso 3, crear QnA](./media/tutorial-dispatch/create-qna-step-3.png)

1. En el paso 4, seleccione la opción _+ Add File_ (+ Agregar archivo), vaya a la carpeta CognitiveModel del código de ejemplo y seleccione el archivo "QnAMaker.tsv". Hay una selección adicional para agregar una personalidad de _charla_ a la instancia de Knowledge Base, pero nuestro ejemplo no incluye esta opción.

    ![Paso 4, crear QnA](./media/tutorial-dispatch/create-qna-step-4.png)

1. En el paso 5, seleccione _Create your knowledge base_ (Crear una Knowledge Base).

1. Una vez creada una instancia de Knowledge Base a partir del archivo cargado, seleccione _Save and train_ (Guardar y entrenar) y, cuando finalice, seleccione la pestaña _PUBLISH_ (Publicar) y publique la aplicación.

1. Una vez que se publique la aplicación de QnA Maker, seleccione la pestaña _SETTINGS_ (Configuración) y desplácese hacia abajo hasta "Deployment details" (Detalles de la implementación). Registre los valores siguientes en la solicitud HTTP de _Postman_ de ejemplo.

    ```text
    POST /knowledge bases/<knowledge-base-id>/generateAnswer
    Host: <your-hostname>  // NOTE - this is a URL.
    Authorization: EndpointKey <qna-maker-resource-key>
    ```
    
    La cadena completa de la dirección URL del nombre de host se verá como "https://< >.azure.net/qnamaker". Estos valores se utilizarán más adelante en el archivo `appsettings.json` o `.env`.

## <a name="dispatch-app-needs-read-access-to-existing-apps"></a>La aplicación de distribución necesita acceso de lectura a las aplicaciones existentes

La herramienta de distribución necesita acceso de creación para leer las aplicaciones LUIS y QnA Maker existentes con el fin de crear una nueva aplicación LUIS primaria que distribuya a las aplicaciones LUIS y QnA Maker. Este acceso se proporciona con los id. de la aplicación y las claves de creación. Necesita un id. y clave para cada una de las dos aplicaciones LUIS y la aplicación QnA Maker.

|Aplicación|Ubicación de la información|
|--|--|
|LUIS|Id. de la aplicación: se encuentra en el [portal de LUIS](https://www.luis.ai) para cada aplicación, Administrar -> Información de la aplicación<br>Clave de creación: se encuentra en el portal de LUIS, esquina superior derecha, seleccione su propio usuario y, a continuación, Configuración.|
|QnA Maker| Id. de la aplicación: se encuentra en el [portal de QnA Maker](https://http://qnamaker.ai) en la página de configuración después de publicar la aplicación. Este es el identificador encontrado en la primera parte del comando POST después de knowledgebase. Un ejemplo de dónde encontrar el id. de la aplicación es `POST /knowledgebases/{APP-ID}/generateAnswer`.<br>Clave de creación: se encuentra en Azure Portal, para el recurso de QnA Maker, en **Claves**. Solo necesita una de las claves.|

## <a name="create-the-dispatch-model"></a>Creación del modelo de distribución

La interfaz de la CLI para la herramienta de distribución crea el modelo para la distribución a la aplicación LUIS o QnA Maker correcta.

1. Abra un símbolo del sistema o una ventana del terminal y cambie los directorios al directorio **CognitiveModels**.
1. Asegúrese de que tiene la versión actual de npm y la herramienta Dispatch.

    ```cmd
    npm i -g npm
    npm i -g botdispatch
    ```

1. Use `dispatch init` para inicializar y cree un archivo `.dispatch` para el modelo de distribución. Créelo con un nombre de archivo que pueda reconocer.

    ```cmd
    dispatch init -n <filename-to-create> --luisAuthoringKey "<your-luis-authoring-key>" --luisAuthoringRegion <your-region>
    ```

1. Utilice `dispatch add` para agregar las aplicaciones LUIS y las instancias de Knowledge Base de QnA Maker al archivo `.dispatch`.

    ```cmd
    dispatch add -t luis -i "<app-id-for-weather-app>" -n "<name-of-weather-app>" -v <app-version-number> -k "<your-luis-authoring-key>" --intentName l_Weather
    dispatch add -t luis -i "<app-id-for-home-automation-app>" -n "<name-of-home-automation-app>" -v <app-version-number> -k "<your-luis-authoring-key>" --intentName l_HomeAutomation
    dispatch add -t qna -i "<knowledge-base-id>" -n "<knowledge-base-name>" -k "<azure-qna-service-key1>" --intentName q_sample-qna
    ```

1. Use `dispatch create` para generar un modelo de distribución desde el archivo `.dispatch`.

    ```cmd
    dispatch create
    ```

1. Publique la aplicación LUIS de distribución que acaba de crear.

## <a name="use-the-dispatch-luis-app"></a>Uso de la aplicación LUIS de distribución

La aplicación LUIS generada define las intenciones para cada una de las aplicaciones secundarias e instancias de Knowledge Base, así como _ninguna_ intención para cuando la expresión no encaje bien.

- `l_HomeAutomation`
- `l_Weather`
- `None`
- `q_sample-qna`

Tenga en cuenta que estos servicios deben publicarse con los nombres correctos para que el bot funcione correctamente. El bot necesita información sobre los servicios publicados para poder acceder a ellos.

El bot necesita los puntos de conexión de predicción de consulta para las tres aplicaciones LUIS (distribución, tiempo y automatización del hogar) y la única instancia de Knowledge Base de QnA Maker. Use la tabla siguiente para buscar los puntos de conexión:

|Aplicación|Ubicación de clave de punto de conexión de consulta|
|--|--|
|LUIS|En el portal de LUIS, para cada aplicación LUIS, en la sección Administrar, seleccione **Configuración de claves y puntos de conexión** para encontrar las claves asociadas con cada aplicación. Si está siguiendo este tutorial, la clave del punto de conexión es la misma clave que `<your-luis-authoring-key>`. La clave de creación permite 1000 visitas al punto de conexión y después expira.|
|QnA Maker|En el portal de QnA Maker, para la instancia de Knowledge Base, en la opción Administrar, use el valor de clave que se muestra en la configuración de Postman para el encabezado **Autorización**, sin el texto de `EndpointKey `.|

Estos valores se usan en **appsettings.json** para C# y el archivo **.env** para JavaScript.

## <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="installing-packages"></a>Instalación de paquetes

Antes de ejecutar esta aplicación por primera vez, asegúrese de que hay varios paquetes nuget instalados:

**Microsoft.Bot.Builder**

**Microsoft.Bot.Builder.AI.Luis**

**Microsoft.Bot.Builder.AI.QnA**

### <a name="manually-update-your-appsettingsjson-file"></a>Actualización manual del archivo appsettings.json

Una vez creadas todas las aplicaciones de servicio, la información de cada una de ellas debe agregarse al archivo "appsettings.json". El código inicial de [Ejemplo en C#][cs-sample] contiene un archivo appsettings.json vacío:

**appsettings.json**  
[!code-json[AppSettings](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/AppSettings.json?range=8-17)]

Para cada una de las entidades que se muestran a continuación, agregue los valores que ha registrado antes en estas instrucciones:

**appsettings.json**
```json
"MicrosoftAppId": "",
"MicrosoftAppPassword": "",
  
"QnAKnowledgebaseId": "<knowledge-base-id>",
"QnAEndpointKey": "<qna-maker-resource-key>",
"QnAEndpointHostName": "<your-hostname>",

"LuisAppId": "<app-id-for-dispatch-app>",
"LuisAPIKey": "<your-luis-endpoint-key>",
"LuisAPIHostName": "<your-dispatch-app-region>",
```
Cuando se hayan realizado todos los cambios, guarde este archivo.

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="installing-packages"></a>Instalación de paquetes

Antes de ejecutar esta aplicación por primera vez, necesitará instalar varios paquetes npm.

```powershell
npm install --save botbuilder
npm install --save botbuilder-ai
```
El bot necesita incluir un paquete adicional para usar el archivo de configuración .env:

```powershell
npm install --save dotenv
```

### <a name="manually-update-your-env-file"></a>Actualización manual del archivo .env

Una vez creadas todas las aplicaciones de servicio, se tiene que agregar la información de cada una al archivo ".env". El código inicial del [Ejemplo de JavaScript][js-sample] contiene un archivo .env vacío. 

**.env**  
[!code-file[EmptyEnv](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/.env?range=1-10)]

Agregue los valores de conexión de servicio como se muestra a continuación:

**.env**
```file
MicrosoftAppId=""
MicrosoftAppPassword=""

QnAKnowledgebaseId="<knowledge-base-id>"
QnAEndpointKey="<qna-maker-resource-key>"
QnAEndpointHostName="<your-hostname>"

LuisAppId=<app-id-for-dispatch-app>
LuisAPIKey=<your-luis-endpoint-key>
LuisAPIHostName=<your-dispatch-app-region>

```
Cuando se hayan realizado todos los cambios, guarde este archivo.

---

### <a name="connect-to-the-services-from-your-bot"></a>Conexión a los servicios desde el bot

Para conectarse a los servicios Dispatch, LUIS y QnA Maker, el bot extrae información del fichero de configuración (el fichero `appsettings.json` o `.env`).

## <a name="ctabcs"></a>[C#](#tab/cs)

En **BotServices.js** la información contenida en el archivo de configuración _appsettings.json_ se usa para conectar el bot de distribución a los servicios `Dispatch` y `SampleQnA`. Los constructores utilizan los valores que proporcionó para conectarse a estos servicios.

**BotServices.cs**  
[!code-csharp[ReadConfigurationInfo](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/BotServices.cs?range=14-30)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

En **dispatchBot.js** la información contenida en el archivo de configuración _.env_ se usa para conectar el bot de distribución a los servicios _LuisRecognizer(dispatch)_ y _QnAMaker_. Los constructores utilizan los valores que proporcionó para conectarse a estos servicios.

**dispatchBot.js**  
[!code-javascript[ReadConfigurationInfo](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=18-31)]

---

### <a name="call-the-services-from-your-bot"></a>Llamada a los servicios desde el bot

Para cada entrada del usuario, la lógica del bot comprueba la entrada del usuario contra el modelo de Dispatch combinado, encuentra la intención devuelta superior y utiliza esa información para llamar al servicio apropiado para la entrada.

## <a name="ctabcs"></a>[C#](#tab/cs)

En el archivo **DispatchBot.cs** cada vez que se llama al método `OnMessageActivityAsync`, comprobamos el mensaje de usuario entrante con el modelo de Dispatch. Después, se pasan `topIntent` y `recognizerResult` del modelo de Dispatch al método correcto para llamar al servicio y devolver el resultado.

**DispatchBot.cs**  
[!code-csharp[OnMessageActivity](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/bots/DispatchBot.cs?range=26-36)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

En el método `onMessage` de **dispatchBot.js**, comprobamos el mensaje de entrada del usuario contra el modelo de Dispatch, buscamos _topIntent_ y después lo pasamos llamando a _dispatchToTopIntentAsync_.

**dispatchBot.js**  

[!code-javascript[OnMessageActivity](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=37-50)]

---

### <a name="work-with-the-recognition-results"></a>Trabajo con los resultados de reconocimiento

## <a name="ctabcs"></a>[C#](#tab/cs)

Cuando el modelo genera un resultado, indica qué servicio puede procesar de la manera más adecuada la declaración. El código de este bot enruta la solicitud al servicio correspondiente y, a continuación, resume la respuesta del servicio que llama. Dependiendo de la _intención_ devuelta desde Dispatch, este código utiliza la intención devuelto para dirigir al modelo LUIS o servicio QnA correcto.

**DispatchBot.cs**  
[!code-csharp[DispatchToTop](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/bots/DispatchBot.cs?range=51-69)]

Si se llama al método `ProcessHomeAutomationAsync` o `ProcessWeatherAsync`, se pasan los resultados del modelo de Dispatch a _luisResult.ConnectedServiceResult_. El método especificado proporciona los comentarios del usuario que muestran la intención superior del modelo de Dispatch, además de una lista clasificada de todas las intenciones y entidades que se detectaron.

Si se llama al método `q_sample-qna`, utiliza la entrada de usuario contenida en la clase turnContext para generar una respuesta desde la instancia de Knowledge Base y mostrar el resultado al usuario.

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Cuando el modelo genera un resultado, indica qué servicio puede procesar de la manera más adecuada la declaración. El código de este ejemplo utiliza el método _topIntent_ reconocido para mostrar cómo dirigir la solicitud al servicio correspondiente.

**DispatchBot.cs**  
[!code-javascript[DispatchToTop](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=67-83)]

Si se llama al método `processHomeAutomation` o `processWeather`, se pasan los resultados del modelo de Dispatch en _recognizerResult.luisResult_. El método especificado proporciona los comentarios del usuario que muestran la intención superior del modelo de Dispatch, además de una lista clasificada de todas las intenciones y entidades que se detectaron.

Si se llama al método `q_sample-qna`, utiliza la entrada de usuario contenida en la clase turnContext para generar una respuesta desde la instancia de Knowledge Base y mostrar el resultado al usuario.

---

> [!NOTE]
> Si se tratara de una aplicación de producción, aquí es donde los métodos LUIS seleccionados se conectarían al servicio especificado, pasarían la entrada del usuario y procesarían los datos de intención y entidad de LUIS devueltos.

## <a name="test-your-bot"></a>Prueba del bot

1. Con el entorno de desarrollo, inicie el código de ejemplo. Tenga en cuenta la dirección _localhost_ que se muestra en la barra de direcciones de la ventana del explorador que abre la aplicación: "https://localhost:<Port_Number>". 
1. Abra Bot Framework Emulator y, a continuación, seleccione `Create a new bot configuration`. Un archivo `.bot` le permite usar el _Inspector_ en el emulador de bot para ver el JSON devuelto desde LUIS y QnA Maker.
1. En el cuadro de diálogo **New bot configuration** (Nueva configuración de bot), escriba el nombre del bot y la dirección URL del punto de conexión, como `http://localhost:3978/api/messages`. Guarde el archivo en la raíz del proyecto de código de ejemplo de bot.
1. Abra el archivo de bot y agregue secciones para las aplicaciones LUIS y QnA Maker. Use [este archivo de ejemplo](https://github.com/microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/sample-bot-file.json) como plantilla para la configuración. Guarde los cambios.
1. Seleccione el nombre del bot en la lista **Mis Bots** para tener acceso a su bot de ejecución. Como referencia, estas son algunas de las preguntas y los comandos que están cubiertos por los servicios creados para el bot:

    - QnA Maker
      - `hi`, `good morning`
      - `what are you`, `what do you do`
    - LUIS (automatización del hogar)
      - `turn on bedroom light`
      - `turn off bedroom light`
      - `make some coffee`
    - LUIS (tiempo)
      - `whats the weather in redmond washington`
      - `what's the forecast for london`
      - `show me the forecast for nebraska`

## <a name="dispatch-for-user-utterance-to-qna-maker"></a>Distribución de la expresión de usuario a QnA Maker

1. En emulador de bot, escriba el texto `hi` y envíe la expresión. El bot envía esta consulta a la aplicación LUIS de distribución y recibe una respuesta que indica qué aplicación secundaria debe obtener esta instrucción para su posterior procesamiento. 

1. Si selecciona la línea `LUIS Trace` en el registro, puede ver la respuesta de LUIS en el emulador de bot. El resultado de LUIS de la aplicación LUIS de distribución se muestra en Inspector. 

    ```json
    {
      "luisResponse": {
        "entities": [],
        "intents": [
          {
            "intent": "q_sample-qna",
            "score": 0.9489713
          },
          {
            "intent": "l_HomeAutomation",
            "score": 0.0612499453
          },
          {
            "intent": "None",
            "score": 0.008567564
          },
          {
            "intent": "l_Weather",
            "score": 0.0025761195
          }
        ],
        "query": "Hi",
        "topScoringIntent": {
          "intent": "q_sample-qna",
          "score": 0.9489713
        }
      }
    }
    ```
    
    Dado que la expresión, `hi`, forma parte de la intención **q_sample qna** de la aplicación LUIS de distribución y se selecciona como `topScoringIntent`, el bot realizará una segunda solicitud, esta vez a la aplicación de QnA Maker con la misma expresión. 

1. Seleccione la línea `QnAMaker Trace` en el registro del emulador de bot. El resultado de QnA Maker se muestra en Inspector. 

```json
{
    "questions": [
        "hi",
        "greetings",
        "good morning",
        "good evening"
    ],
    "answer": "Hello!",
    "score": 1,
    "id": 96,
    "source": "QnAMaker.tsv",
    "metadata": [],
    "context": {
        "isContextOnly": false,
        "prompts": []
    }
}
```

## <a name="resolving-incorrect-top-intent-from-dispatch"></a>Resolución de la intención superior incorrecta de la distribución

Una vez que el bot se esté ejecutando, es posible mejorar su rendimiento mediante la eliminación de expresiones parecidas o superpuestas entre las aplicaciones distribuidas. Por ejemplo, supongamos que en la aplicación LUIS `Home Automation`, las solicitudes como "encender las luces" se asignan a una intención "TurnOnLights", pero las solicitudes como "¿Por qué no se encienden las luces?" se asignan a una intención "None" para que se puedan pasar a QnA Maker. Estas dos expresiones están demasiado cerca para que la aplicación LUIS de distribución determine si la aplicación secundaria correcta es la aplicación LUIS o la aplicación QnA Maker.

Al combinar la aplicación LUIS y la aplicación QnA Maker mediante la distribución, es preciso llevar a cabo _una_ de las acciones siguientes:

* Quitar la intención "None" de la aplicación LUIS `Home Automation` secundaria y, en su lugar, agregar las expresiones de esa intención a la intención "None" de la aplicación de distribuidor.
* Agregar lógica en su bot para pasar los mensajes que coinciden con la intención "None" de la aplicación LUIS de distribución en el servicio QnA Maker. Comparar la puntuación de la aplicación LUIS de distribución y la puntuación de la aplicación QnA Maker. Usar la puntuación más alta. De este modo se quita QnA Maker del ciclo de distribución. 

Cualquiera de las dos acciones anteriores reducirá el número de veces que el bot responderá a los usuarios con el mensaje "Couldn't find an answer" (No se pudo encontrar una respuesta).

### <a name="to-update-or-create-a-new-luis-model"></a>Para actualizar o crear un nuevo modelo de LUIS

Este ejemplo se basa en un modelo de LUIS preconfigurado. Puede encontrar información adicional que le ayudará a actualizar este modelo o crear un nuevo modelo de LUIS, [aquí](https://aka.ms/create-luis-model#updating-your-cognitive-models).

### <a name="to-delete-resources"></a>Para eliminar recursos

en este ejemplo se crea una serie de aplicaciones y recursos que puede eliminar mediante los pasos que se indican a continuación, pero no debe eliminar los recursos en los que se basan *otras aplicaciones o servicios*.

Para eliminar los recursos de LUIS:

1. Inicie sesión en el portal [luis.ai](https://www.luis.ai).
1. Vaya a la página _My Apps_ (Mis aplicaciones).
1. Seleccione las aplicaciones creadas por este ejemplo.
   - `Home Automation`
   - `Weather`
   - `NLP-With-Dispatch-BotDispatch`
1. Haga clic en _Eliminar_ y haga clic en _Aceptar_ para confirmar.

Para eliminar los recursos de QnA Maker:

1. Inicie sesión en el portal [qnamaker.ai](https://www.qnamaker.ai/).
1. Vaya a la página _My knowledge bases_ (Mis bases de conocimiento).
1. Haga clic en el botón Delete (Eliminar) de la base de conocimiento `Sample QnA` y haga clic en _Delete_ para confirmar.

### <a name="best-practice"></a>Procedimiento recomendado

para mejorar los servicios utilizados en este ejemplo, consulte los procedimientos recomendados para [LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-best-practices) y [QnA Maker](https://docs.microsoft.com/azure/cognitive-services/qnamaker/concepts/best-practices).


[howto-luis]: bot-builder-howto-v4-luis.md
[howto-qna]: bot-builder-howto-qna.md

[cs-sample]: https://aka.ms/dispatch-sample-cs
[js-sample]: https://aka.ms/dispatch-sample-js

[dispatch-readme]: https://aka.ms/botbuilder-tools-dispatch

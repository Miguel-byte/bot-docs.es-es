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
ms.date: 04/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b3e488615f318529935d35dbebbed2dd3b734f62
ms.sourcegitcommit: 3e3c9986b95532197e187b9cc562e6a1452cbd95
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2019
ms.locfileid: "65039746"
---
# <a name="use-multiple-luis-and-qna-models"></a>Uso de varios modelos de LUIS y QnA

[!INCLUDE[applies-to](../includes/applies-to.md)]

Si un bot utiliza varios modelos LUIS y bases de conocimiento de QnA Maker, puede utilizar la herramienta de distribución para determinar qué modelo LUIS o base de conocimiento de QnA Maker es el que mejor se adapta a la entrada del usuario. La herramienta Dispatch lo hace mediante la creación de una única aplicación LUIS para dirigir la entrada del usuario al modelo correcto. Para más información sobre la herramienta de distribución, incluidos los comandos de la CLI, consulte el archivo [LÉAME][dispatch-readme].

## <a name="prerequisites"></a>Requisitos previos
- Base de conocimiento de [conceptos básicos de bot](bot-builder-basics.md), [LUIS][howto-luis] y [QnA Maker][howto-qna]. 
- [Herramienta de distribución](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch)
- Una copia de **NLP con Dispatch** desde el repositorio de código de [ejemplo en C#][cs-sample] o [ejemplo en JS][js-sample]
- Una cuenta [luis.ai](https://www.luis.ai/) para publicar aplicaciones de LUIS.
- Una cuenta de [QnA Maker](https://www.qnamaker.ai/) para publicar la base de conocimiento de QnA.

## <a name="about-this-sample"></a>Acerca de este ejemplo

Este ejemplo se basa en un conjunto predefinido de aplicaciones de LUIS y QnA Maker.

## <a name="ctabcs"></a>[C#](#tab/cs)

![Flujo lógico del código de ejemplo](./media/tutorial-dispatch/dispatch-logic-flow.png)

`OnMessageActivityAsync` se llama para cada entrada de usuario recibida. Este módulo busca la intención del usuario con mayor puntuación y pasa el resultado a `DispatchToTopIntentAsync`. DispatchToTopIntentAsync, a su vez, llama al controlador de la aplicación adecuada.

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

## <a name="create-luis-apps-and-qna-kb"></a>Creación de aplicaciones de LUIS y una base de conocimiento de QnA
Antes de que pueda crear el modelo de Dispatch, tendrá que crear y publicar las aplicaciones de LUIS y la base de conocimiento de QnA. En este artículo, se publicarán los siguientes modelos que se incluyen con el ejemplo _NLP con Dispatch_ en la carpeta `\CognitiveModels`: 

| NOMBRE | DESCRIPCIÓN |
|------|------|
| HomeAutomation | Una aplicación LUIS que reconoce una intención de automatización de dispositivos del hogar con datos de la entidad asociada.|
| Tiempo | Una aplicación LUIS que reconoce las intenciones relacionadas con el tiempo con datos de ubicación.|
| QnAMaker  | Una base de conocimiento de QnA Maker que proporciona respuestas a preguntas sencillas sobre el bot. |

### <a name="create-luis-apps"></a>Creación de aplicaciones de LUIS
1. Inicie sesión en el [portal web de LUIS](https://www.luis.ai/). En la sección _Mis aplicaciones_, seleccione la pestaña _Import new app_ (Importar nueva aplicación). Aparecerá el siguiente cuadro de diálogo:

![Importar archivo json de LUIS](./media/tutorial-dispatch/import-new-luis-app.png)

2. Seleccione el botón _Choose app file_ (Elegir archivo de aplicación), vaya a la carpeta CognitiveModel del código de ejemplo y seleccione el archivo "HomeAutomation.json". Deje en blanco el campo de nombre opcional. 

3. Seleccione _Listo_.

4. Una vez que LUIS abra la aplicación Home Automation, seleccione el botón _Train_ (Entrenar). Esto permitirá entrenar la aplicación mediante el conjunto de expresiones que acaba de importar mediante el archivo "home-automation.json".

5. Cuando finalice el entrenamiento, seleccione el botón _Publish_ (Publicar). Aparecerá el siguiente cuadro de diálogo:

![Publicar aplicación de LUIS](./media/tutorial-dispatch/publish-luis-app.png)

6. Elija el entorno "production" (producción) y, después, seleccione el botón _Publish_ (Publicar).

7. Una vez que se ha publicado la nueva aplicación de LUIS, seleccione la pestaña _MANAGE_ (Administrar). En la página "Application Information" (Información de aplicación), registre los valores `Application ID` como "_id-aplicación-para-aplicación_" y `Display name` como "_nombre-de-aplicación_". En la página "Key and Endpoints" (Clave y puntos de conexión), registre los valores `Authoring Key` como "_su-clave-creación-luis_" y `Region` como "_su-región_". Estos valores se utilizarán más adelante en el archivo "appsetting.json".

8. Una vez terminadas, _entrene_ y _publique_ la aplicación meteorológica y la de distribución de LUIS repitiendo los pasos anteriores para el archivo "Weather.json".

### <a name="create-qna-maker-kb"></a>Creación de la base de conocimiento de QnA Maker

El primer paso para configurar una base de conocimiento de QnA Maker es configurar un servicio QnA Maker en Azure. Para ello, siga las instrucciones detalladas que se indican [aquí](https://aka.ms/create-qna-maker). Ahora, inicie sesión en el [portal web de QnAMaker](https://qnamaker.ai). Vaya al paso 2

![Paso 2, crear QnA](./media/tutorial-dispatch/create-qna-step-2.png)

y seleccione
1. La cuenta de Azure AD.
1. El nombre de la suscripción de Azure.
1. El nombre que ha creado para el servicio QnA Maker. (Si el servicio Azure QnA no aparece inicialmente en esta lista desplegable, pruebe a actualizar la página). 

Vaya al paso 3.

![Paso 3, crear QnA](./media/tutorial-dispatch/create-qna-step-3.png)

Proporcione un nombre para la base de conocimiento de QnA Maker. En este ejemplo vamos a usar el nombre "sample-qna".

Vaya al paso 4

![Paso 4, crear QnA](./media/tutorial-dispatch/create-qna-step-4.png)

Seleccione la opción _+ Add File_ (+ Agregar archivo), vaya a la carpeta CognitiveModel del código de ejemplo y seleccione el archivo "QnAMaker.tsv".

Hay una selección adicional para agregar una personalidad de _charla_ a la base de conocimiento, pero nuestro ejemplo no incluye esta opción.

Vaya al paso 5.

Seleccione _Create your KB_ (Crear la base de conocimiento).

Una vez creada una base de conocimiento a partir del archivo cargado, seleccione _Save and train_ (Guardar y entrenar) y, cuando finalice, seleccione la pestaña _PUBLISH_ (Publicar) y publique la aplicación.

Una vez que se publique la aplicación de QnA Maker, seleccione la pestaña _SETTINGS_ (Configuración) y desplácese hacia abajo hasta "Deployment details" (Detalles de la implementación). Registre los valores siguientes en la solicitud HTTP de _Postman_ de ejemplo.

```text
POST /knowledgebases/<knowledge-base-id>/generateAnswer
Host: <your-hostname>  // NOTE - this is a URL.
Authorization: EndpointKey <your-endpoint-key>
```

La cadena completa de la dirección URL del nombre de host se verá como "https://< >.azure.net/qnamaker".

Estos valores se utilizarán más adelante en el archivo `appsettings.json` o `.env`.

Anote los nombres y las identificaciones de las bases de conocimiento de la aplicación LUIS y QnA Maker. También anote la clave de creación de LUIS y la clave de suscripción a Cognitive Services. Necesitará toda esta información para completar este proceso.

## <a name="create-the-dispatch-model"></a>Creación del modelo de distribución

La interfaz de la CLI para la herramienta Dispatch, crea el modelo para su distribución al servicio correcto.

1. Abra un símbolo del sistema o una ventana del terminal y cambie los directorios al directorio **CognitiveModels**.
1. Asegúrese de que tiene la versión actual de npm y la herramienta Dispatch.

    ```cmd
    npm i -g npm
    npm i -g botdispatch
    ```

1. Use `dispatch init` para inicializar y cree un archivo .dispatch para el modelo de Dispatch. Créelo mediante un nombre de archivo que pueda reconocer.

    ```cmd
    dispatch init -n <filename-to-create> --luisAuthoringKey "<your-luis-authoring-key>" --luisAuthoringRegion <your-region>
    ```

1. Utilice `dispatch add` para agregar las aplicaciones de LUIS y las bases de conocimiento de QnA Maker al archivo .dispatch.

    ```cmd
    dispatch add -t luis -i "<app-id-for-weather-app>" -n "<name-of-weather-app>" -v <app-version-number> -k "<your-luis-authoring-key>" --intentName l_Weather
    dispatch add -t luis -i "<app-id-for-home-automation-app>" -n "<name-of-home-automation-app>" -v <app-version-number> -k "<your-luis-authoring-key>" --intentName l_HomeAutomation
    dispatch add -t qna -i "<knowledge-base-id>" -n "<knowledge-base-name>" -k "<your-cognitive-services-subscription-id>" --intentName q_sample-qna
    ```

1. Use `dispatch create` para generar un modelo de Dispatch desde el archivo .dispatch.

    ```cmd
    dispatch create
    ```

1. Publique la aplicación de distribución de LUIS mediante el archivo JSON del modelo de Dispatch generado.

## <a name="use-the-dispatch-model"></a>Uso del modelo de Dispatch

El modelo generado define las intenciones para cada una de las aplicaciones y bases de conocimiento, así como _ninguna_ intención para cuando la expresión no encaje bien.

- `l_HomeAutomation`
- `l_Weather`
- `None`
- `q_sample-qna`

Tenga en cuenta que estos servicios deben publicarse con los nombres correctos para que el bot funcione correctamente.

El bot necesita información sobre los servicios publicados para poder acceder a ellos.

## <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="installing-packages"></a>Instalación de paquetes

Antes de ejecutar esta aplicación por primera vez, asegúrese de que hay varios paquetes nuget instalados:

**Microsoft.Bot.Builder**

**Microsoft.Bot.Builder.AI.Luis**

**Microsoft.Bot.Builder.AI.QnA**

### <a name="manually-update-your-appsettingsjson-file"></a>Actualización manual del archivo appsettings.json

Una vez creadas todas las aplicaciones de servicio, la información de cada una de ellas debe agregarse al archivo "appsettings.json". El código inicial [C# Sample][cs-sample] contiene un archivo appsettings.json vacío:

**appsettings.json** [!code-json[AppSettings](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/AppSettings.json?range=8-17)]

Para cada una de las entidades que se muestran a continuación, agregue los valores que ha registrado antes en estas instrucciones:

**appsettings.json**
```json
"MicrosoftAppId": "",
"MicrosoftAppPassword": "",
  
"QnAKnowledgebaseId": "<knowledge-base-id>",
"QnAAuthKey": "<your-endpoint-key>",
"QnAEndpointHostName": "<your-hostname>",

"LuisAppId": "<app-id-for-dispatch-app>",
"LuisAPIKey": "<your-luis-authoring-key>",
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

Una vez creadas todas las aplicaciones de servicio, se tiene que agregar la información de cada una al archivo ".env". El código inicial del [ejemplo de JavaScript][js-sample] contiene un archivo .env vacío. 

**.env** [!code-file[EmptyEnv](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/.env?range=1-10)]

Agregue los valores de conexión de servicio como se muestra a continuación:

**.env**
```file
MicrosoftAppId=""
MicrosoftAppPassword=""

QnAKnowledgebaseId="<knowledge-base-id>"
QnAAuthKey="<your-endpoint-key>"
QnAEndpointHostName="<your-hostname>"

LuisAppId=<app-id-for-dispatch-app>
LuisAPIKey=<your-luis-authoring-key>
LuisAPIHostName=<your-dispatch-app-region>

```
Cuando se hayan realizado todos los cambios, guarde este archivo.

---

### <a name="connect-to-the-services-from-your-bot"></a>Conexión a los servicios desde el bot

Para conectarse a los servicios Dispatch, LUIS y QnA Maker, el bot extrae información desde la configuración proporcionada anteriormente.

## <a name="ctabcs"></a>[C#](#tab/cs)

En **BotServices.js** la información contenida en el archivo de configuración _appsettings.json_ se usa para conectar el bot de distribución a los servicios `Dispatch` y `SampleQnA`. Los constructores utilizan los valores que proporcionó para conectarse a estos servicios.

**BotServices.cs** [!code-csharp[ReadConfigurationInfo](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/BotServices.cs?range=14-30)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

En **dispatchBot.js** la información contenida en el archivo de configuración _.env_ se usa para conectar el bot de distribución a los servicios _LuisRecognizer(dispatch)_ y _QnAMaker_. Los constructores utilizan los valores que proporcionó para conectarse a estos servicios.

**dispatchBot.js** [!code-javascript[ReadConfigurationInfo](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=18-31)]

---

### <a name="call-the-services-from-your-bot"></a>Llamada a los servicios desde el bot

Para cada entrada del usuario, la lógica del bot comprueba la entrada del usuario contra el modelo de Dispatch combinado, encuentra la intención devuelta superior y utiliza esa información para llamar al servicio apropiado para la entrada.

## <a name="ctabcs"></a>[C#](#tab/cs)

En el archivo **DispatchBot.cs** cada vez que se llama al método `OnMessageActivityAsync`, comprobamos el mensaje de usuario entrante con el modelo de Dispatch. Después, se pasan `topIntent` y `recognizerResult` del modelo de Dispatch al método correcto para llamar al servicio y devolver el resultado.

**DispatchBot.cs** [!code-csharp[OnMessageActivity](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/bots/DispatchBot.cs?range=26-36)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

En el método `onMessage` de **dispatchBot.js**, comprobamos el mensaje de entrada del usuario contra el modelo de Dispatch, buscamos _topIntent_ y después lo pasamos llamando a _dispatchToTopIntentAsync_.

**dispatchBot.js**

[!code-javascript[OnMessageActivity](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=37-50)]

---

### <a name="work-with-the-recognition-results"></a>Trabajo con los resultados de reconocimiento

## <a name="ctabcs"></a>[C#](#tab/cs)

Cuando el modelo genera un resultado, indica qué servicio puede procesar de la manera más adecuada la declaración. El código de este bot enruta la solicitud al servicio correspondiente y, a continuación, resume la respuesta del servicio que llama. Dependiendo de la _intención_ devuelta desde Dispatch, este código utiliza la intención devuelto para dirigir al modelo LUIS o servicio QnA correcto.

**DispatchBot.cs** [!code-csharp[DispatchToTop](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/bots/DispatchBot.cs?range=51-69)]

Si se llama al método `ProcessHomeAutomationAsync` o `ProcessWeatherAsync`, se pasan los resultados del modelo de Dispatch a _luisResult.ConnectedServiceResult_. El método especificado proporciona los comentarios del usuario que muestran la intención superior del modelo de Dispatch, además de una lista clasificada de todas las intenciones y entidades que se detectaron.

Si se llama al método `q_sample-qna`, utiliza la entrada de usuario contenida en la clase turnContext para generar una respuesta desde la base de conocimiento y mostrar el resultado al usuario.

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Cuando el modelo genera un resultado, indica qué servicio puede procesar de la manera más adecuada la declaración. El código de este ejemplo utiliza el método _topIntent_ reconocido para mostrar cómo dirigir la solicitud al servicio correspondiente.

**DispatchBot.cs** [!code-javascript[DispatchToTop](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=67-83)]

Si se llama al método `processHomeAutomation` o `processWeather`, se pasan los resultados del modelo de Dispatch en _recognizerResult.luisResult_. El método especificado proporciona los comentarios del usuario que muestran la intención superior del modelo de Dispatch, además de una lista clasificada de todas las intenciones y entidades que se detectaron.

Si se llama al método `q_sample-qna`, utiliza la entrada de usuario contenida en la clase turnContext para generar una respuesta desde la base de conocimiento y mostrar el resultado al usuario.

---

> [!NOTE]
> Si se tratara de una aplicación de producción, aquí es donde los métodos LUIS seleccionados se conectarían al servicio especificado, pasarían la entrada del usuario y procesarían los datos de intención y entidad de LUIS devueltos.

## <a name="test-your-bot"></a>Prueba del bot

Con el entorno de desarrollo, inicie el código de ejemplo. Tenga en cuenta la dirección de localhost que se muestra en la barra de direcciones de la ventana del explorador abierta por la aplicación: "https://localhost:<Port_Number>". Después de abrir Bot Framework Emulator, seleccione la prueba azul que se describe a continuación `create new bot configuration`.

![Creación de una nueva configuración](./media/tutorial-dispatch/emulator-create-new-configuration.png)

Escriba la dirección de localhost que ha registrado, mediante la adición de "/api/messages" al final: "https://localhost:<Port_Number>/api/messages"

![Conexión del emulador](./media/tutorial-dispatch/emulator-create-and-connect.png)

Ahora haga clic en el botón `Save and connect` para acceder al bot de ejecución. Como referencia, estas son algunas de las preguntas y los comandos que están cubiertos por los servicios creados para el bot:

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

## <a name="additional-information"></a>Información adicional

Una vez que el bot se esté ejecutando, es posible mejorar su rendimiento mediante la eliminación de expresiones parecidas o superpuestas. Por ejemplo, supongamos que en la aplicación de LUIS `Home Automation` para las solicitudes como "encender las luces" se asignan a una intención "TurnOnLights", pero las solicitudes como "¿Por qué no se encienden las luces?" se asignan a una intención "None" para que se puedan pasar a QnA Maker. Al combinar la aplicación de LUIS y el servicio QnA Maker mediante la herramienta de distribución, es preciso llevar a cabo una de las acciones siguientes:

- Quitar la intención "None" de la aplicación de LUIS `Home Automation` original y, en su lugar, agregar las expresiones de esa intención a la intención "None" de la aplicación de distribuidor.
- Si no quita la intención "None" de la aplicación de LUIS original, tendrá que agregar, en su lugar, lógica en su bot para pasar los mensajes que coinciden con esa intención al servicio QnA Maker.

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

para mejorar los servicios utilizados en este ejemplo, consulte los procedimientos recomendados para [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-best-practices) y [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/concepts/best-practices).


[howto-luis]: bot-builder-howto-v4-luis.md
[howto-qna]: bot-builder-howto-qna.md

[cs-sample]: https://aka.ms/dispatch-sample-cs
[js-sample]: https://aka.ms/dispatch-sample-js

[dispatch-readme]: https://aka.ms/botbuilder-tools-dispatch

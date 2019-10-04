---
title: Incorporación de reconocimiento de lenguaje natural al bot | Microsoft Docs
description: Obtenga información sobre cómo usar LUIS para el reconocimiento del lenguaje natural con Bot Framework SDK.
keywords: Language Understanding, LUIS, reconocimiento de intenciones, entidades, software intermedio
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: cedd6e57ec0490a18b23f22c7895e4ee52dc8509
ms.sourcegitcommit: e9cd857ee11945ef0b98a1ffb4792494dfaeb126
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/01/2019
ms.locfileid: "71694536"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>Incorporación de reconocimiento de lenguaje natural al bot

[!INCLUDE[applies-to](../includes/applies-to.md)]
La tarea de entender qué quiere decir el usuario conversacionalmente y contextualmente puede ser difícil, pero hace que la conversación del bot parezca más natural. Language Understanding, denominado LUIS, le permite hacer precisamente esto, de modo que el bot pueda reconocer la intención de los mensajes de usuario, permitir al usuario emplear un lenguaje más natural y dirigir mejor el flujo de conversación. En este tema se explica cómo agregar LUIS a una aplicación de reserva de vuelos para reconocer las diferentes intenciones y entidades que contiene la entrada del usuario. 

## <a name="prerequisites"></a>Requisitos previos
- Una cuenta de [LUIS](https://www.luis.ai)
- El código de este artículo se basa en el ejemplo de **Core Bot**. Necesitará una copia del ejemplo en **[C#](https://aka.ms/cs-core-sample) o en [JavaScript](https://aka.ms/js-core-sample)** . 
- Conocimientos sobre los [conceptos básicos de los bots](bot-builder-basics.md), el [procesamiento de lenguaje natural](https://docs.microsoft.com/azure/cognitive-services/luis/what-is-luis) y la [administración de recursos de bots](bot-file-basics.md).

## <a name="about-this-sample"></a>Acerca de este ejemplo

En este ejemplo de código de Core Bot se muestra una aplicación de reservas de vuelos. Utiliza un servicio de LUIS para reconocer lo que introduce el usuario y devolver la primera intención de LUIS que reconoce. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
Después de cada procesamiento de la entrada del usuario, `DialogBot` guarda el estado actual tanto de `UserState` como de `ConversationState`. Una vez recopilada toda la información necesaria, el ejemplo de código crea una aplicación de reservas de vuelos de demostración. En este artículo se van a tratarán los aspectos de LUIS de este ejemplo. Sin embargo, el flujo general del ejemplo se muestra a continuación:

- Se llama a `OnMembersAddedAsync` cuando se conecta un usuario nuevo y muestra una tarjeta de bienvenida. 
- Se llama a `OnMessageActivityAsync` para cada entrada del usuario recibida.

![Flujo lógico del ejemplo de LUIS](./media/how-to-luis/luis-logic-flow.png)

El módulo `OnMessageActivityAsync` ejecuta el diálogo adecuado a través del método de extensión del diálogo `Run`. Después, el diálogo principal llama a la aplicación auxiliar de LUIS para buscar la intención del usuario con mayor puntuación. Si la intención con mayor puntuación de la entrada del usuario devuelve "BookFlight", la aplicación auxiliar rellena la información del usuario que LUIS devolvió. Después, el diálogo principal inicia `BookingDialog`, que adquiere la información adicional del usuario que sea necesaria, como:

- `Origin`: la ciudad de origen.
- `TravelDate`: la fecha para la que se reserva el vuelo.
- `Destination`: la ciudad de destino.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
Después de cada procesamiento de la entrada del usuario, `dialogBot` guarda el estado actual tanto de `userState` como de `conversationState`. Una vez recopilada toda la información necesaria, el ejemplo de código crea una aplicación de reservas de vuelos de demostración. En este artículo se van a tratarán los aspectos de LUIS de este ejemplo. Sin embargo, el flujo general del ejemplo se muestra a continuación:

- Se llama a `onMembersAdded` cuando se conecta un usuario nuevo y muestra una tarjeta de bienvenida. 
- Se llama a `OnMessage` para cada entrada del usuario recibida.

![Flujo de lógica de javascript del ejemplo de LUIS](./media/how-to-luis/luis-logic-flow-js.png)

El módulo `onMessage` ejecuta `mainDialog`, que recopila los datos de entrada del usuario.
Después, el diálogo principal llama a la aplicación auxiliar de LUIS `FlightBookingRecognizer` para buscar la intención del usuario con mayor puntuación. Si la intención con mayor puntuación de la entrada del usuario devuelve "BookFlight", la aplicación auxiliar rellena la información del usuario que LUIS devolvió.
Al recibir una respuesta, `mainDialog` conserva la información del usuario que ha devuelto LUIS e inicia `bookingDialog`. `bookingDialog` adquiere la información adicional que sea necesaria, como

- `destination` la ciudad de destino.
- `origin` la ciudad de origen.
- `travelDate` la fecha para la que se reserva el vuelo.

---

Para más información acerca de los otros aspectos del ejemplo, como los diálogos o el estado, consulte [Implementación de un flujo de conversación secuencial](bot-builder-prompts.md) o [Guardado de los datos del usuario y la conversación](bot-builder-howto-v4-state.md).

## <a name="create-a-luis-app-in-the-luis-portal"></a>Creación de una aplicación de LUIS en el portal de LUIS
Inicie sesión en el portal LUIS para crear su propia versión de la aplicación LUIS de ejemplo. Las aplicaciones se pueden crear y administrar en **My Apps** (Mis aplicaciones).

1. Seleccione **Import new app** (Importar aplicación nueva). 
1. Haga clic en **Choose App file (JSON format)...** (Elegir archivo de aplicación [formato JSON]). 
1. Seleccione el archivo `FlightBooking.json` que se encuentra en la carpeta `CognitiveModels` del ejemplo. En el campo de **Optional Name** (Nombre opcional), escriba **FlightBooking**. Este archivo contiene tres intenciones: "Book Flight", "Cancel" y "None". Usaremos estas intenciones para entender qué quiso decir el usuario cuando envió un mensaje al bot.
1. [Entrene](https://docs.microsoft.com/azure/cognitive-services/LUIS/luis-how-to-train) la aplicación.
1. [Publique](https://docs.microsoft.com/azure/cognitive-services/LUIS/publishapp) la aplicación en el entorno de *producción*.

### <a name="why-use-entities"></a>Por qué usar entidades
Las entidades de LUIS permiten que el bot entienda inteligentemente ciertas cosas o eventos que son diferentes a las intenciones estándar. Esto le permite recopilar información adicional del usuario, lo que le permite al bot responder de forma más inteligente o, posiblemente, saltarse ciertas preguntas en las que se le pide al usuario esa información. Además de las definiciones de las tres intenciones de LUIS ("Book Flight", "Cancel" y "None") el archivo FlightBooking.json contiene un conjunto de entidades, como "From.Airport" y "To.Airport". Dichas entidades permiten a LUIS detectar y devolver información adicional que se encuentra dentro de la entrada original del usuario original cuando se solicita una nueva reserva de viaje.

Para más información acerca de cómo aparece la información de entidad en un resultado de LUIS, consulte [Extract data from utterance text with intents and entities](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-data-extraction) (Extracción de datos a partir de texto hablado con intenciones y entidades).

## <a name="obtain-values-to-connect-to-your-luis-app"></a>Obtención de valores para conectarse a la aplicación de LUIS
Una vez que la aplicación de LUIS se ha publicado, puede acceder a ella desde su bot. Para acceder a su aplicación de LUIS desde el bot, deberá registrar varios valores. Para recuperar esa información, puede usar el portal de LUIS.

### <a name="retrieve-application-information-from-the-luisai-portal"></a>Recuperación de información de la aplicación desde el portal de LUIS.ai
El archivo de configuración (`appsettings.json` o `.env`) actúa como el lugar en que se reúnen todas las referencias de servicio. La información que recupere se agregará a este archivo en la siguiente sección. 
1. Seleccione la aplicación de LUIS publicada en [luis.ai](https://www.luis.ai).
1. Con la aplicación de LUIS publicada abierta, seleccione la pestaña **MANAGE** (ADMINISTRAR). ![Administrar la aplicación de LUIS](./media/how-to-luis/manage-luis-app.png)
1. Seleccione la pestaña **Application Information** (Información de la aplicación) a la izquierda y registre el valor mostrado para _Application ID_ (Id. de aplicación) como <SU_ID_DE_APLICACIÓN>.
1. Seleccione la pestaña **Keys and Endpoints** (Claves y puntos de conexión) a la izquierda y registre el valor mostrado para _Authoring Key_ (Clave de creación) como <SU_CLAVE_DE_CREACIÓN>.
1. Desplácese hasta el final de la página, registre el valor que se muestra para _región_ como <SU_REGIÓN>.

### <a name="update-the-settings-file"></a>Actualización del archivo de configuración

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Agregue la información necesaria para acceder a la aplicación de LUIS, lo que incluye el identificador de la aplicación, la clave de creación y la región al archivo `appsettings.json`. Estos son los valores que guardó anteriormente de la aplicación de LUIS publicada. Tenga en cuenta que el nombre de host de la API debe tener el formato `<your region>.api.cognitive.microsoft.com`.

**appsetting.json**  
[!code-json[appsettings](~/../BotBuilder-Samples/samples/csharp_dotnetcore/13.core-bot/appsettings.json?range=1-7)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Agregue la información necesaria para acceder a la aplicación de LUIS, lo que incluye el identificador de la aplicación, la clave de creación y la región al archivo `.env`. Estos son los valores que guardó anteriormente de la aplicación de LUIS publicada. Tenga en cuenta que el nombre de host de la API debe tener el formato `<your region>.api.cognitive.microsoft.com`.

**.env**  
[!code[env](~/../BotBuilder-Samples/samples/javascript_nodejs/13.core-bot/.env?range=1-5)]

---

## <a name="configure-your-bot-to-use-your-luis-app"></a>Configuración del bot para usar la aplicación de LUIS

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Asegúrese de que el paquete NuGet **Microsoft.Bot.Builder.AI.Luis** está instalado para el proyecto.

Para conectar con el servicio LUIS, el bot extrae la información que agregó antes del archivo appsetting.json. La clase `FlightBookingRecognizer` contiene código con la configuración del archivo appsetting.json y consulta el servicio de LUIS mediante una llamada al método `RecognizeAsync`.

**FlightBookingRecognizer.cs**  

[!code-csharp[luisHelper](~/../BotBuilder-Samples/samples/csharp_dotnetcore/13.core-bot/FlightBookingRecognizer.cs?range=12-39)]

`FlightBookingEx.cs` contiene la lógica para a extraer *From*, *To* and *TravelDate*; extiende la clase parcial `FlightBooking.cs` que se usa para almacenar los resultados de LUIS cuando se llama a `FlightBookingRecognizer.RecognizeAsync<FlightBooking>` desde `MainDialog.cs`.

**CognitiveModels\FlightBookingEx.cs**  

[!code-csharp[luis helper](~/../BotBuilder-Samples/samples/csharp_dotnetcore/13.core-bot/CognitiveModels/FlightBookingEx.cs?range=8-35)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Para usar LUIS, el proyecto debe instalar el paquete de npm **botbuilder-ai**.

Para conectar con el servicio LUIS, el bot usa la información que agregó antes del archivo `.env`. La clase `flightBookingRecognizer.js` contiene el código que importa la configuración del archivo `.env` y consulta el servicio de LUIS mediante una llamada al método `recognize()`.

**dialogs/flightBookingRecognizer.js**

[!code-javascript[luis helper](~/../BotBuilder-Samples/samples/javascript_nodejs/13.core-bot/dialogs/flightBookingRecognizer.js?range=6-64)]

La lógica para extraer From, To y TravelDate se implementa como métodos auxiliares dentro de `flightBookingRecognizer.js`. Estos métodos se usan después de llamar a `flightBookingRecognizer.executeLuisQuery()` desde `mainDialog.js`.

---

LUIS ya está configurado y conectado para el bot.

## <a name="test-the-bot"></a>Probar el bot

Descargue e instale la versión más reciente de [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).

1. Ejecute el ejemplo localmente en la máquina. Si necesita instrucciones, consulte el archivo Léame en los ejemplos de [C#](https://aka.ms/cs-core-sample) o [JS](https://aka.ms/js-core-sample).

1. En el emulador, escriba un mensaje como "travel to paris" (viajar a París) o "going from paris to berlin" (ir de París a Berlín). Utilice cualquier expresión que se encuentra en el archivo FlightBooking.json para entrenar la intención "Book flight".

![Entrada de reserva de LUIS](./media/how-to-luis/luis-user-travel-input.png)

Si la intención superior que se devuelve desde LUIS se resuelve como "Book flight", el bot realizará preguntas adicionales hasta que tenga suficiente información almacenada para crear la reserva de un viaje. En ese momento devolverá la información de dicha reserva a su usuario. 

![Resultado de reserva de LUIS](./media/how-to-luis/luis-travel-result.png)

En ese momento se restablecerá la lógica del bot del código y podrá crear más reservas. 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Uso de QnA Maker para responder a preguntas](./bot-builder-howto-qna.md)

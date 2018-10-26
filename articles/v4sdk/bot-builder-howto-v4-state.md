---
title: Administración del estado de la conversación y del usuario | Microsoft Docs
description: Obtenga información sobre cómo guardar y recuperar datos con el SDK de Bot Builder para .NET.
keywords: estado de la conversación, estado de usuario, flujo de la conversación
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 21f864ba6f5beba5205e860f4a56697997048dfb
ms.sourcegitcommit: 6c2426c43cd2212bdea1ecbbf8ed245145b3c30d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/08/2018
ms.locfileid: "48852300"
---
# <a name="manage-conversation-and-user-state"></a>Administración del estado de la conversación y del usuario

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Una clave del diseño correcto de bots consiste en realizar el seguimiento del contexto de una conversación, para que el bot recuerde cosas como las respuestas a preguntas anteriores. En función del uso previsto del bot, es posible que incluso sea necesario realizar el seguimiento de la información de estado o almacenamiento durante más tiempo que la duración de la conversación. El *estado* de un bot es la información que recuerda con el fin de responder de forma adecuada a los mensajes entrantes. El SDK Bot Builder proporciona dos clases para almacenar y recuperar datos de estado como un objeto asociado a un usuario o una conversación.

- El **estado de la conversación** ayuda al bot a realizar el seguimiento de la conversación actual que tiene con el usuario. Si es necesario que el bot complete una secuencia de pasos o que cambie entre temas de conversación, se pueden usar las propiedades de la conversación para administrar los pasos en una secuencia o realizar el seguimiento del tema actual. 

- El **estado de usuario** se puede usar para muchos propósitos, como determinar dónde dejó la conversación anterior el usuario o simplemente saludar a un usuario por su nombre cuando regrese. Si almacena las preferencias del usuario, puede usar esa información para personalizar la conversación la próxima vez que chatee. Por ejemplo, podría alertar al usuario sobre un artículo de noticias que trate un tema que le interese, o bien cuando haya una cita disponible. 

Las clases `ConversationState` y `UserState` son clases de estado que son especializaciones de la clase `BotState` con directivas que controlan la vigencia y el alcance de los objetos almacenados en ellas. Componentes que necesitan almacenar el estado, crean y registran una propiedad con un tipo, y usan el descriptor de acceso de la propiedad para acceder al estado. El administrador de estado del bot puede utilizar el almacenamiento de memoria, CosmosDB y Blob Storage. 

> [!NOTE] 
> Utilice el administrador de estado del bot para almacenar las preferencias, el nombre de usuario o lo último que haya pedido, pero no lo utilice para almacenar datos empresariales críticos. Para los datos críticos, cree sus propios componentes de almacenamiento o escríbalos directamente en el [almacenamiento](bot-builder-howto-v4-storage.md).
> El almacenamiento de datos en memoria solo está pensado para realizar pruebas. Este tipo de almacenamiento es volátil y temporal. Los datos se borran cada vez que se reinicia el bot.

## <a name="using-conversation-state-and-user-state-to-direct-conversation-flow"></a>Uso del estado de la conversación y estado de usuario para dirigir el flujo de la conversación
Al diseñar un flujo de conversación, es útil definir un indicador de estado para dirigir el flujo de la conversación. La marca puede ser un simple tipo booleano o un tipo que incluya el nombre del tema actual. La marca puede ayudar a realizar el seguimiento de dónde se encuentra en una conversación. Por ejemplo, un indicador de tipo booleano puede indicar si se está en una conversación o no, mientras que una propiedad de nombre de tema puede indicar en qué conversación se está actualmente.



# <a name="ctabcsharp"></a>[C#](#tab/csharp)
### <a name="conversation-and-user-state"></a>Estado de la conversación y de usuario
Puede usar el [ejemplo Echo Bot With Counter](https://aka.ms/EchoBot-With-Counter) como punto inicial de este tema de procedimientos. En primer lugar, cree la clase `TopicState` para administrar el tema actual de la conversación en `TopicState.cs`, tal como se muestra a continuación:

```csharp
public class TopicState
{
   public string Prompt { get; set; } = "askName";
}
``` 
Después, cree la clase `UserProfile` en `UserProfile.cs` para administrar el estado de usuario.
```csharp
public class UserProfile
{
    public string UserName { get; set; }
    public string TelephoneNumber { get; set; }
}
``` 
La clase `TopicState` tiene una marca para realizar un seguimiento de dónde estamos en la conversación y utiliza el estado de la conversación para almacenarla. El aviso se inicializa como "askName" para iniciar la conversación. Cuando el bot recibe la respuesta del usuario, el aviso se redefinirá como "askNumber" para iniciar la siguiente conversación. La clase `UserProfile` realiza el seguimiento del nombre de usuario y número de teléfono y los almacena en el estado de usuario.

### <a name="property-accessors"></a>Descriptores de acceso de propiedad
La clase `EchoBotAccessors` en nuestro ejemplo se crea como un singleton y se pasa al constructor `class EchoWithCounterBot : IBot` a través de la inserción de dependencias. El constructor de la clase `EchoBotAccessors` inicializa una nueva instancia de la clase `EchoBotAccessors`. Contiene las clases `ConversationState`, `UserState` y `IStatePropertyAccessor` asociadas. El objeto `conversationState` almacena el estado del tema y el objeto `userState` que almacena la información de perfil de usuario. Los objetos `ConversationState` y `UserState` se crean en el archivo Startup.cs. Los objetos de conversación y de estado de usuario son los que persisten en la conversación y en el ámbito del usuario. 

Actualice el constructor para incluir `UserState`, tal como se muestra a continuación:
```csharp
using EchoBotWithCounter;

public EchoBotAccessors(ConversationState conversationState, UserState userState)
{
    ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    UserState = userState ?? throw new ArgumentNullException(nameof(userState));
}
```
Cree nombres únicos para los descriptores de acceso `TopicState` y `UserProfile`.
```csharp
public static string UserProfileName { get; } = $"{nameof(EchoBotAccessors)}.UserProfile";
public static string TopicStateName { get; } = $"{nameof(EchoBotAccessors)}.TopicState";
```
Después, defina dos descriptores de acceso. El primero almacena el tema de la conversación y el segundo almacena el nombre del usuario y el número de teléfono.
```csharp
public IStatePropertyAccessor<TopicState> TopicState { get; set; }
public IStatePropertyAccessor<UserProfile> UserProfile { get; set; }
```

Las propiedades para obtener el estado de conversación ya están definidas, pero necesitará agregar `UserState`, como se muestra a continuación:
```csharp
public ConversationState ConversationState { get; }
public UserState UserState { get; }
```
Después de realizar los cambios, guarde el archivo. A continuación, vamos a actualizar la clase Startup para crear el objeto `UserState` para que conserve cualquier cosa en el ámbito de usuario. `ConversationState` ya existe. 
```csharp

services.AddBot<EchoWithCounterBot>(options =>
{
    ...

    IStorage dataStore = new MemoryStorage();
    
    var conversationState = new ConversationState(dataStore);
    options.State.Add(conversationState);
        
    var userState = new UserState(dataStore);  
    options.State.Add(userState);
});
```
La línea `options.State.Add(ConversationState);` y `options.State.Add(userState);` agregan el estado de la conversación y el estado de usuario, respectivamente. Después, cree y registre los descriptores de acceso de estado. Los descriptores de acceso creados aquí se pasan a la clase derivada de IBot en cada turno. Modifique el código para incluir el estado de usuario, tal como se muestra a continuación:
```csharp
services.AddSingleton<EchoBotAccessors>(sp =>
{
   ...
    var userState = options.State.OfType<UserState>().FirstOrDefault();
    if (userState == null)
    {
        throw new InvalidOperationException("UserState must be defined and added before adding user-scoped state accessors.");
    }
   ...
 });
```

A continuación, cree los dos descriptores de acceso mediante `TopicState` y `UserProfile`, y páselos a la clase `class EchoWithCounterBot : IBot` mediante la inserción de dependencias.
```csharp
services.AddSingleton<EchoBotAccessors>(sp =>
{
   ...
    var accessors = new BotAccessors(conversationState, userState)
    {
        TopicState = conversationState.CreateProperty<TopicState>("TopicState"),
        UserProfile = userState.CreateProperty<UserProfile>("UserProfile"),
     };

     return accessors;
 });
```

La conversación y el estado del usuario se vinculan a un singleton mediante el bloque de código `services.AddSingleton` y se guardan mediante un descriptor de acceso de almacenamiento de estado en el código que comienza con `var accessors = new BotAccessor(conversationState, userState)`.

### <a name="use-conversation-and-user-state-properties"></a>Uso de propiedades de estado de conversación y usuario 
En el controlador `OnTurnAsync` de la clase `EchoWithCounterBot : IBot`, modifique el código para solicitar el nombre de usuario y después el número de teléfono. Para realizar el seguimiento de dónde estamos en la conversación, usamos la propiedad Prompt que se define en el estado del tema. Esta propiedad se ha inicializado como "askName". Cuando se obtiene el nombre de usuario, se establece en "askNumber" y se configura con el nombre de usuario escrito. Después de recibir el número de teléfono, envía un mensaje de confirmación y establece la pregunta en "confirmación" porque se encuentra al final de la conversación.

```csharp
using EchoBotWithCounter;

if (turnContext.Activity.Type == ActivityTypes.Message)
{
    // Get the conversation state from the turn context.
    var convo = await _accessors.TopicState.GetAsync(turnContext, () => new TopicState());
    
    // Get the user state from the turn context.
    var user = await _accessors.UserProfile.GetAsync(turnContext, () => new UserProfile());
    
    // Ask user name. The Prompt was initialiazed as "askName" in the TopicState.cs file.
    if (convo.Prompt == "askName")
    {
        await turnContext.SendActivityAsync("What is your name?");
        
        // Set the Prompt to ask the next question for this conversation
        convo.Prompt = "askNumber";
        
        // Set the property using the accessor
        await _accessors.TopicState.SetAsync(turnContext, convo);
        
        //Save the new prompt into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
    else if (convo.Prompt == "askNumber")
    {
        // Set the UserName that is defined in the UserProfile class
        user.UserName = turnContext.Activity.Text;
        
        // Use the user name to prompt the user for phone number
        await turnContext.SendActivityAsync($"Hello, {user.UserName}. What's your telephone number?");
        
        // Set the Prompt now that we have collected all the data
        convo.Prompt = "confirmation";
                 
        await _accessors.TopicState.SetAsync(turnContext, convo);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        await _accessors.UserProfile.SetAsync(turnContext, user);
        await _accessors.UserState.SaveChangesAsync(turnContext);
    }
    else if (convo.Prompt == "confirmation")
    { 
        // Set the TelephoneNumber that is defined in the UserProfile class
        user.TelephoneNumber = turnContext.Activity.Text;

        await turnContext.SendActivityAsync($"Got it, {user.UserName}. I'll call you later.");

        // reset initial prompt state
        convo.Prompt = "askName"; // Reset for a new conversation.
        
        await _accessors.TopicState.SetAsync(turnContext, convo);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
}
```   

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="conversation-and-user-state"></a>Estado de la conversación y de usuario

Puede usar el [ejemplo Echo Bot With Counter](https://aka.ms/EchoBot-With-Counter-JS) como punto inicial de este tema de procedimientos. Este ejemplo ya utiliza el objeto `ConversationState` para almacenar el número de mensajes. Necesitaremos agregar un objeto `TopicStates` para realizar el seguimiento del estado de conversación y el objeto `UserState` para realizar el seguimiento de la información del usuario en un objeto `userProfile`. 

En el archivo `index.js` del bot principal, agregue el objeto `UserState` a la lista requerida:

**index.js**

```javascript
// Import required bot services. See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

Después, cree `UserState` mediante `MemoryStorage` como proveedor de almacenamiento y páselo como segundo argumento de la clase `MainDialog`.

**index.js**

```javascript
// Create conversation state with in-memory storage provider. 
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);
// Create the main bot.
const bot = new EchBot(conversationState, userState);
```

En el archivo `bot.js`, actualice el constructor para que acepte `userState` como segundo argumento. Después, cree una propiedad `topicState` desde `conversationState` y cree una propiedad `userProfile` desde `userState`.

**bot.js**

```javascript
const TOPIC_STATE = 'topic';
const USER_PROFILE = 'user';

constructor (conversationState, userState) {
    // creates a new state accessor property.see https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors 
    this.conversationState = conversationState;
    this.topicState = this.conversationState.createProperty(TOPIC_STATE);

    // User state
    this.userState = userState;
    this.userProfile = this.userState.createProperty(USER_PROFILE);
}
```

### <a name="use-conversation-and-user-state-properties"></a>Uso de propiedades de estado de conversación y usuario

En el controlador `onTurn` de la clase `MainDialog`, modifique el código para solicitar el nombre de usuario y después el número de teléfono. Para realizar el seguimiento de dónde estamos en la conversación, usamos la propiedad `prompt` que se define en `topicState`. Esta propiedad se inicializa en "askName". Cuando se obtiene el nombre de usuario, se establece en "askNumber" y se configura con el nombre de usuario escrito. Después de recibir el número de teléfono, envía un mensaje de confirmación y establece la pregunta en `undefined` porque se encuentra al final de la conversación.

**dialogs/mainDialog/index.js**

```javascript
// see https://aka.ms/about-bot-activity-message to learn more about the message and other activity types
if (turnContext.activity.type === 'message') {
    // read from state and set default object if object does not exist in storage.
    let topicState = await this.topicState.get(turnContext, {
        //Define the topic state object
        prompt: "askName"
    });
    let userProfile = await this.userProfile.get(turnContext, {  
        // Define the user's profile object
        "userName": "",
        "telephoneNumber": ""
    });

    if(topicState.prompt == "askName"){
        await turnContext.sendActivity("What is your name?");

        // Set next prompt state
        topicState.prompt = "askNumber";

        // Update state
        await this.topicState.set(turnContext, topicState);
    }
    else if(topicState.prompt == "askNumber"){
        // Set the UserName that is defined in the UserProfile class
        userProfile.userName = turnContext.activity.text;

        // Use the user name to prompt the user for phone number
        await turnContext.sendActivity(`Hello, ${userProfile.userName}. What's your telephone number?`);

        // Set next prompt state
        topicState.prompt = "confirmation";

        // Update states
        await this.topicState.set(turnContext, topicState);
        await this.userProfile.set(turnContext, userProfile);
    }
    else if(topicState.prompt == "confirmation"){
        // Set the phone number
        userProfile.telephoneNumber = turnContext.activity.text;

        // Sent confirmation
        await turnContext.sendActivity(`Got it, ${userProfile.userName}. I'll call you later.`)

        // reset initial prompt state
        topicState.prompt = "askName"; // Reset for a new conversation

        // Update states
        await this.topicState.set(turnContext, topicState);
        await this.userProfile.set(turnContext, userProfile);
    }
    
    // Save state changes to storage
    await this.conversationState.saveChanges(turnContext);
    await this.userState.saveChanges(turnContext);
    
}
else {
    await turnContext.sendActivity(`[${context.activity.type} event detected]`);
}
```

---

## <a name="start-your-bot"></a>Inicio del bot
Ejecute el bot localmente.

### <a name="start-the-emulator-and-connect-your-bot"></a>Inicio del emulador y conexión del bot
A continuación, inicie el emulador y, después, conéctese al bot en el emulador:

1. Haga clic en el vínculo **Open bot** (Abrir bot) de la pestaña de bienvenida del emulador. 
2. Seleccione el archivo .bot ubicado en el directorio donde se creó la solución de Visual Studio.

### <a name="interact-with-your-bot"></a>Interacción con el bot

Envíe un mensaje al bot y este responderá con un mensaje.
![Emulador en ejecución](../media/emulator-v4/emulator-running.png)

Si decide administrar usted mismo el estado, consulte [Petición de datos de entrada a los usuarios mediante sus propios mensajes](bot-builder-primitive-prompts.md). Una alternativa consiste en usar el cuadro de diálogo de cascada. El cuadro de diálogo realiza el seguimiento del estado de la conversación de forma automática, por lo que no es necesario crear marcas para realizar el seguimiento del estado. Para más información, consulte [Administración de conversaciones simples con diálogos](bot-builder-dialog-manage-conversation-flow.md).

## <a name="next-steps"></a>Pasos siguientes
Ahora que sabe cómo usar el estado para leer y escribir datos de bot en el almacenamiento, veremos cómo puede leer y escribir directamente en el almacenamiento.

> [!div class="nextstepaction"]
> [Escritura directa en el almacenamiento](bot-builder-howto-v4-storage.md).

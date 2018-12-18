---
title: Guardado de los datos del usuario y la conversación | Microsoft Docs
description: Obtenga información sobre cómo guardar y recuperar datos de estado con el SDK Bot Builder.
keywords: estado de la conversación, estado de usuario, conversación, guardar el estado, administrar estado del bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/26/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 8c3aad54a9e80e8a046a6e31a5109a1de8c61a8b
ms.sourcegitcommit: 91156d0866316eda8d68454a0c4cd74be5060144
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/07/2018
ms.locfileid: "53010510"
---
# <a name="save-user-and-conversation-data"></a>Guardado de los datos del usuario y la conversación

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Inherentemente, un bot no tiene estado. Una vez implementado el bot, no se puede ejecutar en el mismo proceso o en el mismo equipo de turno a otro. Sin embargo, el bot debe poder hacer un seguimiento del contexto de una conversación, para que pueda controlar su comportamiento y recordar las respuestas a las preguntas anteriores. Las características de almacenamiento y estado del SDK permiten agregar un estado al bot.

## <a name="prerequisites"></a>Requisitos previos

- Es necesario conocer los [conceptos básicos de los bots](bot-builder-basics.md) y cómo los bots [administran el estado](bot-builder-concept-state.md).
- El código de este artículo se basa en el ejemplo **StateBot**. Necesitará una copia del ejemplo en [C# ](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/StateBot) o en [JS]().
- [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started), para probar el bot localmente.

## <a name="about-the-sample-code"></a>Acerca del código de ejemplo

En este artículo se describen los aspectos de la configuración de la administración del estado de un bot. Para agregar estado, configuramos las propiedades de estado, la administración del estado y el almacenamiento y, a continuación, las usamos en el bot.

- Cada _propiedad de estado_ contiene información sobre el estado del bot.
- Cada descriptor de acceso a una propiedad de estado permite obtener o establecer el valor de la propiedad de estado correspondiente.
- Cada objeto de administración de estado automatiza la lectura y escritura de la información de estado asociada al almacenamiento.
- La capa de almacenamiento se conecta con el almacenamiento de respaldo del estado, por ejemplo, en memoria (para pruebas), o almacenamiento de Azure Cosmos DB (para producción).

Es necesario configurar el bot con descriptores de acceso a las propiedades de estado, con los que puede obtener y establecer el estado en tiempo de ejecución, cuando está llevando a cabo una actividad. Para crear un descriptor de acceso a las propiedades de estado, se usa un objeto de administración de estado y, para crear un objeto de administración de estado, se usa una capa de almacenamiento. Por lo tanto, vamos a comenzar en nivel de almacenamiento y a continuar desde ahí.

## <a name="configure-storage"></a>Configurar el almacenamiento

Como no tenemos previsto implementar este bot, vamos a usar _almacenamiento de memoria_ para configurar tanto el estado de usuario como el de conversación.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

En **Startup.cs**, configure la capa de almacenamiento.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    IStorage storage = new MemoryStorage();
    // ...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el archivo **index.js**, configure la capa de almacenamiento.

```javascript
// Define state store for your bot.
const memoryStorage = new MemoryStorage();
```

---

## <a name="create-state-management-objects"></a>Creación de objetos de administración de estado

Realizamos un seguimiento del estado tanto del _usuario_ como de la _conversación_, y los usamos para crear _descriptores de acceso a las propiedades de estado_ en el paso siguiente.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

En **Startup.cs**, haga referencia a la capa de almacenamiento al crear los objetos de administración de estado.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    ConversationState conversationState = new ConversationState(storage);
    UserState userState = new UserState(storage);
    // ...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el archivo **index.js**, agregue `UserState` a la instrucción correspondiente.

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

Después, haga referencia a la capa de almacenamiento al crear los objetos de administración de estado de usuarios y conversaciones.

```javascript
// Create conversation and user state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);
```

---

## <a name="create-state-property-accessors"></a>Creación de descriptores de acceso a las propiedades de estado

Para _declarar_ una propiedad de estado, cree primero un descriptor de acceso a la propiedad de estado usando uno de nuestros objetos de administración de estado. Configuramos el bot para que realice el seguimiento de esta información:

- El nombre del usuario, que se definirá en el estado del usuario.
- Si hemos pedido el nombre al usuario e información adicional sobre el mensaje que acaba de enviar.

El bot usa el descriptor de acceso para obtener la propiedad de estado del contexto del turno.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

Primero se definen las clases para que contenga toda la información que queremos a administrar en cada tipo de estado.

- Una clase `UserProfile` para la información de usuario que recopilará el bot.
- Una clase `ConversationData` para hacer un seguimiento de la información acerca de cuándo ha llegado un mensaje y quién lo envió.

```csharp
// Defines a state property used to track information about the user.
public class UserProfile
{
    public string Name { get; set; }
}
```

```csharp
// Defines a state property used to track conversation data.
public class ConversationData
{
    // The time-stamp of the most recent incoming message.
    public string Timestamp { get; set; }

    // The ID of the user's channel.
    public string ChannelId { get; set; }

    // Track whether we have already asked the user's name
    public bool PromptedUserForName { get; set; } = false;
}
```

A continuación, definimos una clase que contiene la información de administración de estado que necesitamos para configurar nuestra instancia del bot.

```csharp
public class StateBotAccessors
{
    public StateBotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }
  
    public static string UserProfileName { get; } = "UserProfile";

    public static string ConversationDataName { get; } = "ConversationData";

    public IStatePropertyAccessor<UserProfile> UserProfileAccessor { get; set; }

    public IStatePropertyAccessor<ConversationData> ConversationDataAccessor { get; set; }
  
    public ConversationState ConversationState { get; }
  
    public UserState UserState { get; }
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Pasamos los objetos de administración de estado directamente al constructor del bot y permitimos al bot que cree él mismo los descriptores de acceso a las propiedades de estado.

En **index.js**, proporcionamos los objetos de administración de estado al crear el bot.

```javascript
// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

En **bot.js**, definimos los identificadores que necesita para la administración y el seguimiento del estado.

```javascript
// The accessor names for the conversation data and user profile state property accessors.
const CONVERSATION_DATA_PROPERTY = 'conversationData';
const USER_PROFILE_PROPERTY = 'userProfile';
```

---

## <a name="configure-your-bot"></a>Configuración del bot

Ya estamos listos para definir los descriptores de acceso a las propiedades de estado y para configurar nuestro bot.
Vamos a usar el objeto de administración de estado de conversación para el descriptor de acceso a las propiedades de estado del flujo de conversación.
Vamos a usar el objeto de administración de estado de usuario para el descriptor de acceso a las propiedades de estado del perfil de usuario.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

En **Startup.cs**, configuramos ASP.NET para proporcionar los objetos de propiedad y administración de estado integrados. Esto se recuperan desde el constructor del bot con el marco de inserción de dependencias de ASP.NET Core.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    services.AddSingleton<StateBotAccessors>(sp =>
    {
        // Create the custom state accessor.
        return new StateBotAccessors(conversationState, userState)
        {
            ConversationDataAccessor = conversationState.CreateProperty<ConversationData>(StateBotAccessors.ConversationDataName),
            UserProfileAccessor = userState.CreateProperty<UserProfile>(StateBotAccessors.UserProfileName),
        };
    });
}
```

En el constructor del bot, se proporciona el objeto `CustomPromptBotAccessors` cuando ASP.NET crea el bot.

```csharp
// Defines a bot for filling a user profile.
public class CustomPromptBot : IBot
{
    private readonly StateBotAccessors _accessors;

    public StateBot(StateBotAccessors accessors, ILoggerFactory loggerFactory)
    {
        // ...
        accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));
    }

    // The bot's turn handler and other supporting code...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el constructor del bot (en el archivo **bot.js**), creamos los descriptores de acceso a las propiedades de estado y los agregamos al bot. También agregamos referencias a los objetos de administración de estado, porque los necesitamos para guardar los cambios de estado que realicemos.

```javascript
constructor(conversationState, userState) {
    // Create the state property accessors for the conversation data and user profile.
    this.conversationData = conversationState.createProperty(CONVERSATION_DATA_PROPERTY);
    this.userProfile = userState.createProperty(USER_PROFILE_PROPERTY);

    // The state management objects for the conversation and user state.
    this.conversationState = conversationState;
    this.userState = userState;
}
```

---

## <a name="access-state-from-your-bot"></a>Estado de acceso desde el bot

En las secciones anteriores se describen los pasos en tiempo de inicialización para agregar a nuestro bot los descriptores de acceso a las propiedades de estado.
Ahora, podemos usar los descriptores de acceso a las propiedades de estado para leer y escribir la información sobre el estado en tiempo de ejecución.

1. Antes de usar nuestras propiedades de estado, usamos cada descriptor de acceso para cargar la propiedad desde el almacenamiento y obtenerla de la caché de estado.
   - Cuando obtenga una propiedad de estado mediante su descriptor de acceso, debe proporcionar un valor predeterminado. De lo contrario, puede obtener un error de valor NULL.
1. Antes de salir del controlador de turnos:
   1. Usamos el método para _establecer_ los descriptores de acceso para insertar los cambios en el estado del bot.
   1. Usamos el método para _guardar cambios_ de los objetos administración de estado para escribir esos cambios en el almacenamiento.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// The bot's turn handler.
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the state properties from the turn context.
        UserProfile userProfile =
            await _accessors.UserProfileAccessor.GetAsync(turnContext, () => new UserProfile());
        ConversationData conversationData =
            await _accessors.ConversationDataAccessor.GetAsync(turnContext, () => new ConversationData());

        if (string.IsNullOrEmpty(userProfile.Name))
        {
            // First time around this is set to false, so we will prompt user for name.
            if (conversationData.PromptedUserForName)
            {
                // Set the name to what the user provided
                userProfile.Name = turnContext.Activity.Text?.Trim();

                // Acknowledge that we got their name.
                await turnContext.SendActivityAsync($"Thanks {userProfile.Name}.");

                // Reset the flag to allow the bot to go though the cycle again.
                conversationData.PromptedUserForName = false;
            }
            else
            {
                // Prompt the user for their name.
                await turnContext.SendActivityAsync($"What is your name?");

                // Set the flag to true, so we don't prompt in the next turn.
                conversationData.PromptedUserForName = true;
            }

            // Save user state and save changes.
            await _accessors.UserProfileAccessor.SetAsync(turnContext, userProfile);
            await _accessors.UserState.SaveChangesAsync(turnContext);
        }
        else
        {
            // Add message details to the conversation data.
            conversationData.Timestamp = turnContext.Activity.Timestamp.ToString();
            conversationData.ChannelId = turnContext.Activity.ChannelId.ToString();

            // Display state data
            await turnContext.SendActivityAsync($"{userProfile.Name} sent: {turnContext.Activity.Text}");
            await turnContext.SendActivityAsync($"Message received at: {conversationData.Timestamp}");
            await turnContext.SendActivityAsync($"Message received from: {conversationData.ChannelId}");
        }

        // Update conversation state and save changes.
        await _accessors.ConversationDataAccessor.SetAsync(turnContext, conversationData);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// The bot's turn handler.
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Get the state properties from the turn context.
        const userProfile = await this.userProfile.get(turnContext, {});
        const conversationData = await this.conversationData.get(
            turnContext, { promptedForUserName: false });

        if (!userProfile.name) {
            // First time around this is undefined, so we will prompt user for name.
            if (conversationData.promptedForUserName) {
                // Set the name to what the user provided.
                userProfile.name = turnContext.activity.text;

                // Acknowledge that we got their name.
                await turnContext.sendActivity(`Thanks ${userProfile.name}.`);

                // Reset the flag to allow the bot to go though the cycle again.
                conversationData.promptedForUserName = false;
            } else {
                // Prompt the user for their name.
                await turnContext.sendActivity('What is your name?');

                // Set the flag to true, so we don't prompt in the next turn.
                conversationData.promptedForUserName = true;
            }
            // Save user state and save changes.
            await this.userProfile.set(turnContext, userProfile);
            await this.userState.saveChanges(turnContext);
        } else {
            // Add message details to the conversation data.
            conversationData.timestamp = turnContext.activity.timestamp.toLocaleString();
            conversationData.channelId = turnContext.activity.channelId;

            // Display state data.
            await turnContext.sendActivity(`${userProfile.name} sent: ${turnContext.activity.text}`);
            await turnContext.sendActivity(`Message received at: ${conversationData.timestamp}`);
            await turnContext.sendActivity(`Message received from: ${conversationData.channelId}`);
        }
        // Update conversation state and save changes.
        await this.conversationData.set(turnContext, conversationData);
        await this.conversationState.saveChanges(turnContext);
    }
}
```

---

## <a name="test-the-bot"></a>Probar el bot

1. Ejecute el ejemplo localmente en la máquina. Si necesita instrucciones, consulte el archivo Léame en el ejemplo de [C#](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/StateBot) o [JS](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/js/stateBot).
1. Use el emulador para probar el bot tal y como se muestra a continuación.

![prueba del bot de estado de ejemplo](media/state-bot-testing-emulator.png)

## <a name="additional-resources"></a>Recursos adicionales

**Privacidad:** Si tiene intención de almacenar datos personales del usuario, debe garantizar el cumplimiento del [Reglamento general de protección de datos](https://blog.botframework.com/2018/04/23/general-data-protection-regulation-gdpr).

**Administración de estados:** Todas las llamadas de administración de estado son asincrónicas y prevalece el último en escribir de forma predeterminada. En la práctica, debe obtener, establecer y guardar el estado lo más próximos en el bot como sea posible.

**Datos empresariales críticos:** Utilice el estado del bot para almacenar las preferencias, el nombre de usuario o lo último que haya solicitado, pero no lo utilice para almacenar datos empresariales críticos. Para los datos críticos, [cree sus propios componentes de almacenamiento](bot-builder-custom-storage.md) o escríbalos directamente en el [almacenamiento](bot-builder-howto-v4-storage.md).

**Recognizer-Text:** El ejemplo usa las bibliotecas Microsoft/Recognizers-Text para analizar y validar la entrada del usuario. Para más información, consulte la página [Información general](https://github.com/Microsoft/Recognizers-Text#microsoft-recognizers-text-overview).

## <a name="next-step"></a>Paso siguiente

Ahora que sabe cómo configurar el estado para ayudarle a leer y escribir los datos del bot en el almacenamiento, vamos a aprender cómo realizar al usuario una serie de preguntas, validar sus respuestas y guardar su entrada.

> [!div class="nextstepaction"]
> [Creación de mensajes propios para recopilar datos de entrada del usuario](bot-builder-primitive-prompts.md)

---
title: Conservación de datos de usuario | Microsoft Docs
description: Obtenga información sobre cómo guardar los datos de estado de los usuario en el almacenamiento en Bot Framework SDK.
keywords: conversación de datos de usuario, almacenamiento, datos de conversación
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/19/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 97a89744282ca1e31a99aa0e45345656e43f5cd0
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225320"
---
# <a name="persist-user-data"></a>Conservación de datos de usuario

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Cuando el bot pide una entrada a los usuarios, lo más probable es que quiera conservar parte de la información en el almacenamiento de alguna forma. Bot Framework SDK permite almacenar las entradas de los usuarios mediante *almacenamiento en memoria* o almacenamiento en una base de datos como *CosmosDB*. Los tipos de almacenamiento local se usan principalmente durante la realización de pruebas o la creación de prototipos del bot. Sin embargo, los tipos de almacenamiento persistente, como el almacenamiento de base de datos, son los mejores para los bots de producción.

En este tema se muestra cómo definir un objeto de almacenamiento y guardar las entradas de usuario en dicho objeto para que se pueda conservar. Usaremos un diálogo para pedir al usuario su nombre, en caso de que no lo tengamos aún. Independientemente del tipo de almacenamiento que decida usar, el proceso de enlazar y conservar los datos es el mismo. El código de este tema usa `CosmosDB` como espacio de almacenamiento para conservar los datos.

## <a name="prerequisites"></a>Requisitos previos

Se requieren ciertos recursos, en función del entorno de desarrollo que se desee usar.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

* [Instalación de Visual Studio 2015 o superior](https://www.visualstudio.com/downloads/).
* [Instalación de la plantilla de Bot Builder V4](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4).

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

* [Instalación de Visual Studio Code](https://www.visualstudio.com/downloads/).
* [Instalación de Node.js v8.5 o superior](https://nodejs.org/en/).
* [Instalación de Yeoman](http://yeoman.io/).
* Instale el generador de plantillas para Bot Builder v4 de Node.js.

    ```shell
    npm install generator-botbuilder
    ```

---

En este tutorial se usan los siguientes paquetes.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Comenzaremos a partir de una plantilla EchoBot básica. Para obtener instrucciones, consulte la [guía de inicio rápido para .NET](~/dotnet/bot-builder-dotnet-quickstart.md).

Instale estos paquetes adicionales desde el Administrador de paquetes NuGet.

* **Microsoft.Bot.Builder.Azure**
* **Microsoft.Bot.Builder.Dialogs**

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Comenzaremos a partir de una plantilla EchoBot básica. Para obtener instrucciones, consulte la [guía de inicio rápido para JavaScript](~/javascript/bot-builder-javascript-quickstart.md).

Instale estos paquetes de NPM adicionales.

```cmd
npm install --save botbuilder-dialogs
npm install --save botbuilder-azure
```

---

Para probar el bot que cree en este tutorial, tendrá que instalar [BotFramework Emulator](https://github.com/Microsoft/BotFramework-Emulator).

## <a name="create-a-cosmosdb-service-and-update-your-application-settings"></a>Creación de un servicio CosmosDB y actualización de la configuración de la aplicación

Para configurar un servicio CosmosDB y una base de datos, siga las instrucciones para [usar CosmosDB](bot-builder-howto-v4-storage.md#using-cosmos-db). Aquí se resumen los pasos:

1. En una nueva ventana del explorador, inicie sesión en <a href="http://portal.azure.com/" target="_blank">Azure Portal</a>.
1. Haga clic en **Crear un recurso > Bases de datos > Azure Cosmos DB**.
1. En la página **Nueva cuenta**, proporcione un nombre único en el campo **ID**. En **API**, seleccione **SQL** y proporcione la información de **Suscripción**, **Ubicación** y **Grupo de recursos**.
1. Haga clic en **Create**(Crear).

A continuación, agregue una colección a ese servicio para usar con este bot.

Registre el identificador de la base de datos y el identificador de la colección que usó para agregar la colección, así como el identificador URI y la clave principal de la configuración de las claves de la colección, ya que serán necesarios para conectar el bot al servicio.

### <a name="update-your-application-settings"></a>Actualización de la configuración de la aplicación

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Actualice el archivo **appsettings.json** para incluir la información de conexión de CosmosDB.

```csharp
{
  // Settings for CosmosDB.
  "CosmosDB": {
    "DatabaseID": "<your-database-identifier>",
    "CollectionID": "<your-collection-identifier>",
    "EndpointUri": "<your-CosmosDB-endpoint>",
    "AuthenticationKey": "<your-primary-key>"
  }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En la carpeta del proyecto, busque el archivo **.env** y agréguele estas entradas con los datos específicos de Cosmos.

**.env**

```text
DB_SERVICE_ENDPOINT=<your-CosmosDB-endpoint>
AUTH_KEY=<authentication key>
DATABASE=<your-primary-key>
COLLECTION=<your-collection-identifier>
```

Luego, en el archivo **index.js** principal del bot, reemplace `storage` para usar `CosmosDbStorage` en lugar de `MemoryStorage`. Durante el tiempo de ejecución, se extraerán las variables de entorno y rellenarán estos campos.

```javascript
const storage = new CosmosDbStorage({
    serviceEndpoint: process.env.DB_SERVICE_ENDPOINT,
    authKey: process.env.AUTH_KEY, 
    databaseId: process.env.DATABASE,
    collectionId: process.env.COLLECTION
});
```

---

## <a name="create-storage-state-manager-and-state-property-accessor-objects"></a>Creación de los objetos del descriptor de acceso de almacenamiento, administrador de estado y de las propiedades del estado

Los bots usan objetos de almacenamiento y administración de estado para administrar y conservar el estado. El administrador proporciona una capa de abstracción que permite acceder a las propiedades de estado mediante descriptores de acceso de las propiedades del estado, independientemente del tipo de almacenamiento subyacente. Use el administrador de estado para escribir datos en el espacio de almacenamiento.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

### <a name="define-a-class-for-your-user-data"></a>Definición de una clase para los datos de usuario

Cambie el nombre del archivo **CounterState.cs** a **UserData.cs**y el de la clase **CounterState** a **UserData**.

Actualice esta clase para que contenga los datos que va a recopilar.

```csharp
/// <summary>
/// Class for storing persistent user data.
/// </summary>
public class UserData
{
    public string Name { get; set; }
}
```

### <a name="define-a-class-for-your-state-and-state-property-accessor-objects"></a>Definición de una clase para el estado y los objetos del descriptor de acceso de las propiedades del estado

Cambie el nombre del archivo **EchoBotAccessors.cs** a **BotAccessors.cs** y cambiar el nombre de la clase **EchoBotAccessors** a **BotAccessors**.

Actualice esta clase para almacenar los objetos del estado y los descriptores de acceso de las propiedades del estado que va a necesitar el bot.

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using System;

public class BotAccessors
{
    public UserState UserState { get; }

    public ConversationState ConversationState { get; }

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    public IStatePropertyAccessor<UserData> UserDataAccessor { get; set; }

    public BotAccessors(UserState userState, ConversationState conversationState)
    {
        this.UserState = userState
            ?? throw new ArgumentNullException(nameof(userState));

        this.ConversationState = conversationState
            ?? throw new ArgumentNullException(nameof(conversationState));
    }
}
```

### <a name="update-the-startup-code-for-your-bot"></a>Actualización del código de inicio para el bot

En el archivo **Startup.cs**, actualice las instrucciones using.

```csharp
using System;
using System.Linq;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Integration;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Bot.Configuration;
using Microsoft.Bot.Connector.Authentication;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Options;
```

En el método `ConfigureServices`, actualice la llamada para agregar el bot a partir del lugar en que se crea el objeto de almacenamiento de respaldo y, después, registre el objeto de descriptores de acceso del bot.

Necesitamos el estado de la conversación del objeto `DialogState` para realizar un seguimiento del estado del diálogo. Vamos a registrar elementos singleton para el descriptor de acceso de las propiedades del estado del diálogo y el conjunto de diálogos que va a usar nuestro bot. El bot creará su propio descriptor de acceso de las propiedades del estado para el estado de usuario.

El descriptor de acceso `BotAccessors` es una manera eficaz de administrar el almacenamiento de varios objetos de estado de un bot.

```cs
public void ConfigureServices(IServiceCollection services)
{
    // Register your bot.
    services.AddBot<UserDataBot>(options =>
    {
        // ...

        // Use persistent storage and create state management objects.
        var cosmosSettings = Configuration.GetSection("CosmosDB");
        IStorage storage = new CosmosDbStorage(
            new CosmosDbStorageOptions
            {
                DatabaseId = cosmosSettings["DatabaseID"],
                CollectionId = cosmosSettings["CollectionID"],
                CosmosDBEndpoint = new Uri(cosmosSettings["EndpointUri"]),
                AuthKey = cosmosSettings["AuthenticationKey"],
            });
        options.State.Add(new ConversationState(storage));
        options.State.Add(new UserState(storage));
    });

    // Register the bot's state and state property accessor objects.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var userState = options.State.OfType<UserState>().FirstOrDefault();
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();

        return new BotAccessors(userState, conversationState)
        {
            UserDataAccessor = userState.CreateProperty<UserData>("UserDataBot.UserData"),
            DialogStateAccessor = conversationState.CreateProperty<DialogState>("UserDataBot.DialogState"),
        };
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

### <a name="update-your-server-code"></a>Actualización del código de servidor

En el archivo **index.js** del proyecto, actualice las siguientes instrucciones require.

```javascript
// Import required bot services.
const { BotFrameworkAdapter, ConversationState, UserState } = require('botbuilder');
const { CosmosDbStorage } = require('botbuilder-azure');
```

Vamos a usar `UserState` para almacenar los datos de este tutorial. Necesitamos crear un objeto `userState` y actualizar esta línea de código para pasar un segundo parámetro a la clase `MainDialog`.

```javascript
// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(storage);
const userState = new UserState(storage);

// Create the main dialog.
const bot = new MyBot(conversationState, userState);
```

Si se produce un error general, borre la conversación y el estado del usuario.

```javascript
// Catch-all for errors.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${error}`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
    // Clear out state
    await conversationState.load(context);
    await conversationState.clear(context);
    await userState.load(context);
    await userState.clear(context);
    // Save state changes.
    await conversationState.saveChanges(context);
    await userState.saveChanges(context);
};
```

Además, actualice el bucle de servidor HTTP para llamar a nuestro objeto bot.

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to main dialog.
        await bot.onTurn(context);
    });
});
```

### <a name="update-your-bot-logic"></a>Actualización de la lógica del bot

En la clase `MyBot`, requiera las bibliotecas necesarias para que el bot funcione. En este tutorial, vamos a usar la biblioteca **Dialogs**.

```javascript
// Required packages for this bot
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, TextPrompt, NumberPrompt } = require('botbuilder-dialogs');

```

Actualice el constructor de la clase `MyBot` para aceptar un segundo, parámetro `userState`. Además, actualice el constructor para definir los estados, diálogos y mensajes necesarios para este tutorial. En este caso, vamos a definir una cascada de dos pasos, donde el _paso 1_ pide al usuario su nombre y el _paso 2_ devuelve la respuesta del usuario. La lógica principal del bot es la que dicta si se conserva esa información.

```javascript
constructor(conversationState, userState) {

    // creates a new state accessor property.
    this.conversationState = conversationState;
    this.userState = userState;

    this.dialogState = this.conversationState.createProperty('dialogState');
    this.userDataAccessor = this.userState.createProperty('userData');

    this.dialogs = new DialogSet(this.dialogState);

    // Add prompts
    this.dialogs.add(new TextPrompt('textPrompt'));

    // Add a waterfall dialog to collect and return the user's name.
    this.dialogs.add(new WaterfallDialog('greetings', [
        async function (step) {
            return await step.prompt('textPrompt', "What is your name?");
        },
        async function (step) {
            return await step.endDialog(step.result);
        }
    ]));
}
```

---

Cuando llega el momento de guardar los datos del usuario, hay varias opciones entre las que se puede elegir. El SDK proporciona varios objetos de estado con ámbitos diferentes entre los que puede elegir. Aquí vamos a usar el estado de la conversación para administrar el objeto de estado del diálogo y el estado del usuario para administrar los datos del usuario.

Para más información acerca del almacenamiento y del estado en general, consulte [almacenamiento](bot-builder-howto-v4-storage.md) y [administración del estado del usuario y de la conversación](bot-builder-howto-v4-state.md).

## <a name="create-a-greeting-dialog"></a>Creación de un diálogo de saludo

Vamos a usar un diálogo para recopilar el nombre del usuario. Para simplificar este escenario, el diálogo devolverá el nombre del usuario y el bot administrará el objeto de datos de usuario y el estado.

Cree una clase **GreetingsDialog** e incluya el siguiente código.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

/// <summary>Defines a dialog for collecting a user's name.</summary>
public class GreetingsDialog : DialogSet
{
    /// <summary>The ID of the main dialog.</summary>
    public const string MainDialog = "main";

    /// <summary>The ID of the text prompt to use in the dialog.</summary>
    private const string TextPrompt = "textPrompt";

    /// <summary>Creates a new instance of this dialog set.</summary>
    /// <param name="dialogState">The dialog state property accessor to use for dialog state.</param>
    public GreetingsDialog(IStatePropertyAccessor<DialogState> dialogState)
        : base(dialogState)
    {
        // Add the text prompt to the dialog set.
        Add(new TextPrompt(TextPrompt));

        // Define the main dialog and add it to the set.
        Add(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            async (stepContext, cancellationToken) =>
            {
                // Ask the user for their name.
                return await stepContext.PromptAsync(TextPrompt, new PromptOptions
                {
                    Prompt = MessageFactory.Text("What is your name?"),
                }, cancellationToken);
            },
            async (stepContext, cancellationToken) =>
            {
                // Assume that they entered their name, and return the value.
                return await stepContext.EndDialogAsync(stepContext.Result, cancellationToken);
            },
        }));
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Vea la sección anterior en la que se crea el diálogo se crea dentro del constructor `MainDialog`.

---

Para más información acerca de los diálogos, consulte [solicitud de entrada](bot-builder-prompts.md) y [administración de un flujo de conversación simple](bot-builder-dialog-manage-conversation-flow.md).

## <a name="update-your-bot-to-use-the-dialog-and-user-state"></a>Actualización del bot para que utilice el estado del diálogo y del usuario

La construcción de bot y la administración de los datos proporcionados por el usuario se trataran por separado.

### <a name="add-the-dialog-and-a-user-state-accessor"></a>Adición del descriptor de acceso del estado del diálogo y del usuario

Es necesario realizar un seguimiento de la instancia del diálogo y del descriptor de acceso de las propiedades del estado en los datos del usuario.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Agregue el código para inicializar nuestro bot.

```cs
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;

/// <summary>Defines the bot for the persisting user data tutorial.</summary>
public class UserDataBot : IBot
{
    /// <summary>The bot's state and state property accessor objects.</summary>
    private BotAccessors Accessors { get; }

    /// <summary>The dialog set that has the dialog to use.</summary>
    private GreetingsDialog GreetingsDialog { get; }

    /// <summary>Create a new instance of the bot.</summary>
    /// <param name="options">The options to use for our app.</param>
    /// <param name="greetingsDialog">An instance of the dialog set.</param>
    public UserDataBot(BotAccessors botAccessors)
    {
        // Retrieve the bot's state and accessors.
        Accessors = botAccessors;

        // Create the greetings dialog.
        GreetingsDialog = new GreetingsDialog(Accessors.DialogStateAccessor);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Vea la sección anterior en la que se definen estos descriptores de acceso dentro del constructor de `MainDialog`.

---

### <a name="update-the-turn-handler"></a>Actualización del controlador de turno

El controlador de turno saludará a los usuarios cuando se incorporen a la conversación y les responderá cuando envíen un mensaje al bot. Si en algún momento el botón desconoce el nombre del usuario, iniciará el diálogo de salido para preguntarle su nombre. Cuando el diálogo finaliza, se guarda su nombre en el estado del usuario mediante un objeto de estado generado por nuestro descriptor de acceso de propiedades de los estados. Cuando el turno finaliza, el descriptor de acceso y el administrador de estado escribirán los cambios en el objeto fuera del almacenamiento.

También vamos a agregar compatibilidad con la actividad _eliminar datos de usuario_.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Actualice el método `OnTurnAsync` del bot.

```cs
/// <summary>Handles incoming activities to the bot.</summary>
/// <param name="turnContext">The context object for the current turn.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    // Retrieve user data from state.
    UserData userData = await Accessors.UserDataAccessor.GetAsync(turnContext, () => new UserData());

    // Establish context for our dialog from the turn context.
    DialogContext dc = await GreetingsDialog.CreateContextAsync(turnContext);

    // Handle conversation update, message, and delete user data activities.
    switch (turnContext.Activity.Type)
    {
        case ActivityTypes.ConversationUpdate:

            // Greet any user that is added to the conversation.
            IConversationUpdateActivity activity = turnContext.Activity.AsConversationUpdateActivity();
            if (activity.MembersAdded.Any(member => member.Id != activity.Recipient.Id))
            {
                if (userData.Name is null)
                {
                    // If we don't already have their name, start a dialog to collect it.
                    await turnContext.SendActivityAsync("Welcome to the User Data bot.");
                    await dc.BeginDialogAsync(GreetingsDialog.MainDialog);
                }
                else
                {
                    // Otherwise, greet them by name.
                    await turnContext.SendActivityAsync($"Hi {userData.Name}! Welcome back to the User Data bot.");
                }
            }

            break;

        case ActivityTypes.Message:

            // If there's a dialog running, continue it.
            if (dc.ActiveDialog != null)
            {
                var dialogTurnResult = await dc.ContinueDialogAsync();
                if (dialogTurnResult.Status == DialogTurnStatus.Complete
                    && dialogTurnResult.Result is string name
                    && !string.IsNullOrWhiteSpace(name))
                {
                    // If it completes successfully and returns a valid name, save the name and greet the user.
                    userData.Name = name;
                    await turnContext.SendActivityAsync($"Pleased to meet you {userData.Name}.");
                }
            }
            else if (userData.Name is null)
            {
                // Else, if we don't have the user's name yet, ask for it.
                await dc.BeginDialogAsync(GreetingsDialog.MainDialog);
            }
            else
            {
                // Else, echo the user's message text.
                await turnContext.SendActivityAsync($"{userData.Name} said, '{turnContext.Activity.Text}'.");
            }

            break;

        case ActivityTypes.DeleteUserData:

            // Delete the user's data.
            userData.Name = null;
            await turnContext.SendActivityAsync("I have deleted your user data.");

            break;
    }

    // Update the user data in the turn's state cache.
    await Accessors.UserDataAccessor.SetAsync(turnContext, userData, cancellationToken);

    // Persist any changes to storage.
    await Accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
    await Accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Actualice el controlador `onTurn` del bot.

```javascript
async onTurn(turnContext) {
    const dc = await this.dialogs.createContext(turnContext); // Create dialog context
    const userData = await this.userDataAccessor.get(turnContext, {});

    switch (turnContext.activity.type) {
        case ActivityTypes.ConversationUpdate:
            if (turnContext.activity.membersAdded[0].name !== 'Bot') {
                if (userData.name) {
                    await turnContext.sendActivity(`Hi ${userData.name}! Welcome back to the User Data bot.`);
                }
                else {
                    // send a "this is what the bot does" message
                    await turnContext.sendActivity('Welcome to the User Data bot.');
                    await dc.beginDialog('greetings');
                }
            }
            break;
        case ActivityTypes.Message:
            // If there is an active dialog running, continue it
            if (dc.activeDialog) {
                var turnResult = await dc.continueDialog();
                if (turnResult.status == "complete" && turnResult.result) {
                    // If it completes successfully and returns a value, save the name and greet the user.
                    userData.name = turnResult.result;
                    await this.userDataAccessor.set(turnContext, userData);
                    await turnContext.sendActivity(`Pleased to meet you ${userData.name}.`);
                }
            }
            // Else, if we don't have the user's name yet, ask for it.
            else if (!userData.name) {
                await dc.beginDialog('greetings');
            }
            // Else, echo the user's message text.
            else {
                await turnContext.sendActivity(`${userData.name} said, ${turnContext.activity.text}.`);
            }
            break;
        case ActivityTypes.DeleteUserData:
            // Delete the user's data.
            // Note: You can use the Emulator to send this activity.
            userData.name = null;
            await this.userDataAccessor.set(turnContext, userData);
            await turnContext.sendActivity("I have deleted your user data.");
            break;
    }

    // Save changes to the conversation and user states.
    await this.conversationState.saveChanges(turnContext);
    await this.userState.saveChanges(turnContext);
}
```

---

## <a name="start-your-bot-in-visual-studio"></a>Inicio del bot en Visual Studio

Compile y ejecute la aplicación.

## <a name="start-the-emulator-and-connect-your-bot"></a>Inicio del emulador y conexión del bot

A continuación, inicie el emulador y, después, conéctese al bot en el emulador:

1. Haga clic en el vínculo **Open bot** (Abrir bot) de la pestaña de bienvenida del emulador.
2. Seleccione el archivo .bot ubicado en el directorio donde se creó la solución de Visual Studio.

## <a name="interact-with-your-bot"></a>Interacción con el bot

Envíe un mensaje al bot y este responderá con un mensaje.
![Emulador en ejecución](../media/emulator-v4/emulator-running.png)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Administración del estado de la conversación y del usuario](bot-builder-howto-v4-state.md)

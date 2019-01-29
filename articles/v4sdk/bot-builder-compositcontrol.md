---
title: Reutilización de diálogos | Microsoft Docs
description: Aprenda a modularizar la lógica del bot mediante el contenedor de diálogos de Bot Framework SDK para Node.js y C#.
keywords: control compuesto, lógica de bot modular
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/16/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0a931ad73ed4d7a71978555df0e77d6b2bd2dbbc
ms.sourcegitcommit: c6ce4c42fc56ce1e12b45358d2c747fb77eb74e2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/22/2019
ms.locfileid: "54453929"
---
# <a name="reuse-dialogs"></a>Reutilización de diálogos

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Imagine que va a crear un bot de hotel que controla varias tareas como, por ejemplo, saludar al usuario, reservar una mesa para cenar, encargar comida, configurar una alarma, mostrar el tiempo actual, etc. Puede controlar cada una de estas tareas del bot con un objeto de diálogo, sin embargo, esto puede hacer que el código del diálogo sea demasiado grande y desordenado.

Para activar las partes del flujo de conversación en los diálogos de componente, puede dividir un conjunto de diálogos de gran tamaño en partes más manejables. De este modo, puede simplificar el código y facilitar la depuración y permitir que varios equipos trabajen simultáneamente en el bot.
También puede crear una biblioteca de diálogos de componentes para reutilizarlos en otros conjuntos de diálogos y bots.

En este ejemplo vamos a crear un bot de hotel que combina los diálogos de los componentes de registro, despertador y reserva de mesa.

Este artículo se basa en información acerca de cómo administrar flujos de conversación [simples](bot-builder-dialog-manage-conversation-flow.md) y [complejos](bot-builder-dialog-manage-complex-conversation-flow.md).

## <a name="create-your-project"></a>Creación del proyecto

Empezamos con la plantilla de bot de eco e incluimos la biblioteca de diálogos en el proyecto.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Comenzaremos a partir de la plantilla **EchoBot**. Para obtener instrucciones, consulte la [guía de inicio rápido para .NET](../dotnet/bot-builder-dotnet-sdk-quickstart.md).

Para usar diálogos, instale el paquete NuGet `Microsoft.Bot.Builder.Dialogs` para su proyecto o solución.
A continuación, haga referencia a la biblioteca Dialogs en las instrucciones using de los archivos de código según sea necesario.

Cambie el nombre del archivo **EchoBotWithCounter.cs** por **HotelBot.cs** y cambie el nombre de la clase a **HotelBot**.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Comenzaremos a partir de la plantilla **Echo**. Para instrucciones, consulte la [guía de inicio rápido para JavaScript](../javascript/bot-builder-javascript-quickstart.md).

La biblioteca `botbuilder-dialogs` se puede descargar desde NPM. Para instalar la biblioteca `botbuilder-dialogs`, ejecute el siguiente comando de NPM:

```cmd
npm install --save botbuilder-dialogs
```

---

## <a name="managing-state"></a>Administración de estados

Hay muchas maneras de configurar la administración de estados de un bot que usa diálogos compuestos. En este bot, cada diálogo de componente devolverá un objeto como el resultado del diálogo. El contexto de llamada administra los valores devueltos. Para más información acerca de la administración de estados, consulte el artículo [Guardar el estado mediante las propiedades del usuario y la conversación](bot-builder-howto-v4-state.md).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Cada diálogo recopilará algo de información y el controlador de turno del bot o el menú principal guardarán esta información en el estado del usuario. Definiremos los diálogos de componente para el registro, la reserva de mesa y la configuración de una alarma. Cada uno devolverá un objeto de la clase adecuada. Agregue cada una de las siguientes clases al proyecto como un nuevo módulo de clases de C#.

```csharp
/// <summary>
/// User state information.
/// </summary>
public class UserInfo
{
    public GuestInfo Guest { get; set; }
    public TableInfo Table { get; set; }
    public WakeUpInfo WakeUp { get; set; }
}

/// <summary>
/// State information associated with the check-in dialog.
/// </summary>
public class GuestInfo
{
    public string Name { get; set; }
    public string Room { get; set; }
}

/// <summary>
/// State information associated with the reserve-table dialog.
/// </summary>
public class TableInfo
{
    public string Number { get; set; }
}

/// <summary>
/// State information associated with the wake-up call dialog.
/// </summary>
public class WakeUpInfo
{
    public string Time { get; set; }
}
```

Cambie el nombre del archivo **EchoBotAccessors.cs** a **BotAccessors.cs** y actualice también el nombre de la clase **BotAccessors**.

A continuación, actualice el archivo con este código. Necesitamos los descriptores de acceso tanto para el estado del diálogo como para la información del usuario que recopilamos.

```csharp
using System;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

/// <summary>
/// Contains the state objects and the state property accessors for the bot.
/// </summary>
public class BotAccessors
{
    // The property accessor keys to use.
    public const string UserInfoAccessorName = "HotelBot.UserInfo";
    public const string DialogStateAccessorName = "HotelBot.DialogState";

    /// <summary>
    /// Initializes a new instance of the <see cref="BotAccessors"/> class.
    /// Contains the <see cref="ConversationState"/> and associated <see cref="IStatePropertyAccessor{T}"/>.
    /// </summary>
    /// <param name="conversationState">The state object that stores the counter.</param>
    public BotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    /// <summary>Gets or sets the state property accessor for the user information we're tracking.</summary>
    /// <value>Accessor for user information.</value>
    public IStatePropertyAccessor<UserInfo> UserInfoAccessor { get; set; }

    /// <summary>Gets or sets the state property accessor for the dialog state.</summary>
    /// <value>Accessor for the dialog state.</value>
    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    /// <summary>Gets the conversation state for the bot.</summary>
    /// <value>The conversation state for the bot.</value>
    public ConversationState ConversationState { get; }

    /// <summary>Gets the user state for the bot.</summary>
    /// <value>The user state for the bot.</value>
    public UserState UserState { get; }
}
```

En el archivo **Startup.cs**, actualice el código en el método `ConfigureServices` que configura el estado y los descriptores de acceso de propiedad de estado del bot.

```csharp
using Microsoft.Bot.Builder.Dialogs;

public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<HotelBot>(options =>
    {
        //...

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        // Create conversation and user state objects.
        options.State.Add(new ConversationState(dataStore));
        options.State.Add(new UserState(dataStore));
    });

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        var userState = options.State.OfType<UserState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState, userState)
        {
            UserInfoAccessor = userState.CreateProperty<UserInfo>(BotAccessors.UserInfoAccessorName),
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorName),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Cada diálogo recopilará algo de información y el controlador de turno del bot o el menú principal guardarán esta información en el estado del usuario. Definiremos los diálogos de componente para el registro, la reserva de mesa y la configuración de una alarma. Cada uno devolverá un objeto con las propiedades adecuadas. Se realizará la agregación de estas propiedades en el estado de usuario del bot.

En el archivo **bot.js**, actualice el constructor del bot para crear descriptores de acceso de propiedad de estado y realizar un seguimiento del estado de los usuarios y del estado de los diálogos.

```javascript
// Define the identifiers for our state property accessors.
const DIALOG_STATE_PROPERTY = 'dialogStatePropertyAccessor';
const USER_INFO_PROPERTY = 'userInfoPropertyAccessor';

constructor(conversationState, userState) {
    // Record the conversation and user state management objects.
    this.conversationState = conversationState;
    this.userState = userState;

    // Create our state property accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    this.userInfoAccessor = userState.createProperty(USER_INFO_PROPERTY);
}
```

En su archivo **index.js**, actualice as clases importadas desde la biblioteca `botbuilder` y el código que crea los objetos de estado y el bot:

```javascript
// Import required bot services.
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

```javascript
// Define state store for your bot.
const memoryStorage = new MemoryStorage();

// Create conversation and user state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);

// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

---

## <a name="about-component-dialogs"></a>Acerca de los diálogos de componentes

Un diálogo de componente permite crear diálogos independientes para tratar escenarios determinados, y divide un conjunto de diálogos grande en elementos más manejables. Cada una de estas piezas tiene su propio conjunto de diálogos y evita los conflictos de nombres con el conjunto de diálogos que lo contiene.

Use el método _add dialog_ para agregar cuadros de diálogo y avisos al diálogo de componente.
El primer elemento que agregue con este método se establece como el diálogo inicial; se puede cambiar si se establece explícitamente la propiedad _initial dialog_ en el constructor del diálogo de componente.
Al iniciar un diálogo de componente, se iniciará su _initial dialog_.

## <a name="define-the-check-in-component-dialog"></a>Definición del diálogo del componente de registro

En primer lugar, vamos a empezar con un sencillo diálogo de registro que pedirá al usuario su nombre y el número de habitación en la que se alojará. Se crea una clase `CheckInDialog` que extiende `ComponentDialog`. Esta clase tiene un constructor que define el nombre del diálogo de raíz que, a su vez, se define como un diálogo en cascada con tres pasos.

Estos son los pasos para el diálogo de registro.

1. Pida el nombre del huésped.
1. Pregunte por la habitación en la que le gustaría permanecer.
1. Envíe un mensaje de confirmación y complete el diálogo.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Agregue una clase `CheckInDialog` al proyecto, con el código siguiente.

A lo largo de este diálogo podemos escribir en un objeto de estado local, al que se puede acceder mediante una propiedad `Values` del contexto de paso. Una vez completado el diálogo, se elimina el objeto de estado local. Por lo tanto, se devuelve un valor del diálogo que contiene la información del invitado.

```csharp
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

public class CheckInDialog : ComponentDialog
{
    private const string GuestKey = nameof(CheckInDialog);
    private const string TextPrompt = "textPrompt";

    // You can start this from the parent using the dialog's ID.
    public CheckInDialog(string id)
        : base(id)
    {
        InitialDialogId = Id;

        // Define the prompts used in this conversation flow.
        AddDialog(new TextPrompt(TextPrompt));

        // Define the conversation flow using a waterfall model.
        WaterfallStep[] waterfallSteps = new WaterfallStep[]
        {
            NameStepAsync,
            RoomStepAsync,
            FinalStepAsync,
        };
        AddDialog(new WaterfallDialog(Id, waterfallSteps));
    }

    private static async Task<DialogTurnResult> NameStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Clear the guest information and prompt for the guest's name.
        step.Values[GuestKey] = new GuestInfo();
        return await step.PromptAsync(
            TextPrompt,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("What is your name?"),
            },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> RoomStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Save the name and prompt for the room number.
        string name = step.Result as string;
        ((GuestInfo)step.Values[GuestKey]).Name = name;
        return await step.PromptAsync(
            TextPrompt,
            new PromptOptions
            {
                Prompt = MessageFactory.Text($"Hi {name}. What room will you be staying in?"),
            },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> FinalStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Save the room number and "sign off".
        string room = step.Result as string;
        ((GuestInfo)step.Values[GuestKey]).Room = room;

        await step.Context.SendActivityAsync(
            "Great, enjoy your stay!",
            cancellationToken: cancellationToken);

        // End the dialog, returning the guest info.
        return await step.EndDialogAsync(
            (GuestInfo)step.Values[GuestKey],
            cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Cree un archivo **checkInDialog.js** y agréguelo a una clase `CheckInDialog` que extiende `ComponentDialog`.

A lo largo de este diálogo podemos escribir en un objeto de estado local, al que se puede acceder mediante una propiedad `values` del contexto de paso. Una vez completado el diálogo, se elimina el objeto de estado local. Por lo tanto, se devuelve un valor del diálogo que contiene la información del invitado.

```JavaScript
const { ComponentDialog, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');

class CheckInDialog extends ComponentDialog {
    constructor(dialogId) {
        super(dialogId);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = dialogId;

        // Define the prompts used in this conversation flow.
        this.addDialog(new TextPrompt('textPrompt'));

        // Define the conversation flow using a waterfall model.
        this.addDialog(new WaterfallDialog(dialogId, [
            async function (step) {
                // Clear the guest information and prompt for the guest's name.
                step.values.guestInfo = {};
                return await step.prompt('textPrompt', "What is your name?");
            },
            async function (step) {
                // Save the name and prompt for the room number.
                step.values.guestInfo.userName = step.result;
                return await step.prompt('textPrompt', `Hi ${step.result}. What room will you be staying in?`);
            },
            async function (step) {
                // Save the room number and "sign off".
                step.values.guestInfo.roomNumber = step.result;
                await step.context.sendActivity(`Great! Enjoy your stay in room ${step.result}!`);

                // End the dialog, returning the guest info.
                return await step.endDialog(step.values.guestInfo);
            }
        ]));
    }
}

exports.CheckInDialog = CheckInDialog;
```

---

## <a name="define-the-reserve-table-and-wake-up-component-dialogs"></a>Definición de los diálogos de los componentes de reserva de mesa y despertador

Una ventaja de usar un diálogo de componente es la posibilidad de utilizar diálogos diferenciados conjuntamente. Como cada `DialogSet` mantiene un conjunto exclusivo de `dialogs`, el uso compartido o la referencia cruzada de `dialogs` no se puede realizar fácilmente. Y es aquí donde entra en juego el diálogo de componentes. Puede encapsular aspectos complejos o intrincados de un flujo de conversación en los diálogos de componente, lo que puede facilitar la administración y el mantenimiento de los diálogos. Vamos a crear los otros dos diálogos de componente: uno para preguntar al usuario qué mesa le gustaría reservar para cenar y otro para crear una llamada de despertador. Una vez más, vamos a usar una clase aparte para cada diálogo y cada diálogo ampliará el `ComponentDialog` principal.

Se diseñarán para aceptar la información de los invitados en las opciones de los diálogos cuando se inician.

Estos son los pasos para el diálogo de la reserva de mesa.

1. Pregunte por la mesa que se va a reservar.
1. Envíe un mensaje de confirmación y complete el diálogo, y devuelva el número de mesa.

Estos son los pasos para el diálogo de despertador.

1. Pregunte la hora a la que desea que se le despierte.
1. Envíe un mensaje de confirmación y complete el diálogo, y devuelva la hora de la alarma.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Agregue una clase `ReserveTableDialog` al proyecto, con el código siguiente.

Obtendremos el nombre del invitado desde el objeto de opciones que se pasa cuando se inicia el diálogo. A continuación, esta vez se devolverá un valor desde el diálogo, uno que contenga el número de mesa.

```csharp
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;

public class ReserveTableDialog : ComponentDialog
{
    private const string TablePrompt = "choicePrompt";

    public ReserveTableDialog(string id)
        : base(id)
    {
        InitialDialogId = Id;

        // Define the prompts used in this conversation flow.
        AddDialog(new ChoicePrompt(TablePrompt));

        // Define the conversation flow using a waterfall model.
        WaterfallStep[] waterfallSteps = new WaterfallStep[]
        {
                TableStepAsync,
                FinalStepAsync,
        };
        AddDialog(new WaterfallDialog(Id, waterfallSteps));
    }

    private static async Task<DialogTurnResult> TableStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        string greeting = step.Options is GuestInfo guest
                && !string.IsNullOrWhiteSpace(guest?.Name)
                ? $"Welcome {guest.Name}" : "Welcome";

        string prompt = $"{greeting}, How many diners will be at your table?";
        string[] choices = new string[] { "1", "2", "3", "4", "5", "6" };
        return await step.PromptAsync(
            TablePrompt,
            new PromptOptions
            {
                Prompt = MessageFactory.Text(prompt),
                Choices = ChoiceFactory.ToChoices(choices),
            },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> FinalStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // ChoicePrompt returns a FoundChoice object.
        string table = (step.Result as FoundChoice).Value;

        // Send a confirmation message.
        await step.Context.SendActivityAsync(
            $"Sounds great;  we will reserve a table for you for {table} diners.",
            cancellationToken: cancellationToken);

        // End the dialog, returning the table info.
        return await step.EndDialogAsync(
            new TableInfo { Number = table },
            cancellationToken);
    }
}
```

Agregue una clase `SetAlarmDialog` al proyecto, con el código siguiente.

Se obtiene el número de habitación del invitado desde el objeto de opciones que se pasa cuando se inicia el diálogo. A continuación, esta vez se devolverá un valor desde el diálogo, que contenga la hora para la que desea que se establezca una llamada de despertador.

```csharp
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

public class SetAlarmDialog : ComponentDialog
{
    private const string AlarmPrompt = "dateTimePrompt";

    public SetAlarmDialog(string id)
        : base(id)
    {
        InitialDialogId = Id;

        // Define the prompts used in this conversation flow.
        // Ideally, we'd add validation to this prompt.
        AddDialog(new DateTimePrompt(AlarmPrompt));

        // Define the conversation flow using a waterfall model.
        WaterfallStep[] waterfallSteps = new WaterfallStep[]
        {
                AlarmStepAsync,
                FinalStepAsync,
        };

        AddDialog(new WaterfallDialog(Id, waterfallSteps));
    }

    private static async Task<DialogTurnResult> AlarmStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        string greeting = step.Options is GuestInfo guest
                && !string.IsNullOrWhiteSpace(guest?.Name)
                ? $"Hi {guest.Name}" : "Hi";

        string prompt = $"{greeting}. When would you like your alarm set for?";
        return await step.PromptAsync(
            AlarmPrompt,
            new PromptOptions { Prompt = MessageFactory.Text(prompt) },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> FinalStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Ambiguous responses can generate multiple results.
        var resolution = (step.Result as IList<DateTimeResolution>)?.FirstOrDefault();

        // Time ranges have a start and no value.
        var alarm = resolution.Value ?? resolution.Start;
        string roomNumber = (step.Options as GuestInfo)?.Room;

        // Send a confirmation message.
        await step.Context.SendActivityAsync(
            $"Your alarm is set to {alarm} for room {roomNumber}.",
            cancellationToken: cancellationToken);

        // End the dialog, returning the alarm info.
        return await step.EndDialogAsync(
            new WakeUpInfo { Time = alarm },
            cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Cree un archivo **reserveTableDialog.js** y agréguelo a una clase `ReserveTableDialog` que extiende `ComponentDialog`.

Obtendremos el nombre del invitado desde el objeto de opciones que se pasa cuando se inicia el diálogo. A continuación, esta vez se devolverá un valor desde el diálogo, uno que contenga el número de mesa.

```JavaScript
const { ComponentDialog, ChoicePrompt, WaterfallDialog } = require('botbuilder-dialogs');

class ReserveTableDialog extends ComponentDialog {
    constructor(dialogId) {
        super(dialogId);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = dialogId;

        // Define the prompts used in this conversation flow.
        this.addDialog(new ChoicePrompt('choicePrompt'));

        // Define the conversation flow using a waterfall model.
        this.addDialog(new WaterfallDialog(dialogId, [
            async function (step) {
                // Welcome the user and ask for their table preference.
                const greeting = step.options && step.options.userName ? `Welcome ${step.options.userName}` : `Welcome`;

                const promptOptions = {
                    prompt: `${greeting}, How many diners will be at your table?`,
                    reprompt: 'That was not a valid choice, please select a table size between 1 and 6 guests.',
                    choices: ['1', '2', '3', '4', '5', '6']
                };
                return await step.prompt('choicePrompt', promptOptions);
            },
            async function (step) {
                const choice = step.result;

                // Send a confirmation message.
                const tableNumber = choice.value;
                await step.context.sendActivity(`Sounds great, we will reserve a table for you for ${tableNumber} diners.`);

                // End the dialog, returning the table info.
                return await step.endDialog({ table: tableNumber });
            }
        ]));
    }
}

exports.ReserveTableDialog = ReserveTableDialog;
```

Cree un archivo **setAlarmDialog.js** y agréguelo a una clase `SetAlarmDialog` que extiende `ComponentDialog`.

Se obtiene el número de habitación del invitado desde el objeto de opciones que se pasa cuando se inicia el diálogo. A continuación, esta vez se devolverá un valor desde el diálogo, que contenga la hora para la que desea que se establezca una llamada de despertador.

```JavaScript
const { ComponentDialog, DateTimePrompt, WaterfallDialog } = require('botbuilder-dialogs');

class SetAlarmDialog extends ComponentDialog {
    constructor(dialogId) {
        super(dialogId);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = dialogId;

        // Define the prompts used in this conversation flow.
        this.addDialog(new DateTimePrompt('datePrompt'));

        this.addDialog(new WaterfallDialog(dialogId, [
            async function (step) {
                step.values.wakeUp = {};
                if (step.options && step.options.roomNumber) {
                    step.values.roomNumber = step.options.roomNumber;
                }

                const greeting = step.options && step.options.userName ? `Hi ${step.options.userName}` : `Hi`;
                return await step.prompt('datePrompt', `${greeting}. What time would you like your alarm to be set?`);
            },
            async function (step) {
                // Ambiguous responses can generate multiple results.
                const resolution = step.result[0];

                // Time ranges have a start and no value.
                const alarm = resolution.value ? resolution.value : resolution.start;
                const roomNumber = step.values.roomNumber;

                // Send a confirmation message.
                await step.context.sendActivity(`Your alarm is set to ${alarm} for room ${roomNumber}.`);

                // End the dialog, returning the alarm info.
                return await step.endDialog({ alarm: alarm });
            }]));

        // Defining the prompt used in this conversation flow
    }
}

exports.SetAlarmDialog = SetAlarmDialog;
```

---

## <a name="add-the-component-dialogs-to-the-bot"></a>Agregue los diálogos de componente al bot

Ahora que hemos definido nuestros tres diálogos de componente, podemos usarlos en nuestro bot.

- Los tres diálogos se agregan al conjunto de diálogos principal de nuestro bot.
- Cuando se inicia una nueva conversación, no hay ningún diálogo activo y la lógica del turno de encendido del bot asume el control.
- Si no tenemos la información de los invitados para el usuario, se inicia el diálogo de registro.
- Una vez que tenemos la información de los invitados, el diálogo principal interviene varias veces, lo que permite al usuario iniciar el diálogo de reserva de mesa o de despertador.

Actualizaremos la lógica para el controlador de turnos del bot.

1. Obtenga el estado del usuario.
1. Continúe con cualquier diálogo activo.
1. Si un diálogo completó este turno, debe ser el diálogo de registro.
   1. Almacene la información de los invitados.
   1. Inicie el diálogo principal.
1. Si el bot aún no envió un mensaje al usuario, no hay ningún diálogo activo ejecutándose.
    1. Si no tenemos la información del invitado, se inicia el diálogo de registro.
    1. De lo contrario, inicie el diálogo principal.
1. Guarde los cambios de estado que pudieran haberse producido.

Estos son los pasos para el diálogo principal.

1. Preguntar al huésped qué desea hacer: reservar una mesa o configurar una llamada de despertador.
1. Iniciar el diálogo secundario correspondiente o enviar un mensaje de _entrada no comprendida_ y empezar de nuevo desde el primer paso.
1. Procesar el valor devuelto desde el diálogo secundario y reiniciar el diálogo principal.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En el archivo **HotelBot.cs**, actualice las instrucciones using.

```csharp
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;
using Microsoft.Extensions.Logging;
```

Actualice el código de inicialización y defina el conjunto de diálogos y el diálogo principal del bot.

```csharp
// Define the IDs for the dialogs in the bot's dialog set.
private const string MainDialogId = "mainDialog";
private const string CheckInDialogId = "checkInDialog";
private const string TableDialogId = "tableDialog";
private const string AlarmDialogId = "alarmDialog";

// Define the dialog set for the bot.
private readonly DialogSet _dialogs;

// Define the state accessors and the logger for the bot.
private readonly BotAccessors _accessors;
private readonly ILogger _logger;

/// <summary>
/// Initializes a new instance of the <see cref="HotelBot"/> class.
/// </summary>
/// <param name="accessors">Contains the objects to use to manage state.</param>
/// <param name="loggerFactory">A <see cref="ILoggerFactory"/> that is hooked to the Azure App Service provider.</param>
public HotelBot(BotAccessors accessors, ILoggerFactory loggerFactory)
{
    if (loggerFactory == null)
    {
        throw new System.ArgumentNullException(nameof(loggerFactory));
    }

    _logger = loggerFactory.CreateLogger<HotelBot>();
    _logger.LogTrace($"{nameof(HotelBot)} turn start.");
    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Define the steps of the main dialog.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        MenuStepAsync,
        HandleChoiceAsync,
        LoopBackAsync,
    };

    // Create our bot's dialog set, adding a main dialog and the three component dialogs.
    _dialogs = new DialogSet(_accessors.DialogStateAccessor)
        .Add(new WaterfallDialog(MainDialogId, steps))
        .Add(new CheckInDialog(CheckInDialogId))
        .Add(new ReserveTableDialog(TableDialogId))
        .Add(new SetAlarmDialog(AlarmDialogId));
}

private static async Task<DialogTurnResult> MenuStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Present the user with a set of "suggested actions".
    List<string> menu = new List<string> { "Reserve Table", "Wake Up" };
    await stepContext.Context.SendActivityAsync(
        MessageFactory.SuggestedActions(menu, "How can I help you?"),
        cancellationToken: cancellationToken);
    return Dialog.EndOfTurn;
}

private async Task<DialogTurnResult> HandleChoiceAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Get the user's info. (Since the type factory is null, this will throw if state does not yet have a value for user info.)
    UserInfo userInfo = await _accessors.UserInfoAccessor.GetAsync(stepContext.Context, null, cancellationToken);

    // Check the user's input and decide which dialog to start.
    // Pass in the guest info when starting either of the child dialogs.
    string choice = (stepContext.Result as string)?.Trim()?.ToLowerInvariant();
    switch (choice)
    {
        case "reserve table":
            return await stepContext.BeginDialogAsync(TableDialogId, userInfo.Guest, cancellationToken);

        case "wake up":
            return await stepContext.BeginDialogAsync(AlarmDialogId, userInfo.Guest, cancellationToken);

        default:
            // If we don't recognize the user's intent, start again from the beginning.
            await stepContext.Context.SendActivityAsync(
                "Sorry, I don't understand that command. Please choose an option from the list.");
            return await stepContext.ReplaceDialogAsync(MainDialogId, null, cancellationToken);
    }
}

private async Task<DialogTurnResult> LoopBackAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Get the user's info. (Because the type factory is null, this will throw if state does not yet have a value for user info.)
    UserInfo userInfo = await _accessors.UserInfoAccessor.GetAsync(stepContext.Context, null, cancellationToken);

    // Process the return value from the child dialog.
    switch (stepContext.Result)
    {
        case TableInfo table:
            // Store the results of the reserve-table dialog.
            userInfo.Table = table;
            await _accessors.UserInfoAccessor.SetAsync(stepContext.Context, userInfo, cancellationToken);
            break;
        case WakeUpInfo alarm:
            // Store the results of the set-wake-up-call dialog.
            userInfo.WakeUp = alarm;
            await _accessors.UserInfoAccessor.SetAsync(stepContext.Context, userInfo, cancellationToken);
            break;
        default:
            // We shouldn't get here, since these are no other branches that get this far.
            break;
    }

    // Restart the main menu dialog.
    return await stepContext.ReplaceDialogAsync(MainDialogId, null, cancellationToken);
}
```

Actualice el controlador de turno del bot para usar el conjunto de diálogos.

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Establish dialog state from the conversation state.
        DialogContext dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

        // Get the user's info.
        UserInfo userInfo = await _accessors.UserInfoAccessor.GetAsync(turnContext, () => new UserInfo(), cancellationToken);

        // Continue any current dialog.
        DialogTurnResult dialogTurnResult = await dc.ContinueDialogAsync();

        // Process the result of any complete dialog.
        if (dialogTurnResult.Status is DialogTurnStatus.Complete)
        {
            switch (dialogTurnResult.Result)
            {
                case GuestInfo guestInfo:
                    // Store the results of the check-in dialog.
                    userInfo.Guest = guestInfo;
                    await _accessors.UserInfoAccessor.SetAsync(turnContext, userInfo, cancellationToken);
                    break;
                default:
                    // We shouldn't get here, since the main dialog is designed to loop.
                    break;
            }
        }

        // Every dialog step sends a response, so if no response was sent,
        // then no dialog is currently active.
        else if (!turnContext.Responded)
        {
            if (string.IsNullOrEmpty(userInfo.Guest?.Name))
            {
                // If we don't yet have the guest's info, start the check-in dialog.
                await dc.BeginDialogAsync(CheckInDialogId, null, cancellationToken);
            }
            else
            {
                // Otherwise, start our bot's main dialog.
                await dc.BeginDialogAsync(MainDialogId, null, cancellationToken);
            }
        }

        // Save the new turn count into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
        await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
    }
    else
    {
        await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En nuestro archivo de bot, **bot.js**, es necesario importar no solo las clases que vamos a usar desde el SDK, sino también las clases que se han definido para nuestros diálogos de componente.

```JavaScript
const { ActivityTypes, MessageFactory } = require('botbuilder');
const { DialogSet, WaterfallDialog, Dialog, DialogTurnStatus } = require('botbuilder-dialogs');

// Import our component dialogs.
const { CheckInDialog } = require("./checkInDialog");
const { ReserveTableDialog } = require("./reserveTableDialog");
const { SetAlarmDialog } = require("./setAlarmDialog");
```

También se necesitará crear un conjunto de diálogos y agregarle todos los diálogos que vamos a usar.

Se definirán los pasos en cascada del diálogo principal como funciones en la clase, en lugar de insertados. Se va a usar `bind()` con estas funciones, de modo que, desde dentro de la función, `this` se resuelva correctamente.

Este es nuestro constructor de bot actualizado.

```JavaScript
constructor(conversationState, userState) {
    // Record the conversation and user state management objects.
    this.conversationState = conversationState;
    this.userState = userState;

    // Create our state property accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    this.userInfoAccessor = userState.createProperty(USER_INFO_PROPERTY);

    // Create our bot's dialog set, adding a main dialog and the three component dialogs.
    this.dialogs = new DialogSet(this.dialogStateAccessor)
        .add(new CheckInDialog('checkInDialog'))
        .add(new ReserveTableDialog('reserveTableDialog'))
        .add(new SetAlarmDialog('setAlarmDialog'))
        .add(new WaterfallDialog('mainDialog', [
            this.promptForChoice.bind(this),
            this.startChildDialog.bind(this),
            this.saveResult.bind(this)
    ]));
}
```

Debajo del constructor de bot, agregue el siguiente código que implementa los pasos para el diálogo principal.

```JavaScript
async promptForChoice(step) {
    const menu = ["Reserve Table", "Wake Up"];
    await step.context.sendActivity(MessageFactory.suggestedActions(menu, 'How can I help you?'));
    return Dialog.EndOfTurn;
}

async startChildDialog(step) {
    // Get the user's info.
    const user = await this.userInfoAccessor.get(step.context);
    // Check the user's input and decide which dialog to start.
    // Pass in the guest info when starting either of the child dialogs.
    switch (step.result) {
        case "Reserve Table":
            return await step.beginDialog('reserveTableDialog', user.guestInfo);
            break;
        case "Wake Up":
            return await step.beginDialog('setAlarmDialog', user.guestInfo);
            break;
        default:
            await step.context.sendActivity("Sorry, I don't understand that command. Please choose an option from the list.");
            return await step.replaceDialog('mainDialog');
            break;
    }
}

async saveResult(step) {
    // Process the return value from the child dialog.
    if (step.result) {
        const user = await this.userInfoAccessor.get(step.context);
        if (step.result.table) {
            // Store the results of the reserve-table dialog.
            user.table = step.result.table;
        } else if (step.result.alarm) {
            // Store the results of the set-wake-up-call dialog.
            user.alarm = step.result.alarm;
        }
        await this.userInfoAccessor.set(step.context, user);
    }
    // Restart the main menu dialog.
    return await step.replaceDialog('mainDialog'); // Show the menu again
}
```

Y ahora actualice el controlador de turno del bot:

```JavaScript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        const user = await this.userInfoAccessor.get(turnContext, {});
        const dc = await this.dialogs.createContext(turnContext);
        const dialogTurnResult = await dc.continueDialog();
        if (dialogTurnResult.status === DialogTurnStatus.complete) {
            user.guestInfo = dialogTurnResult.result;
            await this.userInfoAccessor.set(turnContext, user);
            await dc.beginDialog('mainDialog');
        } else if (!turnContext.responded) {
            if (!user.guestInfo) {
                await dc.beginDialog('checkInDialog');
            } else {
                await dc.beginDialog('mainDialog');
            }
        }
        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

Como puede ver, los diálogos de componente se agregan al diálogo principal del bot de manera parecida a como se agregan los [avisos](bot-builder-prompts.md) a un diálogo. Puede agregar tantos diálogos secundarios al diálogo principal como desee. Cada módulo permitiría agregar funcionalidades y servicios adicionales que el bot puede ofrecer a sus usuarios.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ya sabe cómo usar diálogos de componentes, echemos un vistazo al uso de Language Understanding (LUIS) para ayudar al bot a decidir cuándo empezar los diálogos.

> [!div class="nextstepaction"]
> [Uso de LUIS para el reconocimiento de lenguaje](./bot-builder-howto-v4-luis.md)

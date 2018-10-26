---
title: Administración de un flujo de conversación complejo con diálogos | Microsoft Docs
description: Aprenda a administrar un flujo de conversación complejo con diálogos en el SDK de Bot Builder para Node.js.
keywords: flujo de conversación complejo, repetir, bucle, menú, diálogos, avisos, cascadas, conjunto de diálogos
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 10/03/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 871bfd9f8d693c5082fe1ccf38349f4d3d46ece2
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326572"
---
# <a name="manage-complex-conversation-flows-with-dialogs"></a>Administración de flujos de conversación complejos con diálogos

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

En el último artículo, se demostró el uso de la biblioteca de diálogos para administrar conversaciones sencillas. En [flujos de conversación simples](bot-builder-dialog-manage-conversation-flow.md), el usuario comienza desde el primer paso de una *cascada*, continúa hasta el último paso y el intercambio de conversación finaliza. En este artículo, usaremos diálogos para administrar conversaciones más complejas con partes que se pueden bifurcar y repetir en bucle. Para ello, se usarán varios métodos definidos en el contexto del diálogo y el contexto de pasos de cascada, y se pasarán argumentos entre las distintas partes del diálogo.

Consulte [Biblioteca de diálogos](bot-builder-concept-dialog.md) para más información de contexto sobre los diálogos.

Con el fin de proporcionarle más control sobre la *pila de diálogos*, la **biblioteca de diálogos** ofrece un método _replace dialogs_. Este método le permite intercambiar el diálogo activo en ese momento por otro y, al mismo tiempo, mantener el estado y el flujo de la conversación. Los métodos _begin dialog_ y _replace dialog_ le permiten crear bifurcaciones y bucles según sea necesario para crear interacciones más complejas. Si la complejidad de la conversación aumenta hasta tal punto que los diálogos en cascada se vuelven difíciles de administrar, investigue el uso de [diálogos de componentes](bot-builder-compositcontrol.md) o la construcción de una clase de administración de diálogos personalizada basada en la clase base `Dialog`.

En este artículo se crearán diálogos de ejemplo para un bot de conserje de un hotel que podrían usar los huéspedes para acceder a servicios comunes: reservar una mesa en el restaurante del hotel y pedir una comida al servicio de habitaciones.  Cada una de estas características, junto con un menú que las conecta, se creará en forma de diálogos de un conjunto de diálogos.

El diálogo de nivel superior del bot proporciona al huésped estas dos opciones. Si el huésped quiere reservar una mesa, el diálogo de nivel superior usa el método asincrónico _begin dialog_ para iniciar el diálogo de reserva de la mesa. Si el huésped quiere pedir al servicio de habitaciones, el diálogo de nivel superior inicia en su lugar el pedido de la cena.

El diálogo de pedidos de cena primero pide al huésped que seleccione elementos de comida de un menú y, a continuación, solicita el número de habitación. La selección de elementos de comida es _también_ un diálogo: comienza a funcionar varias veces a medida que el huésped selecciona los elementos del menú antes de enviar el pedido de la cena.

Este diagrama muestra los diálogos que se van a crear en este artículo y su relación entre sí.

![Ilustración de los diálogos que se usan en este artículo](~/media/complex-conversation-flows.png)

## <a name="how-to-branch"></a>Cómo bifurcar

El contexto del diálogo mantiene una _pila de diálogos_ y, para cada diálogo de la pila, lleva el seguimiento de cuál es el siguiente paso. Su método _begin dialog_ inserta un diálogo en la parte superior de la pila y su método _end dialog_ retira el diálogo superior de la pila.

Para realizar la bifurcación, elija uno de los diálogos secundarios para comenzar. Para más información sobre este concepto, consulte [Bifurcación de una conversación](bot-builder-concept-dialog.md#branch-a-conversation).

## <a name="how-to-loop"></a>Cómo repetir en bucle

El método _replace dialog_ del contexto del diálogo le permite reemplazar el diálogo superior de la pila. El estado del diálogo antiguo se descarta y el diálogo nuevo se inicia desde el principio. Puede usar este método para crear un bucle al reemplazar un diálogo por sí mismo. Sin embargo, para [conservar la información que el bot ya ha recopilado](bot-builder-howto-v4-state.md), deberá pasar esa información a la nueva instancia del diálogo en la llamada al método _replace dialog_.

En el ejemplo siguiente, verá que el pedido al servicio de habitaciones está almacenado en el estado del diálogo y que, cuando se repite el diálogo `orderPrompt`, se pasa el estado actual del diálogo para que el estado del diálogo nuevo pueda agregar más elementos a él. Si prefiere almacenar esta información en un estado del bot fuera del diálogo, consulte cómo [conservar los datos de usuario](bot-builder-tutorial-persist-user-inputs.md).

## <a name="create-the-dialogs-for-the-hotel-bot"></a>Creación de los diálogos para el bot del hotel

En esta sección vamos a crear los diálogos para administrar una conversación para el bot del hotel descrito.

### <a name="install-the-dialogs-library"></a>Instalación de la biblioteca de diálogos

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Comenzaremos a partir de una plantilla EchoBot básica. Para obtener instrucciones, consulte la [guía de inicio rápido para .NET](../dotnet/bot-builder-dotnet-sdk-quickstart.md).

Para usar diálogos, instale el paquete NuGet `Microsoft.Bot.Builder.Dialogs` para su proyecto o solución.
A continuación, haga referencia a la biblioteca Dialogs en las instrucciones using de los archivos de código según sea necesario.

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Comenzaremos a partir de una plantilla EchoBot. Para obtener instrucciones, consulte la [guía de inicio rápido para JavaScript](../javascript/bot-builder-javascript-quickstart.md).



La biblioteca `botbuilder-dialogs` se puede descargar desde NPM. Para instalar la biblioteca `botbuilder-dialogs`, ejecute el siguiente comando de NPM:

```cmd
npm install --save botbuilder-dialogs
```

Para usar **dialogs** en el bot, inclúyalo en el código del bot. Por ejemplo, agregue esto a su archivo **index.js**:

```javascript
const { DialogSet } = require('botbuilder-dialogs');
```

Y esto a su archivo **bot.js**:

```javascript
const { DialogSet, NumberPrompt, ChoicePrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

---

### <a name="create-a-dialog-set"></a>Creación de un conjunto de diálogos

Cree un _conjunto de diálogos_ al que vamos a agregar todos los diálogos de este ejemplo.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Cree una clase **HotelDialog** y agregue las instrucciones using que se necesitan.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;
using Microsoft.Bot.Schema;
```

Derive la clase de **DialogSet**. Incluya un constructor que tome un parámetro `IStatePropertyAccessor<DialogState>` para administrar el estado interno de una instancia del conjunto de diálogos. También, defina los identificadores y las claves que se usarán para identificar los diálogos, los avisos y la información de estado de este conjunto de diálogos.

```csharp
/// <summary>Contains the set of dialogs and prompts for the hotel bot.</summary>
public class HotelDialogs : DialogSet
{
    /// <summary>The ID of the top-level dialog.</summary>
    public const string MainMenu = "mainMenu";

    public HotelDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
        : base(dialogStateAccessor)
    {
    }

    /// <summary>Contains the IDs for the other dialogs in the set.</summary>
    private static class Dialogs
    {
        public const string OrderDinner = "orderDinner";
        public const string OrderPrompt = "orderPrompt";
        public const string ReserveTable = "reserveTable";
    }

    /// <summary>Contains the IDs for the prompts used by the dialogs.</summary>
    private static class Inputs
    {
        public const string Choice = "choicePrompt";
        public const string Number = "numberPrompt";
    }

    /// <summary>Contains the keys used to manage dialog state.</summary>
    private static class Outputs
    {
        public const string OrderCart = "orderCart";
        public const string OrderTotal = "orderTotal";
        public const string RoomNumber = "roomNumber";
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el archivo **index.js**, agregue código para crear un descriptor de acceso de propiedad de estado para administrar el estado del diálogo y úselo para crear el conjunto de diálogos que se emplearán para el bot.

```javascript
// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const dialogStateAccessor = conversationState.createProperty('dialogState');

// Create a dialog set for the bot.
const dialogSet = new DialogSet(dialogStateAccessor);

// Create the bot.
const bot = new MyBot(conversationState, dialogSet)
```

A continuación, actualice la actividad que procesa la llamada para usar el objeto de bot.

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to the bot's turn handler.
        await bot.onTurn(context);
    });
});
```

---

### <a name="add-the-prompts-to-the-set"></a>Adición de los avisos al conjunto

Vamos a usar un elemento **ChoicePrompt** para preguntar a los clientes si desean pedir cena o reservar una mesa y también qué opción seleccionar del menú de cenas. Y vamos a usar un elemento **NumberPrompt** para preguntar al cliente el número de habitación cuando pida la cena.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En el constructor **HotelDialogs**, agregue los dos avisos.

```csharp
// Add the prompts.
Add(new ChoicePrompt(Inputs.Choice));
Add(new NumberPrompt<int>(Inputs.Number));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Actualice el constructor del bot y agregue dos avisos al conjunto de diálogos.

```javascript
constructor(conversationState, dialogSet) {
    // Creates a new state accessor property.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.countProperty = conversationState.createProperty(TURN_COUNTER_PROPERTY);
    this.conversationState = conversationState;
    this.dialogSet = dialogSet;

    this.dialogSet.add(new ChoicePrompt('choicePrompt'));
    this.dialogSet.add(new NumberPrompt('numberPrompt'));
}
```

---

### <a name="define-some-of-the-supporting-information"></a>Definición de la información auxiliar

Puesto que necesitamos información sobre cada opción en el menú de cenas, vamos a configurarlo ahora.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Cree una clase estática privada llamada **Lists** que va a contener esta información. También crearemos las clases privadas **WelcomeChoice** y **MenuChoice** para contener la información sobre cada opción.

También vamos a agregar información en la lista de opciones en el diálogo de bienvenida de nivel superior y vamos a crear las listas de apoyo que usaremos más adelante para pedir esta información a los clientes. Es un poco de trabajo por adelantado, pero hará que el código del diálogo sea más sencillo.

```csharp
/// <summary>Describes an option for the top-level dialog.</summary>
private class WelcomeChoice
{
    /// <summary>Gets or sets the text to show the guest for this option.</summary>
    public string Description { get; set; }

    /// <summary>Gets or sets the ID of the associated dialog for this option.</summary>
    public string DialogName { get; set; }
}

/// <summary>Describes an option for the food-selection dialog.</summary>
/// <remarks>We have two types of options. One represents meal items that the guest
/// can add to their order. The other represents a request to process or cancel the
/// order.</remarks>
private class MenuChoice
{
    /// <summary>The request text for cancelling the meal order.</summary>
    public const string Cancel = "Cancel order";

    /// <summary>The request text for processing the meal order.</summary>
    public const string Process = "Process order";

    /// <summary>Gets or sets the name of the meal item or the request.</summary>
    public string Name { get; set; }

    /// <summary>Gets or sets the price of the meal item; or NaN for a request.</summary>
    public double Price { get; set; }

    /// <summary>Gets the text to show the guest for this option.</summary>
    public string Description => double.IsNaN(Price) ? Name : $"{Name} - ${Price:0.00}";
}
```

```csharp
/// <summary>Contains the lists used to present options to the guest.</summary>
private static class Lists
{
    /// <summary>Gets the options for the top-level dialog.</summary>
    public static List<WelcomeChoice> WelcomeOptions { get; } = new List<WelcomeChoice>
    {
        new WelcomeChoice { Description = "Order dinner", DialogName = Dialogs.OrderDinner },
        new WelcomeChoice { Description = "Reserve a table", DialogName = Dialogs.ReserveTable },
    };

    private static readonly List<string> _welcomeList = WelcomeOptions.Select(x => x.Description).ToList();

    /// <summary>Gets the choices to present in the choice prompt for the top-level dialog.</summary>
    public static IList<Choice> WelcomeChoices { get; } = ChoiceFactory.ToChoices(_welcomeList);

    /// <summary>Gets the reprompt action for the top-level dialog.</summary>
    public static Activity WelcomeReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(_welcomeList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }

    /// <summary>Gets the options for the food-selection dialog.</summary>
    public static List<MenuChoice> MenuOptions { get; } = new List<MenuChoice>
    {
        new MenuChoice { Name = "Potato Salad", Price = 5.99 },
        new MenuChoice { Name = "Tuna Sandwich", Price = 6.89 },
        new MenuChoice { Name = "Clam Chowder", Price = 4.50 },
        new MenuChoice { Name = MenuChoice.Process, Price = double.NaN },
        new MenuChoice { Name = MenuChoice.Cancel, Price = double.NaN },
    };

    private static readonly List<string> _menuList = MenuOptions.Select(x => x.Description).ToList();

    /// <summary>Gets the choices to present in the choice prompt for the food-selection dialog.</summary>
    public static IList<Choice> MenuChoices { get; } = ChoiceFactory.ToChoices(_menuList);

    /// <summary>Gets the reprompt action for the food-selection dialog.</summary>
    public static Activity MenuReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(_menuList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el archivo **bot.js**, cree una constante **dinnerMenu** que contenga la información.

```javascript
const dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50",
        "Process order", "Cancel"],
    "Potato Salad - $5.99": {
        Description: "Potato Salad",
        Price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        Description: "Tuna Sandwich",
        Price: 6.89
    },
    "Clam Chowder - $4.50": {
        Description: "Clam Chowder",
        Price: 4.50
    }
}
```

---

### <a name="create-the-welcome-dialog"></a>Creación del diálogo de bienvenida

Este diálogo usa un elemento `ChoicePrompt` para mostrar el menú y luego espera a que el usuario elija una opción. Cuando el usuario elige `Order dinner` o `Reserve a table`, se inicia el diálogo apropiado. Cuando finaliza el diálogo secundario, en lugar de simplemente terminar el diálogo del último paso y dejar que el usuario se pregunte qué pasa, el diálogo `mainMenu` se repite iniciando de nuevo el diálogo `mainMenu`. Con esta simple estructura, el bot siempre mostrará el menú y el usuario siempre sabrá cuáles son sus opciones.

El diálogo `mainMenu` contiene estos pasos de cascada:

* En el primer paso de la cascada, se inicializa o se borra el estado del diálogo, se saluda al cliente y se le pide que elija entre las opciones disponibles: `Order dinner` o `Reserve a table`.
* En el segundo paso, se recupera su selección y, a continuación, se inicia el diálogo secundario asociado a esa selección. Una vez que el diálogo secundario finaliza, este diálogo se reanudará en el paso siguiente.
* En el último paso, se usa el método _replace dialog_ para reemplazar este diálogo por una nueva instancia de sí mismo, lo que convierte el diálogo de bienvenida en un menú en bucle.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dentro del constructor, agregue el diálogo en cascada. A continuación, defina los pasos de la cascada. En este caso, se han definido en una clase anidada.

Dentro del constructor **HotelDialogs**.

```csharp
// Define the steps for and add the main welcome dialog.
WaterfallStep[] welcomeDialogSteps = new WaterfallStep[]
{
    MainDialogSteps.PresentMenuAsync,
    MainDialogSteps.ProcessInputAsync,
    MainDialogSteps.RepeatMenuAsync,
};

Add(new WaterfallDialog(MainMenu, welcomeDialogSteps));
```

Dentro de la clase **HotelDialogs**, agregue las definiciones de paso.

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-dinner dialog.
/// </summary>
private static class MainDialogSteps
{
    public static async Task<DialogTurnResult> PresentMenuAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Greet the guest and ask them to choose an option.
        await stepContext.Context.SendActivityAsync(
            "Welcome to Contoso Hotel and Resort.",
            cancellationToken: cancellationToken);
        return await stepContext.PromptAsync(
            Inputs.Choice,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("How may we serve you today?"),
                RetryPrompt = Lists.WelcomeReprompt,
                Choices = Lists.WelcomeChoices,
            },
            cancellationToken);
    }

    public static async Task<DialogTurnResult> ProcessInputAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Begin a child dialog associated with the chosen option.
        var choice = (FoundChoice)stepContext.Result;
        var dialogId = Lists.WelcomeOptions[choice.Index].DialogName;

        return await stepContext.BeginDialogAsync(dialogId, null, cancellationToken);
    }

    public static async Task<DialogTurnResult> RepeatMenuAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Start this dialog over again.
        return await stepContext.ReplaceDialogAsync(MainMenu, null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el constructor de bot, agregue el diálogo en cascada `mainMenu`.

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected otherwise, show
// the menu again.
this.dialogSet.add(new WaterfallDialog('mainMenu', [
    async function (step) {
        // Welcome the user and send a prompt.
        await step.context.sendActivity("Welcome to Contoso Hotel and Resort.");
        return await step.prompt('choicePrompt', "How may we serve you today?", ['Order Dinner', 'Reserve a table']);
    },
    async function (step) {
        // Handle the user's response to the previous prompt and branch the dialog.
        if (step.result.value.match(/order dinner/ig)) {
            return await step.beginDialog('orderDinner');
        } else if (step.result.value.match(/reserve a table/ig)) {
            return await step.beginDialog('reserveTable');
        }
    },
    async function (step) {
        // Calling replaceDialog will loop the main menu
        return await step.replaceDialog('mainMenu');
    }
]));
```

---

### <a name="create-the-order-dinner-dialog"></a>Creación del diálogo de pedidos de cena

En el diálogo de pedidos de cena, se da la bienvenida al cliente al "servicio de pedidos de cena" y se inicia inmediatamente el diálogo de selección de comidas, que abordaremos en la sección siguiente. Lo importante es que, si el cliente pide al servicio que procese el pedido, el diálogo de selección de comidas devuelve la lista de elementos del pedido. Para finalizar el proceso completo, este diálogo solicita un número de habitación para entregar la comida y, a continuación, envía un mensaje de confirmación. Si el cliente cancela su pedido, este diálogo no pide un número de habitación y finaliza inmediatamente.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Agregue una estructura de datos a la clase **HotelDialog** que podremos usar para hacer un seguimiento del pedido de cena del cliente. Puesto que se conservará en el estado del diálogo, la clase necesita el constructor predeterminado para la serialización.

```csharp
/// <summary>Contains the guest's dinner order.</summary>
private class OrderCart : List<MenuChoice>
{
    public OrderCart() : base() { }

    public OrderCart(OrderCart other) : base(other) { }
}
```

Dentro del constructor, agregue el diálogo en cascada. A continuación, defina los pasos de la cascada. En este caso, se han definido en una clase anidada.

Dentro del constructor **HotelDialogs**.

```csharp
// Define the steps for and add the order-dinner dialog.
WaterfallStep[] orderDinnerDialogSteps = new WaterfallStep[]
{
    OrderDinnerSteps.StartFoodSelectionAsync,
    OrderDinnerSteps.GetRoomNumberAsync,
    OrderDinnerSteps.ProcessOrderAsync,
};

Add(new WaterfallDialog(Dialogs.OrderDinner, orderDinnerDialogSteps));
```

Dentro de la clase **HotelDialogs**, agregue las definiciones de paso.

* En el primer paso, se envía un mensaje de bienvenida y se inicia el diálogo de selección de comidas.
* En el segundo paso, comprobamos si el diálogo de selección de comidas devuelve un carro del pedido.
  * Si es así, se le pedirá su número de habitación y se guarda la información del carro del pedido.
  * Si no, se supone que canceló su pedido y se llama a método _end dialog_ para finalizar este diálogo.
* En el último paso, se registra el número de habitación y se envía un mensaje de confirmación antes de finalizar. Este es el paso en que el bot reenviaría el pedido a un servicio de procesamiento.

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-dinner dialog.
/// </summary>
private static class OrderDinnerSteps
{
    public static async Task<DialogTurnResult> StartFoodSelectionAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync(
            "Welcome to our Dinner order service.",
            cancellationToken: cancellationToken);

        // Start the food selection dialog.
        return await stepContext.BeginDialogAsync(Dialogs.OrderPrompt, null, cancellationToken);
    }

    public static async Task<DialogTurnResult> GetRoomNumberAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        if (stepContext.Result != null && stepContext.Result is OrderCart cart)
        {
            // If there are items in the order, record the order and ask for a room number.
            stepContext.Values[Outputs.OrderCart] = cart;
            return await stepContext.PromptAsync(
                Inputs.Number,
                new PromptOptions
                {
                    Prompt = MessageFactory.Text("What is your room number?"),
                    RetryPrompt = MessageFactory.Text("Please enter your room number."),
                },
                cancellationToken);
        }
        else
        {
            // Otherwise, assume the order was cancelled by the guest and exit.
            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
    }

    public static async Task<DialogTurnResult> ProcessOrderAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Get and save the guest's answer.
        var roomNumber = (int)stepContext.Result;
        stepContext.Values[Outputs.RoomNumber] = roomNumber;

        // Process the dinner order using the collected order cart and room number.

        await stepContext.Context.SendActivityAsync(
            $"Thank you. Your order will be delivered to room {roomNumber} within 45 minutes.",
            cancellationToken: cancellationToken);
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el constructor de bot, agregue el diálogo en cascada `orderDinner`.

```javascript
// Order dinner:
// Help user order dinner from a menu
this.dialogSet.add(new WaterfallDialog('orderDinner', [
    async function (step) {
        await step.context.sendActivity("Welcome to our dinner order service.");

        return await step.beginDialog('orderPrompt', step.values.orderCart = {
            orders: [],
            total: 0
        }); // Prompt for orders
    },
    async function (step) {
        if (step.result == "Cancel") {
            return await step.endDialog();
        } else {
            return await step.prompt('numberPrompt', "What is your room number?");
        }
    },
    async function (step) {
        await step.context.sendActivity(`Thank you. Your order will be delivered to room ${step.result} within 45 minutes.`);
        return await step.endDialog();
    }
]));
```

---

### <a name="create-the-order-prompt-dialog"></a>Creación del diálogo de pedidos

En el diálogo de selección de comidas, presentaremos al cliente una lista de opciones que incluye los elementos de la cena que pueden pedir y las dos solicitudes de procesamiento de pedidos. Este diálogo se repite hasta que el cliente decide que el bot procese o cancele su pedido.

* Cuando el cliente selecciona un elemento de la cena, lo agregamos a su _carro_.
* Si el cliente elige procesar su pedido, primero comprobamos si el carro está vacío.
  * Si está vacío, se envía un mensaje de error y continúa el bucle.
  * En caso contrario, se finaliza este diálogo y se devuelve la información del carro al diálogo primario.
* Si el cliente elige cancelar su pedido, se finaliza el diálogo y no se devuelve información del carro.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dentro del constructor, agregue el diálogo en cascada. A continuación, defina los pasos de la cascada. En este caso, se han definido en una clase anidada.

Dentro del constructor **HotelDialogs**.

```csharp
// Define the steps for and add the order-prompt dialog.
WaterfallStep[] orderPromptDialogSteps = new WaterfallStep[]
{
    OrderPromptSteps.PromptForItemAsync,
    OrderPromptSteps.ProcessInputAsync,
};

Add(new WaterfallDialog(Dialogs.OrderPrompt, orderPromptDialogSteps));
```

* En el primer paso se inicializa el estado del diálogo. Si los argumentos de entrada del diálogo contienen información del carro, la guardamos en el estado del diálogo; en caso contrario, se crea y se agrega un carro vacío. A continuación, se le pedirá al cliente que seleccione una opción del menú de cenas.
* En el paso siguiente, nos centramos en la opción que el cliente ha seleccionado:
  * Si es una solicitud para procesar el pedido, se comprueba si el carro contiene algún elemento.
    * Si es así, se finaliza el diálogo y se devuelve el contenido del carro.
    * En caso contrario, se envía un mensaje de error al cliente y se empieza de nuevo desde el principio del diálogo.
  * Si es una solicitud para cancelar el pedido, se finaliza el diálogo y se devuelve un diccionario vacío.
  * Si es un elemento de la cena, se agrega al carro, se envía un mensaje de estado y el diálogo vuelve a comenzar, pasando el estado actual del pedido.

Dentro de la clase **HotelDialogs**, agregue las definiciones de paso.

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-prompt dialog.
/// </summary>
private static class OrderPromptSteps
{
    public static async Task<DialogTurnResult> PromptForItemAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // First time through, options will be null.
        var cart = (stepContext.Options is OrderCart oldCart && oldCart != null)
            ? new OrderCart(oldCart) : new OrderCart();

        stepContext.Values[Outputs.OrderCart] = cart;
        stepContext.Values[Outputs.OrderTotal] = cart.Sum(item => item.Price);

        return await stepContext.PromptAsync(
            Inputs.Choice,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("What would you like?"),
                RetryPrompt = Lists.MenuReprompt,
                Choices = Lists.MenuChoices,
            },
            cancellationToken);
    }

    public static async Task<DialogTurnResult> ProcessInputAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Get the guest's choice.
        var choice = (FoundChoice)stepContext.Result;
        var menuOption = Lists.MenuOptions[choice.Index];

        // Get the current order from dialog state.
        var cart = (OrderCart)stepContext.Values[Outputs.OrderCart];

        if (menuOption.Name is MenuChoice.Process)
        {
            if (cart.Count > 0)
            {
                // If there are any items in the order, then exit this dialog,
                // and return the list of selected food items.
                return await stepContext.EndDialogAsync(cart, cancellationToken);
            }
            else
            {
                // Otherwise, send an error message and restart from
                // the beginning of this dialog.
                await stepContext.Context.SendActivityAsync(
                    "Your cart is empty. Please add at least one item to the cart.",
                    cancellationToken: cancellationToken);
                return await stepContext.ReplaceDialogAsync(Dialogs.OrderPrompt, null, cancellationToken);
            }
        }
        else if (menuOption.Name is MenuChoice.Cancel)
        {
            await stepContext.Context.SendActivityAsync(
                "Your order has been cancelled.",
                cancellationToken: cancellationToken);

            // Exit this dialog, returning null.
            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
        else
        {
            // Add the selected food item to the order and update the order total.
            cart.Add(menuOption);
            var total = (double)stepContext.Values[Outputs.OrderTotal] + menuOption.Price;
            stepContext.Values[Outputs.OrderTotal] = total;

            await stepContext.Context.SendActivityAsync(
                $"Added {menuOption.Name} (${menuOption.Price:0.00}) to your order." +
                    Environment.NewLine + Environment.NewLine +
                    $"Your current total is ${total:0.00}.",
                cancellationToken: cancellationToken);

            // Present the order options again, passing in the current order state.
            return await stepContext.ReplaceDialogAsync(Dialogs.OrderPrompt, cart);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el constructor de bot, agregue el diálogo en cascada `orderPrompt`.

```javascript
// Helper dialog to repeatedly prompt user for orders
this.dialogSet.add(new WaterfallDialog('orderPrompt', [
    async function (step) {
        // Define a new cart of one does not exists
        step.values.orderCart = step.options;

        return await step.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function (step) {
        const choice = step.result;
        if (choice.value.match(/process order/ig)) {
            if (step.values.orderCart.orders.length > 0) {
                // Process the order
                await step.context.sendActivity("Processing your order.");
                // ...
                step.values.orderCart = undefined; // Reset cart
                return await step.endDialog();
            } else {
                await step.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            }
        } else if (choice.value.match(/cancel/ig)) {
            await step.context.sendActivity("Your order has been canceled.");
            return await step.endDialog(choice.value);
        } else {
            var item = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if (!item) {
                await step.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");

                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            } else {
                // Add the item to cart
                step.values.orderCart.orders.push(item);
                step.values.orderCart.total += item.Price;

                await step.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${step.values.orderCart.total}`);

                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            }
        }
    }
]));
```

El código de ejemplo anterior muestra que el diálogo principal `orderDinner` usa un diálogo auxiliar llamado `orderPrompt` para controlar las opciones del usuario. El diálogo `orderPrompt` muestra el menú de elementos de comida, pide al usuario que elija un elemento, agrega el elemento al carro y le pregunta de nuevo en un bucle. Esto permite al usuario agregar varios artículos a su pedido. El diálogo se repite hasta que el usuario elige `Process order` o `Cancel`. En ese momento, la ejecución se transfiere al diálogo principal (el diálogo `orderDinner`). El diálogo `orderDinner` realiza algunas tareas de última hora si el usuario quiere procesar el pedido; en caso contrario, finaliza y devuelve la ejecución de nuevo a su diálogo principal (p. ej.: `mainMenu`). El diálogo `mainMenu` a su vez continúa ejecutando el último paso que es simplemente volver a mostrar las opciones del menú principal.

---

### <a name="create-the-reserve-table-dialog"></a>Creación del diálogo de reserva de mesa

<!-- TODO: Update the "Manage simple conversation flows" topic to use a reserveTable dialog, and then reference that from here. -->

Para no extender este ejemplo, se muestra solo una implementación mínima del diálogo `reserveTable`.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dentro del constructor, agregue el diálogo en cascada. A continuación, defina los pasos de la cascada. En este caso, se han definido en una clase anidada.

Dentro del constructor **HotelDialogs**.

```csharp
// Define the steps for and add the reserve-table dialog.
WaterfallStep[] reserveTableDialogSteps = new WaterfallStep[]
{
    ReserveTableSteps.StubAsync,
};

Add(new WaterfallDialog(Dialogs.ReserveTable, reserveTableDialogSteps));
```

Dentro de la clase **HotelDialogs**, agregue las definiciones de paso.

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the reserve-table dialog.
/// </summary>
private static class ReserveTableSteps
{
    public static async Task<DialogTurnResult> StubAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync(
            "Your table has been reserved.",
            cancellationToken: cancellationToken);

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el constructor de bot, agregue el diálogo en cascada `reserveTable` del marcador de posición.

```javascript
// Reserve a table:
// Help the user to reserve a table
this.dialogSet.add(new WaterfallDialog('reserveTable', [
    // Replace this waterfall with your reservation steps.
    async function(step){
        await step.context.sendActivity("Your table has been reserved");
        await step.endDialog();
    }
]));
```

---

### <a name="update-the-bot-code-to-call-the-dialogs"></a>Actualización del código de bot para llamar a los diálogos

Actualice el código del controlador de turnos del bot para llamar al diálogo.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Cambie el nombre de **EchoBotAccessors.cs** por **BotAccessors.cs** y cambie el nombre de la clase de `EchoBotAccessors` a `BotAccessors`. Actualice las instrucciones using y la definición de clase para proporcionar el descriptor de acceso de propiedad de estado que se necesita para este bot.

```csharp
using System;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
```

```csharp
/// <summary>
/// This class is created as a Singleton and passed into the IBot-derived constructor.
///  - See <see cref="EchoWithCounterBot"/> constructor for how that is injected.
///  - See the Startup.cs file for more details on creating the Singleton that gets
///    injected into the constructor.
/// </summary>
public class BotAccessors
{
    /// <summary>
    /// Initializes a new instance of the <see cref="BotAccessors"/> class.
    /// Contains the <see cref="ConversationState"/> and associated <see cref="IStatePropertyAccessor{T}"/>.
    /// </summary>
    /// <param name="conversationState">The state object that stores the counter.</param>
    public BotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    /// <summary>
    /// Gets the <see cref="IStatePropertyAccessor{T}"/> name used for the <see cref="DialogState"/> accessor.
    /// </summary>
    /// <remarks>Accessors require a unique name.</remarks>
    /// <value>The accessor name for the dialog state accessor.</value>
    public static string DialogStateAccessorName { get; } = $"{nameof(BotAccessors)}.DialogState";

    /// <summary>
    /// Gets or sets the DialogState property accessor.
    /// </summary>
    /// <value>
    /// The DialogState property accessor.
    /// </value>
    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    /// <summary>
    /// Gets the <see cref="ConversationState"/> object for the conversation.
    /// </summary>
    /// <value>The <see cref="ConversationState"/> object.</value>
    public ConversationState ConversationState { get; }
}
```

Actualice el archivo **Startup.cs** para configurar el singletion `BotAccessors`.

1. Actualice las instrucciones using.

    ```csharp
    using System;
    using System.Linq;
    using Microsoft.AspNetCore.Builder;
    using Microsoft.AspNetCore.Hosting;
    using Microsoft.Bot.Builder;
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

1. Actualice la parte del método `ConfigureServices` que registra los descriptores de acceso de propiedad de estado del bot.

    ```csharp
    // Create and register state accesssors.
    // Acessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState)
        {
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorName),
        };

        return accessors;
    });
    ```

Cambie el nombre del archivo EchoWithCounterBot.cs por HotelBot.cs y cambie el nombre de la clase de EchoWithCounterBot a HotelBot.

1. Actualice el código de inicialización del bot.

    ```csharp
    private readonly BotAccessors _accessors;
    private readonly HotelDialogs _dialogs;
    private readonly ILogger _logger;

    /// <summary>
    /// Initializes a new instance of the <see cref="HotelBot"/> class.
    /// </summary>
    /// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
    /// <param name="loggerFactory">A <see cref="ILoggerFactory"/> that is hooked to the Azure App Service provider.</param>
    public HotelBot(BotAccessors accessors, ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger<HotelBot>();
        _logger.LogTrace("EchoBot turn start.");
        _accessors = accessors;
        _dialogs = new HotelDialogs(_accessors.DialogStateAccessor);
    }
    ```

1. Actualice el controlador de turnos del bot, para que ejecute el diálogo.

    ```csharp
    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        var dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            await dc.ContinueDialogAsync(cancellationToken);
            if (!turnContext.Responded)
            {
                await dc.BeginDialogAsync(HotelDialogs.MainMenu, null, cancellationToken);
            }
        }
        else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
        {
            var activity = turnContext.Activity.AsConversationUpdateActivity();
            if (activity.MembersAdded.Any(member => member.Id != activity.Recipient.Id))
            {
                await dc.BeginDialogAsync(HotelDialogs.MainMenu, null, cancellationToken);
            }
        }

        await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async onTurn(turnContext) {
    let dc = await this.dialogSet.createContext(turnContext);

    // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
    if (turnContext.activity.type === ActivityTypes.Message) {

        await dc.continueDialog();

        if (!turnContext.responded) {
            await dc.beginDialog('mainMenu');
        }
    } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate) {
        // Do we have any new members added to the conversation?
        if (turnContext.activity.membersAdded.length !== 0) {
            // Iterate over all new members added to the conversation
            for (var idx in turnContext.activity.membersAdded) {
                // Greet anyone that was not the target (recipient) of this message.
                // Since the bot is the recipient for events from the channel,
                // context.activity.membersAdded === context.activity.recipient.Id indicates the
                // bot was added to the conversation, and the opposite indicates this is a user.
                if (turnContext.activity.membersAdded[idx].id !== turnContext.activity.recipient.id) {
                    // Start the dialog.
                    await dc.beginDialog('mainMenu');
                }
            }
        }
    }

    // Save state changes
    await this.conversationState.saveChanges(turnContext);
}
```

---

## <a name="next-steps"></a>Pasos siguientes

Puede ofrecer otras opciones en el menú, como "más información" o "ayuda" para mejorar este bot. Para más información sobre la implementación de estos tipos de interrupciones, consulte [Control de las interrupciones de usuario](bot-builder-howto-handle-user-interrupt.md).

Ahora que ha aprendido a usar diálogos, avisos y cascadas para administrar flujos de conversación complejos, echemos un vistazo a cómo podemos dividir nuestros diálogos (como los diálogos `orderDinner` y `reserveTable`) en objetos independientes, en lugar de agruparlos en un conjunto de diálogos de gran tamaño.

> [!div class="nextstepaction"]
> [Creación de lógica de bot modular con control compuesto](bot-builder-compositcontrol.md)

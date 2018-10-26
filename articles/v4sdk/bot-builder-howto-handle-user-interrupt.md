---
title: Control de las interrupciones del usuario | Microsoft Docs
description: Obtenga información sobre cómo controlar el flujo de conversación directa y la interrupción del usuario.
keywords: interrumpir, interrupciones, cambio de tema, interrupción
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/20/2018
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 74d4bb07274643d61da332d6ee1cdfb1a14372dc
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998432"
---
# <a name="handle-user-interruptions"></a>Control de las interrupciones del usuario

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

El control de interrupciones es un aspecto importante de un bot sólido.

Aunque puede pensar que los usuarios siguen el flujo de conversación definido paso a paso, es muy probable que cambien de parecer o formulen una pregunta en mitad del proceso en lugar de responder la pregunta. En estas situaciones, ¿cómo controlaría su bot la entrada del usuario? ¿Como sería la experiencia del usuario? ¿Cómo mantendría los datos de estado del usuario? El hecho de controlar las interrupciones conlleva tener la certeza de que un bot está preparado para controlar situaciones como esta.

No hay ninguna respuesta correcta a estas preguntas, ya que cada situación es única según el escenario para el que se haya diseñado el bot. En este tema, se exploran algunas formas comunes de controlar las interrupciones del usuario y se sugieren algunas formas para implementarlas en su bot.

## <a name="handle-expected-interruptions"></a>Controlar interrupciones esperadas

Un flujo de conversación de procedimientos cuenta con un conjunto básico de pasos que quiere que el usuario siga y cualquier acción del usuario que no corresponda con esos pasos son posibles interrupciones. En un flujo normal, existen interrupciones que puede anticipar.

**Reserva de una mesa**: en un bot de reserva de mesa, los pasos principales pueden ser preguntar al usuario la fecha y hora, el número de personas y el nombre al que se realiza la reserva. En ese proceso, se pueden anticipar algunas interrupciones esperadas, como por ejemplo:

* `cancel`: para salir del proceso.
* `help`: para proporcionar más instrucciones acerca de este proceso.
* `more info`: para dar consejos y sugerencias o proporcionar maneras alternativas para reservar una mesa (p. ej.: un número de teléfono o una dirección de correo electrónico de contacto).
* `show list of available tables`: si es una opción; muestra una lista de mesas disponibles para la fecha y hora indicada por el usuario.

**Pedir comida a domicilio**: en un bot de pedida de comida a domicilio, los pasos principales serían proporcionar una lista de los platos del menú y permitir que el usuario agregue dichos platos a al carro. En este proceso, se pueden anticipar algunas interrupciones esperadas, como por ejemplo:

* `cancel`: para salir del proceso de pedido.
* `more info`: para proporcionar detalles nutricionales sobre cada elemento del menú.
* `help`: para proporcionar ayuda sobre cómo usar el sistema.
* `process order`: para procesar el pedido.

Puede proporcionarlas al usuario como una lista de **acciones sugeridas** o como una sugerencia de modo que el usuario al menos tenga en cuenta qué comandos puede enviar que el bot pueda entender.

Por ejemplo, en el flujo del pedido de cena, se pueden proporcionar interrupciones esperadas junto con los elementos del menú. En este caso, los elementos del menú se envían como una matriz de `choices`.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

Definiremos el conjunto de diálogos como una subclase de **DialogSet**.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;

public class OrderDinnerDialogs : DialogSet
{
    public OrderDinnerDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
        : base(dialogStateAccessor)
    {
    }
}
```

Definiremos un par de clases internas para describir el menú.

```cs
/// <summary>
/// Contains information about an item on the menu.
/// </summary>
public class DinnerItem
{
    public string Description { get; set; }

    public double Price { get; set; }
}

/// <summary>
/// Describes the dinner menu, including the items on the menu and options for
/// interrupting the ordering process.
/// </summary>
public class DinnerMenu
{
    /// <summary>Gets the items on the menu.</summary>
    public static Dictionary<string, DinnerItem> MenuItems { get; } = new Dictionary<string, DinnerItem>
    {
        ["Potato salad"] = new DinnerItem { Description = "Potato Salad", Price = 5.99 },
        ["Tuna sandwich"] = new DinnerItem { Description = "Tuna Sandwich", Price = 6.89 },
        ["Clam chowder"] = new DinnerItem { Description = "Clam Chowder", Price = 4.50 },
    };

    /// <summary>Gets all the "interruptions" the bot knows how to process.</summary>
    public static List<string> Interrupts { get; } = new List<string>
    {
        "More info", "Process order", "Help", "Cancel",
    };

    /// <summary>Gets all of the valid inputs a user can make.</summary>
    public static List<string> Choices { get; }
        = MenuItems.Select(c => c.Key).Concat(Interrupts).ToList();
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Comenzaremos a partir de una plantilla EchoBot básica. Para obtener instrucciones, consulte la [guía de inicio rápido para JavaScript](~/javascript/bot-builder-javascript-quickstart.md).

La biblioteca `botbuilder-dialogs` se puede descargar desde NPM. Para instalar la biblioteca `botbuilder-dialogs`, ejecute el siguiente comando de NPM:

```cmd
npm install --save botbuilder-dialogs
```

En el archivo **bot.js**, haga referencia a las clases a las que vamos a hacer referencia y defina los identificadores que vamos a usar para el diálogo.

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, ChoicePrompt, WaterfallDialog, DialogTurnStatus } = require('botbuilder-dialogs');

// Name for the dialog state property accessor.
const DIALOG_STATE_PROPERTY = 'dialogStateProperty';

// Name of the order-prompt dialog.
const ORDER_PROMPT = 'orderingDialog';

// Name for the choice prompt for use in the dialog.
const CHOICE_PROMPT = 'choicePrompt';
```

Defina las opciones que desea que aparezcan en el diálogo de realización de pedidos.

```javascript
// The options on the dinner menu, including commands for the bot.
const dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50",
        "Process order", "Cancel", "More info", "Help"],
    "Potato Salad - $5.99": {
        description: "Potato Salad",
        price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        description: "Tuna Sandwich",
        price: 6.89
    },
    "Clam Chowder - $4.50": {
        description: "Clam Chowder",
        price: 4.50
    }
}
```

---

En la lógica del pedido, puede comprobarlos mediante la correspondencia de cadenas o expresiones regulares.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

En primer lugar, es preciso definir un asistente que realice un seguimiento de los pedidos.

```cs
/// <summary>Helper class for storing the order.</summary>
public class Order
{
    public double Total { get; set; } = 0.0;

    public List<DinnerItem> Items { get; set; } = new List<DinnerItem>();

    public bool ReadyToProcess { get; set; } = false;

    public bool OrderProcessed { get; set; } = false;
}
```

Agregue algunas constantes para realizar un seguimiento de los identificadores que se van a necesitar.

```csharp
/// <summary>The ID of the top-level dialog.</summary>
public const string MainDialogId = "mainMenu";

/// <summary>The ID of the choice prompt.</summary>
public const string ChoicePromptId = "choicePrompt";

/// <summary>The ID of the order card value, tracked inside the dialog.</summary>
public const string OrderCartId = "orderCart";
```

Actualice el constructor para agregar el aviso que prefiramos y el diálogo en cascada. También se van a definir los métodos que implementan los pasos de la cascada.

```cs
public OrderDinnerDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
    : base(dialogStateAccessor)
{
    // Add a choice prompt for the dialog.
    Add(new ChoicePrompt(ChoicePromptId));

    // Define and add the main waterfall dialog.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptUserAsync,
        ProcessInputAsync,
    };

    Add(new WaterfallDialog(MainDialogId, steps));
}

/// <summary>
/// Defines the first step of the main dialog, which is to ask for input from the user.
/// </summary>
/// <param name="stepContext">The current waterfall step context.</param>
/// <param name="cancellationToken">The cancellation token.</param>
/// <returns>The task to perform.</returns>
private async Task<DialogTurnResult> PromptUserAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Initialize order, continuing any order that was passed in.
    Order order = (stepContext.Options is Order oldCart && oldCart != null)
        ? new Order
        {
            Items = new List<DinnerItem>(oldCart.Items),
            Total = oldCart.Total,
            ReadyToProcess = oldCart.ReadyToProcess,
            OrderProcessed = oldCart.OrderProcessed,
        }
        : new Order();

    // Set the order cart in dialog state.
    stepContext.Values[OrderCartId] = order;

    // Prompt the user.
    return await stepContext.PromptAsync(
        "choicePrompt",
        new PromptOptions
        {
            Prompt = MessageFactory.Text("What would you like for dinner?"),
            RetryPrompt = MessageFactory.Text(
                "I'm sorry, I didn't understand that. What would you like for dinner?"),
            Choices = ChoiceFactory.ToChoices(DinnerMenu.Choices),
        },
        cancellationToken);
}

/// <summary>
/// Defines the second step of the main dialog, which is to process the user's input, and
/// repeat or exit as appropriate.
/// </summary>
/// <param name="stepContext">The current waterfall step context.</param>
/// <param name="cancellationToken">The cancellation token.</param>
/// <returns>The task to perform.</returns>
private async Task<DialogTurnResult> ProcessInputAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the order cart from dialog state.
    Order order = stepContext.Values[OrderCartId] as Order;

    // Get the user's choice from the previous prompt.
    string response = (stepContext.Result as FoundChoice).Value;

    if (response.Equals("process order", StringComparison.InvariantCultureIgnoreCase))
    {
        order.ReadyToProcess = true;

        await stepContext.Context.SendActivityAsync(
            "Your order is on it's way!",
            cancellationToken: cancellationToken);

        // In production, you may want to store something more helpful.
        // "Process" the order and exit.
        order.OrderProcessed = true;
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    else if (response.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
    {
        // Cancel the order.
        await stepContext.Context.SendActivityAsync(
            "Your order has been canceled",
            cancellationToken: cancellationToken);

        // Exit without processing the order.
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    else if (response.Equals("more info", StringComparison.InvariantCultureIgnoreCase))
    {
        // Send more information about the options.
        string message = "More info: <br/>" +
            "Potato Salad: contains 330 calories per serving. Cost: 5.99 <br/>"
            + "Tuna Sandwich: contains 700 calories per serving. Cost: 6.89 <br/>"
            + "Clam Chowder: contains 650 calories per serving. Cost: 4.50";
        await stepContext.Context.SendActivityAsync(
            message,
            cancellationToken: cancellationToken);

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }
    else if (response.Equals("help", StringComparison.InvariantCultureIgnoreCase))
    {
        // Provide help information.
        string message = "To make an order, add as many items to your cart as " +
            "you like. Choose the `Process order` to check out. " +
            "Choose `Cancel` to cancel your order and exit.";
        await stepContext.Context.SendActivityAsync(
            message,
            cancellationToken: cancellationToken);

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }

    // We've checked for expected interruptions. Check for a valid item choice.
    if (!DinnerMenu.MenuItems.ContainsKey(response))
    {
        await stepContext.Context.SendActivityAsync("Sorry, that is not a valid item. " +
            "Please pick one from the menu.");

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }
    else
    {
        // Add the item to cart.
        DinnerItem item = DinnerMenu.MenuItems[response];
        order.Items.Add(item);
        order.Total += item.Price;

        // Acknowledge the input.
        await stepContext.Context.SendActivityAsync(
            $"Added `{response}` to your order; your total is ${order.Total:0.00}.",
            cancellationToken: cancellationToken);

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Defina el diálogo en el constructor de bots. Tenga en cuenta que lo _primero_ que hace el código es buscar y controlar las interrupciones y,después, pasa al siguiente paso lógico.

```javascript
constructor(conversationState) {
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    this.conversationState = conversationState;

    this.dialogs = new DialogSet(this.dialogStateAccessor)
        .add(new ChoicePrompt(CHOICE_PROMPT))
        .add(new WaterfallDialog(ORDER_PROMPT, [
            async (step) => {
                if (step.options && step.options.orders) {
                    // If an order cart was passed in, continue to use it.
                    step.values.orderCart = step.options;
                } else {
                    // Otherwise, start a new cart.
                    step.values.orderCart = {
                        orders: [],
                        total: 0
                    };
                }
                return await step.prompt(CHOICE_PROMPT, "What would you like?", dinnerMenu.choices);
            },
            async (step) => {
                const choice = step.result;
                if (choice.value.match(/process order/ig)) {
                    if (step.values.orderCart.orders.length > 0) {
                        // If the cart is not empty, process the order by returning the order to the parent context.
                        await step.context.sendActivity("Your order has been processed.");
                        return await step.endDialog(step.values.orderCart);
                    } else {
                        // Otherwise, prompt again.
                        await step.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                        return await step.replaceDialog(ORDER_PROMPT);
                    }
                } else if (choice.value.match(/cancel/ig)) {
                    // Cancel the order, and return "cancel" to the parent context.
                    await step.context.sendActivity("Your order has been canceled.");
                    return await step.endDialog("cancelled");
                } else if (choice.value.match(/more info/ig)) {
                    // Provide more information, and then continue the ordering process.
                    var msg = "More info: <br/>Potato Salad: contains 330 calories per serving. <br/>"
                        + "Tuna Sandwich: contains 700 calories per serving. <br/>"
                        + "Clam Chowder: contains 650 calories per serving."
                    await step.context.sendActivity(msg);
                    return await step.replaceDialog(ORDER_PROMPT, step.values.orderCart);
                } else if (choice.value.match(/help/ig)) {
                    // Provide help information, and then continue the ordering process.
                    var msg = `Help: <br/>`
                        + `To make an order, add as many items to your cart as you like then choose `
                        + `the "Process order" option to check out.`
                    await step.context.sendActivity(msg);
                    return await step.replaceDialog(ORDER_PROMPT, step.values.orderCart);
                } else {
                    // The user has chosen a food item from the menu. Add the item to cart.
                    var item = dinnerMenu[choice.value];
                    step.values.orderCart.orders.push(item.description);
                    step.values.orderCart.total += item.price;

                    await step.context.sendActivity(`Added ${item.description} to your cart. <br/>`
                        + `Current total: $${step.values.orderCart.total}`);

                    // Continue the ordering process.
                    return await step.replaceDialog(ORDER_PROMPT, step.values.orderCart);
                }
            }
        ]));
}
```

Actualice el controlador de turnos que llama al diálogo y muestra los resultados del proceso de realización de pedidos.

```javascript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        let dc = await this.dialogs.createContext(turnContext);
        let dialogTurnResult = await dc.continueDialog();
        if (dialogTurnResult.status === DialogTurnStatus.complete) {
            // The dialog completed this turn.
            const result = dialogTurnResult.result;
            if (!result || result === "cancelled") {
                await turnContext.sendActivity('You cancelled your order.');
            } else {
                await turnContext.sendActivity(`Your order came to $${result.total}`);
            }
        } else if (!turnContext.responded) {
            // No dialog was active.
            await turnContext.sendActivity("Let's order dinner...");
            await dc.cancelAllDialogs();
            await dc.beginDialog(ORDER_PROMPT);
        } else {
            // The dialog is active.
        }
    } else {
        await turnContext.sendActivity(`[${turnContext.activity.type} event detected]`);
    }
    // Save state changes
    await this.conversationState.saveChanges(turnContext);
}
```

---

## <a name="handle-unexpected-interruptions"></a>Controlar interrupciones inesperadas

Hay interrupciones que se encuentran fuera del ámbito de lo que su bot está diseñado para hacer.
Aunque no se pueden prever todas las interrupciones, hay patrones de interrupción que puede programar para que su bot controle.

### <a name="switching-topic-of-conversations"></a>Cambiar el tema de las conversaciones

¿Qué ocurre si el usuario está en medio de una conversación y quiere cambiar a otra conversación? Por ejemplo, el bot puede reservar una mesa y pedir una cena.
Mientras el usuario está en el flujo para _reservar una mesa_, en lugar de responder la pregunta "¿Cuántas personas están en la lista?", el usuario envía el mensaje "pedir cena". En este caso, el usuario cambió de opinión y quiere tener una conversación para pedir una cena. ¿Cómo se puede controlar esta interrupción?

Puede cambiar los temas en el flujo para pedir una cena o hacer que sea un problema temporal informando al usuario que se espera un número y solicitárselo de nuevo. Si permite cambiar los temas, a continuación, debe decidir si se guardará el progreso para que el usuario pueda continuar desde donde lo dejó o podría eliminar toda la información que recopiló para que el usuario tenga que reiniciar el proceso desde el principio la próxima vez que quiera reservar una mesa. Para obtener más información acerca de cómo administrar los datos de estado de usuario, consulte el artículo [Save state using conversation and user properties](bot-builder-howto-v4-state.md) (Guardar el estado mediante las propiedades del usuario y la conversación).

### <a name="apply-artificial-intelligence"></a>Aplicar inteligencia artificial

Para las interrupciones que no están en el ámbito, puede intentar adivinar la intención del usuario. Para hacerlo puede usar servicios de IA, como QnA Maker, LUIS o su lógica personalizada, y, luego, ofrecer sugerencias de lo que el bot considera que desea el usuario.

Por ejemplo, en medio del flujo para reservar una mesa, el usuario dice: "Quiero pedir una hamburguesa". En este flujo de conversación, el bot no sabe cómo lidiar con esa entrada. Como el flujo actual no tiene nada que ver con el pedido y el otro comando de conversación del bot es "pedir una cena", el bot no sabe qué hacer con esta entrada. Si aplica LUIS, por ejemplo, podría entrenar el modelo para que reconozca que quiere pedir comida (p. ej.: LUIS puede devolver una intención "orderFood"). Por lo tanto, el bot podría responder: "Parece que quiere pedir una comida. ¿Le gustaría cambiar a nuestro proceso de pedido de cena?" Para más información sobre el aprendizaje de LUIS y cómo detectar las intenciones del usuario, consulte [Uso de LUIS para reconocimiento del lenguaje](bot-builder-howto-v4-luis.md).

### <a name="default-response"></a>Respuesta predeterminada

Si todo lo demás falla, debe enviar una respuesta predeterminada, en lugar de no hacer nada y dejar al usuario preguntándose qué es lo que pasa. La respuesta predeterminada debe indicar al usuario cuáles son los comandos que entiende el bot, de modo que el usuario pueda retomar el proceso.

Puede comprobar nuevamente la marca de contexto **responded** en la parte final de la lógica del bot para ver si el bot envió algo al usuario durante el turno. Si el bot procesa la entrada del usuario, pero no responde, lo más probable es que el bot no sepa qué hacer con la entrada. En ese caso, puede detectarlo y enviar un mensaje predeterminado al usuario.

Si el valor predeterminado del bot es proporcionar al usuario un diálogo `mainMenu`. Depende de usted decidir qué experiencia tendrá el usuario en esta situación con el bot.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
// Check whether we replied. If not then clear the dialog stack and present the main menu.
if (!turnContext.Responded)
{
    await dc.CancelAllDialogsAsync(cancellationToken);
    await dc.BeginDialogAsync(OrderDinnerDialogs.MainDialogId, null, cancellationToken);
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Check to see if anyone replied. If not then clear all the stack and present the main menu
if (!context.responded) {
    await dc.cancelAllDialogs();
    await step.beginDialog('mainMenu');
}
```

---

## <a name="handling-global-interruptions"></a>Control de interrupciones globales

En el ejemplo anterior, se han tratado las interrupciones que pueden producirse en un turno específico de un diálogo concreto. ¿Qué ocurre si lo que se desea es controlar las interrupciones globales, las que pueden producirse en cualquier momento?

Para ello es preciso colocar la lógica de control de interrupciones en el controlador principal del bot (la función que procesa el valor `turnContext` de entrada y decide qué hacer con él).

En el ejemplo siguiente, lo _primero_ que hace el bot es comprobar si en el texto del mensaje entrante hay algún indicador de que el usuario necesita ayuda o desea cancelar una operación (dos interrupciones muy comunes que se encuentran los bots). Una vez que se completa esta comprobación, el bot llama a `dc.continueDialog()` para procesar todos los diálogos activos que siguen pendientes.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
// Check for top-level interruptions.
string utterance = turnContext.Activity.Text.Trim().ToLowerInvariant();

if (utterance == "help")
{
    // Start a general help dialog. Dialogs already on the stack remain and will continue
    // normally if the help dialog exits normally.
    await dc.BeginDialogAsync(OrderDinnerDialogs.HelpDialogId, null, cancellationToken);
}
else if (utterance == "cancel")
{
    // Cancel any dialog on the stack.
    await turnContext.SendActivityAsync("Canceled.", cancellationToken: cancellationToken);
    await dc.CancelAllDialogsAsync(cancellationToken);
}
else
{
    await dc.ContinueDialogAsync(cancellationToken);

    // Check whether we replied. If not then clear the dialog stack and present the main menu.
    if (!turnContext.Responded)
    {
        await dc.CancelAllDialogsAsync(cancellationToken);
        await dc.BeginDialogAsync(OrderDinnerDialogs.MainDialogId, null, cancellationToken);
    }
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Las interrupciones se pueden controlar antes de enviar una respuesta del usuario a un diálogo.

En este fragmento de código se da por supuesto que `helpDialog` y `mainMenu` se encuentran en el conjunto de diálogos.

```javascript
const utterance = (context.activity.text || '').trim();

// Let's look for some interruptions first!
if (utterance.match(/help/ig)) {
    // Launch a new help dialog if the user asked for help
    await dc.beginDialog('helpDialog');
} else if (utterance.match(/cancel/ig)) {
    // Cancel any active dialogs if the user says cancel
    await dc.context.sendActivity('Canceled.');
    await dc.cancelAllDialogs();
}

// If the bot hasn't yet responded...
if (!context.responded) {
    // Continue any active dialog, which might send a response...
    await dc.continueDialog();

    // Finally, if the bot still hasn't sent a response, send instructions.
    if (!context.responded) {
        await dc.cancelAllDialogs();
        await context.sendActivity("Starting the main menu...");
        await dc.beginDialog('mainMenu');
    }
}
```

---

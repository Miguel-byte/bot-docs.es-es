---
title: Administración de un flujo de conversación simple con diálogos | Microsoft Docs
description: Aprenda a administrar un flujo de conversación simple con diálogos en Bot Builder SDK para Node.js.
keywords: flujo de conversación simple, diálogos, avisos, cascadas, conjunto de diálogos
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 9/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 07035c8f0dfc7473192d8c51667ed1f5cefbc555
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999402"
---
# <a name="manage-simple-conversation-flow-with-dialogs"></a>Administración de un flujo de conversación simple con diálogos

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Mediante la biblioteca Dialogs es posible administrar flujos de conversación simples y complejos. En un flujo de conversación simple, el usuario comienza por el primer paso de una *cascada*, continúa hasta el último y la conversación finaliza. Los [flujos de conversación complejos](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md) incluyen ramas y bucles.

<!-- TODO: This paragraph belongs in a conceptual topic. -->

Los diálogos son estructuras del bot que actúan como funciones en el programa del bot. Los diálogos crean los mensajes que envía el bot y llevan a cabo las tareas de cálculo necesarias. Están diseñados para realizar operaciones específicas, en un orden concreto. Se pueden invocar de varias formas, unas veces en respuesta a un usuario y otras en respuesta a algún estímulo exterior, o por otros diálogos.

El uso de diálogos permite al desarrollador de bots guiar el flujo de la conversación. Puede crear varios diálogos y vincularlos para crear cualquier flujo de conversación que desee que administre el bot. La biblioteca **Dialogs** del SDK Bot Builder incluye características integradas como _mensajes_, _diálogos en cascada_ y _diálogos de componentes_ que le ayudan a administrar el flujo de la conversación. Puede utilizar avisos para pedir a los usuarios distintos tipos de información. Puede usar una cascada para combinar varios pasos en una secuencia. Y puede usar los diálogos de componentes para crear sistemas de diálogo modulares que contienen varios diálogos secundarios.

En este artículo, se usan _conjuntos de diálogos_ para crear un flujo de conversación que contiene tanto mensajes como cascadas. Vamos a recurrir al código del **ejemplo de mensaje de varios turnos**  [[C#](https://aka.ms/cs-multi-prompts-sample)|[JS](https://aka.ms/js-multi-prompts-sample)].

Para obtener información general acerca de los diálogos, vea [biblioteca Dialogs](bot-builder-concept-dialog.md) y [estado de diálogos](bot-builder-dialog-state.md).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Para usar diálogos en general, necesita el paquete NuGet `Microsoft.Bot.Builder.Dialogs` en su proyecto o solución.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Para usar diálogos en general, necesita la biblioteca `botbuilder-dialogs`, que puede descargarse de NPM.

---

## <a name="using-dialogs-to-guide-the-user-through-steps"></a>Uso de diálogos para guiar al usuario paso a paso

En este ejemplo, se crea un diálogo de varios pasos para solicitar información al usuario mediante un conjunto de diálogos.

### <a name="create-a-dialog-with-waterfall-steps"></a>Creación de un diálogo con pasos de cascada

**WaterfallDialog** es una implementación concreta de un diálogo que se usa normalmente para recopilar información del usuario o guiarlo en una serie de tareas. Cada uno de los pasos de la conversación se implementa como una función. En cada paso, el bot [pide al usuario que intervenga](bot-builder-prompts.md), espera una respuesta y, a continuación, pasa el resultado al paso siguiente. El resultado de la primera función se pasa como argumento a la función siguiente y así sucesivamente.

Por ejemplo, el siguiente ejemplo de código define una matriz de delegados que representan los pasos de una **cascada**. Después de cada pregunta, el bot confirma que ha recibido la información del usuario. Hay muchas maneras de guardar la información que se recopila en un diálogo. Para ver algunas de las opciones, consulte [Conservación de datos de usuario](bot-builder-tutorial-persist-user-inputs.md).

En este ejemplo se escribe la información directamente en el perfil del usuario tal como se recopila en el diálogo.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En este ejemplo, el diálogo en cascada se define en el archivo del bot.

Haga referencia a los espacios de nombres que se utilizan en este archivo.

```csharp
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;
```

Definir una propiedad de instancia para el conjunto de diálogos.

```csharp
/// <summary>
/// The <see cref="DialogSet"/> that contains all the Dialogs that can be used at runtime.
/// </summary>
private DialogSet _dialogs;
```

Cree el conjunto de diálogos dentro del constructor del bot y agregue tanto los mensajes como el diálogo en cascada al conjunto.

```csharp
/// <summary>
/// Initializes a new instance of the <see cref="MultiTurnPromptsBot"/> class.
/// </summary>
/// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
public MultiTurnPromptsBot(MultiTurnPromptsBotAccessors accessors)
{
    _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

    // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
    _dialogs = new DialogSet(accessors.ConversationDialogState);

    // This array defines how the Waterfall will execute.
    var waterfallSteps = new WaterfallStep[]
    {
        NameStepAsync,
        NameConfirmStepAsync,
        AgeStepAsync,
        ConfirmStepAsync,
        SummaryStepAsync,
    };

    // Add named dialogs to the DialogSet. These names are saved in the dialog state.
    _dialogs.Add(new WaterfallDialog("details", waterfallSteps));
    _dialogs.Add(new TextPrompt("name"));
    _dialogs.Add(new NumberPrompt<int>("age"));
    _dialogs.Add(new ConfirmPrompt("confirm"));
}
```

Y, por último, defina cada paso como un método independiente. También puede definir los pasos en línea mediante expresiones lambda.

```csharp
/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    // Running a prompt here means the next WaterfallStep will be run when the users response is received.
    return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> NameConfirmStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the current profile object from user state.
    var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

    // Update the profile.
    userProfile.Name = (string)stepContext.Result;

    // We can send messages to the user at any point in the WaterfallStep.
    await stepContext.Context.SendActivityAsync(MessageFactory.Text($"Thanks {stepContext.Result}."), cancellationToken);

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    return await stepContext.PromptAsync("confirm", new PromptOptions { Prompt = MessageFactory.Text("Would you like to give your age?") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> AgeStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    if ((bool)stepContext.Result)
    {
        // User said "yes" so we will be prompting for the age.

        // Get the current profile object from user state.
        var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

        // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is a Prompt Dialog.
        return await stepContext.PromptAsync("age", new PromptOptions { Prompt = MessageFactory.Text("Please enter your age.") }, cancellationToken);
    }
    else
    {
        // User said "no" so we will skip the next step. Give -1 as the age.
        return await stepContext.NextAsync(-1, cancellationToken);
    }
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> ConfirmStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the current profile object from user state.
    var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

    // Update the profile.
    userProfile.Age = (int)stepContext.Result;

    // We can send messages to the user at any point in the WaterfallStep.
    if (userProfile.Age == -1)
    {
        await stepContext.Context.SendActivityAsync(MessageFactory.Text($"No age given."), cancellationToken);
    }
    else
    {
        // We can send messages to the user at any point in the WaterfallStep.
        await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your age as {userProfile.Age}."), cancellationToken);
    }

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is a Prompt Dialog.
    return await stepContext.PromptAsync("confirm", new PromptOptions { Prompt = MessageFactory.Text("Is this ok?") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> SummaryStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    if ((bool)stepContext.Result)
    {
        // Get the current profile object from user state.
        var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

        // We can send messages to the user at any point in the WaterfallStep.
        if (userProfile.Age == -1)
        {
            await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your name as {userProfile.Name}."), cancellationToken);
        }
        else
        {
            await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your name as {userProfile.Name} and age as {userProfile.Age}."), cancellationToken);
        }
    }
    else
    {
        // We can send messages to the user at any point in the WaterfallStep.
        await stepContext.Context.SendActivityAsync(MessageFactory.Text("Thanks. Your profile will not be kept."), cancellationToken);
    }

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is the end.
    return await stepContext.EndDialogAsync(cancellationToken: cancellationToken);
}
```

El diálogo se ejecuta desde el bot en el controlador de turnos, que primero crea un contexto de diálogo y continúa o comienza el diálogo, según corresponda, y, después, guarda la conversación y el estado de usuario al final del turno.

```csharp
// Run the DialogSet - let the framework identify the current state of the dialog from
// the dialog stack and figure out what (if any) is the active dialog.
var dialogContext = await _dialogs.CreateContextAsync(turnContext, cancellationToken);
var results = await dialogContext.ContinueDialogAsync(cancellationToken);

// If the DialogTurnStatus is Empty we should start a new dialog.
if (results.Status == DialogTurnStatus.Empty)
{
    await dialogContext.BeginDialogAsync("details", null, cancellationToken);
}
```

```csharp
// Save the dialog state into the conversation state.
await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);

// Save the user profile updates into the user state.
await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En este ejemplo, el diálogo en cascada se define en el archivo **bot.js**.

Importe los objetos que necesita para el código.

```javascript
const { ActivityTypes } = require('botbuilder');
const { ChoicePrompt, DialogSet, NumberPrompt, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

Defina y cree el conjunto de diálogos en el constructor del bot y agregue tanto los mensajes como el diálogo en cascada al conjunto.

```javascript
/**
*
* @param {ConversationState} conversationState A ConversationState object used to store the dialog state.
* @param {UserState} userState A UserState object used to store values specific to the user.
*/
constructor(conversationState, userState) {
    // Create a new state accessor property. See https://aka.ms/about-bot-state-accessors to learn more about bot state and state accessors.
    this.conversationState = conversationState;
    this.userState = userState;

    this.dialogState = this.conversationState.createProperty(DIALOG_STATE_PROPERTY);

    this.userProfile = this.userState.createProperty(USER_PROFILE_PROPERTY);

    this.dialogs = new DialogSet(this.dialogState);

    // Add prompts that will be used by the main dialogs.
    this.dialogs.add(new TextPrompt(NAME_PROMPT));
    this.dialogs.add(new ChoicePrompt(CONFIRM_PROMPT));
    this.dialogs.add(new NumberPrompt(AGE_PROMPT, async (prompt) => {
        if (prompt.recognized.succeeded) {
            if (prompt.recognized.value <= 0) {
                await prompt.context.sendActivity(`Your age can't be less than zero.`);
                return false;
            } else {
                return true;
            }
        }

        return false;
    }));

    // Create a dialog that asks the user for their name.
    this.dialogs.add(new WaterfallDialog(WHO_ARE_YOU, [
        this.promptForName.bind(this),
        this.confirmAgePrompt.bind(this),
        this.promptForAge.bind(this),
        this.captureAge.bind(this)
    ]));

    // Create a dialog that displays a user name after it has been collected.
    this.dialogs.add(new WaterfallDialog(HELLO_USER, [
        this.displayProfile.bind(this)
    ]));
}
```

Y, por último, defina cada paso como un método independiente. También puede definir los pasos en línea mediante expresiones lambda.

```javascript
// This step in the dialog prompts the user for their name.
async promptForName(step) {
    return await step.prompt(NAME_PROMPT, `What is your name, human?`);
}

// This step captures the user's name, then prompts whether or not to collect an age.
async confirmAgePrompt(step) {
    const user = await this.userProfile.get(step.context, {});
    user.name = step.result;
    await this.userProfile.set(step.context, user);
    await step.prompt(CONFIRM_PROMPT, 'Do you want to give your age?', ['yes', 'no']);
}

// This step checks the user's response - if yes, the bot will proceed to prompt for age.
// Otherwise, the bot will skip the age step.
async promptForAge(step) {
    if (step.result && step.result.value === 'yes') {
        return await step.prompt(AGE_PROMPT, `What is your age?`,
            {
                retryPrompt: 'Sorry, please specify your age as a positive number or say cancel.'
            }
        );
    } else {
        return await step.next(-1);
    }
}

// This step captures the user's age.
async captureAge(step) {
    const user = await this.userProfile.get(step.context, {});
    if (step.result !== -1) {
        user.age = step.result;
        await this.userProfile.set(step.context, user);
        await step.context.sendActivity(`I will remember that you are ${ step.result } years old.`);
    } else {
        await step.context.sendActivity(`No age given.`);
    }
    return await step.endDialog();
}

// This step displays the captured information back to the user.
async displayProfile(step) {
    const user = await this.userProfile.get(step.context, {});
    if (user.age) {
        await step.context.sendActivity(`Your name is ${ user.name } and you are ${ user.age } years old.`);
    } else {
        await step.context.sendActivity(`Your name is ${ user.name } and you did not share your age.`);
    }
    return await step.endDialog();
}
```

El diálogo se ejecuta desde el bot en el controlador de turnos, que primero crea un contexto de diálogo y continúa o comienza el diálogo, según corresponda, y, después, guarda la conversación y el estado de usuario al final del turno.

```javascript
// Create a dialog context object.
const dc = await this.dialogs.createContext(turnContext);
```

```javascript
// If the bot has not yet responded, continue processing the current dialog.
await dc.continueDialog();
```

```javascript
// Start the sample dialog in response to any other input.
if (!turnContext.responded) {
    const user = await this.userProfile.get(dc.context, {});
    if (user.name) {
        await dc.beginDialog(HELLO_USER);
    } else {
        await dc.beginDialog(WHO_ARE_YOU);
    }
}
```

```javascript
// Save changes to the user state.
await this.userState.saveChanges(turnContext);

// End this turn by saving changes to the conversation state.
await this.conversationState.saveChanges(turnContext);
```

---

## <a name="dialog-context-and-waterfall-step-context-objects"></a>Contexto de diálogo y contexto de ambiente de los pasos de la cascada

Use el contexto de ambiente del diálogo para interactuar con un conjunto de diálogos desde el controlador de turnos del bot.
Use el contexto de ambiente del paso de cascada para interactuar con un conjunto de diálogos desde un paso de cascada.

## <a name="to-start-a-dialog"></a>Para iniciar un diálogo

Para iniciar un diálogo, pase el valor de *dialogId* que desea iniciar en el método _beginDialog_, _prompt_ o _replaceDialog_ del contexto de diálogo. El método _beginDialog_ insertará el diálogo al principio de la pila, mientras que el método _replaceDialog_ sacará el diálogo actual de la pila e insertará el diálogo sustituto en ella.

El método _prompt_ del contexto del diálogo es un método asistente que toma argumentos y construye las opciones adecuadas para el mensaje y, después, comienza el diálogo de mensajes. Para más información sobre los avisos, consulte [Petición de datos de entrada al usuario](bot-builder-prompts.md).

## <a name="to-end-a-dialog"></a>Para finalizar un diálogo

El método _end dialog_ finaliza un diálogo retirándolo de la pila y devuelve un resultado opcional al diálogo primario.

Es aconsejable llamar explícitamente al método _endDialog_ al final del diálogo.

## <a name="to-clear-the-dialog-stack"></a>Para borrar la pila de diálogos

Si desea quitar todos los diálogos de la pila, puede borrar la pila de diálogos, para lo que debe llamar al método _cancel all dialogs_ del contexto del diálogo.

## <a name="to-repeat-a-dialog"></a>Para repetir un diálogo

Para repetir un diálogo, use el método _replace dialog_, que quitará el diálogo actual de la pila, insertará el diálogo sustituto al principio de la pila y comenzará dicho diálogo. Es una excelente manera de controlar [flujos de conversación complejos](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md) y una buena técnica para administrar menús.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a administrar flujos de conversación simples, vamos a examinar cómo se puede sacar provecho del método _replace dialog_ para controlar flujos de conversación complejos.

> [!div class="nextstepaction"]
> [Administración de flujos de conversación complejos](bot-builder-dialog-manage-complex-conversation-flow.md)

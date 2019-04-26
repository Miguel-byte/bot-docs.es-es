---
title: Implementación de un flujo de conversación secuencial | Microsoft Docs
description: Aprenda a administrar un flujo de conversación simple con diálogos en Bot Framework SDK para Node.js.
keywords: flujo de conversación simple, flujo de conversación secuencial, diálogos, avisos, cascadas, conjunto de diálogos
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/18/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5361b2e411e12b296b60a0f27b560dee5f1f769f
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904868"
---
# <a name="implement-sequential-conversation-flow"></a>Implementación de un flujo de conversación secuencial

[!INCLUDE[applies-to](../includes/applies-to.md)]

Mediante la biblioteca Dialogs es posible administrar flujos de conversación simples y complejos. En una interacción simple, el bot ejecuta una secuencia fija de pasos y la conversación finaliza. En este artículo, usamos un _diálogo de cascada_, algunos _avisos_ y un _conjunto de diálogos_ para crear una interacción simple que pregunta al usuario una serie de preguntas.

## <a name="prerequisites"></a>Requisitos previos
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)
- El código de este artículo se basa en el ejemplo de varios turnos de preguntas **multi-turn-prompt**. Necesitará una copia del ejemplo en [C# ](https://aka.ms/cs-multi-prompts-sample) o en [JS](https://aka.ms/js-multi-prompts-sample).
- Conocimientos sobre los [conceptos básicos de bots](bot-builder-basics.md), la [biblioteca de diálogos](bot-builder-concept-dialog.md), el [estado del diálogo](bot-builder-dialog-state.md) y el archivo [.bot](bot-file-basics.md).


Las secciones siguientes reflejan los pasos a seguir para implementar diálogos simples para la mayoría de los bots:

## <a name="configure-your-bot"></a>Configuración del bot

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Vamos a inicializar el descriptor de acceso de propiedad de estado para el estado del diálogo del bot en el código de configuración del archivo **Startup.cs**.

Definimos una clase `MultiTurnPromptsBotAccessors` para contener los objetos de administración de estado y los descriptores de acceso de propiedad de estado del bot.
En este caso, llamamos solo a partes del código.

```csharp
public class MultiTurnPromptsBotAccessors
{
    // Initializes a new instance of the class.
    public MultiTurnPromptsBotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    public IStatePropertyAccessor<DialogState> ConversationDialogState { get; set; }
    public IStatePropertyAccessor<UserProfile> UserProfile { get; set; }

    public ConversationState ConversationState { get; }
    public UserState UserState { get; }
}
```

La clase de descriptores de acceso se registra en el método `ConfigureServices` de la clase `Startup`.
De nuevo, llamamos solo a partes del código.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<MultiTurnPromptsBotAccessors>(sp =>
    {
        // We need to grab the conversationState we added on the options in the previous step
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        var userState = options.State.OfType<UserState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new MultiTurnPromptsBotAccessors(conversationState, userState)
        {
            ConversationDialogState = conversationState.CreateProperty<DialogState>("DialogState"),
            UserProfile = userState.CreateProperty<UserProfile>("UserProfile"),
        };

        return accessors;
    });
}
```

Mediante la inserción de dependencias, los descriptores de acceso estarán disponibles para el código del constructor del bot.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el archivo **index.js** se definen los objetos de administración de estado.
En este caso, llamamos solo a partes del código.

```javascript
// Import required bot services. See https://aka.ms/bot-services to learn more about the different part of a bot.
const { BotFrameworkAdapter, ConversationState, MemoryStorage, UserState } = require('botbuilder');

// Define the state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state storage system to persist the dialog and user state between messages.
const memoryStorage = new MemoryStorage();

// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);

// Create the main dialog, which serves as the bot's main handler.
const bot = new MultiTurnBot(conversationState, userState);
```

El constructor del bot creará los descriptores de acceso de propiedad de estado del bot: `this.dialogState` y `this.userProfile`.

---

## <a name="update-the-bot-turn-handler-to-call-the-dialog"></a>Actualización del controlador de turnos del bot para llamar al diálogo

Para ejecutar el diálogo, el controlador de turnos del bot debe crear un contexto de diálogo para el conjunto de diálogos que contiene los diálogos del bot. Un bot podría definir varios conjuntos de diálogos, aunque como norma general debe definir solo uno para el bot. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

El controlador de turnos del bot ejecuta el diálogo. El controlador crea primero un elemento `DialogContext` y continúa el diálogo activo o inicia un nuevo diálogo, según corresponda. El controlador, a continuación, guarda el estado de la conversación y del usuario al final del turno.

En la clase `MultiTurnPromptsBot` hemos definido una propiedad `_dialogs` que contiene el conjunto de diálogos, desde la que se genera un contexto de diálogo. Nuevamente, solo se muestra una parte del código del controlador de turnos.

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    // ...
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Run the DialogSet - let the framework identify the current state of the dialog from
        // the dialog stack and figure out what (if any) is the active dialog.
        var dialogContext = await _dialogs.CreateContextAsync(turnContext, cancellationToken);
        var results = await dialogContext.ContinueDialogAsync(cancellationToken);

        // If the DialogTurnStatus is Empty we should start a new dialog.
        if (results.Status == DialogTurnStatus.Empty)
        {
            await dialogContext.BeginDialogAsync("details", null, cancellationToken);
        }
    }

    // ...
    // Save the dialog state into the conversation state.
    await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);

    // Save the user profile updates into the user state.
    await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El código del bot usa algunas de las clases de la biblioteca Dialogs.

```javascript
const { ChoicePrompt, DialogSet, NumberPrompt, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

El controlador de turnos del bot ejecuta el diálogo. El controlador crea primero un elemento `DialogContext` (`dc`) y continúa el diálogo activo o inicia un nuevo diálogo, según corresponda. El controlador, a continuación, guarda el estado de la conversación y del usuario al final del turno.

La clase `MultiTurnBot` se define en el archivo **bot.js**. El constructor de esta clase agrega una propiedad `dialogs` al conjunto de diálogos, desde el que se genera un contexto de diálogo. Este bot recopila los datos del usuario una vez, con el diálogo `WHO_ARE_YOU`. Una vez que se rellena el perfil de usuario, el bot usa el diálogo `HELLO_USER` para responder. Nuevamente, solo se muestra una parte del código del controlador de turnos.

```javascript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Create a dialog context object.
        const dc = await this.dialogs.createContext(turnContext);

        const utterance = (turnContext.activity.text || '').trim().toLowerCase();

        // ...
        // If the bot has not yet responded, continue processing the current dialog.
        await dc.continueDialog();

        // Start the sample dialog in response to any other input.
        if (!turnContext.responded) {
            const user = await this.userProfile.get(dc.context, {});
            if (user.name) {
                await dc.beginDialog(HELLO_USER);
            } else {
                await dc.beginDialog(WHO_ARE_YOU);
            }
        }
    }

    // ...
    // Save changes to the user state.
    await this.userState.saveChanges(turnContext);

    // End this turn by saving changes to the conversation state.
    await this.conversationState.saveChanges(turnContext);
}
```

---

En el controlador de turnos del bot creamos un contexto de diálogo para el conjunto de diálogos. El contexto de diálogo accede a la memoria caché de estado del bot, para recordar en qué parte quedó la conversación en el último turno.

Si hay un diálogo activo, el método _continue dialog_ del contexto de diálogo lo avanza mediante la entrada del usuario que desencadenó este turno; de lo contrario, el bot llama al método _begin dialog_ del contexto de diálogo para iniciar un diálogo.

Por último, se llama al método _save changes_ de los objetos de administración de estado para conservar los cambios que se han producido en este turno.

### <a name="about-dialog-and-bot-state"></a>Acerca del estado del diálogo y el bot

En este bot, definimos dos descriptores de acceso de propiedad de estado:

* Uno se crea en el estado de la conversación de la propiedad de estado del diálogo. El estado del diálogo realiza un seguimiento de dónde está el usuario dentro de los diálogos de un conjunto de diálogos y el contexto de diálogo lo actualiza, del mismo modo que cuando se llama a los métodos begin dialog o continue dialog.
* Uno creado en el estado de usuario de la propiedad de perfil de usuario. El bot lo utiliza para realizar el seguimiento de la información que tiene sobre el usuario y este estado se administra explícitamente en el código del bot.

Los métodos _get_ y _set_ del descriptor de acceso de propiedad de estado obtienen y establecen el valor de la propiedad en la memoria caché del objeto de administración de estado. La memoria caché se rellena la primera vez que se solicita el valor de una propiedad de estado en un turno, pero se debe guardar explícitamente. Para conservar los cambios en ambas propiedades de estado, llamamos al método _save changes_ del objeto de administración de estado correspondiente.

## <a name="initialize-your-bot-and-define-your-dialog"></a>Inicialización del bot y definición del diálogo

Nuestra conversación simple se modela como una serie de preguntas que se hacen al usuario. Las versiones de C# y JavaScript tienen pasos ligeramente diferentes:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. Preguntar su nombre.
1. Preguntar si quieren indicar su edad.
1. Si es así, solicitar su edad; en caso contrario, omitir este paso.
1. Preguntar si la información recopilada es correcta.
1. Enviar un mensaje de estado y finalizar.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Para el diálogo `who_are_you`:

1. Preguntar su nombre.
1. Preguntar si quieren indicar su edad.
1. Si es así, solicitar su edad; en caso contrario, omitir este paso.
1. Enviar un mensaje de estado y finalizar.

Para el diálogo `hello_user`:

1. Mostrar la información de usuario que ha reunido el bot.

---

Hay un par de cosas que recordar al definir los propios pasos en cascada.

* Cada turno del bot refleja la entrada del usuario, seguida de una respuesta del bot. Por lo tanto, se le pide entrada al usuario al final de un paso de cascada y se recibe su respuesta en el siguiente paso de cascada.
* Cada aviso es realmente un diálogo en dos pasos que presenta su aviso y se repite hasta que recibe una entrada "válida". 

En este ejemplo, el diálogo se define en el archivo del bot y se inicializa en el constructor del bot.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Definir una propiedad de instancia para el conjunto de diálogos.

```csharp
// The DialogSet that contains all the Dialogs that can be used at runtime.
private DialogSet _dialogs;
```

Cree el conjunto de diálogos dentro del constructor del bot y agregue tanto los mensajes como el diálogo en cascada al conjunto.

```csharp
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

En este ejemplo se define cada paso como un método independiente. También puede definir los pasos en línea en el constructor mediante expresiones lambda.

```csharp
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    // Running a prompt here means the next WaterfallStep will be run when the users response is received.
    return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
}

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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En este ejemplo, el diálogo en cascada se define en el archivo **bot.js**.

Se definen los identificadores que se usarán para los descriptores de acceso de propiedad de estado, los avisos y los diálogos.

```javascript
const DIALOG_STATE_PROPERTY = 'dialogState';
const USER_PROFILE_PROPERTY = 'user';

const WHO_ARE_YOU = 'who_are_you';
const HELLO_USER = 'hello_user';

const NAME_PROMPT = 'name_prompt';
const CONFIRM_PROMPT = 'confirm_prompt';
const AGE_PROMPT = 'age_prompt';
```

Se define y se crea el conjunto de diálogos en el constructor del bot y se agregan los avisos y el diálogo en cascada al conjunto.
`NumberPrompt` incluye la validación personalizada para asegurarse de que el usuario escribe una edad mayor que 0.

```javascript
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

Puesto que los métodos del paso de diálogo hacen referencia a las propiedades de instancia, es necesario usar el método `bind`, por lo que el objeto `this` se resuelve correctamente dentro de cada método del paso.

En este ejemplo se define cada paso como un método independiente. También puede definir los pasos en línea en el constructor mediante expresiones lambda.

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

---

En este ejemplo se actualiza el estado del perfil de usuario desde el diálogo. Esta práctica puede funcionar para un bot simple, pero no funcionará si desea reutilizar un diálogo en varios bots.

Hay varias opciones para mantener independientes los pasos del diálogo y el estado del bot. Por ejemplo, una vez que el diálogo recopila toda la información, puede:

* Usar el método _end dialog_ para proporcionar los datos recopilados como valor devuelto al contexto primario. Este puede ser el controlador de turnos del bot o un diálogo activo anterior en la pila de diálogos. Así se han diseñado las clases del aviso.
* Generar una solicitud a un servicio adecuado. Esto podría funcionar bien si el bot actúa como un front-end de un servicio de mayor tamaño.

## <a name="test-your-dialog"></a>Prueba del diálogo

Cree y ejecute el bot localmente y, después, interactúe con el bot mediante el emulador.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. El bot envía un mensaje de saludo inicial en respuesta a la actividad de actualización de conversación en la que el usuario se agrega a la conversación.
1. Escriba `hi` u otra entrada. Dado que aún no hay un diálogo activo en este turno, el bot inicia el diálogo `details`.
   * El bot envía el primer aviso del diálogo y espera más entradas.
1. Responda las preguntas a medida que el bot las formula mientras avanza por el diálogo.
1. El último paso del diálogo envía un mensaje `Thanks` en función de las entradas.
   * Cuando el diálogo finaliza, se elimina de la pila de diálogos y el bot ya no tiene un diálogo activo.
1. Escriba `hi` u otra entrada para iniciar el diálogo de nuevo.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. El bot envía un mensaje de saludo inicial en respuesta a la actividad de actualización de conversación en la que el usuario se agrega a la conversación.
1. Escriba `hi` u otra entrada. Dado que aún no hay un diálogo activo en este turno ni perfil de usuario, el bot inicia el diálogo `who_are_you`.
   * El bot envía el primer aviso del diálogo y espera más entradas.
1. Responda las preguntas a medida que el bot las formula mientras avanza por el diálogo.
1. El último paso del diálogo envía un breve mensaje de confirmación.
1. Escriba `hi` u otra entrada.
   * El bot inicia el diálogo de un paso `hello_user`, que muestra información de los datos recopilados y finaliza inmediatamente.

---

## <a name="additional-resources"></a>Recursos adicionales
Puede basarse en la validación integrada de cada tipo de aviso, tal como se muestra aquí, o puede agregar su propia validación personalizada al aviso. Para más información, consulte [Recopilación de datos de entrada del usuario mediante un aviso de diálogo](bot-builder-prompts.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de un flujo de conversación avanzado con bifurcaciones y bucles](bot-builder-dialog-manage-complex-conversation-flow.md)

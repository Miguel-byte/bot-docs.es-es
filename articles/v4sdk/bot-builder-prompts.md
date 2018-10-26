---
title: Petición de datos de entrada a los usuarios mediante la biblioteca Dialogs | Microsoft Docs
description: Obtenga información sobre cómo solicitar datos de entrada a los usuarios en Bot Builder SDK para Node.js.
keywords: mensajes, cuadros de diálogo, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, volver a solicitar, validación
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 9/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 16ef274bc7e8301825e574c566a49d53f01115c1
ms.sourcegitcommit: aef7d80ceb9c3ec1cfb40131709a714c42960965
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/17/2018
ms.locfileid: "49383130"
---
# <a name="prompt-users-for-input-using-the-dialogs-library"></a>Petición de datos de entrada a los usuarios con la biblioteca Dialogs

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

La recopilación de información mediante la publicación de preguntas es una de las principales formas de interacción de un bot con los usuarios. Se puede hacer directamente mediante el método [send activity](~/v4sdk/bot-builder-basics.md#defining-a-turn) del objeto _turn context_ y luego procesar el mensaje entrante siguiente como la respuesta. Sin embargo, el SDK Bot Builder proporciona una biblioteca **Dialogs** que contiene métodos diseñados para facilitar la realización de preguntas y para asegurarse de que la respuesta se ajusta a un tipo de datos concreto o cumple reglas de validación personalizadas. En este tema se detalla cómo hacerlo mediante el uso de **preguntas** para pedir al usuario la información deseada.

En este artículo se describe cómo usar las preguntas dentro de un cuadro de diálogo. Para obtener información sobre el uso de los diálogos en general, consulte [Uso de diálogos para administrar un flujo de conversación simple](bot-builder-dialog-manage-conversation-flow.md).

## <a name="prompt-types"></a>Tipos de avisos

La biblioteca Dialogs ofrece varios tipos de preguntas, y cada uno de ellos se usa para recopilar un tipo de respuesta diferente.

| Prompt | DESCRIPCIÓN |
|:----|:----|
| **AttachmentPrompt** | Solicita al usuario un archivo adjunto como un documento o una imagen. |
| **ChoicePrompt** | Solicita al usuario que elija entre un conjunto de opciones. |
| **ConfirmPrompt** | Solicita al usuario que confirme su acción. |
| **DatetimePrompt** | Solicita al usuario una fecha y una hora. Los usuarios puedan responder con lenguaje natural, como "Mañana a las 8 p.m." o "Viernes a las 10 a.m.". El SDK de Bot Framework usa la entidad precompilada `builtin.datetimeV2` de LUIS. Para obtener más información, vea [builtin.datetimev2](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2). |
| **NumberPrompt** | Solicita al usuario un número. El usuario puede responder con "10" o "diez". Si la respuesta es "diez", por ejemplo, la pregunta convertirá la respuesta en un número y devolverá `10` como resultado. |
| **TextPrompt** | Solicita al usuario una cadena de texto. |

## <a name="add-references-to-prompt-library"></a>Agregar referencias a la biblioteca de preguntas

Para tener la biblioteca **Dialogs** agregue el paquete **botbuilder-dialogs** al bot. Trataremos los diálogos en [Uso de diálogos para administrar un flujo de conversación simple](bot-builder-dialog-manage-conversation-flow.md), pero vamos a usar diálogos en nuestros avisos.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Instale el paquete **Microsoft.Bot.Builder.Dialogs** de NuGet.

Luego, incluya la referencia a la biblioteca desde el código del bot.

```cs
using Microsoft.Bot.Builder.Dialogs;
```

Tendrá que configurar el estado de los diálogos de conversación mediante descriptores de acceso. Por el momento no vamos a profundizar más en este código, pero pueden encontrar más detalles al respecto en el artículo acerca del [estado](bot-builder-howto-v4-state.md).

En las opciones del bot en **Startup.cs**, primero debe definir los objetos de estado y después agregar el elemento singleton que proporciona la clase del descriptor de acceso al constructor del bot. La clase de `BotAccessor` simplemente almacena el estado del usuario y de la conversación, junto con los descriptores de acceso de cada uno de esos elementos. La definición completa de la clase se puede encontrar en el ejemplo vinculado al final de este artículo. 

```cs
    services.AddBot<MultiTurnPromptsBot>(options =>
    {
        InitCredentialProvider(options);

        // Create and add conversation state.
        var convoState = new ConversationState(dataStore);
        options.State.Add(convoState);

        // Create and add user state.
        var userState = new UserState(dataStore);
        options.State.Add(userState);
    });

    services.AddSingleton(sp =>
    {
        // We need to grab the conversationState we added on the options in the previous step
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        if (options == null)
        {
            throw new InvalidOperationException("BotFrameworkOptions must be configured prior to setting up the State Accessors");
        }

        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        if (conversationState == null)
        {
            throw new InvalidOperationException("ConversationState must be defined and added before adding conversation-scoped state accessors.");
        }

        var userState = options.State.OfType<UserState>().FirstOrDefault();
        if (userState == null)
        {
            throw new InvalidOperationException("UserState must be defined and added before adding user-scoped state accessors.");
        }

        // The dialogs will need a state store accessor. Creating it here once (on-demand) allows the dependency injection
        // to hand it to our IBot class that is create per-request.
        var accessors = new BotAccessors(conversationState, userState)
        {
            ConversationDialogState = conversationState.CreateProperty<DialogState>("DialogState"),
            UserProfile = userState.CreateProperty<UserProfile>("UserProfile"),
        };

        return accessors;
    });
```

Después, en el código del bot, defina los siguientes objetos del conjunto de diálogos.

```cs
    private readonly BotAccessors _accessors;

    /// <summary>
    /// The <see cref="DialogSet"/> that contains all the Dialogs that can be used at runtime.
    /// </summary>
    private DialogSet _dialogs;

    /// <summary>
    /// Initializes a new instance of the <see cref="MultiTurnPromptsBot"/> class.
    /// </summary>
    /// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
    public MultiTurnPromptsBot(BotAccessors accessors)
    {
        _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

        // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
        _dialogs = new DialogSet(accessors.ConversationDialogState);

        // ...
        // other constructor items
        // ...
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Cree un bot de JavaScript mediante la plantilla Echo. Para más información, consulte la [guía de inicio rápido de JavaScript](../javascript/bot-builder-javascript-quickstart.md).

Instale el paquete dialogs desde NPM:

```cmd
npm install --save botbuilder-dialogs
```

Para usar **dialogs** en el bot, inclúyalo en el código del bot.

1. En su archivo **bot.js**, agregue lo siguiente.

    ```javascript
    // Import components from the dialogs library.
    const { DialogSet, TextPrompt, WaterfallDialog } = require("botbuilder-dialogs");

    // Name for the dialog state property accessor.
    const DIALOG_STATE_PROPERTY = 'dialogState';

    // Define the names for the prompts and dialogs for the dialog set.
    const TEXT_PROMPT = 'textPrompt';
    const MAIN_DIALOG = 'mainDialog';
    ```

    El _conjunto de diálogos_ contendrá los diálogos de este bot, y se usará la _solicitud de texto_ para pedir al usuario que intervenga. También necesitará un descriptor de acceso de propiedad de estado de diálogo que pueda usar el conjunto de diálogos para realizar un seguimiento de su estado.

1. Actualice el código del constructor del bot. En breve, se le agregarán más cosas.

    ```javascript
      constructor(conversationState) {
        // Track the conversation state object.
        this.conversationState = conversationState;

        // Create a state property accessor for the dialog set.
        this.dialogState = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    }
    ```

---

## <a name="prompt-the-user"></a>Preguntar al usuario

Para solicitar información al usuario, defina una pregunta mediante una de las clases integradas, como **TextPrompt**, agréguela al conjunto de diálogos y asígnele un identificador de diálogo.

Una vez que se agrega una pregunta, úsela en un dialogo en cascada de dos pasos (un diálogo en *cascada* es una manera de definir una secuencia de pasos). Se pueden encadenar varias preguntas para crear conversaciones con varios pasos. Para más información, consulte la sección [Uso de diálogos](bot-builder-dialog-manage-conversation-flow.md#using-dialogs-to-guide-the-user-through-steps) en [Uso de diálogos para administrar un flujo de conversación simple](bot-builder-dialog-manage-conversation-flow.md).

Por ejemplo, el siguiente diálogo pregunta el nombre al usuario y usa la respuesta para saludarle. En el primer turno, el diálogo pregunta el nombre al usuario. La respuesta del usuario se pasa como parámetro a la función del segundo paso, que procesa la información recibida y envía el saludo personalizado.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Cada pregunta que se usa en el cuadro de diálogo también tiene un nombre, que el cuadro de diálogo usa en el bot para acceder a la pregunta. En todos estos ejemplos, los identificadores de pregunta se exponen como constantes.

En el constructor del bot, agregue las definiciones de la cascada de dos pasos y la pregunta que debe usar el diálogo. Aquí se van a agregar como funciones independientes, pero si se prefiere, se pueden definir como un elemento lambda insertado.

```csharp
 public MultiTurnPromptsBot(BotAccessors accessors)
{
    _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

    // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
    _dialogs = new DialogSet(accessors.ConversationDialogState);

    // This array defines how the Waterfall will execute.
    var waterfallSteps = new WaterfallStep[]
    {
        NameStepAsync,
        SayHiAsync,
    };

    _dialogs.Add(new WaterfallDialog("details", waterfallSteps));
    _dialogs.Add(new TextPrompt("name"));
}
```

Después, defina los dos pasos de la cascada en el bot. Para el mensaje de texto va a especificar el identificador del *nombre* del elemento `TextPrompt` que ha definido anteriormente. Observe que los nombres de método coinciden con los de `WaterfallStep[]`. Los próximos ejemplos no incluirán dicho código, pero si desea incorporar más tendrá que agregar el nombre del método en el orden correcto en `WaterfallStep[]`.

```cs
    private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
        // Running a prompt here means the next WaterfallStep will be run when the users response is received.
        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> SayHiAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}");

        return await stepContext.EndDialogAsync(cancellationToken);
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. En el constructor del bot, cree el conjunto de diálogos y agréguele una solicitud de texto y un diálogo en cascada.

    ```javascript
    // Create the dialog set, and add the prompt and the waterfall dialog.
    this.dialogs = new DialogSet(this.dialogState)
        .add(new TextPrompt(TEXT_PROMPT))
        .add(new WaterfallDialog(MAIN_DIALOG, [
            async (step) => {
                // The results of this prompt will be passed to the next step.
                return await step.prompt(TEXT_PROMPT, 'What is your name?');
            },
            async (step) => {
                // The result property contains the result from the previous step.
                const userName = step.result;
                await step.context.sendActivity(`Hi ${userName}!`);
                return await step.endDialog();
            }
        ]));
    ```

1. Actualice el controlador de turnos del bot para ejecutar el diálogo.

    ```javascript
    async onTurn(turnContext) {
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Create a dialog context for the dialog set.
            const dc = await this.dialogs.createContext(turnContext);
            // Continue the dialog if it's active.
            await dc.continueDialog();
            if (!turnContext.responded) {
                // Otherwise, start the dialog.
                await dc.beginDialog(MAIN_DIALOG);
            }
        } else {
            // Send a default message for activity types that we don't handle.
            await turnContext.sendActivity(`[${turnContext.activity.type} event detected]`);
        }
        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        }
    }
    ```

---

> [!NOTE]
> Para iniciar un diálogo, obtenga un contexto de diálogo y use su método _begin dialog_. Para más información, consulte [Uso de diálogos para administrar un flujo de conversación simple](./bot-builder-dialog-manage-conversation-flow.md).

## <a name="reusable-prompts"></a>Preguntas reutilizables

Un mensaje se puede reutilizar para formular distintas preguntas, siempre y cuando las respuestas sean del mismo tipo. Por ejemplo, en el código de ejemplo anterior se definía una pregunta de texto y se usaba para preguntarle el nombre al usuario. También puede usar esa misma pregunta para pedir al usuario otra cadena de texto, como "Where do you work?" (¿Dónde trabajas?).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En el ejemplo, el identificador de nuestro mensaje de texto, *name*, no aumenta la claridad del código. Sin embargo, es un buen ejemplo que el identificador del mensaje puede lo que desee.

Ahora, nuestros métodos incluyen un tercer paso, en el que se pregunta dónde trabaja el usuario.

```cs
    private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
        // Running a prompt here means the next WaterfallStep will be run when the users response is received.
        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> WorkAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}!");

        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Where do you work?") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> SayHiAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"{stepContext.Result} is a cool place!");

        return await stepContext.EndDialogAsync(cancellationToken);
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el constructor del bot, modifique la cascada para formular una segunda pregunta.

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new TextPrompt(TEXT_PROMPT))
    .add(new WaterfallDialog(MAIN_DIALOG, [
    async (step) => {
        // Ask the user for their name.
        return await step.prompt(TEXT_PROMPT, 'What is your name?');
    },
    async (step) => {
        // Acknowledge their response and ask for their place of work.
        const userName = step.result;
        return await step.prompt(TEXT_PROMPT, `Hi ${userName}; where do you work?`);
    },
    async (step) => {
        // Acknowledge their response and exit the dialog.
        const workPlace = step.result;
        await step.context.sendActivity(`${workPlace} is a cool place!`);
        return await step.endDialog();
    }
    ]));
```

---

Si tiene que utilizar varios mensajes diferentes, asigne un *dialogId* a cada uno. Cada diálogo o mensaje que se agrega a un conjunto de diálogos necesita un identificador único. También puede crear diálogos con varios **mensajes** del mismo tipo. Por ejemplo, se podrían crear dos cuadros de diálogo **TextPrompt** para el ejemplo anterior:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
_dialogs.Add(new WaterfallDialog("details", waterfallSteps));
_dialogs.Add(new TextPrompt("name"));
_dialogs.Add(new TextPrompt("workplace"));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Por ejemplo, podría reemplazar esto

```javascript
.add(new TextPrompt(TEXT_PROMPT))
```

por lo siguiente.

```javascript
.add(new TextPrompt('namePrompt'))
.add(new TextPrompt('workPlacePrompt'))
```

Luego, actualice los pasos respectivos de la cascada para usar esos avisos por sus nombres respectivos.

---

Para facilitar la reutilización del código, la definición de un solo elemento `TextPrompt` valdría para estas preguntas, ya que todas ellas esperan una cadena de texto como respuesta. La capacidad de asignar nombres a los diálogos resulta útil cuando hay que aplicar distintas reglas de validación a la entrada de las preguntas. Veamos cómo se pueden validar las respuestas mediante `NumberPrompt`.

## <a name="specify-prompt-options"></a>Especificar las opciones de las preguntas

Cuando se usa una pregunta en un paso de cuadro de diálogo, también se pueden proporcionar opciones de pregunta, como una cadena de nueva pregunta.

Especificar una cadena de nueva pregunta es útil cuando la entrada del usuario no puede satisfacer una pregunta, ya sea porque está en un formato que no se puede analizar, como "mañana" para una pregunta de símbolo, o bien porque la entrada no cumpla un criterio de validación. La pregunta de número puede interpretar una amplia variedad de entradas, "doce" o "un cuarto", así como "12" y "0,25".

La configuración regional es un parámetro opcional en determinados avisos, como **NumberPrompt**. Puede facilitar al mensaje a analizar la entrada con mayor precisión, pero no es obligatorio.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En el código siguiente se agrega una pregunta numérica a un conjunto de diálogos existente, **_dialogs**.

```csharp
_dialogs.Add(new NumberPrompt<int>("age"));
```

En un paso de cuadro de diálogo, el código siguiente podría solicitar entradas al usuario y proporcionar una cadena de nueva pregunta para usarla en caso de que la entrada no se pudiera interpretar como un número.

```csharp
return await stepContext.PromptAsync(
    "age",
    new PromptOptions {
        Prompt = MessageFactory.Text("Please enter your age."),
        RetryPrompt = MessageFactory.Text("I didn't get that. Please enter a valid age."),
    },
    cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Importe la clase `NumberPrompt` de la biblioteca de diálogos.

```javascript
const { NumberPrompt } = require("botbuilder-dialogs");
```

Use el aviso de número del diálogo en cascada y especifique las cadenas de aviso inicial y de reintento.

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new NumberPrompt('partySize'))
    .add(new WaterfallDialog(MAIN_DIALOG, [
    async (step) => {
        // Ask the user for their party size.
        return await step.prompt('partySize', {
            prompt: 'How many people in your party?',
            retryPrompt: 'Sorry, please specify the number of people in your party.'
        });
    },
    async (step) => {
        // Acknowledge their response and exit the dialog.
        const partySize = step.result;
        await step.context.sendActivity(`That's a party of ${partySize}, thanks.`);
        return await step.endDialog();
    }
]));
```

---

El mensaje acerca de las opciones tiene un parámetro adicional necesario: la lista de opciones entre las que puede elegir el usuario.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Si se usa **ChoicePrompt** para pedir al usuario que elija entre un conjunto de opciones, es preciso proporcionar el mensaje con dicho conjunto, que se incluye en un objeto **PromptOptions**. En este caso, se usa **ChoiceFactory** para convertir una lista de opciones en un formato adecuado.

```csharp
private static async Task<DialogTurnResult> FavoriteColorAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}!");

    return await stepContext.PromptAsync(
        "color",
        new PromptOptions {
            Prompt = MessageFactory.Text("What's your favorite color?"),
            Choices = ChoiceFactory.ToChoices(new List<string> { "blue", "green", "red" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Importe la clase `NumberPrompt` de la biblioteca de diálogos.

```javascript
const { ChoicePrompt } = require("botbuilder-dialogs");
```

Use el aviso de selección en el diálogo en cascada y especifique las opciones disponibles.

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
const list = ['green', 'blue', 'red', 'yellow'];
this.dialogs = new DialogSet(this.dialogState)
    .add(new ChoicePrompt('choicePrompt'))
    .add(new WaterfallDialog(MAIN_DIALOG, [
    async (step) => {
        // Ask the user for their party size.
        return await step.prompt('choicePrompt', {
            prompt: 'Please choose a color:',
            retryPrompt: 'Sorry, please choose a color from the list.',
            choices: list
        });
    },
    async (step) => {
        // Acknowledge their response and exit the dialog.
        const choice = step.result;
        await step.context.sendActivity(`That's ${choice.value}, thanks.`);
        return await step.endDialog();
    }
]));
```

---

## <a name="validate-a-prompt-response"></a>Validar la respuesta de una pregunta

Puede validar una respuesta antes de devolver el valor al siguiente paso de la **cascada**. Por ejemplo, para validar **NumberPrompt** en un intervalo de números entre **6** y **20**, debería incluir una función de validación similar a la siguiente:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Cambie cuando el mensaje se agregue al conjunto de diálogos para incluir la función de validador

```cs
_dialogs.Add(new NumberPrompt<int>("partySize", PartySizeValidatorAsync));
```

A partir de ese momento, la validación se define como su propio método e indica true o false en función de que haya pasado la validación, o no. Si se devuelve false, volverá a preguntar al usuario.

```cs
private Task<bool> PartySizeValidatorAsync(PromptValidatorContext<int> promptContext, CancellationToken cancellationToken)
{
    var result = promptContext.Recognized.Value;

    if (result < 6 || result > 20)
    {
        return Task.FromResult(false);
    }

    return Task.FromResult(true);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Agregue un método de validación cuando cree el aviso.

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new NumberPrompt('partySizePrompt', async (promptContext) =>                                                 {
        // Check to make sure a value was recognized.
        if (promptContext.recognized.succeeded) {
            const value = promptContext.recognized.value;
            try {
                if (value < 6) {
                    throw new Error('Party size too small.');
                } else if (value > 20) {
                    throw new Error('Party size too big.')
                } else {
                    return true; // Indicate that this is a valid value.
                }
            } catch (err) {
                await promptContext.context.sendActivity(`${err.message} <br/>Please provide a valid number between 6 and 20.`);
                return false; // Indicate that this is invalid.
            }
        } else {
            return false;
        }
    }))
    .add(new WaterfallDialog(MAIN_DIALOG, [
        async (step) => {
            // Ask the user for their party size.
            return await step.prompt('partySizePrompt', {
                prompt: 'How large is your party?',
                retryPrompt: 'Sorry, please specify a size between 6 and 20.'
            });
        },
        async (step) => {
            // Acknowledge their response and exit the dialog.
            const size = step.result;
            await step.context.sendActivity(`That's a party of ${size}, thanks.`);
            return await step.endDialog();
        }
    ]));
```

---

Del mismo modo, si quiere validar una respuesta **DatetimePrompt** para una fecha y hora en el futuro, puede tener lógica de validación similar a la siguiente:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
    private Task<bool> DateTimeValidatorAsync(PromptValidatorContext<IList<DateTimeResolution>> prompt, CancellationToken cancellationToken)
    {
        if (prompt.Recognized.Succeeded)
        {
            var resolution = prompt.Recognized.Value.First();

            // Verify that the Timex received is within the desired bounds, compared to today.
            var now = DateTime.Now;
            DateTime.TryParse(resolution.Value, out var time);

            if (time < now)
            {
                return Task.FromResult(false);
            }

            return Task.FromResult(true);
        }

        return Task.FromResult(false);
    }
```

```csharp
_dialogs.Add(new DateTimePrompt("date", DateTimeValidatorAsync));
```

Encontrará más ejemplos en el [repositorio de ejemplos](https://aka.ms/bot-samples-readme).

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const { DateTimePrompt } = require("botbuilder-dialogs");
```

```JavaScript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new DateTimePrompt('dateTimePrompt', async (promptContext) => {
        try {
            if (!promptContext.recognized.succeeded) { throw new Error('Value not recognized.') }
            const values = promptContext.recognized.value;
            if (!Array.isArray(values) || values.length < 0) { throw new Error('Value missing.'); }
            if ((values[0].type !== 'datetime') && (values[0].type !== 'date')) { throw new Error('Unsupported type.'); }
            const now = new Date();
            const value = new Date(values[0].value);
            if (value.getTime() < now.getTime()) { throw new Error('Value in the past.') }

            // update the return value of the prompt to be a real date object
            promptContext.recognized.value = [value];
            return true; // indicate valid
        } catch (err) {
            await promptContext.context.sendActivity(`${err} Please specify a date or a date and time in the future, like tomorrow at 9am.`);
            return false; // indicate invalid
        }
    }))
    .add(new WaterfallDialog(MAIN_DIALOG, [
        async (step) => {
            // Ask the user for their party size.
            return await step.prompt('dateTimePrompt', 'When would you like to schedule that for?');
        },
        async (step) => {
            // Acknowledge their response and exit the dialog.
            const time = step.result;
            await step.context.sendActivity(`That's ${time}, thanks.`);
            return await step.endDialog();
        }
    ]));
```

Encontrará más ejemplos en el [repositorio de ejemplos](https://aka.ms/bot-samples-readme).

---

> [!TIP]
> Si el usuario proporciona una respuesta ambigua a las preguntas de fecha y hora, estas se pueden resolver en varias fechas diferentes. En función del uso previsto, puede que le interese comprobar todas las resoluciones proporcionadas por el resultado de la pregunta, en lugar de simplemente la primera.

Se pueden usar técnicas similares para validar las respuestas para cualquiera de los tipos de pregunta.

## <a name="save-user-data"></a>Guardar los datos del usuario

Al solicitar entradas al usuario, tiene varias opciones sobre cómo controlarlas. Por ejemplo, puede consumir y descartar la entrada, guardarla en una variable global, guardarla en un contenedor de almacenamiento volátil o en memoria, guardarla en un archivo, o bien en una base de datos externa. Para obtener más información sobre cómo guardar los datos del usuario, vea [Administración de los datos del usuario](bot-builder-howto-v4-state.md).

## <a name="additional-resources"></a>Recursos adicionales

Para ver un ejemplo completo del uso de algunos de estos avisos, consulte el bot de avisos de varios turnos para [C#](https://aka.ms/cs-multi-prompts-sample) o [JavaScript](https://aka.ms/js-multi-prompts-sample).

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe cómo solicitar entradas al usuario, vamos a mejorar la experiencia de usuario y el código de bot mediante la administración de varios flujos de conversación a través de cuadros de diálogo.

> [!div class="nextstepaction"]
> [Administración de un flujo de conversación simple con diálogos](bot-builder-dialog-manage-conversation-flow.md)

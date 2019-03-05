---
title: Recopilación de datos de entrada del usuario mediante un aviso de diálogo | Microsoft Docs
description: Obtenga información sobre cómo solicitar datos de entrada con la biblioteca de diálogos en Bot Framework SDK.
keywords: avisos, aviso, entrada de usuario, diálogos, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, volver a solicitar, validación
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/19/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 68c01b0f12790393fe0ee7ae0bd28addf2d26ae7
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/21/2019
ms.locfileid: "56591126"
---
# <a name="gather-user-input-using-a-dialog-prompt"></a>Recopilación de datos de entrada del usuario mediante un aviso de diálogo

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

La recopilación de información mediante la publicación de preguntas es una de las principales formas de interacción de un bot con los usuarios. La biblioteca de *diálogos* facilita la formulación de preguntas, así como la validación de la respuesta para asegurarse de que coincide con un tipo de datos específico o cumple con las reglas de validación personalizadas. En este tema se explica cómo crear y llamar avisos desde un diálogo de cascada.

## <a name="prerequisites"></a>Requisitos previos

- El código de este artículo se basa en el ejemplo **DialogPromptBot**. Necesitará una copia del [ejemplo de C#](https://aka.ms/dialog-prompt-cs) o del [ejemplo de JS](https://aka.ms/dialog-prompt-js).
- Es necesario tener un conocimiento básico de la [biblioteca de diálogos](bot-builder-concept-dialog.md) y cómo [administrar las conversaciones](bot-builder-dialog-manage-conversation-flow.md).
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator) para pruebas.

## <a name="using-prompts"></a>Uso de preguntas

Un diálogo puede usar una pregunta solo si el diálogo y la pregunta están en el mismo conjunto de diálogos. Puede usar la misma pregunta en varios pasos de un diálogo y en varios diálogos del mismo conjunto de diálogos. No obstante, puede asociar la validación personalizada con una pregunta en el momento de la inicialización. Para usar una validación diferente para el mismo tipo de pregunta, necesitará varios tipos de pregunta, cada una con su propio código de validación.

### <a name="define-a-state-property-accessor-for-the-dialog-state"></a>Defina un descriptor de acceso de propiedad de estado para el estado del diálogo.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

El ejemplo del aviso de diálogo utilizado en este artículo solicita al usuario información sobre la reserva. Para administrar el tamaño y la fecha del grupo, se define una clase interna para la información de la reserva en el archivo DialogPromptBot.cs.

```csharp
public class Reservation
{
    public int Size { get; set; }

    public string Location { get; set; }

    public string Date { get; set; }
}
```

Después, agregue un descriptor de acceso de propiedad de estado a los datos de la reserva.

```csharp
public class DialogPromptBotAccessors
{
    public DialogPromptBotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState
            ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string DialogStateAccessorKey { get; }
        = "DialogPromptBotAccessors.DialogState";
    public static string ReservationAccessorKey { get; }
        = "DialogPromptBotAccessors.Reservation";

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }
    public IStatePropertyAccessor<DialogPromptBot.Reservation> ReservationAccessor { get; set; }

    public ConversationState ConversationState { get; }
}
```

En Startup.cs, vamos a actualizar el método `ConfigureServices` para establecer los descriptores de acceso.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    IStorage dataStore = new MemoryStorage();
    var conversationState = new ConversationState(dataStore);

    // Create and register state accesssors.
    services.AddSingleton<DialogPromptBotAccessors>(sp =>
    {
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new DialogPromptBotAccessors(conversationState)
        {
            DialogStateAccessor =
                conversationState.CreateProperty<DialogState>(
                    DialogPromptBotAccessors.DialogStateAccessorKey),
            ReservationAccessor =
                conversationState.CreateProperty<DialogPromptBot.Reservation>(
                    DialogPromptBotAccessors.ReservationAccessorKey),
        };

        return accessors;
    });

    // ...
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

No es necesario realizar ningún cambio en el código de servicio HTTP para JavaScript; se puede dejar el archivo index.js tal y como está.

En bot.js, incluimos las instrucciones `require` necesarias para el bot de aviso de diálogo.

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, NumberPrompt, DateTimePrompt, ChoicePrompt, DialogTurnStatus }
    = require('botbuilder-dialogs');
```

Agregue identificadores para los descriptores de acceso, diálogos y avisos de las propiedades de estado.

```javascript
// Define identifiers for our state property accessors.
const DIALOG_STATE_ACCESSOR = 'dialogStateAccessor';
const RESERVATION_ACCESSOR = 'reservationAccessor';

// Define identifiers for our dialogs and prompts.
const RESERVATION_DIALOG = 'reservationDialog';
const SIZE_RANGE_PROMPT = 'rangePrompt';
const LOCATION_PROMPT = 'locationPrompt';
const RESERVATION_DATE_PROMPT = 'reservationDatePrompt';
```

---

### <a name="create-a-dialog-set-and-prompts"></a>Creación de un conjunto de diálogos y preguntas

En general, puede crear y agregar preguntas y diálogos al conjunto de diálogos cuando inicializa el bot. El conjunto de diálogos puede posteriormente resolver el identificador de la pregunta cuando el bot reciba la entrada del usuario.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En la clase `DialogPromptBot` defina identificadores para los diálogos, avisos y valores que se van a seguir dentro del diálogo.

```csharp
// Define identifiers for our dialogs and prompts.
private const string ReservationDialog = "reservationDialog";
private const string PartySizePrompt = "partySizePrompt";
private const string SizeRangePrompt = "sizeRangePrompt";
private const string LocationPrompt = "locationPrompt";
private const string ReservationDatePrompt = "reservationDatePrompt";

// Define keys for tracked values within the dialog.
private const string LocationKey = "location";
private const string PartySizeKey = "partySize";
```

En el constructor del bot, cree el conjunto de diálogos, agregue las solicitudes y agregue el diálogo de la reserva. Se incluye la validación personalizada cuando se crean los avisos y se implementan funciones de validación más adelante.

```csharp
private readonly DialogSet _dialogSet;
private readonly DialogPromptBotAccessors _accessors;

// ...

// Initializes a new instance of the <see cref="DialogPromptBot"/> class.
public DialogPromptBot(DialogPromptBotAccessors accessors, ILoggerFactory loggerFactory)
{
    // ...

    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Create the dialog set and add the prompts, including custom validation.
    _dialogSet = new DialogSet(_accessors.DialogStateAccessor);

    _dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
    _dialogSet.Add(new NumberPrompt<int>(SizeRangePrompt, RangeValidatorAsync));
    _dialogSet.Add(new ChoicePrompt(LocationPrompt));
    _dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));

    // Define the steps of the waterfall dialog and add it to the set.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptForPartySizeAsync,
        PromptForLocationAsync,
        PromptForReservationDateAsync,
        AcknowledgeReservationAsync,
    };

    _dialogSet.Add(new WaterfallDialog(ReservationDialog, steps));
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el constructor, cree propiedades del descriptor de acceso de estado.
A continuación, cree el conjunto de diálogos y agregue los avisos, incluida la validación personalizada.
A continuación, defina los pasos del diálogo de cascada y agréguelo al conjunto.

```javascript
constructor(conversationState) {
    // Creates our state accessor properties.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_ACCESSOR);
    this.reservationAccessor = conversationState.createProperty(RESERVATION_ACCESSOR);
    this.conversationState = conversationState;

    // Create the dialog set and add the prompts, including custom validation.
    this.dialogSet = new DialogSet(this.dialogStateAccessor);
    this.dialogSet.add(new NumberPrompt(SIZE_RANGE_PROMPT, this.rangeValidator));
    this.dialogSet.add(new ChoicePrompt(LOCATION_PROMPT));
    this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));

    // Define the steps of the waterfall dialog and add it to the set.
    this.dialogSet.add(new WaterfallDialog(RESERVATION_DIALOG, [
        this.promptForPartySize.bind(this),
        this.promptForLocation.bind(this),
        this.promptForReservationDate.bind(this),
        this.acknowledgeReservation.bind(this),
    ]));
}
```

---

### <a name="implement-dialog-steps"></a>Implementación de los pasos del diálogo

En el archivo principal del bot, implementamos cada uno de los pasos del diálogo de cascada. Después de agregar una pregunta, llámela desde un paso de un diálogo en cascada y obtenga el resultado de la pregunta en el siguiente paso del diálogo. Para llamar a una pregunta desde un paso de la cascada, llame al _contexto del paso de cascada_ en el método _prompt_ del objeto. El primer parámetro es el identificador de la pregunta que se va a usar y el segundo parámetro contiene las opciones de la pregunta como, por ejemplo, el texto usado para pedir al usuario los datos de entrada.

Estos métodos muestran:

- Cómo llamar a un símbolo del sistema desde un paso de la cascada, incluido cómo pasar _opciones de aviso_.
- Cómo proporcionar parámetros adicionales a un validador personalizado mediante la propiedad _validations_.
- Cómo proporcionar las opciones de un aviso con opciones mediante la propiedad _choices_.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En el archivo **DialogPromptBot.cs**, implementamos los pasos del diálogo en cascada.

Aquí mostramos los dos primeros pasos de la cascada, `PromptForPartySizeAsync` y `PromptForLocationAsync`.

```csharp
// First step of the main dialog: prompt for party size.
private async Task<DialogTurnResult> PromptForPartySizeAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        SizeRangePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("How many people is the reservation for?"),
            RetryPrompt = MessageFactory.Text("How large is your party?"),
            Validations = new Range { Min = 3, Max = 8 },
        },
        cancellationToken);
}

// Second step of the main dialog: prompt for location.
private async Task<DialogTurnResult> PromptForLocationAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    // Record the party size information in the current dialog state.
    var size = (int)stepContext.Result;
    stepContext.Values[PartySizeKey] = size;

    // Prompt for the location.
    return await stepContext.PromptAsync(
        LocationPrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("Please choose a location."),
            RetryPrompt = MessageFactory.Text("Sorry, please choose a location from the list."),
            Choices = ChoiceFactory.ToChoices(new List<string> { "Redmond", "Bellevue", "Seattle" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el archivo **bot.js**, implementamos los pasos del diálogo en cascada.

Aquí mostramos los dos primeros pasos de la cascada, `promptForPartySize` y `promptForLocation`.

```javascript
async promptForPartySize(stepContext) {
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        SIZE_RANGE_PROMPT, {
            prompt: 'How many people is the reservation for?',
            retryPrompt: 'How large is your party?',
            validations: { min: 3, max: 8 },
        });
}

async promptForLocation(stepContext) {
    // Record the party size information in the current dialog state.
    stepContext.values.size = stepContext.result;

    // Prompt for location.
    return await stepContext.prompt(LOCATION_PROMPT, {
        prompt: 'Please choose a location.',
        retryPrompt: 'Sorry, please choose a location from the list.',
        choices: ['Redmond', 'Bellevue', 'Seattle'],
    });
}
```

---

El segundo parámetro del método _prompt_ toma un objeto de _opciones de pregunta_ que tiene las siguientes propiedades.

| Propiedad | DESCRIPCIÓN |
| :--- | :--- |
| _prompt_ | La actividad inicial para enviar al usuario para pedir su entrada. |
| _retry prompt_ | La actividad para enviar al usuario si no se validó su primera entrada. |
| _choices_ | Una lista de opciones entre las que el usuario puede elegir, para su uso en una solicitud de elección. |
| _validations_ | Parámetros adicionales que se van a usar con un validador personalizado. |

Por lo general, las propiedades "prompt" y "retry prompt" son actividades, aunque hay algunas variaciones en la forma en que los distintos lenguajes de programación controlan esto.

Siempre debe especificar la actividad de solicitud inicial para enviar al usuario.

Especificar una pregunta de reintento es útil cuando la entrada del usuario no se puede validar, ya sea porque está en un formato que no se puede analizar, como "mañana" para una pregunta de número, o bien porque la entrada no cumpla un criterio de validación. En este caso, si no se ha proporcionado ninguna solicitud de reintento, la pregunta usará la actividad de solicitud inicial para volver a preguntar al usuario la entrada.

Para una solicitud de elección, siempre debe proporcionar la lista de opciones disponibles.

## <a name="custom-validation"></a>Validación personalizada

Puede validar una respuesta antes de devolver el valor al siguiente paso de la **cascada**. Una función de validador tiene un parámetro _prompt validator context_ que devuelve un valor booleano que indica si la entrada supera la validación.

El contexto del validador de la solicitud incluye las siguientes propiedades:

| Propiedad | DESCRIPCIÓN |
| :--- | :--- |
| _Contexto_ | El contexto del turno actual del bot. |
| _Recognized_ | Un _resultado del reconocedor de la solicitud_ que contiene información sobre la entrada del usuario según la procesa el reconocedor. |
| _Opciones_ | Contiene las _opciones de aviso_ que se proporcionaron en la llamada para iniciar el aviso. |

El resultado del reconocedor de la solicitud tiene las siguientes propiedades:

| Propiedad | DESCRIPCIÓN |
| :--- | :--- |
| _Correcto_ | Indica si el reconocedor ha podido analizar la entrada. |
| _Valor_ | El valor devuelto por el reconocedor. Si es necesario, el código de validación puede modificar este valor. |

### <a name="implement-validation-code"></a>Implementación del código de validación

Puede asociar la validación personalizada con una pregunta en el momento de la inicialización, en el constructor del bot.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// ...
_dialogSet.Add(new NumberPrompt<int>(SizeRangePrompt, RangeValidatorAsync));
// ...
_dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));
// ...
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// ...
this.dialogSet.add(new NumberPrompt(SIZE_RANGE_PROMPT, this.rangeValidator));
// ...
this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));
// ...
```

---

**Validador del tamaño de la entidad**

Hemos limitado el tamaño de los grupos que pueden hacer una reserva. El intervalo válido está definido por la propiedad _validations_ que se usa para llamar al aviso de tamaño del grupo.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the party size is appropriate to make a reservation.
private async Task<bool> RangeValidatorAsync(
    PromptValidatorContext<int> promptContext,
    CancellationToken cancellationToken)
{
    // Check whether the input could be recognized as an integer.
    if (!promptContext.Recognized.Succeeded)
    {
        await promptContext.Context.SendActivityAsync(
            "I'm sorry, I do not understand. Please enter the number of people in your party.",
            cancellationToken: cancellationToken);
        return false;
    }

    // Check whether the party size is appropriate.
    var size = promptContext.Recognized.Value;
    var validRange = promptContext.Options.Validations as Range;
    if (size < validRange.Min || size > validRange.Max)
    {
        await promptContext.Context.SendActivitiesAsync(
            new Activity[]
            {
                MessageFactory.Text($"Sorry, we can only take reservations for parties " +
                    $"of {validRange.Min} to {validRange.Max}."),
                promptContext.Options.RetryPrompt,
            },
            cancellationToken: cancellationToken);
        return false;
    }

    return true;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async rangeValidator(promptContext) {
    // Check whether the input could be recognized as an integer.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the number of people in your party.");
        return false;
    }
    else if (promptContext.recognized.value % 1 != 0) {
        await promptContext.context.sendActivity(
            "I'm sorry, I don't understand fractional people.");
        return false;
    }

    // Check whether the party size is appropriate.
    var size = promptContext.recognized.value;
    if (size < promptContext.options.validations.min
        || size > promptContext.options.validations.max) {
        await promptContext.context.sendActivity(
            'Sorry, we can only take reservations for parties of '
            + `${promptContext.options.validations.min} to `
            + `${promptContext.options.validations.max}.`);
        await promptContext.context.sendActivity(promptContext.options.retryPrompt);
        return false;
    }

    return true;
}
```

---

**Validación de fecha y hora**

En el validador de fechas de reserva, limitamos las reservas a una hora o más a partir de la hora actual. Vamos a conservar la primera resolución que cumpla con nuestros criterios y a borrar el resto.

El código de validación no es exhaustivo y funciona mejor para las entradas que se analizan en una fecha y hora. Muestra algunas de las opciones para validar una solicitud de fecha y hora, y la implementación dependerá de la información que esté intentando recopilar del usuario.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the reservation date is appropriate.
private async Task<bool> DateValidatorAsync(
    PromptValidatorContext<IList<DateTimeResolution>> promptContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Check whether the input could be recognized as an integer.
    if (!promptContext.Recognized.Succeeded)
    {
        await promptContext.Context.SendActivityAsync(
            "I'm sorry, I do not understand. Please enter the date or time for your reservation.",
            cancellationToken: cancellationToken);

        return false;
    }

    // Check whether any of the recognized date-times are appropriate,
    // and if so, return the first appropriate date-time.
    var earliest = DateTime.Now.AddHours(1.0);
    var value = promptContext.Recognized.Value.FirstOrDefault(v =>
        DateTime.TryParse(v.Value ?? v.Start, out var time) && DateTime.Compare(earliest, time) <= 0);

    if (value != null)
    {
        promptContext.Recognized.Value.Clear();
        promptContext.Recognized.Value.Add(value);
        return true;
    }

    await promptContext.Context.SendActivityAsync(
            "I'm sorry, we can't take reservations earlier than an hour from now.",
            cancellationToken: cancellationToken);

    return false;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async dateValidator(promptContext) {
    // Check whether the input could be recognized as an integer.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the date or time for your reservation.");
        return false;
    }

    // Check whether any of the recognized date-times are appropriate,
    // and if so, return the first appropriate date-time.
    const earliest = Date.now() + (60 * 60 * 1000);
    let value = null;
    promptContext.recognized.value.forEach(candidate => {
        // TODO: update validation to account for time vs date vs date-time vs range.
        const time = new Date(candidate.value || candidate.start);
        if (earliest < time.getTime()) {
            value = candidate;
        }
    });
    if (value) {
        promptContext.recognized.value = [value];
        return true;
    }

    await promptContext.context.sendActivity(
        "I'm sorry, we can't take reservations earlier than an hour from now.");
    return false;
}
```

---

La solicitud de fecha y hora devuelve una lista o matriz de las posibles _resoluciones de fecha y hora_ que coinciden con la entrada del usuario. Por ejemplo, las 9:00 podría significar las 9 a.m. o las 9 p.m., y "domingo" también resulta ambiguo. Además, una resolución de fecha y hora puede representar una fecha, una hora, una fecha y hora o un intervalo. La solicitud de fecha y hora usa [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text) para analizar la entrada del usuario.

## <a name="update-the-turn-handler"></a>Actualización del controlador de turno

Actualice el controlador de turnos del bot para iniciar el diálogo y aceptar un valor devuelto desde el diálogo cuando este se complete. Aquí vamos a suponer que el usuario está interactuando con un bot, que este tiene un diálogo en cascada activo y que el siguiente paso del diálogo usa una pregunta.

Cuando el usuario envía un mensaje al bot, hace lo siguiente:

1. El bot recupera la información sobre el estado.
1. El bot crea un contexto de diálogo
    - Si no hay ningún diálogo activo, se inicia el diálogo a menos que el usuario ya haya hecho una reserva.
    - Si hay un diálogo activo, el bot continúa. Si el diálogo termina, se registran los detalles de la reserva en la caché de estado.
1. El bot guarda los cambios de estado.

Cuando un paso del diálogo llama al método _prompt_ del contexto del paso:

1. Se crea una nueva instancia del aviso, se coloca en la pila del diálogo y se inicia. (El diálogo principal espera a que el aviso termine antes de continuar).
1. El aviso envía una actividad al usuario y le pide datos de entrada.

Cuando se envían datos de entrada al aviso:

1. El aviso intenta procesar los datos de entrada, según el tipo de aviso; por ejemplo, un aviso de número o un aviso de opciones.
1. Si el aviso incluye una validación personalizada, se ejecuta el código de la validación personalizada.
1. Si los datos de entrada superan todas las validaciones, el aviso termina y se devuelven los datos de entrada procesados; en caso contrario, el aviso se inicia a sí mismo de nuevo.

**Control de los resultados de las solicitudes**

Lo que haga con el resultado de la solicitud dependerá de por qué solicitó la información al usuario. Las opciones incluyen:

- Usar la información para controlar el flujo del diálogo como, por ejemplo, cuando el usuario responde a una solicitud de confirmación o de elección.
- Almacenar en caché la información de estado del diálogo como, por ejemplo, el establecimiento de un valor en la propiedad _values_ del contexto del paso de cascada y, a continuación, la devolución de la información recopilada cuando finaliza el diálogo.
- Guardar la información en el estado del bot. Esto requeriría que diseñara el diálogo para que accediera a los descriptores de acceso de la propiedad de estado del bot.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(
    ITurnContext turnContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    switch (turnContext.Activity.Type)
    {
        // On a message from the user:
        case ActivityTypes.Message:

            // Get the current reservation info from state.
            var reservation = await _accessors.ReservationAccessor.GetAsync(
                turnContext,
                () => null,
                cancellationToken);

            // Generate a dialog context for our dialog set.
            var dc = await _dialogSet.CreateContextAsync(turnContext, cancellationToken);

            if (dc.ActiveDialog is null)
            {
                // If there is no active dialog, check whether we have a reservation yet.
                if (reservation is null)
                {
                    // If not, start the dialog.
                    await dc.BeginDialogAsync(ReservationDialog, null, cancellationToken);
                }
                else
                {
                    // Otherwise, send a status message.
                    await turnContext.SendActivityAsync(
                        $"We'll see you on {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }
            else
            {
                // Continue the dialog.
                var dialogTurnResult = await dc.ContinueDialogAsync(cancellationToken);

                // If the dialog completed this turn, record the reservation info.
                if (dialogTurnResult.Status is DialogTurnStatus.Complete)
                {
                    reservation = (Reservation)dialogTurnResult.Result;
                    await _accessors.ReservationAccessor.SetAsync(
                        turnContext,
                        reservation,
                        cancellationToken);

                    // Send a confirmation message to the user.
                    await turnContext.SendActivityAsync(
                        $"Your party of {reservation.Size} is confirmed for " +
                        $"{reservation.Date} in {reservation.Location}.",
                        cancellationToken: cancellationToken);
                }
            }

            // Save the updated dialog state into the conversation state.
            await _accessors.ConversationState.SaveChangesAsync(
                turnContext, false, cancellationToken);
            break;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async onTurn(turnContext) {
    switch (turnContext.activity.type) {
        case ActivityTypes.Message:
            // Get the current reservation info from state.
            const reservation = await this.reservationAccessor.get(turnContext, null);

            // Generate a dialog context for our dialog set.
            const dc = await this.dialogSet.createContext(turnContext);

            if (!dc.activeDialog) {
                // If there is no active dialog, check whether we have a reservation yet.
                if (!reservation) {
                    // If not, start the dialog.
                    await dc.beginDialog(RESERVATION_DIALOG);
                }
                else {
                    // Otherwise, send a status message.
                    await turnContext.sendActivity(
                        `We'll see you on ${reservation.date}.`);
                }
            }
            else {
                // Continue the dialog.
                const dialogTurnResult = await dc.continueDialog();

                // If the dialog completed this turn, record the reservation info.
                if (dialogTurnResult.status === DialogTurnStatus.complete) {
                    await this.reservationAccessor.set(
                        turnContext,
                        dialogTurnResult.result);

                    // Send a confirmation message to the user.
                    await turnContext.sendActivity(
                        `Your party of ${dialogTurnResult.result.size} is ` +
                        `confirmed for ${dialogTurnResult.result.date} in ` +
                        `${dialogTurnResult.result.location}.`);
                }
            }

            // Save the updated dialog state into the conversation state.
            await this.conversationState.saveChanges(turnContext, false);
            break;
        case ActivityTypes.EndOfConversation:
        case ActivityTypes.DeleteUserData:
            break;
        default:
            break;
    }
}
```

---

Se pueden usar técnicas similares para validar las respuestas para cualquiera de los tipos de pregunta.

## <a name="test-your-bot"></a>Prueba del bot

1. Ejecute el ejemplo localmente en la máquina. Si necesita instrucciones, consulte el archivo Léame en el ejemplo de [C#](https://aka.ms/dialog-prompt-cs) o [JS](https://aka.ms/dialog-prompt-js).
2. Inicie el emulador y envíe mensajes como se muestra a continuación para probar el bot.

![ejemplo de aviso de diálogo de prueba](~/media/emulator-v4/test-dialog-prompt.png)

## <a name="additional-resources"></a>Recursos adicionales

Para llamar a un aviso directamente desde el controlador de turnos, consulte el ejemplo _prompt-validations_ en [C#](https://aka.ms/cs-prompt-validation-sample) o [JS](https://aka.ms/js-prompt-validation-sample).

La biblioteca de diálogos también incluye una _solicitud de OAuth_ para obtener un _token de OAuth_ con el que acceder a otra aplicación en nombre del usuario. Para más información acerca de la autenticación, consulte cómo [agregar autenticación](bot-builder-authentication.md) al bot.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de un flujo de conversación avanzado con bifurcaciones y bucles](bot-builder-dialog-manage-complex-conversation-flow.md)

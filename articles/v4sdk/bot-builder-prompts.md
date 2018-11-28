---
title: Recopilación de datos de entrada del usuario mediante un aviso de diálogo | Microsoft Docs
description: Obtenga información sobre cómo solicitar datos de entrada con la biblioteca de diálogos en Bot Builder SDK.
keywords: avisos, aviso, entrada de usuario, diálogos, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, volver a solicitar, validación
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1c69b438c739ac9c47d40e53f1300a4773fc1a1d
ms.sourcegitcommit: 6cb37f43947273a58b2b7624579852b72b0e13ea
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/22/2018
ms.locfileid: "52288794"
---
# <a name="gather-user-input-using-a-dialog-prompt"></a>Recopilación de datos de entrada del usuario mediante un aviso de diálogo

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

La recopilación de información mediante la publicación de preguntas es una de las principales formas de interacción de un bot con los usuarios. La biblioteca de *diálogos* facilita la formulación de preguntas, así como la validación de la respuesta para asegurarse de que coincide con un tipo de datos específico o cumple con las reglas de validación personalizadas. En este tema se explica cómo crear y llamar avisos desde un diálogo de cascada.

## <a name="prerequisites"></a>Requisitos previos
- El código de este artículo se basa en el ejemplo de avisos de diálogos. Necesitará una copia del ejemplo en [C# ](https://aka.ms/dialog-prompt-cs) o en [JS](https://aka.ms/dialog-prompt-js).
- Es necesario tener un conocimiento básico de la [biblioteca de diálogos](bot-builder-concept-dialog.md) y cómo [administrar las conversaciones](bot-builder-dialog-manage-conversation-flow.md). 
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator) para pruebas.

## <a name="about-prompt-types"></a>Acerca de los tipos de preguntas

En un segundo plano, las preguntas consisten en un diálogo de dos pasos. En el primero, la pregunta solicita información. En el segundo, devuelve el valor válido, o se reinicia desde el principio con una nueva pregunta. La biblioteca de diálogos ofrece varios tipos de preguntas básicas, y cada uno de ellos se usa para recopilar un tipo de respuesta diferente. Las preguntas básicas pueden interpretar entradas en lenguaje natural como, por ejemplo, "diez" o "una docena" para un número, o "mañana" o "el viernes a las 10 am" para una fecha y hora.

| Prompt | DESCRIPCIÓN | Devuelve |
|:----|:----|:----|
| _Solicitud de archivos adjuntos_ | Pide uno o varios archivos adjuntos como, por ejemplo, documentos o imágenes. | Una colección de objetos _adjuntos_. |
| _Solicitud de elección_ | Pide que se realice una elección entre las diversas opciones de un conjunto. | Un objeto de _opción seleccionada_. |
| _Solicitud de confirmación_ | Solicita una confirmación. | Valor booleano. |
| _Solicitud de fecha y hora_ | Pregunta una fecha y una hora. | Una colección de objetos de _resolución de fecha y hora_. |
| _Solicitud de número_ | Solicita un número. | Un valor numérico. |
| _Solicitud de texto_ | Solicita la entrada de texto general. | Una cadena. |

Para solicitar una entrada al usuario, defina una pregunta mediante una de las clases integradas, como la _pregunta de texto_, y agréguela al conjunto de diálogos. Las preguntas tienen identificadores fijos que deben ser únicos dentro de un conjunto de diálogos. Puede tener un validador personalizado para cada pregunta y, para algunas preguntas, puede especificar una _configuración regional predeterminada_. 

### <a name="prompt-locale"></a>Configuración regional de la pregunta

La configuración regional se usa para determinar el comportamiento específico del idioma de las solicitudes de **elección**, **confirmación**, **fecha y hora** y **número**. Para cualquier entrada del usuario, si el canal ha proporcionado una propiedad de _configuración regional_ en el mensaje del usuario, se usará esa. En caso contrario, si se establece la _configuración regional predeterminada_ de la pregunta, ya sea proporcionándola al llamar al constructor de la misma o estableciéndola posteriormente, esa será la que se use. Si no se proporciona ninguna de las dos, se utiliza el inglés ("en-us") como configuración regional. Nota: La configuración regional consiste en un código ISO 639 de 2, 3 o 4 caracteres que representa un idioma o familia de idiomas.

## <a name="using-prompts"></a>Uso de preguntas

Un diálogo puede usar una pregunta solo si el diálogo y la pregunta están en el mismo conjunto de diálogos. Puede usar la misma pregunta en varios pasos de un diálogo y en varios diálogos del mismo conjunto de diálogos. No obstante, puede asociar la validación personalizada con una pregunta en el momento de la inicialización. Para usar una validación diferente para el mismo tipo de pregunta, necesitará varios tipos de pregunta, cada una con su propio código de validación.

### <a name="define-a-state-property-accessor-for-the-dialog-state"></a>Defina un descriptor de acceso de propiedad de estado para el estado del diálogo.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

El ejemplo del aviso de diálogo utilizado en este artículo solicita al usuario información sobre la reserva. Para administrar el tamaño y la fecha del grupo, se define una clase interna para la información de la reserva en el archivo DialogPromptBot.cs.

```csharp
public class Reservation
{
    public int Size { get; set; }

    public string Date { get; set; }
}
```

Después, agregue un descriptor de acceso de propiedad de estado a los datos de la reserva.

```csharp
public class DialogPromptBotAccessors
{
    public DialogPromptBotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string DialogStateAccessorKey { get; } = "DialogPromptBotAccessors.DialogState";
    public static string ReservationAccessorKey { get; } = "DialogPromptBotAccessors.Reservation";

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

    // Create and register state accesssors.
    // Acessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        // ...

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState)
        {
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(DialogPromptBotAccessors.DialogStateAccessorKey),
            ReservationAccessor = conversationState.CreateProperty<DialogPromptBot.Reservation>(DialogPromptBotAccessors.ReservationAccessorKey),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

No es necesario realizar ningún cambio en el código de servicio HTTP para JavaScript; se puede dejar el archivo index.js tal y como está.

En bot.js, incluimos las instrucciones `require` necesarias para el bot de aviso de diálogo. 
```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, NumberPrompt, DateTimePrompt, ChoicePrompt, DialogTurnStatus } = require('botbuilder-dialogs');
```

Agregue identificadores para los descriptores de acceso, diálogos y avisos de las propiedades de estado.

```javascript
// Define identifiers for state property accessors.
const DIALOG_STATE_ACCESSOR = 'dialogStateAccessor';
const RESERVATION_ACCESSOR = 'reservationAccessor';

// Define identifiers for dialogs and prompts.
const RESERVATION_DIALOG = 'reservationDialog';
const PARTY_SIZE_PROMPT = 'partySizePrompt';
const LOCATION_PROMPT = 'locationPrompt';
const RESERVATION_DATE_PROMPT = 'reservationDatePrompt';
```

---

### <a name="create-a-dialog-set-and-prompts"></a>Creación de un conjunto de diálogos y preguntas

En general, puede crear y agregar preguntas y diálogos al conjunto de diálogos cuando inicializa el bot. El conjunto de diálogos puede posteriormente resolver el identificador de la pregunta cuando el bot reciba la entrada del usuario.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En la clase `DialogPromptBot` defina identificadores para diálogos, avisos y conjuntos de diálogos.
```csharp
private const string ReservationDialog = "reservationDialog";
private const string PartySizePrompt = "partyPrompt";
private const string LocationPrompt = "locationPrompt";
private const string ReservationDatePrompt = "reservationDatePrompt";

private readonly DialogSet _dialogSet;
```

En el constructor del bot, cree el conjunto de diálogos, agregue las solicitudes y agregue el diálogo de la reserva. Se incluye la validación personalizada cuando se crean los avisos y se implementan funciones de validación más adelante.

```csharp
// The following code creates prompts and adds them to an existing dialog set. The DialogSet contains all the dialogs that can 
// be used at runtime. The prompts also references a validation method is not shown here.

public DialogPromptBot(DialogPromptBotAccessors accessors, ILoggerFactory loggerFactory)
{
   // ...

    // Create the dialog set and add the prompts, including custom validation.
    _dialogSet = new DialogSet(_accessors.DialogStateAccessor);
    _dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
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

```javascript
constructor(conversationState) {
    // Creates our state accessor properties.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_ACCESSOR);
    this.reservationAccessor = conversationState.createProperty(RESERVATION_ACCESSOR);
    this.conversationState = conversationState;
    // ...
    }
```

A continuación, cree el conjunto de diálogos y agregue los avisos, incluida la validación personalizada.

```javascript
    // ...
    this.dialogSet = new DialogSet(this.dialogStateAccessor);
    this.dialogSet.add(new NumberPrompt(PARTY_SIZE_PROMPT, this.partySizeValidator));
    this.dialogSet.add(new ChoicePrompt (LOCATION_PROMPT));
    this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));
    // ...
```

A continuación, defina los pasos del diálogo de cascada y agréguelo al conjunto.
```javascript
    // ...
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

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En el archivo DialogPromptBot.cs, implementamos los pasos `PromptForPartySizeAsync`, `PromptForLocationAsync`, `PromptForReservationDateAsync` y `AcknowledgeReservationAsync` del diálogo en cascada.

Aquí solo mostramos `PromptForPartySizeAsync` y `PromptForLocationAsync` que son delegados de dos pasos consecutivos de un diálogo en cascada.

```csharp
private async Task<DialogTurnResult> PromptForPartySizeAsync(WaterfallStepContext stepContext)
{
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    // If the input is not valid, the prompt is restarted, causing it to reprompt for input
    // and this set of steps is repeated next turn. Otherwise, the prompt ends and returns a _dialog turn result_ object 
    // to the parent dialog. Control passes to the next step of your waterfall dialog, with the result of the prompt 
    // available in the waterfall step context's _result_ property.
    return await stepContext.PromptAsync(
        PartySizePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("How many people is the reservation for?"),
            RetryPrompt = MessageFactory.Text("How large is your party?"),
        },
        cancellationToken);
}

private async Task<DialogTurnResult> PromptForLocationAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Record the party size information in the current dialog state.
    int size = (int)stepContext.Result;
    stepContext.Values["size"] = size;

    return await stepContext.PromptAsync(
        "locationPrompt",
        new PromptOptions
        {
            Prompt = MessageFactory.Text("Please choose a location."),
            RetryPrompt = MessageFactory.Text("Sorry, please choose a location from the list."),
            Choices = ChoiceFactory.ToChoices(new List<string> { "Redmond", "Bellevue", "Seattle" }),
        },
        cancellationToken);
}
```
En el ejemplo anterior se muestra cómo usar una pregunta de elección que ofrece las tres propiedades. El método `PromptForLocationAsync` se usa como un paso de un diálogo en cascada y nuestro conjunto de diálogos contiene el diálogo en cascada y una solicitud de elección con el identificador `locationPrompt`.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En este caso, `PARTY_SIZE_PROMPT` y `LOCATION_PROMPT` son los identificadores de las preguntas y `promptForPartySize` y `promptForLocation` son dos funciones de pasos consecutivos de un diálogo en cascada.

```javascript
async promptForPartySize(stepContext) {
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        PARTY_SIZE_PROMPT, {
            prompt: 'How many people is the reservation for?',
            retryPrompt: 'How large is your party?'
        });
}

async promptForLocation(stepContext) {
    // Prompt for location
    return await stepContext.prompt(
        LOCATION_PROMPT, 'Select a location.', ['Redmond', 'Bellevue', 'Seattle']
    );
}
```

---

El segundo parámetro del método _prompt_ toma un objeto de _opciones de pregunta_ que tiene las siguientes propiedades.

| Propiedad | DESCRIPCIÓN |
| :--- | :--- |
| _prompt_ | La actividad inicial para enviar al usuario para pedir su entrada. |
| _retry prompt_ | La actividad para enviar al usuario si no se validó su primera entrada. |
| _choices_ | Una lista de opciones entre las que el usuario puede elegir, para su uso en una solicitud de elección. |

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

El resultado del reconocedor de la solicitud tiene las siguientes propiedades:

| Propiedad | DESCRIPCIÓN |
| :--- | :--- |
| _Correcto_ | Indica si el reconocedor ha podido analizar la entrada. |
| _Valor_ | El valor devuelto por el reconocedor. Si es necesario, el código de validación puede modificar este valor. |

### <a name="implement-validation-code"></a>Implementación del código de validación

**Validador del tamaño de la entidad**

Limitamos las reservas a entidades de 6 a 20 personas.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private async Task<bool> PartySizeValidatorAsync(
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
    int size = promptContext.Recognized.Value;
    if (size < 6 || size > 20)
    {
        await promptContext.Context.SendActivityAsync(
            "Sorry, we can only take reservations for parties of 6 to 20.",
            cancellationToken: cancellationToken);
        return false;
    }

    return true;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async partySizeValidator(promptContext) {
    // Check whether the input could be recognized as an integer.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the number of people in your party.");
        return false;
    }
    if (promptContext.recognized.value % 1 != 0) {
        await promptContext.context.sendActivity(
            "I'm sorry, I don't understand fractional people.");
        return false;
    }
    // Check whether the party size is appropriate.
    var size = promptContext.recognized.value;
    if (size < 6 || size > 20) {
        await promptContext.context.sendActivity(
            'Sorry, we can only take reservations for parties of 6 to 20.');
        return false;
    }

    return true;
}
```

---

**Validación de fecha y hora**

En el validador de fechas de reserva, limitamos las reservas a una hora o más a partir de la hora actual. Vamos a conservar la primera resolución que cumpla con nuestros criterios y a borrar el resto. El código de validación que se muestra a continuación no es exhaustivo, y funciona mejor para las entradas que se analizan en una fecha y hora. Muestra algunas de las opciones para validar una solicitud de fecha y hora, y la implementación dependerá de la información que esté intentando recopilar del usuario.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the reservation date is appropriate.
// Reservations must be made at least an hour in advance.
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
    DateTime earliest = DateTime.Now.AddHours(1.0);
    DateTimeResolution value = promptContext.Recognized.Value.FirstOrDefault(v =>
        DateTime.TryParse(v.Value ?? v.Start, out DateTime time) && DateTime.Compare(earliest,time) <= 0);
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

### <a name="update-the-turn-handler"></a>Actualización del controlador de turno

Actualice el controlador de turnos del bot para iniciar el diálogo y aceptar un valor devuelto desde el diálogo cuando este se complete. Aquí vamos a suponer que el usuario está interactuando con un bot, que este tiene un diálogo en cascada activo y que el siguiente paso del diálogo usa una pregunta.

1. Cuando el usuario envía un mensaje al bot, hace lo siguiente:
   1. El controlador de turnos del bot crea un contexto de diálogo y llama a su método _continue_.
   1. El control pasa al siguiente paso del diálogo activo que, en este caso, es el diálogo en cascada.
   1. El paso llama al método _prompt_ del contexto del paso de cascada para pedir al usuario la entrada de datos.
   1. El contexto del paso de cascada inserta la pregunta en la pila y la inicia.
   1. La pregunta envía una actividad al usuario para pedir su entrada.
1. Cuando el usuario envía su siguiente mensaje al bot, hace lo siguiente:
   1. El controlador de turnos del bot crea un contexto de diálogo y llama a su método _continue_.
   1. El control pasa al siguiente paso del diálogo activo que, en este caso, es el segundo turno de la pregunta.
   1. La pregunta valida la entrada del usuario.

      
**Control de los resultados de las solicitudes**

Lo que haga con el resultado de la solicitud dependerá de por qué solicitó la información al usuario. Las opciones incluyen:

* Usar la información para controlar el flujo del diálogo como, por ejemplo, cuando el usuario responde a una solicitud de confirmación o de elección.
* Almacenar en caché la información de estado del diálogo como, por ejemplo, el establecimiento de un valor en la propiedad _values_ del contexto del paso de cascada y, a continuación, la devolución de la información recopilada cuando finaliza el diálogo.
* Guardar la información en el estado del bot. Esto requeriría que diseñara el diálogo para que accediera a los descriptores de acceso de la propiedad de estado del bot. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    switch (turnContext.Activity.Type)
    {
        // On a message from the user:
        case ActivityTypes.Message:

            // Get the current reservation info from state.
            Reservation reservation = await _accessors.ReservationAccessor.GetAsync(
                turnContext, () => null, cancellationToken);

            // Generate a dialog context for our dialog set.
            DialogContext dc = await _dialogSet.CreateContextAsync(turnContext, cancellationToken);

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
                        $"We'll see you {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }
            else
            {
                // Continue the dialog.
                DialogTurnResult dialogTurnResult = await dc.ContinueDialogAsync(cancellationToken);

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
                        $"Your party of {reservation.Size} is confirmed for {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }

            // Save the updated dialog state into the conversation state.
            await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
            break;

        // Handle other incoming activity types as appropriate to your bot.
        default:
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
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
                        `We'll see you ${reservation.date}.`);
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
                        `confirmed for ${dialogTurnResult.result.date}.`);
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

---
title: Uso de la biblioteca de diálogos para recopilar datos de entrada del usuario | Microsoft Docs
description: Obtenga información sobre cómo solicitar datos de entrada con la biblioteca de diálogos en Bot Builder SDK.
keywords: mensajes, cuadros de diálogo, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, volver a solicitar, validación
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/02/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 150d5f0a68d897ac278026a7cf36609aca05bb80
ms.sourcegitcommit: 984705927561cc8d6a84f811ff24c8c71b71c76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/02/2018
ms.locfileid: "50965723"
---
# <a name="use-dialog-library-to-gather-user-input"></a>Uso de la biblioteca de diálogos para recopilar datos de entrada del usuario

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

La recopilación de información mediante la publicación de preguntas es una de las principales formas de interacción de un bot con los usuarios. Se puede hacer directamente mediante el método [send activity](~/v4sdk/bot-builder-basics.md#defining-a-turn) del objeto _turn context_ y luego procesar el mensaje entrante siguiente como la respuesta. Sin embargo, el SDK Bot Builder proporciona una [biblioteca de diálogos](bot-builder-concept-dialog.md) que contiene métodos diseñados para facilitar la realización de preguntas y para asegurarse de que la respuesta se ajusta a un tipo de datos concreto o cumple reglas de validación personalizadas. En este tema se detalla cómo hacerlo mediante el uso de objetos de preguntas para pedir al usuario la información deseada.

En este artículo se describe cómo crear preguntas y cómo llamarlas desde un diálogo.
Para más información sobre cómo solicitar datos de entrada al usuario sin usar diálogos, consulte cómo [preguntar al usuario datos de entrada mediante sus propios mensajes](bot-builder-primitive-prompts.md).
Para más información sobre cómo usar los diálogos en general, consulte [Uso de diálogos para administrar un flujo de conversación simple](bot-builder-dialog-manage-conversation-flow.md).

## <a name="prompt-types"></a>Tipos de avisos

En un segundo plano, las preguntas consisten en un diálogo de dos pasos. En el primero, la pregunta solicita información. En el segundo, devuelve el valor válido, o se reinicia desde el principio con una nueva pregunta.

La biblioteca de diálogos ofrece varios tipos de preguntas básicas, y cada uno de ellos se usa para recopilar un tipo de respuesta diferente.

| Prompt | DESCRIPCIÓN | Devuelve |
|:----|:----|:----|
| _Solicitud de archivos adjuntos_ | Pide uno o varios archivos adjuntos como, por ejemplo, documentos o imágenes. | Una colección de objetos _adjuntos_. |
| _Solicitud de elección_ | Pide que se realice una elección entre las diversas opciones de un conjunto. | Un objeto de _opción seleccionada_. |
| _Solicitud de confirmación_ | Solicita una confirmación. | Valor booleano. |
| _Solicitud de fecha y hora_ | Pregunta una fecha y una hora. | Una colección de objetos de _resolución de fecha y hora_. |
| _Solicitud de número_ | Solicita un número. | Un valor numérico. |
| _Solicitud de texto_ | Solicita la entrada de texto general. | Una cadena. |

La biblioteca también incluye una _solicitud de OAuth_ para obtener un _token de OAuth_ con el que acceder a otra aplicación en nombre del usuario. Para más información acerca de la autenticación, consulte cómo [agregar autenticación al bot](bot-builder-authentication.md).

Las preguntas básicas pueden interpretar entradas en lenguaje natural como, por ejemplo, "diez" o "una docena" para un número, o "mañana" o "el viernes a las 10 am" para una fecha y hora.

## <a name="using-prompts"></a>Uso de preguntas

Un diálogo puede usar una pregunta solo si el diálogo y la pregunta están en el mismo conjunto de diálogos.

1. Defina un descriptor de acceso de propiedad de estado para el estado del diálogo.
1. Cree un conjunto de diálogos.
1. Cree las preguntas y agréguelas al conjunto de diálogos.
1. Cree un diálogo que use las preguntas y agréguelo al conjunto de diálogos.
1. Dentro del diálogo, agregue llamadas a las preguntas y recupere los resultados de las mismas.

En este artículo se analiza cómo crear las preguntas y cómo llamarlas desde un diálogo en cascada.
Para más información acerca de los diálogos en general, consulte el artículo sobre la [biblioteca de diálogos](bot-builder-concept-dialog.md).
Para un análisis de un bot completo que usa diálogos y preguntas, consulte cómo [usar diálogos para administrar flujos de conversación simples](bot-builder-dialog-manage-conversation-flow.md).

Puede usar la misma pregunta en varios pasos de un diálogo y en varios diálogos del mismo conjunto de diálogos.
No obstante, puede asociar la validación personalizada con una pregunta en el momento de la inicialización.
Por tanto, si necesita una validación diferente para el mismo tipo de pregunta, necesitará varios tipos de pregunta, cada una con su propio código de validación.

### <a name="create-a-prompt"></a>Creación de una pregunta

Para solicitar una entrada al usuario, defina una pregunta mediante una de las clases integradas, como la _pregunta de texto_, y agréguela al conjunto de diálogos.

* La pregunta tiene un identificador fijo. (Los identificadores deben ser únicos dentro de un conjunto de diálogos).
* La pregunta puede tener un validador personalizado. (Consulte la [validación personalizada](#custom-validation)).
* Para algunas preguntas puede especificar una _configuración regional predeterminada_.

En general, puede crear y agregar preguntas y diálogos al conjunto de diálogos cuando inicializa el bot. El conjunto de diálogos puede posteriormente resolver el identificador de la pregunta cuando el bot reciba la entrada del usuario.

Por ejemplo, el siguiente código crea dos preguntas de texto y las agrega a un conjunto de diálogos existente. La segunda pregunta de texto hace referencia a un método de validación que no aparece aquí.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En este caso, `_dialogs` contiene un conjunto de diálogos existente y `NameValidator` es un método de validación.

```csharp
_dialogs.Add(new TextPrompt("nickNamePrompt"));
_dialogs.Add(new TextPrompt("namePrompt", NameValidator));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En este caso, `this.dialogs` contiene un conjunto de diálogos existente y `NameValidator` es una función de validación.

```javascript
this.dialogs.add(new TextPrompt('nickNamePrompt'));
this.dialogs.add(new TextPrompt('namePrompt', NameValidator));
```

---

#### <a name="locales"></a>Locales

La configuración regional se usa para determinar el comportamiento específico del idioma de las solicitudes de **elección**, **confirmación**, **fecha y hora** y **número**. Para cualquier entrada del usuario:

* Si el canal ha proporcionado una propiedad de _configuración regional_ en el mensaje del usuario, se usará esa.
* En caso contrario, si se establece la _configuración regional predeterminada_ de la pregunta, ya sea proporcionándola al llamar al constructor de la misma o estableciéndola posteriormente, esa será la que se use.
* En caso contrario, se usará el inglés ("en-us") como configuración regional.

> [!NOTE]
> La configuración regional consiste en un código ISO 639 de 2, 3 o 4 caracteres que representa un idioma o familia de idiomas.

### <a name="call-a-prompt-from-a-waterfall-dialog"></a>Llamada a una pregunta desde un diálogo en cascada

Una vez que se agrega una pregunta, llámela desde un paso de un diálogo en cascada y obtenga el resultado de la pregunta en el siguiente paso del diálogo.
Para llamar a una pregunta desde un paso de la cascada, llame al _contexto del paso de cascada_ en el método _prompt_ del objeto. El primer parámetro es el identificador de la pregunta que se va a usar y el segundo parámetro contiene las opciones de la pregunta como, por ejemplo, el texto usado para pedir al usuario los datos de entrada.

Supongamos que el usuario está interactuando con un bot, que este tiene un diálogo en cascada activo y que el siguiente paso del diálogo usa una pregunta.

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
      * Si la entrada no es válida, la pregunta se reinicia, lo cual hace que se vuelva a solicitar una entrada, y este conjunto de pasos se repite en el siguiente turno.
      * En caso contrario, la pregunta finaliza y devuelve un objeto de _resultado de turnos del diálogo_ al diálogo principal. El control pasa al siguiente paso del diálogo en cascada, con el resultado de la pregunta disponible en la propiedad _result_ del contexto del paso de cascada.

<!--
> [!NOTE]
> A waterfall step delegate takes a _waterfall step context_ parameter and returns a _dialog turn result_.
> A prompt's result is contained within the prompt's return value (a dialog turn result object) when it ends.
> The waterfall dialog provides the return value in the waterfall step context parameter when it calls the next waterfall step.
-->

Cuando se devuelve una pregunta, la propiedad _result_ del contexto del paso de cascada se establece en el valor devuelto de la pregunta.

Este ejemplo muestra los elementos de dos pasos de cascada consecutivos. El primero usa una pregunta para pedir al usuario su nombre. El segundo obtiene el valor devuelto de la pregunta.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En este caso, `name` es el identificador de una pregunta de texto y `NameStepAsync` y `GreetingStepAsync` son dos delegaciones de pasos consecutivos de un diálogo en cascada.

```csharp
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // ...

    // Prompt for the user's name.
    return await stepContext.PromptAsync(
        "name",
         new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") },
         cancellationToken);
}

private static async Task<DialogTurnResult> GreetingStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the user's name from the prompt result.
    string name = (string)stepContext.Result;
    await stepContext.Context.SendActivityAsync(
        MessageFactory.Text($"Pleased to meet you, {name}."),
         cancellationToken);

    // ...
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En este caso, `name` es el identificador de una pregunta de texto y `nameStep` y `greetingStep` son dos funciones de pasos consecutivos de un diálogo en cascada.

```javascript
async nameStep(step) {
    // ...

    return await step.prompt('name', 'Please enter your name.');
}

async greetingStep(step) {
    // Get the user's name from the prompt result.
    const name = step.result;
    await step.context.sendActivity(`Pleased to meet you, ${name}.`);

    // ...
}
```

---

### <a name="call-a-prompt-from-the-bots-turn-handler"></a>Llamada a una pregunta desde el controlador de turnos del bot

Es posible llamar a una pregunta directamente desde el controlador de turnos mediante el método _prompt_ del contexto de diálogo.
Debería llamar al método _continue dialog_ del contexto de diálogo del siguiente turno y revisar su valor devuelto, un objeto de _resultado de turnos del diálogo_. Para ver un ejemplo de cómo hacerlo, consulte el ejemplo de validaciones de preguntas ([C#](https://aka.ms/cs-prompt-validation-sample) | [JS](https://aka.ms/js-prompt-validation-sample)) o consulte cómo [solicitar al usuario una entrada de datos mediante sus propias preguntas](bot-builder-primitive-prompts.md) si desea usar un método alternativo.

## <a name="prompt-options"></a>Opciones de pregunta

El segundo parámetro del método _prompt_ toma un objeto de _opciones de pregunta_ que tiene las siguientes propiedades.

| Propiedad | DESCRIPCIÓN |
| :--- | :--- |
| _prompt_ | La actividad inicial para enviar al usuario para pedir su entrada. |
| _retry prompt_ | La actividad para enviar al usuario si no se validó su primera entrada. |
| _choices_ | Una lista de opciones entre las que el usuario puede elegir, para su uso en una solicitud de elección. |

Por lo general, las propiedades "prompt" y "retry prompt" son actividades, aunque hay algunas variaciones en la forma en que los distintos lenguajes de programación controlan esto.

Siempre debe especificar la actividad de solicitud inicial para enviar al usuario.

Especificar una solicitud de reintento es útil cuando la entrada del usuario no se puede validar, ya sea porque está en un formato que no se puede analizar, como "mañana" para una pregunta de número, o bien porque la entrada no cumpla un criterio de validación. En este caso, si no se ha proporcionado ninguna solicitud de reintento, la pregunta usará la actividad de solicitud inicial para volver a preguntar al usuario la entrada.

Para una solicitud de elección, siempre debe proporcionar la lista de opciones disponibles.

En este ejemplo se muestra cómo usar una solicitud de elección que ofrece las tres propiedades. El método de _color favorito_ se usa como un paso de un diálogo en cascada y nuestro conjunto de diálogos contiene el diálogo en cascada y una solicitud de elección con el identificador `colorChoice`.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private static async Task<DialogTurnResult> FavoriteColorAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // ...

    return await stepContext.PromptAsync(
        "colorChoice",
        new PromptOptions {
            Prompt = MessageFactory.Text("Please choose a color."),
            RetryPrompt = MessageFactory.Text("Sorry, please choose a color from the list."),
            Choices = ChoiceFactory.ToChoices(new List<string> { "blue", "green", "red" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el SDK de JavaScript, puede proporcionar una cadena para las propiedades `prompt` y `retryPrompt`. La solicitud convierte estas a actividades de mensaje para usted.

```javascript
async favoriteColor(step) {
    // ...

    return await step.prompt('colorChoice', {
        prompt: 'Please choose a color:',
        retryPrompt: 'Sorry, please choose a color from the list.',
        choices: [ 'red', 'green', 'blue' ]
    });
}
```

---

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

### <a name="setup"></a>Configuración

Es necesario realizar algo de configuración antes de agregar el código de validación.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En el archivo **.cs** del bot, defina una clase interna para obtener información de la reserva.

```csharp
public class Reservation
{
    public int Size { get; set; }

    public string Date { get; set; }
}
```

En **BotAccessors.cs**, agregue un descriptor de acceso de propiedad de estado para los datos de la reserva.

```csharp
public class BotAccessors
{
    public BotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string DialogStateAccessorKey { get; } = "BotAccessors.DialogState";
    public static string ReservationAccessorKey { get; } = "BotAccessors.Reservation";

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }
    public IStatePropertyAccessor<ReservationBot.Reservation> ReservationAccessor { get; set; }

    public ConversationState ConversationState { get; }
}
```

En **Startup.cs**, actualice `ConfigureServices` para establecer los descriptores de acceso.

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
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorKey),
            ReservationAccessor = conversationState.CreateProperty<ReservationBot.Reservation>(BotAccessors.ReservationAccessorKey),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

No es necesario realizar ningún cambio en el código de servicio HTTP para JavaScript; se puede dejar el archivo **index.js** tal y como está.

En **bot.js**, actualice las instrucciones require y agregue los identificadores para los descriptores de acceso de propiedad de estado.

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, NumberPrompt, DateTimePrompt, DialogTurnStatus } = require('botbuilder-dialogs');

// Define identifiers for our state property accessors.
const DIALOG_STATE_ACCESSOR = 'dialogStateAccessor';
const RESERVATION_ACCESSOR = 'reservationAccessor';
```

---

En el archivo del bot, agregue los identificadores para los diálogos y solicitudes.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Define identifiers for our dialogs and prompts.
private const string ReservationDialog = "reservationDialog";
private const string PartySizePrompt = "partyPrompt";
private const string ReservationDatePrompt = "reservationDatePrompt";
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Define identifiers for our dialogs and prompts.
const RESERVATION_DIALOG = 'reservationDialog';
const PARTY_SIZE_PROMPT = 'partySizePrompt';
const RESERVATION_DATE_PROMPT = 'reservationDatePrompt';
```

---

### <a name="define-the-prompts-and-dialogs"></a>Definición de solicitudes y diálogos

En el código del constructor del bot, cree el conjunto de diálogos, agregue las solicitudes y agregue el diálogo de la reserva.
Se incluye la validación personalizada cuando se crean las solicitudes. A continuación implementaremos las funciones de validación.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public ReservationBot(BotAccessors accessors, ILoggerFactory loggerFactory)
{
    // ...
    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Create the dialog set and add the prompts, including custom validation.
    _dialogSet = new DialogSet(_accessors.DialogStateAccessor);
    _dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
    _dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));

    // Define the steps of the waterfall dialog and add it to the set.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptForPartySizeAsync,
        PromptForReservationDateAsync,
        AcknowledgeReservationAsync,
    };
    _dialogSet.Add(new WaterfallDialog(ReservationDialog, steps));
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
constructor(conversationState) {
    // Creates our state accessor properties.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_ACCESSOR);
    this.reservationAccessor = conversationState.createProperty(RESERVATION_ACCESSOR);
    this.conversationState = conversationState;

    // Create the dialog set and add the prompts, including custom validation.
    this.dialogSet = new DialogSet(this.dialogStateAccessor);
    this.dialogSet.add(new NumberPrompt(PARTY_SIZE_PROMPT, partySizeValidator));
    this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, dateValidator));

    // Define the steps of the waterfall dialog and add it to the set.
    this.dialogSet.add(new WaterfallDialog(RESERVATION_DIALOG, [
        this.promptForPartySize.bind(this),
        this.promptForReservationDate.bind(this),
        this.acknowledgeReservation.bind(this),
    ]));
}
```

---

### <a name="implement-validation-code"></a>Implementación del código de validación

Implemente el validador del número de comensales. Limitaremos la reserva a grupos de 6 a 20 personas.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
/// <summary>Validates whether the party size is appropriate to make a reservation.</summary>
/// <param name="promptContext">The validation context.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>Reservations can be made for groups of 6 to 20 people.
/// If the task is successful, the result indicates whether the input was valid.</remarks>
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

La solicitud de fecha y hora devuelve una lista o matriz de las posibles _resoluciones de fecha y hora_ que coinciden con la entrada del usuario. Por ejemplo, las 9:00 podría significar las 9 a.m. o las 9 p.m., y "domingo" también resulta ambiguo. Además, una resolución de fecha y hora puede representar una fecha, una hora, una fecha y hora o un intervalo. La solicitud de fecha y hora usa [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text) para analizar la entrada del usuario.

Implemente el validador de la fecha de reserva. Vamos a limitar las reservas a una hora o más a partir de la hora actual. Vamos a conservar la primera resolución que cumpla con nuestros criterios y a borrar el resto.

Este código de validación no es exhaustivo. Funciona mejor para una entrada que se analiza en una fecha y hora. Muestra algunas de las opciones para validar una solicitud de fecha y hora, y la implementación dependerá de la información que esté intentando recopilar del usuario.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
/// <summary>Validates whether the reservation date is appropriate.</summary>
/// <param name="promptContext">The validation context.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>Reservations must be made at least an hour in advance.
/// If the task is successful, the result indicates whether the input was valid.</remarks>
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

### <a name="implement-the-dialog-steps"></a>Implementación de los pasos del diálogo

Use las solicitudes que agregamos al conjunto de diálogos. Agregamos la validación a las solicitudes cuando las creamos en el constructor del bot. La primera vez que la solicitud pide al usuario una entrada, envía la actividad _prompt_ desde las opciones que se proporcionan. Si se produce un error en la validación, se envía la actividad _retry prompt_ para solicitar al usuario una entrada diferente.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
/// <summary>First step of the main dialog: prompt for party size.</summary>
/// <param name="stepContext">The context for the waterfall step.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>If the task is successful, the result contains information from this step.</remarks>
private async Task<DialogTurnResult> PromptForPartySizeAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        PartySizePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("How many people is the reservation for?"),
            RetryPrompt = MessageFactory.Text("How large is your party?"),
        },
        cancellationToken);
}

/// <summary>Second step of the main dialog: record the party size and prompt for the
/// reservation date.</summary>
/// <param name="stepContext">The context for the waterfall step.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>If the task is successful, the result contains information from this step.</remarks>
private async Task<DialogTurnResult> PromptForReservationDateAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Record the party size information in the current dialog state.
    int size = (int)stepContext.Result;
    stepContext.Values["size"] = size;

    // Prompt for the reservation date. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        ReservationDatePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("Great. When will the reservation be for?"),
            RetryPrompt = MessageFactory.Text("What time should we make your reservation for?"),
        },
        cancellationToken);
}

/// <summary>Third step of the main dialog: return the collected party size and reservation date.</summary>
/// <param name="stepContext">The context for the waterfall step.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>If the task is successful, the result contains information from this step.</remarks>
private async Task<DialogTurnResult> AcknowledgeReservationAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Retrieve the reservation date.
    DateTimeResolution resolution = (stepContext.Result as IList<DateTimeResolution>).First();
    string time = resolution.Value ?? resolution.Start;

    // Send an acknowledgement to the user.
    await stepContext.Context.SendActivityAsync(
        "Thank you. We will confirm your reservation shortly.",
        cancellationToken: cancellationToken);

    // Return the collected information to the parent context.
    Reservation reservation = new Reservation
    {
        Date = time,
        Size = (int)stepContext.Values["size"],
    };
    return await stepContext.EndDialogAsync(reservation, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async promptForPartySize(stepContext) {
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        PARTY_SIZE_PROMPT, {
            prompt: 'How many people is the reservation for?',
            retryPrompt: 'How large is your party?'
        });
}

async promptForReservationDate(stepContext) {
    // Record the party size information in the current dialog state.
    stepContext.values.size = stepContext.result;

    // Prompt for the reservation date. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        RESERVATION_DATE_PROMPT, {
            prompt: 'Great. When will the reservation be for?',
            retryPrompt: 'What time should we make your reservation for?'
        });
}

async acknowledgeReservation(stepContext) {
    // Retrieve the reservation date.
    const resolution = stepContext.result[0];
    const time = resolution.value || resolution.start;

    // Send an acknowledgement to the user.
    await stepContext.context.sendActivity(
        'Thank you. We will confirm your reservation shortly.');

    // Return the collected information to the parent context.
    return await stepContext.endDialog({ date: time, size: stepContext.values.size });
}
```

---

### <a name="update-the-turn-handler"></a>Actualización del controlador de turno

Actualice el controlador de turnos del bot para iniciar el diálogo y aceptar un valor devuelto desde el diálogo cuando este se complete.

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
        default:
            break;
    }
}
```

---

Encontrará más ejemplos en el [repositorio de ejemplos](https://aka.ms/bot-samples-readme).

Se pueden usar técnicas similares para validar las respuestas para cualquiera de los tipos de pregunta.

## <a name="handling-prompt-results"></a>Control de los resultados de las solicitudes

Lo que haga con el resultado de la solicitud dependerá de por qué solicitó la información al usuario. Las opciones incluyen:

* Usar la información para controlar el flujo del diálogo como, por ejemplo, cuando el usuario responde a una solicitud de confirmación o de elección.
* Almacenar en caché la información de estado del diálogo como, por ejemplo, el establecimiento de un valor en la propiedad _values_ del contexto del paso de cascada y, a continuación, la devolución de la información recopilada cuando finaliza el diálogo.
* Guardar la información en el estado del bot. Esto requeriría que diseñara el diálogo para que accediera a los descriptores de acceso de la propiedad de estado del bot.

Consulte los recursos adicionales para más información sobre temas y ejemplos que incluyan estos escenarios.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración de un flujo de conversación simple](bot-builder-dialog-manage-conversation-flow.md)
* [Administración de flujos de conversación complejos](bot-builder-dialog-manage-complex-conversation-flow.md)
* [Creación de un conjunto integrado de diálogos](bot-builder-compositcontrol.md)
* [Conservación de los datos de los cuadros de diálogo](bot-builder-tutorial-persist-user-inputs.md)
* Ejemplo de **solicitud de varios turnos** ([C#](https://aka.ms/cs-multi-prompts-sample) | [JS](https://aka.ms/js-multi-prompts-sample))

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe cómo solicitar entradas al usuario, vamos a mejorar la experiencia de usuario y el código de bot mediante la administración de varios flujos de conversación a través de cuadros de diálogo.

> [!div class="nextstepaction"]
> [Administración de flujos de conversación complejos](bot-builder-dialog-manage-complex-conversation-flow.md)

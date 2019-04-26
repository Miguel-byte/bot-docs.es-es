---
title: Creación de mensajes propios para recopilar datos de entrada del usuario | Microsoft Docs
description: Aprenda a administrar un flujo de conversación con avisos primitivos en Bot Framework SDK.
keywords: flujo de la conversación, avisos, estado de conversación, estado de usuario, avisos personalizados
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/19/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 69e39ca5263f021fe8bf197f5b2941e6bf09e556
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904988"
---
# <a name="create-your-own-prompts-to-gather-user-input"></a>Creación de mensajes propios para recopilar datos de entrada del usuario

[!INCLUDE[applies-to](../includes/applies-to.md)]

A menudo, una conversación entre un usuario y un bot implica mostrar preguntas (avisos) al usuario para obtener información, analizar la respuesta del usuario y, a continuación, actuar en dicha información.

El bot debe hacer un seguimiento del contexto de una conversación, para que pueda controlar su comportamiento y recordar las respuestas a las preguntas anteriores. El *estado* de un bot es la información que sigue con el fin de responder de forma adecuada a los mensajes entrantes.

## <a name="prerequisites"></a>Requisitos previos

- El código de este artículo se basa en el ejemplo para **solicitar una entrada a los usuarios**. Necesitará una copia del ejemplo en [C# ](https://aka.ms/cs-primitive-prompt-sample) o en [JS](https://aka.ms/js-primitive-prompt-sample).
- Conocimientos sobre la [administración del estado](bot-builder-concept-state.md) y cómo [guardar los datos de usuario y conversación](bot-builder-howto-v4-state.md).
- [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started), para probar el bot localmente.

## <a name="about-the-sample-code"></a>Acerca del código de ejemplo

En este artículo, le formulamos al usuario una serie de preguntas, validamos algunas de sus respuestas y guardamos sus comentarios.
Usamos el controlador de turnos del bot y las propiedades de estado de usuario y de conversación para administrar el flujo de la conversación y la colección de entradas.

1. Definición y configuración de estados
1. Uso de las propiedades de estado para dirigir la conversación
   1. Actualice el controlador de turnos del bot.
   1. Implemente un método auxiliar para administrar la colección de datos de usuario.
   1. Implemente los métodos de validación para la entrada del usuario.

## <a name="define-and-configure-state"></a>Definición y configuración de estados

Necesitamos configurar el bot para que realice el seguimiento de la siguiente información:

- El nombre del usuario, la edad y la fecha elegida, que se definirá en el estado de usuario.
- Lo que acabamos de preguntar al usuario, que se definirá en el estado de conversación.

Como no tenemos previsto implementar este bot, vamos a configurar tanto el estado de usuario como el de conversación para que utilicen el _almacenamiento en memoria_. A continuación, se describen algunos aspectos claves del código de configuración.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Se definen las siguientes variables.

- Una clase `UserProfile` para la información de usuario que recopilará el bot.
- Una clase `ConversationFlow` para rastrear la información sobre dónde estamos en la conversación.
- Una enumeración `ConversationFlow.Question` interna para el seguimiento de dónde nos encontramos en la conversación.
- Una clase `CustomPromptBotAccessors` en la que se agrupa la información de la administración de estado.

La clase de descriptores de acceso del bot contiene nuestros objetos de administración de estado y del descriptor de acceso de la propiedad de estado, y se pasa al bot mediante la inyección de dependencias en ASP.NET Core. En el bot, se registra la información del descriptor de acceso de propiedad de estado que aparece cuando el bot crea cada turno.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Se crean los objetos de administración de estado y se pasan cuando se crea el bot.
En el bot, se definen los identificadores para las propiedades de estado y para realizar el seguimiento de dónde se está en la conversación, y después se registran los objetos de administración de estado y se crean los descriptores de acceso de propiedad de estado en el constructor del bot.

---

## <a name="use-state-properties-to-direct-the-conversation"></a>Uso de las propiedades de estado para dirigir la conversación

Cuando se tienen configuradas las propiedades de estado, se pueden usar en el bot.

- Defina el [controlador de turnos](#the-bots-turn-handler) para acceder al estado y llamar al método auxiliar.
- Implemente un [método auxiliar](#filling-out-the-user-profile) para administrar la colección del perfil de usuario.
- Implemente los [métodos de validación](#parse-and-validate-input) para analizar y validar la entrada del usuario.

### <a name="the-bots-turn-handler"></a>Controlador de turnos del bot

Se utilizan los descriptores de acceso de propiedad de estado para obtener las propiedades de estado del contexto de turnos.
Si es necesario rellenar el perfil de usuario, se llama al método auxiliar y, después, guarde los cambios de estado.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En **CustomPromptBot.cs**, obtenga la propiedad de estado y llame al método auxiliar. (Tenga en cuenta que la propiedad de instancia `_accessors` se establece en el constructor del bot).

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
   if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the state properties from the turn context.
        ConversationFlow flow = await _accessors.ConversationFlowAccessor.GetAsync(turnContext, () => new ConversationFlow());
        UserProfile profile = await _accessors.UserProfileAccessor.GetAsync(turnContext, () => new UserProfile());

        await FillOutUserProfileAsync(flow, profile, turnContext);

        // Update state and save changes.
        await _accessors.ConversationFlowAccessor.SetAsync(turnContext, flow);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        await _accessors.UserProfileAccessor.SetAsync(turnContext, profile);
        await _accessors.UserState.SaveChangesAsync(turnContext);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En **bot.js**, obtenga la propiedad de estado y llame al método auxiliar. (Tenga en cuenta que la propiedad de instancia `conversationFlow` se establece en el constructor del bot).

```javascript
// The bot's turn handler.
async onTurn(turnContext) {
    // This bot listens for message activities.
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Get the state properties from the turn context.
        const flow = await this.conversationFlow.get(turnContext, { lastQuestionAsked: question.none });
        const profile = await this.userProfile.get(turnContext, {});

        await MyBot.fillOutUserProfile(flow, profile, turnContext);

        // Update state and save changes.
        await this.conversationFlow.set(turnContext, flow);
        await this.conversationState.saveChanges(turnContext);

        await this.userProfile.set(turnContext, profile);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

### <a name="filling-out-the-user-profile"></a>Rellenado del perfil de usuario

Comenzaremos por recopilar la información. Cada uno proporcionará una interfaz similar.

- El valor devuelto indica si la entrada es una respuesta válida para esta pregunta.
- Si la validación es correcta, se genera un valor normalizado y analizado para guardarlo.
- Si se produce un error de validación, se genera un mensaje con el que el bot puede volver a pedir la información.

 En la [sección siguiente](#parse-and-validate-input), se van a definir los métodos auxiliares para analizar y validar la entrada del usuario.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private static async Task FillOutUserProfileAsync(ConversationFlow flow, UserProfile profile, ITurnContext turnContext)
{
    string input = turnContext.Activity.Text?.Trim();
    string message;
    switch (flow.LastQuestionAsked)
    {
        case ConversationFlow.Question.None:
            await turnContext.SendActivityAsync("Let's get started. What is your name?");
            flow.LastQuestionAsked = ConversationFlow.Question.Name;
            break;
        case ConversationFlow.Question.Name:
            if (ValidateName(input, out string name, out message))
            {
                profile.Name = name;
                await turnContext.SendActivityAsync($"Hi {profile.Name}.");
                await turnContext.SendActivityAsync("How old are you?");
                flow.LastQuestionAsked = ConversationFlow.Question.Age;
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
        case ConversationFlow.Question.Age:
            if (ValidateAge(input, out int age, out message))
            {
                profile.Age = age;
                await turnContext.SendActivityAsync($"I have your age as {profile.Age}.");
                await turnContext.SendActivityAsync("When is your flight?");
                flow.LastQuestionAsked = ConversationFlow.Question.Date;
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
        case ConversationFlow.Question.Date:
            if (ValidateDate(input, out string date, out message))
            {
                profile.Date = date;
                await turnContext.SendActivityAsync($"Your cab ride to the airport is scheduled for {profile.Date}.");
                await turnContext.SendActivityAsync($"Thanks for completing the booking {profile.Name}.");
                await turnContext.SendActivityAsync($"Type anything to run the bot again.");
                flow.LastQuestionAsked = ConversationFlow.Question.None;
                profile = new UserProfile();
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
    }
}


```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Manages the conversation flow for filling out the user's profile.
static async fillOutUserProfile(flow, profile, turnContext) {
    const input = turnContext.activity.text;
    let result;
    switch (flow.lastQuestionAsked) {
        // If we're just starting off, we haven't asked the user for any information yet.
        // Ask the user for their name and update the conversation flag.
        case question.none:
            await turnContext.sendActivity("Let's get started. What is your name?");
            flow.lastQuestionAsked = question.name;
            break;

        // If we last asked for their name, record their response, confirm that we got it.
        // Ask them for their age and update the conversation flag.
        case question.name:
            result = this.validateName(input);
            if (result.success) {
                profile.name = result.name;
                await turnContext.sendActivity(`I have your name as ${profile.name}.`);
                await turnContext.sendActivity('How old are you?');
                flow.lastQuestionAsked = question.age;
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }

        // If we last asked for their age, record their response, confirm that we got it.
        // Ask them for their date preference and update the conversation flag.
        case question.age:
            result = this.validateAge(input);
            if (result.success) {
                profile.age = result.age;
                await turnContext.sendActivity(`I have your age as ${profile.age}.`);
                await turnContext.sendActivity('When is your flight?');
                flow.lastQuestionAsked = question.date;
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }

        // If we last asked for a date, record their response, confirm that we got it,
        // let them know the process is complete, and update the conversation flag.
        case question.date:
            result = this.validateDate(input);
            if (result.success) {
                profile.date = result.date;
                await turnContext.sendActivity(`Your cab ride to the airport is scheduled for ${profile.date}.`);
                await turnContext.sendActivity(`Thanks for completing the booking ${profile.name}.`);
                await turnContext.sendActivity('Type anything to run the bot again.');
                flow.lastQuestionAsked = question.none;
                profile = {};
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }
    }
}
```

---

### <a name="parse-and-validate-input"></a>Análisis y validación de la entrada

Vamos a usar los siguientes criterios para validar la entrada.

- El **nombre** debe ser una cadena que no esté vacía. Lo normalizaremos al recortar los espacios en blanco.
- La **edad** debe estar entre 18 y 120. Lo normalizaremos al devolver un entero.
- La **fecha** debe ser cualquier fecha u hora al menos una hora en el futuro.
  La normalizaremos al devolver solo la parte de fecha de la entrada analizada.

> [!NOTE]
> Para la entrada de edad y fecha, utilizamos las bibliotecas [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text/) para realizar el análisis inicial.
> Aunque proporcionamos código de ejemplo, no explicamos cómo funcionan las bibliotecas de los reconocedores de texto, y esta es solo una forma de analizar la entrada.
> Para más información acerca de estas bibliotecas, consulte el archivo **Léame** del repositorio.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Agregue los siguientes métodos de validación al bot.

```csharp
private static bool ValidateName(string input, out string name, out string message)
{
    name = null;
    message = null;

    if (string.IsNullOrWhiteSpace(input))
    {
        message = "Please enter a name that contains at least one character.";
    }
    else
    {
        name = input.Trim();
    }

    return message is null;
}

private static bool ValidateAge(string input, out int age, out string message)
{
    age = 0;
    message = null;

    // Try to recognize the input as a number. This works for responses such as "twelve" as well as "12".
    try
    {
        // Attempt to convert the Recognizer result to an integer. This works for "a dozen", "twelve", "12", and so on.
        // The recognizer returns a list of potential recognition results, if any.
        List<ModelResult> results = NumberRecognizer.RecognizeNumber(input, Culture.English);
        foreach (var result in results)
        {
            // result.Resolution is a dictionary, where the "value" entry contains the processed string.
            if (result.Resolution.TryGetValue("value", out object value))
            {
                age = Convert.ToInt32(value);
                if (age >= 18 && age <= 120)
                {
                    return true;
                }
            }
        }

        message = "Please enter an age between 18 and 120.";
    }
    catch
    {
        message = "I'm sorry, I could not interpret that as an age. Please enter an age between 18 and 120.";
    }

    return message is null;
}

private static bool ValidateDate(string input, out string date, out string message)
{
    date = null;
    message = null;

    // Try to recognize the input as a date-time. This works for responses such as "11/14/2018", "9pm", "tomorrow", "Sunday at 5pm", and so on.
    // The recognizer returns a list of potential recognition results, if any.
    try
    {
        List<ModelResult> results = DateTimeRecognizer.RecognizeDateTime(input, Culture.English);

        // Check whether any of the recognized date-times are appropriate,
        // and if so, return the first appropriate date-time. We're checking for a value at least an hour in the future.
        DateTime earliest = DateTime.Now.AddHours(1.0);
        foreach (ModelResult result in results)
        {
            // result.Resolution is a dictionary, where the "values" entry contains the processed input.
            var resolutions = result.Resolution["values"] as List<Dictionary<string, string>>;
            foreach (var resolution in resolutions)
            {
                // The processed input contains a "value" entry if it is a date-time value, or "start" and
                // "end" entries if it is a date-time range.
                if (resolution.TryGetValue("value", out string dateString)
                    || resolution.TryGetValue("start", out dateString))
                {
                    if (DateTime.TryParse(dateString, out var candidate)
                        && earliest < candidate)
                    {
                        date = candidate.ToShortDateString();
                        return true;
                    }
                }
            }
        }

        message = "I'm sorry, please enter a date at least an hour out.";
    }
    catch
    {
        message = "I'm sorry, I could not interpret that as an appropriate date. Please enter a date at least an hour out.";
    }

    return false;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Agregue los siguientes métodos de validación al bot.

```javascript
// Validates name input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateName(input) {
    const name = input && input.trim();
    return name != undefined
        ? { success: true, name: name }
        : { success: false, message: 'Please enter a name that contains at least one character.' };
};

// Validates age input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateAge(input) {

    // Try to recognize the input as a number. This works for responses such as "twelve" as well as "12".
    try {
        // Attempt to convert the Recognizer result to an integer. This works for "a dozen", "twelve", "12", and so on.
        // The recognizer returns a list of potential recognition results, if any.
        const results = Recognizers.recognizeNumber(input, Recognizers.Culture.English);
        let output;
        results.forEach(function (result) {
            // result.resolution is a dictionary, where the "value" entry contains the processed string.
            const value = result.resolution['value'];
            if (value) {
                const age = parseInt(value);
                if (!isNaN(age) && age >= 18 && age <= 120) {
                    output = { success: true, age: age };
                    return;
                }
            }
        });
        return output || { success: false, message: 'Please enter an age between 18 and 120.' };
    } catch (error) {
        return {
            success: false,
            message: "I'm sorry, I could not interpret that as an age. Please enter an age between 18 and 120."
        };
    }
}

// Validates date input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateDate(input) {
    // Try to recognize the input as a date-time. This works for responses such as "11/14/2018", "today at 9pm", "tomorrow", "Sunday at 5pm", and so on.
    // The recognizer returns a list of potential recognition results, if any.
    try {
        const results = Recognizers.recognizeDateTime(input, Recognizers.Culture.English);
        const now = new Date();
        const earliest = now.getTime() + (60 * 60 * 1000);
        let output;
        results.forEach(function (result) {
            // result.resolution is a dictionary, where the "values" entry contains the processed input.
            result.resolution['values'].forEach(function (resolution) {
                // The processed input contains a "value" entry if it is a date-time value, or "start" and
                // "end" entries if it is a date-time range.
                const datevalue = resolution['value'] || resolution['start'];
                // If only time is given, assume it's for today.
                const datetime = resolution['type'] === 'time'
                    ? new Date(`${now.toLocaleDateString()} ${datevalue}`)
                    : new Date(datevalue);
                if (datetime && earliest < datetime.getTime()) {
                    output = { success: true, date: datetime.toLocaleDateString() };
                    return;
                }
            });
        });
        return output || { success: false, message: "I'm sorry, please enter a date at least an hour out." };
    } catch (error) {
        return {
            success: false,
            message: "I'm sorry, I could not interpret that as an appropriate date. Please enter a date at least an hour out."
        };
    }
}
```

---

## <a name="test-the-bot-locally"></a>Prueba local del bot
1. Ejecute el ejemplo localmente en la máquina. Si necesita instrucciones, consulte el archivo Léame en el ejemplo de [C#](https://aka.ms/cs-primitive-prompt-sample) o [JS](https://aka.ms/js-primitive-prompt-sample).
1. Pruébelo con el emulador, tal como se muestra a continuación.

![primitive-prompts](media/primitive-prompts.png)

## <a name="additional-resources"></a>Recursos adicionales

La [biblioteca de diálogos](bot-builder-concept-dialog.md) proporciona clases que automatizan muchos aspectos de la administración de las conversaciones. 

## <a name="next-step"></a>Paso siguiente

> [!div class="nextstepaction"]
> [Implementación de flujo de conversación secuencial](bot-builder-dialog-manage-conversation-flow.md)

---
title: Administración de un flujo de conversación con avisos primitivos | Microsoft Docs
description: Aprenda a administrar un flujo de conversación con avisos primitivos en el SDK de Bot Builder.
keywords: flujo de conversación, avisos
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 7/20/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0b563197c111a37cf2f0f14fef183d52f38cca66
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999171"
---
# <a name="prompt-users-for-input-using-your-own-prompts"></a>Petición de datos de entrada a los usuarios mediante sus propios mensajes

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

A menudo, una conversación entre un usuario y un bot implica mostrar preguntas (avisos) al usuario para obtener información, analizar la respuesta del usuario y, a continuación, actuar en dicha información. En el tema sobre cómo [preguntar a los usuarios mediante la biblioteca Dialogs](bot-builder-prompts.md), se detalla cómo pedir datos de entrada a los usuarios mediante la biblioteca **Dialogs**. Entre otras cosas, la biblioteca **Dialogs** se encarga del seguimiento de la conversación actual y de la pregunta que se está planteando en cada momento. También proporciona métodos para solicitar y validar diferentes tipos de información como texto, números, fecha y hora, etc.

En determinadas situaciones, puede que la biblioteca integrada **Dialogs** no sea ser la solución adecuada para su bot; **Dialogs** podría agregar demasiada sobrecarga para bots simples, ser demasiado rígida o, por el contrario, no conseguir lo que su bot necesita hacer. En esos casos, puede omitir la biblioteca e implementar su propia lógica de preguntas. En este tema se mostrará cómo configurar un *bot Echo* básico para que pueda administrar la conversación con sus propios avisos.

## <a name="track-prompt-states"></a>Seguimiento del estado de los avisos

En una conversación controlada por avisos, necesita controlar dónde está en la conversación y qué pregunta se está haciendo en cada momento. En el código, esto se traduce en administrar un par de marcas.

Por ejemplo, puede crear un par de propiedades que desea controlar.

Estos estados realizan un seguimiento de qué tema y qué aviso están actualmente activados. Para asegurarse de que estas marcas funcionarán según lo esperado cuando se implementen en la nube, vamos a almacenarlas en el [estado de la conversación](bot-builder-howto-v4-state.md). 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Vamos a crear dos clases para realizar el seguimiento del estado. **TopicState** para realizar un seguimiento del progreso de los avisos de la conversación y **UserProfile** para realizar el seguimiento de la información que proporciona el usuario. Esta información se va a almacenar en el [estado](bot-builder-howto-v4-state.md) del bot en un paso posterior.

```csharp
/// <summary>
/// Contains conversation state information about the conversation in progress.
/// </summary>
public class TopicState
{
    /// <summary>
    /// Identifies the current "topic" of conversation.
    /// </summary>
    public string Topic { get; set; }

    /// <summary>
    /// Indicates whether we asked the user a question last turn, and
    /// if so, what it was.
    /// </summary>
    public string Prompt { get; set; }
}
```

```csharp
/// <summary>
/// Contains user state information for the user's profile.
/// </summary>
public class UserProfile
{
    public string UserName { get; set; }

    public int? Age { get; set; }

    public string WorkPlace { get; set; }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**

```javascript
const storage = new MemoryStorage(); // Volatile memory
const conversationState = new ConversationState(storage);
const userState = new UserState(storage);
const dialogState = conversationState.createProperty('dialogState');
const userProfile = userState.createProperty('userProfile');
```

Coloque este código dentro de la lógica principal del bot.

```javascript
// Pull the state of the dialog out of the conversation state manager.
const convo = await dialogState.get(context, {
    prompt: undefined,
    topic: 'profile'
});

// Pull the user profile out of the user state manager.
const userProfile = await userProfile.get(context, {  // Define the user's profile object
        "userName": undefined,
        "age": undefined,
        "workPlace": undefined
    }
);
```

---

## <a name="manage-a-topic-of-conversation"></a>Administración de un tema de conversación

En una conversación secuencial, como aquellas en las que se reúne información del usuario, necesita saber cuándo ha escrito el usuario la conversación y en qué parte de ella está. Puede realizar un seguimiento de esto en el controlador principal de turnos del bot. Para ello, puede establecer y comprobar las marcas del aviso definidas anteriormente y, a continuación, actuar en consecuencia. El ejemplo siguiente muestra cómo puede usar estas marcas para recopilar información del perfil del usuario a través de la conversación.

El código del bot se presenta aquí y se describe en la sección siguiente.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Para ASP.NET Core, debemos configurar el bot y la inyección de dependencias en primer lugar.

Cambie el nombre del archivo **EchoWithCounterBot.cs** a **PrimitivePromptsBot.cs** y actualice también el nombre de la clase. Esta clase contiene la lógica del bot y volveremos para actualizarla en breve.

Cambie el nombre del archivo **EchoBotAccessors.cs** a **BotAccessors.cs** y actualice también el nombre de la clase. Esta clase contiene los objetos de administración de estado y los descriptores de acceso de la propiedad de estado del bot. Actualice la definición según lo siguiente.

```csharp
using Microsoft.Bot.Builder;
using System;

/// <summary>
/// Contains the state and state property accessors for the primitive prompts bot.
/// </summary>
public class BotAccessors
{
    public const string TopicStateName = "PrimitivePrompts.TopicStateAccessor";

    public const string UserProfileName = "PrimitivePrompts.UserProfileAccessor";

    public ConversationState ConversationState { get; }

    public UserState UserState { get; }

    public IStatePropertyAccessor<TopicState> TopicStateAccessor { get; set; }

    public IStatePropertyAccessor<UserProfile> UserProfileAccessor { get; set; }

    public BotAccessors(ConversationState conversationState, UserState userState)
    {
        if (conversationState is null)
        {
            throw new ArgumentNullException(nameof(conversationState));
        }

        if (userState is null)
        {
            throw new ArgumentNullException(nameof(userState));
        }

        this.ConversationState = conversationState;
        this.UserState = userState;
    }
}
```

En el archivo **Startup.cs**, actualice el método `ConfigureServices` a partir de donde se configura el objeto `IStorage`.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<PrimitivePromptsBot>(options =>
    {
        // ...

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        var conversationState = new ConversationState(dataStore);
        options.State.Add(conversationState);

        var userState = new UserState(dataStore);
        options.State.Add(userState);
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
            TopicStateAccessor = conversationState.CreateProperty<TopicState>(BotAccessors.TopicStateName),
            UserProfileAccessor = userState.CreateProperty<UserProfile>(BotAccessors.UserProfileName),
        };

        return accessors;
    });
}
```

Por último, actualice la lógica del bot en el archivo **PrimitivePromptsBot.cs**.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

public class PrimitivePromptsBot : IBot
{
    public const string ProfileTopic = "profile";

    /// <summary>
    /// Describes a field in the user profile.
    /// </summary>
    private class UserFieldInfo
    {
        /// <summary>
        /// The ID to use for this field.
        /// </summary>
        public string Key { get; set; }

        /// <summary>
        /// The prompt to use to ask for a value for this field.
        /// </summary>
        public string Prompt { get; set; }

        /// <summary>
        /// Gets the value of the corresponding field.
        /// </summary>
        public Func<UserProfile, string> GetValue { get; set; }

        /// <summary>
        /// Sets the value of the corresponding field.
        /// </summary>
        public Action<UserProfile, string> SetValue { get; set; }
    }

    /// <summary>
    /// The prompts for the user profile, indexed by field name.
    /// </summary>
    private static List<UserFieldInfo> UserFields { get; } = new List<UserFieldInfo>
    {
        new UserFieldInfo {
            Key = nameof(UserProfile.UserName),
            Prompt = "What is your name?",
            GetValue = (profile) => profile.UserName,
            SetValue = (profile, value) => profile.UserName = value,
        },
        new UserFieldInfo {
            Key = nameof(UserProfile.Age),
            Prompt = "How old are you?",
            GetValue = (profile) => profile.Age.HasValue? profile.Age.Value.ToString() : null,
            SetValue = (profile, value) =>
            {
                if (int.TryParse(value, out int age))
                {
                    profile.Age = age;
                }
            },
        },
        new UserFieldInfo {
            Key = nameof(UserProfile.WorkPlace),
            Prompt = "Where do you work?",
            GetValue = (profile) => profile.WorkPlace,
            SetValue = (profile, value) => profile.WorkPlace = value,
        },
    };

    /// <summary>
    /// The state and state accessors for the bot.
    /// </summary>
    private BotAccessors Accessors { get; }

    public PrimitivePromptsBot(BotAccessors accessors)
    {
        Accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));
    }

    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        if (turnContext.Activity.Type is ActivityTypes.Message)
        {
            // Use the state property accessors to get the topic state and user profile.
            TopicState topicState = await Accessors.TopicStateAccessor.GetAsync(
                turnContext,
                () => new TopicState { Topic = ProfileTopic, Prompt = null },
                cancellationToken);
            UserProfile userProfile = await Accessors.UserProfileAccessor.GetAsync(
                turnContext,
                () => new UserProfile(),
                cancellationToken);

            // Check whether we need more information.
            if (topicState.Topic is ProfileTopic)
            {
                // If we're expecting input, record it in the user's profile.
                if (topicState.Prompt != null)
                {
                    UserFieldInfo field = UserFields.First(f => f.Key.Equals(topicState.Prompt));
                    field.SetValue(userProfile, turnContext.Activity.Text.Trim());
                }

                // Determine which fields are not yet set.
                List<UserFieldInfo> emptyFields = UserFields.Where(f => f.GetValue(userProfile) is null).ToList();

                if (emptyFields.Any())
                {
                    // If all the fields are empty, send a welcome message.
                    if (emptyFields.Count == UserFields.Count)
                    {
                        await turnContext.SendActivityAsync("Welcome new user, please fill out your profile information.");
                    }

                    // We have at least one empty field. Prompt for the next empty field,
                    // and update the prompt flag to indicate which prompt we just sent,
                    // so that the response can be captured at the beginning of the next turn.
                    UserFieldInfo field = emptyFields.First();
                    await turnContext.SendActivityAsync(field.Prompt);
                    topicState.Prompt = field.Key;
                }
                else
                {
                    // Our user profile is complete!
                    await turnContext.SendActivityAsync($"Thank you, {userProfile.UserName}. Your profile is complete.");
                    topicState.Prompt = null;
                    topicState.Topic = null;
                }
            }
            else if (turnContext.Activity.Text.Trim().Equals("hi", StringComparison.InvariantCultureIgnoreCase))
            {
                await turnContext.SendActivityAsync($"Hi. {userProfile.UserName}.");
            }
            else
            {
                await turnContext.SendActivityAsync("Hi. I'm the Contoso cafe bot.");
            }

            // Use the state property accessors to update the topic state and user profile.
            await Accessors.TopicStateAccessor.SetAsync(turnContext, topicState, cancellationToken);
            await Accessors.UserProfileAccessor.SetAsync(turnContext, userProfile, cancellationToken);

            // Save any state changes to storage.
            await Accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
            await Accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**

```javascript

server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === ActivityTypes.Message);

        // Set up a list of fields that we need to collect from the user.
        const fields = {
            userName: "What is your name?",
            age: "How old are you?",
            workPlace: "Where do you work?"
        }

        // Pull the state of the dialog out of the conversation state manager.
        const convo = await dialogState.get(context, {
            topic: 'profile',
            prompt: undefined
        });

        // Pull the user profile out of the user state manager.
        const userProfile = await userProfile.get(context, {  // Define the user's profile object
                "userName": undefined,
                "age": undefined,
                "workPlace": undefined
            }
        );

        if (isMessage) {

            if (convo.topic === 'profile') {
                // If a prompt flag is set in the conversation state, use it to capture the incoming value
                // into the appropriate field of the user profile.
                if (convo.prompt) {
                    userProfile[convo.prompt] = context.activity.text;
                }

                 // Determine which fields are not yet set.
                 const empty_fields = [];
                 Object.keys(fields).forEach(function(key) {
                    if (!userProfile[key]) {
                        empty_fields.push({
                           key: key,
                           prompt: fields[key]
                        });
                     }
                 });

                 if (empty_fields.length) {

                    // If all the fields are empty, send a welcome message.
                    if (empty_fields.length == Object.keys(fields).length) {
                        await context.sendActivity('Welcome new user, please fill out your profile information.');
                    }

                    // We have at least one empty field. Prompt for the next empty field.
                    await context.sendActivity(empty_fields[0].prompt);

                    // update the flag to indicate which prompt we just sent
                    // so that the response can be captured at the beginning of the next turn.
                    convo.prompt = empty_fields[0].key;

                 } else {
                    // Our user profile is complete!
                    await context.sendActivity('Thank you. Your profile is complete.');
                    convo.prompt = null;
                    convo.topic = null;

                 }
            } else if (context.activity.text && context.activity.text.match(/hi/ig)) {
                // Check to see if the user said "hi" and respond with a greeting
                await context.sendActivity(`Hi ${ userProfile.userName }.`);
            } else {
                // Default message
                await context.sendActivity("Hi. I'm the Contoso bot.");
            }
        }

        // End the turn by writing state changes back to storage
        await conversationState.saveChanges(context);
        await userState.saveChanges(context);
    });
});

```

---

El ejemplo de código anterior inicializa la marca _topic_ como `profile` con el fin de iniciar la conversación de recopilación de perfil. Para avanzar en la conversación, el bot actualiza la marca _prompt_ a un valor que representa la pregunta actual. Cuando esta marca está establecida en el valor apropiado, el bot sabrá qué hacer con el siguiente mensaje recibido del usuario y procesarlo en consecuencia.

Por último, las marcas se restablecen cuando el bot termina de pedir información. En caso contrario, el bot quedaría en un bucle, sin moverse después de la última pregunta.

Puede extender este patrón para administrar flujos de conversación más complejos mediante la definición de otras marcas o la bifurcación de la conversación en función de la entrada del usuario.

## <a name="manage-multiple-topics-of-conversations"></a>Administración de varios temas de conversación

Una vez que es capaz de controlar un tema de conversación, puede ampliar la lógica del bot para controlar varios temas de conversación. Se pueden controlar varios temas comprobando condiciones adicionales para, después, tomar la ruta adecuada.

Puede ampliar el ejemplo anterior para permitir otras características y temas de conversación, como reservar una mesa o realizar un pedido. Agregue marcas adicionales al estado del tema según sea necesario para realizar un seguimiento de la conversación.

También resultará útil dividir el código en funciones o métodos independientes, lo que facilita seguir el flujo de la conversación. Un patrón común es hacer que el bot evalúe el mensaje y el estado y, a continuación, delegar el control a las funciones que implementan los detalles de la característica.

Para ayudar a los usuarios a moverse mejor entre varios temas de conversación, considere proporcionar un menú principal. Por ejemplo, mediante el uso de [acciones sugeridas](bot-builder-howto-add-suggested-actions.md) puede dar al usuario la opción de qué tema elegir en lugar de tener que averiguar lo que puede hacer el bot.

## <a name="validate-user-input"></a>Validación de entradas de usuario

La biblioteca **Dialogs** proporciona formas integradas para validar la entrada del usuario, pero también podemos hacerlo con nuestros propios avisos. Por ejemplo, si se solicita la edad del usuario, deseamos asegurarnos de que se obtiene un número y no algo como "Bob" como respuesta.

El análisis de un número o de una fecha y hora es una tarea compleja que queda fuera del ámbito de este tema. Afortunadamente, hay una biblioteca que podemos aprovechar. Para analizar esta información, usamos la biblioteca [Text Recognizer de Microsoft](https://github.com/Microsoft/Recognizers-Text). Este paquete está disponible mediante NuGet o también puede descargarlo desde el repositorio. (Se incluye en la biblioteca **Dialogs**, que vale la pena mencionar, aunque no la vamos a usar aquí).

Esta biblioteca es especialmente útil para analizar entradas complejas, como fechas, horas o números de teléfono. En este ejemplo, se valida un número para el tamaño de la reserva de una cena, pero la misma idea se puede extender para validaciones mucho más complejas.

En el ejemplo siguiente solo se muestra el uso de `RecognizeNumber`. Encontrará detalles sobre cómo usar otros métodos de Recognizer de la biblioteca en la [documentación del repositorio](https://github.com/Microsoft/Recognizers-Text/blob/master/README.md).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Para usar la biblioteca **Microsoft.Recognizers.Text.Number**, incluya el paquete de NuGet y agréguelo a las instrucciones using.

```csharp
using Microsoft.Recognizers.Text.Number;
using Microsoft.Recognizers.Text;
using System.Linq;
```

A continuación, defina una función que realmente realiza la validación.

```csharp
private async Task<bool> ValidatePartySize(ITurnContext context, string value)
{
    try
    {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = NumberRecognizer.RecognizeNumber(value, Culture.English);

        // Attempt to convert the Recognizer result to an integer
        int.TryParse(result.First().Text, out int partySize);

        if (partySize < 6)
        {
            throw new Exception("Party size too small.");
        }
        else if (partySize > 20)
        {
            throw new Exception("Party size too big.");
        }

        // If we got through this, the number is valid
        return true;
    }
    catch (Exception)
    {
        await context.SendActivityAsync("Error with your party size. < br /> Please specify a number between 6 - 20.");
        return false;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Para usar la biblioteca **Recognizers**, debe incluirla en una instrucción require en **app.js**:

```javascript
// Required packages for this bot
var Recognizers = require('@microsoft/recognizers-text-suite');
```

A continuación, defina una función que realmente realiza la validación.

```javascript
// Support party size between 6 and 20 only
async function validatePartySize(context, input){
    try {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = Recognizers.recognizeNumber(input, Recognizers.Culture.English);
        var value = parseInt(results[0].resolution.value);

        if (value < 6) {
            throw new Error(`Party size too small.`);
        } else if(value > 20){
            throw new Error(`Party size too big.`);
        }
        return true; // Return the valid value
    } catch (err){
        await context.sendActivity(`${err.message} <br/>Please specify a number between 6 - 20.`);
        return false;
    }
}
```

---

Al procesar la respuesta del usuario al aviso, llame a la función de validación antes de continuar con el siguiente aviso. Si se produce un error en la validación, formule la pregunta de nuevo.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
if (topicState.Prompt == "partySize")
{
    if (await ValidatePartySize(turnContext, turnContext.Activity.Text))
    {
        // Save user's response in our state, ReservationInfo, which
        // is a new class we've added to our state
        // UserFieldInfo partySize;
        partySize.SetValue(userProfile, turnContext.Activity.Text);

        // Ask next question.
        topicState.Prompt = "reserveName";
        await turnContext.SendActivityAsync("Who's name will this be under?");
    }
    else
    {
        // Ask again.
        await turnContext.SendActivityAsync("How many people are in your party?");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**

```javascript
// ...
if (convo.prompt == "partySize") {
    if (await validatePartySize(context, context.activity.text)) {
        // Save user's response
        reservationInfo.partySize = context.activity.text;

        // Ask next question
        convo.prompt = "reserveName";
        await context.sendActivity("Who's name will this be under?");
    } else {
        // Ask again
        await context.sendActivity("How many people are in your party?");
    }
}
// ...
```

---

## <a name="next-step"></a>Paso siguiente

Ahora que conoce cómo administrar los estados del aviso, echemos un vistazo a cómo aprovechar la biblioteca **Dialogs** para pedir datos de entrada a los usuarios.

> [!div class="nextstepaction"]
> [Petición de datos de entrada a los usuarios mediante el uso de Dialogs](bot-builder-prompts.md)

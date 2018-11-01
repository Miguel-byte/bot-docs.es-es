---
title: Desarrollo de la bienvenida al usuario | Microsoft Docs
description: Aprenda a desarrollar su bot para proporcionar una experiencia de bienvenida al usuario.
keywords: introducción, desarrollo, experiencia de usuario, bienvenida, experiencia personalizada, C#, JS
author: dashel
ms.author: dashel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/31/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f5549385106cab890269e9a5fafacc00dc331520
ms.sourcegitcommit: a496714fb72550a743d738702f4f79e254c69d06
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/01/2018
ms.locfileid: "50736613"
---
# <a name="send-welcome-message-to-users"></a>Envío de mensajes de bienvenida a los usuarios

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

En nuestro anterior artículo sobre diseño de [bienvenida al usuario](./bot-builder-welcome-user.md) se comentaban distintos procedimientos recomendados que se pueden implementar para garantizar una interacción inicial satisfactoria de los usuarios con el bot. En este artículo se amplía el tema con códigos de ejemplo breves que le ayudarán a dar la bienvenida a los usuarios al bot.

## <a name="same-welcome-for-different-channels"></a>Misma bienvenida para diferentes canales

En el siguiente ejemplo se observa en caso de actividad de _actualización de la conversación_ nueva, envía solo un mensaje de bienvenida cuando el usuario se une a la conversación y establece una marca de estado de solicitud para ignorar las entradas iniciales del usuario en la conversación. El código de ejemplo siguiente usa los ejemplos de bienvenida al usuario del repositorio de GitHub para código [C#](https://aka.ms/bot-welcome-sample-cs) y [JS](https://aka.ms/bot-welcome-sample-js).

[!INCLUDE [alert-await-send-activity](../includes/alert-await-send-activity.md)]

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Este conjunto de bibliotecas se utiliza para admitir el siguiente ejemplo de código de C#

```csharp
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;
```

Ahora es necesario crear un objeto de estado para un usuario determinado de una conversación y su descriptor de acceso.

```csharp
/// The state object is used to keep track of various state related to a user in a conversation.
/// In this example, we are tracking if the bot has replied to customer first interaction.
public class WelcomeUserState
{
    public bool DidBotWelcomeUser { get; set; } = false;
}

/// Initializes a new instance of the <see cref="WelcomeUserStateAccessors"/> class.
public class WelcomeUserStateAccessors
{
    public WelcomeUserStateAccessors(UserState userState)
    {
        this.UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    public IStatePropertyAccessor<bool> DidBotWelcomeUser { get; set; }

    public UserState UserState { get; }
}
```

A continuación, simplemente se comprueban las actividades de actualización para ver si se ha agregado un nuevo usuario a la conversación y se le envía un mensaje de bienvenida.

```csharp
public class WelcomeUserBot : IBot
{
// Generic message to be sent to user
private const string _genericMessage = @"This is a simple Welcome Bot sample. You can say 'intro' to 
                                         see the introduction card. If you are running this bot in the Bot 
                                         Framework Emulator, press the 'Start Over' button to simulate user joining a bot or a channel";

// The bot state accessor object. Use this to access specific state properties.
private readonly WelcomeUserStateAccessors _welcomeUserStateAccessors;

// Initializes a new instance of the <see cref="WelcomeUserBot"/> class.
public WelcomeUserBot(WelcomeUserStateAccessors statePropertyAccessor)
{
    _welcomeUserStateAccessors = statePropertyAccessor ?? throw new System.ArgumentNullException("state accessor can't be null");
}

// Every conversation turn for our WelcomeUser Bot will call this method, including
// any type of activities such as ConversationUpdate or ContactRelationUpdate which
// are sent when a user joins a conversation.
// This bot doesn't use any dialogs; it's "single turn" processing, meaning a single
// request and response.
// This bot uses UserState to keep track of first message a user sends.
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = new CancellationToken())
{
    // Use state accessor to extract the didBotWelcomeUser flag
    var didBotWelcomeUser = await _welcomeUserStateAccessors.DidBotWelcomeUser.GetAsync(turnContext, () => false);

    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Your bot should proactively send a welcome message to a personal chat the first time
        // (and only the first time) a user initiates a personal chat with your bot.
        if (didBotWelcomeUser == false)
        {
            // Update user state flag to reflect bot handled first user interaction.
            await _welcomeUserStateAccessors.DidBotWelcomeUser.SetAsync(turnContext, true);
            await _welcomeUserStateAccessors.UserState.SaveChangesAsync(turnContext);

            // the channel should sends the user name in the 'From' object
            var userName = turnContext.Activity.From.Name;

            await turnContext.SendActivityAsync($"You are seeing this message because this was your first message ever to this bot.", cancellationToken: cancellationToken);
            await turnContext.SendActivityAsync($"It is a good practice to welcome the user and provide a personal greeting. For example, welcome {userName}.", cancellationToken: cancellationToken);
        }
    }
    else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate) // Greet when users are added to the conversation.
    {
        if (turnContext.Activity.MembersAdded.Any())
        {
            // Iterate over all new members added to the conversation
            foreach (var member in turnContext.Activity.MembersAdded)
            {
                // Greet anyone that was not the target (recipient) of this message
                // the 'bot' is the recipient for events from the channel,
                // turnContext.Activity.MembersAdded == turnContext.Activity.Recipient.Id indicates the
                // bot was added to the conversation.
                if (member.Id != turnContext.Activity.Recipient.Id)
                {
                    await turnContext.SendActivityAsync($"Hi there - {member.Name}. Welcome to the 'Welcome User' Bot. This bot will introduce you to welcoming and greeting users.", cancellationToken: cancellationToken);
                    await turnContext.SendActivityAsync($"You are seeing this message because the bot recieved atleast one 'ConversationUpdate' event,indicating you (and possibly others) joined the conversation. If you are using the emulator, pressing the 'Start Over' button to trigger this event again. The specifics of the 'ConversationUpdate' event depends on the channel. You can read more information at https://aka.ms/about-botframewor-welcome-user", cancellationToken: cancellationToken);
                    await turnContext.SendActivityAsync($"It is a good pattern to use this event to send general greeting to user, explaning what your bot can do. In this example, the bot handles 'hello', 'hi', 'help' and 'intro. Try it now, type 'hi'", cancellationToken: cancellationToken);
                }
            }
        }
    }
    // save any state changes made to your state objects.
    await _welcomeUserStateAccessors.UserState.SaveChangesAsync(turnContext);
}
```

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Este código de JavaScript envía un mensaje de bienvenida cuando se agrega un usuario. Esto se realiza mediante la comprobación de la actividad de la conversación y la verificación de que se ha agregado un nuevo miembro a la conversación.

``` javascript
// Import required Bot Framework classes.
const { ActivityTypes } = require('botbuilder');
const { CardFactory } = require('botbuilder');

// Welcomed User property name
const WELCOMED_USER = 'DidBotWelcomeUser';

class MainDialog {
    constructor (userState) {
        // Creates a new user property accessor.
        // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
        this.welcomedUserPropery = userState.createProperty(WELCOMED_USER);
    }

    async onTurn(turnContext) 
    {
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        if (turnContext.activity.type === ActivityTypes.Message) 
        {
            // Read UserState. If the 'DidBotWelcomeUser' does not exist (first time ever for a user)
            // set the default to false.
            let didBotWelcomeUser = await this.welcomedUserPropery.get(turnContext, false);

            // Your bot should proactively send a welcome message to a personal chat the first time
            // (and only the first time) a user initiates a personal chat with your bot.
            if (didBotWelcomeUser === false) 
            {
                // The channel should send the user name in the 'From' object
                let userName = turnContext.activity.from.name;
                await turnContext.sendActivity("You are seeing this message because this was your first message ever sent to this bot.");
                await turnContext.sendActivity(`It is a good practice to welcome the user and provdie personal greeting. For example, welcome ${userName}.`);
                
                // Set the flag indicating the bot handled the user's first message.
                await this.welcomedUserPropery.set(turnContext, true);
            }
            . . .
            
            // Save state changes
            await this.userState.saveChanges(turnContext);
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate) {
            // Send greeting when users are added to the conversation.
            await this.sendWelcomeMessage(turnContext);
        } else {
            // Generic message for all other activities
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
    }
    
    // Sends welcome messages to conversation members when they join the conversation.
    // Messages are only sent to conversation members who aren't the bot.
    async sendWelcomeMessage(turnContext) {
        // If any new membmers added to the conversation
        if (turnContext.activity && turnContext.activity.membersAdded) {
            // Define a promise that will welcome the user
            async function welcomeUserFunc(conversationMember) {
                // Greet anyone that was not the target (recipient) of this message.
                // The bot is the recipient of all events from the channel, including all ConversationUpdate-type activities
                // turnContext.activity.membersAdded !== turnContext.activity.aecipient.id indicates 
                // a user was added to the conversation 
                if (conversationMember.id !== this.activity.recipient.id) {
                    // Because the TurnContext was bound to this function, the bot can call
                    // `TurnContext.sendActivity` via `this.sendActivity`;
                    await this.sendActivity(`Welcome to the 'Welcome User' Bot. This bot will introduce you to welcoming and greeting users.`);
                    await this.sendActivity("You are seeing this message because the bot recieved atleast one 'ConversationUpdate'" + 
                                            "event,indicating you (and possibly others) joined the conversation. If you are using the emulator, "+ 
                                            "pressing the 'Start Over' button to trigger this event again. The specifics of the 'ConversationUpdate' "+
                                            "event depends on the channel. You can read more information at https://aka.ms/about-botframewor-welcome-user");
                    await this.sendActivity(`It is a good pattern to use this event to send general greeting to user, explaining what your bot can do. `+ 
                                            `In this example, the bot handles 'hello', 'hi', 'help' and 'intro. ` +
                                            `Try it now, type 'hi'`);
                }
            }
    
            // Prepare Promises to greet the  user.
            // The current TurnContext is bound so `replyForReceivedAttachments` can also send replies.
            const replyPromises = turnContext.activity.membersAdded.map(welcomeUserFunc.bind(turnContext));
            await Promise.all(replyPromises);
        }
    }
}

module.exports = MainDialog;
```

---

## <a name="discard-initial-user-input"></a>Descarte de las entradas iniciales del usuario

Para garantizar una buena experiencia para el usuario en todos los canales posibles, evitamos procesar datos de respuesta no válidos mediante una solicitud inicial y la configuración de palabras clave que buscar en las respuestas del usuario.

## <a name="ctabcsharpmulti"></a>[C#](#tab/csharpmulti)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = new CancellationToken())
{
    // Use state accessor to extract the didBotWelcomeUser flag
    var didBotWelcomeUser = await _welcomeUserStateAccessors.DidBotWelcomeUser.GetAsync(turnContext, () => false);

    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Your bot should proactively send a welcome message to a personal chat the first time
        // (and only the first time) a user initiates a personal chat with your bot.
        if (didBotWelcomeUser == false)
        {
            // Previous Code Sample
        }
        else
        {
            // This example hardcodes specific uterances. You should use LUIS or QnA for more advance language understanding.
            var text = turnContext.Activity.Text.ToLowerInvariant();
            switch (text)
            {
                case "hello":
                case "hi":
                    await turnContext.SendActivityAsync($"You said {text}.", cancellationToken: cancellationToken);
                    break;
                case "intro":
                case "help":
                default:
                    await turnContext.SendActivityAsync(_genericMessage, cancellationToken: cancellationToken);
                    break;
            }
        }
    }
    else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate) // Greet when users are added to the conversation.
    {
        if (turnContext.Activity.MembersAdded.Any())
        {
            // Iterate over all new members added to the conversation
            foreach (var member in turnContext.Activity.MembersAdded)
            {
                // Previous Code Sample
            }
        }
    }
    else
    {
        // Default behaivor for all other type of events.
        var ev = turnContext.Activity.AsEventActivity();
        await turnContext.SendActivityAsync($"Received event: {ev.Name}");
    }
    // save any state changes made to your state objects.
    await _welcomeUserStateAccessors.UserState.SaveChangesAsync(turnContext);
}

```

## <a name="javascripttabjsmulti"></a>[JavaScript](#tab/jsmulti)

``` javascript
class MainDialog 
{ 
    // Previous Code Sample
    
    async onTurn(turnContext) 
    {
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        if (turnContext.activity.type === ActivityTypes.Message) {
            
            // Previous Code Sample
            
            if (didBotWelcomeUser === false) 
            {
                // Previous Code Sample
            }
            else 
            {
                // This example hardcodes specific uterances. You should use LUIS or QnA for more advance language understanding.
                let text = turnContext.activity.text.toLowerCase();
                switch (text) 
                {
                    case "hello":
                    case "hi":
                        await turnContext.sendActivity(`You said "${turnContext.activity.text}"`);
                        break;
                    case "intro":
                    case "help":
                    default :
                        await turnContext.sendActivity(`This is a simple Welcome Bot sample. You can say 'intro' to 
                                                        see the introduction card. If you are running this bot in the Bot 
                                                        Framework Emulator, press the 'Start Over' button to simulate user joining a bot or a channel`);
                }
            }
        }
       
       // Previous Sample Code
    } 
}
```

---

## <a name="using-adaptive-card-greeting"></a>Uso de tarjetas adaptables de saludo

Otra forma de saludar a los usuarios es usar tarjetas adaptables de saludo. Puede aprender más sobre las tarjetas adaptables de saludo aquí: [Envío de una tarjeta adaptable](./bot-builder-howto-add-media-attachments.md).

## <a name="ctabcsharpwelcomeback"></a>[C#](#tab/csharpwelcomeback)

```csharp
// Sends an adaptive card greeting.
private static async Task SendIntroCardAsync(ITurnContext turnContext, CancellationToken cancellationToken)
{
    var response = turnContext.Activity.CreateReply();

    var introCard = File.ReadAllText(@".\Resources\IntroCard.json");

    response.Attachments = new List<Attachment>
    {
        new Attachment()
        {
            ContentType = "application/vnd.microsoft.card.adaptive",
            Content = JsonConvert.DeserializeObject(introCard),
        },
    };

    await turnContext.SendActivityAsync(response, cancellationToken);
}
```

A continuación, podemos enviarle la tarjeta mediante el siguiente comando await. Pongamos esto en los bots: _switch (texto)_ _case "hel

```csharp
switch (text)
{
    case "hello":
    case "hi":
        await turnContext.SendActivityAsync($"You said {text}.", cancellationToken: cancellationToken);
        break;
    case "intro":
    case "help":
        await SendIntroCardAsync(turnContext, cancellationToken);
        break;
    default:
        await turnContext.SendActivityAsync(_genericMessage, cancellationToken: cancellationToken);
        break;
}
```


## <a name="javascripttabjswelcomeback"></a>[JavaScript](#tab/jswelcomeback)

Primero se agrega la tarjeta adaptable al bot en la parte superior de _index.js_, justo debajo de las importaciones.

``` javascript
// Adaptive Card content
const IntroCard = require('./Resources/IntroCard.json');
```

A continuación, podemos utilizar el siguiente código en la sección _switch (texto)_ _case "help"_ del bot para responder a la solicitud de los usuarios con la tarjeta adaptable.

``` javascript
switch (text) 
{
    case "hello":
    case "hi":
        await turnContext.sendActivity(`You said "${turnContext.activity.text}"`);
        break;
    case "intro":
    case "help":
        await turnContext.sendActivity({
            text: 'Intro Adaptive Card',
            attachments: [CardFactory.adaptiveCard(IntroCard)]
        });
        break;
    default :
        await turnContext.sendActivity(`This is a simple Welcome Bot sample. You can say 'intro' to 
                                        see the introduction card. If you are running this bot in the Bot 
                                        Framework Emulator, press the 'Start Over' button to simulate user joining a bot or a channel`);
}
```
---

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Petición de datos de entrada a los usuarios con la biblioteca Dialogs](bot-builder-prompts.md)

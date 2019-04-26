---
title: Envío de mensajes de bienvenida a los usuarios | Microsoft Docs
description: Aprenda a desarrollar su bot para proporcionar una experiencia de bienvenida al usuario.
keywords: overview, develop, user experience, welcome, personalized experience, C#, JS, welcome message, bot, greet, greeting
author: DanDev33
ms.author: v-dashel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/19/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 19b96b860f0faa98a484f17f7814324a5f62f0eb
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904938"
---
# <a name="send-welcome-message-to-users"></a>Envío de mensajes de bienvenida a los usuarios

[!INCLUDE[applies-to](../includes/applies-to.md)]

El objetivo principal al crear cualquier bot es que el usuario participe en una conversación que tenga sentido. Una de las mejores formas de lograr este objetivo es asegurarse de que desde el momento en que un usuario se conecta por primera vez, comprende la finalidad principal del bot y sus funcionalidades, es decir, el motivo de que se haya creado el bot. Este artículo proporciona ejemplos de código que le ayudarán a dar la bienvenida a los usuarios del bot.

## <a name="prerequisites"></a>Requisitos previos

- Comprender los [conceptos básicos de bot](bot-builder-basics.md). 
- Una copia del **ejemplo de bienvenida al usuario** en [C#](https://aka.ms/bot-welcome-sample-cs) o en [JS](https://aka.ms/bot-welcome-sample-js). El código del ejemplo se utiliza para explicar cómo enviar mensajes de bienvenida.

## <a name="same-welcome-for-different-channels"></a>Misma bienvenida para diferentes canales

Cada vez que los usuarios interactúan por primera vez con el bot, se debe generar un mensaje de bienvenida. Para lograr esto, puede supervisar los tipos de actividad del bot y esperar nuevas conexiones. Según el canal, cada nueva conexión puede generar hasta dos actividades de actualización de la conversación.

- Una cuando el bot del usuario está conectado a la conversación.
- Y otra cuando el usuario se une a la conversación.

Resulta tentador generar simplemente un mensaje de bienvenida cada vez que se detecta una nueva actualización de la conversación, pero eso puede provocar resultados inesperados cuando se accede al bot mediante diversos canales.

Algunos canales crean una actualización de la conversación cuando un usuario se conecta inicialmente a ese canal y otra solo después de recibir un mensaje de entrada inicial del usuario. Otros canales generan ambas actividades cuando el usuario se conecta inicialmente al canal. Si simplemente espera un evento de actualización de la conversación y muestra un mensaje de bienvenida en un canal con dos actividades de actualización de la conversación, el usuario podría recibir lo siguiente:

![Doble mensaje de bienvenida](./media/double_welcome_message.png)

Este mensaje duplicado se puede evitar mediante la generación de un mensaje de bienvenida inicial solo para el segundo evento de actualización de la conversación. El segundo evento se puede detectar en estos dos casos:

- Se ha producido un evento de actualización de la conversación.
- Se ha agregado un nuevo miembro (usuario) a la conversación.

El ejemplo de código siguiente se supervisa para cualquier *nueva actividad de actualización de la conversación* y envía solo un mensaje de bienvenida al nuevo usuario que se une a la conversación. Después del primer evento _conversationUpdate_ del bot, el propio bot se agrega a sí mismo como _Recipient_ de la actividad del canal. Después, el código comprueba si se ha agregado Member == _turnContext.Activity.Recipient.Id_. Si es "true", ha detectado el evento de actualización de la conversación inicial y omite el código que comprueba los usuarios nuevos que se conectan.

[!INCLUDE [alert-await-send-activity](../includes/alert-await-send-activity.md)]

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dentro del código de ejemplo de C#, **Startup.cs** ha definido "WelcomeUserStateAccessors" como un singleton o servicio, y ha agregado "UserState" al estado de la aplicación. Ahora, los usaremos para crear un objeto de estado para un usuario determinado de una conversación y su descriptor de acceso.

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
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    public static string WelcomeUserName { get; } = $"{nameof(WelcomeUserStateAccessors)}.WelcomeUserState";

    public IStatePropertyAccessor<WelcomeUserState> WelcomeUserState { get; set; }

    public UserState UserState { get; }
}
```

En **WelcomeUserBot**, se comprueban las actividades de actualización para ver si se ha agregado un nuevo usuario a la conversación y se le envía un mensaje de bienvenida.

```csharp
// Messages sent to the user.
private const string WelcomeMessage = @"This is a simple Welcome Bot sample.This bot will introduce you
                                        to welcoming and greeting users. You can say 'intro' to see the
                                        introduction card. If you are running this bot in the Bot Framework
                                        Emulator, press the 'Start Over' button to simulate user joining
                                        a bot or a channel";

private const string InfoMessage = @"You are seeing this message because the bot received at least one
                                    'ConversationUpdate' event, indicating you (and possibly others)
                                    joined the conversation. If you are using the emulator, pressing
                                    the 'Start Over' button to trigger this event again. The specifics
                                    of the 'ConversationUpdate' event depends on the channel. You can
                                    read more information at:
                                        https://aka.ms/about-botframework-welcome-user";

private const string PatternMessage = @"It is a good pattern to use this event to send general greeting
                                        to user, explaining what your bot can do. In this example, the bot
                                        handles 'hello', 'hi', 'help' and 'intro. Try it now, type 'hi'";

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
    // use state accessor to extract the didBotWelcomeUser flag
    var didBotWelcomeUser = await _welcomeUserStateAccessors.WelcomeUserState.GetAsync(turnContext, () => new WelcomeUserState());

    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Your bot should proactively send a welcome message to a personal chat the first time
        // (and only the first time) a user initiates a personal chat with your bot.
        if (didBotWelcomeUser.DidBotWelcomeUser == false)
        {
            didBotWelcomeUser.DidBotWelcomeUser = true;
            // Update user state flag to reflect bot handled first user interaction.
            await _welcomeUserStateAccessors.WelcomeUserState.SetAsync(turnContext, didBotWelcomeUser);
            await _welcomeUserStateAccessors.UserState.SaveChangesAsync(turnContext);

            // the channel should sends the user name in the 'From' object
            var userName = turnContext.Activity.From.Name;

            await turnContext.SendActivityAsync(
                $"You are seeing this message because this was your first message ever to this bot.",
                cancellationToken: cancellationToken);
            await turnContext.SendActivityAsync(
                $"It is a good practice to welcome the user and provide personal greeting. For example, welcome {userName}.",
                cancellationToken: cancellationToken);
        }
    }

    // Greet when users are added to the conversation.
    else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
    {
        if (turnContext.Activity.MembersAdded != null)
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
                    await turnContext.SendActivityAsync($"Hi there - {member.Name}. {WelcomeMessage}", cancellationToken: cancellationToken);
                    await turnContext.SendActivityAsync(InfoMessage, cancellationToken: cancellationToken);
                    await turnContext.SendActivityAsync(PatternMessage, cancellationToken: cancellationToken);
                }
            }
        }
    }
    else
    {
        // Default behavior for all other type of activities.
        await turnContext.SendActivityAsync($"{turnContext.Activity.Type} activity detected");
    }

    // save any state changes made to your state objects.
    await _welcomeUserStateAccessors.UserState.SaveChangesAsync(turnContext);
}
```

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Este código de JavaScript envía un mensaje de bienvenida cuando se agrega un usuario. Esto se realiza mediante la comprobación de la actividad de la conversación y la verificación de que se ha agregado un nuevo miembro a la conversación.

```javascript
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License.

// Import required Bot Framework classes.
const { ActivityTypes } = require('botbuilder');
const { CardFactory } = require('botbuilder');

// Adaptive Card content
const IntroCard = require('./resources/IntroCard.json');

// Welcomed User property name
const WELCOMED_USER = 'welcomedUserProperty';

class WelcomeBot {
    /**
     *
     * @param {UserState} User state to persist boolean flag to indicate
     *                    if the bot had already welcomed the user
     */
    constructor(userState) {
        // Creates a new user property accessor.
        // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
        this.welcomedUserProperty = userState.createProperty(WELCOMED_USER);

        this.userState = userState;
    }
    /**
     *
     * @param {TurnContext} context on turn context object.
     */
    async onTurn(turnContext) {
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Read UserState. If the 'DidBotWelcomedUser' does not exist (first time ever for a user)
            // set the default to false.
            const didBotWelcomedUser = await this.welcomedUserProperty.get(turnContext, false);

            // Your bot should proactively send a welcome message to a personal chat the first time
            // (and only the first time) a user initiates a personal chat with your bot.
            if (didBotWelcomedUser === false) {
                // The channel should send the user name in the 'From' object
                let userName = turnContext.activity.from.name;
                await turnContext.sendActivity('You are seeing this message because this was your first message ever sent to this bot.');
                await turnContext.sendActivity(`It is a good practice to welcome the user and provide personal greeting. For example, welcome ${ userName }.`);

                // Set the flag indicating the bot handled the user's first message.
                await this.welcomedUserProperty.set(turnContext, true);
            } else {
                // ...
            }
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

    /**
     * Sends welcome messages to conversation members when they join the conversation.
     * Messages are only sent to conversation members who aren't the bot.
     * @param {TurnContext} turnContext
     */
    async sendWelcomeMessage(turnContext) {
        // Do we have any new members added to the conversation?
        if (turnContext.activity.membersAdded.length !== 0) {
            // Iterate over all new members added to the conversation
            for (let idx in turnContext.activity.membersAdded) {
                // Greet anyone that was not the target (recipient) of this message.
                // Since the bot is the recipient for events from the channel,
                // context.activity.membersAdded === context.activity.recipient.Id indicates the
                // bot was added to the conversation, and the opposite indicates this is a user.
                if (turnContext.activity.membersAdded[idx].id !== turnContext.activity.recipient.id) {
                    await turnContext.sendActivity(`Welcome to the 'Welcome User' Bot. This bot will introduce you to welcoming and greeting users.`);
                    await turnContext.sendActivity("You are seeing this message because the bot received at least one 'ConversationUpdate'" +
                                            'event,indicating you (and possibly others) joined the conversation. If you are using the emulator, ' +
                                            "pressing the 'Start Over' button to trigger this event again. The specifics of the 'ConversationUpdate' " +
                                            'event depends on the channel. You can read more information at https://aka.ms/about-botframework-welcome-user');
                    await turnContext.sendActivity(`It is a good pattern to use this event to send general greeting to user, explaining what your bot can do. ` +
                                            `In this example, the bot handles 'hello', 'hi', 'help' and 'intro. ` +
                                            `Try it now, type 'hi'`);
                }
            }
        }
    }
}

module.exports.WelcomeBot = WelcomeBot;
```

---

## <a name="discard-initial-user-input"></a>Descarte de las entradas iniciales del usuario

También es importante tener en cuenta cuándo la intervención del usuario puede contener realmente información útil, y esto también puede variar según el canal. Para asegurarse de que el usuario tenga una buena experiencia en todos los canales posibles, comprobamos el indicador de estado de _didBotWelcomeUser_ y, si es "false", se evita procesar estos datos de entrada del usuario inicial. En su lugar, proporcionamos al usuario un aviso inicial para obtener su respuesta. La variable _didBotWelcomeUser_ se establece en "true" y nuestro código procesa los datos de entrada del usuario en todas las actividades de mensajes adicionales.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = new CancellationToken())
{
    // use state accessor to extract the didBotWelcomeUser flag
    var didBotWelcomeUser = await _welcomeUserStateAccessors.WelcomeUserState.GetAsync(turnContext, () => new WelcomeUserState());

    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        if (didBotWelcomeUser.DidBotWelcomeUser == false)
        {
            // See previous code sample.
        }
        else
        {
            // This example hard-codes specific utterances. You should use LUIS or QnA for more advanced language understanding.
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
                    await turnContext.SendActivityAsync(WelcomeMessage, cancellationToken: cancellationToken);
                    break;
            }
        }
    }

    // Greet when users are added to the conversation.
    // Note that all channels do not send the conversation update activity.
    // If you find that this bot works in the emulator, but does not in
    // another channel the reason is most likely that the channel does not
    // send this activity.
    else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
    {
        // See previous code sample.
    }

    // save any state changes made to your state objects.
    await _welcomeUserStateAccessors.UserState.SaveChangesAsync(turnContext);
}
```

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
class MainDialog
{
    // Previous Code Sample
    async onTurn(turnContext)
    {
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Previous Code Sample
            if (didBotWelcomeUser === false) {
                // Previous Code Sample
            } else  {
                // This example uses an exact match on user's input utterance.
                // Consider using LUIS or QnA for Natural Language Processing.
                let text = turnContext.activity.text.toLowerCase();
                switch (text) {
                case 'hello':
                case 'hi':
                    await turnContext.sendActivity(`You said "${ turnContext.activity.text }"`);
                    break;
                case 'intro':
                case 'help':
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
            }
        }
       // Previous Sample Code
    }
}
```

---

## <a name="using-adaptive-card-greeting"></a>Uso de tarjetas adaptables de saludo

Otra forma de saludar a los usuarios es usar tarjetas adaptables de saludo. Puede aprender más sobre las tarjetas adaptables de saludo aquí: [Envío de una tarjeta adaptable](./bot-builder-howto-add-media-attachments.md).

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Sends an adaptive card greeting.
private static async Task SendIntroCardAsync(ITurnContext turnContext, CancellationToken cancellationToken)
{
    var response = turnContext.Activity.CreateReply();

    var card = new HeroCard();
    card.Title = "Welcome to Bot Framework!";
    card.Text = @"Welcome to Welcome Users bot sample! This Introduction card
                    is a great way to introduce your Bot to the user and suggest
                    some things to get them started. We use this opportunity to
                    recommend a few next steps for learning more creating and deploying bots.";
    card.Images = new List<CardImage>() { new CardImage("https://aka.ms/bf-welcome-card-image") };
    card.Buttons = new List<CardAction>()
    {
        new CardAction(ActionTypes.OpenUrl,
            "Get an overview", null, "Get an overview", "Get an overview",
            "https://docs.microsoft.com/en-us/azure/bot-service/?view=azure-bot-service-4.0"),
        new CardAction(ActionTypes.OpenUrl,
            "Ask a question", null, "Ask a question", "Ask a question",
            "https://stackoverflow.com/questions/tagged/botframework"),
        new CardAction(ActionTypes.OpenUrl,
            "Learn how to deploy", null, "Learn how to deploy", "Learn how to deploy",
            "https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-deploy-azure?view=azure-bot-service-4.0"),
    };

    response.Attachments = new List<Attachment>() { card.ToAttachment() };
    await turnContext.SendActivityAsync(response, cancellationToken);
}
```

A continuación, podemos enviarle la tarjeta mediante el siguiente comando await. Pongamos esto en el elemento _switch (text) case "help"_ del bot.

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
        await turnContext.SendActivityAsync(WelcomeMessage, cancellationToken: cancellationToken);
        break;
}
```

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Primero se agrega la tarjeta adaptable al bot en la parte superior de _index.js_, justo debajo de las importaciones.

```javascript
// Adaptive Card content
const IntroCard = require('./Resources/IntroCard.json');
```

A continuación, podemos utilizar el siguiente código en la sección _switch (texto)_ _case "help"_ del bot para responder a la solicitud de los usuarios con la tarjeta adaptable.

```javascript
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

## <a name="test-the-bot"></a>Probar el bot

Consulte las instrucciones del archivo [Léame](https://aka.ms/bot-welcome-sample-cs) para saber cómo ejecutar y probar el bot.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Recopilación de entradas del usuario](bot-builder-prompts.md)

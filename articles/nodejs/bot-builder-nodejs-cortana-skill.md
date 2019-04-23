---
title: Creación de un bot habilitado por voz con habilidades de Cortana | Microsoft Docs
description: Obtenga información sobre cómo crear un bot habilitado para voz con habilidades de Cortana y Bot Framework SDK para Node.js.
author: DeniseMak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/10/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 690c456a1baa94eab1f0efbed6ce2c2e1f5cb280
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2019
ms.locfileid: "59508162"
---
# <a name="build-a-speech-enabled-bot-with-cortana-skills"></a>Creación de un bot habilitado para la voz con habilidades de Cortana

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-cortana-skill.md)
> - [Node.js](../nodejs/bot-builder-nodejs-cortana-skill.md)

Bot Framework SDK para Node.js le permite crear un bot habilitado para voz al conectarlo al canal de Cortana como una habilidad de Cortana. Las habilidades de Cortana permiten proporcionar funcionalidades mediante Cortana como respuesta a entradas de voz de un usuario.

> [!TIP]
> Para más información sobre qué es una habilidad y qué puede hacer, vea [The Cortana Skills Kit][CortanaGetStarted] (El kit de habilidades de Cortana).

La creación de una habilidad de Cortana con Bot Framework requiere muy poco conocimiento específico de Cortana y se compone principalmente de la creación de un bot. Una de las diferencias principales respecto a otros bots que ha creado es que Cortana tiene componentes visuales y de audio. Para el componente visual, Cortana proporciona un área del lienzo para representar el contenido, como tarjetas. Para el componente de audio, proporcione texto o SSML en los mensajes del bot para que Cortana los lea al usuario, dándole una voz. 

> [!NOTE]
> Cortana está disponible en muchos dispositivos diferentes. Algunos tienen una pantalla mientras que otros, como los altavoces independientes, puede que no. Asegúrese de que su bot es capaz de actuar en ambos escenarios. Consulte las [entidades específicas de Cortana][CortanaSpecificEntities] para obtener información sobre cómo comprobar la información del dispositivo.

## <a name="adding-speech-to-your-bot"></a>Adición de voz al bot

Los mensajes de voz de su bot se representan en el lenguaje de marcado de síntesis de voz (SSML). Bot Framework SDK permite incluir SSML en las respuestas del bot para controlar lo que dice y lo que muestra.

### <a name="sessionsay"></a>session.say

El bot usa el método **session.say** para hablar con el usuario, en lugar de **session.send**. Incluye parámetros opcionales para enviar salidas de SSML, así como datos adjuntos como tarjetas. 

El método tiene este formato:

```session.say(displayText: string, speechText: string, options?: object)```

| Parámetro | DESCRIPCIÓN |
|------|------|
| **displayText** | Un mensaje de texto para mostrar en la interfaz de usuario de Cortana.|
| **speechText** | El texto o SSML que Cortana lee al usuario. |
| **options** | Un objeto [IMessage][IMessage] que puede contener una sugerencia de entrada o datos adjuntos. Las sugerencias de entrada indican si el bot acepta, espera o ignora la entrada. Los datos adjuntos de la tarjeta se muestran en el lienzo de Cortana a continuación de la información **displayText**.   |

La propiedad **InputHint** ayuda a indicar a Cortana si su bot está esperando una entrada del usuario. Si usa un mensaje integrado, este valor se establece automáticamente en el valor predeterminado **expectingInput**.


| Valor | DESCRIPCIÓN |
|------|------|
| **acceptingInput** | El bot está listo para recibir información de forma pasiva, pero no está esperando una respuesta. Cortana acepta la entrada del usuario si el usuario mantiene presionado el botón del micrófono.|
| **expectingInput** | Indica que el bot está esperando una respuesta del usuario de forma activa. Cortana escucha hasta que el usuario habla por el micrófono.  |
||NOTA:  _No_ use **expectingInput** en dispositivos sin interfaz de usuario (dispositivos sin pantalla). Consulte las [pregungas más frecuentes sobre el kit de habilidades de Cortana](https://review.docs.microsoft.com/en-us/cortana/skills/faq).|
| **ignoringInput** | Cortana está ignorando las entradas. El bot puede enviar esta sugerencia si se está procesando activamente una solicitud e ignorará la entrada de los usuarios hasta que se complete la solicitud.  |

El ejemplo siguiente muestra cómo Cortana lee texto sin formato o SSML:

```javascript

// Have Cortana read plain text
session.say('This is the text that Cortana displays', 'This is the text that is spoken by Cortana.');

// Have Cortana read SSML
session.say('This is the text that Cortana displays', '<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">This is the text that is spoken by Cortana.</speak>');

```

Este ejemplo muestra cómo informar a Cortana que se espera la intervención del usuario. El micrófono quedará encendido.

```javascript

// Add an InputHint to let Cortana know to expect user input
session.say('Hi there', 'Hi, what’s your name?', {
    inputHint: builder.InputHint.expectingInput
});

```
<!-- TODO: tip about time limit and batching -->

### <a name="prompts"></a>Mensajes

Además de usar el método **session.say()**, también puede pasar texto o SSML a mensajes integrados con las opciones **speak** y **retrySpeak**.  

```javascript

builder.Prompts.text(session, 'text based prompt', {                                    
    speak: 'Cortana reads this out initially',                                               
    retrySpeak: 'This message is repeated by Cortana after waiting a while for user input',  
    inputHint: builder.InputHint.expectingInput                                              
});

```

<!-- TODO: Link to SSML library -->

Para presentar al usuario una lista de opciones, utilice **Prompts.choice**. La opción **synonyms** permite tener más flexibilidad en el reconocimiento de grabaciones de voz del usuario. La opción **value** se devuelve en **results.response.entity**. La opción **action** especifica la etiqueta que el bot muestra para la opción.

**Prompts.choice** admite opciones ordinales. Esto significa que el usuario puede decir "el primero", "el segundo" o "el tercero" para elegir un elemento de una lista. Por ejemplo, dado el siguiente mensaje, si el usuario pide a Cortana "la segunda opción", el mensaje devolverá el valor 8.

```javascript

        var choices = [
            { value: '4', action: { title: '4 Sides' }, synonyms: 'four|for|4 sided|4 sides' },
            { value: '8', action: { title: '8 Sides' }, synonyms: 'eight|ate|8 sided|8 sides' },
            { value: '12', action: { title: '12 Sides' }, synonyms: 'twelve|12 sided|12 sides' },
            { value: '20', action: { title: '20 Sides' }, synonyms: 'twenty|20 sided|20 sides' },
        ];
        builder.Prompts.choice(session, 'choose_sides', choices, { 
            speak: speak(session, 'choose_sides_ssml') // use helper function to format SSML
        });

```

En el ejemplo anterior, el SSML de la propiedad **speak** del mensaje adopta el formato con el uso de cadenas almacenadas en un archivo de mensajes localizados con el formato siguiente. 

```json

{
    "choose_sides": "__Number of Sides__",
    "choose_sides_ssml": [
        "How many sides on the dice?",
        "Pick your poison.",
        "All the standard sizes are supported."
    ]
}

```

Una función de la aplicación auxiliar compila después el elemento raíz necesario del documento de lenguaje de marcado de síntesis de voz (SSML). 

```javascript

module.exports.speak = function (template, params, options) {
    options = options || {};
    var output = '<speak xmlns="http://www.w3.org/2001/10/synthesis" ' +
        'version="' + (options.version || '1.0') + '" ' +
        'xml:lang="' + (options.lang || 'en-US') + '">';
    output += module.exports.vsprintf(template, params);
    output += '</speak>';
    return output;
}
```

> [!TIP]
> Puede encontrar un módulo de utilidad pequeño (ssml.js) para generar respuestas basadas en SSML del bot en la [habilidad de ejemplo Roller](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill).
> También hay varias bibliotecas útiles de SSML disponibles en [npm](https://www.npmjs.com/search?q=ssml) que facilitan la creación de SSML con un formato adecuado.

## <a name="display-cards-in-cortana"></a>Visualización de tarjetas en Cortana

Además de las respuestas de voz, Cortana puede mostrar datos adjuntos de tarjeta. Cortana es compatible con las siguientes tarjetas enriquecidas:
* [HeroCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.herocard.html)
* [ReceiptCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.receiptcard.html)
* [ThumbnailCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.thumbnailcard.html)

Consulte los [procedimientos recomendados para el diseño de tarjetas][CardDesign] para ver el diseño de estas tarjetas en Cortana. Para obtener un ejemplo de cómo agregar una tarjeta enriquecida a un bot, vea [Envío de tarjetas enriquecidas](bot-builder-nodejs-send-rich-cards.md). 

El código siguiente muestra cómo agregar las propiedades **speak** y **inputHint** a un mensaje que contiene una tarjeta Hero.

```javascript

bot.dialog('HelpDialog', function (session) {
    var card = new builder.HeroCard(session)
        .title('help_title')
        .buttons([
            builder.CardAction.imBack(session, 'roll some dice', 'Roll Dice'),
            builder.CardAction.imBack(session, 'play yahtzee', 'Play Yahtzee')
        ]);
    var msg = new builder.Message(session)
        .speak(speak(session, 'I\'m roller, the dice rolling bot. You can say \'roll some dice\''))
        .addAttachment(card)
        .inputHint(builder.InputHint.acceptingInput); // Tell Cortana to accept input
    session.send(msg).endDialog();
}).triggerAction({ matches: /help/i });

/** This helper function builds the required root element of a Speech Synthesis Markup Language (SSML) document. */
module.exports.speak = function (template, params, options) {
    options = options || {};
    var output = '<speak xmlns="http://www.w3.org/2001/10/synthesis" ' +
        'version="' + (options.version || '1.0') + '" ' +
        'xml:lang="' + (options.lang || 'en-US') + '">';
    output += module.exports.vsprintf(template, params);
    output += '</speak>';
    return output;
}

```
## <a name="sample-rollerskill"></a>Sample: RollerSkill
El código en las secciones siguientes procede de una habilidad de Cortana de ejemplo para lanzar los dados. Descargue el código completo para el bot desde el [repositorio de ejemplos de Bot Builder](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill).

La habilidad se invoca al decir el [nombre de invocación][InvocationNameGuidelines] a Cortana. Para la habilidad Roller, después de que [agregue el bot al canal de Cortana][CortanaChannel] y lo registre como una habilidad de Cortana, puede invocarlo si dice a Cortana “Pedir a Roller” o “Pedir a Roller que lance los dados”.

### <a name="explore-the-code"></a>Exploración del código

El ejemplo RollerSkill empieza con la apertura de una tarjeta mediante algunos botones para decir al usuario qué opciones tiene a su disposición.

```javascript

/**
 *   Create your bot with a default message handler that receive messages from the user.
 * - This function is be called anytime the user's utterance isn't
 *   recognized by the other handlers in the bot.
 */
var bot = new builder.UniversalBot(connector, function (session) {
    // Just redirect to our 'HelpDialog'.
    session.replaceDialog('HelpDialog');
});

//...

bot.dialog('HelpDialog', function (session) {
    var card = new builder.HeroCard(session)
        .title('help_title')
        .buttons([
            builder.CardAction.imBack(session, 'roll some dice', 'Roll Dice'),
            builder.CardAction.imBack(session, 'play craps', 'Play Craps')
        ]);
    var msg = new builder.Message(session)
        .speak(speak(session, 'help_ssml'))
        .addAttachment(card)
        .inputHint(builder.InputHint.acceptingInput);
    session.send(msg).endDialog();
}).triggerAction({ matches: /help/i });
```

### <a name="prompt-the-user-for-input"></a>Solicitud de entrada al usuario

El diálogo siguiente configura un juego personalizado para que el bot lo juegue.  Pregunta al usuario cuántos lados quiere que tengan los dados y cuántos quiere lanzar. Una vez creada la estructura del juego, la pasará a un "PlayGameDialog" independiente.

Para iniciar el diálogo, el controlador **triggerAction()** de este diálogo permite a un usuario decir algo como "Me gustaría lanzar algunos dados". Usa una expresión regular para que coincida con la entrada del usuario, pero podría usar fácilmente solo una [intención de LUIS](./bot-builder-nodejs-recognize-intent-luis.md). 


```javascript
bot.dialog('CreateGameDialog', [
    function (session) {
        // Initialize game structure.
        // - dialogData gives us temporary storage of this data in between
        //   turns with the user.
        var game = session.dialogData.game = { 
            type: 'custom', 
            sides: null, 
            count: null,
            turns: 0
        };

        var choices = [
            { value: '4', action: { title: '4 Sides' }, synonyms: 'four|for|4 sided|4 sides' },
            { value: '6', action: { title: '6 Sides' }, synonyms: 'six|sex|6 sided|6 sides' },
            { value: '8', action: { title: '8 Sides' }, synonyms: 'eight|8 sided|8 sides' },
            { value: '10', action: { title: '10 Sides' }, synonyms: 'ten|10 sided|10 sides' },
            { value: '12', action: { title: '12 Sides' }, synonyms: 'twelve|12 sided|12 sides' },
            { value: '20', action: { title: '20 Sides' }, synonyms: 'twenty|20 sided|20 sides' },
        ];
        builder.Prompts.choice(session, 'choose_sides', choices, { 
            speak: speak(session, 'choose_sides_ssml') 
        });
    },
    function (session, results) {
        // Store users input
        // - The response comes back as a find result with index & entity value matched.
        var game = session.dialogData.game;
        game.sides = Number(results.response.entity);

        /**
         * Ask for number of dice.
         */
        var prompt = session.gettext('choose_count', game.sides);
        builder.Prompts.number(session, prompt, {
            speak: speak(session, 'choose_count_ssml'),
            minValue: 1,
            maxValue: 100,
            integerOnly: true
        });
    },
    function (session, results) {
        // Store users input
        // - The response is already a number.
        var game = session.dialogData.game;
        game.count = results.response;

        /**
         * Play the game we just created.
         * 
         * replaceDialog() ends the current dialog and start a new
         * one in its place. We can pass arguments to dialogs so we'll pass the
         * 'PlayGameDialog' the game we created.
         */
        session.replaceDialog('PlayGameDialog', { game: game });
    }
]).triggerAction({ matches: [
    /(roll|role|throw|shoot).*(dice|die|dye|bones)/i,
    /new game/i
 ]});

```

### <a name="render-results"></a>Representación de resultados

 Este diálogo es nuestro bucle de juego principal. El bot almacena la estructura del juego en **session.conversationData**, de tal forma que, cuando el usuario diga "volver a lanzar", podremos volver a lanzar el mismo conjunto de dados.

```javascript

bot.dialog('PlayGameDialog', function (session, args) {
    // Get current or new game structure.
    var game = args.game || session.conversationData.game;
    if (game) {
        // Generate rolls
        var total = 0;
        var rolls = [];
        for (var i = 0; i < game.count; i++) {
            var roll = Math.floor(Math.random() * game.sides) + 1;
            if (roll > game.sides) {
                // Accounts for 1 in a million chance random() generated a 1.0
                roll = game.sides;
            }
            total += roll;
            rolls.push(roll);
        }

        // Format roll results
        var results = '';
        var multiLine = rolls.length > 5;
        for (var i = 0; i < rolls.length; i++) {
            if (i > 0) {
                results += ' . ';
            }
            results += rolls[i];
        }

        // Render results using a card
        var card = new builder.HeroCard(session)
            .subtitle(game.count > 1 ? 'card_subtitle_plural' : 'card_subtitle_singular', game)
            .buttons([
                builder.CardAction.imBack(session, 'roll again', 'Roll Again'),
                builder.CardAction.imBack(session, 'new game', 'New Game')
            ]);
        if (multiLine) {
            //card.title('card_title').text('\n\n' + results + '\n\n');
            card.text(results);
        } else {
            card.title(results);
        }
        var msg = new builder.Message(session).addAttachment(card);

        // Determine bots reaction for speech purposes
        var reaction = 'normal';
        var min = game.count;
        var max = game.count * game.sides;
        var score = total/max;
        if (score == 1.0) {
            reaction = 'best';
        } else if (score == 0) {
            reaction = 'worst';
        } else if (score <= 0.3) {
            reaction = 'bad';
        } else if (score >= 0.8) {
            reaction = 'good';
        }

        // Check for special craps rolls
        if (game.type == 'craps') {
            switch (total) {
                case 2:
                case 3:
                case 12:
                    reaction = 'craps_lose';
                    break;
                case 7:
                    reaction = 'craps_seven';
                    break;
                case 11:
                    reaction = 'craps_eleven';
                    break;
                default:
                    reaction = 'craps_retry';
                    break;
            }
        }

        // Build up spoken response
        var spoken = '';
        if (game.turn == 0) {
            spoken += session.gettext('start_' + game.type + '_game_ssml') + ' ';
        } 
        spoken += session.gettext(reaction + '_roll_reaction_ssml');
        msg.speak(ssml.speak(spoken));

        // Increment number of turns and store game to roll again
        game.turn++;
        session.conversationData.game = game;

        /**
         * Send card and bot's reaction to user. 
         */

        msg.inputHint(builder.InputHint.acceptingInput);
        session.send(msg).endDialog();
    } else {
        // User started session with "roll again" so let's just send them to
        // the 'CreateGameDialog'
        session.replaceDialog('CreateGameDialog');
    }
}).triggerAction({ matches: /(roll|role|throw|shoot) again/i });

```

## <a name="next-steps"></a>Pasos siguientes
Si su bot se ejecuta localmente o se implementa en la nube, puede invocarlo desde Cortana. Consulte [Prueba de una habilidad de Cortana](../bot-service-debug-cortana-skill.md) para conocer los pasos necesarios para probar sus habilidades de Cortana.


## <a name="additional-resources"></a>Recursos adicionales
* [Kit de habilidades de Cortana][CortanaGetStarted]
* [Incorporación de voz a mensajes](bot-builder-nodejs-text-to-speech.md)
* [Referencias de SSML][SSMLRef]
* [Procedimientos recomendados para el diseño de voz de Cortana][VoiceDesign]
* [Procedimientos recomendados para el diseño de tarjetas de Cortana][CardDesign]
* [Centro de desarrollo de Cortana][CortanaDevCenter]
* [Procedimientos recomendados para probar y depurar a Cortana][Cortana-TestBestPractice]


[CortanaGetStarted]: /cortana/getstarted
[BFPortal]: https://dev.botframework.com/


[SSMLRef]: https://aka.ms/cortana-ssml
[IMessage]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage.html
[Send]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#send
[CortanaDevCenter]: https://developer.microsoft.com/en-us/cortana

[CortanaSpecificEntities]: https://aka.ms/lgvcto
[CortanaAuth]: https://aka.ms/vsdqcj

[InvocationNameGuidelines]: https://aka.ms/cortana-invocation-guidelines
[VoiceDesign]: https://aka.ms/cortana-design-voice
[CardDesign]: https://aka.ms/cortana-design-card
[Cortana-Debug]: https://aka.ms/cortana-enable-debug
[Cortana-Publish]: https://aka.ms/cortana-publish


[CortanaTry]: https://aka.ms/try-cortana-bot
[CortanaChannel]: https://aka.ms/bot-cortana-channel
[Cortana-TestBestPractice]: https://aka.ms/cortana-test-best-practice

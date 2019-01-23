---
title: Creación de una habilidad de Cortana mediante .NET | Microsoft Docs
description: Obtenga información sobre los conceptos básicos para la creación de una habilidad de Cortana en Bot Framework SDK para .NET.
keywords: Bot Framework, habilidad de Cortana, voz,. NET, SDK, conceptos clave, conceptos básicos
author: DeniseMak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 98fc10a806a4c8d1a4d6563934d92b0e0cdbb771
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224780"
---
# <a name="build-a-speech-enabled-bot-with-cortana-skills"></a>Creación de un bot habilitado para la voz con habilidades de Cortana

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-cortana-skill.md)
> - [Node.js](../nodejs/bot-builder-nodejs-cortana-skill.md)


Bot Framework SDK para .NET le permite crear un bot habilitado para voz al conectarlo al canal de Cortana como una habilidad de Cortana. 


> [!TIP]
> Para más información sobre qué es una habilidad y qué puede hacer, vea [The Cortana Skills Kit][CortanaGetStarted] (El kit de habilidades de Cortana).

La creación de una habilidad de Cortana con Bot Framework requiere muy poco conocimiento específico de Cortana y consiste básicamente en compilar un bot. Una de las posibles diferencias principales respecto a otros bots que ha creado en el pasado es que Cortana tiene un componente visual y uno de audio. Para el componente visual, Cortana proporciona un área del lienzo para representar el contenido, como tarjetas. Para el componente de audio, proporcione texto o SSML en los mensajes del bot para que Cortana los lea al usuario, dándole una voz. 

> [!NOTE]
> Cortana está disponible en muchos dispositivos diferentes. Algunos tienen una pantalla mientras que otros, como los altavoces independientes, puede que no. Asegúrese de que su bot es capaz de actuar en ambos escenarios. Consulte las [entidades específicas de Cortana][CortanaSpecificEntities] para obtener información sobre cómo comprobar la información del dispositivo.

## <a name="adding-speech-to-your-bot"></a>Adición de voz al bot

Los mensajes de voz de su bot se representan en el lenguaje de marcado de síntesis de voz (SSML). Bot Framework SDK permite incluir SSML en las respuestas del bot para controlar lo que dice y lo que muestra.  También puede controlar el estado del micrófono de Cortana al especificar que el bot acepta, espera o ignora la interacción del usuario.

Establezca la propiedad `Speak` del objeto `IMessageActivity` para especificar el mensaje que Cortana dirá. Si se especifica texto sin formato, Cortana determinará cómo se pronuncian las palabras. 

```cs
Activity reply = activity.CreateReply("This is the text that Cortana displays."); 
reply.Speak = "This is the text that Cortana will say.";
```

Si desea más control sobre el timbre, el tono y énfasis, dele formato de [lenguaje de marcado de síntesis de voz (SSML)](http://www.w3.org/TR/speech-synthesis/) a la propiedad `Speak`.  

El ejemplo de código siguiente especifica que la palabra “texto” se debe pronunciar con una cantidad de énfasis moderada:
```cs
Activity reply = activity.CreateReply("This is the text that will be displayed.");
reply.Speak = "<speak version=\"1.0\" xmlns=\"http://www.w3.org/2001/10/synthesis\" xml:lang=\"en-US\">This is the <emphasis level=\"moderate\">text</emphasis> that will be spoken.</speak>";
```


La propiedad **InputHint** ayuda a indicar a Cortana si su bot está esperando una entrada del usuario. El valor predeterminado es **ExpectingInput** para un mensaje y **AcceptingInput** para otros tipos de respuestas.


| Valor | DESCRIPCIÓN |
|------|------|
| **AcceptingInput** | El bot está listo para recibir información de forma pasiva, pero no está esperando una respuesta. Cortana acepta la entrada del usuario si el usuario mantiene presionado el botón del micrófono.|
| **ExpectingInput** | Indica que el bot está esperando una respuesta del usuario de forma activa. Cortana escucha hasta que el usuario habla por el micrófono.  |
| **IgnoringInput** | Cortana está ignorando las entradas. Su bot puede enviar esta sugerencia si se está procesando activamente una solicitud e ignorará la entrada de los usuarios hasta que se complete la solicitud.  |

<!-- TODO: tip about time limit and batching -->

En este ejemplo se muestra cómo puede informar a Cortana que se espera la intervención del usuario. El micrófono quedará encendido.
```cs
// Add an InputHint to let Cortana know to expect user input
Activity reply = activity.CreateReply("This is the text that will be displayed."); 
reply.Speak = "This is the text that will be spoken.";
reply.InputHint = InputHints.ExpectingInput;
```



## <a name="display-cards-in-cortana"></a>Visualización de tarjetas en Cortana

Además de las respuestas de voz, Cortana puede mostrar datos adjuntos de tarjeta. Cortana es compatible con las siguientes tarjetas enriquecidas:

| Tipo de tarjeta | DESCRIPCIÓN |
|----|----|
| [HeroCard][heroCard] | Una tarjeta que normalmente contiene una sola imagen grande, uno o varios botones y texto. |
| [ThumbnailCard][thumbnailCard] | Una tarjeta que normalmente contiene una sola imagen miniatura, uno o varios botones, y texto. |
| [ReceiptCard][receiptCard] | Una tarjeta que permite que un bot proporcione un recibo al usuario. Normalmente, contiene la lista de elementos que se incluyen en el recibo, la información de impuestos y del total, y texto adicional. |
| [SignInCard][signinCard] | Una tarjeta que permite al bot solicitar que un usuario inicie sesión. Normalmente contiene texto y uno o más botones en los cuales el usuario puede hacer clic para comenzar el proceso de inicio de sesión. |


Consulte [Card design best practices][CardDesign] (Prácticas recomendadas para el diseño de tarjetas) para ver el diseño de estas tarjetas en Cortana. Para obtener un ejemplo de cómo usar una tarjeta enriquecida en un bot, consulte [Add rich card attachments to messages](bot-builder-dotnet-add-rich-card-attachments.md) (Adición de tarjetas enriquecidas a mensajes). 

<!--
The following code demonstrates how to add the `Speak` and `InputHint` properties to a message containing a `HeroCard`.
-->


## <a name="sample-rollerskill"></a>Sample: RollerSkill
El código en las secciones siguientes procede de una habilidad de Cortana de ejemplo para lanzar los dados. Descargue el código completo para el bot desde el [repositorio de ejemplos de Bot Builder](https://github.com/Microsoft/BotBuilder-Samples/).

La habilidad se invoca al decir el [nombre de invocación][InvocationNameGuidelines] a Cortana. Para la habilidad Roller, después de que [agregue el bot al canal de Cortana][CortanaChannel] y lo registre como una habilidad de Cortana, puede invocarlo si dice a Cortana “Pedir a Roller” o “Pedir a Roller que lance los dados”.

### <a name="explore-the-code"></a>Exploración del código



Para invocar los diálogos correspondientes, los controladores de actividad definidos en `RootDispatchDialog.cs` usan expresiones regulares para establecer coincidencias con la entrada del usuario. Por ejemplo, el controlador en el ejemplo siguiente se desencadena si el usuario dice algo como "Me gustaría lanzar los dados". Los sinónimos se incluyen en la expresión regular para que las expresiones parecidas desencadenen el diálogo.
```cs
        [RegexPattern("(roll|role|throw|shoot).*(dice|die|dye|bones)")]
        [RegexPattern("new game")]
        [ScorableGroup(1)]
        public async Task NewGame(IDialogContext context, IActivity activity)
        {
            context.Call(new CreateGameDialog(), AfterGameCreated);
        }
```

El diálogo `CreateGameDialog` configura un juego personalizado para que el bot lo reproduzca. Usa `PromptDialog` para preguntar al usuario cuántos lados quiere que tengan los dados y cuántos quiere lanzar. Tenga en cuenta que el objeto `PromptOptions` que se usa para inicializar el mensaje contiene una propiedad `speak` para la versión oral del mensaje.

```cs
    [Serializable]
    public class CreateGameDialog : IDialog<GameData>
    {
        public async Task StartAsync(IDialogContext context)
        {
            context.UserData.SetValue<GameData>(Utils.GameDataKey, new GameData());

            var descriptions = new List<string>() { "4 Sides", "6 Sides", "8 Sides", "10 Sides", "12 Sides", "20 Sides" };
            var choices = new Dictionary<string, IReadOnlyList<string>>()
             {
                { "4", new List<string> { "four", "for", "4 sided", "4 sides" } },
                { "6", new List<string> { "six", "sex", "6 sided", "6 sides" } },
                { "8", new List<string> { "eight", "8 sided", "8 sides" } },
                { "10", new List<string> { "ten", "10 sided", "10 sides" } },
                { "12", new List<string> { "twelve", "12 sided", "12 sides" } },
                { "20", new List<string> { "twenty", "20 sided", "20 sides" } }
            };

            var promptOptions = new PromptOptions<string>(
                Resources.ChooseSides,
                choices: choices,
                descriptions: descriptions,
                speak: SSMLHelper.Speak(Utils.RandomPick(Resources.ChooseSidesSSML))); // spoken prompt

            PromptDialog.Choice(context, this.DiceChoiceReceivedAsync, promptOptions);
        }

        private async Task DiceChoiceReceivedAsync(IDialogContext context, IAwaitable<string> result)
        {
            GameData game;
            if (context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out game))
            {
                int sides;
                if (int.TryParse(await result, out sides))
                {
                    game.Sides = sides;
                    context.UserData.SetValue<GameData>(Utils.GameDataKey, game);
                }

                var promptText = string.Format(Resources.ChooseCount, sides);

                var promptOption = new PromptOptions<long>(promptText, choices: null, speak: SSMLHelper.Speak(Utils.RandomPick(Resources.ChooseCountSSML)));

                var prompt = new PromptDialog.PromptInt64(promptOption);
                context.Call<long>(prompt, this.DiceNumberReceivedAsync);
            }
        }

        private async Task DiceNumberReceivedAsync(IDialogContext context, IAwaitable<long> result)
        {
            GameData game;
            if (context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out game))
            {
                game.Count = await result;
                context.UserData.SetValue<GameData>(Utils.GameDataKey, game);
            }

            context.Done(game);
        }
    }
```

`PlayGameDialog` representa los resultados, los muestra en `HeroCard` y crea un mensaje de voz para decirlo con el método `Speak`.

```cs
   [Serializable]
    public class PlayGameDialog : IDialog<object>
    {
        private const string RollAgainOptionValue = "roll again";

        private const string NewGameOptionValue = "new game";

        private GameData gameData;

        public PlayGameDialog(GameData gameData)
        {
            this.gameData = gameData;
        }

        public async Task StartAsync(IDialogContext context)
        {
            if (this.gameData == null)
            {
                if (!context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out this.gameData))
                {
                    // User started session with "roll again" so let's just send them to
                    // the 'CreateGameDialog'
                    context.Done<object>(null);
                }
            }

            int total = 0;
            var randomGenerator = new Random();
            var rolls = new List<int>();

            // Generate Rolls
            for (int i = 0; i < this.gameData.Count; i++)
            {
                var roll = randomGenerator.Next(1, this.gameData.Sides);
                total += roll;
                rolls.Add(roll);
            }

            // Format rolls results
            var result = string.Join(" . ", rolls.ToArray());
            bool multiLine = rolls.Count > 5;

            var card = new HeroCard()
            {
                Subtitle = string.Format(
                    this.gameData.Count > 1 ? Resources.CardSubtitlePlural : Resources.CardSubtitleSingular,
                    this.gameData.Count,
                    this.gameData.Sides),
                Buttons = new List<CardAction>()
                {
                    new CardAction(ActionTypes.ImBack, "Roll Again", value: RollAgainOptionValue),
                    new CardAction(ActionTypes.ImBack, "New Game", value: NewGameOptionValue)
                }
            };

            if (multiLine)
            {
                card.Text = result;
            }
            else
            {
                card.Title = result;
            }

            var message = context.MakeMessage();
            message.Attachments = new List<Attachment>()
            {
                card.ToAttachment()
            };

            // Determine bots reaction for speech purposes
            string reaction = "normal";

            var min = this.gameData.Count;
            var max = this.gameData.Count * this.gameData.Sides;
            var score = total / max;
            if (score == 1)
            {
                reaction = "Best";
            }
            else if (score == 0)
            {
                reaction = "Worst";
            }
            else if (score <= 0.3)
            {
                reaction = "Bad";
            }
            else if (score >= 0.8)
            {
                reaction = "Good";
            }

            // Check for special craps rolls
            if (this.gameData.Type == "Craps")
            {
                switch (total)
                {
                    case 2:
                    case 3:
                    case 12:
                        reaction = "CrapsLose";
                        break;
                    case 7:
                        reaction = "CrapsSeven";
                        break;
                    case 11:
                        reaction = "CrapsEleven";
                        break;
                    default:
                        reaction = "CrapsRetry";
                        break;
                }
            }

            // Build up spoken response
            var spoken = string.Empty;
            if (this.gameData.Turns == 0)
            {
                spoken += Utils.RandomPick(Resources.ResourceManager.GetString($"Start{this.gameData.Type}GameSSML"));
            }

            spoken += Utils.RandomPick(Resources.ResourceManager.GetString($"{reaction}RollReactionSSML"));

            message.Speak = SSMLHelper.Speak(spoken);

            // Increment number of turns and store game to roll again
            this.gameData.Turns++;
            context.UserData.SetValue<GameData>(Utils.GameDataKey, this.gameData);

            // Send card and bots reaction to user.
            message.InputHint = InputHints.AcceptingInput;
            await context.PostAsync(message);

            context.Done<object>(null);
        }
    }
```
## <a name="next-steps"></a>Pasos siguientes

Si su bot se ejecuta localmente o se implementa en la nube, puede invocarlo desde Cortana. Consulte [Test a Cortana skill](../bot-service-debug-cortana-skill.md) (Prueba de una habilidad de Cortana) para conocer los pasos necesarios para probar su habilidad de Cortana.


## <a name="additional-resources"></a>Recursos adicionales
* [Kit de habilidades de Cortana][CortanaGetStarted]
* [Incorporación de voz a mensajes](bot-builder-dotnet-text-to-speech.md)
* [Referencias de SSML][SSMLRef]
* [Procedimientos recomendados para el diseño de voz de Cortana][VoiceDesign]
* [Procedimientos recomendados para el diseño de tarjetas de Cortana][CardDesign]
* [Centro de desarrollo de Cortana][CortanaDevCenter]
* [Procedimientos recomendados para probar y depurar a Cortana][Cortana-TestBestPractice]
* <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Referencia de Bot Framework SDK para .NET</a>

[CortanaGetStarted]: /cortana/getstarted
[BFPortal]: https://dev.botframework.com/

[SSMLRef]: https://aka.ms/cortana-ssml
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

[heroCard]: /dotnet/api/microsoft.bot.connector.herocard

[thumbnailCard]: /dotnet/api/microsoft.bot.connector.thumbnailcard 

[receiptCard]: /dotnet/api/microsoft.bot.connector.receiptcard 

[signinCard]: /dotnet/api/microsoft.bot.connector.signincard 



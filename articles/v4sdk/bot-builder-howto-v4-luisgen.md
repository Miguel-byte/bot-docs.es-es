---
title: Extraer resultados LUIS escritos | Microsoft Docs
description: Obtenga información sobre cómo se usa LUIS para extraer entidades con el SDK de Bot Builder.
keywords: intenciones, entidades, LUISGen, extraer
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 5/16/17
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f0e428ca0aa1b0208538e2de7fc0a293763062a1
ms.sourcegitcommit: 54ed5000c67a5b59e23b667547565dd96c7302f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/13/2018
ms.locfileid: "49315211"
---
# <a name="extract-intents-and-entities-using-luisgen"></a>Extraer intenciones y entidades mediante LUISGen

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Además de reconocer la intención, una aplicación de LUIS puede extraer entidades (es decir, palabras importantes) para cumplir la solicitud de un usuario. Por ejemplo, en una reserva de restaurante, la aplicación LUIS puede extraer el tamaño del grupo, la fecha de la reserva o la ubicación del restaurante del mensaje del usuario. 


Puede usar la [herramienta LUISGen](https://aka.ms/botbuilder-tools-luisgen) para crear clases que faciliten la extracción de las entidades de LUIS en el código del bot.

En una línea de comandos de Node.js, instale `luisgen` en la ruta de acceso global.
```
npm install -g luisgen
```

# <a name="ctabcs"></a>[C#](#tab/cs)

## <a name="generate-a-luis-results-class"></a>Generar una clase de resultados de LUIS

Descargue el [ejemplo CafeBot LUIS](https://aka.ms/contosocafebot-luis) y, en su carpeta raíz, ejecute LUISGen:

```
luisgen Assets\LU\models\LUIS\cafeLUISModel.json -cs ContosoCafeBot.CafeLUISModel
```

## <a name="examine-the-generated-code"></a>Examinar el código generado
Esto genera **cafeLUISModel.cs**, que puede agregar al proyecto. Proporciona una clase `cafeLuisModel` para obtener resultados fuertemente tipados de LUIS.

Esta clase tiene un valor enum para obtener las intenciones definidas en la aplicación LUIS.
```cs
public enum Intent {
    Book_Table, 
    Greeting, 
    None, 
    Who_are_you_intent
};
```
Asimismo, también tiene una propiedad `Entities`. Dado que puede haber varias repeticiones de una entidad en el mensaje de un usuario, la clase `_Entities` define una matriz para cada tipo de entidad. 
```cs
public class _Entities
{
    // Simple entities
    public string[] partySize;

    // Built-in entities
    public Microsoft.Bot.Builder.Ai.Luis.DateTimeSpec[] datetime;
    public double[] number;

    // Lists
    public string[][] cafeLocation;

    // Instance
    public class _Instance
    {
        public Microsoft.Bot.Builder.Ai.Luis.InstanceData[] partySize;
        public Microsoft.Bot.Builder.Ai.Luis.InstanceData[] datetime;
        public Microsoft.Bot.Builder.Ai.Luis.InstanceData[] number;
        public Microsoft.Bot.Builder.Ai.Luis.InstanceData[] cafeLocation;
    }
    [JsonProperty("$instance")]
    public _Instance _instance;
}
public _Entities Entities;
```

> [!NOTE]
> Todos los tipos de entidades son matrices, ya que LUIS puede detectar más de una entidad de un tipo especificado en la declaración de un usuario. Por ejemplo, si el usuario dice "hacer reservas para mañana a las 5 de la tarde, y a las 9 de la noche el sábado siguiente", "mañana a las 5 de la tarde" y "a las 9 de la tarde el sábado siguiente" se devuelven en los resultados `datetime`.
>

|Entidad | Escriba | Ejemplo | Notas |
|-------|-----|------|---|
|partySize| string[]| Grupo de `four`.| Una entidad sencilla reconoce las cadenas. En este ejemplo, Entities.partySize[0] es `"four"`.
|Datetime| [DateTimeSpec](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.datetimespec?view=botbuilder-4.0.0-alpha)[]| realizar una reserva para el `9pm tomorrow`.| Cada objeto **DateTimeSpec** tiene un campo timex con el valor posible de horas especificadas en formato **timex**. Puede obtener más información sobre timex aquí: http://www.timeml.org/publications/timeMLdocs/timeml_1.2.1.html#timex3      Puede obtener más información sobre la biblioteca que realiza el reconocimiento aquí: https://github.com/Microsoft/Recognizers-Text.
|número| double[]| Un grupo de `four` que incluye `2` niños. | `number` identificará todos los números, no sólo de tamaño del grupo. <br/> En la declaración "El grupo de cuatro incluye 2 niños" `Entities.number[0]` es 4, y `Entities.number[1]` es 2.
|cafelocation| string[][] | Reserva en la ubicación `Seattle`.| cafeLocation es una entidad de la lista, lo que significa que contiene miembros reconocidos de listas. Es una matriz de matrices, en caso de que una entidad reconocida sea miembro de más de una lista. Por ejemplo, "reserva en Washington" puede corresponder a una lista que hace referencia al estado de Washington y a Washington D.C.

La propiedad `_Instance` proporciona [InstanceData](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.instancedata?view=botbuilder-4.0.0-alpha) para ofrecer más detalles en cada entidad reconocida.

## <a name="check-intents-in-your-bot"></a>Comprobar las intenciones en el bot
En **CafeBot.cs**, eche un vistazo al código de `OnTurn`. Puede ver que el bot llama a LUIS y comprueba las intenciones para decidir con qué cuadro de diálogo debe comenzar. Los resultados de LUIS de la llamada a [`Recognize`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.luisrecognizer?view=botbuilder-4.0.0-alpha#methods) se pasan como un argumento al diálogo `BookTable`.



```cs
if(!context.Responded)
{
    // call LUIS and get results
    LuisRecognizer lRecognizer = createLUISRecognizer();
    // Use the generated class as the type parameter to Recognize()
    cafeLUISModel lResult = await lRecognizer.Recognize<cafeLUISModel>(utterance, ct);
    Dictionary<string,object> lD = new Dictionary<string,object>();
    if(lResult != null) {
        lD.Add("luisResult", lResult);
    }
    
    // top level dispatch
    switch (lResult.TopIntent().intent)
    {
        case cafeLUISModel.Intent.Greeting:
            await context.SendActivity("Hello!");
            if (userState.sendCards) await context.SendActivity(CreateCardResponse(context.Activity, createWelcomeCardAttachment()));
            break;

        case cafeLUISModel.Intent.Book_Table:
            await dc.Begin("BookTable", lD);
            break;

        case cafeLUISModel.Intent.Who_are_you_intent:
            await context.sendActivity("I'm the Contoso Cafe bot.");
            break;

        case cafeLUISModel.Intent.None:
        default:
            await getQnAResult(context);
            break;
    }
}
```

## <a name="extract-entities-in-a-dialog"></a>Extraer entidades de un diálogo

Eche un vistazo a `Dialogs/BookTable.cs`. El cuadro de diálogo `BookTable` contiene una secuencia de pasos en cascada, cada uno de los cuales busca una entidad en los resultados de LUIS que se pasaron a `args`. Si no se encuentra la entidad, el cuadro de diálogo se salta la solicitud llamando a `next()`. Si se encuentra, el cuadro de diálogo lo solicita y la respuesta del usuario a la solicitud se recibe en el siguiente paso en cascada.

```cs
    Dialogs.Add("BookTable",
        new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                dc.ActiveDialog.State = new Dictionary<string, object>();
                IDictionary<string,object> state = dc.ActiveDialog.State;

                // add any LUIS entities to active dialog state.
                if(args.ContainsKey("luisResult")) {
                    cafeLUISModel lResult = (cafeLUISModel)args["luisResult"];
                    updateContextWithLUIS(lResult, ref state);
                }
                
                // prompt if we do not already have cafelocation
                if(state.ContainsKey("cafeLocation")) {
                    state["bookingLocation"] = state["cafeLocation"];
                    await next();
                } else {
                    await dc.Prompt("choicePrompt", "Which of our locations would you like?", promptOptions);
                }
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("cafeLocation")) {
                    var choiceResult = (FoundChoice)args["Value"];
                    state["bookingLocation"] = choiceResult.Value;
                }
                bool promptForDateTime = true;
                if(state.ContainsKey("datetime")) {
                    // validate timex
                    var inputdatetime = new string[] {(string)state["datetime"]};
                    var results = evaluateTimeX((string[])inputdatetime);
                    if(results.Count != 0) {
                        var timexResolution = results.First().TimexValue;
                        var timexProperty = new TimexProperty(timexResolution.ToString());
                        var bookingDateTime = $"{timexProperty.ToNaturalLanguage(DateTime.Now)}";
                        state["bookingDateTime"] = bookingDateTime;
                        promptForDateTime = false;
                    }
                }
                // prompt if we do not already have date and time
                if(promptForDateTime) {
                    await dc.Prompt("timexPrompt", "When would you like to arrive? (We open at 4PM.)",
                                    new PromptOptions { RetryPromptString = "We only accept reservations for the next 2 weeks and in the evenings between 4PM - 8PM" });
                } else {
                    await next();
                }                       
                
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("datetime")) { 
                    var timexResult = (TimexResult)args;
                    var timexResolution = timexResult.Resolutions.First();
                    var timexProperty = new TimexProperty(timexResolution.ToString());
                    var bookingDateTime = $"{timexProperty.ToNaturalLanguage(DateTime.Now)}";
                    state["bookingDateTime"] = bookingDateTime;
                }
                // prompt if we already do not have party size
                if(state.ContainsKey("partySize")) {
                    state["bookingGuestCount"] = state["partySize"];
                    await next();
                } else {
                    await dc.Prompt("numberPrompt", "How many in your party?");
                }
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("partySize")) {
                    state["bookingGuestCount"] = args["Value"];
                }

                await dc.Prompt("confirmationPrompt", $"Thanks, Should I go ahead and book a table for {state["bookingGuestCount"].ToString()} guests at our {state["bookingLocation"].ToString()} location for {state["bookingDateTime"].ToString()} ?");
            },
            async (dc, args, next) =>
            {
                var dialogState = dc.ActiveDialog.State;

                // TODO: Verify user said yes to confirmation prompt

                // TODO: book the table! 

                await dc.Context.SendActivity($"Thanks, I have {dialogState["bookingGuestCount"].ToString()} guests booked for our {dialogState["bookingLocation"].ToString()} location for {dialogState["bookingDateTime"].ToString()}.");
            }
        }
    );
}

// This helper method updates dialog state with any LUIS results
private void updateContextWithLUIS(cafeLUISModel lResult, ref IDictionary<string,object> dialogContext) {
    if(lResult.Entities.cafeLocation != null && lResult.Entities.cafeLocation.GetLength(0) > 0) {
        dialogContext.Add("cafeLocation", lResult.Entities.cafeLocation[0][0]);
    }
    if(lResult.Entities.partySize != null && lResult.Entities.partySize.GetLength(0) > 0) {
        dialogContext.Add("partySize", lResult.Entities.partySize[0][0]);
    } else {
        if(lResult.Entities.number != null && lResult.Entities.number.GetLength(0) > 0) {
            dialogContext.Add("partySize", lResult.Entities.number[0]);
        }
    }
    if(lResult.Entities.datetime != null && lResult.Entities.datetime.GetLength(0) > 0) {
        dialogContext.Add("datetime", lResult.Entities.datetime[0].Expressions[0]);
    }
}
```
## <a name="run-the-sample"></a>Ejecución del ejemplo

Abra `ContosoCafeBot.sln` en Visual Studio 2017 y ejecute el bot. Use [Bot Framework Emulator](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator) para conectarse al bot de muestra.

En el emulador, diga `reserve a table` para iniciar el cuadro de diálogo de reserva.

![ejecute el bot](media/how-to-luisgen/run-bot.png)

# <a name="typescripttabjs"></a>[TypeScript](#tab/js)

Descargue el [ejemplo CafeBot_LUIS](https://aka.ms/contosocafebot-typescript-luis-dialogs) y, en su carpeta raíz, ejecute LUISGen:

```
luisgen cafeLUISModel.json -ts CafeLUISModel
```

Esto genera **CafeLUISModel.ts**, que puede agregar al proyecto. Puede obtener un resultado escrito del reconocedor LUIS utilizando los tipos del archivo generado.


```typescript
// call LUIS and get typed results
await luisRec.recognize(context).then(async (res : any) => 
{    
    // get a typed result
    var typedresult = res as CafeLUISModel;   
    
```

## <a name="pass-the-typed-result-to-a-dialog"></a>Pasar el resultado escrito a un cuadro de diálogo

Examine el código en **luisbot.ts**. En el controlador `processActivity`, el bot pasa el resultado escrito a un cuadro de diálogo.

```typescript
// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = context.activity.type === 'message';

        // Create dialog context 
        const state = conversationState.get(context);
        const dc = dialogs.createContext(context, state);
            
        if (!isMessage) {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }

        // Check to see if anyone replied. 
        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {

                
                await luisRec.recognize(context).then(async (res : any) => 
                {    
                    var typedresult = res as CafeLUISModel;                
                    let topIntent = LuisRecognizer.topIntent(res);    
                    switch (topIntent)
                    {
                        case Intents.Book_Table: {
                            await context.sendActivity("Top intent is Book_Table ");                          
                            await dc.begin('reserveTable', typedresult);
                            break;
                        }
                        
                        case Intents.Greeting: {
                            await context.sendActivity("Top intent is Greeting");
                            break;
                        }
    
                        case Intents.Who_are_you_intent: {
                            await context.sendActivity("Top intent is Who_are_you_intent");
                            break;
                        }
                        default: {
                            await dc.begin('default', topIntent);
                            break;
                        }
                    }
    
                }, (err) => {
                    // there was some error
                    console.log(err);
                }
                );                                
            }
        }
    });
});
```

## <a name="check-for-existing-entities-in-a-dialog"></a>Buscar las entidades existentes en un cuadro de diálogo

En **luisbot.ts**, el diálogo `reserveTable` llama a una función auxiliar `SaveEntities` para comprobar las entidades que detectó la aplicación LUIS. Si se encuentran las entidades, se guardan en el estado del diálogo. Cada paso en cascada del diálogo comprueba si una entidad se guardó en el estado del diálogo; si no es así, lo solicita.

```typescript
dialogs.add('reserveTable', [
    async function(dc, args, next){
        var typedresult = args as CafeLUISModel;

        // Call a helper function to save the entities in the LUIS result
        // to dialog state
        await SaveEntities(dc, typedresult);

        await dc.context.sendActivity("Welcome to the reservation service.");
        
        if (dc.activeDialog.state.dateTime) {
            await next();     
        }
        else {
            await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
        }
    },
    async function(dc, result, next){
        if (!dc.activeDialog.state.dateTime) {
            // Save the dateTimePrompt result to dialog state
            dc.activeDialog.state.dateTime = result[0].value;
        }

        // If we don't have party size, ask for it next
        if (!dc.activeDialog.state.partySize) {
            await dc.prompt('textPrompt', "How many people are in your party?");
        } else {
            await next();
        }
    },
    async function(dc, result, next){
        if (!dc.activeDialog.state.partySize) {
            dc.activeDialog.state.partySize = result;
        }
        // Ask for the reservation name next
        await dc.prompt('textPrompt', "Whose name will this be under?");
    },
    async function(dc, result){
        dc.activeDialog.state.Name = result;

        // Save data to conversation state
        var state = conversationState.get(dc.context);

        // Copy the dialog state to the conversation state
        state = dc.activeDialog.state;

        // TODO: Add in <br/>Location: ${state.cafeLocation}
        var msg = `Reservation confirmed. Reservation details:             
            <br/>Date/Time: ${state.dateTime} 
            <br/>Party size: ${state.partySize} 
            <br/>Reservation name: ${state.Name}`;
            
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

La función de ayuda `SaveEntities` busca entidades `datetime` y `partysize`. La entidad `datetime` es una [entidad precompilada](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2).

```typescript
// Helper function that saves any entities found in the LUIS result
// to the dialog state
async function SaveEntities( dc: DialogContext<TurnContext>, typedresult) {
    // Resolve entities returned from LUIS, and save these to state
    if (typedresult.entities)
    {
        console.log(`Entities found.`);
        let datetime = typedresult.entities.datetime;

        if (datetime) {
            // Use the first date or time found in the utterance
            if (datetime[0].timex) {
                timexValues = datetime[0].timex;
                // Datetime results from LUIS are represented in timex format:
                // http://www.timeml.org/publications/timeMLdocs/timeml_1.2.1.html#timex3                                
                // More information on the library which does the recognition can be found here: 
                // https://github.com/Microsoft/Recognizers-Text

                if (datetime[0].type === "datetime") {
                    // To parse timex, here you use the resolve and creator from
                    // @microsoft/recognizers-text-data-types-timex-expression
                    // The second parameter is an array of constraints the results must satisfy
                    var resolution = Resolver.evaluate(
                        // array of timex values to evaluate. There may be more than one: "today at 6" can be 6AM or 6PM.
                        timexValues,
                        // constrain results to times between 4pm and 8pm                        
                        [Creator.evening]);
                    if (resolution[0]) {
                        // toNaturalLanguage takes the current date into account to create a friendly string
                        dc.activeDialog.state.dateTime = resolution[0].toNaturalLanguage(new Date());
                        // You can also use resolution.toString() to format the date/time.
                    } else {
                        // time didn't satisfy constraint.
                        dc.activeDialog.state.dateTime = null;
                    }
                } 
                else  {
                    console.log(`Type ${datetime[0].type} is not yet supported. Provide both the date and the time.`);
                }
            }                                                
        }
        let partysize = typedresult.entities.partySize;
        if (partysize) {
            console.log(`partysize entity detected.${partysize}`);
            // use first partySize entity that was found in utterance
            dc.activeDialog.state.partySize = partysize[0];
        }
        let cafelocation = typedresult.entities.cafeLocation;

        if (cafelocation) {
            console.log(`location entity detected.${cafelocation}`);
            // use first cafeLocation entity that was found in utterance
            dc.activeDialog.state.cafeLocation = cafelocation[0][0];
        }
    } 
}
```

## <a name="run-the-sample"></a>Ejecución del ejemplo

1. Si no tiene instalado el compilador de TypeScript, instálelo con este comando:

```
npm install --global typescript
```

2. Instale las dependencias antes de ejecutar el bot; para ello, ejecute `npm install` en el directorio raíz de la muestra:

```
npm install
```

3. Desde el directorio raíz, compile la muestra usando `tsc`. Esto generará `luisbot.js`.

4. Ejecute `luisbot.js` en el directorio `lib`.

5. Use [Bot Framework Emulator](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator) para ejecutar la muestra.

6. En el emulador, diga `reserve a table` para iniciar el cuadro de diálogo de reserva.

![ejecutar el bot](media/how-to-luisgen/run-bot.png)

---


## <a name="additional-resources"></a>Recursos adicionales

Para más información sobre LUIS, consulte [Language Understanding](./bot-builder-concept-luis.md).


## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Combinar LUIS y QnA con la herramienta de distribución](./bot-builder-tutorial-dispatch.md)



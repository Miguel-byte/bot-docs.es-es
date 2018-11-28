---
redirect_url: /bot-framework/bot-builder-prompts
ms.openlocfilehash: e7cfbad19290b3ef61d40dc90493db8f530a9a4e
ms.sourcegitcommit: 4661b9bb31d74731dbbb16e625be088b44ba5899
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/16/2018
ms.locfileid: "51826942"
---
<a name="--"></a><!--
---
title: Formulación de preguntas al usuario | Microsoft Docs description: Aprenda a usar el modelo de cascada para solicitar al usuario varias entradas en Bot Builder SDK.
keywords: waterfalls, dialogs, asking a question, prompts author: v-ducvo ms.author: v-ducvo manager: kamrani ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date: 5/10/2018 monikerRange: 'azure-bot-service-4.0'
---

# <a name="ask-the-user-questions"></a>Formulación de preguntas al usuario

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

En esencia, un bot se basa en la conversación con un usuario. La conversación puede adoptar [muchas formas](bot-builder-conversations.md); puede que sea corta o puede ser más compleja, puede ser para formular preguntas o para responder a estas. La forma que adopte la conversación depende de varios factores, pero todos ellos implican una conversación.

Este tutorial le guía por el proceso de creación de una conversación, desde formular una pregunta sencilla hasta la creación de un bot con varios turnos. Nuestro ejemplo trata de la reserva de una mesa para cenar, pero puede imaginarse un bot que haga varias cosas mediante una conversación con varios turnos como, por ejemplo, realizar un pedido, responder preguntas frecuentes, realizar reservas, etc.

Un bot de chat interactivo puede responder a una entrada del usuario o preguntarle a este una entrada específica. Este tutorial le mostrará cómo formular al usuario una pregunta mediante la biblioteca `Prompts` que forma parte de `Dialogs`. Se puede considerar a los [diálogos](../bot-service-design-conversation-flow.md) como el contenedor que define la estructura de una conversación. Los avisos que van incluidos en los diálogos se tratan con mayor detalle en su propio [artículo de procedimientos](bot-builder-prompts.md).

## <a name="prerequisite"></a>Requisito previo

El código de este tutorial se compilará sobre el **bot básico** que creó en la experiencia de [introducción](~/bot-service-quickstart.md).

## <a name="get-the-package"></a>Obtención del paquete

# <a name="ctabcstab"></a>[C#](#tab/cstab)

Instale el paquete **Microsoft.Bot.Builder.Dialogs** del administrador de paquetes NuGet.

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)
Vaya a la carpeta de proyecto del bot e instale el paquete `botbuilder-dialogs` de NPM:

```cmd
npm install --save botbuilder-dialogs@preview
```

---

## <a name="import-package-to-bot"></a>Importación del paquete al bot

# <a name="ctabcstab"></a>[C#](#tab/cstab)

Agregue referencias a diálogos y avisos en el código del bot.

```cs
// ...
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts;
// ...
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Abra el archivo **app.js** e incluya la biblioteca `botbuilder-dialogs` en el código de bot.

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

Esto le dará acceso a la biblioteca `DialogSet` y `Prompts` que va a utilizar para formular las preguntas al usuario. `DialogSet` es solo una colección de diálogos que se estructura en un patrón de **cascada**. Esto significa simplemente que un diálogo sigue a otro.

## <a name="instantiate-a-dialogs-object"></a>Creación de una instancia de un objeto de diálogos

Cree una instancia de un objeto `dialogs`. Usará este objeto de diálogo para administrar el proceso de pregunta y respuesta.

# <a name="ctabcstab"></a>[C#](#tab/cstab)
Declare una variable de miembro en la clase de bot e inicialícela en el constructor del bot. 
```cs
public class MyBot : IBot
{
    private readonly DialogSet dialogs;
    public MyBot()
    {
        dialogs = new DialogSet();
    }
    // The rest of the class definition is omitted here
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```
---

## <a name="define-a-waterfall-dialog"></a>Definición de un diálogo en cascada

Para formular una pregunta, necesitará al menos un diálogo en **cascada** con dos pasos. En este ejemplo, va a construir un diálogo en **cascada** de dos pasos en el que el primer paso pide al usuario su nombre y el segundo paso saluda al usuario por su nombre. 

# <a name="ctabcstab"></a>[C#](#tab/cstab)

Modifique el constructor del bot para agregar el diálogo:
```csharp
public MyBot()
{
    dialogs = new DialogSet();
    dialogs.Add("greetings", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Prompt("textPrompt","What is your name?");
        },
        async(dc, args, next) =>
        {
            await dc.Context.SendActivity($"Hi {args["Text"]}!");
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, results){
        var userName = results;
        await dc.context.sendActivity(`Hello ${userName}!`);
        await dc.end(); // Ends the dialog
    }
]);
```

---

La pregunta se formula mediante un método `textPrompt` que venía con la biblioteca `Prompts`. La biblioteca `Prompts` ofrece un conjunto de avisos que permiten pedir a los usuarios diversos tipos de información. Para más información sobre otros tipos de avisos, consulte [Petición de datos de entrada al usuario](~/v4sdk/bot-builder-prompts.md).

Para que los avisos funcionen, deberá agregar un aviso al objeto `dialogs` con el identificador de diálogo `textPrompt` y crearlo con el constructor `TextPrompt()`.

# <a name="ctabcstab"></a>[C#](#tab/cstab)

```cs
public MyBot()
{
    dialogs = new DialogSet();
    dialogs.Add("greetings", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Prompt("textPrompt","What is your name?");
        },
        async(dc, args, next) =>
        {
            await dc.Context.SendActivity($"Hi {args["Text"]}!");
            await dc.End();
        }
    });
    // add the prompt, of type TextPrompt
    dialogs.Add("textPrompt", new Builder.Dialogs.TextPrompt());
}

```
Una vez que el usuario responda la pregunta, la respuesta se puede encontrar en el parámetro `args` del paso 2.

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
```
Una vez que el usuario responda la pregunta, la respuesta se puede encontrar en el parámetro `results` del paso 2. En este caso, `results` se asigna a una variable local `userName`. En otro tutorial posterior le mostraremos cómo conservar las entradas de usuario en un almacenamiento de su elección.

---


Ahora que ha definido `dialogs` para formular una pregunta, debe llamar al diálogo para iniciar el proceso de aviso.

## <a name="start-the-dialog"></a>Inicio del diálogo

# <a name="ctabcstab"></a>[C#](#tab/cstab)

Modifique la lógica del bot a algo parecido a esto:

```cs

public async Task OnTurn(ITurnContext context)
{
    // We'll cover state later, in the next tutorial
    var state = ConversationState<Dictionary<string, object>>.Get(context);
    var dc = dialogs.CreateContext(context, state);
    if (context.Activity.Type == ActivityTypes.Message)
    {
        await dc.Continue();
        
        if(!context.Responded)
        {
            await dc.Begin("greetings");
        }
    }
}
```

La lógica del bot está en el método `OnTurn()`. Una vez que el usuario dice "Hola", el bot iniciará el diálogo `greetings`. El primer paso del diálogo `greetings` pide al usuario su nombre. El usuario enviará una respuesta con su nombre como una actividad de mensaje y el resultado se enviará al paso dos de la cascada mediante el método `dc.Continue()`. El segundo paso de la cascada, tal y como lo ha definido, saludará a los usuarios por su nombre y finalizará el diálogo. 

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Modifique el método `processActivity()` del bot **básico** para que tenga este aspecto:

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);
        const dc = dialogs.createContext(context, convo);

        if (isMessage) {
            // Check for valid intents
            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greetings');
            }
        }

        if(!context.responded){
            // Continue executing the "current" dialog, if any.
            await dc.continue();

            if(!context.responded && isMessage){
                // Default message
                await context.sendActivity("Hi! I'm a simple bot. Please say 'Hi'.");
            }
        }
    });
});
```

La lógica del bot está en el método `processActivity()`. Una vez que el usuario dice "Hola", el bot iniciará el diálogo `greetings`. El primer paso del diálogo `greetings` pide al usuario su nombre. El usuario le enviará una respuesta con su nombre como mensaje `text` de la actividad. Como el mensaje no coincidía con las intenciones esperadas y el bot no ha enviado ninguna respuesta al usuario, el resultado se envía al paso dos de la cascada mediante el método `dc.continue()`. El segundo paso de la cascada, tal y como lo ha definido, saludará a los usuarios por su nombre y finalizará el diálogo. Si, por ejemplo, el paso dos no envió al usuario el mensaje de saludo, el método `processActivity` finalizará con el envío del *mensaje predeterminado* al usuario.

---



## <a name="define-a-more-complex-waterfall-dialog"></a>Definición de un cuadro de diálogo en cascada más complejo

Ahora que hemos analizado cómo funciona un diálogo en cascada y cómo crear uno, vamos a probar un diálogo más complejo para reservar una mesa.

Para administrar la solicitud de reserva de mesa, deberá definir un diálogo en **cascada** con cuatro pasos. En esta conversación, también se usarán `DatetimePrompt` y `NumberPrompt` además de `TextPrompt`.



# <a name="ctabcstab"></a>[C#](#tab/cstab)

Empiece con la plantilla Echo Bot y cambie el nombre del bot a CafeBot. Agregue un `DialogSet` y algunas variables estáticas de miembros.

```cs

namespace CafeBot
{
    public class CafeBot : IBot
    {
        private readonly DialogSet dialogs;

        // Usually, we would save the dialog answers to our state object, which will be covered in a later tutorial.
        // For purpose of this example, let's use the three static variables to store our reservation information.
        static DateTime reservationDate;
        static int partySize;
        static string reservationName;

        // the rest of the class definition is omitted here
        // but is discussed in the rest of this article
    }
}
```

Defina el diálogo `reserveTable`. Puede agregar el diálogo dentro del constructor de clases del bot.
```cs
public CafeBot()
{
    dialogs = new DialogSet();

    // Define our dialog
    dialogs.Add("reserveTable", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Context.SendActivity("Welcome to the reservation service.");

            await dc.Prompt("dateTimePrompt", "Please provide a reservation date and time.");
        },
        async(dc, args, next) =>
        {
            var dateTimeResult = ((DateTimeResult)args).Resolution.First();

            reservationDate = Convert.ToDateTime(dateTimeResult.Value);
            
            // Ask for next info
            await dc.Prompt("partySizePrompt", "How many people are in your party?");

        },
        async(dc, args, next) =>
        {
            partySize = (int)args["Value"];

            // Ask for next info
            await dc.Prompt("textPrompt", "Whose name will this be under?");
        },
        async(dc, args, next) =>
        {
            reservationName = args["Text"];
            string msg = "Reservation confirmed. Reservation details - " +
            $"\nDate/Time: {reservationDate.ToString()} " +
            $"\nParty size: {partySize.ToString()} " +
            $"\nReservation name: {reservationName}";
            await dc.Context.SendActivity(msg);
            await dc.End();
        }
    });

     // Add a prompt for the reservation date
     dialogs.Add("dateTimePrompt", new Microsoft.Bot.Builder.Dialogs.DateTimePrompt(Culture.English));
     // Add a prompt for the party size
     dialogs.Add("partySizePrompt", new Microsoft.Bot.Builder.Dialogs.NumberPrompt<int>(Culture.English));
     // Add a prompt for the user's name
     dialogs.Add("textPrompt", new Microsoft.Bot.Builder.Dialogs.TextPrompt());
}
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

El diálogo `reserveTable` tendrá este aspecto:

```javascript
// Reserve a table:
// Help the user to reserve a table
var reservationInfo = {};

dialogs.add('reserveTable', [
    async function(dc, args, next){
        await dc.context.sendActivity("Welcome to the reservation service.");

        reservationInfo = {}; // Clears any previous data
        await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
    },
    async function(dc, result){
        reservationInfo.dateTime = result[0].value;

        // Ask for next info
        await dc.prompt('partySizePrompt', "How many people are in your party?");
    },
    async function(dc, result){
        reservationInfo.partySize = result;

        // Ask for next info
        await dc.prompt('textPrompt', "Whose name will this be under?");
    },
    async function(dc, result){
        reservationInfo.reserveName = result;
        
        // Reservation confirmation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${reservationInfo.dateTime} 
            <br/>Party size: ${reservationInfo.partySize} 
            <br/>Reservation name: ${reservationInfo.reserveName}`;
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

---

El flujo de conversación del diálogo `reserveTable` formulará al usuario 3 preguntas en los tres primeros pasos de la cascada. El paso cuatro procesa la respuesta a la última pregunta y envía al usuario la confirmación de la reserva.



# <a name="ctabcstab"></a>[C#](#tab/cstab)
Cada paso de la cascada del diálogo `reserveTable` usa un aviso para pedir información al usuario. El código siguiente se usó para agregar los avisos al conjunto de diálogos.

```cs
dialogs.Add("dateTimePrompt", new Microsoft.Bot.Builder.Dialogs.DateTimePrompt(Culture.English));
dialogs.Add("partySizePrompt", new Microsoft.Bot.Builder.Dialogs.NumberPrompt<int>(Culture.English));
dialogs.Add("textPrompt", new Microsoft.Bot.Builder.Dialogs.TextPrompt());
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Para que esta cascada funcione, también necesitará agregar los avisos al objeto `dialogs`:

```javascript
// Define prompts
// Generic prompts
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt());
dialogs.add('partySizePrompt', new botbuilder_dialogs.NumberPrompt());
```

---

Ahora, está listo para enlazarlo a la lógica del bot.

## <a name="start-the-dialog"></a>Inicio del diálogo

# <a name="ctabcstab"></a>[C#](#tab/cstab)
Modifique la opción `OnTurn` del bot para que contenga el código siguiente:
```cs
public async Task OnTurn(ITurnContext context)
{
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // The type parameter PropertyBag inherits from 
        // Dictionary<string,object>
        var state = ConversationState<Dictionary<string, object>>.Get(context);
        var dc = dialogs.CreateContext(context, state);
        await dc.Continue();

        // Additional logic can be added to enter each dialog depending on the message received
        
        if(!context.Responded)
        {
            if (context.Activity.Text.ToLowerInvariant().Contains("reserve table"))
            {
                await dc.Begin("reserveTable");
            }
            else
            {
                await context.SendActivity($"You said '{context.Activity.Text}'");
            }
        }
    }
}
```


En **Startup.cs**, cambie la inicialización del middleware ConversationState para que use una clase que deriva de `Dictionary<string,object>` en lugar de derivar de `EchoState`.

Por ejemplo, en `Configure()`:
```cs
options.Middleware.Add(new ConversationState<Dictionary<string, object>>(dataStore));
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Para capturar la intención del usuario para esta solicitud, agregue unas líneas de código al método `processActivity()`. Modifique el método `processActivity()` del bot para que tenga este aspecto:

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);
        const dc = dialogs.createContext(context, convo);

        if (isMessage) {
            // Check for valid intents
            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greetings');
            }
            else if(context.activity.text.match(/reserve table/ig)){
                await dc.begin('reserveTable');
            }
        }

        if(!context.responded){
            // Continue executing the "current" dialog, if any.
            await dc.continue();

            if(!context.responded && isMessage){
                // Default message
                await context.sendActivity("Hi! I'm a simple bot. Please say 'Hi' or 'Reserve table'.");
            }
        }
    });
});
```

En tiempo de ejecución, cada vez que el usuario envía el mensaje que contiene la cadena `reserve table`, el bot iniciará la conversación `reserveTable`.

---



## <a name="next-steps"></a>Pasos siguientes

En este tutorial, el bot guarda las entradas del usuario en variables dentro de nuestro bot. Si desea almacenar o conservar esta información, deberá agregar el estado a la capa de middleware. Vamos a analizar más detalladamente cómo conservar los datos de estado del usuario en el siguiente tutorial. 

> [!div class="nextstepaction"]
> [Conservación de datos de usuario](bot-builder-tutorial-persist-user-inputs.md)

-->

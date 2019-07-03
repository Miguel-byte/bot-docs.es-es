---
title: Administración de un flujo de conversación con diálogos | Microsoft Docs
description: Aprenda a administrar una conversación con diálogos entre un bot y un usuario en Bot Framework SDK para Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 497a00dd9327d3b4c26c1468bba04f18e2244ca1
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405036"
---
# <a name="manage-conversation-flow-with-dialogs"></a>Administración del flujo de conversación con diálogos

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-manage-conversation-flow.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-manage-conversation-flow.md)

La administración del flujo de conversación es una tarea esencial en la compilación de bots. Un bot debe ser capaz de realizar tareas importantes con elegancia y de controlar las interrupciones correctamente. Gracias a Bot Framework SDK para Node.js, puede administrar el flujo de conversación mediante diálogos.

Un diálogo es similar a una función de un programa. Por lo general está diseñado para realizar una operación específica y se puede invocar con la frecuencia que sea necesaria. Puede encadenar varios diálogos juntos para así controlar casi cualquier flujo de conversación que quiera que administre el bot. Bot Framework SDK para Node.js incluye características integradas como [avisos](bot-builder-nodejs-dialog-prompt.md) y [cascadas](bot-builder-nodejs-dialog-waterfall.md), que le ayudarán a administrar el flujo de conversación.

En este artículo se proporciona una serie de ejemplos para explicar cómo administrar flujos de conversación sencillos y complejos en los que el bot puede controlar las interrupciones y reanudar el flujo correctamente mediante diálogos. Los ejemplos se basan en los siguientes escenarios: 

1. El bot realizará la reserva de una cena.
2. El bot puede procesar una solicitud de "Ayuda" en cualquier fase de la reserva.
3. El bot puede procesar ayuda contextual en el paso actual de la reserva.
4. El bot puede controlar varios temas de conversación.

## <a name="manage-conversation-flow-with-a-waterfall"></a>Administración de un flujo de conversación mediante una cascada

Una [cascada](bot-builder-nodejs-dialog-waterfall.md) es un diálogo que permite al bot guiar fácilmente a un usuario por una serie de tareas. En este ejemplo, el bot de reserva le hace al usuario una serie de preguntas para poder procesar la solicitud de reserva. El bot pedirá al usuario la información siguiente:

1. Fecha y hora de la reserva
2. Número de asistentes
3. Nombre de la persona que realiza la reserva

El siguiente ejemplo de código le muestra cómo usar una cascada para guiar al usuario por una serie de avisos.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a dinner reservation bot that uses a waterfall technique to prompt users for input.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        builder.Prompts.time(session, "Please provide a reservation date and time (e.g.: June 6th at 5pm)");
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        builder.Prompts.number(session, "How many people are in your party?");
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        builder.Prompts.text(session, "Whose name will this reservation be under?");
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;
        
        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 
```

La funcionalidad principal de este bot se produce en el diálogo predeterminado. El diálogo predeterminado se define cuando se crea el bot: 

```javascript
var bot = new builder.UniversalBot(connector, [..waterfall steps..]); 
```

Además, durante este proceso de creación, puede establecer qué [almacenamiento de datos](bot-builder-nodejs-state.md) va a usar. Por ejemplo, para usar el almacenamiento en memoria, puede establecerlo de la siguiente forma:

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
var bot = new builder.UniversalBot(connector, [..waterfall steps..]).set('storage', inMemoryStorage); // Register in-memory storage 
```

El diálogo predeterminado se crea como una matriz de funciones que define los pasos de la cascada. En el ejemplo hay cuatro funciones, por lo que la cascada consta de cuatro pasos. Cada paso realiza una sola tarea y los resultados se procesan en el paso siguiente. El proceso continúa hasta el último paso, en el que se confirma la reserva y finaliza el diálogo.

En la siguiente captura de pantalla se muestran los resultados de este bot que se ejecuta en [Bot Framework Emulator](../bot-service-debug-emulator.md):

![Administración del flujo de conversación mediante una cascada](../media/bot-builder-nodejs-dialog-manage-conversation/waterfall-results.png)

### <a name="prompt-user-for-input"></a>Petición de datos de entrada al usuario

Cada paso de este ejemplo usa un aviso para solicitar datos de entrada al usuario. Un aviso es un tipo especial de diálogo que solicita datos de entrada al usuario, espera una respuesta y devuelve la respuesta al paso siguiente de la cascada. Consulte [Petición de datos de entrada al usuario](bot-builder-nodejs-dialog-prompt.md) para más información acerca de los diferentes tipos de avisos que puede usar en su bot.

En este ejemplo, el bot usa `Prompts.text()` para solicitar una respuesta libre al usuario en formato de texto. El usuario puede responder con cualquier texto y el bot debe decidir cómo controlar la respuesta. `Prompts.time()` usa la biblioteca [Chrono](https://github.com/wanasit/chrono) para analizar una cadena en busca de información sobre la fecha y hora. Esto permite que su bot comprenda más lenguaje natural para especificar la fecha y la hora. Por ejemplo:  "6 de junio de 2017 a las 9 pm", "hoy a las 7:30 pm", "el próximo lunes a las 6 pm", etc.

> [!TIP] 
> La hora que especifica el usuario se convierte a la hora UTC en función de la zona horaria del servidor que hospeda el bot. Como el servidor puede estar ubicado en una zona horaria diferente a la del usuario, asegúrese de tener en cuenta las zonas horarias. Para convertir la fecha y hora a la hora local del usuario, considere la posibilidad de pedir al usuario que indique en qué zona horaria se encuentra.

## <a name="manage-a-conversation-flow-with-multiple-dialogs"></a>Administración de un flujo de conversación con varios diálogos

Otra técnica para administrar un flujo de conversación es usar una combinación de cascada y varios diálogos. La cascada le permite encadenar funciones en un diálogo, mientras que estos le permiten dividir una conversación en fragmentos más pequeños de funcionalidad que se pueden reutilizar en cualquier momento.

Por ejemplo, piense en el bot para la reserva de una cena. El siguiente ejemplo de código le muestra el ejemplo anterior reescrito para poder usar una cascada y varios diálogos.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a dinner reservation bot that uses multiple dialogs to prompt users for input.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        session.beginDialog('askForDateTime');
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        session.beginDialog('askForPartySize');
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        session.beginDialog('askForReserverName');
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;

        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

// Dialog to ask for a date and time
bot.dialog('askForDateTime', [
    function (session) {
        builder.Prompts.time(session, "Please provide a reservation date and time (e.g.: June 6th at 5pm)");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);

// Dialog to ask for number of people in the party
bot.dialog('askForPartySize', [
    function (session) {
        builder.Prompts.text(session, "How many people are in your party?");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
])

// Dialog to ask for the reservation name.
bot.dialog('askForReserverName', [
    function (session) {
        builder.Prompts.text(session, "Who's name will this reservation be under?");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

Los resultados de la ejecución de este bot son exactamente los mismos que los del bot anterior, en el que solo se usaba una cascada. No obstante, desde el punto de vista de la programación, hay dos diferencias principales:

1. El diálogo predeterminado se dedica a administrar el flujo de la conversación.
2. Un diálogo independiente administra la tarea de cada paso de la conversación. En este caso, el bot necesitaba tres fragmentos de información, por lo que realiza tres avisos al usuario. Ahora, cada aviso está contenido en su propio diálogo.

Con esta técnica, puede separar el flujo de conversación de la lógica de la tarea. Esto permite que los diálogos se puedan reutilizar en flujos de conversación diferentes si fuera necesario. 

## <a name="respond-to-user-input"></a>Respuesta a entradas del usuario

En el proceso de guiar al usuario por una serie de tareas, si el usuario tiene preguntas o si desea solicitar información adicional antes de responder, ¿cómo se podrían controlar esas solicitudes? Por ejemplo, independientemente de donde esté el usuario de la conversación, ¿cómo debería responder el bot si el usuario escribe "Ayuda", "Asistencia" o "Cancelar"? ¿Qué ocurre si el usuario desea información adicional sobre un paso? ¿Qué ocurre si el usuario cambia de opinión y desea abandonar la tarea actual para iniciar una tarea completamente diferente?

Bot Framework SDK para Node.js permite que un bot escuche determinadas entradas en un contexto global o en uno local en el ámbito del diálogo actual. Estas entradas se denominan [acciones](bot-builder-nodejs-dialog-actions.md), y permiten que el bot escuche las entradas del usuario según una cláusula `matches`. Depende del bot decidir cómo responder a entradas específicas del usuario.

### <a name="handle-global-action"></a>Control de acciones globales

Si desea que el bot pueda controlar acciones en cualquier punto de una conversación, use `triggerAction`. Los desencadenadores permiten al bot invocar un diálogo específico si la entrada coincide con el término especificado. Por ejemplo, si desea admitir una opción global de "Ayuda", puede crear un diálogo de ayuda y adjuntar una acción `triggerAction` que escuche en busca de entradas que coincidan con "Ayuda".

El ejemplo de código siguiente le muestra cómo adjuntar una acción `triggerAction` a un diálogo para especificar que se debe invocar al diálogo cuando el usuario escribe "ayuda".

```javascript
// The dialog stack is cleared and this dialog is invoked when the user enters 'help'.
bot.dialog('help', function (session, args, next) {
    session.endDialog("This is a bot that can help you make a dinner reservation. <br/>Please say 'next' to continue");
})
.triggerAction({
    matches: /^help$/i,
});
```

De forma predeterminada, cuando se ejecuta una acción `triggerAction`, la pila del diálogo se borra y el diálogo desencadenado se convierte en el nuevo diálogo predeterminado. En este ejemplo, cuando se ejecuta la acción `triggerAction`, se borra la pila de diálogo y el diálogo `help` se agrega a la pila como nuevo diálogo predeterminado. Si este no es el comportamiento deseado, puede agregar la opción `onSelectAction` a la acción `triggerAction`. La opción `onSelectAction` permite al bot iniciar un nuevo diálogo sin borrar la pila de diálogo, lo que permite redirigir la conversación temporalmente y volver a reanudarla más adelante donde lo dejó.

En el ejemplo de código siguiente se muestra cómo usar la opción `onSelectAction` con la acción `triggerAction` para que el diálogo `help` se agregue a la pila de diálogos existente (y esta no se borra).

```javascript
bot.dialog('help', function (session, args, next) {
    session.endDialog("This is a bot that can help you make a dinner reservation. <br/>Please say 'next' to continue");
})
.triggerAction({
    matches: /^help$/i,
    onSelectAction: (session, args, next) => {
        // Add the help dialog to the dialog stack 
        // (override the default behavior of replacing the stack)
        session.beginDialog(args.action, args);
    }
});
```

En este ejemplo, el diálogo `help` toma el control de la conversación cuando el usuario escribe "ayuda". Como la acción `triggerAction` contiene la opción `onSelectAction`, el diálogo `help` se inserta en la pila de diálogo existente y esta no se borra. Cuando finaliza el diálogo `help`, se elimina de la pila de diálogo y la conversación se reanuda en el punto en que se interrumpió con el comando `help`.

### <a name="handle-contextual-action"></a>Control de acciones contextuales

En el ejemplo anterior, se invoca el diálogo `help` si el usuario escribe "ayuda" en cualquier punto de la conversación. En este caso, el diálogo solo puede ofrecer orientaciones de ayuda generales ya que no hay ningún contexto específico para la solicitud de ayuda del usuario. Pero, ¿qué ocurre si el usuario desea solicitar ayuda con respecto a un momento específico de la conversación? En ese caso, el diálogo `help` se debe desencadenar dentro del contexto del diálogo actual.

Por ejemplo, piense en el bot para la reserva de una cena. ¿Qué sucede si un usuario desea conocer el tamaño máximo del grupo cuando se le pregunta por el número de asistentes de su grupo? Para manejar este escenario, puede adjuntar una acción `beginDialogAction` al diálogo `askForPartySize`, que escuche la entrada de usuario "ayuda".

El siguiente ejemplo de código muestra cómo adjuntar ayuda contextual a un diálogo mediante `beginDialogAction`.

```javascript
// Dialog to ask for number of people in the party
bot.dialog('askForPartySize', [
    function (session) {
        builder.Prompts.text(session, "How many people are in your party?");
    },
    function (session, results) {
       session.endDialogWithResult(results);
    }
])
.beginDialogAction('partySizeHelpAction', 'partySizeHelp', { matches: /^help$/i });

// Context Help dialog for party size
bot.dialog('partySizeHelp', function(session, args, next) {
    var msg = "Party size help: Our restaurant can support party sizes up to 150 members.";
    session.endDialog(msg);
})
```

En este ejemplo, cada vez que el usuario escriba "help", el bot insertará el diálogo `partySizeHelp` en la pila. Ese diálogo envía un mensaje de ayuda al usuario y, después, finaliza el diálogo y devuelve el control de nuevo al diálogo `askForPartySize`, que vuelve a preguntar el tamaño del grupo al usuario.

Es importante tener en cuenta que esta ayuda contextual solo se ejecuta cuando el usuario está en el diálogo `askForPartySize`. En caso contrario, se ejecutará el mensaje de ayuda de la acción `triggerAction` en su lugar. En otras palabras, la cláusula local `matches` siempre tiene prioridad sobre la cláusula global `matches`. Por ejemplo, si una acción `beginDialogAction` coincide con **ayuda**, las coincidencias de **ayuda** de la acción `triggerAction` no se ejecutarán. Para más información, consulte [Prioridad de acciones](bot-builder-nodejs-dialog-actions.md#action-precedence).

### <a name="change-the-topic-of-conversation"></a>Cambio del tema de conversación

De forma predeterminada, la ejecución de una acción `triggerAction` borra la pila de diálogo y restablece la conversación, empezando por el diálogo especificado. Este comportamiento es preferible cuando el bot tiene que cambiar de un tema de conversación a otro como, por ejemplo, cuando un usuario en medio del proceso normal de reserva de una cena decide pedir que se le sirva la cena en la habitación de su hotel. 

El siguiente ejemplo se basa en el anterior de forma que el bot permita al usuario hacer una reserva para cenar en un restaurante o bien pedir que se le envíe la cena. En este bot, el diálogo predeterminado es un diálogo de saludo que presenta dos opciones para el usuario: `Dinner Reservation` y `Order Dinner`.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This bot enables users to either make a dinner reservation or order dinner.
var bot = new builder.UniversalBot(connector, function(session){
    var msg = "Welcome to the reservation bot. Please say `Dinner Reservation` or `Order Dinner`";
    session.send(msg);
}).set('storage', inMemoryStorage); // Register in-memory storage 
```

La lógica de reserva de cena que estaba en el diálogo predeterminado del ejemplo anterior está ahora en su propio diálogo llamado `dinnerReservation`. El flujo de `dinnerReservation` sigue siendo el mismo que en la versión de varios diálogos analizada anteriormente. La única diferencia es que el diálogo tiene una acción `triggerAction` adjunta. Observe que en esta versión, `confirmPrompt` pide al usuario que confirme que desea cambiar el tema de conversación antes de invocar el nuevo diálogo. Es recomendable incluir un aviso `confirmPrompt` en escenarios como este porque una vez que se borra la pila de diálogo, se redirigirá al usuario al nuevo tema de conversación, abandonando por tanto el tema que estaba en curso.

```javascript
// This dialog helps the user make a dinner reservation.
bot.dialog('dinnerReservation', [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        session.beginDialog('askForDateTime');
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        session.beginDialog('askForPartySize');
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        session.beginDialog('askForReserverName');
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;

        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
])
.triggerAction({
    matches: /^dinner reservation$/i,
    confirmPrompt: "This will cancel your current request. Are you sure?"
});
```

El segundo tema de conversación se define en el diálogo `orderDinner` mediante una cascada. Este diálogo simplemente muestra el menú de la cena y pide al usuario el número de una habitación una vez especificado el pedido. Se adjunta una acción `triggerAction` al diálogo para especificar que debe invocarse si el usuario escribe "encargar cena", y para asegurarse de que se pide al usuario que confirme su selección en caso de que muestre intención de cambiar el tema de conversación.

```javascript
// This dialog help the user order dinner to be delivered to their hotel room.
var dinnerMenu = {
    "Potato Salad - $5.99": {
        Description: "Potato Salad",
        Price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        Description: "Tuna Sandwich",
        Price: 6.89
    },
    "Clam Chowder - $4.50":{
        Description: "Clam Chowder",
        Price: 4.50
    }
};

bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function (session, results) {
        if (results.response) {
            var order = dinnerMenu[results.response.entity];
            var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
            session.dialogData.order = order;
            session.send(msg);
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endDialog(msg);
        }
    }
])
.triggerAction({
    matches: /^order dinner$/i,
    confirmPrompt: "This will cancel your order. Are you sure?"
});
```

Una vez que el usuario inicia una conversación y selecciona `Dinner Reservation` o `Order Dinner`, puede cambiar de opinión en cualquier momento. Por ejemplo, si el usuario está en medio del proceso para realizar una reserva de cena y escribe "encargar cena", el bot confirmará mediante un mensaje del tipo: "Esto cancelará la solicitud actual. ¿Está seguro de que desea continuar? Si el usuario escribe "no", se cancelará la solicitud y el usuario podrá continuar con el proceso de reserva de la cena. Si el usuario escribe "sí", el bot borrará la pila de diálogo y transferirá el control de la conversación al diálogo `orderDinner`.

## <a name="end-conversation"></a>Fin de la conversación

En los ejemplos anteriores, los diálogos se cierran con `session.endDialog` o `session.endDialogWithResult`. Ambas opciones cierran el diálogo, lo borran de la pila y devuelven el control al diálogo de llamada. En los casos en los que el usuario ha llegado al final de la conversación, se debería usar `session.endConversation` para indicar que la conversación ha finalizado.

El método [`session.endConversation`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#endconversation) finaliza una conversación y, opcionalmente, envía un mensaje al usuario. Por ejemplo, el diálogo `orderDinner` del ejemplo anterior podría finalizar la conversación mediante `session.endConversation`, como se muestra en el siguiente ejemplo de código.

```javascript
bot.dialog('orderDinner', [
    //...waterfall steps...
    // Last step
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endConversation(msg);
        }
    }
]);
```

Una llamada a `session.endConversation` finalizará la conversación, con lo cual se borrará la pila de diálogo y se restablecerá el almacenamiento de [`session.conversationData`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#conversationdata). Para más información sobre el almacenamiento de datos, consulte [Administración de datos de estado](bot-builder-nodejs-state.md).

Llamar a `session.endConversation` es algo lógico cuando el usuario completa el flujo de conversación para el que se ha diseñado el bot. También puede usar `session.endConversation` para finalizar la conversación en los casos en los que el usuario escribe "cancelar" o "adiós" en medio de una conversación. Para ello, basta con adjuntar una acción `endConversationAction` al diálogo y hacer que este desencadenador escuche coincidencias de entrada de "cancelar" o "adiós".

El ejemplo de código siguiente muestra como adjuntar una acción `endConversationAction` a un diálogo para finalizar la conversación si el usuario escribe "cancelar" o "adiós".

```javascript
bot.dialog('dinnerOrder', [
    //...waterfall steps...
])
.endConversationAction(
    "endOrderDinner", "Ok. Goodbye.",
    {
        matches: /^cancel$|^goodbye$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

Dado que finalizar una conversación con `session.endConversation` o `endConversationAction` borrará la pila de diálogo y forzará al usuario a empezar de nuevo, debería incluir un aviso `confirmPrompt` para asegurarse de que el usuario realmente desea hacer eso.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha explorado las maneras de administrar conversaciones que son secuenciales por naturaleza. ¿Qué ocurre si desea repetir un diálogo o usar el patrón de bucles en la conversación? Veamos cómo puede hacerlo sustituyendo los diálogos de la pila.

> [!div class="nextstepaction"]
> [Sustitución de diálogos](bot-builder-nodejs-dialog-replace.md)


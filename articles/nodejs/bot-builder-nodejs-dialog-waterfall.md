---
title: Definición de los pasos de conversación con cascadas | Microsoft Docs
description: Aprenda a usar cascadas para definir los pasos de una conversación con el SDK de Bot Builder para Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 2fa857e57d5be4751476874b8c193c7053a1bf39
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000282"
---
# <a name="define-conversation-steps-with-waterfalls"></a>Definición de los pasos de la conversación con cascadas

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Una conversación es una serie de mensajes que se intercambian un usuario y un bot. Cuando el objetivo del bot es llevar al usuario por una serie de pasos, puede usar una cascada para definir los pasos de la conversación.

Una cascada es una implementación específica de un [diálogo](bot-builder-nodejs-dialog-overview.md) que se usa normalmente para recopilar información del usuario o guiarlo en una serie de tareas. Las tareas se implementan como una matriz de funciones donde los resultados de la primera función se pasan como entrada a la función siguiente y así sucesivamente. Cada función representa normalmente un paso del proceso general. En cada paso, el bot pide al usuario que intervenga, espera una respuesta y, a continuación, pasa el resultado al paso siguiente.

Este artículo le ayudará a entender cómo funciona una cascada y cómo puede usarla para [administrar el flujo de conversación](bot-builder-nodejs-dialog-manage-conversation.md).

## <a name="conversation-steps"></a>Pasos de la conversación

Una conversación suele implica varios intercambios de solicitud/respuesta entre el usuario y el bot. Cada intercambio de solicitud/respuesta es un paso adelante en la conversación. Puede crear una conversación mediante una cascada con tan solo dos pasos.

Por ejemplo, considere el siguiente diálogo `greetings`. El primer paso de la cascada pide al usuario el nombre y el segundo paso usa la respuesta para saludar al usuario por su nombre.

```javascript
bot.dialog('greetings', [
    // Step 1
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    // Step 2
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

Esto es posible gracias al uso de mensajes. El SDK de Bot Builder para Node.js proporciona varios tipos diferentes de [mensajes](bot-builder-nodejs-dialog-prompt.md) integrados que puede usar para pedir al usuario diversos tipos de información.

El ejemplo de código siguiente muestra un diálogo que usa mensajes para recopilar diversos fragmentos de información del usuario mediante una cascada de 4 pasos.

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
        builder.Prompts.text(session, "How many people are in your party?");
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

En este ejemplo, el diálogo predeterminado tiene cuatro funciones, cada una de las cuales representa un paso de la cascada. Cada paso pide al usuario que intervenga y envía los resultados al paso siguiente para su procesamiento. Este proceso continúa hasta que se ejecuta el último paso, que confirma la reserva y finaliza el diálogo.

En la siguiente captura de pantalla se muestran los resultados de este bot que se ejecuta en [Bot Framework Emulator](~/bot-service-debug-emulator.md).

![Administración del flujo de conversación mediante una cascada](~/media/bot-builder-nodejs-dialog-manage-conversation/waterfall-results.png)

## <a name="create-a-conversation-with-multiple-waterfalls"></a>Creación de una conversación con varias cascadas

Puede usar varias cascadas dentro de una conversación para definir cualquier estructura de conversación que requiera el bot. Por ejemplo, podría usar una cascada para administrar el flujo de conversación y usar cascadas adicionales para recopilar información del usuario. Cada cascada se encapsula en un diálogo y se puede invocar mediante una llamada a la función `session.beginDialog`.

En el ejemplo de código siguiente se muestra cómo usar varias cascadas en una conversación. La cascada dentro del diálogo `greetings` administra el flujo de la conversación, mientras que la cascada dentro del diálogo `askName` recopila información del usuario.

```javascript
bot.dialog('greetings', [
    function (session) {
        session.beginDialog('askName');
    },
    function (session, results) {
        session.endDialog('Hello %s!', results.response);
    }
]);
bot.dialog('askName', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

## <a name="advance-the-waterfall"></a>Avance en la cascada

Una cascada avanza paso a paso en la secuencia con que las funciones están definidas en la matriz. La primera función dentro de una cascada puede recibir los argumentos que se pasan al diálogo. Cualquier función dentro de una cascada puede usar la función `next` para avanzar al paso siguiente sin pedirle al usuario que intervenga.

En el ejemplo de código siguiente se muestra cómo usar la función `next` dentro de un diálogo que lleva al usuario por el proceso de proporcionar información para su perfil de usuario. En cada paso de la cascada, el bot pide al usuario un fragmento de información (si es necesario) y el siguiente paso de la cascada procesa la respuesta del usuario (si la hay). El paso final de la cascada en el diálogo `ensureProfile` finaliza ese diálogo y devuelve la información de perfil completada al diálogo de llamada, que envía un saludo personalizado al usuario.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This bot ensures user's profile is up to date.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.beginDialog('ensureProfile', session.userData.profile);
    },
    function (session, results) {
        session.userData.profile = results.response; // Save user profile.
        session.send(`Hello ${session.userData.profile.name}! I love ${session.userData.profile.company}!`);
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

bot.dialog('ensureProfile', [
    function (session, args, next) {
        session.dialogData.profile = args || {}; // Set the profile or create the object.
        if (!session.dialogData.profile.name) {
            builder.Prompts.text(session, "What's your name?");
        } else {
            next(); // Skip if we already have this info.
        }
    },
    function (session, results, next) {
        if (results.response) {
            // Save user's name if we asked for it.
            session.dialogData.profile.name = results.response;
        }
        if (!session.dialogData.profile.company) {
            builder.Prompts.text(session, "What company do you work for?");
        } else {
            next(); // Skip if we already have this info.
        }
    },
    function (session, results) {
        if (results.response) {
            // Save company name if we asked for it.
            session.dialogData.profile.company = results.response;
        }
        session.endDialogWithResult({ response: session.dialogData.profile });
    }
]);
```

> [!NOTE]
> En este ejemplo se usan dos contenedores de datos diferentes para almacenar los datos: `dialogData` y `userData`. Si el bot se distribuye entre varios nodos de proceso, cada paso de la cascada podría procesarse con un nodo diferente. Para más información sobre cómo almacenar datos de bot, consulte [Administración de datos de estado](bot-builder-nodejs-state.md).

## <a name="end-a-waterfall"></a>Finalización de una cascada

Un diálogo que se crea con una cascada se debe terminar explícitamente, de lo contrario, el bot repetirá la cascada indefinidamente. Puede finalizar una cascada mediante uno de los métodos siguientes:

* `session.endDialog`: use este método para finalizar la cascada si no hay ningún dato para devolver al diálogo de llamada.

* `session.endDialogWithResult`: use este método para finalizar la cascada si hay datos para devolver al diálogo de llamada. El argumento `response` que se devuelve puede ser un objeto JSON o cualquier tipo de datos primitivo de JavaScript. Por ejemplo: 
  ```javascript
  session.endDialogWithResult({
    response: { name: session.dialogData.name, company: session.dialogData.company }
  });
  ```

* `session.endConversation`: use este método para finalizar la cascada si el final de esta representa el final de la conversación.

Como alternativa al uso de uno de estos tres métodos para finalizar una cascada, puede asociar el desencadenador `endConversationAction` al diálogo. Por ejemplo: 

```javascript
bot.dialog('dinnerOrder', [
    //...waterfall steps...,
    // Last step
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endConversation(msg);
        }
    }
])
.endConversationAction(
    "endOrderDinner", "Ok. Goodbye.",
    {
        matches: /^cancel$|^goodbye$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

## <a name="next-steps"></a>Pasos siguientes

Con una cascada puede recopilar información del usuario con *mensajes*. Vamos a profundizar en cómo puede solicitar la intervención del usuario.

> [!div class="nextstepaction"]
> [Solicitud de entrada de usuario](bot-builder-nodejs-dialog-prompt.md)

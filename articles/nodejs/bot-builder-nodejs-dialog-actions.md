---
title: Controlar las acciones del usuario | Microsoft Docs
description: Obtenga información sobre cómo controlar las acciones del usuario al permitir que el bot escuche y controle la entrada de usuario que contenga ciertas palabras clave mediante el Bot Builder SDK para Node.js.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 26f6e9520fe5d2ebb83ceb4e6a497a35e9d2611f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999262"
---
# <a name="handle-user-actions"></a>Controlar acciones del usuario

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-global-handlers.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-actions.md)

Normalmente los usuarios intentan tener acceso a cierta funcionalidad dentro de un bot con palabras clave, como "ayuda", "cancelar" o "empezar de nuevo". Los usuarios hacen esto en el medio de una conversación, cuando el bot está esperando una respuesta diferente. Al implementar **acciones**, puede diseñar el bot para que controle dichas solicitudes de manera más correcta. Los controladores examinarán la entrada del usuario para las palabras clave que especifique, por ejemplo, "ayuda", "cancelar" o "empezar de nuevo" y responder según corresponda. 

![cómo hablan los usuarios](../media/designing-bots/capabilities/trigger-actions.png)

## <a name="action-types"></a>Tipos de acción

Los tipos de acciones que puede adjuntar a un cuadro de diálogo se muestran en la tabla siguiente. El vínculo para cada nombre de acción le llevará a una sección que proporcionan más detalles sobre esa acción.

| . | Ámbito | DESCRIPCIÓN |
|------|------| ---- |
| [triggerAction](#bind-a-triggeraction) | Global | Enlaza una acción al cuadro de diálogo que borrará la pila del cuadro de diálogo y se insertará en la parte inferior de la pila. Use la opción `onSelectAction` para invalidar este comportamiento predeterminado. |
| [customAction](#bind-a-customaction) | Global | Enlaza una acción personalizada al bot que pueda procesar información o realizar una acción sin afectar a la pila del cuadro de diálogo. Use la opción `onSelectAction` para personalizar la funcionalidad de esta acción. |
[beginDialogAction](#bind-a-begindialogaction) | Contextual | Enlaza una acción al cuadro de diálogo que inicia otro cuadro de diálogo cuando se desencadena. El cuadro de diálogo inicial se insertará en la pila y se extraerá una vez que finalice. |
[reloadAction](#bind-a-reloadaction) | Contextual | Enlaza una acción al cuadro de diálogo que hace que otro cuadro de diálogo se recargue cuando se desencadena. Puede usar `reloadAction` para controlar las expresiones del usuario, como "empezar de nuevo". |
[cancelAction](#bind-a-cancelaction) | Contextual | Enlaza una acción al cuadro de diálogo que cancela otro cuadro de diálogo cuando se desencadena. Puede usar `cancelAction` para controlar las expresiones del usuario, como "cancelar" o "no importa". |
[endConversationAction](#bind-an-endconversationaction) | Contextual | Enlaza una acción al cuadro de diálogo que finaliza la conversación con el usuario cuando se desencadena. Puede usar `endConversationAction` para controlar las expresiones del usuario, como "adiós". |

## <a name="action-precedence"></a>Prioridad de acciones 

Cuando un bot recibe una expresión de un usuario, comprueba la expresión frente a todas las acciones registradas en la pila del cuadro de diálogo. La coincidencia se inicia desde la parte superior de la pila hasta el final. Si no hay ninguna coincidencia, luego comprueba la expresión frente a la opción `matches` de todas las acciones globales.

La prioridad de acciones es importante en casos donde se usa el mismo comando en contextos diferentes. Por ejemplo, puede usar el comando "Help" para el bot como ayuda general. También puede usar "Help" para cada una de las tareas, pero estos comandos de ayuda son contextuales a cada tarea. Para obtener un ejemplo funcional que ahonda en este punto, consulte [Respond to user input](bot-builder-nodejs-dialog-manage-conversation-flow.md#respond-to-user-input) (Responder a la entrada del usuario).

## <a name="bind-actions-to-dialog"></a>Enlazar acciones al cuadro de diálogo

Las expresiones del usuario o los clics en botones pueden *desencadenar* una acción, que está asociada con un [cuadro de diálogo](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html).
Si se especifica *matches*, la acción escuchará al usuario hasta que pronuncie una palabra o frase que desencadene la acción.  La opción `matches` puede tomar una expresión regular o el nombre de un [reconocedor][RecognizeIntent].
Para enlazar la acción al clic en un botón, use [CardAction.dialogAction()] [ CardAction] para desencadenar la acción.

Las acciones son *encadenables*, lo que le permite enlazar tantas acciones a un cuadro de diálogo como desee.

### <a name="bind-a-triggeraction"></a>Enlazar triggerAction

Para enlazar [triggerAction][triggerAction] a un cuadro de diálogo, haga lo siguiente:

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will clear the dialog stack and pushes
// the 'orderDinner' dialog onto the bottom of stack.
.triggerAction({
    matches: /^order dinner$/i
});
```

Enlazar `triggerAction` a un cuadro de diálogo lo registra en el bot. Una vez que se desencadena, `triggerAction` borrará la pila del cuadro de diálogo e insertará el cuadro de diálogo desencadenado en la pila. Esta acción está concebida para cambiar entre [temas de conversación](bot-builder-nodejs-dialog-manage-conversation-flow.md#change-the-topic-of-conversation) o para permitir que los usuarios soliciten tareas independientes arbitrarias. Si desea invalidar el comportamiento por el que esta acción borra la pila del cuadro de diálogo, agregue una opción `onSelectAction` a `triggerAction`.

El siguiente fragmento de código muestra cómo proporcionar ayuda general de un contexto global sin borrar la pila del cuadro de diálogo.

```javascript
bot.dialog('help', function (session, args, next) {
    //Send a help message
    session.endDialog("Global help menu.");
})
// Once triggered, will start a new dialog as specified by
// the 'onSelectAction' option.
.triggerAction({
    matches: /^help$/i,
    onSelectAction: (session, args, next) => {
        // Add the help dialog to the top of the dialog stack 
        // (override the default behavior of replacing the stack)
        session.beginDialog(args.action, args);
    }
});
```

En este caso, `triggerAction` está asociado al cuadro de diálogo `help` mismo (en contraposición al cuadro de diálogo `orderDinner`). La opción `onSelectAction` le permite iniciar este cuadro de diálogo sin borrar la pila del cuadro de diálogo. Esto permite controlar las solicitudes globales, como "ayuda", "acerca de", "soporte", etc. Tenga en cuenta que la opción `onSelectAction` llama de manera explícita al método `session.beginDialog` para iniciar el cuadro de diálogo desencadenado. El identificador del cuadro de diálogo desencadenado se proporciona a través de `args.action`. No codifique manualmente el identificador del cuadro de diálogo (p. ej.: 'help') en este método o, de lo contrario, es posible que obtenga errores en tiempo de ejecución. Si desea desencadenar un mensaje de ayuda contextual para la tarea `orderDinner` misma, piense en la posibilidad de asociar `beginDialogAction` al cuadro de diálogo `orderDinner` en su lugar.

### <a name="bind-a-customaction"></a>Enlazar customAction

A diferencia de otros tipos de acción, `customAction` no tiene definida ninguna acción predeterminada. Depende de usted definir lo que hace esta acción. La ventaja de usar `customAction` es que tendrá la opción de procesar las solicitudes del usuario sin manipular la pila del cuadro de diálogo. Cuando se desencadena `customAction`, la opción `onSelectAction` puede procesar la solicitud sin insertar nuevos cuadros de diálogo en la pila. Una vez completada la acción, el control se devuelve al cuadro de diálogo que se encuentra en la parte superior de la pila, y el bot puede continuar.

Puede usar `customAction` para proporcionar solicitudes generales y de acción rápida, como "¿Cuál es la temperatura exterior ahora?", "¿Qué hora es ahora en París?", "Recordarme comprar leche a las 5 p. m. hoy", etc. Estas son acciones generales que el bot puede realizar sin manipular la pila.

Otra diferencia clave con `customAction` es que se enlaza con el bot en lugar de con un cuadro de diálogo.

El ejemplo de código siguiente muestra cómo enlazar `customAction` al `bot` que escucha solicitudes para establecer un recordatorio.

```javascript
bot.customAction({
    matches: /remind|reminder/gi,
    onSelectAction: (session, args, next) => {
        // Set reminder...
        session.send("Reminder is set.");
    }
})
```

### <a name="bind-a-begindialogaction"></a>Enlazar beginDialogAction

Enlazar `beginDialogAction` a un cuadro de diálogo registrará la acción en el cuadro de diálogo. Este método iniciará otro cuadro de diálogo cuando se desencadene. El comportamiento de esta acción es similar a llamar al método [beginDialog](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#begindialog). El nuevo cuadro de diálogo se inserta en la parte superior de la pila del cuadro de diálogo, por lo que no termina automáticamente la tarea actual. La tarea actual continúa una vez que finaliza el nuevo cuadro de diálogo. 

El fragmento de código siguiente muestra cómo enlazar [beginDialogAction][beginDialogAction] a un cuadro de diálogo.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will start the 'showDinnerCart' dialog.
// Then, the waterfall will resumed from the step that was interrupted.
.beginDialogAction('showCartAction', 'showDinnerCart', {
    matches: /^show cart$/i
});

// Show dinner items in cart
bot.dialog('showDinnerCart', function(session){
    for(var i = 1; i < session.conversationData.orders.length; i++){
        session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
    }

    // End this dialog
    session.endDialog(`Your total is: $${session.conversationData.orders[0].Price}`);
});
```

En casos donde deba pasar argumentos adicionales en el nuevo cuadro de diálogo, puede agregar una opción [`dialogArgs`](https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idialogactionoptions#dialogargs) a la acción.

Mediante el ejemplo anterior, puede modificarlo para que acepte argumentos pasados a través de `dialogArgs`.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will start the 'showDinnerCart' dialog.
// Then, the waterfall will resumed from the step that was interrupted.
.beginDialogAction('showCartAction', 'showDinnerCart', {
    matches: /^show cart$/i,
    dialogArgs: {
        showTotal: true;
    }
});

// Show dinner items in cart with the option to show total or not.
bot.dialog('showDinnerCart', function(session, args){
    for(var i = 1; i < session.conversationData.orders.length; i++){
        session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
    }

    if(args && args.showTotal){
        // End this dialog with total.
        session.endDialog(`Your total is: $${session.conversationData.orders[0].Price}`);
    }
    else{
        session.endDialog(); // Ends without a message.
    }
});
```

### <a name="bind-a-reloadaction"></a>Enlazar reloadAction

Enlazar `reloadAction` a un cuadro de diálogo lo registrará en el cuadro de diálogo. Enlazar esta acción a un cuadro de diálogo hará que el cuadro de diálogo se reinicie cuando se desencadene la acción. Desencadenar esta acción es similar a llamar al método [replaceDialog](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#replacedialog). Esto es útil para implementar la lógica para controlar las expresiones del usuario, como "empezar de nuevo" o para crear [bucles](bot-builder-nodejs-dialog-replace.md#repeat-an-action).

El fragmento de código siguiente muestra cómo enlazar [reloadAction][reloadAction] a un cuadro de diálogo.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will restart the dialog.
.reloadAction('startOver', 'Ok, starting over.', {
    matches: /^start over$/i
});
```

En casos donde deba pasar argumentos adicionales en el cuadro de diálogo recargado, puede agregar una opción [`dialogArgs`](https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idialogactionoptions#dialogargs) a la acción. Esta opción se pasa al parámetro `args`. Si se reescribe el código de ejemplo anterior para recibir un argumento en una acción de recargar, tendrá un aspecto similar al siguiente:

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    function(session, args, next){
        if(args && args.isReloaded){
            // Reload action was triggered.
        }

        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    }
    //...other waterfall steps...
])
// Once triggered, will restart the dialog.
.reloadAction('startOver', 'Ok, starting over.', {
    matches: /^start over$/i,
    dialogArgs: {
        isReloaded: true;
    }
});
```

### <a name="bind-a-cancelaction"></a>Enlazar cancelAction

Enlazar `cancelAction` lo registrará en el cuadro de diálogo. Una vez desencadenada, esta acción finalizará repentinamente el cuadro de diálogo. Una vez que finaliza el diálogo, el cuadro de diálogo primario se reanuda con el código que indica `canceled`. Esta acción le permite controlar las expresiones "no importa" o "cancelar". Si en su lugar necesita cancelar mediante programación un cuadro de diálogo, consulte [Cancel a dialog](bot-builder-nodejs-dialog-replace.md#cancel-a-dialog) (Cancelar un cuadro de diálogo). Para obtener más información sobre *código reanudado*, consulte [Prompt results](bot-builder-nodejs-dialog-prompt.md#prompt-results) (Solicitar resultados). 

El fragmento de código siguiente muestra cómo enlazar [cancelAction][cancelAction] a un cuadro de diálogo.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
//Once triggered, will end the dialog.
.cancelAction('cancelAction', 'Ok, cancel order.', {
    matches: /^nevermind$|^cancel$|^cancel.*order/i
});
```

### <a name="bind-an-endconversationaction"></a>Enlazar endConversationAction

Enlazar `endConversationAction` lo registrará en el cuadro de diálogo. Una vez desencadenada, esta acción finaliza la conversación con el usuario. Desencadenar esta acción es similar a llamar al método [endConversation](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#endconversation). Una vez que finaliza una conversación, el Bot Builder SDK para Node.js borrará la pila del cuadro de diálogo y los datos de estado persistente. Para obtener más información sobre los datos de estado persistente, consulte [Manage state data](bot-builder-nodejs-state.md) (Administrar datos de estado).

El fragmento de código siguiente muestra cómo enlazar [endConversationAction][endConversationAction] a un cuadro de diálogo.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will end the conversation.
.endConversationAction('endConversationAction', 'Ok, goodbye!', {
    matches: /^goodbye$/i
});
```

## <a name="confirm-interruptions"></a>Confirmar las interrupciones

La mayoría, si no todas estas acciones interrumpen el flujo normal de una conversación. Muchas son perjudiciales y deben controlarse con cuidado. Por ejemplo, `triggerAction`, `cancelAction` o `endConversationAction` borrarán la pila del cuadro de diálogo. Si el usuario cometió el error de desencadenar una de estas acciones, tendrá que volver a empezar la tarea. Para asegurarse de que el usuario en verdad tuvo la intención de desencadenar estas acciones, puede agregar una opción `confirmPrompt` a estas acciones. `confirmPrompt` preguntará si el usuario está seguro de cancelar o finalizar la tarea actual. Permite al usuario cambiar de opinión y continuar el proceso.

El fragmento de código siguiente se muestra [cancelAction][cancelAction] con [confirmPrompt](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#confirmprompt) para asegurarse de que el usuario realmente desea cancelar el proceso de pedido.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Confirm before triggering the action.
// Once triggered, will end the dialog. 
.cancelAction('cancelAction', 'Ok, cancel order.', {
    matches: /^nevermind$|^cancel$|^cancel.*order/i,
    confirmPrompt: "Are you sure?"
});
```

Una vez que se desencadena esta acción, le preguntará al usuario "¿Estás seguro?". El usuario tendrá que responder "Sí" para seguir con la acción o "No" para cancelar la acción y continuar donde estaba.

## <a name="next-steps"></a>Pasos siguientes

Las **acciones** le permiten prever las solicitudes del usuario y permiten al bot controlar correctamente esas solicitudes. Muchas de estas acciones son perjudiciales para el flujo de la conversación actual. Si desea permitir que los usuarios tengan la capacidad de salir y reanudar un flujo de conversación, debe guardar el estado del usuario antes de salir. Echemos un vistazo más de cerca a cómo guardar el estado del usuario y administrar datos de estado.

> [!div class="nextstepaction"]
> [Administración de datos de estado](bot-builder-nodejs-state.md)


[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[cancelAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#cancelaction

[reloadAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#reloadaction

[beginDialogAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#begindialogaction

[endConversationAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#endconversationaction

[RecognizeIntent]: bot-builder-nodejs-recognize-intent-messages.md

[CardAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.cardaction#dialogaction

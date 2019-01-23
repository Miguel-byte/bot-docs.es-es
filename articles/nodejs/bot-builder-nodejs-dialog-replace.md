---
title: Reemplazar los cuadros de diálogo | Microsoft Docs
description: Aprenda a reemplazar los diálogos para volver a solicitar la entrada y administrar el flujo de la conversación con Bot Framework SDK para Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 58d2c3fc4a1fb266b74402541fc937f0b52fa189
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224990"
---
# <a name="replace-dialogs"></a>Reemplazar cuadros de diálogo

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

La capacidad de reemplazar un cuadro de diálogo puede ser útil cuando necesita validar la entrada del usuario o repetir una acción durante el transcurso de una conversación. Con Bot Framework SDK para Node.js, puede reemplazar un diálogo usando el método [`session.replaceDialog`](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#replacedialog). Este método le permite finalizar el diálogo actual y reemplazarlo con un nuevo cuadro de diálogo sin tener que volver a la persona que llama. 

## <a name="create-custom-prompts-to-validate-input"></a>Crear mensajes personalizados para validar la entrada

Bot Framework SDK para Node.js incluye la validación de entradas para algunos tipos de [avisos](bot-builder-nodejs-dialog-prompt.md) como `Prompts.time` y `Prompts.choice`. Para validar la entrada de texto que recibe en respuesta a `Prompts.text`, debe crear su propia lógica de validación y solicitudes personalizadas. 

Es posible que quiera validar una entrada si esta debe cumplir con un determinado valor, patrón, rango o criterio que usted mismo haya definido. Si una entrada no puede hacer la validación, el bot puede solicitar al usuario esa información nuevamente mediante el método `session.replaceDialog`.

En el siguiente ejemplo de código, se muestra cómo crear un aviso personalizado para validar la entrada del usuario para un número de teléfono.

```javascript
// This dialog prompts the user for a phone number. 
// It will re-prompt the user if the input does not match a pattern for phone number.
bot.dialog('phonePrompt', [
    function (session, args) {
        if (args && args.reprompt) {
            builder.Prompts.text(session, "Enter the number using a format of either: '(555) 123-4567' or '555-123-4567' or '5551234567'")
        } else {
            builder.Prompts.text(session, "What's your phone number?");
        }
    },
    function (session, results) {
        var matched = results.response.match(/\d+/g);
        var number = matched ? matched.join('') : '';
        if (number.length == 10 || number.length == 11) {
            session.userData.phoneNumber = number; // Save the number.
            session.endDialogWithResult({ response: number });
        } else {
            // Repeat the dialog
            session.replaceDialog('phonePrompt', { reprompt: true });
        }
    }
]);
```

En este ejemplo, inicialmente se solicita al usuario que proporcione su número de teléfono. La lógica de validación utiliza una expresión regular que coincide con un rango de dígitos de la entrada del usuario. Si la entrada contiene 10 u 11 dígitos, ese número se devuelve en la respuesta. De lo contrario, el método `session.replaceDialog` se ejecuta para repetir el cuadro de diálogo `phonePrompt`, que se encargará de solicitar al usuario que realice nuevamente la entrada, esta vez proporcionando información más específica con respecto al formato que debe tener la entrada que se espera.

Cuando llame al método `session.replaceDialog`, especifique el nombre del cuadro de diálogo a repetir y una lista de argumentos. En este ejemplo, la lista de argumentos contiene `{ reprompt: true }`, lo que permite que el bot proporcione mensajes de solicitud diferentes dependiendo de si se trata de un aviso inicial o una repetición de la solicitud, pero puede especificar los argumentos que el bot necesite. 

## <a name="repeat-an-action"></a>Repetir una acción

Puede haber ocasiones en el curso de una conversación en las que desee repetir un diálogo para permitir al usuario completar una determinada acción varias veces. Por ejemplo, si el bot ofrece una variedad de servicios, inicialmente puede mostrar el menú de servicios, guiar al usuario a través del proceso de solicitud de un servicio y, a continuación, volver a mostrar el menú de servicios, lo que permite al usuario solicitar otro servicio. Para lograr esto, puede usar el método [`session.replaceDialog`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#replacedialog) para volver a mostrar el menú de servicios, en lugar de finalizar la conversación con el método ['session.endConversation`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#endconversation). 

En el siguiente ejemplo se muestra cómo usar el método `session.replaceDialog` para implementar este tipo de escenario. En primer lugar, se define el menú de servicios:

```javascript
// Main menu
var menuItems = { 
    "Order dinner": {
        item: "orderDinner"
    },
    "Dinner reservation": {
        item: "dinnerReservation"
    },
    "Schedule shuttle": {
        item: "scheduleShuttle"
    },
    "Request wake-up call": {
        item: "wakeupCall"
    },
}
```

El diálogo predeterminado invoca al diálogo `mainMenu`, por lo que el menú se presentará al usuario al comienzo de la conversación. Además, se adjunta un [`triggerAction`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction) al diálogo `mainMenu` para que el menú también se muestre siempre que la entrada del usuario sea "menú principal". Cuando se muestra el menú al usuario y este selecciona una opción, se invoca el diálogo que corresponde a la selección del usuario mediante el método `session.beginDialog`.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a reservation bot that has a menu of offerings.
var bot = new builder.UniversalBot(connector, [
    function(session){
        session.send("Welcome to Contoso Hotel and Resort.");
        session.beginDialog("mainMenu");
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

// Display the main menu and start a new request depending on user input.
bot.dialog("mainMenu", [
    function(session){
        builder.Prompts.choice(session, "Main Menu:", menuItems);
    },
    function(session, results){
        if(results.response){
            session.beginDialog(menuItems[results.response.entity].item);
        }
    }
])
.triggerAction({
    // The user can request this at any time.
    // Once triggered, it clears the stack and prompts the main menu again.
    matches: /^main menu$/i,
    confirmPrompt: "This will cancel your request. Are you sure?"
});
```

En este ejemplo, si el usuario elige la opción 1 para especificar que la cena se entregue en su habitación, se invocará el cuadro de diálogo `orderDinner` y se guiará al usuario a través del proceso para pedir la cena. Al final del proceso, el bot confirmará el orden y mostrará el menú principal nuevamente mediante el método `session.replaceDialog`.

```javascript
// Menu: "Order dinner"
// This dialog allows user to order dinner to be delivered to their hotel room.
bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function (session, results) {
        if (results.response) {
            var order = dinnerMenu[results.response.entity];
            var msg = `You ordered: %(Description)s for a total of $${order.Price}.`;
            session.dialogData.order = order;
            session.send(msg);
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${results.response}.`;
            session.send(msg);
            session.replaceDialog("mainMenu"); // Display the menu again.
        }
    }
])
.reloadAction(
    "restartOrderDinner", "Ok. Let's start over.",
    {
        matches: /^start over$/i
    }
)
.cancelAction(
    "cancelOrder", "Type 'Main Menu' to continue.", 
    {
        matches: /^cancel$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

Dos desencadenadores se adjuntan al cuadro de diálogo `orderDinner` para permitir al usuario "comenzar de nuevo" o "cancelar" el pedido en cualquier momento durante el proceso. 

El primer desencadenador es [`reloadAction`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#reloadaction), el cual permite al usuario comenzar nuevamente el proceso de pedido enviando la entrada "Comenzar de nuevo". Cuando el desencadenador coincide con el enunciado "empezar de nuevo", `reloadAction` reinicia el diálogo desde el principio. 

El segundo desencadenador es [`cancelAction`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#cancelaction), el cual permite al usuario anular el proceso de pedido por completo al enviar la entrada "cancelar". Este desencadenador no muestra de nuevo el menú principal automáticamente, sino que envía un mensaje que le indica al usuario qué hacer; por ejemplo, mostrando el mensaje "Escriba" 'Menú principal' para continuar".

### <a name="dialog-loops"></a>Bucles del cuadro de diálogo 

En el ejemplo anterior, el usuario solo puede seleccionar un elemento en función del pedido. Es decir, si el usuario desea pedir dos elementos del menú, deberá completar todo el proceso de pedido para el primer artículo y, a continuación, repetir todo el proceso nuevamente para pedir el segundo artículo. 

En el siguiente ejemplo se muestra cómo mejorar el bot anterior refactorizando el menú de la cena en un diálogo separado. Al hacerlo, el bot puede mostrar el menú de la cena en un bucle y, por lo tanto, permite al usuario seleccionar varios elementos en un solo pedido.

En primer lugar, agregue la opción "Comprar" al menú. Esta opción permitirá al usuario salir del proceso de selección de artículos y continuar con el proceso de compra.

```javascript
// The dinner menu
var dinnerMenu = { 
    //...other menu items...,
    "Check out": {
        Description: "Check out",
        Price: 0 // Order total. Updated as items are added to order.
    }
};
```

A continuación, refactorice la solicitud de pedido en su propio cuadro de diálogo para que el bot pueda repetir el menú y permitir al usuario agregar varios elementos a su pedido.

```javascript
// Add dinner items to the list by repeating this dialog until the user says `check out`. 
bot.dialog("addDinnerItem", [
    function(session, args){
        if(args && args.reprompt){
            session.send("What else would you like to have for dinner tonight?");
        }
        else{
            // New order
            // Using the conversationData to store the orders
            session.conversationData.orders = new Array();
            session.conversationData.orders.push({ 
                Description: "Check out",
                Price: 0
            })
        }
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function(session, results){
        if(results.response){
            if(results.response.entity.match(/^check out$/i)){
                session.endDialog("Checking out...");
            }
            else {
                var order = dinnerMenu[results.response.entity];
                session.conversationData.orders[0].Price += order.Price; // Add to total.
                var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
                session.send(msg);
                session.conversationData.orders.push(order);
                session.replaceDialog("addDinnerItem", { reprompt: true }); // Repeat dinner menu
            }
        }
    }
])
.reloadAction(
    "restartOrderDinner", "Ok. Let's start over.",
    {
        matches: /^start over$/i
    }
);
```

En este ejemplo, los pedidos se almacenan en un almacén de datos del bot cuyo ámbito se basa en la conversación actual, `session.conversationData.orders`. Cada vez que se hace un pedido, la variable se vuelve a iniciar con una nueva matriz, y cada vez que el usuario elige un artículo, el bot lo agrega a la matriz `orders` y añade el precio al total a pagar, que se almacena en la variable `Price` del proceso de compra. Cuando el usuario termina de seleccionar artículos en su pedido, puede decir "comprar" y continuar con el resto del proceso de pedido.

> [!NOTE]
> Para obtener más información sobre el almacenamiento de datos del bot, consulte [Administrar datos de estado](bot-builder-nodejs-state.md). 

Por último, actualice el segundo paso de cascada que se encuentra dentro del cuadro de diálogo `orderDinner`, para procesar el pedido con la confirmación. 

```javascript
// Menu: "Order dinner"
// This dialog allows user to order dinner and have it delivered to their room.
bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        session.beginDialog("addDinnerItem");
    },
    function (session, results) {
        if (results.response) {
            // Display itemize order with price total.
            for(var i = 1; i < session.conversationData.orders.length; i++){
                session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
            }
            session.send(`Your total is: $${session.conversationData.orders[0].Price}`);

            // Continue with the check out process.
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${results.response}`;
            session.send(msg);
            session.replaceDialog("mainMenu");
        }
    }
])
//...attached triggers...
;
```

## <a name="cancel-a-dialog"></a>Cancelar un cuadro de diálogo

Si bien el método `session.replaceDialog` se puede usar para reemplazar el cuadro de diálogo *actual* con uno nuevo, no se puede usar para reemplazar un cuadro de diálogo ubicado más abajo en la pila de diálogos. Para reemplazar un diálogo dentro de la pila de diálogos que no sea el diálogo *actual*, use el método [`session.cancelDialog`](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#canceldialog) en su lugar. 

El método `session.cancelDialog` se puede usar para finalizar un diálogo independientemente de dónde esté en la pila de diálogos y, opcionalmente, se puede invocar un nuevo diálogo en su lugar. Para llamar al método `session.cancelDialog`, especifique el id. del diálogo para cancelarlo y, opcionalmente, especifique el id. del diálogo que se invocará en su lugar. Por ejemplo, este fragmento de código cancela el diálogo `orderDinner` y lo reemplaza con el cuadro de diálogo `mainMenu`:

```javascript
session.cancelDialog('orderDinner', 'mainMenu'); 
```

Cuando se llama al método `session.cancelDialog`, la pila de diálogos se examinará hacia atrás y se cancelará el primer resultado que coincida con ese diálogo; gracias a esto, ese diálogo y sus diálogos secundarios (si los hay) se eliminarán de la pila de diálogos. El control será devuelto al diálogo de llamadas, el cual puede comprobar si hay un código [`results.resumed`](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#resumed) igual a [`ResumeReason.notCompleted`](http://docs.botframework.com/en-us/node/builder/chat-reference/enums/_botbuilder_d_.resumereason.html#notcompleted) para detectar la cancelación.

Como alternativa a la opción de especificar el id. del diálogo para cancelarlo cuando se llame al método `session.cancelDialog`, puede especificar en su lugar el índice basado en cero del diálogo que se va a cancelar, donde el índice representa la posición del diálogo en la pila de diálogos. Por ejemplo, el siguiente fragmento de código finaliza el diálogo activo (índice = 0) y comienza el diálogo `mainMenu` en su lugar. El diálogo `mainMenu` se invoca en la posición 0 de la pila de diálogos, convirtiéndose así en el nuevo cuadro de diálogo predeterminado.

```javascript
session.cancelDialog(0, 'mainMenu');
```

Le recomendamos que mire la muestra que se discutió anteriormente en los [bucles de diálogo ](#dialog-loops). Cuando el usuario llega al menú de selección de elementos, ese diálogo (`addDinnerItem`) es el cuarto diálogo de la pila de diálogos: `[default dialog, mainMenu, orderDinner, addDinnerItem]`. ¿Cómo se puede permitir que el usuario cancele su pedido desde el cuadro de diálogo `addDinnerItem`? Si adjunta un desencadenador `cancelAction` al cuadro de diálogo `addDinnerItem`, solo llevará al usuario al cuadro de diálogo anterior (`orderDinner`), así que el usuario accederá nuevamente al cuadro de diálogo `addDinnerItem`.

Aquí es donde el `session.cancelDialog` método resulta útil. Comenzando con el [ejemplo de bucles de diálogo](#dialog-loops), agregue "Cancelar pedido" como una opción explícita dentro del menú de la cena.

```javascript
// The dinner menu
var dinnerMenu = { 
    //...other menu items...,
    "Check out": {
        Description: "Check out",
        Price: 0      // Order total. Updated as items are added to order.
    },
    "Cancel order": { // Cancel the order and back to Main Menu
        Description: "Cancel order",
        Price: 0
    }
};
```

A continuación, actualice el cuadro de diálogo `addDinnerItem` para comprobar que la solicitud "Cancelar pedido" aparece. Si se detecta la opción "Cancelar", utilice el método `session.cancelDialog` para cancelar el cuadro de diálogo predeterminado (es decir, el cuadro de diálogo en el índice 0 de la pila) e invoque el cuadro de diálogo `mainMenu` en su lugar. 

```javascript
// Add dinner items to the list by repeating this dialog until the user says `check out`. 
bot.dialog("addDinnerItem", [
    //...waterfall steps...,
    // Last step
    function(session, results){
        if(results.response){
            if(results.response.entity.match(/^check out$/i)){
                session.endDialog("Checking out...");
            }
            else if(results.response.entity.match(/^cancel/i)){
                // Cancel the order and start "mainMenu" dialog.
                session.cancelDialog(0, "mainMenu");
            }
            else {
                //...add item to list and prompt again...
                session.replaceDialog("addDinnerItem", { reprompt: true }); // Repeat dinner menu.
            }
        }
    }
])
//...attached triggers...
;
```

Al usar el método `session.cancelDialog` de esta manera, puede implementar cualquier flujo de conversación que requiera su bot.

## <a name="next-steps"></a>Pasos siguientes

Como puede ver, para reemplazar los diálogos en la pila, debe usar varios tipos de **acciones** para realizar esa tarea. Estas acciones le ofrecen una gran flexibilidad a la hora de gestionar el flujo de la conversación. Echemos un vistazo más de cerca a las **acciones** y a cómo controlar mejor las acciones de los usuarios.

> [!div class="nextstepaction"]
> [Control de acciones del usuario](bot-builder-nodejs-dialog-actions.md)

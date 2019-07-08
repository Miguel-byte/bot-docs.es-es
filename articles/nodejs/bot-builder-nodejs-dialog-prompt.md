---
title: Petición de datos de entrada al usuario | Microsoft Docs
description: Aprenda a usar avisos para recopilar información del usuario con Bot Framework SDK para Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1cad11c8b1dde800543c919ab579b0112e7d3036
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404992"
---
# <a name="prompt-for-user-input"></a>Petición de datos de entrada al usuario

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Bot Framework SDK para Node.js proporciona un conjunto de avisos integrados para simplificar la recopilación de información de un usuario. 

Cada vez que un bot necesita que intervenga el usuario, se usa un *aviso*. Para pedir al usuario una serie de datos de entrada, se pueden encadenar avisos en una cascada. Puede usar avisos junto con la [cascada](bot-builder-nodejs-dialog-waterfall.md) como ayuda para [administrar el flujo de conversación](bot-builder-nodejs-manage-conversation-flow.md) del bot. 

Este artículo le ayudará a comprender cómo funcionan los avisos y cómo usarlos para recopilar información de los usuarios.

## <a name="prompts-and-responses"></a>Avisos y respuestas

Siempre que necesite la intervención de un usuario, puede enviar un aviso, esperar a que el usuario proporcione los datos de entrada y, luego, procesar la información y enviar una respuesta al usuario.

El siguiente código de ejemplo pide al usuario su nombre y responde con un mensaje de saludo.

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

Con esta construcción básica, puede modelar el flujo de conversación y agregar tantos avisos y respuestas como el bot necesite.

## <a name="prompt-results"></a>Resultados de los avisos 

Los avisos integrados se implementan como [diálogos](bot-builder-nodejs-dialog-overview.md) que devuelven la respuesta del usuario en el campo `results.response`. En el caso de objetos JSON, las respuestas se devuelven en el campo `results.response.entity`. Cualquier tipo de [controlador de diálogo](bot-builder-nodejs-dialog-overview.md#dialog-handlers) puede recibir el resultado de un aviso. Una vez que el bot recibe una respuesta, puede usarla o pasarla de nuevo al diálogo de llamada mediante la invocación del método [`session.endDialogWithResult`][EndDialogWithResult].

En el código de ejemplo siguiente se muestra cómo devolver el resultado de un aviso al diálogo de llamada con el método `session.endDialogWithResult`. En este ejemplo, el diálogo `greetings` usa el resultado del aviso que devuelve el diálogo `askName` para saludar al usuario por su nombre.

```javascript
// Ask the user for their name and greet them by name.
bot.dialog('greetings', [
    function (session) {
        session.beginDialog('askName');
    },
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
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

## <a name="prompt-types"></a>Tipos de avisos
Bot Framework SDK para Node.js incluye varios tipos diferentes de avisos integrados. 

|**Tipo de aviso**     | **Descripción** |     
| ------------------ | --------------- |
|[Prompts.text](#promptstext) | Pide al usuario que escriba una cadena de texto. |     
|[Prompts.confirm](#promptsconfirm) | Pide al usuario que confirme una acción.| 
|[Prompts.number](#promptsnumber) | Pide al usuario que escriba un número.     |
|[Prompts.time](#promptstime) | Pide al usuario una hora o la fecha y la hora.      |
|[Prompts.choice](#promptschoice) | Pide al usuario que elija de una lista de opciones.    |
|[Prompts.attachment](#promptsattachment) | Pide al usuario que cargue una imagen o un vídeo.|       

En las secciones siguientes se proporcionan detalles adicionales sobre cada tipo de aviso.

### <a name="promptstext"></a>Prompts.text

Use el método [Prompts.text()][PromptsText] para pedir al usuario una **cadena de texto**. El aviso devuelve la respuesta del usuario como [IPromptTextResult][IPromptTextResult].

```javascript
builder.Prompts.text(session, "What is your name?");
```

### <a name="promptsconfirm"></a>Prompts.confirm

Use el método [Prompts.confirm()][PromptsConfirm] para pedir al usuario que confirme una acción con una respuesta **sí o no**. El aviso devuelve la respuesta del usuario como [IPromptConfirmResult](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptconfirmresult.html).

```javascript
builder.Prompts.confirm(session, "Are you sure you wish to cancel your order?");
```

### <a name="promptsnumber"></a>Prompts.number

Use el método [Prompts.number()][PromptsNumber] para pedir al usuario un **número**. El aviso devuelve la respuesta del usuario como [IPromptNumberResult](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptnumberresult.html).

```javascript
builder.Prompts.number(session, "How many would you like to order?");
```

### <a name="promptstime"></a>Prompts.time

Use el método [Prompts.time()][PromptsTime] para pedir al usuario una **hora** o una **fecha y hora**. El aviso devuelve la respuesta del usuario como [IPromptTimeResult](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html). El marco usa la biblioteca [Chrono](https://github.com/wanasit/chrono) para analizar la respuesta del usuario y admite tanto respuestas relativas (por ejemplo, "en 5 minutos)" como no relativas (p. ej., "6 de junio a las 2 de la tarde").

El campo [results.response](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html#response), que representa la respuesta del usuario, contiene un objeto [entity](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ientity.html) que especifica la fecha y la hora. Para resolver la fecha y hora en un objeto `Date` de JavaScript, use el método [EntityRecognizer.resolveTime()](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#resolvetime).

> [!TIP] 
> La hora que especifica el usuario se convierte a la hora UTC en función de la zona horaria del servidor que hospeda el bot. Como el servidor puede estar ubicado en una zona horaria diferente a la del usuario, asegúrese de tener en cuenta las zonas horarias. Para convertir la fecha y hora a la hora local del usuario, considere la posibilidad de pedir al usuario que indique en qué zona horaria se encuentra.

```javascript
bot.dialog('createAlarm', [
    function (session) {
        session.dialogData.alarm = {};
        builder.Prompts.text(session, "What would you like to name this alarm?");
    },
    function (session, results, next) {
        if (results.response) {
            session.dialogData.name = results.response;
            builder.Prompts.time(session, "What time would you like to set an alarm for?");
        } else {
            next();
        }
    },
    function (session, results) {
        if (results.response) {
            session.dialogData.time = builder.EntityRecognizer.resolveTime([results.response]);
        }

        // Return alarm to caller  
        if (session.dialogData.name && session.dialogData.time) {
            session.endDialogWithResult({ 
                response: { name: session.dialogData.name, time: session.dialogData.time } 
            }); 
        } else {
            session.endDialogWithResult({
                resumed: builder.ResumeReason.notCompleted
            });
        }
    }
]);
```

### <a name="promptschoice"></a>Prompts.choice

Use el método [Prompts.choice()][PromptsChoice] para pedir al usuario que **elija de una lista de opciones**. Para transmitir su selección, el usuario puede escribir el número asociado con la opción que haya elegido o escribir el nombre de la opción que elija. Se admiten coincidencias totales y parciales del nombre de la opción. El aviso devuelve la respuesta del usuario como [IPromptChoiceResult](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptchoiceresult.html). 

Para especificar el estilo de la lista que se presenta al usuario, establezca la propiedad [IPromptOptions.listStyle](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptoptions.html#liststyle). En la tabla siguiente se muestran los valores de enumeración `ListStyle` de esta propiedad.


Los valores de enumeración `ListStyle` son los siguientes:

| Índice | NOMBRE | DESCRIPCIÓN |
| ---- | ---- | ---- |
| 0 | None | No se representa ninguna lista. Se usa cuando la lista se incluye como parte del aviso. |
| 1 | inline | Las opciones se representan como una lista alineada del formulario "1. rojo, 2. verde, o 3. azul". |
| 2 | list | Las opciones se representan como una lista numerada. |
| 3 | button | Las opciones se representan como botones para canales que admiten botones. Para otros canales se representan como texto. |
| 4 | auto | El estilo se selecciona automáticamente según el canal y el número de opciones. | 

Puede acceder a esta enumeración desde el objeto `builder`, o bien puede proporcionar un índice para elegir un elemento `ListStyle`. Por ejemplo, ambas instrucciones del siguiente fragmento de código realizan la misma cosa.

```javascript
// ListStyle passed in as Enum
builder.Prompts.choice(session, "Which color?", "red|green|blue", { listStyle: builder.ListStyle.button });

// ListStyle passed in as index
builder.Prompts.choice(session, "Which color?", "red|green|blue", { listStyle: 3 });
```

Para especificar la lista de opciones, puede usar una cadena delimitada por barras verticales (`|`), una matriz de cadenas o un mapa de objetos.

Una cadena delimitada por barras verticales: 

```javascript
builder.Prompts.choice(session, "Which color?", "red|green|blue");
```

Una matriz de cadenas:

```javascript
builder.Prompts.choice(session, "Which color?", ["red","green","blue"]);
```

Un mapa de objetos: 

```javascript
var salesData = {
    "west": {
        units: 200,
        total: "$6,000"
    },
    "central": {
        units: 100,
        total: "$3,000"
    },
    "east": {
        units: 300,
        total: "$9,000"
    }
};

bot.dialog('getSalesData', [
    function (session) {
        builder.Prompts.choice(session, "Which region would you like sales for?", salesData); 
    },
    function (session, results) {
        if (results.response) {
            var region = salesData[results.response.entity];
            session.send(`We sold ${region.units} units for a total of ${region.total}.`); 
        } else {
            session.send("OK");
        }
    }
]);
```

### <a name="promptsattachment"></a>Prompts.attachment

Use el método [Prompts.attachment()][PromptsAttachment] para pedir al usuario que cargue un archivo, como una imagen o un vídeo. El aviso devuelve la respuesta del usuario como [IPromptAttachmentResult](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptattachmentresult.html).

```javascript
builder.Prompts.attachment(session, "Upload a picture for me to transform.");
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe cómo dirigir a los usuarios por los pasos de una cascada y pedirles información, vamos a ver algunas formas de ayudarle a administrar mejor el flujo de conversación.

> [!div class="nextstepaction"]
> [Administración de flujo de conversación](bot-builder-nodejs-dialog-manage-conversation-flow.md)


[SendAttachments]: bot-builder-nodejs-send-receive-attachments.md
[SendCardWithButtons]: bot-builder-nodejs-send-rich-cards.md
[RecognizeUserIntent]: bot-builder-nodejs-recognize-intent-messages.md
[SaveUserData]: bot-builder-nodejs-save-user-data.md

[UniversalBot]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
[ChatConnector]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[Session]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session


[SendTyping]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping

[EndDialogWithResult]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#enddialogwithresult

[IPromptResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html

[Result_Response]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#response

[ResumeReason]: https://docs.botframework.com/node/builder/chat-reference/enums/_botbuilder_d_.resumereason.html

[Result_Resumed]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#resumed

[entity]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ientity.html

[ResolveTime]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#resolvetime

[PromptsRef]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html

[PromptsText]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#text

[IPromptTextResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttextresult.html

[PromptsConfirm]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#confirm

[IPromptConfirmResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptconfirmresult.html

[PromptsNumber]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#number

[IPromptNumberResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptnumberresult.html

[PromptsTime]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#time

[IPromptTimeResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html

[PromptsChoice]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#choice

[IPromptChoiceResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptchoiceresult.html

[PromptsAttachment]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#attachment

[IPromptAttachmentResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptattachmentresult.html

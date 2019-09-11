---
title: Introducción a los diálogos | Microsoft Docs
description: Aprenda a usar diálogos en Bot Framework SDK para Node.js para modelar conversaciones y administrar el flujo de conversación.
author: DucVo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 9ec846c5d464821538902dd726e8b9ee68d4bae1
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299868"
---
# <a name="dialogs-in-the-bot-framework-sdk-for-nodejs"></a>Diálogos en Bot Framework SDK para Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-dialogs.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-overview.md)

Los diálogos de Bot Framework SDK para Node.js le permiten modelar las conversaciones y administrar el flujo de conversación. Un bot se comunica con un usuario a través de conversaciones. Las conversaciones se organizan en diálogos. Los diálogos pueden contener preguntas y pasos en cascada. A medida que el usuario interactúa con el bot, el bot inicia, detiene y cambia entre diferentes diálogos en respuesta a los mensajes del usuario. La comprensión del funcionamiento de los diálogos es clave para diseñar y crear correctamente bots excelentes. 

En este artículo se presentan los conceptos de diálogo. Después de leer este artículo, siga los vínculos de la sección [Pasos siguientes](#next-steps) para profundizar más en estos conceptos.

## <a name="conversations-through-dialogs"></a>Conversaciones a través de diálogos

Bot Framework SDK para Node.js define una conversación como la comunicación entre un bot y un usuario a través de uno o varios diálogos. Un diálogo, en su nivel más básico, es un módulo reutilizable que realiza una operación o recopila información de un usuario. Puede englobar la lógica compleja del bot en un código de diálogo reutilizable.

Una conversación se puede estructurar y cambiar de muchas maneras:

- Puede originarse desde el [diálogo predeterminado](#default-dialog).
- Se puede redirigir desde un diálogo a otro.
- Se puede reanudar.
- Puede seguir un patrón de [cascada](bot-builder-nodejs-dialog-waterfall.md), que guía al usuario a través de un conjunto de pasos o [pregunta](bot-builder-nodejs-dialog-prompt.md) al usuario una serie de cuestiones.
- Puede usar [acciones](bot-builder-nodejs-dialog-actions.md) que escuchan palabras o frases que desencadenan un diálogo distinto. 

Puede pensar en una conversación como un padre en diálogos. Por lo tanto, una conversación contiene una *pila de diálogos* y mantiene su propio conjunto de datos de estado; concretamente, `conversationData` y `privateConversationData`. Un diálogo, por otro parte, mantiene `dialogData`. Para obtener más información sobre los datos de estado, consulte [Administración de datos de estado](bot-builder-nodejs-state.md).

## <a name="dialog-stack"></a>Pila de diálogos

Un bot interactúa con un usuario a través de una serie de diálogos que se mantienen en una pila de diálogos. Los diálogos se insertan y se extraen de la pila en el transcurso de una conversación. La pila funciona como una pila LIFO normal; es decir, el último diálogo agregado será el primero en completarse. Una vez completado un cuadro de diálogo, el control se devuelve al diálogo anterior en la pila.

Cuando se inicia una conversación de bot por primera vez o cuando finaliza una conversación, la pila de diálogos está vacía. En este momento, cuando el usuario envía un mensaje al bot, el bot responde con el *diálogo predeterminado*.

## <a name="default-dialog"></a>Diálogo predeterminado

Antes de la versión 3.5 de Bot Framework, un diálogo *raíz* se definía mediante la adición de un diálogo denominado `/`, que conduce a convenciones de nomenclatura similares a las de las direcciones URL. Esta convención de nomenclatura no era adecuada para la nomenclatura de los diálogos. 

> [!NOTE]
> A partir de la versión 3.5 de Bot Framework, el *diálogo predeterminado* se registra como el segundo parámetro del constructor de [`UniversalBot`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html#constructor).  

En el fragmento de código siguiente se muestra cómo definir el diálogo predeterminado al crear el objeto `UniversalBot`.

```javascript
var bot = new builder.UniversalBot(connector, [
    //...Default dialog waterfall steps...
    ]);
```

El diálogo predeterminado se ejecuta siempre que la pila de diálogos está vacía y ningún otro cuadro de diálogo se [desencadena](bot-builder-nodejs-dialog-actions.md) a través de LUIS u otro [reconocedor](bot-builder-nodejs-recognize-intent-messages.md). Como el diálogo predeterminado es la primera respuesta del bot al usuario, dicho diálogo debe proporcionar información contextual al usuario, como una lista de los comandos disponibles o una introducción a lo que puede hacer el bot.

## <a name="dialog-handlers"></a>Controladores de diálogos

El controlador de diálogos administra el flujo de una conversación. Para avanzar a través de una conversación, el controlador de diálogos dirige el flujo iniciando y finalizando diálogos. 

## <a name="starting-and-ending-dialogs"></a>Inicio y finalización de diálogos

Para iniciar un diálogo nuevo (e insertarlo en la pila), utilice [`session.beginDialog()`](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#begindialog). Para finalizar un diálogo (y quitarlo de la pila, devolviendo el control al diálogo que realiza la llamada), use [`session.endDialog()`](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#enddialog) o [`session.endDialogWithResult()`](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#enddialogwithresult). 

## <a name="using-waterfalls-and-prompts"></a>Uso de cascadas y preguntas

La [cascada](bot-builder-nodejs-dialog-waterfall.md) es una manera sencilla de modelar y administrar el flujo de conversación. Una cascada contiene una secuencia de pasos. En cada paso, puede completar una acción en nombre del usuario o [pedir](bot-builder-nodejs-dialog-prompt.md) información al usuario.

Una cascada se implementa mediante un diálogo que se compone de una colección de funciones. Cada función define un paso de la cascada. En el ejemplo de código siguiente se muestra una conversación simple que usa una cascada de dos pasos para preguntar al usuario cómo se llama y saludarlo por su nombre.

```javascript
// Ask the user for their name and greet them by name.
bot.dialog('greetings', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

Cuando un bot llega al final de la cascada sin finalizar el diálogo, el siguiente mensaje del usuario reiniciará ese diálogo en el primer paso de la cascada. Esto puede provocar frustración, ya que el usuario puede sentir que está atrapado en un bucle. Para evitar esta situación, cuando una conversación o un diálogo llega a su fin, es recomendable llamar explícitamente a `endDialog`, `endDialogWithResult` o `endConversation`.

## <a name="next-steps"></a>Pasos siguientes

Para profundizar más en los diálogos, es importante entender cómo funciona el patrón de cascada y cómo usarlo para guiar a los usuarios a través de un proceso.

> [!div class="nextstepaction"]
> [Definición de los pasos de la conversación con cascadas](bot-builder-nodejs-dialog-waterfall.md)

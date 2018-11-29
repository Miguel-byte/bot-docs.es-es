---
title: Uso del botón para la entrada de datos | Microsoft Docs
description: Aprenda a enviar acciones sugeridas dentro de mensajes mediante Bot Builder SDK para JavaScript.
keywords: acciones sugeridas, botones, entrada adicional
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/08/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 10b9fa9664e8c18cdc5dcd2fcf3ae400296a4abb
ms.sourcegitcommit: 6c719b51c9e4e84f5642100a33fe346b21360e8a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/28/2018
ms.locfileid: "52451987"
---
# <a name="use-button-for-input"></a>Uso del botón para la entrada de datos

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Puede habilitar el bot para que presente botones en los que el usuario puede pulsar para proporcionar una entrada. Los botones mejoran la experiencia del usuario al permitir que este responda una pregunta o haga una selección simplemente pulsando un botón, en lugar de tener que escribir una respuesta con un teclado. A diferencia de los botones que aparecen en las tarjetas enriquecidas (que permanecen visibles y accesibles para el usuario incluso después de que se pulsen), los botones que aparecen en el panel de acciones sugeridas desaparecerán una vez que el usuario haya hecho una selección. Esto evita que el usuario pulse botones obsoletos dentro de una conversación y simplifica el desarrollo de bots (ya que no necesitará tener en cuenta ese escenario). 

## <a name="suggest-action-using-button"></a>Acción sugerida con un botón

Las *acciones sugeridas* permiten al bot presentar botones. Puede crear una lista de acciones sugeridas (también conocidas como "respuestas rápidas") que se mostrarán al usuario para un único turno de la conversación: 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Puede acceder al código fuente que se emplea aquí desde [GitHub](https://aka.ms/SuggestedActionsCSharp)

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply("What is your favorite color?");

reply.SuggestedActions = new SuggestedActions()
{
    Actions = new List<CardAction>()
    {
        new CardAction() { Title = "Red", Type = ActionTypes.ImBack, Value = "Red" },
        new CardAction() { Title = "Yellow", Type = ActionTypes.ImBack, Value = "Yellow" },
        new CardAction() { Title = "Blue", Type = ActionTypes.ImBack, Value = "Blue" },
    },

};
await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
Puede acceder al código fuente que se emplea aquí desde [GitHub](https://aka.ms/SuggestActionsJS).

```javascript
const { ActivityTypes, MessageFactory, TurnContext } = require('botbuilder');

async sendSuggestedActions(turnContext) {
    var reply = MessageFactory.suggestedActions(['Red', 'Yellow', 'Blue'], 'What is the best color?');
    await turnContext.sendActivity(reply);
}
```

---

## <a name="additional-resources"></a>Recursos adicionales

Puede acceder al código fuente completo que aparece aquí desde GitHub [[C#](https://aka.ms/SuggestedActionsCSharp) | [JS](https://aka.ms/SuggestActionsJS)].

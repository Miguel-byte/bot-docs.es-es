---
title: Envío de mensajes | Microsoft Docs
description: Obtenga información sobre cómo enviar mensajes con Bot Builder SDK.
keywords: envío de mensajes, actividades de mensajes, mensaje de texto simple, voz, mensaje hablado
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 08/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2e2c5f54d4ca077ad2b916787613f782779707ac
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996792"
---
# <a name="send-text-and-spoken-messages"></a>Envío de mensajes de texto y de voz

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

La forma principal comunicación del bot con los usuarios y de recepción de comunicación es mediante actividades de **mensaje**. Algunos mensajes pueden constar simplemente de texto sin formato, mientras que otros pueden tener un contenido más rico, como tarjetas o datos adjuntos. El controlador de turnos del bot recibe mensajes del usuario y puede enviar respuestas al usuario desde ahí. El objeto de contexto de turnos proporciona métodos para enviar mensajes al usuario. Para más información acerca del procesamiento de actividades en general, consulte [Procesamiento de actividades](~/v4sdk/bot-builder-basics.md#the-activity-processing-stack).

Este artículo describe cómo enviar mensajes de voz y de texto simple. Para enviar contenido más rico, consulte el procedimiento de [Incorporación de datos adjuntos multimedia enriquecidos](bot-builder-howto-add-media-attachments.md). Para más información sobre cómo usar los objetos de aviso, consulte cómo [pedir datos de entrada a los usuarios](bot-builder-prompts.md).

## <a name="send-a-simple-text-message"></a>Envío de un mensaje de texto simple

Para enviar un mensaje de texto simple, especifique la cadena que desea enviar como la actividad:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En el método **OnTurn** del bot, utilice el método **SendActivityAsync** del objeto de contexto de turnos para enviar una respuesta de mensaje única. También puede usar el método **SendActivitiesAsync** del objeto para enviar varias respuestas a la vez.

```cs
await context.SendActivityAsync("Greetings from sample message.");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el controlador de turnos del bot, utilice el método **SendActivity** del objeto de contexto de turno para enviar una respuesta de mensaje única. También puede usar el método **sendActivities** del objeto para enviar varias respuestas a la vez.

```javascript
await context.sendActivity("Greetings from sample message.");
```

---

## <a name="send-a-spoken-message"></a>Envío de un mensaje de voz

Algunos canales admiten bots habilitados para voz, lo que les permite hablar con el usuario. Un mensaje puede tener contenido escrito y hablado.

> [!NOTE]
> Para los canales que no admiten voz, se omite el contenido de voz.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Usar el parámetro opcional **speak** para proporcionar el texto que se dirá como parte de la respuesta.

```cs
await context.SendActivityAsync(
    "This is the text to be displayed.",
    "This is the text to be spoken.");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Para agregar la voz, necesitará `Microsoft.Bot.Builder.MessageFactory` para construir el mensaje. `MessageFactory` se usa más con [elementos multimedia enriquecidos](bot-builder-howto-add-media-attachments.md), donde se explica más, aunque solo lo utilizaremos aquí.

```javascript
// Require MessageFactory from botbuilder
const {MessageFactory} = require('botbuilder');

const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.');
await context.sendActivity(basicMessage);
```

---

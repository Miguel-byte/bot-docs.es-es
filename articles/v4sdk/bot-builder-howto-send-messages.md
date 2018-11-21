---
title: Envío y recepción de mensajes de texto | Microsoft Docs
description: Obtenga información sobre cómo enviar y recibir mensajes de texto con el SDK Bot Builder.
keywords: enviar mensaje, actividades de mensaje, mensaje de texto simple, mensaje, mensaje de texto, recibir mensaje
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/08/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a01a64e032acfde2b3711e3efbb3886439888c42
ms.sourcegitcommit: 5c40e2e21adb3a779022d45704c29cf11ed7f4a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/10/2018
ms.locfileid: "51506200"
---
# <a name="send-and-receive-text-message"></a>Envío y recepción de mensajes de texto 

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

La forma principal comunicación del bot con los usuarios y de recepción de comunicación es mediante actividades de **mensaje**. Algunos mensajes pueden constar simplemente de texto sin formato, mientras que otros pueden tener un contenido más rico, como tarjetas o datos adjuntos. El controlador de turnos del bot recibe mensajes del usuario y puede enviar respuestas al usuario desde ahí. El objeto de contexto de turnos proporciona métodos para enviar mensajes al usuario. Este artículo describe cómo enviar mensajes de texto simples.

## <a name="send-a-text-message"></a>Enviar un mensaje de texto

Para enviar un mensaje de texto simple, especifique la cadena que desea enviar como la actividad:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En el método `OnTurnAsync` del bot, utilice el método `SendActivityAsync` del objeto de contexto de turno para enviar un único mensaje de respuesta. También puede usar el método `SendActivitiesAsync` del objeto para enviar varias respuestas a la vez.

```cs
await turnContext.SendActivityAsync($"Welcome!");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el controlador `onTurn` del bot, utilice el método `sendActivity` del objeto de contexto de turno para enviar un único mensaje de respuesta. También puede usar el método `sendActivities` del objeto para enviar varias respuestas a la vez.

```javascript
await context.sendActivity("Welcome!");
```
---
## <a name="receive-a-text-message"></a>Recibir un mensaje de texto

Para recibir un mensaje de texto simple, utilice la propiedad *text* del objeto *activity*. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En el método `OnTurnAsync` del bot, use el siguiente código para recibir un mensaje. 

```cs
var responseMessage = turnContext.Activity.Text;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el método `OnTurnAsync` del bot, use el siguiente código para recibir un mensaje. 
```javascript
let text = turnContext.activity.text;
```
---


## <a name="additional-resources"></a>Recursos adicionales
Para más información acerca del procesamiento de actividades en general, consulte [Procesamiento de actividades](~/v4sdk/bot-builder-basics.md#the-activity-processing-stack). Para enviar contenido más rico, consulte el procedimiento de incorporación de datos adjuntos [multimedia enriquecidos](bot-builder-howto-add-media-attachments.md).

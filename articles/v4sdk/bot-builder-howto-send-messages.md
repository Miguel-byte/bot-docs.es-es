---
title: Envío y recepción de mensajes de texto | Microsoft Docs
description: Obtenga información sobre cómo enviar y recibir mensajes de texto con Bot Framework SDK.
keywords: enviar mensaje, actividades de mensaje, mensaje de texto simple, mensaje, mensaje de texto, recibir mensaje
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/16/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0d4279df31aba6cecb12b7d8d7262069aed8836b
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033390"
---
# <a name="send-and-receive-text-message"></a>Envío y recepción de mensajes de texto

[!INCLUDE[applies-to](../includes/applies-to.md)]

La forma principal comunicación del bot con los usuarios y de recepción de comunicación es mediante actividades de **mensaje**. Algunos mensajes pueden constar simplemente de texto sin formato, mientras que otros pueden tener un contenido más rico, como tarjetas o datos adjuntos. El controlador de turnos del bot recibe mensajes del usuario y puede enviar respuestas al usuario desde ahí. El objeto de contexto de turnos proporciona métodos para enviar mensajes al usuario. Este artículo describe cómo enviar mensajes de texto simples.

Markdown es compatible con la mayoría de los campos de texto, pero la compatibilidad puede variar según el canal.

## <a name="send-a-text-message"></a>Enviar un mensaje de texto

Para enviar un mensaje de texto simple, especifique la cadena que desea enviar como la actividad:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En los controladores de actividades del bot, utilice el método `SendActivityAsync` del objeto de contexto de turno para enviar un único mensaje de respuesta. También puede usar el método `SendActivitiesAsync` del objeto para enviar varias respuestas a la vez.

```cs
await turnContext.SendActivityAsync($"Welcome!");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En los controladores de actividades del bot, utilice el método `sendActivity` del objeto de contexto de turno para enviar un único mensaje de respuesta. También puede usar el método `sendActivities` del objeto para enviar varias respuestas a la vez.

```javascript
await context.sendActivity("Welcome!");
```
---
## <a name="receive-a-text-message"></a>Recibir un mensaje de texto

Para recibir un mensaje de texto simple, utilice la propiedad *text* del objeto *activity*. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En los controladores de actividades del bot, use el siguiente código para recibir un mensaje. 

```cs
var responseMessage = turnContext.Activity.Text;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En los controladores de actividades del bot, use el siguiente código para recibir un mensaje.

```javascript
let text = turnContext.activity.text;
```

---

## <a name="additional-resources"></a>Recursos adicionales

- Para más información acerca del procesamiento de actividades en general, consulte [Procesamiento de actividades](~/v4sdk/bot-builder-basics.md#the-activity-processing-stack).
- Para más información sobre el formato, consulte la [sección de actividad de mensajes](https://aka.ms/botSpecs-activitySchema#message-activity) en el esquema de actividad de Bot Framework.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Incorporación de elementos multimedia a los mensajes](./bot-builder-howto-add-media-attachments.md)

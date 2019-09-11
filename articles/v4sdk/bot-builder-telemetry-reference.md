---
title: Eventos generados por los datos de telemetría de Bot Framework Service | Microsoft Docs
description: Obtenga información sobre los eventos que se desencadenan con las nuevas características de telemetría.
keywords: telemetry, appinsights, monitor bot
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: dbb58a6a287124d0b393a4fc5a617e89840d1542
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299159"
---
# <a name="events-generated-by-the-bot-framework-service-telemetry"></a>Eventos generados por los datos de telemetría de Bot Framework Service

## <a name="channel-service-events"></a>Eventos de servicios de canal

Además de `WaterfallDialog`, del que se habló en el [tema sobre telemetría](bot-builder-telemetry.md) y que genera eventos desde el código del bot, el servicio Bot Framework Channel también registra eventos.  Esto ayuda a diagnosticar problemas con los canales o errores de bot generales.

### <a name="customevent-activity"></a>CustomEvent: "Activity"
**Registrado de:** servicio Channel Registrado por el servicio Channel cuando se recibe un mensaje.

### <a name="exception-bot-errors"></a>Excepción: "Bot Errors"
**Registrado de:** servicio Channel Registrado por el servicio Channel cuando una llamada al bot devuelve una respuesta HTTP distinta de 2XX.

## <a name="customevent-waterfallstart"></a>CustomEvent: "WaterfallStart" 

Cuando se inicia un diálogo WaterfallDialog, se registra un evento `WaterfallStart`.

- `user_id` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `session_id` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Este es el identificador de diálogo (cadena) que se pasa a la cascada.  Se puede considerar el "tipo de cascada")
- `customDimensions.InstanceID` (único para cada instancia del diálogo)

## <a name="customevent-waterfallstep"></a>CustomEvent: "WaterfallStep" 

Registra los pasos individuales de un diálogo en cascada.

- `user_id` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `session_id` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Este es el identificador de diálogo (cadena) que se pasa a la cascada.  Se puede considerar el "tipo de cascada")
- `customDimensions.StepName` (nombre del método o `StepXofY` si es una expresión lambda)
- `customDimensions.InstanceID` (único para cada instancia del diálogo)

## <a name="customevent-waterfalldialogcomplete"></a>CustomEvent: "WaterfallDialogComplete"

Registra cuando se completa un diálogo en cascada.

- `user_id` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `session_id` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Este es el identificador de diálogo (cadena) que se pasa a la cascada.  Se puede considerar el "tipo de cascada")
- `customDimensions.InstanceID` (único para cada instancia del diálogo)

## <a name="customevent-waterfalldialogcancel"></a>CustomEvent: "WaterfallDialogCancel" 

Registra cuando se cancela un diálogo en cascada.

- `user_id` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `session_id` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Este es el identificador de diálogo (cadena) que se pasa a la cascada.  Se puede considerar el "tipo de cascada")
- `customDimensions.StepName` (nombre del método o `StepXofY` si es una expresión lambda)
- `customDimensions.InstanceID` (único para cada instancia del diálogo)

## <a name="customevent-botmessagereceived"></a>CustomEvent: BotMessageReceived 
Registra cuando el bot recibe un mensaje nuevo de un usuario.

Si no se reemplaza, este evento se registra desde `Microsoft.Bot.Builder.TelemetryLoggerMiddleware` con el método `Microsoft.Bot.Builder.IBotTelemetry.TrackEvent()`.

- Identificador de sesión  
  - Cuando se usa Application Insights, se registra desde `TelemetryBotIdInitializer` como el identificador de **sesión** (*Temeletry.Context.Session.Id*) utilizado en Application Insights.  
  - Corresponde al [Identificador de conversación](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation) tal como se define en el protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `session_id`.

- Identificador de usuario
  - Cuando se usa Application Insights, se registra desde `TelemetryBotIdInitializer` como el identificador de **usuario** (*Temeletry.Context.User.Id*) utilizado en Application Insights.  
  - El valor de esta propiedad es una combinación de las propiedades [Identificador de canal](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id) e [ de usuario](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) (concatenadas) tal como se define mediante el protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `user_id`.

- Identificador de actividad 
  - Cuando se usa Application Insights, se registra desde `TelemetryBotIdInitializer` como una propiedad en el evento.
  - Corresponde al [Identificador de actividad](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#Id) tal como se define en el protocolo de Bot Framework.
  - El nombre de la propiedad es `activityId`.

- Identificador de canal
  - Cuando se usa Application Insights, se registra desde `TelemetryBotIdInitializer` como una propiedad en el evento.  
  - Corresponde al [Identificador de canal](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `channelId`.

- ActivityType 
  - Cuando se usa Application Insights, se registra desde `TelemetryBotIdInitializer` como una propiedad en el evento.  
  - Corresponde al [Tipo de actividad](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#type) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `activityType`.

- Texto
  - Se registra **opcionalmente** cuando la propiedad `logPersonalInformation` se establece en `true`.
  - Corresponde al campo [Texto de actividad](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#text) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `text`.

- Speak

  - Se registra **opcionalmente** cuando la propiedad `logPersonalInformation` se establece en `true`.
  - Corresponde al campo [Texto hablado de actividad](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#speak) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `speak`.

  - 

- FromId
  - Corresponde al campo [Desde el identificador](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `fromId`.

- FromName
  - Se registra **opcionalmente** cuando la propiedad `logPersonalInformation` se establece en `true`.
  - Corresponde al campo [Desde el nombre](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `fromName`.

- RecipientId
  - Corresponde al campo [Desde el nombre](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `fromName`.

- RecipientName
  - Corresponde al campo [Desde el nombre](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `fromName`.

- ConversationId
  - Corresponde al campo [Desde el nombre](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `fromName`.

- ConversationName
  - Corresponde al campo [Desde el nombre](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `fromName`.

- Configuración regional
  - Corresponde al campo [Desde el nombre](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `fromName`.

## <a name="customevent-botmessagesend"></a>CustomEvent: BotMessageSend 
**Registrado de:** TelemetryLoggerMiddleware 

Registra cuando el bot envía un mensaje.

- UserID  ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- SessionID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Channel ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityType ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ReplyToID
- RecipientId
- ConversationName
- Configuración regional
- RecipientName (opcional para información de identificación personal)
- Text (opcional para información de identificación personal)
- Speak (opcional para información de identificación personal)


## <a name="customevent-botmessageupdate"></a>CustomEvent: BotMessageUpdate
**Registrado de:** TelemetryLoggerMiddleware Registra cuando el bot actualiza un mensaje (poco frecuente)
- UserID  ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- SessionID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Channel ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityType ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- RecipientId
- ConversationId
- ConversationName
- Configuración regional
- Text (opcional para información de identificación personal)


## <a name="customevent-botmessagedelete"></a>CustomEvent: BotMessageDelete
**Registrado de:** TelemetryLoggerMiddleware Registra cuando el bot elimina un mensaje (poco frecuente)
- UserID  ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- SessionID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Channel ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityType ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- RecipientId
- ConversationId
- ConversationName

## <a name="customevent-luisevent"></a>CustomEvent: LuisEvent
**Registrado de:** LuisRecognizer

Registra los resultados del servicio LUIS.

- UserID  ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- SessionID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Channel ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityType ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ApplicationId
- Intención
- IntentScore
- Intent2 
- IntentScore2 
- FromId
- SentimentLabel
- SentimentScore
- Entities (como JSON)
- Question (opcional para información de identificación personal)

## <a name="customevent-qnamessage"></a>CustomEvent: QnAMessage
**Registrado de:** QnAMaker

Registra los resultados del servicio QnA Maker.

- UserID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- SessionID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Channel ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityType ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Username (opcional para información de identificación personal)
- Question (opcional para información de identificación personal)
- MatchedQuestion
- QuestionId
- Respuesta
- Score
- ArticleFound


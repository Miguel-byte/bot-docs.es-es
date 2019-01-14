---
title: Análisis conversacional con PowerBI | Microsoft Docs
description: Aprenda cómo la plantilla del bot empresarial utiliza Application Insights para las conclusiones mediante PowerBI
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 88208a2f5b0eb88d3b2964e63a21585484166d73
ms.sourcegitcommit: 2d84d5d290359ac3cfb8c8f977164f799666f1ab
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/09/2019
ms.locfileid: "54152178"
---
# <a name="enterprise-bot-template---conversational-analytics-using-powerbi-dashboard-and-application-insights"></a>Plantillas del bot empresarial: análisis conversacional con el panel de PowerBI y Application Insights

> [!NOTE]
> Este tema se aplica a la versión v4 del SDK. 

El bot implementado empieza a procesar mensajes y se ve el flujo de datos de telemetría a la instancia de Application Insights del grupo de recursos. 

Estos datos de telemetría se ven en la hoja de Application Insights de Azure Portal y con Log Analytics. Además, PowerBI puede usarlos para proporcionar conclusiones empresariales más generales sobre el uso del bot.

En [Telemetría de Application Insights Conversational](https://aka.ms/botPowerBiTemplate) encontrará un ejemplo de panel de PowerBI. 

Esto es a modo de ejemplo y muestra cómo se pueden empezar a crear conclusiones propias. Con el tiempo, mejoraremos estas presentaciones. 


## <a name="middleware-processing"></a>Procesamiento de middleware

Los contenedores de datos de telemetría en torno a las clases QnAMaker y LuisRecognizer sirven para garantizar una salida de telemetría coherente independientemente del escenario y para habilitar el funcionamiento de un panel estándar en cada proyecto.

```TelemetryLuisRecognizer``` y ```TelemetryQnAMaker``` ofrecen propiedades en el constructor que permiten que los desarrolladores deshabiliten el registro de nombres de usuario y los mensajes originales. Sin embargo, esto reduce la cantidad de información disponible.

## <a name="telemetry-captured"></a>Datos de telemetría capturados

Con ```TelemetryLuisRecognizer``` y ```TelemetryQnAMaker``` se capturan 4 eventos distintos de telemetría que están habilitados de forma predeterminada en la plantilla empresarial. 

Cada intención de LUIS que use el proyecto llevará el prefijo LuisIntent para facilitar la identificación de las intenciones en el panel.

```
-BotMessageReceived
    - ActivityId
    - Channel
    - FromId
    - FromName
    - ConversationId
    - ConversationName
    - Locale
    - Text
    - RecipientId
    - RecipientName
```
  
```
-BotMessageSent
    - ActivityId,
    - Channel
    - RecipientId
    - ConversationId
    - ConversationName
    - Locale
    - RecipientId
    - RecipientName
    - ReplyToId
    - Text
```

```
- LuisIntent.*
    - ActivityId
    - Intent
    - IntentScore
    - SentimentLabel
    - SentimentScore
    - ConversationId
    - Question
    - DialogId
```

```
- QnAMaker
    - ActivityId
    - ConversationId
    - OriginalQuestion
    - FromName
    - ArticleFound
    - Question
    - Answer
    - Score
```
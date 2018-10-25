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
ms.openlocfilehash: 9c5a156ff949c4014b69638e9cd5abb4b0ede149
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999397"
---
# <a name="enterprise-bot-template---conversational-analytics-using-powerbi-dashboard-and-application-insights"></a>Plantillas del bot empresarial: análisis conversacional con el panel de PowerBI y Application Insights

> [!NOTE]
> Este tema se aplica a la versión v4 del SDK. 

El bot implementado empieza a procesar mensajes y se ve el flujo de datos de telemetría a la instancia de Application Insights del grupo de recursos. 

Estos datos de telemetría se ven en la hoja de Application Insights de Azure Portal y con Log Analytics. Además, PowerBI puede usarlos para proporcionar conclusiones empresariales más generales sobre el uso del bot.

En la carpeta de PowerBI del proyecto que ha creado se proporciona un ejemplo de panel de PowerBI. Esto es a modo de ejemplo y muestra cómo se pueden empezar a crear conclusiones propias. Con el tiempo, mejoraremos estas presentaciones. 

## <a name="getting-started"></a>Introducción

- Descargue Power BI Desktop de [aquí](https://powerbi.microsoft.com/en-us/desktop/).
 
- Recupere el valor de ```Application Id``` para el recurso de Application Insights que usa el bot. Para ello, vaya a la página de acceso a la API en la sección de configuración de la hoja de Application Insights en Azure.

Haga doble clic en el archivo de la plantilla de Power BI proporcionado ubicado en la carpeta de PowerBI de la solución. Se le pedirá para el ```Application Id``` que recuperó en el paso anterior. Complete la autenticación si se le solicita con sus credenciales de suscripción de Azure, quizá deba hacer clic en la opción de configuración de la cuenta profesional para iniciar sesión.

El panel resultante está ahora vinculado a su instancia de Application Insights y debería ver información inicial detallada en él si se han enviado y recibido mensajes.

>Tenga en cuenta que la presentación de opiniones no mostrará datos porque el script de implementación actual no permite opiniones al publicar el modelo de LUIS. [Volver a publicar](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-how-to-publish-app) el modelo de LUIS y habilitar las opiniones funcionará.

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
    - Conversationid
    - ConversationName
    - Locale
    - UserName
    - Text
```
  
```
-BotMessageSent
    - ActivityId,
    - Channel
    - RecipientId
    - Conversationid
    - ConversationName
    - Locale
    - ReceipientName
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
```

```
- QnAMaker
    - ActivityId
    - ConversationId
    - OriginalQuestion
    - UserName
    - QnAItemFound
    - Question
    - Answer
    - Score
```
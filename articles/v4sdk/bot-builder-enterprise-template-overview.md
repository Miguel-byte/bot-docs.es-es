---
title: Introducción detallada a la plantilla del bot de empresa | Microsoft Docs
description: Obtenga información sobre las decisiones de diseño de la plantilla del bot de empresa
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 582ec21d36e15fcbaef2d26616a9e55a04d8e5f2
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/07/2019
ms.locfileid: "57568222"
---
# <a name="enterprise-bot-template---overview"></a>Enterprise Bot Template (introducción)

> [!NOTE]
> Este tema se aplica a la versión v4 del SDK. 

Enterprise Bot Template proporciona una base sólida de los procedimientos recomendados y los servicios necesarios para crear una experiencia conversacional, lo que reduce el esfuerzo y eleva el listón de la calidad. La plantilla aprovecha [SDK Bot Builder v4](https://github.com/Microsoft/botbuilder) y las [herramientas de Bot Builder](https://github.com/Microsoft/botbuilder-tools) para proporcionar las siguientes características:

Característica      | Descripción |
------------ | -------------
Introducción | Mensaje de introducción con una [tarjeta adaptable]() en el inicio de conversación
Indicadores de escritura  | Indicadores visuales de escritura automáticos durante las conversaciones y repetidos en operaciones de larga duración
Modelo base de LUIS  | Admite intenciones comunes como **Cancel**, **Help**, **Escalate**, etc.
Diálogos base | Flujos de diálogo para capturar información básica del usuario, así como la lógica de interrupción de las intenciones Cancel y Help
Respuestas base  | Respuestas de texto y de voz de las intenciones y los diálogos base
Preguntas más frecuentes | Integración con [QnA Maker](https://www.qnamaker.ai) para responder preguntas generales desde una base de conocimiento 
Charla | Un modelo de charla profesional para proporcionar respuestas estándar a consultas comunes ([más información](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/chit-chat-knowledge-base))
Distribuidor | Un modelo integrado de [distribución](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig) para identificar si una determinada expresión deben procesarla LUIS o QnA Maker.
Compatibilidad con idiomas | Disponible en inglés, francés, italiano, alemán, español y chino
Transcripciones | Transcripciones de todas las conversaciones almacenadas en Azure Storage
Telemetría  | Integración con [Application Insights](https://azure.microsoft.com/en-gb/services/application-insights/) para recopilar datos de telemetría de todas las conversaciones
Análisis | Un panel de Power BI de ejemplo para empezar a usar información detallada en las conversaciones.
Implementación automatizada | Implementación fácil de los servicios mencionados anteriormente mediante las [herramientas de Bot Builder](https://github.com/Microsoft/botbuilder-tools)

# <a name="background"></a>Fondo

## <a name="introduction-card"></a>Tarjeta de presentación
Las tarjetas de presentación mejoran las conversaciones, ya que proporcionan información general acerca de las funcionalidades del bot y proporcionan expresiones de ejemplo que resultan muy útiles al empezar a trabajar. También establece la personalidad del bot al usuario.

## <a name="base-luis-model-with-common-intents"></a>Modelo base de LUIS con intenciones comunes
El modelo base de LUIS que se incluye en la plantilla abarca una selección de las intenciones más comunes que deben controlar la mayoría de los bots. Las intenciones siguientes están disponibles para usarlas en su bot de inmediato:

Intención       | Expresiones de ejemplo |
-------------|-------------|
Cancelar       |*cancelar*, *no importa*|
Escalate     |*¿puedo hablar con una persona?*|
FinishTask   |*listo*, *todo terminado*|
Goback       |*volver*|
Ayuda         |*¿puede ayudarme?*|
Repetir       |*¿puede repetirlo?*|
SelectAny    |*cualquiera*|
SelectItem   |*el primero*|
SelectNone   |*ninguno*|
ShowNext     |*mostrar más*|
ShowPrevious |*mostrar anterior*|
StartOver    |*restart*|
Stop         |*stop*|

## <a name="qna-maker"></a>QnA Maker

[QnA Maker](https://www.qnamaker.ai/) proporciona a personas que no sean desarrolladores la capacidad de generar una colección de parejas de preguntas y respuestas que se pueden usar en un bot. Estos conocimientos pueden importarse de los orígenes de datos de las preguntas frecuentes y de los manuales de los productos, o bien se pueden crear manualmente en el portal de QnA Maker.

En la plantilla se proporcionan dos modelos de ejemplo de QnA Maker, uno de las preguntas más frecuentes y otro de una [charla profesional](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/chit-chat-knowledge-base). 

## <a name="dispatch"></a>Dispatch

El servicio Dispatch se usa para administrar el enrutamiento entre varios modelos de LUIS y bases de conocimiento de QnA Maker mediante la extracción de expresiones de cada servicio y la creación de un modelo de LUIS de envío central.

Esto permite al bot identificar rápidamente el componente que debe controlar una expresión dada y garantiza que los datos de QnA Maker se consideran en el nivel máximo de procesamiento de intención, en lugar de en la parte más baja de una jerarquía.

Esta herramienta también permite evaluar los modelos, lo que resalta las expresiones que se superponen y las discrepancias entre los servicios.

## <a name="telemetry"></a>Telemetría

Proporcionar información de las conversaciones del bot puede ayudarle a conocer los niveles de involucración de los usuarios, las características que estos usan y las preguntas que el bot no puede controlar.

[Application Insights](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) captura tanto la telemetría operativa del bot de servicio, como los eventos específicos de las operaciones. Estos eventos se pueden agregar a la información accionable mediante herramientas como [Power BI](https://powerbi.microsoft.com/en-us/what-is-power-bi/). Hay disponible un panel de Power BI de ejemplo que se puede usar con Enterprise Bot Template que muestra esta funcionalidad.

# <a name="next-steps"></a>Pasos siguientes
Consulte [Introducción](bot-builder-enterprise-template-getting-started.md) para aprender a crear e implementar Enterprise Bot. 

# <a name="resources"></a>Recursos
El código fuente completo de Enterprise Bot Template está disponible en [GitHub](https://github.com/Microsoft/AI/tree/master/templates/Enterprise-Template).

---
title: Introducción a la plantilla del bot de empresa | Microsoft Docs
description: Más información sobre las funcionalidades de la plantilla del bot de empresa
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 43bc3c7606a12084690d71f8b6ea2dc3b2e5984d
ms.sourcegitcommit: 6c719b51c9e4e84f5642100a33fe346b21360e8a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/28/2018
ms.locfileid: "52451997"
---
# <a name="enterprise-bot-template"></a>Plantilla del bot de empresa 

> [!NOTE]
> Este tema se aplica a la versión v4 del SDK. 

La creación de una experiencia conversacional de alta calidad requiere un conjunto de funcionalidades fundamentales. Para ayudarle a crear satisfactoriamente excelentes experiencias de conversación, hemos creado una plantilla de bot de empresa. Esta plantilla reúne todos los procedimientos recomendados y los componentes auxiliares que hemos identificado mediante la creación de experiencias de conversación. 

Esta plantilla simplifica enormemente la creación de un nuevo proyecto de bot. La plantilla proporcionará las siguientes funcionalidades desde el principio, aprovechando el [SDK Bot Builder v4](https://github.com/Microsoft/botbuilder) y las [herramientas de Bot Builder](https://github.com/Microsoft/botbuilder-tools).

Característica | DESCRIPCIÓN |
------------ | -------------
Mensaje de introducción | Mensaje de introducción con una tarjeta adaptable en el inicio de conversación. Explica las funcionalidades de los bots y proporciona botones para guiar las preguntas iniciales. Los desarrolladores pueden personalizar esto según corresponda.
Indicadores de escritura automatizada  | Envíe indicadores visuales de escritura durante las conversaciones y repita en las operaciones de larga ejecución.
Configuración del archivo controlado por .bot | Toda la información de configuración del bot como, por ejemplo, LUIS, puntos de conexión de Dispatcher, Application Insights, etc está encapsulada en el archivo .bot y se usa para controlar el inicio del bot.
Intenciones de conversación básicas  | Intenciones básicas (saludo, despedida, ayuda, cancelar, etc.) en inglés, francés, italiano, alemán, español y chino. Estas se proporcionan en los archivos .LU (reconocimiento de lenguaje) y permiten una fácil modificación.
Respuestas de conversación básicas  | Respuestas a las intenciones de conversación básicas extraídas en clases de vistas independientes. Estas evolucionarán en los nuevos archivos de generación de lenguaje (LG) del futuro.
Detección de contenido inadecuado o información de identificación personal (PII)  |Detecte datos inadecuados o información de identificación personal en las conversaciones entrantes mediante el uso de [Content Moderator](https://azure.microsoft.com/en-us/services/cognitive-services/content-moderator/) en un componente de middleware.
Transcripciones  | Transcripciones de todas las conversaciones almacenadas en Azure Storage
Distribuidor | Un modelo integrado de [distribución](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig) para identificar si una determinada expresión debe procesarla LUIS + Code o enviarla a QnA Maker.
Integración con QnA Maker  | Integración con [QnA Maker](https://www.qnamaker.ai) para dar respuesta a preguntas generales desde una base de conocimientos que puede aprovechar orígenes de datos existentes (por ejemplo, manuales PDF).
Conclusiones de conversación  | Integración con [Application Insights](https://azure.microsoft.com/en-gb/services/application-insights/) para recopilar datos de telemetría de todas las conversaciones y un panel de Power BI de ejemplo para ayudarle a comenzar con las conclusiones sobre sus experiencias de conversación.

Además, se implementan automáticamente todos los recursos de Azure necesarios para el bot: el registro del bot, Azure App Service, LUIS, QnA Maker, Content Moderator, CosmosDB, Azure Storage y Application Insights. Además, se crean modelos básicos de LUIS, QnA Maker y Dispatch, y se entrenan y publican para permitir la realización inmediata de pruebas de las intenciones básicas y el enrutamiento.

Una vez que se crea la plantilla y se ejecutan los pasos de implementación se puede presionar F5 para realizar una prueba completa. Esto proporciona una base sólida desde la que iniciar la experiencia de conversación, lo cual reduce el esfuerzo de varios días que necesitaba cada proyecto, y aumenta la calidad de la conversación.

Para empezar, vaya a [Creación del proyecto](bot-builder-enterprise-template-create-project.md). Para más información acerca de los procedimientos recomendados y los conocimientos que han impulsado las características anteriores, consulte el tema [detalles de la plantilla](bot-builder-enterprise-template-overview-detail.md). 

El código fuente completo para la plantilla del bot de empresa está disponible en esta [ubicación de GitHub](https://github.com/Microsoft/AI/tree/master/templates/Enterprise-Template)

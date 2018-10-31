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
ms.openlocfilehash: 2ee47d472518e54cf07b86648e270e40f8f015c0
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997232"
---
# <a name="enterprise-template---detailed-overview"></a>Introducción detallada a la plantilla de empresa

> [!NOTE]
> Este tema se aplica a la versión v4 del SDK. 

La plantilla del bot de empresa reúne una serie de procedimientos recomendados que hemos identificado mediante la creación de experiencias de conversación y automatiza la integración de los componentes que creemos muy ventajosos para los desarrolladores de Azure Bot Service. En esta sección se cubre información de fondo de las decisiones clave para ayudar a explicar el funcionamiento de la plantilla.

## <a name="introduction-card"></a>Tarjeta de presentación

Uno de los problemas clave que se produce en muchas experiencias conversacionales es que los usuarios finales no saben cómo empezar, lo que produce preguntas generales para las cuales el bot no tiene las mejores respuestas. Las primeras impresiones son importantes Una tarjeta de presentación ofrece la oportunidad de mostrar las funcionalidades del bot a un usuario final y le sugiere preguntas iniciales con las que puede empezar. También es una excelente oportunidad para exponer la personalidad del bot.

Se proporciona una tarjeta de presentación estándar sencilla que se puede adaptar según sea necesario.

## <a name="basic-language-understanding-luis-intents"></a>Intenciones básicas de Language Understanding (LUIS)

Todos los bots deben ser capaces de reconocer un nivel básico de idioma en la conversación. Por ejemplo, los saludos son algo básico que todos los bots deben controlar con facilidad. Normalmente, los desarrolladores necesitan crear estas intenciones base y proporcionar datos de entrenamiento inicial para comenzar. La plantilla del bot de empresa ofrece archivos de LU de ejemplo para empezar y evita que los proyectos deban crearlos cada ve, al tiempo que garantiza un nivel base de funcionalidad lista para el uso.

Los archivos de LU proporcionan las siguientes intenciones en inglés, francés, italiano, alemán y español.

> Greeting, Help, Cancel, Restart, Escalate, ConfirmYes, ConfirmNo, ConfirmMore, Next, Goodbye

El formato de [LU](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) es similar a Markdown, lo que facilita la modificación y el control de código fuente. Después, la herramienta [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) convierte los archivos .LU en modelos de LUIS para que los pueda publicar en su suscripción de LUIS ya sea mediante el portal o la herramienta de la CLI de [LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS) (línea de comandos) asociada.

## <a name="content-moderator"></a>Content Moderator

Content Moderator habilita la detección de posibles blasfemias y ayuda a comprobar los datos de carácter personal (DCP). Esto puede resultar útil para la integración en los bots para que reaccionen a las blasfemias o si el usuario comparte datos de carácter personal. Por ejemplo, un bot se puede disculpar y rechazar a una persona o puede no guardar registros de telemetría si detecta información de carácter personal.

Se proporciona un componente de middleware que supervisa los textos y las superficies con ```TextModeratorResult``` en el objeto TurnState.

## <a name="telemetry"></a>Telemetría

Proporcionar información sobre la involucración de los usuarios del Bot es muy importante. Esta información puede ayudarle a comprender los niveles de implicación de los usuarios, qué características del bot usan (intenciones) y las preguntas que las personas formulan que el bot no es capaz de responder (destacan las lagunas de conocimiento del bot que podrían solventarse con nuevos artículos de QnAMaker, por ejemplo).

La integración de Application Insights proporciona datos técnicos/operativos importantes desde el principio, pero también puede usarse para capturar los eventos específicos del bot: mensajes enviados y recibidos y las operaciones de LUIS y QnAMaker.

La telemetría a nivel de bot está intrínsecamente vinculada a los datos de telemetría técnicos y operativos, por lo que podrá supervisar la respuesta que se envió a un usuario y su pregunta.

La combinación de un componente de middleware con una clase de contenedor en torno a las clases de SDK de QnAMaker y LuisRecognizer supone una forma elegante de recopilar un conjunto coherente de eventos. Las herramientas de Application Insights pueden usar estos eventos coherentes con herramientas como PowerBI.

Con cada proyecto creado con la plantilla del bot de empresa se proporciona un panel de PowerBI de ejemplo. Consulte la sección de [PowerBI](bot-builder-enterprise-template-powerbi.md) para más información.

## <a name="dispatcher"></a>Distribuidor

Un modelo de diseño clave utilizado para mejorar el rendimiento de la primera ola de experiencias conversacionales era aprovechar Language Understanding (LUIS) y QnAMaker. LUIS podría entrenarse con tareas que el bot pudiera hacer para el usuario final y QnAMaker se entrenaría con conocimiento más general.

Todas las grabaciones de voz entrantes (preguntas) se enrutarían a LUIS para el análisis. Si la intención de una grabación de voz no se identificara, se marcaría como None. Entonces se usaría QnAMaker para intentar encontrar una respuesta para el usuario final.

Si bien este modelo funcionó bien, dos escenarios clave pueden suponer problemas.

- Si las grabaciones de voz del modelo LUIS y QnAMaker se solapan ligeramente (en ocasiones ocurre), esto provocaría un comportamiento extraño en el que LUIS intentaría procesar una pregunta que debería haberse dirigido a QnAMaker.
- Con dos o más modelos LUIS, el bot tendría que invocarlos a todos y realizar una especie de comparación de evaluación de las intenciones para determinar dónde enviar la grabación de voz concreta. Como no existe comparación eficaz de puntuación de referencia común entre modelos, la experiencia del usuario empeora.

El [distribuidor](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig) es una solución elegante a en este caso, ya que extrae las grabaciones de voz de los modelos LUIS configurados y las preguntas de QnAMaker y crea un modelo LUIS de distribución central.

Así, el bot puede identificar rápidamente el modelo LUIS o el componente que debe administrar una grabación determinada y garantiza que los datos de QnAMaker se consideran en el máximo nivel de intención, sin procesarlos únicamente como con intención None como antes.

Esta herramienta de distribución también facilita la evaluación al resaltar la confusión y superponer los modelos LUIS y las bases de conocimiento de QnAMaker para destacar los problemas antes de la implementación.

El distribuidor se usa en el núcleo de cada proyecto que se crea con la plantilla del bot de empresa. El modelo de distribuidor se usa en la clase `MainDialog` para identificar si el destino es un modelo de LUIS o QnA. En el caso de LUIS, se invoca el modelo LUIS secundario para que se devuelva la intención y las entidades como de costumbre.

## <a name="qnamaker"></a>QnAMaker

[QnAMaker](https://www.qnamaker.ai/) ofrece a personas que no sean desarrolladores conocimientos generales en forma de parejas de preguntas y respuestas. Este conocimiento puede importarse desde orígenes de datos de preguntas frecuentes, manuales de productos e interactivamente desde el portal de QnaMaker.

En la carpeta de QnA de CogSvcModels se proporciona un conjunto de entradas de QnA de ejemplo en formato de archivo [LU](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md). Después, se usa [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) como parte del script de implementación para crear un archivo JSON de QnAMaker que la CLI de [QnAMaker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker) (línea de comandos) usará después para publicar elementos en la base de conocimiento de QnAMaker.

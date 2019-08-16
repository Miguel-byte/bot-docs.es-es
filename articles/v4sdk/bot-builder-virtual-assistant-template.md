---
title: Introducción a la plantilla de Virtual Assistant | Microsoft Docs
description: Más información acerca de la plantilla de Virtual Assistant
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6bf567729e0c4799672f773ddcfadb4fabfa36fc
ms.sourcegitcommit: 7b3d2b5b9b8ce77887a9e6124a347ad798a139ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2019
ms.locfileid: "68991949"
---
# <a name="virtual-assistant---template-outline"></a>Virtual Assistant: esquema de la plantilla

> [!NOTE]
> Este tema se aplica a la versión v4 del SDK. 

La plantilla de Virtual Assistant reúne una serie de procedimientos recomendados que hemos identificado mediante la creación de experiencias de conversación y automatiza la integración de los componentes que creemos muy ventajosos para los desarrolladores de Bot Framework. En esta sección se cubre información de fondo de las decisiones clave para ayudar a explicar el funcionamiento de la plantilla.

Característica      | DESCRIPCIÓN |
------------ | -------------
Introducción | Mensaje de introducción con una [tarjeta adaptable]() en el inicio de conversación
Indicadores de escritura  | Indicadores visuales de escritura automáticos durante las conversaciones y repetidos en operaciones de larga duración
Modelo base de LUIS  | Admite intenciones comunes como **Cancel**, **Help**, **Escalate**, etc.
Diálogos base | Flujos de diálogo para capturar información básica del usuario, así como la lógica de interrupción de las intenciones Cancel y Help
Respuestas base  | Respuestas de texto y de voz de las intenciones y los diálogos base
Preguntas más frecuentes | Integración con [QnA Maker](https://www.qnamaker.ai) para responder preguntas generales desde una base de conocimiento 
Charla | Un modelo de charla profesional para proporcionar respuestas estándar a consultas comunes ([más información](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/chit-chat-knowledge-base))
Distribuidor | Un modelo integrado de [distribución](https://docs.microsoft.com/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig) para identificar si una determinada expresión deben procesarla LUIS o QnA Maker.
Compatibilidad con idiomas | Disponible en inglés, francés, italiano, alemán, español y chino
Transcripciones | Transcripciones de todas las conversaciones almacenadas en Azure Storage
Telemetría  | Integración con [Application Insights](https://azure.microsoft.com/services/application-insights/) para recopilar datos de telemetría de todas las conversaciones
Análisis | Un panel de Power BI de ejemplo para empezar a usar información detallada en las conversaciones.
Implementación automatizada | Sencilla implementación de los servicios mencionados mediante las plantillas de Azure Resource Manager.

## <a name="introduction-card"></a>Tarjeta de presentación

Uno de los problemas clave que se produce en muchas experiencias conversacionales es que los usuarios finales no saben cómo empezar, lo que produce preguntas generales para las cuales el bot no tiene las mejores respuestas. Las primeras impresiones son importantes Una tarjeta de presentación ofrece la oportunidad de mostrar las funcionalidades del bot a un usuario final y le sugiere preguntas iniciales con las que puede empezar. También es una excelente oportunidad para exponer la personalidad del bot.

De forma estándar se proporciona una tarjeta de presentación sencilla que se puede adaptar tanto como sea necesario y en las interacciones posteriores, cuando el usuario haya completado el diálogo de incorporación (que aparece al hacer clic en el botón de primeros pasos de la tarjeta de presentación), se muestra una tarjeta de usuario que ha vuelto

![Ejemplo de introducción de tarjeta](./media/enterprise-template/vabotintrocard.png)

## <a name="basic-language-understanding-luis-intents"></a>Intenciones básicas de Language Understanding (LUIS)

Todos los bots deben ser capaces de reconocer un nivel básico de idioma en la conversación. Por ejemplo, los saludos son algo básico que todos los bots deben controlar con facilidad. Normalmente, los desarrolladores necesitan crear estas intenciones base y proporcionar datos de entrenamiento inicial para comenzar. La plantilla de Virtual Assistant proporciona archivos .lu de ejemplo con los que se puede empezar a trabajar y evita que todos los proyectos tengan que crearlos cada vez, al tiempo que garantiza un nivel base de funcionalidad de forma estándar.

Los archivos .lu proporcionan las siguientes intenciones en inglés, chino, francés, italiano, alemán y español.

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

El formato [.lu](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) es similar a Markdown, lo que facilita la modificación y el control de código fuente. Después, la herramienta [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) convierte los archivos .lu en modelos de LUIS para que los pueda publicar en su suscripción de LUIS ya sea mediante el portal o la herramienta de la CLI de [LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS) (línea de comandos) asociada.

## <a name="telemetry"></a>Telemetría

Proporcionar información sobre la involucración de los usuarios del Bot es muy importante. Esta información puede ayudarle a comprender los niveles de implicación de los usuarios, qué características del bot usan (intenciones) y las preguntas que las personas formulan que el bot no es capaz de responder (destacan las lagunas de conocimiento del bot que podrían solventarse con nuevos artículos de QnAMaker, por ejemplo).

La integración de Application Insights proporciona datos técnicos y operativos importantes desde el principio, pero también se puede usar para capturar eventos relacionados con un bot concreto: mensajes enviados y recibidos, y las operaciones de LUIS y QnA Maker.

La telemetría a nivel de bot está intrínsecamente vinculada a los datos de telemetría técnicos y operativos, por lo que podrá supervisar la respuesta que se envió a un usuario y su pregunta.

La combinación de un componente de middleware con una clase de contenedor en torno a las clases de SDK de QnAMaker y LuisRecognizer supone una forma elegante de recopilar un conjunto coherente de eventos. Las herramientas de Application Insights pueden usar estos eventos coherentes con herramientas como PowerBI.

Un panel de Power BI de ejemplo que forma parte del repositorio de Bot Framework Solutions y funciona de forma estándar con todas las plantillas de Virtual Assistant. Para más información, consulte la sección de [Analytics](https://aka.ms/bfsanalytics).

![Ejemplo de Analytics](./media/enterprise-template/powerbi-conversationanalytics-luisintents.png)

## <a name="dispatcher"></a>Distribuidor

Un patrón de diseño clave que se usa para mejorar el rendimiento de la primera ola de experiencias conversacionales era aprovechar Language Understanding (LUIS) y QnA Maker. LUIS podría entrenarse con tareas que el bot pudiera hacer para el usuario final y QnA Maker se entrenaría con conocimiento más general.

Todas las grabaciones de voz entrantes (preguntas) se enrutarían a LUIS para el análisis. Si la intención de una grabación de voz no se identificara, se marcaría como *None*. Entonces se usaría QnA Maker para intentar encontrar una respuesta para el usuario final.

Si bien este modelo funcionó bien, dos escenarios clave pueden suponer problemas.

- Si las grabaciones de voz del modelo LUIS y QnA Maker se solapan ligeramente (en ocasiones ocurre), esto provocaría un comportamiento extraño en el que LUIS intentaría procesar una pregunta que debería haberse dirigido a QnA Maker.
- Con dos o más modelos LUIS, el bot tendría que invocarlos a todos y realizar una especie de comparación de evaluación de las intenciones para determinar dónde enviar la grabación de voz concreta. Como no existe comparación eficaz de puntuación de referencia común entre modelos, la experiencia del usuario empeora.

El [distribuidor](https://docs.microsoft.com/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig) es una solución elegante en este caso, ya que extrae las grabaciones de voz de los modelos LUIS configurados y las preguntas de QnA Maker y crea un modelo LUIS de distribución central.

Así, el bot puede identificar rápidamente el modelo LUIS o el componente que debe administrar una grabación determinada y garantiza que los datos de QnA Maker se consideran en el máximo nivel de intención, sin procesarlos únicamente como con intención *None* como antes.

Esta herramienta de distribución también facilita la evaluación al resaltar la confusión, los problemas y superponer los modelos LUIS y las bases de conocimiento de QnA Maker antes de la implementación.

Dispatcher se usa en el núcleo de cada proyecto que se crea con la plantilla. El modelo de distribuidor se usa en la clase `MainDialog` para identificar si el destino es un modelo de LUIS o QnA. En el caso de LUIS, se invoca el modelo LUIS secundario para que se devuelva la intención y las entidades. Dispatcher también se usa para la detección de interrupciones.

![Ejemplo de Dispatch](./media/enterprise-template/dispatchexample.png)

## <a name="qna-maker"></a>QnA Maker

[QnA Maker](https://www.qnamaker.ai/) ofrece a personas que no sean desarrolladores conocimientos generales en forma de parejas de preguntas y respuestas. Este conocimiento puede importarse desde orígenes de datos de preguntas frecuentes, manuales de productos e interactivamente desde el portal de QnA Maker.

Se proporcionan dos modelos de QnA Maker de ejemplo en el formato de archivo [.lu](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) en la carpeta QnA de CognitiveModels, uno de preguntas frecuentes y otro de charla. Después, se usa [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) como parte del script de implementación para crear un archivo JSON de QnA Maker que la CLI de [QnA Maker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker) (línea de comandos) usará después para publicar elementos en la base de conocimiento de QnA Maker.

![Ejemplo de QnA ChitChat](./media/enterprise-template/qnachitchatexample.png)

## <a name="content-moderator"></a>Content Moderator

Content Moderator es un componente opcional que permite la detección de posibles blasfemias y ayuda a comprobar los datos de carácter personal (DCP). Por ejemplo, un bot se puede disculpar y rechazar a una persona en caso de palabras soeces o puede no guardar registros de telemetría si detecta información de carácter personal.

Se proporciona un componente de middleware que supervisa los textos y las superficies con ```TextModeratorResult``` en el objeto TurnState.

## <a name="next-steps"></a>Pasos siguientes
Consulte los [tutoriales](https://aka.ms/bfstutorials) para aprender a crear e implementar Virtual Assistant. 

## <a name="additional-resources"></a>Recursos adicionales
El código fuente completo de la plantilla de Virtual Assistant se puede encontrar en [GitHub](https://aka.ms/bfsolutions).


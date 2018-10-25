---
title: Plantillas de Bot Service | Microsoft Docs
description: Obtenga información sobre las diferentes plantillas que puede usar al crear un bot con Bot Service.
author: robstand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: ce7b82d6d3bffe57d758370525cdd7dcc6cddea6
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999802"
---
# <a name="bot-service-templates"></a>Plantillas de Bot Service

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Bot Service incluye cinco plantillas para ayudarle a empezar con la creación de bots. Estas plantillas proporcionan un bot totalmente funcional listo para usar que le ayudará a empezar a trabajar rápidamente. Al [crear un bot](bot-service-quickstart.md), se elige una plantilla y el lenguaje del SDK del bot.

Cada plantilla proporciona un punto de partida que se basa en una característica fundamental para un bot. 

## <a name="basic-bot"></a>Bot básico
Para crear un bot que usa cuadros de diálogo para responder a la entrada del usuario, elija la plantilla Básica. La plantilla **Básica** crea un bot que tiene el conjunto mínimo de archivos y código para empezar a trabajar. El bot reproduce al usuario lo que escribe. Puede usar esta plantilla para empezar a crear el flujo de conversación en el bot.

## <a name="form-bot"></a>Bot de formulario
Para crear un bot que recopila datos de entrada de un usuario mediante una conversación guiada, elija la plantilla **Formulario**. Un bot de formulario está diseñado para recopilar un conjunto específico de información del usuario. Por ejemplo, un bot que está diseñado para obtener el pedido de sándwich de un usuario necesitaría recopilar información como el tipo de pan, la elección de ingredientes, el tamaño del sándwich y así sucesivamente.

Los bots creados con la plantilla de formulario en lenguaje C# utilizan [FormFlow](dotnet/bot-builder-dotnet-formflow.md) para administrar los formularios y los bots creados con la plantilla de formulario en lenguaje Node.js usan [cascadas](nodejs/bot-builder-nodejs-dialog-waterfall.md) para administrar los formularios.

## <a name="language-understanding-bot"></a>Bot de comprensión del lenguaje
Para crear un bot que usa modelos de lenguaje natural para comprender la intención del usuario, elija la plantilla **Comprensión del lenguaje**. Esta plantilla aprovecha las ventajas de <a href="https://www.luis.ai" target="_blank">Language Understanding (LUIS)</a> para proporcionar comprensión del lenguaje natural.

Si un usuario envía un mensaje, como "obtener noticias acerca de las compañías de realidad virtual", el bot puede usar LUIS para interpretar el significado del mensaje. Con LUIS, puede implementar rápidamente un punto de conexión HTTP que interpretará la entrada del usuario en cuanto a la intención que transmite (buscar noticias) y las entidades clave que están presentes (las compañías de realidad virtual). LUIS le permite especificar el conjunto de intenciones y entidades que son relevantes para la aplicación y, a continuación, le guía por el proceso de creación de una aplicación de comprensión del lenguaje.

Cuando se crea un bot con la plantilla de comprensión del lenguaje, Bot Service crea una aplicación de LUIS correspondiente que está vacía (es decir, que siempre devuelve `None`). Para actualizar el modelo de la aplicación de LUIS para que sea capaz de interpretar la entrada del usuario, inicie sesión en <a href="https://www.luis.ai" target="_blank">LUIS</a>, haga clic en **Mis aplicaciones**, seleccione la aplicación que el servicio creó y, a continuación, cree las intenciones, especifique las entidades y entrene la aplicación.

## <a name="question-and-answer-bot"></a>Bot de preguntas y respuestas
Para crear un bot que desglosa datos semiestructurados, como pares de preguntas y respuestas en respuestas distintas y útiles, elija la plantilla **Preguntas y respuestas**. Esta plantilla aprovecha el servicio <a href="https://qnamaker.ai">QnA Maker</a> para analizar las preguntas y proporcionar respuestas. 

Cuando se crea un bot con la plantilla de preguntas y respuestas, debe iniciar sesión en <a href="https://qnamaker.ai">QnA Maker</a> y crear un nuevo servicio de QnA. Este servicio de QnA le proporcionará el identificador de la base de conocimiento y la clave de suscripción que puede usar para actualizar los valores de la [Configuración de la aplicación](bot-service-manage-settings.md) para **QnAKnowldegebasedId** y **QnASubscriptionKey**. Una vez que se establecen esos valores, el bot puede responder a las preguntas que el servicio QnA tiene en su base de conocimiento.

## <a name="proactive-bot"></a>Bot proactivo
Para crear un bot que pueda enviar mensajes proactivos al usuario, elija la **plantilla Proactiva**. Normalmente, cada mensaje que un bot envía al usuario se relaciona directamente con la anterior entrada del usuario. En algunos casos, sin embargo, un bot podría necesitar enviar al usuario información que no está directamente relacionada con el mensaje más reciente del usuario. Este tipo de mensajes se llaman **mensajes proactivos**. Los mensajes proactivos pueden ser útiles en una variedad de escenarios. Por ejemplo, si un bot establece un temporizador o un recordatorio, es posible que deba notificar al usuario cuando llegue la hora. O bien, si un bot recibe una notificación sobre un evento externo, es posible que deba comunicar esa información al usuario. 

Cuando se crea un bot con la plantilla proactiva, se crean automáticamente varios recursos de Azure y se agregan al grupo de recursos. De forma predeterminada, estos recursos de Azure ya están configurados para habilitar un escenario de mensajería proactiva muy sencillo. 

| Recurso | DESCRIPCIÓN |
|----|----|
| Azure Storage | Se utiliza para crear la cola. |
| Aplicación de función de Azure | Una función de Azure `queueTrigger` que se desencadena cuando hay un mensaje en la cola. Se comunica con Bot Service mediante [Direct Line](https://docs.microsoft.com/bot-framework/rest-api/bot-framework-rest-direct-line-3-0-concepts). Esta función usa enlace de bot para enviar el mensaje como parte de la carga del desencadenador. Nuestra función de ejemplo reenvía el mensaje del usuario tal como viene de la cola.
| Servicio de bots | El bot. Contiene la lógica que recibe el mensaje del usuario, agrega el mensaje a la cola de Azure, recibe los desencadenadores de la función de Azure y devuelve el mensaje que recibió en la carga del desencadenador. |

El siguiente diagrama muestra cómo funcionan los eventos desencadenados al crear un bot con la plantilla proactiva.

![Información general del ejemplo de bot proactivo](~/media/bot-proactive-diagram.png)

El proceso comienza cuando el usuario envía un mensaje al bot mediante servidores de Bot Framework (paso 1).

## <a name="next-steps"></a>Pasos siguientes
Ahora que conoce las diferentes plantillas, obtenga información sobre el uso de Cognitive Services en bots.

> [!div class="nextstepaction"]
> [Cognitive Services para bots](bot-service-concept-intelligence.md)

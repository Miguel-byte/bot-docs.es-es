---
title: Conversaciones en Bot Builder SDK | Microsoft Docs
description: Describe qué es una conversación en Bot Builder SDK.
keywords: flujo de conversación, reconocimiento de intenciones, un solo turno, varios turnos, conversación de bot
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/01/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 19f0b67454a8c0a4bf171579f8e481e630db83ac
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998922"
---
# <a name="conversation-flow"></a>Flujo de conversación
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Para diseñar del flujo de conversación de un bot es necesario decidir cómo responde el bot cuando el usuario le dice algo. Un bot reconoce en primer lugar la tarea o el tema de conversación basándose en un mensaje del usuario. Para determinar la tarea o el tema (lo que se denomina *intención*) asociados al mensaje de un usuario, el bot puede buscar palabras o patrones en el texto del mensaje o bien puede aprovechar servicios como [Language Understanding](bot-builder-concept-luis.md) y [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/overview/overview).

Una vez que ha reconocido la intención del usuario, en función del escenario, el bot podría satisfacer la solicitud del usuario con una sola respuesta, con lo que completaría la conversación en un solo turno, o podría requerir una serie de turnos. Para los flujos de conversación de varios turnos, Bot Builder SDK proporciona [administración de estados](./bot-builder-howto-v4-state.md) para llevar un seguimiento de una conversación, [avisos](bot-builder-prompts.md) para pedir información y [cuadros de diálogo](bot-builder-dialog-manage-conversation-flow.md) para encapsular los flujos de conversación.

En un bot complejo con varios subsistemas, usted podría usar varios servicios para reconocer la intención, uno para cada subcomponente del bot. La [herramienta de distribución](bot-builder-tutorial-dispatch.md) coloca en un mismo lugar los resultados de varios servicios cuando se combinan subsistemas de conversación en un bot.

<!-- 
A conversation identifies a series of activities sent between a bot and a user on a specific channel and represents an interaction between one or more bots and either a _direct_ conversation with a specific user or a _group_ conversation with multiple users.
A bot communicates with a user on a channel by receiving activities from, and sending activities to the user.

- Each user has an ID that is unique per channel.
- Each conversation has an ID that is unique per channel.
- The channel sets the conversation ID when it starts the conversation.
- The bot cannot start a conversation; however, once it has a conversation ID, it can resume that conversation.
- Not all channels support group conversations.
-->

## <a name="single-turn-conversation"></a>Conversación de un solo turno

El flujo de conversación más sencillo es el de un solo turno. En un flujo de un solo turno, el bot finaliza su tarea en un turno, que consta de un mensaje del usuario y una respuesta del bot.

<!-- The following isn't always true, it's a generalization -->

El tipo más sencillo de bot de un solo turno no necesita llevar un seguimiento del estado de la conversación. Cada vez que recibe un mensaje, responde únicamente en función del contexto del mensaje entrante actual, sin tener conocimiento de turnos de conversación anteriores.

![Bot del tiempo de un solo turno](./media/concept-conversation/weather-single-turn.png)

Un bot meteorológico podría tener un flujo de un solo turno. Simplemente proporciona al usuario un informe meteorológico, sin necesidad de preguntar la ciudad o la fecha. Toda la lógica para mostrar el informe meteorológico se basa en el mensaje que el bot acaba de recibir. En cada turno de una conversación, el bot recibe un [contexto de turno](bot-builder-concept-activity-processing.md#turn-context) que puede usar para determinar qué hará a continuación y cómo fluye la conversación.

## <a name="multiple-turns"></a>Varios turnos

La mayoría de los tipos de conversación no se puede completar en un solo turno, por lo que un bot también puede tener un flujo de conversación de varios turnos. Algunos escenarios que requieren varios turnos de conversación incluyen lo siguiente:

* Un bot que le pide al usuario la información adicional que necesita para completar una tarea. El bot debe comprobar si cuenta con todos los parámetros para cumplir la tarea.
* Un bot que guía al usuario a través de los pasos en un proceso, como la realización de un pedido. El bot debe llevar un seguimiento de dónde se encuentra el usuario en la secuencia de pasos.

Por ejemplo, un bot meteorológico podría tener un flujo de varios turnos si responde a la pregunta "¿Qué tiempo hace?" preguntando por la ciudad.

![Bot del tiempo de varios turnos](./media/concept-conversation/weather-multi-turn.png)

Cuando el usuario responde al mensaje del bot en el que le pregunta por la ciudad y el bot recibe la respuesta "Seattle", el bot necesita tener contexto guardado para entender que el mensaje actual es la respuesta a una pregunta anterior y que forma parte de una solicitud para obtener el tiempo. Los bots de varios turnos llevan un seguimiento del estado para responder de forma adecuada a los mensajes nuevos.

Consulte [Administración del estado de la conversación y el usuario](bot-builder-howto-v4-state.md) para más información.

> [!NOTE]
> Las conversaciones de varios turnos con clientes de la API de REST deben llevar un seguimiento de su propio estado, por ejemplo, en una base de datos o Table Storage.

## <a name="conversation-topics"></a>Temas de conversación

Tal vez diseñe su bot para que controle más de un tipo de tareas. Por ejemplo, podría tener un bot que proporcione flujos de conversación diferentes para saludar al usuario, realizar un pedido, cancelar un pedido y obtener ayuda. Una manera de controlar este cambio en la conversación entre tareas o temas diferentes consiste en reconocer la intención (lo que el usuario quiere hacer) del mensaje actual.

### <a name="recognize-intent"></a>Reconocimiento de intenciones

El SDK Bot Builder proporciona _reconocedores_ que procesan un mensaje para determinar la intención, de modo que el bot pueda iniciar el flujo de conversación correspondiente. Llame al método asincrónico _recognize_ del reconocedor para determinar la intención del usuario a partir del contenido del mensaje. A continuación, puede llamar al método _get top scoring intent_ en el resultado para obtener la predicción más alta del reconocedor.

Un reconocedor podría utilizar expresiones regulares, reconocimiento del lenguaje u otra lógica que desarrolle. Los siguientes son ejemplos de reconocedores posibles:

* Un reconocedor que utiliza QnA Maker para detectar si un usuario realiza una pregunta incluida en una base de conocimiento.
* Un reconocedor que usa Language Understanding (LUIS) para entrenar un servicio con ejemplos de las maneras en las que un usuario podría pedir ayuda y lo asigna a la intención `Help`.
* Un reconocedor personalizado que usa expresiones regulares para buscar comandos.
* Un reconocedor personalizado que utiliza un servicio para traducir la entrada.

### <a name="consider-how-to-interrupt-conversation-flow-or-change-topics"></a>Cómo interrumpir el flujo de conversación o cambiar de tema

Una manera de llevar un seguimiento del punto de la conversación en el que se encuentra consiste en usar el [estado de la conversación](bot-builder-howto-v4-state.md). De este modo, se guarda información sobre el tema activo o sobre los pasos de una secuencia que se han completado.

Cuando un bot se vuelve más complejo, también se puede concebir una secuencia de flujos de conversación que se producen en una pila. Por ejemplo, el bot invoca el flujo del nuevo pedido y, después, invoca el de la búsqueda de productos. Después, el usuario selecciona un producto y lo confirma, con lo que se completa el flujo de búsqueda de productos y, luego, el pedido.

Sin embargo, una conversación rara vez sigue una ruta tan lineal y lógica. Los usuarios no se comunican en "pilas", sino que tienden a cambiar de opinión con frecuencia. Considere el siguiente ejemplo:

![El usuario dice algo inesperado](./media/concept-conversation/interruption.png)

Mientras que el bot podría haber construido lógicamente una pila de flujos, el usuario podría decidir hacer algo completamente diferente o formular una pregunta que no tenga ninguna relación con el tema actual. En el ejemplo, el usuario hace una pregunta en lugar de facilitar la respuesta afirmativa o negativa que el flujo espera. ¿Cómo debe responder el flujo?

* Insiste en que el usuario responda primero a la pregunta.
* Pasa por alto todo lo que el usuario ha hecho hasta el momento, restablece la pila del flujo al completo y empieza desde cero intentando responder a la pregunta del usuario.
* Intenta responder a la pregunta del usuario y, después, regresa a la pregunta con respuesta afirmativa o negativa para intentar reanudar la conversación desde ahí.

No hay una respuesta correcta a esta pregunta, ya que la mejor solución dependerá de los aspectos concretos del escenario y de cómo espera el usuario que responda el bot. Consulte cómo [usar diálogos](bot-builder-dialog-manage-conversation-flow.md) y [controlar las interrupciones](bot-builder-howto-handle-user-interrupt.md) para administrar el flujo de la conversación.

## <a name="conversation-lifetime"></a>Duración de la conversación

<!-- Note: these activities are dependent on whether the channel actually sends them. Also, we should add links --> Un bot recibe una actividad de _actualización de la conversación_ cada vez que se ha agregado a una conversación, se han agregado otros miembros a la conversación (o se han eliminado de ella) o han cambiado los metadatos de la conversación.
Puede que le interese que el bot reaccione a las actividades de actualización de la conversación saludando a los usuarios o presentándose.

Un bot recibe una actividad de _finalización de la conversación_ para indicar que el usuario ha puesto fin a la conversación. Un bot podría enviar una actividad de _finalización de la conversación_ para indicar que la conversación está llegando a su fin.
Si va a almacenar información sobre la conversación, puede que le interese borrar esa información cuando la conversación haya finalizado.

<!--  Types of conversations -->

El bot puede admitir interacciones de varios turnos en las que pide a los usuarios varias partes de información. Se puede centrar en una tarea muy específica o admitir varios tipos de tareas.
El SDK Bot Builder tiene compatibilidad integrada con Language Understanding (LUIS) y QnA Maker para agregar al bot características de lenguaje natural del tipo "preguntas y respuestas".

## <a name="conversations-channels-and-users"></a>Conversaciones, canales y usuarios

Las conversaciones pueden ser de dos tipos: una conversación _directa_ con un usuario específico o una conversación de _grupo_ con varios usuarios.
Para comunicarse con un usuario en un canal, un bot recibe actividades del usuario y a su vez le envía actividades.

* Cada usuario tiene un identificador único para cada canal.
* Cada conversación tiene un identificador único para cada canal.
* El canal establece el identificador de conversación cuando inicia la conversación.
* El bot no puede iniciar una conversación, pero en cuanto dispone de un identificador de conversación, puede reanudarla.
* No todos los canales admiten conversaciones de grupo.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Administración de un flujo de conversación simple con diálogos](bot-builder-dialog-manage-conversation-flow.md)

<!-- In addition, your bot can send activities back to the user, either _proactively_, in response to internal logic, or _reactively_, in response to an activity from the user or channel.-->
<!--TODO: Link to messaging how tos.-->

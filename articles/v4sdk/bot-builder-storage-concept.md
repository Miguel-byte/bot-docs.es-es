---
redirect_url: /bot-framework/bot-builder-howto-v4-state
ms.openlocfilehash: a0d2b1295be1271e827d617ad09878ee8cfcd356
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54223920"
---
<a name="--"></a><!--
---
título: Estado y almacenamiento | Microsoft Docs descripción: Describe el administrador de estado, el estado de la conversación y el estado del usuario dentro de Bot Framework SDK.
Palabras clave: LUIS, estado de la conversación, estado del usuario, almacenamiento, administración del estado autor: DeniseMak ms.author: v-demak manager: kamrani ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date: 02/15/2018 monikerRange: 'azure-bot-service-4.0'
---

# <a name="state-and-storage"></a>Estado y almacenamiento
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Una clave del diseño correcto de bots consiste en realizar el seguimiento del contexto de una conversación, para que el bot recuerde cosas como las respuestas a preguntas anteriores.
En función del uso previsto del bot, es posible que incluso sea necesario realizar el seguimiento de la información de estado o almacenamiento durante más tiempo que la duración de la conversación.
El *estado* de un bot es la información que recuerda con el fin de responder de forma adecuada a los mensajes entrantes. Bot Framework SDK proporciona clases para almacenar y recuperar datos de estado como un objeto asociado a un usuario o una conversación.

* Las **propiedades de la conversación** ayudan al bot a realizar el seguimiento de la conversación actual que tiene con el usuario. Si es necesario que el bot complete una secuencia de pasos o que cambie entre temas de conversación, se pueden usar las propiedades de la conversación para administrar los pasos en una secuencia o realizar el seguimiento del tema actual. Como las propiedades de la conversación reflejan el estado de la conversación actual, normalmente se borran al final de una conversación, cuando el bot recibe una actividad de _fin de la conversación_.
* Las **propiedades de usuario** se pueden usar para muchos propósitos, como determinar dónde dejó la conversación anterior el usuario o simplemente saludar a un usuario por su nombre cuando regrese. Si almacena las preferencias del usuario, puede usar esa información para personalizar la conversación la próxima vez que chatee. Por ejemplo, podría alertar al usuario sobre un artículo de noticias sobre un tema que le interesa, o bien cuando haya una cita disponible. Debe borrarlas si el bot recibe una actividad _eliminar datos de usuario_.

Puede usar [Almacenamiento](bot-builder-howto-v4-storage.md) para leer y escribir en el almacenamiento persistente. Esto permite que el bot haga cosas como actualizar recursos compartidos, registrar confirmaciones de asistencia o votos, o bien leer datos meteorológicos históricos. Al igual que una aplicación usa el almacenamiento para lograr sus objetivos, el bot puede hacerlo dentro de la conversación con el usuario.

<!-- 
*Conversation state* pertains to the current conversation that the user is having with your bot. When the conversation ends, your bot deletes this data.

You can also store *user state* that persists after a conversation ends. For example, if you store a user's preferences, you can use that information to customize the conversation the next time you chat. For example, you might alert the user to a news article about a topic that interests her, or alert a user when an appointment becomes available. 
-->

<!-- You should generally avoid saving state using a global variable or function closures.
Doing so will create issues when you want to scale out your bot. Instead, use the conversation state and user state middleware that the BotBuilder SDK provides --> 

<!--
## Types of underlying storage

The SDK provides bot state manager middleware to persist conversation and user state. State can be accessed using the bot's context. This state manager can use Azure Table Storage, file storage, or memory storage as the underlying data storage. You can also create your own storage components for your bot.

Bots built using Azure Table Storage can be designed to be stateless and scalable across multiple compute nodes.

> [!NOTE] 
> File and memory storage won't scale across nodes.

## Writing directly to storage

You can also use the Bot Framework SDK to read and write data directly to storage, without using middleware or without using the bot context. This can be appropriate to data that your bot uses, that comes from a source outside your bot's conversation flow.

For example, let's say your bot allows the user to ask for the weather report, and your bot retrieves the weather report for a specified date, by reading it from an external database. The content of the weather database isn't dependent on user information or the conversation context, so you could just read it directly from storage instead of using the state manager.  See [How to write directly to storage](bot-builder-howto-v4-storage.md) for an example.

## Next steps

Next, lets get into how activities are processed, in depth, and how we respond to them.

> [!div class="nextstepaction"]
> [Activity Processing](bot-builder-concept-activity-processing.md)

## Additional resources

- [How to save state](bot-builder-howto-v4-state.md)
- [How to write directly to storage](bot-builder-howto-v4-storage.md)

-->

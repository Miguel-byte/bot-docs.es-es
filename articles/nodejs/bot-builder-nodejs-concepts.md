---
title: Conceptos clave de Bot Framework SDK para Node.js | Microsoft Docs
description: Descripción de los conceptos clave y las herramientas para la creación e implementación de bots de conversación disponibles en Bot Framework SDK para Node.js.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: a3cff9a77de098ee524334183ba891068f176b6e
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464780"
---
# <a name="key-concepts-in-the-bot-framework-sdk-for-nodejs"></a>Conceptos clave de Bot Framework SDK para Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-concepts.md)
> - [Node.js](../nodejs/bot-builder-nodejs-concepts.md)

En este artículo se presentan los conceptos clave de Bot Framework SDK para Node.js. Consulte la [información general sobre Bot Framework](../overview-introduction-bot-framework.md) para una introducción a Bot Framework.

## <a name="connector"></a>Conector
Bot Framework Connector es un servicio que conecta el bot a varios *canales*, que son clientes como [Teams](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-create), Skype, Facebook, Slack y SMS. 

Connector facilita la comunicación entre el bot y el usuario mediante la retransmisión de mensajes del bot al canal y viceversa. La lógica del bot está hospedada como un servicio web que recibe mensajes de los usuarios a través del servicio Connector y las respuestas del bot se envían a Connector mediante HTTPS POST. 

El SDK de Bot Framework para Node.js proporciona las clases [UniversalBot][UniversalBot] and [ChatConnector][ChatConnector] para configurar el bot para que envíe y reciba mensajes a través de Bot Framework Connector. La clase `UniversalBot` conforma los cerebros del bot. Es responsable de administrar todas las conversaciones que el bot tiene con un usuario. La clase `ChatConnector` conecta el bot al servicio de Bot Framework Connector.
Si quiere ver un ejemplo que muestra cómo usar estas clases, consulte [Creación de un bot con Bot Framework SDK para Node.js](bot-builder-nodejs-quickstart.md).

El conector también normaliza los mensajes que envía el bot a canales de modo que pueda desarrollar el bot independientemente de la plataforma. Normalizar un mensaje implica convertirlo del esquema de Bot Framework al esquema del canal. En casos donde el canal no admite todos los aspectos del esquema de Framework, el conector intentará convertir el mensaje a un formato compatible con el canal. Por ejemplo, si el bot envía al canal SMS un mensaje que contiene una tarjeta con los botones de acción, el conector puede presentar la tarjeta como imagen e incluir las acciones como vínculos en el texto del mensaje. [Channel Inspector][ChannelInspector] es una herramienta web que muestra cómo el conector presenta los mensajes en distintos canales.

`ChatConnector` requiere configurar un punto de conexión de API dentro del bot. Con el SDK de Node.js, esto habitualmente se logra al instalar el módulo `restify` de Node.js. También se pueden crear bots para la consola con [ConsoleConnector][ConsoleConnector], que no requiere ningún punto de conexión de API.

## <a name="messages"></a>error de Hadoop

Los mensajes pueden constar de texto que se mostrará, texto que se dirá, datos adjuntos, tarjetas enriquecidas y acciones sugeridas. Puede usar el método [session.send][SessionSend] para enviar mensajes en respuesta a un mensaje del usuario. El bot puede llamar a `send` tantas veces como desee en respuesta a un mensaje del usuario. Para ver un ejemplo que muestre esto, consulte [Respond to user messages][RespondMessages] (Respuesta a mensajes de usuario).

Para ver un ejemplo que muestra cómo enviar una tarjeta gráfica enriquecida con botones interactivos en los que el usuario hace clic para iniciar una acción, consulte [Add rich cards to messages](bot-builder-nodejs-send-rich-cards.md) (Incorporación de tarjetas enriquecidas a mensajes). Para ver un ejemplo que muestra cómo enviar y recibir datos adjuntos, consulte [Send attachments](bot-builder-nodejs-send-receive-attachments.md) (Envío de datos adjuntos). Para ver un ejemplo que muestra cómo enviar un mensaje que especifica el texto que el bot dirá en un canal habilitado para voz, consulte [Add speech to messages](bot-builder-nodejs-text-to-speech.md) (Incorporación de voz a mensajes). Para ver un ejemplo que muestra cómo enviar acciones sugeridas, consulte [Send suggested actions](bot-builder-nodejs-send-suggested-actions.md) (Envío de acciones sugeridas).

## <a name="dialogs"></a>Diálogos
Los diálogos ayudan a organizar la lógica de la conversación en el bot y son fundamentales para [diseñar el flujo de la conversación](../bot-service-design-conversation-flow.md). Para ver una introducción a los diálogos, consulte [Manage a conversation with dialogs](bot-builder-nodejs-dialog-manage-conversation.md) (Administración de una conversación con diálogos).

## <a name="actions"></a>Acciones
Querrá diseñar el bot de manera que pueda controlar interrupciones como solicitudes de cancelación o ayuda en cualquier momento durante el flujo de la conversación. Bot Framework SDK para Node.js proporciona controladores de mensajes globales que desencadenan acciones como la cancelación o la invocación de otros diálogos. Consulte [Controlar las acciones del usuario](bot-builder-nodejs-dialog-actions.md) para ver ejemplos de cómo usar los controladores [triggerAction][triggerAction].
<!--[Handling cancel](bot-builder-nodejs-manage-conversation-flow.md#handling-cancel), [Confirming interruptions](bot-builder-nodejs-manage-conversation-flow.md#confirming-interruptions) and-->


## <a name="recognizers"></a>Reconocedores
Cuando los usuarios piden algo al bot, como "ayuda" o "buscar noticias", el bot debe entender qué es lo que pide el usuario y luego realizar la acción correspondiente. Puede diseñar el bot de manera que reconozca las intenciones en función de la información del usuario y que asocie esa intención con acciones. 

Puede usar el reconocedor integrado de expresiones regulares que proporciona Bot Framework SDK, llamar a un servicio externo (como la API de LUIS) o implementar un reconocedor personalizado para determinar la intención del usuario. Consulte [Recognize user intent](bot-builder-nodejs-recognize-intent-messages.md) (Reconocimiento de intenciones del usuario) para ver ejemplos que muestran cómo agregar reconocedores al bot y usarlos para desencadenar acciones.


## <a name="saving-state"></a>Almacenamiento del estado

Una clave del diseño correcto de bots consiste en realizar el seguimiento del contexto de una conversación, para que el bot recuerde cosas como la última pregunta que hizo el usuario. Los bots creados con Bot Framework SDK están diseñados para no tener estado, con el fin de que se puedan escalar fácilmente para ejecutarse en varios nodos de proceso. Bot Framework proporciona un sistema de almacenamiento que almacena datos de bot de manera que el servicio web del bot se puede escalar. Debido a esto, generalmente debe evitar guardar el estado con un cierre de función o una variable global. Si lo hace, generará problemas cuando intente escalar horizontalmente el bot. En su lugar, use las propiedades siguientes del objeto [session][Session] del bot para guardar los datos relativos a un usuario o una conversación:

* **userData** almacena información globalmente para el usuario a través de todas las conversaciones.
* **conversationData** almacena información globalmente para una sola conversación. Estos datos son visibles para todos los participantes de la conversación, por lo que debe tener cuidado al almacenar datos en esta propiedad. Esta opción está habilitada de manera predeterminada y puede deshabilitarla con la configuración [persistConversationData][PersistConversationData] del bot.
* **privateConversationData** almacena información globalmente para una sola conversación, pero se trata de datos privados específicos para el usuario actual. Estos datos abarcan todos los diálogos, por lo que resulta útil para almacenar el estado temporal que quiere limpiar cuando termina la conversación.
* **dialogData** conserva la información de una instancia de diálogo único. Esto es esencial para almacenar la información temporal entre los pasos de una [cascada](bot-builder-nodejs-dialog-waterfall.md) en un diálogo.

Para ver ejemplos que muestran cómo usar estas propiedades para almacenar y recuperar datos, consulte [Manage state data](bot-builder-nodejs-state.md) (Administración de datos de estado).

## <a name="natural-language-understanding"></a>Comprensión del lenguaje natural

Bot Builder le permite usar LUIS para agregar la comprensión del lenguaje natural al bot con la clase [LuisRecognizer][LuisRecognizer]. Puede agregar una instancia de **LuisRecognizer** que hace referencia al modelo de lenguaje publicado y luego agregar controladores para que realicen acciones en respuesta a las expresiones del usuario. Vea este tutorial de 10 minutos para ver a LUIS en acción:

* [Tutorial de Microsoft LUIS][LUISVideo] (vídeo)

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Introducción a los diálogos](bot-builder-nodejs-dialog-overview.md)



[PersistConversationData]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iuniversalbotsettings.html#persistconversationdata
[UniversalBot]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
[ChatConnector]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[ConsoleConnector]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.consoleconnector.html

[ChannelInspector]: ../bot-service-channel-inspector.md

[Session]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html
[SessionSend]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#send

[triggerAction]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction
[waterfall]: bot-builder-nodejs-prompts.md

[RespondMessages]:bot-builder-nodejs-use-default-message-handler.md

[LUISRecognizer]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer
[LUISVideo]: https://vimeo.com/145499419

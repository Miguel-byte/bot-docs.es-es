---
title: Conceptos clave de Bot Framework Direct Line API 3.0 | Microsoft Docs
description: Comprenda los conceptos claves de Bot Framework Direct Line API 3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/06/2019
ms.openlocfilehash: 6727530ec6267a63e28e103bdfc12183ebed8016
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299592"
---
# <a name="key-concepts-in-direct-line-api-30"></a>Conceptos clave de Direct Line API 3.0

Para habilitar la comunicación entre el bot y su propia aplicación cliente, use la API Direct Line. En este artículo se presentan los conceptos clave de Direct Line API 3.0 y se proporciona información acerca de los recursos pertinentes para desarrolladores.

## <a name="authentication"></a>Authentication

Las solicitudes de Direct Line API 3.0 se pueden autenticar ya sea mediante un **secreto**, que se obtiene desde la página de configuración del canal de Direct Line en el <a href="https://dev.botframework.com/" target="_blank">portal de Bot Framework</a>, o mediante un **token**  que se obtiene en el entorno en tiempo de ejecución. Para más información, consulte [Autenticación](bot-framework-rest-direct-line-3-0-authentication.md).

## <a name="starting-a-conversation"></a>Inicio de una conversación

Los clientes pueden abrir las conversaciones de Direct Line explícitamente y estas se pueden ejecutar siempre y cuando el cliente y el bot participen y tengan credenciales válidas. Para obtener más información, consulte [Inicio de una conversación](bot-framework-rest-direct-line-3-0-start-conversation.md).

## <a name="sending-messages"></a>Envío de mensajes

Con Direct Line API 3.0, un cliente puede enviar mensajes a su bot mediante la emisión de solicitudes `HTTP POST`. Un cliente puede enviar un único mensaje por solicitud. Para obtener más información, consulte [Envío de una actividad al bot](bot-framework-rest-direct-line-3-0-send-activity.md).

## <a name="receiving-messages"></a>Recepción de mensajes

Con Direct Line API 3.0, un cliente puede recibir mensajes de su bot mediante streaming de `WebSocket` o mediante la emisión de solicitudes `HTTP GET`. Con cualquiera de estas técnicas, un cliente puede recibir varios mensajes desde el bot a la vez como parte de un `ActivitySet`. Para más información, consulte [Recepción de actividades del bot](bot-framework-rest-direct-line-3-0-receive-activities.md).

## <a name="developer-resources"></a>Recursos para el desarrollador

### <a name="client-libraries"></a>Bibliotecas de clientes

Bot Framework proporciona bibliotecas cliente que facilitan el acceso a Direct Line API 3.0 mediante C# y Node.js. 

- Para usar la biblioteca cliente dentro de un proyecto de Visual Studio, instale el <a href="https://www.nuget.org/packages/Microsoft.Bot.Connector.DirectLine" target="_blank">paquete NuGet</a> `Microsoft.Bot.Connector.DirectLine`. 

- Para usar la biblioteca cliente de Node.js, instale la biblioteca `botframework-directlinejs` mediante <a href="https://www.npmjs.com/package/botframework-directlinejs" target="_blank">NPM</a> (o <a href="https://github.com/Microsoft/BotFramework-DirectLineJS" target="_blank">descargue</a> el origen).

::: moniker range="azure-bot-service-3.0"

### <a name="sample-code"></a>Código de ejemplo

El repositorio de GitHub <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples" target="_blank">BotBuilder-Samples</a> contiene varios ejemplos que muestran el uso de Direct Line API 3.0 con C# y Node.js.

| Muestra | Idioma | DESCRIPCIÓN |
|----|----|----|
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-DirectLine" target="_blank">Bot de ejemplo de Direct Line</a> | C# | Bot de ejemplo y cliente personalizado que se comunican entre sí mediante Direct Line API. |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-DirectLineWebSockets" target="_blank">Bot de ejemplo de Direct Line (mediante WebSockets cliente)</a> | C# | Bot de ejemplo y cliente personalizado que se comunican entre sí mediante Direct Line API y WebSockets. |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node/core-DirectLine" target="_blank">Bot de ejemplo de Direct Line</a> | JavaScript | Bot de ejemplo y cliente personalizado que se comunican entre sí mediante Direct Line API. |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node/core-DirectLineWebSockets" target="_blank">Bot de ejemplo de Direct Line (mediante WebSockets cliente)</a> | JavaScript | Bot de ejemplo y cliente personalizado que se comunican entre sí mediante Direct Line API y WebSockets. |

::: moniker-end

### <a name="web-chat-control"></a>Control Chat en web 

Bot Framework proporciona un control que le permite insertar un bot con tecnología de Direct Line en la aplicación cliente. Para obtener más información, consulte <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Control Chat en web de Microsoft Bot Framework</a>.

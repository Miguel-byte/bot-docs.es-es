---
title: Conceptos clave de la API Direct Line 1.1 de Bot Framework | Microsoft Docs
description: Comprenda los conceptos claves la API Direct Line 1.1 de Bot Framework.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 288fde1d8c4333256b956b270fe37a56217632ca
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404652"
---
# <a name="key-concepts-in-direct-line-api-11"></a>Conceptos clave de la API Direct Line 1.1 de Bot Framework

Para habilitar la comunicación entre el bot y su propia aplicación cliente, use la API Direct Line. 

> [!IMPORTANT]
> En este artículo se presentan los conceptos clave de la API Direct Line 1.1 y se proporciona información acerca de los recursos pertinentes para desarrolladores. Si va a crear una nueva conexión entre la aplicación cliente y el bot, use [Direct Line API 3.0](bot-framework-rest-direct-line-3-0-concepts.md) en su lugar.

## <a name="authentication"></a>Authentication

Las solicitudes de la API Direct Line 1.1 se pueden autenticar ya sea mediante un **secreto**, que se obtiene desde la página de configuración del canal de Direct Line en el <a href="https://dev.botframework.com/" target="_blank">portal de Bot Framework</a>, o mediante un **token** , que se obtiene en el tiempo de ejecución.  Para más información, consulte [Autenticación](bot-framework-rest-direct-line-1-1-authentication.md).

## <a name="starting-a-conversation"></a>Inicio de una conversación

Los clientes pueden abrir las conversaciones de Direct Line explícitamente y estas se pueden ejecutar siempre y cuando el cliente y el bot participen y tengan credenciales válidas. Para obtener más información, consulte [Inicio de una conversación](bot-framework-rest-direct-line-1-1-start-conversation.md).

## <a name="sending-messages"></a>Envío de mensajes

Con la API Direct Line 1.1, un cliente puede enviar mensajes a su bot mediante la emisión de solicitudes `HTTP POST`. Un cliente puede enviar un único mensaje por solicitud. Para más información, consulte [Envío de una actividad al bot](bot-framework-rest-direct-line-1-1-send-message.md).

## <a name="receiving-messages"></a>Recepción de mensajes

Con la API Direct Line 1.1, un cliente puede recibir mensajes mediante el sondeo de solicitudes `HTTP GET`. En respuesta a cada solicitud, un cliente puede recibir varios mensajes del bot como parte de `MessageSet`. Para obtener más información, consulte [Receive messages from the bot](bot-framework-rest-direct-line-1-1-receive-messages.md) (Recepción de mensajes del bot).

## <a name="developer-resources"></a>Recursos para el desarrollador

### <a name="client-library"></a>Biblioteca de cliente

Bot Framework proporciona una biblioteca cliente que facilita el acceso a la API Direct Line 1.1 a través de C#. Para usar la biblioteca cliente dentro de un proyecto de Visual Studio, instale el <a href="https://www.nuget.org/packages/Microsoft.Bot.Connector.DirectLine/1.1.1" target="_blank">paquete NuGet v1.x </a> `Microsoft.Bot.Connector.DirectLine`. 

Como alternativa al uso de la biblioteca cliente de C#, puede generar su propia biblioteca cliente en el lenguaje que quiera mediante el <a href="https://docs.botframework.com/restapi/directline/swagger.json" target="_blank">archivo Swagger de la API Direct Line 1.1</a>.

### <a name="web-chat-control"></a>Control Chat en web 

Bot Framework proporciona un control que le permite insertar un bot con tecnología de Direct Line en la aplicación cliente. Para obtener más información, consulte <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Control Chat en web de Microsoft Bot Framework</a>.

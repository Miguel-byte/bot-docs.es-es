---
title: Conexión de un bot a Direct Line Speech (versión preliminar)
titleSuffix: Bot Service
description: Información general y pasos necesarios para conectar un bot de Bot Framework existente al canal de Direct Line Speech para la interacción de la entrada de voz y salida de voz con una alta confiabilidad y baja latencia.
services: bot-service
author: trrwilson
manager: nitinme
ms.service: bot-service
ms.subservice: bot-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: travisw
ms.custom: ''
ms.openlocfilehash: 8e0d2939078e1e27162c7056373e95790a03eb88
ms.sourcegitcommit: 5042e31bc6b2762d7a6636e98c8f496b90ea33c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65240435"
---
# <a name="connect-a-bot-to-direct-line-speech-preview"></a>Conexión de un bot a Direct Line Speech (versión preliminar)

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

Puede configurar un bot para permitir que las aplicaciones cliente se comuniquen con él a través del canal de Direct Line Speech.

Una vez creado el bot, su incorporación a Direct Line Speech permitirá una baja latencia y conexión de alta confiabilidad con aplicaciones cliente que utilizan [Speech SDK](https://aka.ms/speech/sdk). Estas conexiones se optimizan para las conversaciones de entrada de voz y salida de voz. Para más información acerca de Direct Line Speech y cómo crear aplicaciones cliente, visite la página del [asistente virtual personalizado por voz](https://aka.ms/bots/speech/va).  

## <a name="sign-up-for-direct-line-speech-preview"></a>Suscripción a la versión preliminar de Direct Line Speech

Actualmente, Direct Line Speech está en versión preliminar y requiere una suscripción rápida en [Azure Portal](https://portal.azure.com). Vea los detalles a continuación. Una vez que se apruebe, accederá al canal.

## <a name="add-the-direct-line-speech-channel"></a>Incorporación del canal Direct Line Speech

1. Para agregar el canal de Direct Line Channel, abra primero el bot en [Azure Portal](https://portal.azure.com) y haga clic en **Channels** (Canales) en la hoja de configuración.

    ![resaltado de la ubicación para la selección de canales a los que conectarse](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-selectchannel.png "selección de canales")

1. En la página de selección de canales, busque y haga clic en `Direct Line Speech` para elegir el canal.

    ![selección del canal de Direct Line Speech](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-connectspeechchannel.png "conexión de Direct Line Speech")

1. Si aún no se ha aprobado el acceso, verá una página para solicitar el acceso. Rellene la información solicitada y haga clic en "Request" (Solicitar). Se mostrará una página de confirmación. Mientras la solicitud esté pendiente de aprobación, no podrá pasar de esta página.   

1. Una vez que se apruebe el acceso, se mostrará una página de configuración de Direct Line Speech. Una vez que haya revisado los términos de uso, haga clic en `Save` para confirmar la selección del canal.

    ![guardado la habilitación del canal de Direct Line Speech](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-savechannel.png "guardar la configuración del canal")

## <a name="enable-the-bot-framework-protocol-streaming-extensions"></a>Habilitación de las extensiones de streaming del protocolo de Bot Framework

Con el canal de Direct Line Speech conectado al bot, necesita habilitar la compatibilidad con las extensiones de streaming del protocolo de Bot Framework para que la interacción de baja latencia sea óptima.

1. Si no lo ha hecho aún, abra la hoja del bot en [Azure Portal](https://portal.azure.com). 

1. Haga clic en **Settings** (Configuración) bajo la categoría **Bot Management** (Administración de bot), que está inmediatamente debajo de **Channels** (Canales). Haga clic en la casilla **Enable Streaming Endpoint** (Habilitar punto de conexión de streaming).

    ![habilitar el protocolo de streaming](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-enablestreamingsupport.png "habilitar la compatibilidad con las extensiones de streaming")

1. Haga clic en **Guardar** en la parte superior de la página.

1. En la misma hoja, en la categoría **App Service Settings** (Configuración de App Service), haga clic en **Configuration** (Configuración).

    ![vaya a la configuración de App Service](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-configureappservice.png "configurar App Service")

1. Haga clic en `General settings` y seleccione la opción para habilitar la compatibilidad con `Web socket`.

    ![habilitar WebSockets para App Service](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-enablewebsockets.png "habilitar WebSockets")

1. Haga clic en `Save` en la parte superior de la página de configuración.

1. Las extensiones de streaming del protocolo de Bot Framework ya están habilitadas para el bot. Ya está listo para actualizar el código del bot e [integrar la compatibilidad con las extensiones de streaming](https://aka.ms/botframework/addstreamingprotocolsupport) a un proyecto de bot existente.

## <a name="manage-secret-keys"></a>Administrar claves secretas

Las aplicaciones cliente necesitarán un secreto de canal para conectarse al bot a través del canal de Direct Line Speech. Una vez que haya guardado la selección de canales, puede recuperar estas claves secretas en la página **Configure Direct Line Speech** (Configurar Direct Line Speech) de Azure Portal.

![obtención de claves secretas para Direct Line Speech](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-getspeechsecretkeys.png "getting secret keys for Direct Line Speech")

## <a name="adding-protocol-support-to-your-bot"></a>Adición de la compatibilidad con protocolos al bot

Con el canal de Direct Line Speech conectado y la compatibilidad de las extensiones de streaming del protocolo de Bot Framework, lo único que queda es agregar código al bot para proporcionar compatibilidad con la comunicación optimizada. Siga las instrucciones que encontrará en el artículo acerca de la [adición de extensiones de streaming a un bot](https://aka.ms/botframework/addstreamingprotocolsupport) para asegurarse de que la compatibilidad con Direct Line Speech es total.

## <a name="known-issues"></a>Problemas conocidos

Tenga en cuenta que el servicio está en versión preliminar y está sujeto a cambios, lo que puede afectar tanto al desarrollo como al rendimiento general de los bots. Esta es una lista de los problemas conocidos: 

1. El servicio está actualmente implementado en la [región de Azure](https://azure.microsoft.com/en-us/global-infrastructure/regions/) oeste de EE. UU. 2. Pronto se implementará en otras regiones, por lo que todos los clientes se beneficiarán de interacciones de voz de baja latencia con sus bots.

1. Se realizarán cambios menores en los campos de control, como [serviceUrl](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#service-url)

1. Las actividades [conversationUpdate](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation-update-activity) y [endOfCoversation](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#end-of-conversation-activity) usadas para señalar el inicio y final de las conversaciones, y que habitualmente se utilizan al generar mensajes de bienvenida, se actualizarán para mantener la coherencia con otros canales

1. [SigninCard](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-add-rich-cards?view=azure-bot-service-4.0) aún no es compatible con el canal 

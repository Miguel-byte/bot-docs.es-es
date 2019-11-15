---
title: Conexión de un bot con Direct Line Speech
titleSuffix: Bot Service
description: Información general y pasos necesarios para conectar un bot de Bot Framework existente al canal de Direct Line Speech para la interacción de la entrada de voz y salida de voz con una alta confiabilidad y baja latencia.
services: bot-service
author: trrwilson
manager: nitinme
ms.service: bot-service
ms.topic: conceptual
ms.date: 11/02/2019
ms.author: travisw
ms.openlocfilehash: 55bb6b63f35b2cb064229ed0a827af422ca83882
ms.sourcegitcommit: 490810d278d1c8207330b132f28a5eaf2b37bd07
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/05/2019
ms.locfileid: "73592232"
---
# <a name="connect-a-bot-to-direct-line-speech"></a>Conexión de un bot con Direct Line Speech

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

Puede configurar un bot para permitir que las aplicaciones cliente se comuniquen con él a través del canal de Direct Line Speech.

Una vez creado el bot, su incorporación a Direct Line Speech permitirá una baja latencia y conexión de alta confiabilidad con aplicaciones cliente que utilizan [Speech SDK](https://aka.ms/speech/sdk). Estas conexiones se optimizan para las conversaciones de entrada de voz y salida de voz. Para más información acerca de Direct Line Speech y cómo crear aplicaciones cliente, visite la página del [asistente virtual personalizado por voz](https://aka.ms/bots/speech/va). 

## <a name="add-the-direct-line-speech-channel"></a>Incorporación del canal Direct Line Speech

1. Para agregar el canal de Direct Line Speech, abra primero el bot en [Azure Portal](https://portal.azure.com) y, desde los recursos, seleccione el recurso **Registro de canales de bot**. Haga clic en **Canales** en la hoja de configuración.

    ![resaltado de la ubicación para la selección de canales a los que conectarse](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-selectchannel.png "selección de canales")

1. En la página de selección de canales, busque y haga clic en `Direct Line Speech` para elegir el canal.

    ![selección del canal de Direct Line Speech](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-connectspeechchannel.png "conexión con Direct Line Speech")

1. El canal de Direct Line Speech requiere un recurso de Cognitive Services. Puede usar un recurso existente o crear un nuevo recurso de Cognitive Services como se indica en estas [instrucciones](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account). 

    ![selección del canal de Direct Line Speech](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-cognitivesericesaccount-selection.png "selección de recurso de Cogntive Services")

1. Una vez que haya revisado los términos de uso, haga clic en `Save` para confirmar la selección del canal.

    ![guardar la habilitación del canal de Direct Line Speech](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-savechannel.png "Guardar la configuración del canal")

## <a name="enable-the-bot-framework-protocol-streaming-extensions"></a>Habilitación de las extensiones de streaming del protocolo de Bot Framework

Con el canal de Direct Line Speech conectado al bot, necesita habilitar la compatibilidad con las extensiones de streaming del protocolo de Bot Framework para que la interacción de baja latencia sea óptima.

1. Si no lo ha hecho aún, abra la hoja del bot en [Azure Portal](https://portal.azure.com). 

1. Haga clic en **Settings** (Configuración) bajo la categoría **Bot Management** (Administración de bot), que está inmediatamente debajo de **Channels** (Canales). Haga clic en la casilla **Enable Streaming Endpoint** (Habilitar punto de conexión de streaming).

    ![habilitación del protocolo de streaming](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-enablestreamingsupport.png "habilitar la compatibilidad con la extensión de streaming")

1. Haga clic en **Guardar** en la parte superior de la página.

1. En la misma hoja, en la categoría **App Service Settings** (Configuración de App Service), haga clic en **Configuration** (Configuración).

    ![ir a la configuración de App Service](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-configureappservice.png "configurar App Service")

1. Haga clic en `General settings` y seleccione la opción para habilitar la compatibilidad con `Web socket`.

    ![habilitar WebSockets para App Service](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-enablewebsockets.png "habilitar WebSockets")

1. Haga clic en `Save` en la parte superior de la página de configuración.

1. Las extensiones de streaming del protocolo de Bot Framework ya están habilitadas para el bot. Ya está listo para actualizar el código del bot e [integrar la compatibilidad con las extensiones de streaming](https://aka.ms/botframework/addstreamingprotocolsupport) a un proyecto de bot existente.

## <a name="adding-protocol-support-to-your-bot"></a>Adición de la compatibilidad con protocolos al bot

Con el canal de Direct Line Speech conectado y la compatibilidad de las extensiones de streaming del protocolo de Bot Framework, lo único que queda es agregar código al bot para proporcionar compatibilidad con la comunicación optimizada. Siga las instrucciones que encontrará en el artículo acerca de la [adición de extensiones de streaming a un bot](https://aka.ms/botframework/addstreamingprotocolsupport) para asegurarse de que la compatibilidad con Direct Line Speech es total.



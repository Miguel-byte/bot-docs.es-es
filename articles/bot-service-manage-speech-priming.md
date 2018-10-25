---
title: Configuración de la preparación para la voz | Microsoft Docs
description: Obtenga información sobre cómo configurar la preparación para la voz para el servicio de bots mediante Azure Portal.
keywords: preparación para la voz, reconocimiento de voz, LUIS
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 12/13/2017
ms.openlocfilehash: 5cb47be530f9f82d83272684e6405730c72f3cb7
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997158"
---
# <a name="configure-speech-priming"></a>Configuración de la preparación para la voz

La preparación para la voz mejora el reconocimiento de textos orales que se usan habitualmente en el bot. Para los bots compatibles con voz que usan los canales de Cortana y Chat en web, la preparación para la voz usa ejemplos especificados en las aplicaciones de Language Understanding (LUIS) para mejorar la precisión del reconocimiento de voz de palabras importantes.

Es posible que el bot ya se haya integrado con una aplicación de LUIS, pero también puede crear una aplicación de LUIS para asociarla al bot de preparación para la voz. La aplicación de LUIS contiene ejemplos de lo que espera que los usuarios digan al bot. Las palabras importantes que quiere que el bot reconozca deben estar etiquetadas como entidades. Por ejemplo, en un bot de ajedrez debe asegurarse de que, cuando el usuario diga "Mover alfil", no se interprete como "Mover marfil". La aplicación de LUIS debe incluir ejemplos en los que "alfil" se etiquete como una entidad.

> [!NOTE]
> Para usar la preparación para la voz con el canal Chat en web, debe usar el servicio de Bing Speech. Consulte [Enable speech in the Web Chat channel](~/bot-service-channel-connect-webchat-speech.md) (Habilitación de voz en el canal Chat en web) para obtener una explicación sobre cómo usar el servicio Bing Speech.

> [!IMPORTANT]
> La preparación para la voz solo se aplica a los bots configurados para el canal Cortana o Chat en web.

> [!IMPORTANT]
> No se admite la preparación desde aplicaciones de LUIS regionales que no sean de EE. UU. incluidas: eu.luis.ai y au.luis.ai

## <a name="change-the-list-of-luis-apps-your-bot-uses"></a>Cambio de la lista de aplicaciones de LUIS que usa el bot

Para cambiar la lista de aplicaciones de LUIS que utiliza Bing Speech con el bot, realice los pasos siguientes:

1. Haga clic en **Speech priming** (Preparación para la voz) en la hoja del servicio de bots. Aparecerá la lista de aplicaciones de LUIS disponibles.
2. Seleccione las aplicaciones de LUIS que quiera que use Bing Speech.
 
    a. Para seleccionar una aplicación de LUIS de la lista, mantenga el puntero sobre el modelo de LUIS hasta que aparezca una casilla de verificación y, a continuación, active la casilla.
     
    b. Para seleccionar una aplicación de LUIS que no está en la lista, desplácese hacia abajo y escriba el GUID de id. de aplicación de LUIS en el cuadro de texto.
     
3. Haga clic en **Guardar** para guardar la lista de aplicaciones de LUIS asociadas con Bing Speech para el bot.

![Panel de la preparación para la voz](~/media/bot-service-manage-speech-priming/speech-priming.png)

## <a name="additional-resources"></a>Recursos adicionales

- Para obtener más información sobre la habilitación de voz en el Chat en web, consulte [Enable speech in the Web Chat channel](~/bot-service-channel-connect-webchat-speech.md) (Habilitación de voz en el canal Chat en web).
- Para obtener más información acerca de la preparación para la voz, consulte [Speech Support in Bot Framework – Web Chat to Directline, to Cortana](https://blog.botframework.com/2017/06/26/Speech-To-Text/) (Compatibilidad con la voz en Bot Framework: de Chat en web a Direct Line y a Cortana).
- Para obtener más información sobre las aplicaciones de LUIS, consulte [Language Understanding Intelligent Service](https://www.luis.ai).

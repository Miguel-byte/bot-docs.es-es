---
title: Canales adicionales | Microsoft Docs
description: Aprenda a configurar canales adicionales para el bot.
keywords: canales de bot, hangouts, Twilio, facebook, azure portal
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/08/2019
ms.openlocfilehash: d6cf068f3cd4d57ff9cde2be6d41e084cee6524d
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693745"
---
# <a name="additional-channels"></a>Canales adicionales

Además de los canales enumerados en estos documentos, hay varios canales adicionales disponibles como adaptador, mediante las [plataformas ofrecidas](https://botkit.ai/docs/v4/platforms/) a través de Botkit, o accesibles mediante los [repositorios de la comunidad](https://github.com/BotBuilderCommunity/). A continuación se muestran los canales adicionales que se ofrecen:

- [Webex Teams](https://botkit.ai/docs/v4/platforms/webex.html)
- [Websocket y Webhooks](https://botkit.ai/docs/v4/platforms/web.html)
- [Google Hangouts y Asistente de Google](https://github.com/BotBuilderCommunity/) (disponible a través de la comunidad)
- [Amazon Alexa](https://github.com/BotBuilderCommunity/) (disponible a través de la comunidad)

## <a name="currently-available-adapters"></a>Adaptadores disponibles actualmente

[Aquí](https://botkit.ai/docs/v4/platforms/) se puede encontrar una lista completa de los adaptadores disponibles. Observará que algunos canales están disponibles también como adaptadores y es decisión suya cuándo se debe utilizar un canal en lugar de un adaptador.

### <a name="when-to-use-an-adapter"></a>Cuándo usar un adaptador

1. El servicio no admite el canal que desea.
2. Los requisitos de seguridad y cumplimiento de la implementación determinan que no puede confiar en un servicio externo.
3. La profundidad de las características que necesita en un canal determinado puede no ser compatible.

### <a name="when-to-use-a-channel"></a>Cuándo usar un canal

1. Cuando requiere compatibilidad entre canales: el bot debe trabajar en más de uno de los canales disponibles
2. Compatibilidad integrada: Microsoft mantiene y revisa continuamente los servicios de cada canal cada vez que un tercero realiza una actualización.
3. Permite el acceso a otros canales exclusivos de Microsoft como en el caso de un rápido crecimiento de Microsoft Teams.
4. Si desea confiar en una interfaz gráfica de usuario para habilitar canales adicionales para el bot.

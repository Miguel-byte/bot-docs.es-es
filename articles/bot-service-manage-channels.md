---
title: Configuración de un bot para ejecutarlo en uno o más canales | Microsoft Docs
description: Obtenga información sobre cómo configurar un bot para ejecutarlo en uno o más canales mediante el Portal de Framework Bot.
keywords: canales de bot, configurar, cortana, facebook messenger, kik, slack, skype, azure portal
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/22/2018
ms.openlocfilehash: 448bd1acd39445c39e49f27c3b9446523124bf87
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/12/2019
ms.locfileid: "59541001"
---
# <a name="connect-a-bot-to-channels"></a>Conexión de un bot a canales

Un canal es una conexión entre el bot y las aplicaciones de comunicación. Un bot se configura para conectarse a los canales en los que quiere que esté disponible. El servicio Bot Framework, que se configura a través de Azure Portal, conecta su bot a estos canales y facilita la comunicación entre el bot y el usuario. Puede conectarse a muchos servicios muy utilizados, como [Cortana](bot-service-channel-connect-cortana.md), [Facebook Messenger](bot-service-channel-connect-facebook.md), [Kik](bot-service-channel-connect-kik.md) y [Slack](bot-service-channel-connect-slack.md), entre otros. El canal [Web Chat](bot-service-channel-connect-webchat.md) están preconfigurados de forma automática. Además de los canales estándar proporcionados con Bot Connector Service, también puede conectar el bot a su propia aplicación cliente con [línea directa](bot-service-channel-connect-directline.md) como canal.

El servicio Bot Framework le permite desarrollar un bot independientemente del canal mediante la normalización de los mensajes que el bot envía a un canal. Esto implica convertirlo del esquema de Bot Framework al esquema del canal. Sin embargo, si el canal no es compatible con todos los aspectos del esquema de Bot Framework, el servicio intentará convertir el mensaje a un formato que sea compatible con el canal. Por ejemplo, si el bot envía al canal de correo electrónico un mensaje que contiene una tarjeta con los botones de acción, el conector puede enviar la tarjeta como imagen e incluir las acciones como vínculos en el texto del mensaje.

Para la mayoría de los canales, se debe proporcionar información de configuración de canal para ejecutar el bot en el canal. La mayoría de los canales requieren que el bot tenga una cuenta en el canal y otros, como Facebook Messenger, requieren que también tenga una aplicación registrada con el canal.

Para configurar el bot para que se conecte a un canal, siga los pasos siguientes:

1. Inicie sesión en <a href="https://portal.azure.com" target="_blank">Azure Portal</a>.
2. Seleccione el bot que quiera configurar.
3. En la hoja Servicio de bots, haga clic en **Canales** en **Administración de bots**.
4. Haga clic en el icono del canal al que quiera agregar el bot.

![Conexión a los canales](./media/channels/connect-to-channels.png)

Después de configurar el canal, los usuarios de ese canal pueden empezar a usar el bot.

## <a name="publish-a-bot"></a>Publicación de un bot

El proceso de publicación es diferente para cada canal.

[!INCLUDE [publishing](./includes/snippet-publish-to-channel.md)]

## <a name="additional-resources"></a>Recursos adicionales

El SDK incluye ejemplos que se pueden usar para crear bots. Visite el [repositorio de ejemplos en GitHub](https://github.com/Microsoft/BotBuilder-samples) para ver una lista de ejemplos.

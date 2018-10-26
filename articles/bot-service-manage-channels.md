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
ms.openlocfilehash: f5ca01b592266d005c8f000e2351ddcf812acb6d
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997390"
---
# <a name="connect-a-bot-to-channels"></a>Conexión de un bot a canales

Un canal es una conexión entre el bot y las aplicaciones de comunicación. Un bot se configura para conectarse a los canales en los que quiere que esté disponible. El servicio Bot Framework, que se configura a través de Azure Portal, conecta su bot a estos canales y facilita la comunicación entre el bot y el usuario. Puede conectarse a muchos servicios muy utilizados, como [Cortana](bot-service-channel-connect-cortana.md), [Facebook Messenger](bot-service-channel-connect-facebook.md), [Kik](bot-service-channel-connect-kik.md) y [Slack](bot-service-channel-connect-slack.md), entre otros. [Skype](https://dev.skype.com/bots) y Chat en web están preconfigurados de forma automática. Además de los canales estándar que proporciona el servicio Bot Connector, también puede conectar el bot a su propia aplicación cliente que usa DirectLine como canal.

El servicio Bot Framework le permite desarrollar un bot independientemente del canal mediante la normalización de los mensajes que el bot envía a un canal. Esto implica convertirlo del esquema del generador de bots al esquema del canal. Sin embargo, si el canal no es compatible con todos los aspectos del esquema del generador de bots, el servicio intentará convertir el mensaje a un formato que sea compatible con el canal. Por ejemplo, si el bot envía al canal SMS un mensaje que contiene una tarjeta con los botones de acción, el conector puede enviar la tarjeta como imagen e incluir las acciones como vínculos en el texto del mensaje.



Para la mayoría de los canales, se debe proporcionar información de configuración de canal para ejecutar el bot en el canal. La mayoría de los canales requieren que el bot tenga una cuenta en el canal y otros, como Facebook Messenger, requieren que también tenga una aplicación registrada con el canal.

Para configurar el bot para que se conecte a un canal, siga los pasos siguientes:

1. Inicie sesión en <a href="https://portal.azure.com" target="_blank">Azure Portal</a>.
1. Seleccione el bot que quiera configurar.
3. En la hoja Servicio de bots, haga clic en **Canales** en **Administración de bots**.
4. Haga clic en el icono del canal al que quiera agregar el bot.

![Conexión a los canales](./media/channels/connect-to-channels.png)

Después de configurar el canal, los usuarios de ese canal pueden empezar a usar el bot.

## <a name="publish-a-bot"></a>Publicación de un bot

El proceso de publicación es diferente para cada canal.

[!INCLUDE [publishing](./includes/snippet-publish-to-channel.md)]

## <a name="additional-resources"></a>Recursos adicionales
El SDK incluye ejemplos que se pueden usar para crear bots. Visite el repositorio de [GitHub](https://github.com/Microsoft/BotBuilder-samples) para ver la lista de ejemplos.

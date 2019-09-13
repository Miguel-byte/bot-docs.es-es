---
title: Creación de un bot de Telegram | Microsoft Docs
description: Obtenga información sobre cómo configurar la conexión de un bot a Telegram.
keywords: configurar bot, Telegram, canal de bots, bot de Telegram, token de acceso
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: cdf9a98d77f876fb582432ab9b4704d2ac98d45f
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/05/2019
ms.locfileid: "70385975"
---
# <a name="connect-a-bot-to-telegram"></a>Conexión de un bot a Telegram

Puede configurar el bot para que se comunique con las personas mediante la aplicación de mensajería Telegram.

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a name="visit-the-bot-father-to-create-a-new-telegram-bot"></a>Visite BotFather para crear un nuevo bot de Telegram

<a href="https://telegram.me/botfather" target="_blank">Creae un nuevo bot de Telegram</a> con BotFather.

![Visitar BotFather](~/media/channels/tg-StepVisitBotFather.png)

## <a name="create-a-new-telegram-bot"></a>Creación de un nuevo bot de Telegram
Para crear un nuevo bot de Telegram, envíe el comando `/newbot`.

![Crear nuevo bot](~/media/channels/tg-StepNewBot.png)

### <a name="specify-a-friendly-name"></a>Especificar un nombre descriptivo

Asigne un nombre descriptivo al bot de Telegram.

![Dar un nombre descriptivo al bot](~/media/channels/tg-StepNameBot.png)

### <a name="specify-a-username"></a>Especificar un nombre de usuario

Asigne un nombre de usuario único al bot de Telegram.

![Dar un nombre único al bot](~/media/channels/tg-StepUsername.png)

### <a name="copy-the-access-token"></a>Copiar el token de acceso

Copie el token de acceso del bot de Telegram.

![Copiar token de acceso](~/media/channels/tg-StepBotCreated.png)

## <a name="enter-the-telegram-bots-access-token"></a>Escribir el token de acceso del bot de Telegram

Vaya a la sección **Canales** del bot en Azure Portal y haga clic en el botón **Telegram**. 

> [!NOTE]
>  La interfaz de usuario de Azure Portal tendrá un aspecto ligeramente diferente si ya ha conectado el bot a Telegram. 

![Seleccionar Telegram en canales](~/media/channels/tg-connectBot-Azure.png)

Pegue el token que copió anteriormente en el campo **Token de acceso** y haga clic en **Guardar**.

![Token de acceso de Telegram](~/media/channels/tg-accessToken-Azure.png)

El bot está ya configurado correctamente para comunicarse con los usuarios en Telegram. 

![Bot de Telegram habilitado](~/media/channels/tg-botEnabled-Azure.png)

---
title: Creación de un bot de Telegram | Microsoft Docs
description: Obtenga información sobre cómo configurar la conexión de un bot a Telegram.
keywords: configurar bot, Telegram, canal de bots, bot de Telegram, token de acceso
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: efe38392117fb871b2b98e3f1d8d798bfaef0c41
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563760"
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

Pegue el token que copió anteriormente en el campo **Token de acceso** y haga clic en **Enviar**.

## <a name="enable-the-bot"></a>Habilitar el bot
Seleccione **Enable this bot on Telegram** (Habilitar este bot en Telegram). A continuación, haga clic en **I'm done configuring Telegram** (Finalicé la configuración de Telegram).

Cuando haya completado estos pasos, el bot se configurará correctamente para comunicarse con los usuarios en Telegram.

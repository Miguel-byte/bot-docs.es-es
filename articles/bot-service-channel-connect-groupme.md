---
title: Conexión de un bot a GroupMe | Microsoft Docs
description: Aprenda a configurar la conexión de un bot a GroupMe.
keywords: canal de bot, GroupMe, crear GroupMe, credenciales
author: RobStand
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: a2004293ff10cfbc7132f58b7c0c834a2012cfd1
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998572"
---
# <a name="connect-a-bot-to-groupme"></a>Conexión de un bot a GroupMe

Puede configurar un bot para comunicarse con personas que usan la aplicación de mensajería GroupMe.

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a name="sign-up-for-a-groupme-account"></a>Registrarse para obtener una cuenta de GroupMe

Si no tiene una cuenta de GroupMe, [regístrese para obtener una nueva cuenta](https://web.groupme.com/signup).

## <a name="create-a-groupme-application"></a>Creación de una aplicación GroupMe

[Cree una aplicación GroupMe](https://dev.groupme.com/applications/new) para su bot.

Use esta dirección URL de devolución de llamada: `https://groupme.botframework.com/Home/Login`

![Creación de una aplicación](~/media/channels/GM-StepApp.png)

## <a name="gather-credentials"></a>Obtención de las credenciales

1. En el campo **Redirect URL** (URL de redireccionamiento), copie el valor que aparece después de **client_id=**.
2. Copie el valor de **Access Token** (Token de acceso).

![Copia del identificador de cliente y el token de acceso](~/media/channels/GM-StepClientId.png)


## <a name="submit-credentials"></a>Envío de las credenciales

1. En dev.botframework.com, pegue el valor **client_id** que acaba de copiar en el campo **Client ID** (Id. de cliente).
2. Pegue el valor de **Access Token** (Token de acceso) en el campo **Access Token** (Token de acceso).
2. Haga clic en **Save**(Guardar).

![Escribir credenciales](~/media/channels/GM-StepClientIDToken.png)

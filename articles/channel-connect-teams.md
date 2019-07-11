---
title: Conexión de un bot a Teams | Microsoft Docs
description: Aprenda a configurar un bot para acceder mediante la interfaz de Team.
keywords: Teams, bot channel, configure Teams
author: kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/05/2019
ms.openlocfilehash: d2609e4294416691e156ba3dbd09eabc8e0d3423
ms.sourcegitcommit: b498649da0b44f073dc5b23c9011ea2831edb31e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/05/2019
ms.locfileid: "67592234"
---
# <a name="connect-a-bot-to-teams"></a>Conexión de un bot a Teams

Para agregar el canal de Microsoft Teams, abra el bot en [Azure Portal](https://portal.azure.com), haga clic en la hoja **Canales** y luego en **Teams**.

![Agregar un canal de Teams](media/teams/connect-teams-channel.png)

A continuación, haga clic en **Guardar**.

![Guardar un canal de Teams](media/teams/save-teams-channel.png)

Después de agregar el canal de Teams, vaya a la página **Canales** y haga clic en **Get bot embed code** (Obtener el código para insertar del bot).

![Obtener código insertado](media/teams/get-embed-code.png)

- Copie la parte _https_ del código que se muestra en el cuadro de diálogo **Get bot embed code** (Obtener el código para insertar del bot). Por ejemplo, `https://teams.microsoft.com/l/chat/0/0?users=28:b8a22302e-9303-4e54-b348-343232`. 

- En el explorador, pegue esta dirección y elija la aplicación Microsoft Teams (cliente o web) que use para agregar el bot a Teams. Verá que el bot aparece como un contacto al que puede enviar mensajes y del que puede recibir mensajes en Microsoft Teams. 

## <a name="additional-information"></a>Información adicional
Para obtener información específica sobre Microsoft Teams, consulte la [documentación](https://docs.microsoft.com/en-us/microsoftteams/platform/overview) de Teams. 

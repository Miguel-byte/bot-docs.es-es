---
title: Conexión de un bot a Teams | Microsoft Docs
description: Aprenda a configurar un bot para acceder mediante la interfaz de Team.
keywords: Teams, bot channel, configure Teams
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/26/2019
ms.openlocfilehash: 1ba0873c880bfb9b1e0f449039e98859e5571028
ms.sourcegitcommit: 0b647dc6716b0c06f04ee22ebdd7b53039c2784a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2019
ms.locfileid: "70076525"
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

> [!IMPORTANT] 
> No se recomienda agregar un bot por GUID, salvo con fines de prueba. Si lo hace, se limitará en gran medida la funcionalidad de un bot. Los bots de producción se deben agregar a Teams como parte de una aplicación. Consulte [Creación de un bot](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-create) y [Prueba y depuración del bot de Microsoft Teams](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-test).


## <a name="additional-information"></a>Información adicional
Para obtener información específica sobre Microsoft Teams, consulte la [documentación](https://docs.microsoft.com/en-us/microsoftteams/platform/overview) de Teams. 

---
ms.openlocfilehash: b320aadc876074a76fe209ad55a81cb70b1ddcac
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230548"
---
El <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">control de chat en web de código abierto</a> se comunica con los bots mediante [Direct Line API](https://docs.botframework.com/restapi/directline3/#navtitle), lo que permite que se envíen y reciban `activities` entre el cliente y el bot. El tipo más común de actividad es `message`, pero hay otros tipos también. Por ejemplo, el tipo de actividad `typing` indica que un usuario está escribiendo o que el bot está trabajando para compilar una respuesta. 

Puede usar el mecanismo de Backchannel para intercambiar información entre el cliente y el bot sin presentarla al usuario estableciendo el tipo de actividad en `event`. El control de chat en web ignorará automáticamente las actividades en las que `type="event"`.
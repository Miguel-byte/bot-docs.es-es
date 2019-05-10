---
ms.openlocfilehash: cf0e23e349ace78958f861ea2e650a5ebcb78466
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563741"
---
El <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">control de chat en web de código abierto</a> se comunica con los bots mediante [Direct Line API](https://docs.botframework.com/en-us/restapi/directline3/#navtitle), lo que permite que se envíen y reciban `activities` entre el cliente y el bot. El tipo más común de actividad es `message`, pero hay otros tipos también. Por ejemplo, el tipo de actividad `typing` indica que un usuario está escribiendo o que el bot está trabajando para compilar una respuesta. 

Puede usar el mecanismo de Backchannel para intercambiar información entre el cliente y el bot sin presentarla al usuario estableciendo el tipo de actividad en `event`. El control de chat en web ignorará automáticamente las actividades en las que `type="event"`.
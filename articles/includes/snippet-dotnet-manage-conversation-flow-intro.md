---
ms.openlocfilehash: 7eed8c8328456bad43f3bdfe0029df09efa062d6
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230537"
---
Este diagrama muestra el flujo de pantalla de una aplicación tradicional en comparación con el flujo del diálogo de un bot. 

![bot](~/media/designing-bots/core/dialogs-screens.png)

En una aplicación tradicional, todo comienza con la **pantalla principal**.
La **pantalla principal** invoca la **pantalla de nuevo pedido**.
La **pantalla de nuevo pedido** permanece en control hasta que cierra o invoca otras pantallas. Si la **pantalla de nuevo pedido** se cierra, se devuelve al usuario a la **pantalla principal**.

En un bot, todo comienza con el **diálogo raíz**. El **diálogo raíz** invoca el **diálogo de nuevo pedido**. En ese momento, el **diálogo de nuevo pedido** toma el control de la conversación y permanece en control hasta que se cierra o invoca otros diálogos. Si el **diálogo de nuevo pedido** se cierra, se devuelve el control de la conversación al **diálogo raíz**.
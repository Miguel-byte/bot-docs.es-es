---
ms.openlocfilehash: 86faca976bc95a5e91e17749096cd148483edc61
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230683"
---
## <a name="payment-process-overview"></a>Información general del proceso de pago

El proceso de pago consta de tres partes distintas:

1. El bot envía una solicitud de pago.

2. El usuario inicia sesión con una cuenta de Microsoft para proporcionar la información de pago, de envío y de contacto. Las devoluciones de llamada se envían al bot para indicar en qué momento el bot debe realizar determinadas operaciones (actualizar la dirección de envío, actualizar la opción de envío, realizar el pago).

3. El bot procesa las devoluciones de llamada que recibe, incluida la actualización de la dirección de envío, la actualización de la opción de envío y la realización del pago. 

El bot debe implementar solo el primer y el tercer paso de este proceso; el paso dos se lleva a cabo fuera del contexto del bot. 

---
ms.openlocfilehash: 91b2729aa10c8ac1985e62845126296e8141ef77
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230649"
---
> [!IMPORTANT]
> El subproceso que administra el turno de bot principal se ocupa de desechar el objeto de contexto cuando termina. **Asegúrese de usar `await` para las llamadas de actividad**, a fin de que el subproceso principal espere la actividad generada antes de finalizar su procesamiento y desechar el contexto de turno. De otro modo, si una respuesta (incluidos sus controladores) tarda mucho e intenta actuar sobre el objeto de contexto, es posible que reciba un error relativo a que _el contexto se ha eliminado_.
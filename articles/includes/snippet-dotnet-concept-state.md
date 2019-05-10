---
ms.openlocfilehash: 0b991c438c0006d1fb4bafa90982f73f4a18be77
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563448"
---
Bot Builder Framework permite que el bot almacene y recupere datos de estado asociados a un usuario, una conversación o un usuario específico en el contexto de una conversación específica. Los datos de estado se pueden usar para muchos propósitos, como determinar dónde se dejó la conversación anterior o simplemente saludar a un usuario por su nombre cuando vuelva. Si almacena las preferencias del usuario, puede usar esa información para personalizar la conversación la próxima vez que chatee. Por ejemplo, podría alertar al usuario sobre un artículo de noticias que trate un tema que le interese, o bien cuando haya una cita disponible. 

Para fines de creación de prototipos y pruebas, puede usar el almacenamiento de datos en memoria de Bot Builder Framework. Para los bots de producción, puede implementar su propio adaptador de almacenamiento o usar una de las extensiones de Azure. Las extensiones de Azure permiten almacenar datos de estado del bot en Table Storage, Cosmos DB o SQL. En este artículo se muestra cómo usar el adaptador de almacenamiento en memoria para almacenar datos de estado del bot. 

> [!IMPORTANT]
> No se recomienda usar la API de servicio de estado de Bot Framework para entornos de producción y es posible que se encuentre en desuso en una versión futura. No obstante, sí que le recomendamos que actualice el código del bot para que use el adaptador de almacenamiento en memoria para realizar pruebas o usar una de las **extensiones de Azure** para bots de producción.

---
ms.openlocfilehash: 12ead266dea859c84450e08ae12ed98d4952d698
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54226020"
---
La funcionalidad **middleware** de Bot Framework SDK permite al bot interceptar todos los mensajes que se intercambian entre usuario y bot. Para cada mensaje que se intercepta, puede elegir realizar acciones como guardar el mensaje en un almacén de datos que especifique, lo que crea un registro de la conversación o inspeccionar el mensaje de alguna manera y realizar cualquier acción que el código especifique. 

> [!NOTE]
> Bot Framework no guarda automáticamente los detalles de la conversación, ya que de hacerlo podría capturar potencialmente información privada que bots y usuarios no desean compartir con terceros. Si el bot guarda los detalles de la conversación, debe describir lo que se hará con los datos y comunicarlo al usuario.
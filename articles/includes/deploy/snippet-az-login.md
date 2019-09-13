---
ms.openlocfilehash: c664749bc6ad63d1b0f60e001b2603898e58a9ea
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386006"
---
Una vez que haya creado y probado localmente un bot, puede implementarlo en Azure. Abra un símbolo del sistema para iniciar sesión en Azure Portal.

```cmd
az login
```
Se abrirá una ventana del explorador, lo que le permite iniciar sesión.

> [!NOTE]
> Si implementa el bot en una nube que no es de Azure, como Gov (US), debe ejecutar `az cloud set --name <name-of-cloud>` antes de `az login`, donde &lt;name-of-cloud> es el nombre de una nube registrada, como `AzureUSGovernment`. Si desea volver a la nube pública, puede ejecutar `az cloud set --name AzureCloud`. 

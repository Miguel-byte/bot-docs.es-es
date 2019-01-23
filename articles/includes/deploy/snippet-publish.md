---
ms.openlocfilehash: 88732d2d5490962d7a899d936767e7dd148e94c5
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360899"
---
Publicación del bot local en Azure. Este paso puede tardar unos minutos.

```cmd
az bot publish --name <bot-resource-name> --proj-name "<project-file-name>" --resource-group <resource-group-name> --code-dir <directory-path> --verbose --version v4
```

| Opción | DESCRIPCIÓN |
|:---|:---|
| --name | El nombre de recurso del bot en Azure. |
| --proj-name | Para C#, use el nombre de archivo del proyecto de inicio (sin .csproj) que se tiene que publicar. Por ejemplo: `EnterpriseBot`. Para Node.js, use el punto de entrada principal del bot. Por ejemplo, `index.js`. |
| --resource-group | Nombre del grupo de recursos. |
| --code-dir | El directorio desde el que cargar el código del bot. |

Una vez que esto se complete con un mensaje de "implementación correcta", el bot estará implementado en Azure.
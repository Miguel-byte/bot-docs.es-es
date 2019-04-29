---
ms.openlocfilehash: bb7520e8e99ad6326d7d00d8190dae306bf11afa
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2019
ms.locfileid: "59905168"
---
Publicación del bot local en Azure. Este paso puede tardar unos minutos.

```cmd
az bot publish --name <bot-resource-name> --proj-file-path "<project-file-name>" --resource-group <resource-group-name> --code-dir <directory-path> --verbose --version v4
```

| Opción | DESCRIPCIÓN |
|:---|:---|
| --name | El nombre de recurso del bot en Azure. |
| --proj-file-path | Para C#, use el nombre de archivo del proyecto de inicio (sin .csproj) que se tiene que publicar. Por ejemplo: `EnterpriseBot`. Para Node.js, use el punto de entrada principal del bot. Por ejemplo, `index.js`. |
| --resource-group | Nombre del grupo de recursos. |
| --code-dir | El directorio desde el que cargar el código del bot. |

Una vez que esto se complete con un mensaje de "implementación correcta", el bot estará implementado en Azure.

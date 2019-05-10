---
ms.openlocfilehash: 867e65b25878f810e3247eb3cace95f4d31e11db
ms.sourcegitcommit: 4ff7a8772124a567f43e2c3e13aded368c4002e3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/03/2019
ms.locfileid: "65035731"
---
Use un directorio temporal fuera del directorio de proyecto actual. 

Este comando creará un subdirectorio en la ruta de guardar. No obstante, la ruta especificada ya debe existir.

```cmd
az bot download --name <bot-resource-name> --resource-group <resource-group-name> --save-path "<path>"
```

| Opción | DESCRIPCIÓN |
|:---|:---|
| --name | El nombre del bot en Azure. |
| --resource-group | El nombre del grupo de recursos en el que está el bot. |
| --save-path | Un directorio existente en el que descargar el código del bot. |
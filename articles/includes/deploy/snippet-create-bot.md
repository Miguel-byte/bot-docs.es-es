---
ms.openlocfilehash: f95c8a37b1207a26dab0a714b86412a9dba2dcb4
ms.sourcegitcommit: 4ff7a8772124a567f43e2c3e13aded368c4002e3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/03/2019
ms.locfileid: "65035757"
---
```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```

| Opción | DESCRIPCIÓN |
|:---|:---|
| --name | Nombre único que se usa para implementar el bot en Azure. El nombre puede ser el mismo que el de su bot local. NO incluya espacios ni guiones bajos en el nombre. |
| --location | Ubicación geográfica utilizada para crear los recursos del servicio de bot. Por ejemplo, `eastus`, `westus`, `westus2`, etc. |
| --lang | El idioma que se usará para crear el bot: `Csharp`, o `Node`; el valor predeterminado es `Csharp`. |
| --resource-group | Nombre del grupo de recursos en el que se crea el bot. Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
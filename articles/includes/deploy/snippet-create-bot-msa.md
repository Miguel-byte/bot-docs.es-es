---
ms.openlocfilehash: 14d9632ad578014a36b5f13e6dee883e2a6e1722
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/19/2019
ms.locfileid: "67252645"
---
1. Vaya a [**Portal de registro de aplicaciones**](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade).
1. Haga clic en **Agregar una aplicación** para registrar la aplicación, cree el **Id. de la aplicación** y **genere una nueva contraseña**. Si ya tiene una aplicación y una contraseña pero no recuerda la contraseña, deberá generar una nueva en la sección de secretos de aplicación.
1. Guarde tanto el identificador de la aplicación como la contraseña que acaba de generar, con el fin de que pueda usarlos con el comando `az bot create`.  

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name> --appid "<application-id>" --password "<application-password>" --verbose
```

| Opción | DESCRIPCIÓN |
|:---|:---|
| --name | Nombre único que se usa para implementar el bot en Azure. El nombre puede ser el mismo que el de su bot local. NO incluya espacios ni guiones bajos en el nombre. |
| --location | Ubicación geográfica utilizada para crear los recursos del servicio de bot. Por ejemplo, `eastus`, `westus`, `westus2`, etc. |
| --lang | El idioma que se usará para crear el bot: `Csharp`, o `Node`; el valor predeterminado es `Csharp`. |
| --resource-group | Nombre del grupo de recursos en el que se crea el bot. Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --appid | El identificador de cuenta Microsoft (identificador de MSA) que se va a usar con el bot. |
| --password | La contraseña de cuenta Microsoft (MSA) del bot. |

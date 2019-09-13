---
ms.openlocfilehash: fd279af81e0c94ac22f54429cb8ae330e5d60661
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386027"
---
Va a crear un nuevo grupo de recursos en Azure y, a continuación, usará la plantilla de ARM para crear los recursos que se especifican en ella. En este caso, se proporcionará un plan de App Service, una aplicación web y el registro de canales de bot.

```cmd
az deployment create --name "<name-of-deployment>" --template-file "template-with-new-rg.json" --location "location-name" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" botSku=F0 newAppServicePlanName="<name-of-app-service-plan>" newWebAppName="<name-of-web-app>" groupName="<new-group-name>" groupLocation="<location>" newAppServicePlanLocation="<location>"
```

| Opción   | DESCRIPCIÓN |
|:---------|:------------|
| Nombre | Nombre descriptivo para la implementación. |
| template-file | La ruta de acceso a la plantilla de ARM. Puede usar el archivo `template-with-new-rg.json` que se proporciona en la carpeta `deploymentTemplates` del proyecto. |
| location |Ubicación. Los valores de: `az account list-locations`. Puede configurar la ubicación predeterminada mediante `az configure --defaults location=<location>`. |
| parameters | Proporcione los valores de los parámetros de implementación. El valor `appId` que se obtuvo al ejecutar el comando `az ad app create`. `appSecret` es la contraseña que proporcionó en el paso anterior. El parámetro `botId` debe ser único globalmente y se utiliza como identificador inmutable del bot. También sirve para configurar el nombre para mostrar del bot, nombre que es mutable. `botSku` es el plan de tarifa y puede ser F0 (gratis) o S1 (Estándar). `newAppServicePlanName` es el nombre del plan de App Service. `newWebAppName` es el nombre de la aplicación web que va a crear. `groupName` es el nombre del grupo de recursos de Azure que va a crear. `groupLocation` es la ubicación del grupo de recursos de Azure. `newAppServicePlanLocation` es la ubicación del plan de App Service. |

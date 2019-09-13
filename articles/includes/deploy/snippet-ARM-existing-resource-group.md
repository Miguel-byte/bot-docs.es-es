---
ms.openlocfilehash: 029275eae6f7b0b4448613ede898575eb491a56f
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386068"
---
Cuando se usa un grupo de recursos existente, puede usar un plan de App Service que ya exista o crear uno nuevo. A continuación, se muestran los pasos para ambas opciones. 

**Opción 1: Plan de App Service existente** 

En este caso, se usará un plan de App Service ya existente, pero creando una aplicación web y el registro de canales de bot. 

> [!NOTE]
> Este comando establece el identificador y el nombre para mostrar del bot. El parámetro `botId` debe ser único globalmente y se utiliza como identificador inmutable del bot. El nombre para mostrar del bot es mutable.

```cmd
az group deployment create --name "<name-of-deployment>" --resource-group "<name-of-resource-group>" --template-file "template-with-preexisting-rg.json" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" newWebAppName="<name-of-web-app>" existingAppServicePlan="<name-of-app-service-plan>" appServicePlanLocation="<location>"
```

**Opción 2: Nuevo plan de App Service**

En este caso, se creará un plan de App Service, una aplicación web y el registro de canales de bot. 

```cmd
az group deployment create --name "<name-of-deployment>" --resource-group "<name-of-resource-group>" --template-file "template-with-preexisting-rg.json" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" newWebAppName="<name-of-web-app>" newAppServicePlanName="<name-of-app-service-plan>" appServicePlanLocation="<location>"
```

| Opción   | DESCRIPCIÓN |
|:---------|:------------|
| Nombre | Nombre descriptivo para la implementación. |
| resource-group | El nombre del grupo de recursos de Azure. |
| template-file | La ruta de acceso a la plantilla de ARM. Puede usar el archivo `template-with-preexisting-rg.json` que se proporciona en la carpeta `deploymentTemplates` del proyecto. |
| location |Ubicación. Los valores de: `az account list-locations`. Puede configurar la ubicación predeterminada mediante `az configure --defaults location=<location>`. |
| parameters | Proporcione los valores de los parámetros de implementación. El valor `appId` que se obtuvo al ejecutar el comando `az ad app create`. `appSecret` es la contraseña que proporcionó en el paso anterior. El parámetro `botId` debe ser único globalmente y se utiliza como identificador inmutable del bot. También sirve para configurar el nombre para mostrar del bot, nombre que es mutable. `newWebAppName` es el nombre de la aplicación web que va a crear. `newAppServicePlanName` es el nombre del plan de App Service. `newAppServicePlanLocation` es la ubicación del plan de App Service. |

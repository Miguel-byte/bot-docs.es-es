---
title: Implementación del bot | Microsoft Docs
description: Implemente su bot en la nube de Azure.
keywords: deploy bot, azure deploy bot, publish bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: afb27ad20ec8585c2ca30810a9be6858adc17187
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693519"
---
# <a name="deploy-your-bot"></a>Implementación del bot

[!INCLUDE [applies-to](./includes/applies-to.md)]

En este artículo, le vamos a mostrar cómo implementar el bot en Azure. Es conveniente leerlo antes de seguir los pasos, con el fin de tener un conocimiento completo de lo que implica la implementación de un bot.

## <a name="prerequisites"></a>Requisitos previos
- Si no tiene una suscripción a Azure, cree una [cuenta](https://azure.microsoft.com/free/) antes de empezar.
- Un bot de CSharp, JavaScript o TypeScript que haya desarrollado en la máquina local.
- La versión más reciente de la [CLI de Azure](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest).

## <a name="1-prepare-for-deployment"></a>1. Preparación de la implementación
Cuando crea un bot mediante las plantillas de Visual Studio o Yeoman, el código fuente que se genera, contiene una carpeta `deploymentTemplates` con plantillas de ARM. El proceso de implementación que se documenta aquí utiliza la plantilla ARM para aprovisionar los recursos necesarios para el bot en Azure mediante la CLI de Azure. 

> [!IMPORTANT]
> Con el lanzamiento de Bot Framework SDK 4.3, la utilización del archivo .bot está _en desuso_ en favor de un archivo appsettings.json o .env para administrar los recursos. Para más información sobre cómo migrar la configuración del archivo .bot al archivo appsettings.json o .env, consulte [Administración de recursos del bot](v4sdk/bot-file-basics.md).

### <a name="login-to-azure"></a>Inicio de sesión en Azure

Ya ha creado y probado localmente un bot y ahora quiere implementarlo en Azure. Abra un símbolo del sistema para iniciar sesión en Azure Portal.

```cmd
az login
```
Se abrirá una ventana del explorador, lo que le permite iniciar sesión.

### <a name="set-the-subscription"></a>Establecimiento de la suscripción
Establezca la suscripción predeterminada que desea usar.

```cmd
az account set --subscription "<azure-subscription>"
```

Si no está seguro de la suscripción que debe usar para implementar el bot y desea ver la lista de suscripciones para su cuenta, use el comando `az account list`. Vaya a la carpeta del bot.

### <a name="create-an-app-registration"></a>Creación de un registro de aplicación
Registrar la aplicación significa que puede usar Azure AD para autenticar usuarios y solicitar acceso a recursos de usuarios. El bot requiere una aplicación registrada en Azure que le proporcione acceso al servicio Bot Framework para enviar y recibir mensajes autenticados. Para crear el registro de una aplicación mediante la CLI de Azure, ejecute el siguiente comando:

```cmd
az ad app create --display-name "displayName" --password "AtLeastSixteenCharacters_0" --available-to-other-tenants
```

| Opción   | DESCRIPCIÓN |
|:---------|:------------|
| display-name | Nombre para mostrar de la aplicación. |
| contraseña | Contraseña de aplicación, también conocida como "secreto de cliente". La contraseña debe tener al menos 16 caracteres, contener al menos un carácter alfabético en mayúsculas o minúsculas, y contener al menos 1 carácter especial|
| available-to-other-tenants| Se puede usar la aplicación desde cualquier inquilino de Azure AD. Debe ser `true` para permitir al bot que funcione con los canales de Azure Bot Service.|

El comando anterior da como resultado un JSON con la clave `appId`, guarde el valor de esta clave para la implementación de ARM, donde se usará para el parámetro `appId`. La contraseña proporcionada se usará para el parámetro `appSecret`.

Puede implementar el bot en un grupo de recursos nuevo o en uno ya existente. Elija la opción que mejor funcione en su caso.

# <a name="deploy-via-arm-template-with-new-resource-grouptabnewrg"></a>[Implementación mediante la plantilla de ARM (con un **grupo de recursos** nuevo)](#tab/newrg)

### <a name="create-azure-resources"></a>Creación de recursos de Azure

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

# <a name="deploy-via-arm-template-with-existing--resource-grouptaberg"></a>[Implementación mediante la plantilla de ARM (con un **grupo de recursos** ya existente)](#tab/erg)

### <a name="create-azure-resources"></a>Creación de recursos de Azure

Cuando se usa un grupo de recursos existente, puede usar un plan de App Service que ya exista o crear uno nuevo. A continuación, se muestran los pasos para ambas opciones. 

**Opción 1: Plan de App Service existente** 

En este caso, se usará un plan de App Service ya existente, pero creando una aplicación web y el registro de canales de bot. 

_Nota: El parámetro botId debe ser único globalmente y se utiliza como identificador inmutable del bot. También sirve para configurar el nombre para mostrar del bot, nombre que es mutable._

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

---

### <a name="retrieve-or-create-necessary-iiskudu-files"></a>Recuperación o creación de los archivos IIS/Kudu necesarios

**Para bots de C#**

```cmd
az bot prepare-deploy --lang Csharp --code-dir "." --proj-file-path "MyBot.csproj"
```

Debe proporcionar la ruta de acceso al archivo .csproj relacionada con --code-dir. Esto puede realizarse mediante el argumento --proj-file-path. El comando debería resolver --code-dir y --proj-file-path en "./MyBot.csproj"

**Para bots de JavaScript**

```cmd
az bot prepare-deploy --code-dir "." --lang Javascript
```

Este comando capturará un archivo web.config que es necesario para que las aplicaciones de Node.js funcionen con IIS en Azure App Services. Asegúrese de que el archivo web.config se guarda en la raíz del bot.

**Para bots de TypeScript**

```cmd
az bot prepare-deploy --code-dir "." --lang Typescript
```

Este comando funciona de forma similar al de JavaScript que se mencionó anteriormente, pero para un bot de TypeScript.

### <a name="zip-up-the-code-directory-manually"></a>Comprimir el directorio de código manualmente

Cuando se usa [ZIP deploy API](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url) para implementar el código del bot, el comportamiento de la aplicación web o de Kudu es el siguiente:

_Kudu supone de forma predeterminada que las implementaciones de los archivos ZIP están listas para ejecutarse y que no necesitan pasos de compilación adicionales durante la implementación como, por ejemplo, npm install o dotnet restore/dotnet publish._

Por tanto, es importante incluir el código compilado con todas las dependencias necesarias en el archivo ZIP que se va a implementar en la aplicación web ya que, de lo contrario, el bot no funcionará como se espera.

> [!IMPORTANT]
> Antes de comprimir los archivos de proyecto, asegúrese de que está _en_ la carpeta correcta. 
> - Para bots de C#, es la carpeta que contiene el archivo .csproj. 
> - Para bots de JS, es la carpeta que contiene el archivo app.js o index.js. 
>
> Seleccione todos los archivos y comprímalos **mientras está en esa carpeta** y, a continuación, ejecute el comando también en esa carpeta.
>
> Si la ubicación de la carpeta raíz es incorrecta, el **bot no podrá ejecutarse en Azure Portal**.

## <a name="2-deploy-code-to-azure"></a>2. Implementación del código en Azure
En este momento, estamos preparados para implementar el código en la aplicación web de Azure. Ejecute el siguiente comando en la línea de comandos para realizar la implementación mediante la implementación de inserción del archivo ZIP de Kudu para una aplicación web.

```cmd
az webapp deployment source config-zip --resource-group "<new-group-name>" --name "<name-of-web-app>" --src "code.zip" 
```

| Opción   | DESCRIPCIÓN |
|:---------|:------------|
| resource-group | El nombre del grupo de recursos de Azure que ha creado anteriormente. |
| Nombre | Nombre de la aplicación web que ha usado anteriormente. |
| src  | La ruta de acceso al archivo ZIP que ha creado. |

## <a name="3-test-in-web-chat"></a>3. Probar en Chat en web
- En Azure Portal, vaya a la hoja del bot de la aplicación web.
- En la sección **Administración del bot**, haga clic en **Probar en Chat en web**. Azure Bot Service cargará el control Chat en web y se conectará al bot.
- Espere unos segundos después de una implementación correcta; también puede reiniciar la aplicación web para borrar toda la memoria caché. Vuelva a la hoja Bot de aplicación web y pruebe con el Chat en web proporcionado en Azure Portal.

## <a name="additional-information"></a>Información adicional
La implementación de un bot en Azure conllevará el pago de los servicios que se usen. El artículo acerca de la [facturación y administración de costos](https://docs.microsoft.com/en-us/azure/billing/) sirve de ayuda para conocer la facturación de Azure, para supervisar el uso y los costos, y para administrar cuentas y suscripciones.

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Configuración de la implementación continua](bot-service-build-continuous-deployment.md)

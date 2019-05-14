---
ms.openlocfilehash: 4b5181babf728861107a0c7bc28f844491761a7a
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033889"
---
Antes de comenzar la implementación, asegúrese de que tiene la versión más reciente de la [CLI de Azure](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) y de la [CLI de DotNet](https://dotnet.microsoft.com/download). Si no tiene la CLI de DotNet, instálela mediante la opción Runtime de .Net Core desde el vínculo que se ha proporcionado. 

### <a name="login-to-azure-cli-and-set-your-subscription"></a>Inicio de sesión en la CLI de Azure y establecimiento de la suscripción
Ya ha creado y probado localmente un bot y ahora quiere implementarlo en Azure. Abra un símbolo del sistema para iniciar sesión en Azure Portal.

```cmd
az login
```
### <a name="set-the-subscription"></a>Establecimiento de la suscripción

Establezca la suscripción predeterminada que desea usar.

```cmd
az account set --subscription "<azure-subscription>"
```

Si no está seguro de la suscripción que debe usar para implementar el bot y desea ver la lista de suscripciones para su cuenta, use el comando `az account list`.

Vaya a la carpeta del bot.
`cd <local-bot-folder>`

### <a name="create-a-web-app-bot-in-azure"></a>Creación de un bot de aplicación web en Azure 

Si no dispone de un grupo de recursos para publicar el bot local, créelo:

```cmd
az group create --name <resource-group-name> --location <geographic-location> --verbose
```

| Opción     | DESCRIPCIÓN |
|:-----------|:---|
| Nombre     | Nombre único para el grupo de recursos. NO incluya espacios ni guiones bajos en el nombre. |
| location | Ubicación geográfica utilizada para crear el grupo de recursos. Por ejemplo, `eastus`, `westus`, `westus2`, etc. Use `az account list-locations` para obtener una lista de ubicaciones. |

A continuación, cree el recurso de bot en el cual publicará el bot. Esto aprovisionará los recursos necesarios en Azure y creará una aplicación web de bot, que sobrescribirá con el bot local. 

Antes de continuar, lea las instrucciones que se le aplican en función del tipo de cuenta de correo electrónico que usa para iniciar sesión en Azure.

#### <a name="msa-email-account"></a>Cuenta de correo electrónico MSA
Si va a usar una cuenta de correo electrónico MSA, tendrá que crear el identificador y la contraseña de la aplicación en el Portal de registro de aplicaciones que va a usar con el comando `az bot create`.
1. Vaya a [**Portal de registro de aplicaciones**](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade).
1. Haga clic en **Agregar una aplicación** para registrar la aplicación, cree el **Id. de la aplicación** y **genere una nueva contraseña**. Si ya tiene una aplicación y una contraseña pero no recuerda la contraseña, deberá generar una nueva en la sección de secretos de aplicación.
1. Guarde tanto el identificador de la aplicación como la contraseña que acaba de generar, con el fin de que pueda usarlos con el comando `az bot create`.  

```cmd
az bot create --kind webapp --name <bot-name-in-azure> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name> --appid "<application-id>" --password "<application-password>" --verbose
```

| Opción | DESCRIPCIÓN |
|:---|:---|
| Nombre | Nombre único que se usa para implementar el bot en Azure. El nombre puede ser el mismo que el de su bot local. NO incluya espacios ni guiones bajos en el nombre. |
| location | Ubicación geográfica utilizada para crear los recursos del servicio de bot. Por ejemplo, `eastus`, `westus`, `westus2`, etc. |
| resource-group | Nombre del grupo de recursos en el que se crea el bot. Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| appid | El identificador de cuenta Microsoft (identificador de MSA) que se va a usar con el bot. |
| contraseña | La contraseña de cuenta Microsoft (MSA) del bot. |

#### <a name="business-or-school-account"></a>Cuenta empresarial o educativa

```cmd
az bot create --kind webapp --name <bot-name-in-azure> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```
| Opción | DESCRIPCIÓN |
|:---|:---|
| Nombre | Nombre único que se usa para implementar el bot en Azure. El nombre puede ser el mismo que el de su bot local. NO incluya espacios ni guiones bajos en el nombre. |
| location | Ubicación geográfica utilizada para crear los recursos del servicio de bot. Por ejemplo, `eastus`, `westus`, `westus2`, etc. |
| lang | El idioma que se usará para crear el bot: `Csharp`, o `Node`; el valor predeterminado es `Csharp`. |
| resource-group | Nombre del grupo de recursos en el que se crea el bot. Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |

#### <a name="update-appsettingsjson-or-env-file"></a>Actualización de los archivos appsettings.json o .env
Tras crear el bot es aconsejable ver que la siguiente información se muestra en la ventana de consola: 

```JSON
{
  "appId": "as234-345b-4def-9047-a8a44b4s",
  "appPassword": "34$#w%^$%23@334343",
  "endpoint": "https://mybot.azurewebsites.net/api/messages",
  "id": "mybot",
  "name": "mybot",
  "resourceGroup": "botresourcegroup",
  "serviceName": "mybot",
  "subscriptionId": "234532-8720-5632-a3e2-a1qw234",
  "tenantId": "32f955bf-33f1-43af-3ab-23d009defs47",
  "type": "abs"
}
```

Deberá copiar los valores de `appId` y `appPassword`, y pegarlos en los archivos appsettings.json o env. Por ejemplo: 

```JSON
{
  MicrosoftAppId: "as234-345b-4def-9047-a8a44b4s",
  MicrosoftAppPassword: "34$#w%^$%23@334343"
}
```
Tenga en cuenta que si los archivos appsettings.json o env tiene claves adicionales para otros servicios que haya aprovisionado para el bot, no debe eliminar dichas entradas.

Guarde el archivo.

A continuación, en función del lenguaje de programación (**C#**  o **JS**) usó para crear el bot, siga los pasos que se le aplican.

**C# Bot:** 

Abra un símbolo del sistema y vaya a la carpeta del proyecto. Ejecute los siguientes comandos desde la línea de comandos.

| Tarea | Get-Help |
|:-----|:--------|
| 1. Restauración de las dependencias del proyecto | `dotnet restore`|
| 2. Compilación del proyecto     | `dotnet build` |
| 3. Compresión de los archivos del proyecto | Utilice cualquier utilidad para comprimir los archivos del proyecto. Vaya a la carpeta que contiene el archivo .csproj y seleccione todos los archivos y carpetas en este nivel para crear la carpeta comprimida. |
| 4. Establecimiento de la configuración de la implementación de la compilación | `az webapp config appsettings set --resource-group <resource-group-name> --name <bot-name> --settings SCM_DO_BUILD_DEPLOYMENT=false`|
| 5. Establecimiento de los argumentos del generador de scripts | `az webapp config appsettings set --resource-group <resource-group-name> --name <bot-name> --settings SCM_SCRIPT_GENERATOR_ARGS="--aspNetCore mybot.csproj"`|

**Bot JS:**
1. Descargue el archivo web.config [aquí](https://github.com/projectkudu/kudu/wiki/Using-a-custom-web.config-for-Node-apps) y guárdelo en la carpeta del proyecto. 
1. Edite el archivo y reemplace todas las apariciones de "server.js" por "index.js". 
1. Guarde el archivo.

Abra un símbolo del sistema y vaya a la carpeta del proyecto. Ejecute los siguientes comandos desde la línea de comandos.

| Tarea | Get-Help |
|:-----|:--------|
| 1. Instalación de módulos de Node | `npm install` |
| 2. Compresión de los archivos del proyecto | Utilice cualquier utilidad para comprimir los archivos del proyecto. Vaya a la carpeta que contiene el archivo .csproj y seleccione todos los archivos y carpetas en este nivel para crear la carpeta comprimida. |
| 3. Establecimiento de la configuración de la implementación de la compilación | `az webapp config appsettings set --resource-group <resource-group-name> --name <bot-name> --settings SCM_DO_BUILD_DEPLOYMENT=false`|

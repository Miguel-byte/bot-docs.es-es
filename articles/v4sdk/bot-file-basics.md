---
title: Administración de los recursos con un archivo .bot | Microsoft Docs
description: Describe el propósito y el uso del archivo de bot.
keywords: archivo de bot, .bot, archivo .bot, recursos del bot, administrar recursos de bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 03/30/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 14552c55da4b1f9b581b81917496de179e92762b
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2019
ms.locfileid: "58811507"
---
# <a name="manage-bot-resources"></a>Administración de recursos de bot

Los bots suelen usar servicios diferentes, como [LUIS.ai](https://luis.ai) o [QnaMaker.ai](https://qnamaker.ai). Cuando está desarrollando un bot, es preciso que pueda realizar un seguimiento de todos ellos. Puede usar diversos métodos como appsettings.json, web.config o .env. 

> [!IMPORTANT]
> Antes de la versión Bot Framework SDK 4.3, se ofrecía el archivo .bot como un mecanismo para administrar los recursos. Sin embargo, en adelante, se recomienda usar el archivo appsettings.json o .env para administrar estos recursos. Los bots que utilicen el archivo .bot seguirán funcionando por ahora, aunque este archivo está **_en desuso_**. Si ha estado usando un archivo .bot para administrar recursos, siga los pasos aplicables para migrar la configuración. 

## <a name="migrating-settings-from-bot-file"></a>Migración de la configuración a partir de un archivo .bot
Las siguientes secciones explican cómo migrar la configuración a partir del archivo .bot. Siga las instrucciones del escenario que sea aplicable en su caso.

**Escenario 1: bot local que tiene un archivo .bot**

En este escenario tiene un bot local que usa un archivo .bot, pero _este no se ha migrado_ a Azure Portal. Siga los pasos siguientes para migrar la configuración del archivo .bot al archivo appsettings.json o .env.

- Si el archivo .bot está cifrado, deberá descifrarlo mediante el comando siguiente:

```cli
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear` command.
```

- Abra el archivo .bot descifrado, copie los valores y agréguelos al archivo appsettings.json o .env.
- Actualice el código para que lea la configuración del archivo appsettings.json o .env.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En el método `ConfigureServices`, use el objeto de configuración que proporciona ASP.NET Core, por ejemplo: 

**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

En JavaScript, haga referencia a las variables .env fuera del objeto `process.env`, por ejemplo:
   
**index.js**

```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```
---

Si es necesario, aprovisione recursos y conéctelos al bot mediante el archivo appsettings.json o .env.

**Escenario 2: Bot implementado en Azure con un archivo .bot**

En este escenario ya ha implementado un bot en Azure Portal mediante el archivo .bot y ahora desea migrar la configuración del archivo .bot al archivo appsettings.json o .env.

- Descargue el código del bot de Azure Portal. Al descargar el código, se le pedirá que incluya el archivo appsettings.json o .env que contiene los valores de MicrosoftAppId y MicrosoftAppPassword y el resto de configuraciones adicionales. 
- Abra el archivo appsettings.json o .env _descargado_ y copie la configuración en el archivo appsettings.json o .env _local_. No olvide quitar las entradas botSecret y botFilePath del archivo appsettings.json o .env local.
- Actualice el código para que lea la configuración del archivo appsettings.json o .env.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
En el método `ConfigureServices`, use el objeto de configuración que proporciona ASP.NET Core, por ejemplo: 

**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
En JavaScript, haga referencia a las variables .env fuera del objeto `process.env`, por ejemplo:
   
**index.js**

```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```
---

También tendrá que quitar `botFilePath` y `botFileSecret` de la sección **Configuración de la aplicación** en **Azure Portal**.

_Si es necesario_, aprovisione recursos y conéctelos al bot mediante el archivo appsettings.json o .env.

**Escenario 3: bots que utilizan el archivo appsettings.json o .env**

En este escenario se aborda un caso en el que está empezando a desarrollar bots mediante SDK 4.3 desde cero y no dispone de ningún archivo .bot ya existente para migrar. Todas las configuraciones que desea usar en su bot están en el archivo appsettings.json o .env como se muestra a continuación:

```JSON
{
  "MicrosoftAppId": "<your-AppId>",
  "MicrosoftAppPassword": "<your-AppPwd>"
}
```

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Para leer la configuración anterior en el código C#, usará el objeto de configuración que proporciona ASP .NET Core, por ejemplo: **Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
En JavaScript, haga referencia a las variables .env fuera del objeto `process.env`, por ejemplo: **index.js**.
```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```

---

Si es necesario, aprovisione recursos y conéctelos al bot mediante el archivo appsettings.json o .env.


## <a name="faq"></a>Preguntas más frecuentes
**P:** Quiero crear un bot V4 en Azure Portal. ¿Cómo ha cambiado la experiencia de Azure Portal con la eliminación del archivo .bot?

**R:** Cuando cree un bot en Azure Portal, no se creará el archivo .bot. Puede usar la sección **Configuración de la aplicación** de **Azure Portal** para buscar los identificadores y las claves. Cuando descarga el código, estos valores se almacenan en el archivo appsettings.json o .env. En el bot, puede actualizar el código para que lea la configuración antes de realizar la llamada al servicio individual. Una vez actualizado el código del bot, puede usar az bot publish para implementarlo.

**P:** ¿Qué sucede con los bots V3?

**R:** El escenario para los bots de la versión V3 es similar al de los de la versión V4 sin un archivo de bot. El proceso de implementación seguirá funcionando. 

## <a name="additional-resources"></a>Recursos adicionales
- Para ver los pasos para implementar el bot, consulte el tema sobre [implementación](../bot-builder-deploy-az-cli.md).
- Para proteger las claves y secretos, es recomendable usar Azure Key Vault. Azure Key Vault es una herramienta para almacenar secretos y acceder a ellos de forma segura. Secretos como, por ejemplo, el punto de conexión y las claves de creación del bot. Proporciona una solución de [administración de claves](https://docs.microsoft.com/en-us/azure/key-vault/key-vault-whatis) y facilita la creación y control de las claves de cifrado para tener un control estricto sobre los secretos.


<!--

# Manage resources with a .bot file

Bots usually consume lots of different services, such as [LUIS.ai](https://luis.ai) or [QnaMaker.ai](https://qnamaker.ai). When you are developing a bot, there is no uniform place to store the metadata about the services that are in use.  This prevents us from building tooling that looks at a bot as a whole.

To address this problem, we have created a **.bot file** to act as the place to bring all service references together in one place to 
enable tooling.  For example, the Bot Framework Emulator ([V4](https://aka.ms/Emulator-wiki-getting-started)) uses a  .bot file to create a unified view over the connected services your bot consumes.  

With a .bot file, you can register services like:

* **Localhost** local debugger endpoints
* [**Azure Bot Service**](https://azure.microsoft.com/en-us/services/bot-service/) Azure Bot Service registrations.
* [**LUIS.AI**](https://www.luis.ai/) LUIS gives your bot the ability to communicate with people using natural language.. 
* [**QnA Maker**](https://qnamaker.ai/) Build, train and publish a simple question and answer bot based on FAQ URLs, structured documents or editorial content in minutes.
* [**Dispatch**](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch) models for dispatching across multiple services.
* [**Azure Application Insights**](https://azure.microsoft.com/en-us/services/application-insights/) for insights and bot analytics.
* [**Azure Blob Storage**](https://azure.microsoft.com/en-us/services/storage/blobs/) for bot state persistence. 
* [**Azure Cosmos DB**](https://azure.microsoft.com/en-us/services/cosmos-db/) - globally distributed, multi-model database service to persist bot state.

Apart from these, your bot might rely on other custom services. You can leverage the [generic service](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/add-services.md) capability to connect a generic service configuration.

## When is a .bot file created? 
- If you create a bot using [Azure Bot Service](https://ms.portal.azure.com/#blade/Microsoft_Azure_Marketplace/GalleryResultsListBlade/selectedSubMenuItemId/%7B%22menuItemId%22%3A%22gallery%2FCognitiveServices_MP%2FBotService%22%2C%22resourceGroupId%22%3A%22%22%2C%22resourceGroupLocation%22%3A%22%22%2C%22dontDiscardJourney%22%3Afalse%2C%22launchingContext%22%3A%7B%22source%22%3A%5B%22GalleryFeaturedMenuItemPart%22%5D%2C%22menuItemId%22%3A%22CognitiveServices_MP%22%2C%22subMenuItemId%22%3A%22BotService%22%7D%7D), a .bot file is automatically created for you with list of connected services provisioned. The .bot is encrypted by default.
- If you create a bot using Bot Framework V4 SDK [Template](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) for Visual Studio or using Bot Builder [Yeoman Generator](https://www.npmjs.com/package/generator-botbuilder), a .bot file is automatically created. No connected services are provisioned in this flow and the bot file is not encrypted.
- If you are starting with [BotBuilder-samples](https://github.com/Microsoft/botbuilder-samples), every sample for Bot Framework V4 SDK includes a .bot file and the .bot file is not encrypted. 
- You can also create a bot file using the [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) tool.

## What does a bot file look like? 
Take a look at a sample [.bot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/sample-bot-file.json) file.
To learn about encrypting and decrypting the .bot file, see [Bot Secrets](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/bot-file-encryption.md).

## Why do I need a .bot file?

A .bot file is **not** a requirement to build bots with Bot Framework SDK. You can continue to use appsettings.json, web.config, env, 
keyvault or any mechanism you see fit to keep track of service references and keys that your bot depends on. However, to test
the bot using the Emulator, you'll need a .bot file. The good news is that Emulator can create a .bot file for testing. To do that, 
start the Emulator, click on the **create a new bot configuration** link on the Welcome page. In the dialog box that appears, type a **Bot name** and an **Endpoint URL**. Then connect.

The advantages of using .bot file are:
- Provides a standard way of storing resources regardless of the language/platform you use.   
- Bot Framework Emulator and CLI tools rely on and work great with tracking connected services in a consistent format (in a .bot file) 
- Elegant tooling solutions around services creation and management is harder without a well defined schema (.bot file).  


## Using .bot file in your Bot Framework SDK bot

You can use the .bot file to get service configuration information in your bot's code. The BotFramework-Configuration library available 
for [C#](https://www.nuget.org/packages/Microsoft.Bot.Configuration) and [JS](https://www.npmjs.com/package/botframework-config) helps you load a bot file and supports several methods to query and get the appropriate service configuration information.

## Additional resources
Refer to [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) readme file for more information on using a bot file.

-->


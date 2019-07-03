---
title: Administración de los recursos del bot | Microsoft Docs
description: Describe el propósito y el uso del archivo de bot.
keywords: archivo de bot, .bot, archivo .bot, msbot, recursos del bot, administrar recursos del bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a550de06be6584c5d9ba5eca4ffffffd49a518d2
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404306"
---
# <a name="manage-bot-resources"></a>Administración de recursos de bot

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

Los bots suelen usar servicios diferentes, como [LUIS.ai](https://luis.ai) o [QnaMaker.ai](https://qnamaker.ai). Cuando está desarrollando un bot, es preciso que pueda realizar un seguimiento de todos ellos. Puede usar diversos métodos como appsettings.json, web.config o .env. 

> [!IMPORTANT]
> Antes de la versión Bot Framework SDK 4.3, se ofrecía el archivo .bot como un mecanismo para administrar los recursos. Sin embargo, en adelante, se recomienda usar el archivo appsettings.json o .env para administrar estos recursos. Los bots que utilicen el archivo .bot seguirán funcionando por ahora, aunque este archivo está **_en desuso_** . Si ha estado usando un archivo .bot para administrar recursos, siga los pasos aplicables para migrar la configuración. 

## <a name="migrating-settings-from-bot-file"></a>Migración de la configuración a partir de un archivo .bot
Las siguientes secciones explican cómo migrar la configuración a partir del archivo .bot. Siga las instrucciones del escenario que sea aplicable en su caso.

**Escenario 1 : bot local que tiene un archivo .bot**

En este escenario tiene un bot local que usa un archivo .bot, pero _este no se ha migrado_ a Azure Portal. Siga los pasos siguientes para migrar la configuración del archivo .bot al archivo appsettings.json o .env.

- Si el archivo .bot está cifrado, deberá descifrarlo mediante el comando siguiente:

```cli
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
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

*Si es necesario*, aprovisione recursos y conéctelos al bot mediante el archivo appsettings.json o .env.

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

*Si es necesario*, aprovisione recursos y conéctelos al bot mediante el archivo appsettings.json o .env.

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

## <a name="additional-resources"></a>Recursos adicionales
- Para ver los pasos para implementar el bot, consulte el tema sobre [implementación](../bot-builder-deploy-az-cli.md).
- Aprenda a usar [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-overview) para proteger y administrar claves criptográficas y secretos usados por servicios y aplicaciones en la nube.

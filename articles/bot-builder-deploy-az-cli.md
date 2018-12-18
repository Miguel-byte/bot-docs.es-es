---
title: Implementación de un bot mediante la CLI de Azure | Microsoft Docs
description: Implemente su bot en la nube de Azure.
keywords: implementar bot, implementación de azure, publicar bot, bot de implementación de az, bot de implementación de visual studio, publicación de msbot, clonado de msbot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/11/2018
ms.openlocfilehash: a7cb9cb1e3df14f2f46bc5a4c3a633f5212b5dfd
ms.sourcegitcommit: 0b421ff71617f03faf55ea175fb91d1f9e348523
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/12/2018
ms.locfileid: "53286621"
---
# <a name="deploy-your-bot-using-azure-cli"></a>Implementación de un bot mediante la CLI de Azure

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

Después de que haya creado el bot y lo haya probado localmente, puede implementarlo en Azure para que sea posible acceder a él desde cualquier lugar. La implementación de un bot en Azure conllevará el pago de los servicios que se usen. El artículo acerca de la [facturación y administración de costos](https://docs.microsoft.com/en-us/azure/billing/) sirve de ayuda para conocer la facturación de Azure, para supervisar el uso y los costos, y para administrar cuentas y suscripciones.

En dicho artículo, podrá ver cómo implementar bots de C# y JavaScript en Azure mediante la herramienta `msbot`. Es conveniente leerlo antes de seguir los pasos, con el fin de tener un conocimiento completo de lo que implica la implementación de un bot.


## <a name="prerequisites"></a>Requisitos previos
- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
- Instale el [SDK de .NET Core](https://dotnet.microsoft.com/download) (como mínimo la versión 2.2). 
- Instale de la versión más reciente de la [CLI de Azure](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest).
- Instale la extensión `botservice` más reciente para la herramienta `az`. 
  - En primer lugar, quite la versión anterior, para lo que debe usar el comando `az extension remove -n botservice`. Después, use el comando `az extension add -n botservice` para instalar la versión más reciente.
- Instale la versión más reciente de la herramienta [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot).
  - Si la operación de clonación incluye recursos de LUIS o de Dispatch, necesitará la [CLI de LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#installation).
  - Si la operación de clonación incluye recursos de QnA Maker, necesitará la [CLI de QnA Maker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#as-a-cli).
- Instale [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started).
- Instale y configure [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29).
- Conocimientos del archivo [.bot](v4sdk/bot-file-basics.md).

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>Implementación de los bots de JavaScript y C# mediante az cli
Ya ha creado un bot y ahora quiere implementarlo en Azure. En estos pasos se supone que ha creado los recursos de Azure necesarios y ha actualizado las referencias del servicio en el archivo .bot mediante el comando `msbot connect` o mediante Bot Framework Emulator. Si el archivo .bot no está actualizado, es posible que el proceso de implementación se complete sin errores o advertencias, pero el bot implementado no funcionará.

Abra un símbolo del sistema para iniciar sesión en Azure Portal.

```cmd
az login
```
Se abrirá una ventana del explorador, lo que le permite iniciar sesión. 

### <a name="set-the-subscription"></a>Establecimiento de la suscripción 
Para establecer la suscripción utilice el siguiente comando:

```cmd
az account set --subscription "<azure-subscription>"
``` 

Si no está seguro de la suscripción que debe usar para implementar el bot y desea ver la lista de `subscriptions` para su cuenta, use el comando `az account list`.

Vaya a la carpeta del bot. 
```cmd 
cd <local-bot-folder>
```

### <a name="azure-subscription-account"></a>Cuenta de suscripción de Azure
Antes de continuar, lea las instrucciones que se le aplican en función del tipo de cuenta de correo electrónico que usa para iniciar sesión en Azure.

**Cuenta de correo electrónico MSA**

Si usa una cuenta de correo electrónico [MSA](https://en.wikipedia.org/wiki/Microsoft_account), deberá crear el appId y appSecret que se van a usar con el comando `msbot clone services`. 

- Vaya a [Portal de registro de aplicaciones](https://apps.dev.microsoft.com/). Haga clic en **Agregar una aplicación** para registrar la aplicación, cree el **Id. de la aplicación** y **genere una nueva contraseña**. 
- Guarde tanto el identificador de la aplicación como la contraseña que acaba de generar, con el fin de que pueda usarlos con el comando `msbot clone services`. 
- Para realizar la implementación, use el comando aplicable a su bot.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --proj-file "<your.csproj>" --name "<bot-name>" --appid "xxxxxxxx" --password "xxxxxxx" --verbose`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>"   --code-dir . --name "<bot-name>" --appid "xxxxxxxx" --password "xxxxxxx" --verbose`


[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

**Cuenta empresarial o educativa**

Si usa una cuenta de correo electrónico que le haya proporcionado su empresa o centro educativo para iniciar sesión en Azure, no es preciso que cree el identificador de la aplicación y la contraseña. Para realizar la implementación, use el comando aplicable a su bot.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --proj-file "<your-project-file>" --name "<bot-name>"`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --code-dir . --name "<bot-name>"`


[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

#### <a name="save-the-secret-used-to-decrypt-bot-file"></a>Almacenamiento del secreto usado para descifrar el archivo .bot
Es importante tener en cuenta que el proceso de implementación crea un _archivo .bot nuevo y lo cifra mediante un secreto_. Mientras se implementa el bot, verá en la línea de comandos el mensaje siguiente, en el que se le pide que guarde el secreto del archivo .bot. 

`The secret used to decrypt myAzBot.bot is:`
`hT6U1SIPQeXlebNgmhHYxcdseXWBZlmIc815PpK0WWA=`

`NOTE: This secret is not recoverable and you should save it in a safe place according to best security practices.
      Copy this secret and use it to open the <file.bot> the first time.`
      
Guarde el secreto del archivo .bot para su uso posterior. El nuevo archivo .bot cifrado se usa en Azure Portal con botFileSecret. Si necesita cambiar el nombre o el secreto del archivo del bot más adelante, vaya a la sección **Configuración de App Service -> Configuración de la aplicación** del portal. Tenga en cuenta que en el archivo appsettings.json o .env, el nombre de archivo del bot se actualiza con el archivo más reciente de bot que se haya creado.  

### <a name="test-your-bot"></a>Prueba del bot
En el emulador, utilice el punto de conexión de producción para probar la aplicación. Si desea probarla de forma local, asegúrese de que su bot se ejecuta en su equipo local. 

### <a name="to-update-your-bot-code-in-azure"></a>Para actualizar el código del bot en Azure
NO use el comando `msbot clone services` para actualizar el código del bot en Azure. Debe usar el comando `az bot publish`, tal como se muestra a continuación:

```cmd
az bot publish --name "<your-azure-bot-name>" --proj-file "<your-proj-file>" --resource-group "<azure-resource-group>" --code-dir "<folder>" --verbose --version v4
```

| Argumentos        | Descripción |
|----------------  |-------------|
| `name`      | El nombre que utilizó la primera vez que implementó el bot en Azure.|
| `proj-file` | En el caso del bot de C#, es el archivo .csproj. En el caso del bot de JS/TS, es el nombre de archivo del proyecto de startup (por ejemplo, index.js o index.ts) de su bot local.|
| `resource-group` | El grupo de recursos de Azure que ha usado el comando `msbot clone services`.|
| `code-dir`  | Apunta a la carpeta del bot local.|



## <a name="additional-resources"></a>Recursos adicionales

Cuando se implementa un bot, estos recursos normalmente se crean en Azure Portal:

| Recursos      | Descripción |
|----------------|-------------|
| Bot de aplicación web | Bot de Azure Bot Service que se implementa en Azure App Service.|
| [App Service](https://docs.microsoft.com/en-us/azure/app-service/)| Permite crear y hospedar aplicaciones web.|
| [Plan de App Service](https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)| Define un conjunto de recursos de proceso para que una aplicación web se ejecute.|
| [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview)| Proporciona las herramientas necesarias para recopilar y analizar datos de telemetría.|
| [Cuenta de almacenamiento](https://docs.microsoft.com/en-us/azure/storage/common/storage-introduction)| Proporciona almacenamiento en la nube altamente disponible, seguro, duradero, escalable y redundante.|

Para ver la documentación de los comandos `az bot`, consulte el tema de esta [referencia](https://docs.microsoft.com/en-us/cli/azure/bot?view=azure-cli-latest).

Si no está familiarizado con el grupo de recursos de Azure, consulte este tema de [terminología](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#terminology).

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Configuración de la implementación continua](bot-service-build-continuous-deployment.md)

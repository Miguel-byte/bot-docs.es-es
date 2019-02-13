---
title: Implementación de bots del repositorio botbuilder-samples | Microsoft Docs
description: Implemente su bot en la nube de Azure.
keywords: implementar bot, implementación de azure, publicar bot, bot de implementación de az, bot de implementación de visual studio, publicación de msbot, clonado de msbot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3ca8ac4bfe14ed20f11a0ab26d8102ac21e60e2b
ms.sourcegitcommit: 32615b88e4758004c8c99e9d564658a700c7d61f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/04/2019
ms.locfileid: "55711959"
---
# <a name="deploy-bots-from-botbuilder-samples-repo"></a>Implementación de bots del repositorio botbuilder-samples

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

En este artículo, le mostraremos cómo implementar los ejemplos de C# y JavaScript que están en el repositorio [botbuilder-samples](https://github.com/Microsoft/BotBuilder-Samples) de Azure.

Las instrucciones para implementar los bots de ejemplo son _diferentes_ de las instrucciones para [implementar un bot que podría crear con todos los recursos ya aprovisionados de Azure](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-deploy-az-cli?view=azure-bot-service-4.0&tabs=csharp).

> [!IMPORTANT]
> Si implementa un bot desde el repositorio [botbuilder-samples](https://github.com/Microsoft/BotBuilder-Samples) de Azure, aprovisionará los servicios de Azure y ello conllevará el pago de los servicios que use.
> El artículo acerca de la [facturación y administración de costos](https://docs.microsoft.com/en-us/azure/billing/) sirve de ayuda para conocer la facturación de Azure, para supervisar el uso y los costos, y para administrar cuentas y suscripciones.

Es conveniente leerlo antes de seguir los pasos, con el fin de tener un conocimiento completo de lo que implica la implementación de un bot.

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
- Instale el [SDK de .NET Core](https://dotnet.microsoft.com/download) (como mínimo la versión 2.2). (Utilice `dotnet --version` para ver qué versión tiene).
- Instale de la versión más reciente de la [CLI de Azure](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest). (Utilice `az --version` para ver qué versión tiene).
- Instale la versión más reciente de la herramienta [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot).
  - Si la operación de clonación incluye recursos de LUIS o de Dispatch, necesitará la [CLI de LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#installation).
  - Si la operación de clonación incluye recursos de QnA Maker, necesitará la [CLI de QnA Maker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#as-a-cli).
- Instale [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started).
- Instale y configure [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29).
- Conocimientos de archivos [.bot](v4sdk/bot-file-basics.md).

Con msbot 4.3.2 y posteriores, necesita la versión 2.0.54 de la CLI de Azure o posterior. Si ha instalado la extensión botservice, elimínela con este comando.

```cmd
az extension remove --name botservice
```

### <a name="c"></a>C\#

 `msbot clone services` no carga los archivos de código en Azure, solo la biblioteca de vínculos dinámicos y algunos otros archivos. Debe compilar el ejemplo antes de ejecutar este comando.

### <a name="service-names"></a>Nombres de servicio

Además de implementar el bot, el comando `msbot clone services` aprovisiona todos los servicios asociados a la suscripción elegida.

Si cualquiera de esas combinaciones de nombre-servicio ya existen en la suscripción, el comando generará un error y deberá eliminar la implementación parcial antes de volver a iniciar. Esto incluye aplicaciones de LUIS, bases de conocimiento de QnA Maker y modelos de distribución.

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>Implementación de los bots de JavaScript y C# mediante az cli

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

### <a name="clone-the-sample"></a>Clonación del ejemplo

**Cuenta de suscripción de Azure**: Antes de continuar, lea las instrucciones que se le aplican en función del tipo de cuenta de correo electrónico que usa para iniciar sesión en Azure.

**Creación de servicios**: El comand `msbot clone services` creará los servicios de Azure para el bot.

1. Muestra los servicios que se crearán y le pide que confirme la operación antes de continuar. Si rechaza, el comando se cierra sin crear ninguno de los servicios.
1. Tiene que autenticarse con Azure antes de continuar.

**Servicios de LUIS**: Si su bot usa LUIS o Distribución, deberá incluir la clave de creación de LUIS en el comando `msbot clone services`.

#### <a name="msa-email-account"></a>Cuenta de correo electrónico MSA

Si usa una cuenta de correo electrónico [MSA](https://en.wikipedia.org/wiki/Microsoft_account), deberá crear el appId y appSecret que se van a usar con el comando `msbot clone services`.

- Vaya a [Portal de registro de aplicaciones](https://apps.dev.microsoft.com/). Haga clic en **Agregar una aplicación** para registrar la aplicación, cree el **Id. de la aplicación** y **genere una nueva contraseña**.
> NOTA: Si la contraseña generada contiene el carácter "|", Azure rechazará la contraseña. Para resolver este problema, genere otra contraseña nueva.
- Guarde tanto el identificador de la aplicación como la contraseña que acaba de generar, con el fin de que pueda usarlos con el comando `msbot clone services`.
- Para realizar la implementación, use el comando aplicable a su bot.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --proj-file "<your.csproj>" --name "<bot-name>" --appId "xxxxxxxx" --appSecret "xxxxxxx" --verbose --luisAuthoringKey <luis-authoring-key>`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>"   --code-dir . --name "<bot-name>" --appId "xxxxxxxx" --appSecret "xxxxxxx" --verbose --luisAuthoringKey <luis-authoring-key>`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

#### <a name="business-or-school-account"></a>Cuenta empresarial o educativa

Si usa una cuenta de correo electrónico que le haya proporcionado su empresa o centro educativo para iniciar sesión en Azure, no es preciso que cree el identificador de la aplicación y la contraseña. Para realizar la implementación, use el comando aplicable a su bot.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --proj-file "<your-project-file>" --name "<bot-name>" --luisAuthoringKey <luis-authoring-key>`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --code-dir . --name "<bot-name>" --luisAuthoringKey <luis-authoring-key>`

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
az bot publish --name "<your-azure-bot-name>" --proj-name "<your-proj-name>" --resource-group "<azure-resource-group>" --code-dir "<folder>" --verbose --version v4
```

| Argumentos        | DESCRIPCIÓN |
|----------------  |-------------|
| `name`      | El nombre que utilizó la primera vez que implementó el bot en Azure.|
| `proj-name` | Para C#, use el nombre de archivo del proyecto de inicio (sin .csproj) que se tiene que publicar. Por ejemplo: `EnterpriseBot`. Para Node.js, use el punto de entrada principal del bot. Por ejemplo, `index.js`. |
| `resource-group` | El grupo de recursos de Azure que ha usado el comando `msbot clone services`.|
| `code-dir`  | Apunta a la carpeta del bot local.|

## <a name="additional-resources"></a>Recursos adicionales

Cuando se implementa un bot, estos recursos normalmente se crean en Azure Portal:

| Recursos      | DESCRIPCIÓN |
|----------------|-------------|
| Bot de aplicación web | Bot de Azure Bot Service que se implementa en Azure App Service.|
| [App Service](https://docs.microsoft.com/en-us/azure/app-service/)| Permite crear y hospedar aplicaciones web.|
| [plan de App Service](https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)| Define un conjunto de recursos de proceso para que una aplicación web se ejecute.|
| [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview)| Proporciona las herramientas necesarias para recopilar y analizar datos de telemetría.|
| [Cuenta de almacenamiento](https://docs.microsoft.com/en-us/azure/storage/common/storage-introduction)| Proporciona almacenamiento en la nube altamente disponible, seguro, duradero, escalable y redundante.|

Para ver la documentación de los comandos `az bot`, consulte el tema de esta [referencia](https://docs.microsoft.com/en-us/cli/azure/bot?view=azure-cli-latest).

Si no está familiarizado con el grupo de recursos de Azure, consulte este tema de [terminología](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#terminology).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Configuración de la implementación continua](bot-service-build-continuous-deployment.md)

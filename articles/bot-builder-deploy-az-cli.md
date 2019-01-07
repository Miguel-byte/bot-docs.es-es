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
ms.date: 12/14/2018
ms.openlocfilehash: 19960940a40fa291534bc1f88290bc6a7da109e0
ms.sourcegitcommit: 8c10aa7372754596a3aa7303a3a893dd4939f7e9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/20/2018
ms.locfileid: "53654340"
---
# <a name="deploy-your-bot-using-azure-cli"></a>Implementación de un bot mediante la CLI de Azure

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

Después de que haya creado el bot y lo haya probado localmente, puede implementarlo en Azure para que sea posible acceder a él desde cualquier lugar. La implementación de un bot en Azure conllevará el pago de los servicios que se usen. El artículo acerca de la [facturación y administración de costos](https://docs.microsoft.com/en-us/azure/billing/) sirve de ayuda para conocer la facturación de Azure, para supervisar el uso y los costos, y para administrar cuentas y suscripciones.

En dicho artículo, podrá ver cómo implementar bots de C# y JavaScript en Azure mediante la cli de `az` y `msbot`. Es conveniente leerlo antes de seguir los pasos, con el fin de tener un conocimiento completo de lo que implica la implementación de un bot.


## <a name="prerequisites"></a>Requisitos previos
- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
- Instale de la versión más reciente de la [CLI de Azure](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest).
- Instale la extensión `botservice` más reciente para la herramienta `az`.
  - En primer lugar, quite la versión anterior, para lo que debe usar el comando `az extension remove -n botservice`. Después, use el comando `az extension add -n botservice` para instalar la versión más reciente.
- Instale la versión más reciente de la herramienta [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot).
- Instale la versión más reciente de [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started).
- Instale y configure [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29).
- Conocimientos del archivo [.bot](v4sdk/bot-file-basics.md).

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>Implementación de los bots de JavaScript y C# mediante az cli

Ya ha creado y probado localmente un bot y ahora quiere implementarlo en Azure. En estos pasos se supone que ha creado los recursos de Azure necesarios.

Abra un símbolo del sistema para iniciar sesión en Azure Portal.

```cmd
az login
```

Se abrirá una ventana del explorador, lo que le permite iniciar sesión.

### <a name="set-the-subscription"></a>Establecimiento de la suscripción

Establezca la suscripción predeterminada que desea usar.

```cmd
az account set --subscription "<azure-subscription>"
```

Si no está seguro de la suscripción que debe usar para implementar el bot y desea ver la lista de `subscriptions` para su cuenta, use el comando `az account list`.

Vaya a la carpeta del bot.

```cmd
cd <local-bot-folder>
```

### <a name="create-a-web-app-bot"></a>Creación de un bot de aplicación web

Cree el recurso de bot en el cual publicará el bot.

Antes de continuar, lea las instrucciones que se le aplican en función del tipo de cuenta de correo electrónico que usa para iniciar sesión en Azure.

#### <a name="msa-email-account"></a>Cuenta de correo electrónico MSA

Si va a usar una cuenta de correo electrónico [MSA](https://en.wikipedia.org/wiki/Microsoft_account), tendrá que crear el identificador de la aplicación y la contraseña en el portal de registro de aplicación que va a usar con el comando `az bot create`.

1. Vaya a [**Portal de registro de aplicaciones**](https://apps.dev.microsoft.com/).
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

#### <a name="business-or-school-account"></a>Cuenta empresarial o educativa

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```

| Opción | DESCRIPCIÓN |
|:---|:---|
| --name | Nombre único que se usa para implementar el bot en Azure. El nombre puede ser el mismo que el de su bot local. NO incluya espacios ni guiones bajos en el nombre. |
| --location | Ubicación geográfica utilizada para crear los recursos del servicio de bot. Por ejemplo, `eastus`, `westus`, `westus2`, etc. |
| --lang | El idioma que se usará para crear el bot: `Csharp`, o `Node`; el valor predeterminado es `Csharp`. |
| --resource-group | Nombre del grupo de recursos en el que se crea el bot. Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |

### <a name="download-the-bot-from-azure"></a>Descarga del bot de Azure

A continuación, descargue el bot que acaba de crear. Este comando creará un subdirectorio en la ruta de guardar. No obstante, la ruta especificada ya debe existir.

```cmd
az bot download --name <bot-resource-name> --resource-group <resource-group-name> --save-path "<path>"
```

| Opción | DESCRIPCIÓN |
|:---|:---|
| --name | El nombre del bot en Azure. |
| --resource-group | El nombre del grupo de recursos en el que está el bot. |
| --save-path | Un directorio existente en el que descargar el código del bot. |

### <a name="decrypt-the-downloaded-bot-file"></a>Descrifrado del archivo .bot descargado

La información confidencial del archivo .bot está cifrada.

Obtenga la clave de cifrado.

1. Inicie sesión en [Azure Portal](http://portal.azure.com/).
1. Abra el recurso de bot de aplicación web de su bot.
1. Abra la **configuración de la aplicación** del bot.
1. En la ventana **Configuración de la aplicación**, desplácese hacia abajo a **Configuración de la aplicación**.
1. Busque **botFileSecret** y copie su valor.

Descifre el archivo .bot.

```cmd
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

| Opción | DESCRIPCIÓN |
|:---|:---|
| --bot | La ruta de acceso relativa al archivo .bot descargado. |
| --secret | La clave de cifrado. |

### <a name="use-the-downloaded-bot-file-in-your-project"></a>Uso del archivo .bot descargado en el proyecto

Copie el archivo .bot descifrado en el directorio que contiene el proyecto de bot local.

Actualice su bot para usar este nuevo archivo .bot.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En **appsettings.json**, actualice la propiedad **botFilePath** para que apunte al nuevo archivo .bot.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En **.env**, actualice la propiedad **botFilePath** para que apunte al nuevo archivo .bot.

---

### <a name="update-the-bot-file"></a>Actualización del archivo .bot

Si el bot usa LUIS, QnA Maker o servicios de distribución, deberá agregar referencias a ellos en el archivo .bot. En caso contrario, puede omitir este paso.

1. Abra el bot en Bot Framework Emulator con el nuevo archivo .bot. No es necesario que el bot se esté ejecutando localmente.
1. En el panel **BOT EXPLORER** (Explorador de bots), expanda la sección **SERVICES** (Servicios).
1. Para agregar referencias a las aplicaciones de LUIS, haga clic en el signo más (+) situado a la derecha de **SERVICES** (Servicios).
   1. Seleccione **Add Language Understanding (LUIS)** (Agregar Language Understanding (LUIS).
   1. Si se le solicita que inicie sesión en la cuenta de Azure, hágalo.
   1. Presenta una lista de las aplicaciones de LUIS a las que tiene acceso. Seleccione las del bot.
1. Para agregar referencias a una base de conocimiento de QnA Maker, haga clic en el signo más (+) situado a la derecha de **SERVICES** (Servicios).
   1. Seleccione **Add QnA Maker** (Agregar QnA Maker).
   1. Si se le solicita que inicie sesión en la cuenta de Azure, hágalo.
   1. Presenta una lista de las bases de conocimiento a las que tiene acceso. Seleccione las del bot.
1. Para agregar referencias a los modelos de distribución, haga clic en el signo más (+) situado a la derecha de **SERVICES** (Servicios).
   1. Seleccione **Add Dispatch** (Agregar distribución).
   1. Si se le solicita que inicie sesión en la cuenta de Azure, hágalo.
   1. Presenta una lista de modelos de distribución a los que tiene acceso. Seleccione los del bot.

### <a name="test-your-bot-locally"></a>Prueba local del bot

En este punto, el bot debería funcionar del mismo modo que lo hacía con el archivo .bot antiguo. Asegúrese de que funciona según lo esperado con el nuevo archivo .bot.

### <a name="publish-your-bot-to-azure"></a>Publicación del bot en Azure

<!-- TODO: re-encrypt your .bot file? -->

Publicación del bot local en Azure. Este paso puede tardar unos minutos.

```cmd
az bot publish --name <bot-resource-name> --proj-file "<project-file-name>" --resource-group <resource-group-name> --code-dir <directory-path> --verbose --version v4
```

<!-- Question: What should --proj-file be for a Node project? -->

| Opción | DESCRIPCIÓN |
|:---|:---|
| --name | El nombre de recurso del bot en Azure. |
| --proj-file | El nombre de archivo del proyecto (sin .csproj) que se tiene que publicar. Por ejemplo:  EnterpriseBot. |
| --resource-group | Nombre del grupo de recursos. |
| --code-dir | El directorio desde el que cargar el código del bot. |

Una vez que esto se complete con un mensaje de "implementación correcta", el bot estará implementado en Azure.

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

Borre la configuración de la clave de cifrado.

1. Inicie sesión en [Azure Portal](http://portal.azure.com/).
1. Abra el recurso de bot de aplicación web de su bot.
1. Abra la **configuración de la aplicación** del bot.
1. En la ventana **Configuración de la aplicación**, desplácese hacia abajo a **Configuración de la aplicación**.
1. Busque **botFileSecret** y elimínelo.

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

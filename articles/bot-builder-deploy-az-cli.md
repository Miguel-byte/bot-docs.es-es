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
ms.date: 02/07/2019
ms.openlocfilehash: b4c3b982bf061b3a24c6d240b05dc40b0cf07816
ms.sourcegitcommit: 8183bcb34cecbc17b356eadc425e9d3212547e27
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/09/2019
ms.locfileid: "55971434"
---
# <a name="deploy-your-bot"></a>Implementación del bot 

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

Después de que haya creado el bot y lo haya probado localmente, puede implementarlo en Azure para que sea posible acceder a él desde cualquier lugar. La implementación de un bot en Azure conllevará el pago de los servicios que se usen. El artículo acerca de la [facturación y administración de costos](https://docs.microsoft.com/en-us/azure/billing/) sirve de ayuda para conocer la facturación de Azure, para supervisar el uso y los costos, y para administrar cuentas y suscripciones.

En dicho artículo, mostraremos cómo implementar bots de C# y JavaScript en Azure. Es conveniente leerlo antes de seguir los pasos, con el fin de tener un conocimiento completo de lo que implica la implementación de un bot.

## <a name="prerequisites"></a>Requisitos previos
- Instale la versión más reciente de la herramienta [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot).
- Un bot de [CSharp](./dotnet/bot-builder-dotnet-sdk-quickstart.md) o [JavaScript](./javascript/bot-builder-javascript-quickstart.md) que haya desarrollado en el equipo local. 

## <a name="create-a-web-app-bot-in-azure"></a>Creación de un bot de aplicación web en Azure
Esta sección es opcional si ya ha creado un bot de Azure que quiere utilizar.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Haga clic en el vínculo **Crear un nuevo recurso** que se encuentra en la esquina superior izquierda de Azure Portal y, a continuación, seleccione **IA + Machine Learning > Web App Bot** (Bot de aplicación web).
1. Se abrirá una nueva hoja con información sobre el bot de aplicación web. 
1. En la hoja **Servicio de bots**, proporcione la información necesaria sobre el bot.
1. Haga clic en **Crear** para crear el servicio e implementar el bot en la nube. Este proceso puede tardar varios minutos.

## <a name="download-the-source-code"></a>Descarga del código fuente
1. En la sección **Bot Management** (Administración de bots), haga clic en **Build** (Compilar).
1. Haga clic en el vínculo **Download Bot source code** (Descargar código fuente del bot) del panel derecho.
1. Siga las indicaciones para descargar el código y, después, descomprima la carpeta.

## <a name="decrypt-the-bot-file"></a>Descifre el archivo .bot.
El código fuente que descargó desde Azure Portal incluye un archivo .bot cifrado. Deberá descifrarlo para copiar los valores en el archivo .bot local.  

1. En Azure Portal, abra el recurso Bot de aplicación web de su bot.
1. Abra la **configuración de la aplicación** del bot.
1. En la ventana **Configuración de la aplicación**, vaya a **Configuración de la aplicación**.
1. Busque **botFileSecret** y copie su valor.

Use `msbot cli` para descifrar el archivo.
```
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

## <a name="update-the-bot-file"></a>Actualización del archivo .bot
Abra el archivo .bot que ha descifrado. Copie las entradas incluidas en la sección `services` y agruéguelas al archivo .bot local. Por ejemplo: 

```
{
   "type": "endpoint",
   "name": "production",
   "endpoint": "https://<something>.azurewebsites.net/api/messages",
   "appId": "<App Id>",
   "appPassword": "<App Password>",
   "id": "2
}
```

Guarde el archivo.
 
## <a name="setup-a-repository"></a>Configuración de un repositorio
Cree un repositorio git con el proveedor de control de código fuente de git que prefiera. Confirme el código en el repositorio.
 
## <a name="update-app-settings-in-azure"></a>Actualización de la configuración de la aplicación en Azure
1. En Azure Portal, abra el recurso **Bot de aplicación web** de su bot.
1. Abra la **configuración de la aplicación** del bot.
1. En la ventana **Configuración de la aplicación**, vaya a **Configuración de la aplicación**.
1. Busque **botFileSecret** y elimínelo.
1. Actualice el nombre del archivo bot para que coincida con el archivo que ha insertado en el repositorio.
1. Guarde los cambios.


## <a name="deploy-using-azure-deployment-center"></a>Implementación con el Centro de implementación de Azure
Ahora, deberá conectar su repositorio git con Azure App Services. Siga las instrucciones del tema [Configuración de una implementación continua](https://docs.microsoft.com/en-us/azure/app-service/deploy-continuous-deployment). Tenga en cuenta que se recomienda compilar con `App Service Kudu build server`.

## <a name="test-your-deployment"></a>Prueba de la implementación
Espere unos segundos después de una implementación correcta y, opcionalmente, reinicie la aplicación web para borrar toda la memoria caché. Vuelva a la hoja Bot de aplicación web y pruebe con el Chat en web proporcionado en Azure Portal.

## <a name="additional-resources"></a>Recursos adicionales
- [How to investigate common issues with continuous deployment](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment)

<!--

## Prerequisites

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]


## Deploy JavaScript and C# bots using az cli

You've already created and tested a bot locally, and now you want to deploy it to Azure. These steps assume that you have created the required Azure resources.

[!INCLUDE [az login snippet](~/includes/deploy/snippet-az-login.md)]

### Create a Web App Bot

If you don't already have a resource group to which to publish your bot, create one:

[!INCLUDE [az create group snippet](~/includes/deploy/snippet-az-create-group.md)]

[!INCLUDE [az create web app snippet](~/includes/deploy/snippet-create-web-app.md)]

Before proceeding, read the instructions that apply to you based on the type of email account you use to log in to Azure.

#### MSA email account

If you are using an [MSA](https://en.wikipedia.org/wiki/Microsoft_account) email account, you will need to create the app ID and app password on the Application Registration Portal to use with `az bot create` command.

[!INCLUDE [create bot msa snippet](~/includes/deploy/snippet-create-bot-msa.md)]

#### Business or school account

[!INCLUDE [create bot snippet](~/includes/deploy/snippet-create-bot.md)]

### Download the bot from Azure

Next, download the bot you just created. 
[!INCLUDE [download bot snippet](~/includes/deploy/snippet-download-bot.md)]

### Decrypt the downloaded .bot file and use in your project

The sensitive information in the .bot file is encrypted.

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### Update the .bot file

If your bot uses LUIS, QnA Maker, or Dispatch services, you will need to add references to them to your .bot file. Otherwise, you can skip this step.

1. Open your bot in the BotFramework Emulator, using the new .bot file. The bot does not need to be running locally.
1. In the **BOT EXPLORER** panel, expand the **SERVICES** section.
1. To add references to LUIS apps, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add Language Understanding (LUIS)**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of LUIS applications you have access to. Select the ones for your bot.
1. To add references to a QnA Maker knowledge base, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add QnA Maker**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of knowledge bases you have access to. Select the ones for your bot.
1. To add references to Dispatch models, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add Dispatch**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of Dispatch models you have access to. Select the ones for your bot.

### Test your bot locally

At this point, your bot should work the same way it did with the old .bot file. Make sure that it works as expected with the new .bot file.

### Publish your bot to Azure

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]


[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

## Additional resources

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## Next steps
> [!div class="nextstepaction"]
> [Set up continous deployment](bot-service-build-continuous-deployment.md)

-->

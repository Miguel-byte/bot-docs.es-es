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
ms.date: 04/12/2019
ms.openlocfilehash: 4532fe55705524573de55017e633289255a20ab9
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2019
ms.locfileid: "59508222"
---
# <a name="deploy-your-bot"></a>Implementación del bot

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

Después de que haya creado el bot y lo haya probado localmente, puede implementarlo en Azure para que sea posible acceder a él desde cualquier lugar. La implementación de un bot en Azure conllevará el pago de los servicios que se usen. El artículo acerca de la [facturación y administración de costos](https://docs.microsoft.com/en-us/azure/billing/) sirve de ayuda para conocer la facturación de Azure, para supervisar el uso y los costos, y para administrar cuentas y suscripciones.

En dicho artículo, mostraremos cómo implementar bots de C# y JavaScript en Azure. Es conveniente leerlo antes de seguir los pasos, con el fin de tener un conocimiento completo de lo que implica la implementación de un bot.

## <a name="prerequisites"></a>Requisitos previos
- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](http://portal.azure.com) antes de empezar.
- Un bot de [**CSharp**](./dotnet/bot-builder-dotnet-sdk-quickstart.md) o [**JavaScript**](./javascript/bot-builder-javascript-quickstart.md) que haya desarrollado en la máquina local.

## <a name="1-prepare-for-deployment"></a>1. Preparación de la implementación
El proceso de implementación requiere un bot de aplicación web de destino en Azure para que el bot local se pueda implementar en él. El bot local usa el bot de aplicación web de destino y los recursos que se aprovisionan con él en Azure para la implementación. Esto es necesario porque el bot local no tiene aprovisionados todos los recursos de Azure necesarios. Al crear un bot de aplicación web de destino, se aprovisionan automáticamente los siguientes recursos:
-   Bot de aplicación web: usará este bot para implementar en él el bot local.
-   Plan de App Service: proporciona los recursos que debe ejecutar una aplicación de App Service.
-   App Service: servicio donde se hospedan las aplicaciones web.
-   Cuenta de Azure Storage: contiene todos los objetos de datos de Azure Storage (blobs, archivos, colas, tablas y discos).

Durante la creación del bot de aplicación web de destino, también se generan un identificador de aplicación y una contraseña para el bot. En Azure, el identificador de aplicación y la contraseña admiten la [autenticación y autorización del servicio](https://docs.microsoft.com/azure/app-service/overview-authentication-authorization). Recuperará parte de esta información para usarla en el código del bot local. 

> [!IMPORTANT]
> El lenguaje de programación de la plantilla de bot utilizada en Azure Portal debe coincidir con el lenguaje de programación en el que está escrito el bot.

La creación de un bot de aplicación web nuevo es opcional si ya ha creado un bot de Azure que quiere utilizar.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Haga clic en el vínculo **Crear un nuevo recurso** que se encuentra en la esquina superior izquierda de Azure Portal y, a continuación, seleccione **IA + Machine Learning > Web App Bot** (Bot de aplicación web).
1. Se abrirá una nueva hoja con información sobre el bot de aplicación web. 
1. En la hoja **Servicio de bots**, proporcione la información necesaria sobre el bot.
1. Haga clic en **Crear** para crear el servicio e implementar el bot en la nube. Este proceso puede tardar varios minutos.

### <a name="download-the-source-code"></a>Descarga del código fuente
Después de crear el bot de aplicación web de destino, deberá descargar el código del bot desde Azure Portal en el equipo local. El motivo para descargar código es obtener las referencias del servicio (por ejemplo, MicrosoftAppID, MicrosoftAppPassword, LUIS o QnA) que están en el archivo appsettings.json o .env. 

1. En la sección **Bot Management** (Administración de bots), haga clic en **Build** (Compilar).
1. Haga clic en el vínculo **Download Bot source code** (Descargar código fuente del bot) del panel derecho.
1. Siga las indicaciones para descargar el código y, después, descomprima la carpeta.
    1. [!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

### <a name="update-your-local-appsettingsjson-or-env-file"></a>Actualización del archivo appsettings.json o .env local

Abra el archivo appsettings.json o .env que descargó. Copie **todas** las entradas de la lista y agréguelas al archivo appsettings.json o .env _local_. Resuelva todas las entradas de servicio duplicadas o los identificadores de servicio duplicados. Mantenga las referencias de servicio adicionales de las que dependa el bot.

Guarde el archivo.

### <a name="update-local-bot-code"></a>Actualización del código de bot local
Actualice el archivo Startup.cs o index.js local para que use el archivo appsettings.json o .env en lugar de utilizar el archivo .bot. El archivo .bot está en desuso y estamos trabajando para actualizar las plantillas VSIX, los generadores de Yeoman, y el resto de ejemplos y documentos para que todos usen el archivo appsettings.json o .env en lugar del archivo .bot. Mientras tanto, deberá realizar cambios en el código del bot. 

Actualice el código para que lea la configuración del archivo appsettings.json o .env. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
En el método `ConfigureServices`, use el objeto de configuración que proporciona ASP.NET Core, por ejemplo: 

**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```

# <a name="jstabjs"></a>[JS](#tab/js)

En JavaScript, haga referencia a las variables .env fuera del objeto `process.env`, por ejemplo:
   
**index.js**

```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```
---

- Guarde el archivo y pruebe el bot.

### <a name="setup-a-repository"></a>Configuración de un repositorio

Para permitir la implementación continua, cree un repositorio git con el proveedor de control de código fuente de git que prefiera. Confirme el código en el repositorio.

Asegúrese de que la raíz del repositorio tenga los archivos correctos, tal y como se explica en [Preparación del repositorio](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment#prepare-your-repository).

### <a name="update-app-settings-in-azure"></a>Actualización de la configuración de la aplicación en Azure
El bot local no utiliza un archivo .bot cifrado, pero Azure Portal _sí_ está configurado para usarlo. Para resolver este problema, puede quitar el valor de **botFileSecret** almacenado en la configuración del bot de Azure.
1. En Azure Portal, abra el recurso **Bot de aplicación web** de su bot.
1. Abra la **configuración de la aplicación** del bot.
1. En la ventana **Configuración de la aplicación**, vaya a **Configuración de la aplicación**.
1. Compruebe si su bot incluye las entradas **botFileSecret** y **botFilePath**. Si es así, elimínelo.
1. Guarde los cambios.

## <a name="2-deploy-using-azure-deployment-center"></a>2. Implementación con el Centro de implementación de Azure

Ahora, necesita cargar el código del bot a Azure. Siga las instrucciones del tema [Implementación continua en Azure App Service](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment).

Tenga en cuenta que se recomienda compilar con `App Service Kudu build server`.

Una vez que haya configurado la implementación continua, se publican los cambios que se confirmen en el repositorio. Sin embargo, si agrega servicios al bot, deberá agregar entradas para ellos al archivo .bot.

## <a name="3-test-your-deployment"></a>3. Prueba de la implementación

Espere unos segundos después de una implementación correcta; también puede reiniciar la aplicación web para borrar toda la memoria caché. Vuelva a la hoja Bot de aplicación web y pruebe con el Chat en web proporcionado en Azure Portal.

## <a name="additional-resources"></a>Recursos adicionales
- [How to investigate common issues with continuous deployment](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment)


---
title: Bot de .NET con la extensión de App Service para Direct Line
titleSuffix: Bot Service
description: Habilitar un bot de .NET para trabajar con la extensión de App Service para Direct Line
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 07/25/2019
ms.openlocfilehash: 3ed589bff5c3740dddcfb62226714006313ae330
ms.sourcegitcommit: 312a4593177840433dfee405335100ce59aac347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/12/2019
ms.locfileid: "73933677"
---
# <a name="configure-net-bot-for-extension"></a>Configuración de un bot de .NET para la extensión

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

En este artículo se describe cómo actualizar un bot para trabajar con **canalizaciones con nombre** y cómo habilitar la extensión de App Service para Direct Line en el recurso de **Azure App Service** en el que se hospeda el bot.  

## <a name="prerequisites"></a>Requisitos previos

Para realizar los pasos que se describen a continuación, debe tener un recurso de **Azure App Service** y **App Service** relacionados en Azure.

## <a name="enable-direct-line-app-service-extension"></a>Habilitación de la extensión de App Service para Direct Line

En esta sección se describe cómo habilitar la extensión de App Service para Direct Line mediante las claves de la configuración de canales del bot y el recurso de **Azure App Service** en el que se hospeda el bot.

## <a name="update-net-bot-to-use-direct-line-app-service-extension"></a>Actualización de un bot de .NET para utilizar la extensión de App Service para Direct Line

1. Abra el proyecto del bot en Visual Studio.
1. Agregue el paquete **Extensión de streaming de NuGet** al proyecto:
    1. En el proyecto, haga clic con el botón derecho en **Dependencias** y seleccione **Administrar paquetes de NuGet**.
    1. En la pestaña *Examinar*, haga clic en **Incluir versión preliminar** para mostrar los paquetes en versión preliminar.
    1. Seleccione el paquete **Microsoft.Bot.Builder.StreamingExtensions**.
    1. Haga clic en el botón **Instalar** para instalar el paquete y lea y acepte el contrato de licencia.
1. Permita que la aplicación use la **canalización con nombre de Bot Framework**:
    - Abra el archivo `Startup.cs` .
    - En el método ``Configure``, agregue código a ``UseBotFrameworkNamedPipe``.

    ```csharp

    using Microsoft.Bot.Builder.StreamingExtensions;

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }
        else
        {
            app.UseHsts();
        }

        app.UseDefaultFiles();
        app.UseStaticFiles();

        // Allow bot to use named pipes.
        app.UseBotFrameworkNamedPipe();

        app.UseMvc();
    }
    ```

1. Guarde el archivo `Startup.cs`.
1. Abra el archivo `appsettings.json` y escriba los valores siguientes:
    1. `"MicrosoftAppId": "<secret Id>"`
    1. `"MicrosoftAppPassword": "<secret password>"`

    Los valores son los elementos **appid** y **appSecret** asociados con el grupo de registro del servicio.

1. **Publique** el bot en la instancia de Azure App Service.
1. En el explorador, vaya a https://<instancia_de_app_service>.azurewebsites.net/.bot. Si todo está correcto, la página devolverá este contenido JSON: `{"k":true,"ib":true,"ob":true,"initialized":true}`. Esta es la información que se obtiene cuando **todo funciona correctamente**; en ella:

    - **k** determina si la extensión Direct Line de App Service (ASE) puede leer una clave de extensión de su configuración. 
    - **initialized** determina si la extensión Direct Line de App Service puede usar la clave de extensión para descargar los metadatos de bot de Azure Bot Service.
    - **ib** determina si la extensión Direct Line de App Service puede establecer una conexión entrante con el bot.
    - **ob** determina si la extensión Direct Line de App Service puede establecer una conexión saliente con el bot. 


### <a name="gather-your-direct-line-extension-keys"></a>Recopilación de las claves de la extensión Direct Line

1. En el explorador, vaya a [Azure Portal](https://portal.azure.com/).
1. En Azure Portal, busque el recurso de **Azure Bot Service**.
1. Haga clic en **Canales** para configurar los canales del bot.
1. Si aún no está habilitado, haga clic en el canal de **Direct Line** para habilitarlo. 
1. Si ya está habilitado, en la tabla Conectar a canales, haga clic en el vínculo **Editar** en la fila de Direct Line.
1. Desplácese hacia abajo hasta la sección Claves de la extensión de App Service. 
1. Haga clic en el vínculo **Mostrar** para mostrar una de las claves y, a continuación, copie su valor.

![Claves de la extensión de App Service](./media/channels/direct-line-extension-extension-keys.png)

### <a name="enable-the-direct-line-app-service-extension"></a>Habilitación de la extensión de App Service para Direct Line

1. En el explorador, vaya a [Azure Portal](https://portal.azure.com/).
1. En Azure Portal, busque la página del recurso de **Azure App Service** de la aplicación web donde se hospedará el bot.
1. Haga clic en **Configuración**. En la sección *Configuración de la aplicación*, agregue la siguiente nueva configuración:

    |NOMBRE|Valor|
    |---|---|
    |DirectLineExtensionKey|<Clave_de_la_extensión_de_App_Service_de_la_seccion_1>|
    |DIRECTLINE_EXTENSION_VERSION|latest|

1. En la sección *Configuración*, haga clic en la sección **Configuración general** y active **Sockets web**.
1. Haga clic en **Guardar** para guardar la configuración. Esto reinicia la instancia de Azure App Service.

---
title: Publicación de una instancia de Bot Service de control de código fuente o Visual Studio | Microsoft Docs
description: Aprenda a publicar un bot de Bot Service una vez desde Visual Studio o de forma continua desde el control de código fuente.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: 38b26ed5a50409de64518562faabf532f45c857e
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999152"
---
# <a name="publish-a-bot-to-bot-service"></a>Publicación de un bot de Bot Service

Después de actualizar el código fuente del bot de C#, puede publicarlo en un bot en ejecución de Bot Service mediante Visual Studio. También puede publicar un código fuente de bot de C# o Node.js que se haya escrito en cualquier entorno de desarrollo integrado (IDE) de forma automática cada vez que compruebe un archivo de origen en un servicio de control de código fuente.


## <a name="publish-a-bot-on-app-service-plan-from-the-online-code-editor"></a>Publicación de un bot en el plan de App Service desde el editor de código en línea

Si no ha configurado la implementación continua, puede modificar los archivos de origen en el editor de código en línea. Para implementar el código fuente modificado, siga estos pasos.

4. Haga clic en el icono Abrir consola.  
    ![Icono de la consola](~/media/azure-bot-service-console-icon.png)
2. En la ventana de consola, escriba **build.cmd** y presione la tecla Entrar.


## <a name="publish-c-bot-on-app-service-plan-from-visual-studio"></a>Publicación del bot de C# en el plan de App Service desde Visual Studio 

Para configurar la publicación desde Visual Studio usando el archivo `.PublishSettings`, siga los pasos a continuación:

1. En Azure Portal, haga clic en el Bot Service, y luego en la pestaña **COMPILACIÓN** y haga clic en **Descargar archivo zip**.
3. Extraiga en una carpeta local el contenido del archivo zip descargado.
4. En el Explorador busque el archivo de solución de Visual Studio (.sln) para el bot, y haga doble clic en él.
4. En Visual Studio, haga clic en **Ver** y después en **Explorador de soluciones**.
5. En el panel del Explorador de soluciones haga clic con el botón derecho en el proyecto y luego haga clic en **Publicar...** Se abre la ventana Publicar. 
6. En la ventana Publicar haga clic en **Crear nuevo perfil**, haga clic en **Importar perfil** y después en **Aceptar**.
7. Vaya a la carpeta del proyecto y, a continuación, a la carpeta **PostDeployScripts**, seleccione el archivo que termine en `.PublishSettings` y haga clic en **Abrir**.

Ya ha configurado la publicación para este proyecto. Para publicar el código fuente local en el Bot Service, haga clic con el botón derecho en el proyecto, después en **Publicar...**  y haga clic en el botón **Publicar**. 

## <a name="set-up-continuous-deployment"></a>Configurar la implementación continua

De forma predeterminada, Bot Service le permite desarrollar su bot directamente en el explorador con el editor de Azure, sin necesidad de un control de código fuente o un editor local. De todas formas, el editor de Azure no le permiten administrar los archivos dentro de la aplicación (por ejemplo, agregar archivos, cambiarles el nombre o eliminarlos). Si desea tener la capacidad para administrar los archivos dentro de la aplicación, puede configurar la implementación continua y usar el entorno de desarrollo integrado (IDE) y el sistema de control de código fuente de su elección (por ejemplo, Visual Studio Team, GitHub, Bitbucket). Con la implementación continua configurada, los cambios de código que confirme al control de código fuente se implementarán automáticamente en Azure. Después de configurar la implementación continua, puede [depurar su bot localmente](bot-service-debug-bot.md).

> [!NOTE]
> Si habilita la implementación continua para el bot, tiene que comprobar los cambios de código para el servicio de control de código fuente. Si desea modificar el código en el editor de Azure una vez más, primero tiene que [deshabilitar la implementación continua](#disable-continuous-deployment).

Puede habilitar la implementación continua de la aplicación bot mediante los pasos siguientes.

## <a name="set-up-continuous-deployment-for-a-bot-on-an-app-service-plan"></a>Configuración de la implementación continua para un bot en un plan de App Service

En esta sección se describe cómo habilitar la implementación continua para un bot creado con Bot Service que tenga un plan de hospedaje de App Service.

1. En Azure Portal, busque su bot de Azure, haga clic en la pestaña **COMPILACIÓN** y busque la sección **Continuous deployment from source control** (Implementación continua desde el control de código fuente).
2. Para Visual Studio Online o GitHub, proporcione un token de acceso emitido en esos sitios web. El origen se extraerá desde Azure al repositorio de origen.
3. Para otros sistemas de control de código fuente, seleccione **otros** y siga los pasos que aparecen. 
3. Hacer clic en **Habilitar**.  

### <a name="create-an-empty-repository-and-download-bot-source-code"></a>Creación de un repositorio vacío y descarga del código fuente de bot

Siga estos pasos si desea usar un servicio de control de código fuente *distinto* de Visual Studio Online o Github. Visual Studio Online y Github extraerán el código fuente para el bot de Azure, por lo que los usuarios de esos dos servicios pueden omitir estos pasos.

3. Para un bot en un plan de App Service, busque la página del bot en Azure, haga clic en la pestaña **COMPILACIÓN**, busque la sección **Descargar el código fuente** y haga clic en **Descargar archivo zip**.
1. Cree un repositorio vacío dentro de uno de los sistemas de control de código fuente que admite Azure.

    ![Sistema de control de código fuente](~/media/continuous-integration-sourcecontrolsystem.png)

3. Extraiga el contenido del archivo zip descargado en la carpeta local donde se va a sincronizar su origen de implementación.
4. Haga clic en **Configurar** y siga los pasos que aparecen. 

## <a name="set-up-continuous-deployment-for-a-bot-on-a-consumption-plan"></a>Configuración de la implementación continua para un bot en un plan de consumo 

Elija el origen de implementación para el bot y conecte el repositorio. 

1. Dentro Azure Portal, en el bot de Azure, haga clic en la pestaña **CONFIGURACIÓN** y haga clic en **Configurar** para expandir la sección **Implementación continua**.  
2. Siga los pasos y haga clic en la casilla para confirmar que está preparado. 
3. Haga clic en **Configurar**, seleccione el origen de implementación que corresponde al sistema de control de código fuente en el que creó anteriormente el repositorio vacío, y complete los pasos para conectarlo.   


## <a name="disable-continuous-deployment"></a>Deshabilitación de la implementación continua 

Cuando se deshabilita la implementación continua, el servicio de control de código fuente sigue funcionando, pero los cambios que haya introducido no se publican automáticamente en Azure. Para deshabilitar la implementación continua, realice los pasos a continuación:

1. Si el bot tiene un plan de hospedaje de App Service dentro de Azure Portal, busque su bot de Azure, haga clic en la pestaña **COMPILACIÓN** y busque la sección **Continuous deployment from source control** (Implementación continua desde el control de código fuente), *o...* 
2. si su bot tiene un plan de consumo, haga clic en la pestaña **Configuración**, expanda la sección **Implementación continua** y haga clic en **Configurar**.
3. En el panel **Implementaciones**, seleccione el servicio de control de código fuente donde la implementación continua está habilitada, y haga clic en **Desconectar**.  


## <a name="additional-resources"></a>Recursos adicionales

Para aprender cómo depurar el bot localmente después de haber configurado la implementación continua, consulte [Depuración de un bot de Bot Service](bot-service-debug-bot.md).

En este artículo se resaltan las características específicas de la implementación continua de Bot Service. Para más información acerca de la implementación continua en relación con Azure App Service, consulte <a href="https://azure.microsoft.com/en-us/documentation/articles/app-service-continuous-deployment/" target="_blank">Implementación continua en Azure App Service</a>.

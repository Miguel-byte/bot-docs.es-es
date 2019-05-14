---
title: Configuración de implementación continua para Bot Service | Microsoft Docs
description: Obtenga información sobre cómo configurar una implementación continua desde el control de código fuente para una instancia de Bot Service.
keywords: implementación continua, publicar, implementar, azure portal
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/01/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d755572c6559ca1de7f0cf93a120273aa0fff947
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033110"
---
# <a name="set-up-continuous-deployment"></a>Configurar la implementación continua

[!INCLUDE [applies-to](./includes/applies-to.md)]

En este artículo se muestra cómo configurar la implementación continua en el bot. Puede habilitar la implementación continua para implementar automáticamente los cambios de código desde el repositorio de origen en Azure. En este tema, hablaremos sobre cómo configurar la implementación continua para GitHub. Para más información sobre cómo configurar la implementación continua con otros sistemas de control de código fuente, consulte la sección de recursos adicionales en la parte inferior de esta página.

## <a name="prerequisites"></a>Requisitos previos
- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](http://portal.azure.com) antes de empezar.
- **Debe** [implementar el bot en Azure](bot-builder-deploy-az-cli.md) antes de habilitar la implementación continua.

## <a name="prepare-your-repository"></a>Preparación del repositorio
Asegúrese de que la raíz del repositorio tiene los archivos correctos en el proyecto. Esto le permitirá obtener compilaciones automáticas desde el servidor de compilación de Kudu para Azure App Service. 

|Tiempo de ejecución | Archivos del directorio raíz |
|:-------|:---------------------|
| ASP.NET Core | .sln o .csproj |
| Node.js | server.js, app.js o package.json con un script de inicio |


## <a name="continuous-deployment-using-github"></a>Implementación continua con GitHub
Para habilitar la implementación continua con GitHub, vaya a la página de **App Service** del bot en Azure Portal.

Haga clic en **Centro de implementación** > **GitHub** > **Autorizar**.

![Implementación continua](~/media/azure-bot-build/azure-deployment.png)

En la ventana del explorador que se abre, haga clic en **Autorizar AzureAppService**. 

![Permiso de Azure GitHub](~/media/azure-bot-build/azure-deployment-github.png)

Después de autorizar **AzureAppService**, vuelva al **Centro de implementación** en Azure Portal.

1. Haga clic en **Continue**. 

1. Seleccione **Servicio de compilación de App Service**.

1. Haga clic en **Continue**.

1. Seleccione **Organización**, **Repositorio** y **Rama**.

1. Haga clic en **Continuar** y, después, en **Finalizar** para completar la configuración.

Con esto, la configuración de la implementación continua con GitHub está completa. Cada vez que confirme en el repositorio de código fuente, los cambios se implementarán automáticamente en Azure Bot Service.

## <a name="disable-continuous-deployment"></a>Deshabilitación de la implementación continua

Si bien el bot está configurado para la implementación continua, no puede usar el editor de código en línea para realizar cambios en el bot. Si desea usar el editor de código en línea, puede deshabilitar temporalmente la implementación continua.

Para deshabilitar la implementación continua, haga lo siguiente:
1. En [Azure Portal](https://portal.azure.com), vaya a la hoja **All App service settings** (Todos los valores de App Service) del bot y haga clic en **Centro de implementación**. 
1. Haga clic en **Desconectar** para deshabilitar la implementación continua. Para volver a habilitar la implementación continua, repita los pasos de las secciones anteriores correspondientes.

## <a name="additional-resources"></a>Recursos adicionales
- Para habilitar la implementación continua desde BitBucket y Azure DevOps Services, consulte [Implementación continua con Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/deploy-continuous-deployment).



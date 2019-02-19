---
redirect_url: /bot-framework/bot-builder-deploy-az-cli
ms.openlocfilehash: 8149471553658df6778e2983bae114e80c846c9b
ms.sourcegitcommit: 8183bcb34cecbc17b356eadc425e9d3212547e27
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/09/2019
ms.locfileid: "55971435"
---
<a name="--"></a><!--
---
título: Configuración de implementación continua para Bot Service | Microsoft Docs description: Obtenga información sobre cómo configurar una implementación continua desde el control de código fuente para una instancia de Bot Service. keywords: continuous deployment, publish, deploy, azure portal author: ivorb ms.author: v-ivorb manager: kamrani ms.topic: article ms.service: bot-service ms.date: 06/12/2018
---

# <a name="set-up-continuous-deployment"></a>Configurar la implementación continua
Si el código está insertado en **GitHub** o **Azure DevOps (anteriormente Visual Studio Team Services)**, use la implementación continua para implementar automáticamente los cambios de código desde el repositorio de origen en Azure. En este tema, hablaremos sobre cómo configurar la implementación continua para **GitHub** y **Azure DevOps**.

> [!NOTE]
> En el escenario que se describe en este artículo se supone que ha implementado un bot en Azure y que desea habilitar la implementación continua en dicho bot. Además, debe saber que tras la configuración de la implementación continua, el editor de código en línea de Azure Portal pasa a ser de solo lectura.

## <a name="continuous-deployment-using-github"></a>Implementación continua con GitHub

Para configurar la implementación continua mediante el repositorio de GitHub que contiene el código fuente que desea implementar en Azure, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), vaya a la hoja **All App service settings** (Todos los valores de App Service) del bot y haga clic en **Deployment options (Classic)** [Opciones de implementación (clásica)]. 

1. Haga clic en **Elegir origen** y seleccione **GitHub**.

   ![Elija GitHub](~/media/azure-bot-build/continuous-deployment-setup-github.png)

1. Haga clic en **Autorización**, luego en **Autorizar** y siga los avisos para proporcionar autorización para que Azure acceda a su cuenta de GitHub.

1. Haga clic en **Elegir proyecto** y elija un proyecto.

1. Haga clic en **Elegir rama** y elija una rama.

1. Haga clic en **Aceptar** para completar el proceso de configuración.

Ahora la configuración de la implementación continua con GitHub está completa. Cada vez que confirme en el repositorio de código fuente, los cambios se implementarán automáticamente en Azure Bot Service.

## <a name="continuous-deployment-using-azure-devops"></a>Implementación continua con Azure DevOps

1. En [Azure Portal](https://portal.azure.com), vaya a la hoja **All App service settings** (Todos los valores de App Service) del bot y haga clic en **Deployment options (Classic)** [Opciones de implementación (clásica)]. 
2. Haga clic en **Elegir origen** y seleccione **Visual Studio Team Services**. Tenga en cuenta que Visual Studio Team Services es ahora Azure DevOps Services.

   ![Elija Visual Studio Team Services:](~/media/azure-bot-build/continuous-deployment-setup-vs.png)

3. Haga clic en **Elegir la cuenta** y seleccione una cuenta.

> [!NOTE]
> Si no ve su cuenta en la lista, deberá [vincular la cuenta a su suscripción de Azure](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/connect-organization-to-azure-ad?view=vsts&tabs=new-nav). Solo se admiten proyectos VSTS Git.

4. Haga clic en **Elegir proyecto** y elija un proyecto.
5. Haga clic en **Elegir rama** y elija una rama.
6. Haga clic en **Aceptar** para completar el proceso de configuración.

   ![Configuración de Visual Studio](~/media/azure-bot-build/continuous-deployment-setup-vs-configuration.png)

Ahora la configuración de la implementación continua con Azure DevOps está completa. Cada vez que confirme, los cambios se implementarán automáticamente en Azure.

## <a name="disable-continuous-deployment"></a>Deshabilitación de la implementación continua

Si bien el bot está configurado para la implementación continua, no puede usar el editor de código en línea para realizar cambios en el bot. Si desea usar el editor de código en línea, puede deshabilitar temporalmente la implementación continua.

Para deshabilitar la implementación continua, haga lo siguiente:
1. En [Azure Portal](https://portal.azure.com), vaya a la hoja **All App service settings** (Todos los valores de App Service) del bot y haga clic en **Deployment options (Classic)** [Opciones de implementación (clásica)]. 
2. Haga clic en **Desconectar** para deshabilitar la implementación continua. Para volver a habilitar la implementación continua, repita los pasos de las secciones anteriores correspondientes.

## <a name="additional-information"></a>Información adicional
- Visual Studio Team Services es ahora [Azure DevOps Services](https://docs.microsoft.com/en-us/azure/devops/?view=vsts)


-->

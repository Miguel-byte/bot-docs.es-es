---
title: Configuración de implementación continua para Bot Service | Microsoft Docs
description: Obtenga información sobre cómo configurar una implementación continua desde el control de código fuente para una instancia de Bot Service.
keywords: implementación continua, publicar, implementar, azure portal
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
ms.openlocfilehash: 62cbbcc560e049776b8aa891c167b9a6eaba3264
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997802"
---
# <a name="set-up-continuous-deployment"></a>Configurar la implementación continua
Si el código está insertado en **GitHub** o **Azure DevOps (anteriormente Visual Studio Team Services)**, use la implementación continua para implementar automáticamente los cambios de código desde el repositorio de origen en Azure. En este tema, hablaremos sobre cómo configurar la implementación continua para **GitHub** y **Azure DevOps**.

> [!NOTE]
> Una vez configurada la implementación continua, el editor de código en línea de Azure Portal se convierte en *SOLO LECTURA*.

## <a name="continuous-deployment-using-github"></a>Implementación continua con GitHub

Para configurar una implementación continua con GitHub, haga lo siguiente:

1. Utilice el repositorio de GitHub que contiene el código fuente que desea implementar en Azure. Puede [bifurcar](https://help.github.com/articles/fork-a-repo/) un repositorio existente o crear el suyo propio y cargar el código fuente relacionado en el repositorio de GitHub.
2. En [Azure Portal](https://portal.azure.com), vaya a la hoja **Compilar** del bot y haga clic en **Configurar la implementación continua**. 
3. Haga clic en **Configuración**.
   
   ![Configurar implementación continua](~/media/azure-bot-build/continuous-deployment-setup.png)

4. Haga clic en **Elegir origen** y seleccione **GitHub**.

   ![Elija GitHub](~/media/azure-bot-build/continuous-deployment-setup-github.png)

5. Haga clic en **Autorización**, luego en **Autorizar** y siga los avisos para proporcionar autorización para que Azure acceda a su cuenta de GitHub.

6. Haga clic en **Elegir proyecto** y elija un proyecto.

7. Haga clic en **Elegir rama** y elija una rama.

8. Haga clic en **Aceptar** para completar el proceso de configuración.

Ahora la configuración de la implementación continua con GitHub está completa. Cada vez que confirme en el repositorio de código fuente, los cambios se implementarán automáticamente en Azure Bot Service.

## <a name="continuous-deployment-using-azure-devops"></a>Implementación continua con Azure DevOps

1. En [Azure Portal](https://portal.azure.com), en la hoja **Compilar** del bot, haga clic en **Configurar la implementación continua**. 
2. Haga clic en **Configuración**.
   
   ![Configurar implementación continua](~/media/azure-bot-build/continuous-deployment-setup.png)

3. Haga clic en **Elegir origen** y seleccione **Visual Studio Team Services**. Tenga en cuenta que Visual Studio Team Services es ahora Azure DevOps Services.

   ![Elija Visual Studio Team Services:](~/media/azure-bot-build/continuous-deployment-setup-vs.png)

4. Haga clic en **Elegir la cuenta** y seleccione una cuenta.

> [!NOTE]
> Si no ve su cuenta, asegúrese de que esté vinculada a su suscripción a Azure. Para vincular la cuenta a la suscripción de Azure, vaya a Azure Portal y abra **Organizaciones de Azure DevOps Services (anteriormente Team Services)**. Verá una lista de las organizaciones que tiene en Azure DevOps. Haga clic en la que contiene el código fuente del bot que desea implementar y encontrará un botón**Conectar AAD**. Si la organización que ha elegido no está vinculada a la suscripción de Azure, este botón estará activo. Por lo tanto, haga clic en este botón para configurar la conexión. Es posible que deba esperar un poco para que surta efecto.

5. Haga clic en **Elegir proyecto** y elija un proyecto.

> [!NOTE]
> Solo se admiten proyectos VSTS Git.

6. Haga clic en **Elegir rama** y elija una rama.
7. Haga clic en **Aceptar** para completar el proceso de configuración.

   ![Configuración de Visual Studio](~/media/azure-bot-build/continuous-deployment-setup-vs-configuration.png)

Ahora la configuración de la implementación continua con Azure DevOps está completa. Cada vez que confirme, los cambios se implementarán automáticamente en Azure.

## <a name="disable-continuous-deployment"></a>Deshabilitación de la implementación continua

Si bien el bot está configurado para la implementación continua, no puede usar el editor de código en línea para realizar cambios en el bot. Si desea usar el editor de código en línea, puede deshabilitar temporalmente la implementación continua.

Para deshabilitar la implementación continua, haga lo siguiente:

1. Desde la hoja **Compilar** del bot, haga clic en **Configure continuous deployment** (Configurar la implementación continua). 
2. Haga clic en **Desconectar** para deshabilitar la implementación continua. Para volver a habilitar la implementación continua, repita los pasos de las secciones anteriores correspondientes.

## <a name="additional-information"></a>Información adicional
- [Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/?view=vsts)

---
title: Creación de un nuevo proyecto con el bot de empresa | Microsoft Docs
description: Aprenda a crear un bot mediante la plantilla del bot de empresa.
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4b85b0e2d3c1ae8c30ea9d5d9fa62783c2968744
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645493"
---
# <a name="enterprise-bot-template---creating-a-new-project"></a>Plantilla del bot de empresa: Creación de un nuevo proyecto

> [!NOTE]
> Este tema se aplica a la versión v4 del SDK. 

Esta plantilla reúne todos los procedimientos recomendados y los componentes auxiliares que hemos identificado mediante la creación de experiencias de conversación. La plantilla está disponible en las siguientes plataformas del SDK Bot Builder:

- .NET
- Node.js (próximamente)

## <a name="net"></a>.NET

La plantilla del bot de empresa está disponible para. NET, especialmente para las versiones **V4** del SDK. Está disponible como un paquete [VSIX](https://docs.microsoft.com/en-us/visualstudio/extensibility/anatomy-of-a-vsix-package). Para descargar, haga clic en el siguiente vínculo:

- [Plantilla del bot de empresa para el SDK Bot Builder v4](https://aka.ms/GetEnterpriseBotTemplate)

#### <a name="prerequisites"></a>Requisitos previos

- [Visual Studio 2017 o superior](https://www.visualstudio.com/downloads/)
- [Cuenta de Azure](https://azure.microsoft.com/en-us/free/)
- [Azure PowerShell](https://docs.microsoft.com/en-us/powershell/azure/overview?view=azurermps-6.8.1)

### <a name="install-the-template"></a>Instalación de la plantilla

Desde el directorio de guardado, simplemente abra el paquete VSIX y la plantilla del bot de empresa se instalará en Visual Studio y estará disponible la próxima vez que la abra.

Para crear un nuevo proyecto de bot con la plantilla, simplemente abra Visual Studio y seleccione **Archivo** > **Nuevo** > **Proyecto** y, desde Visual C#, seleccione **Bot Framework** > Plantilla de bot de empresa. Esto creará localmente un nuevo proyecto de bot que puede modificar como quiera. 

![Archivar nueva plantilla de proyecto](media/enterprise-template/EnterpriseBot-NewProject.png)

## <a name="deploy-your-bot"></a>Implementación del bot

Ahora que ha creado el proyecto, el siguiente paso consiste en crear la infraestructura de Azure auxiliar y realizar la configuración e implementación que permita al bot funcionar desde el principio. Continúe en [Implementación del bot](bot-builder-enterprise-template-deployment.md).

> Debe ejecutar este paso ya que, de lo contrario, la inicialización del bot (AppInsights) y las dependencias de LUIS no estarán disponibles.
## <a name="customize-your-bot"></a>Personalización del bot

Después de comprobar que ha implementado correctamente el bot desde el principio, puede personalizarlo para adaptarlo a sus necesidades y al escenario. Continúe con [Personalización del bot](bot-builder-enterprise-template-customize.md).

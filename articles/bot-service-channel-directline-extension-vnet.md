---
title: Uso de la extensión de App Service para Direct Line en una red virtual
titleSuffix: Bot Service
description: Uso de la extensión de App Service para Direct Line en una red virtual
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 07/25/2019
ms.openlocfilehash: a37ae6f3a9a9cd43c28d353745f0da9065a7886b
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2019
ms.locfileid: "68757757"
---
## <a name="use-direct-line-app-service-extension-within-a-vnet"></a>Uso de la extensión de App Service para Direct Line en una red virtual

En este artículo se describe cómo usar la extensión de App Service para Direct Line con una red virtual de Azure (VNET).

### <a name="create-an-app-service-environment-and-other-azure-resources"></a>Creación de un entorno de App Service Environment y otros recursos de Azure

1. La extensión de App Service para Direct Line está disponible en todos los servicios de **Azure App Service**, incluidos los hospedados en un entorno de **Azure App Service Environment**. Un entorno de Azure App Service Environment proporciona aislamiento y es ideal para trabajar en una red virtual.
    - Puede encontrar instrucciones para crear un entorno de App Service Environment externo en el artículo [Creación de un entorno de App Service externo](https://docs.microsoft.com/en-us/azure/app-service/environment/create-external-ase).
    - Puede encontrar instrucciones para crear un entorno de App Service Environment interno en el artículo [Creación y uso de un entorno de App Service con Load Balancer interno](https://docs.microsoft.com/en-us/azure/app-service/environment/create-ilb-ase).
1. Una vez que haya creado el entorno de App Service Environment, debe agregar un plan de App Service en él donde puede implementar los bots (y, por tanto, ejecutar la extensión de App Service para Direct Line). Para ello, siga estos pasos:
    - Vaya a https://portal.azure.com/.
    - Cree un nuevo recurso "Plan de App Service".
    - En Región, seleccione su entorno de App Service Environment.
    - Termine de crear el plan de App Service.

### <a name="configure-the-vnet-network-security-groups-nsg"></a>Configuración de los grupos de seguridad de red (NSG) de la red virtual

1. La extensión de App Service para Direct Line requiere una conexión de salida para que pueda emitir solicitudes HTTP. Esto se puede configurar como una regla de salida en el grupo de seguridad de red de la red virtual que está asociada a la subred del entorno de App Service Environment. La regla que se requiere es la siguiente:

|Source|Any|
|---|---|
|Puerto de origen|*|
|Destino|Direcciones IP|
|Direcciones IP de destino|52.155.168.246, 13.83.242.172|
|Intervalos de puertos de destino|443|
|Protocolo|Any|
|Action|Allow|


![Arquitectura de la extensión de Direct Line](./media/channels/direct-line-extension-vnet.png)

>[!NOTE]
> Las direcciones IP proporcionadas son explícitamente para la versión preliminar. Se va a trasladar a una etiqueta de servicio para Azure Bot Service más adelante en el año, lo que cambiará esta configuración.

### <a name="configure-your-bots-app-service"></a>Configuración de la instancia de App Service del bot

Para la versión preliminar, deberá cambiar el modo en que la extensión de App Service para Direct Line se comunica con Azure. Para hacerlo, agregue una nueva **Configuración de la aplicación de App Service** a la aplicación mediante el portal o el archivo `applicationsettings.json`:

- Propiedad: DirectLineExtensionABSEndpoint
- Valor: https://dlase.botframework.com/v3/extension

>[!NOTE]
> Solo será necesario para la versión preliminar de la extensión de App Service para Direct Line.

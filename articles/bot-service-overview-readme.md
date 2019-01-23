---
title: Cómo funciona Bot Service | Microsoft Docs
description: Conozca las características y funcionalidades de Bot Service.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 85ef0fde39980bab1b891518e338fddbd56b275a
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224460"
---
# <a name="how-bot-service-works"></a>Cómo funciona Bot Service

Bot Service ofrece los componentes principales para la creación de bots, incluido Bot Framework SDK para desarrollar bots y Bot Framework para conectar bots a canales. Bot Service ofrece cinco plantillas entre las que puede elegir al crear sus bots con compatibilidad con .NET y Node.js.

> [!IMPORTANT]
> Para usar Bot Service debe tener una suscripción a Microsoft Azure. Si aún no tiene una suscripción, puede registrarse para obtener una <a href="https://azure.microsoft.com/en-us/free/" target="_blank">cuenta gratuita</a>.

## <a name="hosting-plans"></a>Planes de hospedaje
Bot Service ofrece dos planes de hospedaje para los bots. Puede elegir el que se ajuste a sus necesidades.

### <a name="app-service-plan"></a>Plan de App Service

Un bot que usa un plan de App Service es una aplicación web de Azure estándar que se puede configurar para asignar una capacidad predefinida con costos y escalado predecibles. Con un bot que usa este plan de hospedaje, puede:

* Editar código fuente de bot en línea con un editor de código en el explorador avanzado.
* Descargar, depurar y volver a publicar el bot de C# con Visual Studio.
* Configurar la implementación continua de forma fácil para Visual Studio Online y Github.
* Use el código de ejemplo preparado para Bot Framework SDK.

### <a name="consumption-plan"></a>Plan de consumo
Un bot que usa un plan de consumo es un bot sin servidor que se ejecuta en <a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a>, y usa los precios de pago por ejecución de Azure Functions. Un bot que usa este plan de hospedaje puede escalarse para administrar picos enormes de tráfico. Puede editar el código fuente de bot en línea con un editor de código en el explorador básico. Para más información sobre el entorno de ejecución de un bot de plan de consumo, consulte <a target='_blank' href='/azure/azure-functions/functions-scale'>Planes de consumo y de App Service de Azure Functions</a>.

## <a name="templates"></a>Plantillas

Bot Service le permite crear rápida y fácilmente un bot en C# o Node.js mediante cinco plantillas.

[!INCLUDE [Bot Service templates](~/includes/snippet-abs-templates.md)]

[Más información](bot-service-concept-templates.md) sobre las diferentes plantillas y cómo pueden ayudarle a crear bots.

## <a name="develop-and-deploy"></a>Desarrollo e implementación

De forma predeterminada, Bot Service le permite desarrollar su bot directamente en el explorador con el editor de código en línea, sin necesidad de una cadena de herramientas. 

Puede desarrollar y depurar el bot localmente con Bot Framework SDK y un IDE como Visual Studio 2017. Puede publicar su bot directamente en Azure con Visual Studio 2017 o la CLI de Azure. También puede [configurar la implementación continua](bot-service-continuous-deployment.md) con el sistema de control de código fuente de su elección, como VSTS o GitHub. Con la implementación continua configurada, puede desarrollar y depurar en un IDE en el equipo local, y todos los cambios en el código que se confirmen en el control de código fuente se implementarán automáticamente en Azure.  

> [!TIP]
> Para evitar conflictos, después de habilitar la implementación continua asegúrese de modificar el código únicamente mediante esta implementación y no por medio de otros mecanismos.

## <a name="manage-your-bot"></a>Administración de bots 

Durante el proceso de creación de un bot con Bot Service, especificará un nombre para el bot, configurará su plan de hospedaje, elegirá un plan de tarifa y configurará algunas otras opciones. Una vez creado el bot, puede cambiar su configuración, configurarlo para ejecutarse en uno o más canales y probarlo en Chat en web. 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de un bot con Bot Service](bot-service-quickstart.md)
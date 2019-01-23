---
ms.openlocfilehash: 8eb125b2d0f0c9981cafb763ba809c7d042d8f77
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225990"
---
Bot Service ofrece dos planes de hospedaje distintos para bots. La conversión del código fuente de un bot de un plan a otro es un proceso manual.   

## <a name="app-service-plan"></a>Plan de App Service

Un bot que usa un plan de App Service es una aplicación web de Azure estándar que se puede configurar para asignar una capacidad predefinida con costos y escalado predecibles. Con un bot que usa este plan de hospedaje, puede:

* Editar código fuente de bot en línea con un editor de código en el explorador avanzado.
* Descargar, depurar y volver a publicar el bot de C# con Visual Studio.
* Configurar la implementación continua de forma fácil para Visual Studio Online y Github.
* Use el código de ejemplo preparado para Bot Framework SDK.

## <a name="consumption-plan"></a>Plan de consumo

Un bot que usa un plan de consumo es un bot sin servidor que se ejecuta en <a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a>, y usa los precios de pago por ejecución de Azure Functions. Un bot que usa este plan de hospedaje puede escalarse para administrar picos enormes de tráfico. Puede editar el código fuente de bot en línea con un editor de código en el explorador básico. Para más información sobre el entorno de ejecución de un bot de plan de consumo, consulte <a target='_blank' href='/azure/azure-functions/functions-scale'>Planes de consumo y de App Service de Azure Functions</a>.

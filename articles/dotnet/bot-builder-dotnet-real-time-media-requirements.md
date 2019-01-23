---
title: Requisitos y consideraciones para bots de elementos multimedia en tiempo real | Microsoft Docs
description: Información acerca de los requisitos y las consideraciones importantes relacionadas con la creación de bots de elementos multimedia en tiempo real para Skype con Bot Framework SDK para .NET.
author: ssulzer
ms.author: ssulzer
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d8cd326a3027fe5fcb440d9b205ba7d32a8b1640
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224940"
---
# <a name="requirements-and-considerations-for-real-time-media-bots"></a>Requisitos y consideraciones para bots de elementos multimedia en tiempo real

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

No todas las instrucciones aplicables al desarrollo de bots de mensajería y llamadas IVR se aplican por igual a la creación de bots de elementos multimedia en tiempo real. En este artículo se describe algunos de los importantes requisitos y consideraciones para el desarrollo de bots de elementos multimedia en tiempo real. 

> [!NOTE]
> Dado que la plataforma de elementos multimedia en tiempo real para bots es una tecnología de versión preliminar, las instrucciones de este artículo está sujetas a cambios.

## <a name="real-time-media-bot-development-requires-cnet-and-windows-server"></a>El desarrollo de bots de elementos multimedia en tiempo real requiere C# / .NET y Windows Server

- El bot de elementos multimedia en tiempo real requiere la biblioteca `Microsoft.Skype.Bots.Media` para .NET (disponible en <a href="https://www.nuget.org/" target="_blank">NuGet</a>) para acceder a los elementos multimedia de audio y vídeo, y el bot tiene que implementarse en una máquina Windows Server (o un sistema operativo invitado Windows Server en Azure). Por ello, el bot debe desarrollarse con C# y el estándar de .NET Framework e implementarse en Azure. No se pueden usar las API de C++ y Node.js para acceder a los elementos multimedia en tiempo real.

- El bot de elementos multimedia en tiempo real se tiene que ejecutar en una versión reciente de la biblioteca `Microsoft.Skype.Bots.Media`. NET. El bot debe usar o bien la versión disponible más reciente del paquete <a href="https://www.nuget.org/" target="_blank">NuGet</a>, o una versión que no tenga más de tres meses. Las versiones anteriores de la biblioteca quedarán en desuso y no funcionarán después de unos meses.

## <a name="real-time-media-calls-stay-on-the-machine-where-they-were-created"></a>Las llamadas multimedia en tiempo real permanecen en la máquina en la que se crearon

- Los bots de elementos multimedia en tiempo real tienen un estado. La llamada multimedia en tiempo real está anclada a la instancia de máquina virtual que aceptó la llamada entrante: los elementos multimedia de voz y vídeo del autor de la llamada de Skype fluirán a esa instancia de máquina virtual, y los elementos multimedia que el bot envía de vuelta al autor de la llamada también tienen que originarse en esa misma máquina virtual.

- Si hay una llamada de contenido multimedia en tiempo real en curso, cuando se detiene la máquina virtual, esas llamadas se cortarán de forma abrupta. Si el bot puede tener conocimiento del cierre pendiente de máquina virtual, puede intentar finalizar las llamadas de forma "elegante".

## <a name="real-time-media-bots-must-be-directly-accessible-on-the-internet"></a>Los bots de elementos multimedia en tiempo real tienen que ser accesibles directamente en Internet

- Cada instancia de máquina virtual que hospeda un bot de elementos multimedia en tiempo real tiene que ser accesible directamente desde Internet. Para los bots de elementos multimedia en tiempo real hospedados en Azure, cada instancia de máquina virtual tiene que tener una dirección IP pública de nivel de instancia (ILPIP). Para más información acerca de cómo obtener y configurar una ILPIP, consulte <a href="/azure/virtual-network/virtual-networks-instance-level-public-ip" target="_blank">Introducción a las direcciones IP públicas a nivel de instancia (clásica)</a>. De forma predeterminada, una suscripción de Azure puede obtener 5 direcciones ILPIP. Póngase en contacto con soporte técnico de Azure para aumentar la cuota de la suscripción.

- Debido a los requisitos de la dirección IP pública, los bots de elementos multimedia en tiempo real tienen que hospedarse en una instancia de Azure Virtual Machine "IaaS" o en una instancia "clásica" de Azure Cloud Service. Otros entornos en tiempo de ejecución, como Bot Service o Service Fabric con Virtual Machine Scale Sets, no se admiten, ya que no son compatibles con ILPIP.

- Los bots de elementos multimedia en tiempo real tampoco son compatibles con [Bot Framework Emulator](../bot-service-debug-emulator.md).

## <a name="scalability-and-performance-considerations"></a>Consideraciones de escalabilidad y rendimiento

- Los bots de elementos multimedia en tiempo real requieren más capacidad de proceso y de red (ancho de banda) que los bots de mensajería y pueden incurrir en costos operativos significativamente mayores. Un desarrollador de bot de elementos multimedia en tiempo real tiene que medir con cuidado la escalabilidad del bot y asegúrese de que el bot no acepta más llamadas simultáneas de las que puede administrar. Un bot con vídeo habilitado puede tener capacidad para mantener solo una o dos llamadas multimedia en tiempo real simultáneas por núcleo de CPU.

- La versión preliminar actual de la plataforma de elementos multimedia en tiempo real para bots tiene ciertas limitaciones de escalabilidad que los desarrolladores de bots deben tener en cuenta (estas limitaciones pueden mejorarse en futuras versiones): 
  1. Una única instancia de máquina virtual no puede tener más de 10 sockets de vídeo creados en cualquier momento en el tiempo.
  2. La plataforma de elementos multimedia en tiempo real no aprovecha en la actualidad ninguna unidad de procesamiento gráfico (GPU) que esté disponible en la máquina virtual para descargar H.264 para codificación y descodificación de vídeo. En su lugar, la codificación y descodificación de vídeo se realiza en software en la CPU. Si una GPU está disponible, el bot puede aprovecharla para su propia representación gráfica (por ejemplo, si el bot usa un motor de gráficos 3D).

- La instancia de máquina virtual que hospeda el bot de elementos multimedia en tiempo real tiene que tener al menos 2 núcleos de CPU. Para Azure, se recomienda una máquina virtual de serie Dv2. La información detallada sobre los tipos de máquina virtual de Azure está disponible en la <a href="/azure/virtual-machines/windows/sizes-general" target="_blank">documentación de Azure</a>. 

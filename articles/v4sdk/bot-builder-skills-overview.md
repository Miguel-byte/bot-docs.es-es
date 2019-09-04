---
title: Introducción a las aptitudes de Bot Framework | Microsoft Docs
description: Más información acerca de las aptitudes de Bot Framework
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 566eb58f98335b9c55499051c6a23a6e0e0b6bc5
ms.sourcegitcommit: eacf1522d648338eebefe2cc5686c1f7866ec6a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2019
ms.locfileid: "70167119"
---
# <a name="virtual-assistant---skills-overview"></a>Virtual Assistant: introducción a las aptitudes

> [!NOTE]
> Este tema se aplica a la versión v4 del SDK. 

## <a name="overview"></a>Información general

Los desarrolladores pueden componer experiencias de conversación mediante la combinación de funcionalidades de conversación reutilizables, conocidas como aptitudes.

Dentro de una empresa, esto puede ser la creación de un bot primario que reúna varios bots secundarias que pertenezcan a distintos equipos, o de manera más amplia, que aproveche las funcionalidades comunes que proporcionan otros desarrolladores. Con esta versión preliminar de las aptitudes, los desarrolladores pueden crear un bot (normalmente a través de la plantilla de Virtual Assistant) y agregar o quitar aptitudes con una operación de la línea de comandos que incorpore todos los cambios de configuración y envío.     

Las aptitudes son en sí mismas bots, invocadas de forma remota y está disponible una plantilla de desarrollador de aptitudes (.NET, TS) para facilitar la creación de nuevas aptitudes.

Un objetivo de diseño clave de las aptitudes ha sido mantener el protocolo de actividad coherente y garantizar que el desarrollo estaba lo más cerca posible de cualquier bot de la versión V4 del SDK. 

![Escenarios de aptitudes](./media/enterprise-template/skills-scenarios.png)

## <a name="bot-framework-skills"></a>Aptitudes de Bot Framework

En este momento están disponibles las siguientes aptitudes de Bot Framework, con la tecnología de Microsoft Graph y en varios idiomas.

![Escenarios de aptitudes](./media/enterprise-template/skills-at-build.png)

| NOMBRE | DESCRIPCIÓN |
| ---- | ----------- |
|[Aptitud de calendario](https://aka.ms/bf-calendar-skill)|Agregar funcionalidades de calendario al asistente. Con tecnología de Microsoft Graph y Google.|
|[Aptitud de correo electrónico](https://aka.ms/bf-email-skill)|Agregar funcionalidades de correo al asistente. Con tecnología de Microsoft Graph y Google.|
|[Aptitud de tarea pendientes](https://aka.ms/bf-todo-skill)|Agregar funcionalidades de administración de tarea al asistente. Con tecnología de Microsoft Graph.|
|[Aptitud de punto de interés](https://aka.ms/bf-poi-skill)|Buscar puntos de interés y direcciones. Con tecnología de Azure Maps y FourSquare.|
|[Aptitud de automoción](https://aka.ms/bf-autos-kill)|Aptitud vertical de la industria para mostrar la habilitación del control de características del automóvil.|
|[Aptitudes experimentales](https://aka.ms/bf-experimental-skills)|Noticias, reservas en restaurantes y previsiones meteorológicas.|

## <a name="getting-started"></a>Introducción

Consulte los [tutoriales](https://aka.ms/bfs-tutorials) para aprender a aprovechar las aptitudes existentes y a crear las propias.

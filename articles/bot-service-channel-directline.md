---
title: Acerca del canal de Direct Line
titleSuffix: Bot Service
description: Características del canal de Direct Line
services: bot-service
author: ivorb
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.date: 11/01/2019
ms.author: kamrani
ms.openlocfilehash: 1caf469a7ec37e932ae2b7ffde85d0dbf24938a3
ms.sourcegitcommit: 312a4593177840433dfee405335100ce59aac347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/12/2019
ms.locfileid: "73933478"
---
# <a name="about-direct-line"></a>Acerca de Direct Line

El canal de Direct Line de Bot Framework es una manera fácil de integrar el bot en la aplicación móvil, página web u otro tipo de aplicación.
Direct Line está disponible en tres formas:
- Servicio Direct Line: un servicio global y sólido para la mayoría de los desarrolladores
- Extensión Direct Line de App Service: funcionalidad de Direct Line dedicada para la seguridad y el rendimiento (disponible en versión preliminar pública)
- Direct Line Speech: optimizado para voz de alto rendimiento (disponibilidad general)

Puede elegir qué oferta de Direct Line es la mejor para usted mediante la evaluación de las características que ofrece cada una y las necesidades de su solución. Estas ofertas serán simplificadas con el tiempo.

|                            | Direct Line | Extensión de App Service de Direct Line | Direct Line Speech |
|----------------------------|-------------|-----------------------------------|--------------------|
| Disponibilidad y licencias    | Disponibilidad general | Versión preliminar privada, sin SLA  | GA |
| Rendimiento en reconocimiento de voz y de texto a voz | Estándar | Estándar | Alto rendimiento |
| Es compatible con los exploradores web heredados | Sí | Disponibilidad general | Sí |
| Compatibilidad con Bot Framework SDK | Todas las v3, v4 | Requiere v4.63 y versiones posteriores | Requiere v4.63 y versiones posteriores |
| Compatibilidad con SDK de cliente    | JS, C# | JS, C# | C++, C#, Unity, JS|
| Funciona con Chat en web  | Sí | Sí | Sin|
| VNET | Sin | Vista previa | Sin |


## <a name="addtional-resources"></a>Recursos adicionales
- [Conectar un bot con Direct Line](bot-service-channel-connect-directline.md)
- [Conexión de un bot con Direct Line Speech](bot-service-channel-connect-directlinespeech.md)

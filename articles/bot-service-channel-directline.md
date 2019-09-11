---
title: Acerca del canal de Direct Line
titleSuffix: Bot Service
description: Características del canal de Direct Line
services: bot-service
author: ivorb
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: kamrani
ms.openlocfilehash: 3a1bf9ef71ec3121cc845a2a93fd191eacc28f8f
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298194"
---
## <a name="about-direct-line"></a>Acerca de Direct Line

El canal de Direct Line de Bot Framework es una manera fácil de integrar el bot en la aplicación móvil, página web u otro tipo de aplicación.
Direct Line está disponible en tres formas:
- Servicio Direct Line: un servicio global y sólido para la mayoría de los desarrolladores
- Extensiones de App Service de Direct Line: funcionalidad dedicada de Direct Line para la seguridad y el rendimiento (disponible en la versión preliminar privada a partir de mayo de 2019)
- Direct Line Speech: optimizado para voz de alto rendimiento (disponible en la versión preliminar privada a partir de mayo de 2019)

Puede elegir qué oferta de Direct Line es la mejor para usted mediante la evaluación de las características que ofrece cada una y las necesidades de su solución. Estas ofertas serán simplificadas con el tiempo.

|                            | Direct Line | Extensión de App Service de Direct Line | Direct Line Speech |
|----------------------------|-------------|-----------------------------------|--------------------|
| Disponibilidad y licencias    | Disponibilidad general | Versión preliminar privada, sin Acuerdo de Nivel de Servicio  | Versión preliminar privada, sin Acuerdo de Nivel de Servicio |
| Rendimiento en reconocimiento de voz y de texto a voz | Estándar | Estándar | Alto rendimiento |
| OAuth integrado           | Sí | Sí | Sin |
| Telemetría integrada       | Sí | Sí | Sin |
| Es compatible con los exploradores web heredados | Sí | No | Sin |
| Compatibilidad con Bot Framework SDK | Todas las v3, v4 | Necesaria v4.5+ | Necesaria v4.5+ |
| Compatibilidad con SDK de cliente    | JS, C# | JS, C# | C++, C#, Unity |
| Funciona con Chat en web  | Sí | Sí | Sin|
| API de historial de conversaciones | Sí | Sí| Sin|
| VNET | Sin | Versión preliminar* | Sin |

_* Las extensiones de App Service de Direct Line se pueden usar en redes virtuales, pero aún no permiten la restricción de las llamadas salientes._

## <a name="addtional-resources"></a>Recursos adicionales
- [Conectar un bot con Direct Line](bot-service-channel-connect-directline.md)
- [Conexión de un bot con Direct Line Speech](bot-service-channel-connect-directlinespeech.md)

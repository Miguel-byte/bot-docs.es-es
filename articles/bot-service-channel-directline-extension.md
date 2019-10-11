---
title: Extensión de App Service para Direct Line
titleSuffix: Bot Service
description: Características de la extensión de App Service para Direct Line
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 07/09/2019
ms.openlocfilehash: c4c54e50450ae81098992c880e23a049229fa09f
ms.sourcegitcommit: 7e901f5f39a0cfb0d37e532321b90a1dcf4baadd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/08/2019
ms.locfileid: "72039759"
---
# <a name="direct-line-app-service-extension"></a>Extensión de App Service de Direct Line

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

> [!WARNING]
> La **extensión Direct Line de App Service** está en **versión preliminar** pública.  

La extensión de App Service para Direct Line permite que los clientes se conecten directamente con el host en el que se encuentra el bot. Esto proporciona aislamiento de la carga de trabajo y, en algunos casos, un rendimiento mejorado. En la imagen siguiente se muestra la arquitectura general:

![Arquitectura de la extensión de Direct Line](./media/channels/direct-line-extension-architecture.png)

La extensión de App Service para Direct Line agrega un nuevo conjunto de extensiones de streaming al protocolo de Bot Framework, que reemplazan a HTTP para intercambiar mensajes con un transporte que permite enviar solicitudes bidireccionales mediante un **WebSocket persistente**.

Antes de las extensiones de streaming, Direct Line API ofrecía una manera para que un cliente pudiera enviar actividades a Direct Line y dos maneras para que un cliente pudiera recuperar actividades de Direct Line. Los mensajes se enviaban mediante HTTP POST y se recibían mediante HTTP GET (sondeo) o mediante la apertura de un WebSocket para recibir elementos ActivitySets.
Las extensiones de streaming amplían el uso del WebSocket y permiten enviar **toda la comunicación de mensajería** mediante ese WebSocket. También se pueden usar las extensiones de streaming entre los servicios del canal y el bot.

La extensión de App Service para Direct Line está preinstalada en todas las instancias de Azure App Services en todos los centros de datos de todo el mundo. Microsoft la mantiene y administra sin ningún trabajo de implementación adicional por parte del cliente. Está deshabilitada de forma predeterminada en Azure App Services, pero se puede activar con facilidad para que pueda conectarse al bot hospedado.


## <a name="see-also"></a>Otras referencias

|NOMBRE|DESCRIPCIÓN|
|---|---|
|[Configuración de un bot de .NET para la extensión](bot-service-channel-directline-extension-net-bot.md)|Actualizar un bot para trabajar con **canalizaciones con nombre** y habilitar la extensión de App Service para Direct Line en el recurso de **Azure App Service** en el que se hospeda el bot  |
|[Creación de un cliente de .NET con extensión](bot-service-channel-directline-extension-net-client.md)|Creación de un cliente de .NET en C# que se conecta a la extensión de App Service para Direct Line|
|[Uso de la extensión con WebChat](bot-service-channel-directline-extension-webchat-client.md)|Uso de WebChat con la extensión de App Service para Direct Line|
|[Uso de la extensión en la red virtual](bot-service-channel-directline-extension-vnet.md)|Uso de la extensión de App Service para Direct Line con una red virtual de Azure (VNET)|

## <a name="addtional-resources"></a>Recursos adicionales

- [Conectar un bot con Direct Line](bot-service-channel-connect-directline.md)
- [Conexión de un bot con Direct Line Speech](bot-service-channel-connect-directlinespeech.md)

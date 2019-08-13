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
ms.openlocfilehash: 27067cf2582de63cf67785cfaaa70b9a33685894
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2019
ms.locfileid: "68757767"
---
## <a name="direct-line-app-service-extension"></a>Extensión de App Service de Direct Line

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

La extensión de App Service para Direct Line establece un conjunto de **canalizaciones con nombre persistentes** para conectarse a un bot mediante un elemento **BotAdapter** que el cliente puede agregar.

Agrega un nuevo conjunto de extensiones de streaming al protocolo de Bot Framework. Estas extensiones reemplazan a HTTP como el transporte para intercambiar mensajes por un transporte que permite enviar solicitudes bidireccionales mediante un **WebSocket persistente**. Esto aumenta el rendimiento y permite más aislamiento durante el intercambio de información.

Antes de las extensiones de streaming, Direct Line API ofrecía una manera para que un cliente pudiera enviar actividades a Direct Line y dos maneras para que un cliente pudiera recuperar actividades de Direct Line. Los mensajes se enviaban mediante HTTP POST y se recibían mediante HTTP GET (sondeo) o mediante la apertura de un WebSocket para recibir elementos ActivitySets.
Las extensiones de streaming amplían el uso del WebSocket y permiten enviar **toda la comunicación de mensajería** mediante ese WebSocket. También se pueden usar las extensiones de streaming entre los servicios del canal y el bot.


La extensión de App Service para Direct Line está preinstalada en todas las instancias de Azure App Services en todos los centros de datos de todo el mundo. Microsoft la mantiene y administra sin ningún trabajo de implementación adicional por parte del cliente.
Está deshabilitada de forma predeterminada en Azure App Services, pero se puede activar con facilidad para que pueda conectarse al bot hospedado.

En la imagen siguiente se muestra la arquitectura general:

![Arquitectura de la extensión de Direct Line](./media/channels/direct-line-extension-architecture.png)

## <a name="see-also"></a>Otras referencias

|NOMBRE|DESCRIPCIÓN|
|---|---|
|[Bot de .NET con extensión](bot-service-channel-directline-extension-net-bot.md)|Actualizar un bot para trabajar con **canalizaciones con nombre** y habilitar la extensión de App Service para Direct Line en el recurso de **Azure App Service** en el que se hospeda el bot  |
|[Cliente de .NET con la extensión](bot-service-channel-directline-extension-net-client.md)|Crear un cliente de .NET en C# que se conecta a la extensión de App Service para Direct Line|
|[WebChat con extensión](bot-service-channel-directline-extension-webchat-client.md)|Uso de WebChat con la extensión de App Service para Direct Line|
|[Uso de la extensión en la red virtual](bot-service-channel-directline-extension-vnet.md)|Usar la extensión de App Service para Direct Line con una red virtual de Azure (VNET)|

## <a name="addtional-resources"></a>Recursos adicionales

- [Conectar un bot con Direct Line](bot-service-channel-connect-directline.md)
- [Conexión de un bot con Direct Line Speech](bot-service-channel-connect-directlinespeech.md)

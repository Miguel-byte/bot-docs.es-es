---
title: Conceptos clave de los servicios Bot Connector y Bot State | Microsoft Docs
description: Conozca los conceptos clave de los servicios Bot Connector y Bot State de Bot Framework.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/16/2019
ms.openlocfilehash: 0e97ad58fc81b52086eb691950845eec1665b064
ms.sourcegitcommit: e815e786413296deea0bd78e5a495df329a9a7cb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2019
ms.locfileid: "70876084"
---
# <a name="key-concepts"></a>Conceptos clave

Puede usar el servicio Bot Connector para comunicarse con los usuarios a través de varios canales, como Skype, correo electrónico, Slack y muchos más. En este artículo se presentan los conceptos clave del servicio Bot Connector.

## <a name="bot-connector-service"></a>Servicio Bot Connector

El servicio Bot Connector permite al bot intercambiar mensajes con canales configurados en el <a href="https://dev.botframework.com/" target="_blank">portal de Bot Framework</a>. Usa los estándares del sector REST y JSON a través de HTTPS y permite la autenticación con tokens de portador JWT. Para obtener información detallada sobre cómo usar el servicio Bot Connector, vea [Autenticación](bot-framework-rest-connector-authentication.md) y los demás artículos de esta sección.

### <a name="activity"></a>Actividad

Bot Connector Service intercambia información entre el bot y el canal (usuario) pasando un objeto [Actividad][]. El tipo de actividad más común es **message**, pero hay otros tipos de actividades que se pueden usar para comunicar distintos tipos de información a un bot o canal. Para obtener información detallada sobre las actividades del servicio Bot Connector, consulte [Introducción a las actividades]https://aka.ms/botSpecs-activitySchema.

## <a name="bot-state-service"></a>Servicio Bot State

El servicio de estado de Microsoft bot Framework se retirará a partir del 30 de marzo de 2018. Anteriormente, los bots creados en Azure Bot Service o en el SDK Bot Builder tenían una conexión predeterminada con este servicio hospedado por Microsoft para almacenar los datos de estado del bot. Los bots se deberán actualizar para usar su propio almacenamiento del estado.

## <a name="authentication"></a>Authentication

Los servicio Bot Connector habilita la autenticación con tokens de portador de JWT. Para obtener información detallada sobre cómo autenticar las solicitudes salientes que envía el bot de Bot Framework, cómo autenticar las solicitudes entrantes que recibe su bot desde Bot Framework y mucho más, vea [Autenticación](bot-framework-rest-connector-authentication.md). 

## <a name="client-libraries"></a>Bibliotecas de clientes

Bot Framework proporciona bibliotecas cliente que se pueden usar para crear bots en C# o Node.js. 

- Para crear un bot con C#, use [Bot Framework SDK para C#](../dotnet/bot-builder-dotnet-overview.md). 
- Para crear un bot con Node.js, use [Bot Framework SDK para Node.js](../nodejs/index.md). 

Además de modelar el servicio Bot Connector, cada Bot Framework SDK también proporciona un sistema eficaz para crear diálogos que contienen la lógica de la conversación, mensajes integrados para respuestas tan sencillas como Sí/No, cadenas, números y enumeraciones, compatibilidad integrada con marcos eficaces de inteligencia artificial, como <a href="https://www.luis.ai/" target="_blank">LUIS</a> y mucho más. 

> [!NOTE]
> Como alternativa al uso de los SDK para C# o Node.js, puede generar su propia biblioteca cliente en el lenguaje de su elección mediante el <a href="https://aka.ms/connector-swagger-file" target="_blank">archivo Swagger de Bot Connector</a>.

## <a name="additional-resources"></a>Recursos adicionales

Para más información sobre cómo crear bots con el servicio Bot Connector, consulte los artículos de esta sección, que comienzan por el artículo de [Autenticación](bot-framework-rest-connector-authentication.md). Si tiene problemas o sugerencias sobre el servicio Bot Connector, consulte el artículo [Soporte técnico](../bot-service-resources-links-help.md) para obtener una lista de los recursos disponibles. 

[Actividad]: bot-framework-rest-connector-api-reference.md#activity-object

---
title: Conceptos clave de los servicios Bot Connector y Bot State | Microsoft Docs
description: Conozca los conceptos clave de los servicios Bot Connector y Bot State de Bot Framework.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: a3d6bd957b835a0b8d86e47595ce28506c32d636
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224560"
---
# <a name="key-concepts"></a>Conceptos clave

Puede usar los servicios Bot Connector y Bot State para comunicarse con los usuarios a través de varios canales, como Skype, correo electrónico, Slack y muchos más. En este artículo se presentan los conceptos clave de los servicios Bot Connector y Bot State.

> [!IMPORTANT]
> No se recomienda la API del servicio de estado de Bot Framework para entornos de producción y es posible que se encuentre en desuso en una versión futura. Se recomienda actualizar el código del bot para que use el almacenamiento en memoria para realizar pruebas o usar una de las **extensiones de Azure** para bots de producción. Para obtener más información, consulte el tema **Administración de datos de estado** para la implementación de [.NET](~/dotnet/bot-builder-dotnet-state.md) o [Node](~/nodejs/bot-builder-nodejs-state.md).

## <a name="bot-connector-service"></a>Servicio Bot Connector

El servicio Bot Connector permite al bot intercambiar mensajes con canales configurados en el <a href="https://dev.botframework.com/" target="_blank">portal de Bot Framework</a>. Usa los estándares del sector REST y JSON a través de HTTPS y permite la autenticación con tokens de portador JWT. Para obtener información detallada sobre cómo usar el servicio Bot Connector, vea [Autenticación](bot-framework-rest-connector-authentication.md) y los demás artículos de esta sección.

### <a name="activity"></a>Actividad

Bot Connector Service intercambia información entre el bot y el canal (usuario) pasando un objeto [Activity][Activity]. El tipo de actividad más común es **message**, pero hay otros tipos de actividades que se pueden usar para comunicar distintos tipos de información a un bot o canal. Para obtener información detallada sobre las actividades del servicio Bot Connector, vea [Introducción a las actividades](bot-framework-rest-connector-activities.md).

## <a name="bot-state-service"></a>Servicio Bot State

El servicio Bot State permite que el bot almacene y recupere datos de estado asociados a un usuario, una conversación o un usuario específico en el contexto de una conversación específica. Usa los estándares del sector REST y JSON a través de HTTPS y permite la autenticación con tokens de portador JWT. Cualquier dato que guarde con el servicio Bot State se almacenará en Azure y se cifrará en reposo.

El servicio Bot State solo es útil junto con el servicio Bot Connector. Es decir, solo puede utilizarse para almacenar datos de estado que están relacionados con las conversaciones que su bot mantiene mediante el servicio Bot Connector. Para más información sobre cómo usar el servicio Bot State, vea [Autenticación](bot-framework-rest-connector-authentication.md) y [Administración de datos de estado](bot-framework-rest-state.md).

## <a name="authentication"></a>Autenticación

Los servicios Bot Connector y Bot State habilitan la autenticación con tokens de portador de JWT. Para obtener información detallada sobre cómo autenticar las solicitudes salientes que envía el bot de Bot Framework, cómo autenticar las solicitudes entrantes que recibe su bot desde Bot Framework y mucho más, vea [Autenticación](bot-framework-rest-connector-authentication.md). 

## <a name="client-libraries"></a>Bibliotecas de clientes

Bot Framework proporciona bibliotecas cliente que se pueden usar para crear bots en C# o Node.js. 

- Para crear un bot con C#, use [Bot Framework SDK para C#](../dotnet/bot-builder-dotnet-overview.md). 
- Para crear un bot con Node.js, use [Bot Framework SDK para Node.js](../nodejs/index.md). 

Además de modelar los servicios Bot Connector y Bot State, cada Bot Framework SDK también proporciona un sistema eficaz para crear diálogos que contienen lógica de conversación, mensajes integrados para respuestas tan sencillas como Sí/No, cadenas, números y enumeraciones, compatibilidad integrada para marcos eficaces de inteligencia artificial, como <a href="https://www.luis.ai/" target="_blank">LUIS</a> y mucho más. 

> [!NOTE]
> Como alternativa al uso de los SDK para C# o Node.js, puede generar sus propia biblioteca cliente en el lenguaje que quiera mediante el <a href="https://raw.githubusercontent.com/Microsoft/BotBuilder/master/CSharp/Library/Microsoft.Bot.Connector.Shared/Swagger/ConnectorAPI.json" target="_blank">archivo Swagger de Bot Connector</a> y el <a href="https://raw.githubusercontent.com/Microsoft/BotBuilder/master/CSharp/Library/Microsoft.Bot.Connector.Shared/Swagger/StateAPI.json" target="_blank">archivo Swagger de Bot State</a>.

## <a name="additional-resources"></a>Recursos adicionales

Para obtener más información sobre cómo crear bots con los servicios Bot Connector y Bot State, consulte los artículos de esta sección, que comienzan con [Autenticación](bot-framework-rest-connector-authentication.md). Si tiene problemas o sugerencias sobre los servicios Bot Connector y Bot State, consulte el [soporte técnico](../bot-service-resources-links-help.md) para obtener una lista de los recursos disponibles. 

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
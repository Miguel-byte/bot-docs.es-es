---
title: Novedades | Microsoft Docs
description: Conozca las novedades de Bot Framework.
keywords: bot framework, azure bot service
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 07/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1bbea2f12af976a9d967e7c62baf416b8938f8aa
ms.sourcegitcommit: b053c0ca7f2e9e60679f7e82e583c57ae83fcb50
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/19/2019
ms.locfileid: "68336740"
---
# <a name="whats-new-in-bot-framework-july-2019"></a>Novedades de Bot Framework (julio de 2019)

[!INCLUDE[applies-to](includes/applies-to.md)]

El SDK de Bot Framework v4 es un [SDK de código abierto][1a] que permite a los desarrolladores modelar y crear conversaciones sofisticadas con su lenguaje de programación favorito.

En este artículo se resumen las nuevas características y principales mejoras en Bot Framework y Azure Bot Service.

|   | C#  | JS  | Python |   
|---|:---:|:---:|:------:|
|SDK |[4.5][1] | [4.5][2] | [4.4.0b2 (versión preliminar)][3] | 
|Docs | [docs][5] |[docs][5] |  | |
|Ejemplos |[.NET Core][6], [WebAPI][10] |[Node.js][7] , [TypeScript][8], [es6][9]  | [Python][111] | | 

[1a]:https://github.com/microsoft/botframework-sdk/#readme
[1]:https://github.com/Microsoft/botbuilder-dotnet/#packages
[2]:https://github.com/Microsoft/botbuilder-js#packages
[3]:https://github.com/Microsoft/botbuilder-python#packages
[5]:https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0
[6]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore
[7]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs
[8]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_typescript
[9]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_es6
[10]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi
[111]:https://github.com/Microsoft/botbuilder-python/tree/master/samples


## <a name="bot-framework-channels"></a>Canales de Bot Framework
- [Direct Line Speech (versión preliminar pública)](https://aka.ms/streaming-extensions) | [documentación](https://docs.microsoft.com/azure/bot-service/directline-speech-bot?view=azure-bot-service-4.0): Bot Framework y los servicios de voz de Microsoft proporcionan un canal que permite la transmisión bidireccional de voz y texto desde el cliente a la aplicación de bot mediante WebSockets.  

- [Extensión de App Service para Direct Line (versión preliminar)](https://portal.azure.com) | [documentación](https://aka.ms/directline-ase): Una versión de Direct Line que permite a los clientes conectarse directamente a los bots mediante Direct Line API. Esto ofrece numerosas ventajas, como un mayor rendimiento y más aislamiento. La extensión de App Service para Direct Line está disponible en todos los servicios de Azure App Service, incluidos los hospedados en un entorno de Azure App Service Environment. Un entorno de Azure App Service Environment proporciona aislamiento y es ideal para trabajar en una red virtual. Una red virtual le permite crear su propio espacio privado en Azure y es fundamental para su red en la nube porque ofrece aislamiento, segmentación y otras importantes ventajas. 

## <a name="bot-framework-sdk"></a>SDK Bot Framework
- [Diálogo adaptable (SDK v4.6 versión preliminar)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog#readme) | [documentación](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/docs) | [Ejemplos de C#](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/csharp_dotnetcore): Los diálogo adaptables ahora permiten a los desarrolladores actualizar dinámicamente el flujo de conversación en función del contexto y los eventos. Esto es especialmente útil cuando se trabaja con cambios de contexto de la conversación e interrupciones en medio de una conversación. 
  
- [SDK Bot Framework para Python (versión preliminar 2)](https://github.com/microsoft/botbuilder-python) | [ejemplos](https://github.com/Microsoft/botbuilder-python/tree/master/samples): El SDK para Python ahora es compatible con OAuth, Promtps y CosmosDB, e incluye toda la funcionalidad principal del SDK 4.5, además de ejemplos que le ayudarán a conocer las nuevas características del SDK.

## <a name="bot-framework-testing"></a>Pruebas de Bot Framework
- [Prueba unitaria](http://aka.ms/bot-test-package) | [documentación](https://aka.ms/testing-framework) | [Ejemplo de C#](https://aka.ms/corebot-test) | [Ejemplo de JS](https://aka.ms/js-core-test-sample): Atendiendo a la petición de clientes y desarrolladores de herramientas de prueba mejoradas, la versión de julio del SDK presenta una nueva funcionalidad de pruebas unitarias. El paquete Microsoft.bot.Builder.testing simplifica la ejecución de pruebas unitarias de diálogos en el bot. 

- [Pruebas de canal](https://github.com/Microsoft/BotFramework-Emulator/releases): El inspector de bots se incorporó en Microsoft Build 2019 y es una característica nueva de Bot Framework Emulator que permite depurar y probar bots en canales como Microsoft Teams, Slack, Cortana, etc. Cuando use el bot en canales específicos, los mensajes se reflejarán en Bot Framework Emulator, donde podrá inspeccionar los datos de los mensajes que el bot recibió. Además, también se muestra una instantánea del estado de la memoria del bot para cualquier turno determinado entre el canal y el bot.

## <a name="web-chat"></a>Chat en web
- Atendiendo a la petición de los clientes de empresa, hemos agregado un [ejemplo de chat web](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/19.a.single-sign-on-for-enterprise-apps#single-sign-on-demo-for-enterprise-apps-using-oauth) que muestra cómo autorizar a un usuario para acceder a los recursos de una aplicación empresarial con un bot. Se usan dos tipos de recursos para mostrar la interoperabilidad de OAuth con Microsoft Graph y la API de GitHub.

## <a name="solutions"></a>Soluciones
- [Acelerador de soluciones de asistente virtual](https://github.com/Microsoft/botframework-solutions#readme): proporciona un conjunto de plantillas, aceleradores de soluciones y aptitudes que ayudan a crear experiencias de conversación sofisticadas. El nuevo cliente de aplicación Android para asistente virtual, que se integra con Direct-Line Speech y Virtual Assistant, muestra cómo puede interactuar un cliente de dispositivo con el asistente virtual y representar tarjetas adaptables. Las actualizaciones también incluyen compatibilidad con Direct-Line Speech y Microsoft Teams.
  
- [Dynamics 365 Virtual Agent for Customer Service (versión preliminar pública)](https://dynamics.microsoft.com/en-us/ai/virtual-agent-for-customer-service/): con la versión preliminar pública, puede proporcionar un servicio de atención al cliente excepcional con agentes virtuales inteligentes y adaptables. Los expertos del servicio de atención al cliente pueden crear y mejorar fácilmente los bots con información controlada por inteligencia artificial.
  
- [Chat para Dynamics 365](https://www.powerobjects.com/powerpacks/powerchat/): el chat para Dynamics 365 ofrece varias funcionalidades para que los agentes de soporte técnico y los usuarios finales puedan interactuar eficazmente y sigan siendo muy productivos. Charle en directo y realice un seguimiento de las conversaciones de los visitantes de su sitio web en Microsoft Dynamics 365.

## <a name="additional-information"></a>Información adicional
- Puede ver los anuncios anteriores [aquí](what-is-new-archive.md).

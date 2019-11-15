---
title: Novedades | Microsoft Docs
description: Conozca las novedades de Bot Framework.
keywords: bot framework, azure bot service
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.date: 11/04/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 8cb03814a9fbb7fdbbfb457400eca3cf6a67ec17
ms.sourcegitcommit: 490810d278d1c8207330b132f28a5eaf2b37bd07
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/05/2019
ms.locfileid: "73592238"
---
# <a name="whats-new-november-2019-ignite"></a>Novedades de noviembre de 2019 (Ignite)

[!INCLUDE[applies-to](includes/applies-to.md)]

El SDK de Bot Framework v4 es un [SDK de código abierto](https://github.com/microsoft/botframework-sdk/#readme) que permite a los desarrolladores modelar y crear conversaciones sofisticadas con su lenguaje de programación favorito.

En este artículo se resumen las nuevas características y principales mejoras en Bot Framework y Azure Bot Service.


|   | C#  | JS  | Python |  Java | 
|---|:---:|:---:|:------:|:-----:|
|Release |[4.6 Disponibilidad general][1] | [4.6 Disponibilidad general][2] | [Beta 4][3] | [Versión preliminar 3][3a]|
|Docs | [docs][5] |[docs][5] |  | |
|Ejemplos |[.NET Core][6], [WebAPI][10] |[Node.js][7], [TypeScript][8], [es6][9]  | | | 


[1]:https://github.com/Microsoft/botbuilder-dotnet/#packages
[2]:https://github.com/Microsoft/botbuilder-js#packages
[3]:https://github.com/Microsoft/botbuilder-python#packages
[3a]:https://github.com/Microsoft/botbuilder-java#packages
[5]:https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0
[6]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore
[7]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs
[8]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/typescript_nodejs
[9]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_es6
[10]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi

#### <a name="bot-framework-sdk-for-microsoft-teams-ga"></a>Bot Framework SDK para Microsoft Teams (Disponibilidad general)
La versión 4.6 de Bot Framework SDK integra totalmente la compatibilidad con la creación de bots de Teams que permiten a los usuarios utilizarlos en conversaciones de chat de canal o de grupo. Al agregar un bot a un equipo o a un chat, todos los usuarios de la conversación pueden aprovechar las ventajas de la funcionalidad del bot directamente en la conversación.  [Docs](https://docs.microsoft.com/azure/bot-service/bot-builder-basics-teams)

#### <a name="bot-framework-for-power-virtual-agent-preview"></a>Bot Framework para Power Virtual Agents (versión preliminar)

Power Virtual Agents está diseñado para permitir que los usuarios empresariales creen bots dentro de una experiencia SaaS de creación de bots basada en la interfaz de usuario, sin tener que programar o administrar servicios de inteligencia artificial específicos. Power Virtual Agents se puede extender con Microsoft Bot Framework, lo que permite a los desarrolladores y usuarios empresariales colaborar en la creación de bots para sus organizaciones. [Docs](https://docs.microsoft.com/dynamics365/ai/customer-service-virtual-agent/overview)


#### <a name="bot-framework-sdk-for-skills-preview"></a>Bot Framework SDK para Aptitudes (versión preliminar)

- **Aptitudes para bots**: Cree aptitudes de conversación reutilizables para agregar funcionalidad a un bot. Aproveche las aptitudes predefinidas, como el calendario, el correo electrónico, la tarea, el punto de interés, los conocimientos sobre automóviles, el tiempo y las noticias. Las aptitudes incluyen modelos de lenguajes, diálogos, preguntas y respuestas y código de integración que se entregan para personalizar y ampliar según sea necesario. [Docs](https://microsoft.github.io/botframework-solutions/overview/skills/)

- **Aptitudes para Power Virtual Agents: próximamente**: En el caso de los bots creados con Power Virtual Agents, puede crear nuevas aptitudes para estos bots con Bot Framework y Azure Cognitive Services sin necesidad de crear un nuevo bot desde cero. 

#### <a name="adaptive-dialogs-preview"></a>Diálogos adaptables (versión preliminar)
Los diálogos adaptables permiten a los desarrolladores actualizar dinámicamente el flujo de la conversación en función del contexto y los eventos. Esto es especialmente útil cuando se trabaja con cambios de contexto de la conversación e interrupciones en medio de una conversación. [[Docs][48] | [Ejemplos de C#][49]] 

#### <a name="language-generation-preview"></a>Generación de lenguaje (versión preliminar)
La generación de lenguaje permite a los desarrolladores separar la lógica que se usa para generar respuestas del bot, incluida la capacidad de definir varias variaciones en una frase, ejecutar expresiones simples basadas en el contexto y hacer referencia a la memoria de la conversación. [[Documentación][44] | [Ejemplos de C#][45]]

#### <a name="common-expression-language-preview"></a>Lenguaje de expresiones comunes (versión preliminar)
El lenguaje de expresiones comunes permite evaluar el resultado de una lógica basada en condiciones en tiempo de ejecución. El lenguaje de expresiones comunes se puede usar en Bot Framework SDK y en los componentes de IA de conversación, como los diálogos adaptables y la generación de lenguaje. [[Docs][40] | [API][41]]


[40]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/common-expression-language#readme
[41]:https://github.com/Microsoft/BotBuilder-Samples/blob/master/experimental/common-expression-language/api-reference.md
[43]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation#readme
[44]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation/docs
[45]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation/csharp_dotnetcore
[46]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation/javascript_nodejs/13.core-bot
[47]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog#readme
[48]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/docs
[49]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/csharp_dotnetcore
[50]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/declarative

## <a name="additional-information"></a>Información adicional
- Puede ver los anuncios anteriores [aquí](what-is-new-archive.md).

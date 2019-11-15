---
title: Novedades | Microsoft Docs
description: Conozca las novedades de Bot Framework.
keywords: bot framework, azure bot service
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.date: 11/01/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a57e5bd2b17cb2ac7553e71ac66f2bdfe97f3b35
ms.sourcegitcommit: 312a4593177840433dfee405335100ce59aac347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/12/2019
ms.locfileid: "73933748"
---
# <a name="whats-new-july-2019"></a>Novedades (julio de 2019)

[!INCLUDE[applies-to](includes/applies-to.md)]

El SDK de Bot Framework v4 es un [SDK de código abierto][1a] que permite a los desarrolladores modelar y crear conversaciones sofisticadas con su lenguaje de programación favorito.

En este artículo se resumen las nuevas características y principales mejoras en Bot Framework y Azure Bot Service.

|   | C#  | JS  | Python |   
|---|:---:|:---:|:------:|
|SDK |[4.5][1] | [4.5][2] | [4.4.0b2 (versión preliminar)][3] | 
|Docs | [docs][5] |[docs][5] |  | |
|Ejemplos |[.NET Core][6], [WebAPI][10] |[Node.js][7], [TypeScript][8], [es6][9]  | [Python][111] | | 

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
- [Documentos](https://aka.ms/testing-framework) | Paquetes de pruebas unitarias ([C#](https://aka.ms/nuget-botbuilder-testing)/ [JavaScript](https://aka.ms/npm-botbuilder-testing)) | [Ejemplo de C#](https://aka.ms/cs-core-test-sample) | [Ejemplo de JavaScript](https://aka.ms/js-core-test-sample): Atendiendo a la petición de clientes y desarrolladores de herramientas de prueba mejoradas, la versión de julio del SDK presenta una nueva funcionalidad de pruebas unitarias. El paquete Microsoft.bot.Builder.testing simplifica la ejecución de pruebas unitarias de diálogos en el bot.  

- [Pruebas de canal](https://github.com/Microsoft/BotFramework-Emulator/releases) | [(documentación)](https://aka.ms/channel-testing): 

El inspector de bots se incorporó en Microsoft Build 2019 y es una característica nueva de Bot Framework Emulator que permite depurar y probar bots en canales como Microsoft Teams, Slack, Cortana, etc. Cuando use el bot en canales específicos, los mensajes se reflejarán en Bot Framework Emulator, donde podrá inspeccionar los datos de los mensajes que el bot recibió. Además, también se muestra una instantánea del estado de la memoria del bot para cualquier turno determinado entre el canal y el bot.

## <a name="web-chat"></a>Chat en web
- Atendiendo a la petición de los clientes de empresa, hemos agregado un [ejemplo de chat web](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/19.a.single-sign-on-for-enterprise-apps#single-sign-on-demo-for-enterprise-apps-using-oauth) que muestra cómo autorizar a un usuario para acceder a los recursos de una aplicación empresarial con un bot. Se usan dos tipos de recursos para mostrar la interoperabilidad de OAuth con Microsoft Graph y la API de GitHub.

## <a name="solutions"></a>Soluciones
- [Acelerador de soluciones de asistente virtual](https://github.com/Microsoft/botframework-solutions#readme): proporciona un conjunto de plantillas, aceleradores de soluciones y aptitudes que ayudan a crear experiencias de conversación sofisticadas. El nuevo cliente de aplicación Android para asistente virtual, que se integra con Direct-Line Speech y Virtual Assistant, muestra cómo puede interactuar un cliente de dispositivo con el asistente virtual y representar tarjetas adaptables. Las actualizaciones también incluyen compatibilidad con Direct-Line Speech y Microsoft Teams.
  
- [Dynamics 365 Virtual Agent for Customer Service (versión preliminar pública)](https://dynamics.microsoft.com/en-us/ai/virtual-agent-for-customer-service/): con la versión preliminar pública, puede proporcionar un servicio de atención al cliente excepcional con agentes virtuales inteligentes y adaptables. Los expertos del servicio de atención al cliente pueden crear y mejorar fácilmente los bots con información controlada por inteligencia artificial.
  
- [Chat para Dynamics 365](https://www.powerobjects.com/powerpacks/powerchat/): el chat para Dynamics 365 ofrece varias funcionalidades para que los agentes de soporte técnico y los usuarios finales puedan interactuar eficazmente y sigan siendo muy productivos. Charle en directo y realice un seguimiento de las conversaciones de los visitantes de su sitio web en Microsoft Dynamics 365.

## <a name="whats-new-may-2019"></a>Novedades (mayo de 2019)

|   | C#  | JS  | Python |  Java | 
|---|:---:|:---:|:------:|:-----:|
|SDK |[4.4.3][1] | [4.4.0][2] | [4.4.0b1 (versión preliminar)][3] | [4.0.0a6 (versión preliminar)][3a]|
|Docs | [docs][5] |[docs][5] |  | |
|Ejemplos |[.NET Core][6], [WebAPI][10] |[Node.js][7], [TypeScript][8], [es6][9]  | [Python][111] | | 

[1a]:https://github.com/microsoft/botframework-sdk/#readme
[1]:https://github.com/Microsoft/botbuilder-dotnet/#packages
[2]:https://github.com/Microsoft/botbuilder-js#packages
[3]:https://github.com/Microsoft/botbuilder-python#packages
[3a]:https://github.com/Microsoft/botbuilder-java#packages
[4]:https://github.com/Microsoft/botbuilder-java#packages
[5]:https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0
[6]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore
[7]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs
[8]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_typescript
[9]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_es6
[10]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi
[111]:https://github.com/Microsoft/botbuilder-python/tree/master/samples

<a name="V4-whats-new"></a>
## <a name="bot-framework-sdk-new-in-preview"></a>SDK Bot Framework (Nuevo. En versión preliminar)

- [Diálogo adaptable][47] | [docs][48] | [Ejemplos de C#][49]: Los diálogos adaptables permiten a los desarrolladores crear conversaciones que se pueden cambiar dinámicamente a medida que la conversación progresa.  Tradicionalmente, los desarrolladores han trazado un mapa de todo el flujo de una conversación al principio, lo que limita la flexibilidad de la conversación.  Los diálogos adaptables les permiten ser más flexibles, responder a los cambios de contexto e insertar nuevos pasos o diálogos secundarios completos en la conversación a medida que esta avanza. 

- [Generación de lenguaje][43] | [docs][44] | [Ejemplos de C#][45]: generación del lenguaje; que permite al desarrollador extraer las cadenas insertadas de los archivos de código y recursos, y administrarlas mediante un entorno de ejecución de generación del lenguaje y un formato de archivo.  La generación del lenguaje permite a los clientes definir múltiples variaciones de una frase, ejecutar expresiones simples basadas en el contexto, referirse a la memoria conversacional, y con el tiempo permitirá traer funcionalidades adicionales que ofrecerán una experiencia conversacional más natural.

- [Lenguaje de expresiones comunes][40] | [api][41]: Tanto los diálogos adaptables como la generación del lenguaje dependen y utilizan un lenguaje de expresiones comunes para potenciar las conversaciones de los bot.

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

## <a name="botkit"></a>Botkit
[Botkit][100] es una herramienta de desarrollo y un SDK para crear bots de chat, aplicaciones e integraciones personalizadas para las principales plataformas de mensajería. Los bots de Botkit utilizan `hear()` para escuchar a los desencadenadores, `ask()` para preguntar y `say()` para responder. Los desarrolladores pueden usar esta sintaxis para construir diálogos, ahora compatibles con la última versión del SDK Bot Framework. 

Además, Botkit trae consigo seis adaptadores de plataforma que permiten a las aplicaciones de Javascript comunicarse directamente con las plataformas de mensajería: [Slack][102], [Webex Teams][103], [Google Hangouts][104], [Facebook Messenger][105], [Twilio][106] y [Web chat][107].

Botkit forma parte de Microsoft Bot Framework y está publicado bajo la [licencia de código abierto de MIT][101].

[100]:https://github.com/howdyai/botkit#readme
[101]:https://github.com/howdyai/botkit/blob/master/LICENSE.md
[102]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-slack#readme
[103]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-webex#readme
[104]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-hangouts#readme
[105]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-facebook#readme
[106]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-twilio-sms#readme
[107]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-web#readme

## <a name="bot-framework-solutions-new-in-preview"></a>Soluciones de Bot Framework (Nuevo. En versión preliminar)

En el [repositorio de soluciones de Bot Framework](https://github.com/Microsoft/AI#readme) se pueden encontrar un conjunto de plantillas, aceleradores de soluciones y aptitudes para ayudar a crear experiencias conversacionales avanzadas, similares a las de un asistente.

| NOMBRE | DESCRIPCIÓN |  
|:------------|:------------| 
|[**Virtual Assistant**](https://github.com/Microsoft/AI/tree/master/docs#virtual-assistant) | Los clientes tienen una gran necesidad de ofrecer un asistente de conversación adaptado a su marca, personalizado para sus clientes y disponible en una amplia variedad de dispositivos y lienzos. <br/><br/> La plantilla empresarial simplifica enormemente la creación de un nuevo proyecto de bots que incluye: intenciones básicos de conversación, integración con Dispatch, QnA Maker, Application Insights y una implementación automatizada.|
|[**Aptitudes**](https://github.com/Microsoft/AI/blob/master/docs/overview/skills.md)| Los desarrolladores pueden componer experiencias de conversación mediante la combinación de funcionalidades de conversación reutilizables, conocidas como aptitudes. Las aptitudes son en sí mismas bots, invocadas de forma remota y está disponible una plantilla de desarrollador de aptitudes (.NET, TS) para facilitar la creación de nuevas aptitudes. 
|[**Análisis**](https://github.com/Microsoft/AI/blob/master/docs/readme.md#analytics)| Obtenga información clave sobre el estado y el comportamiento del bot con las soluciones de análisis de inteligencia artificial conversacional. Revise la telemetría disponible, ejemplos de consultas de Application Insights y paneles de Power BI para comprender la amplitud de las conversaciones del bot con los usuarios. |

## <a name="azure-bot-service"></a>Azure Bot Service
Azure Bot Service le permite hospedar bots inteligentes de nivel empresarial con total propiedad y control de sus datos. Los desarrolladores pueden registrar y conectar los bots a usuarios de Skype, Microsoft Teams, Cortana, Web Chat y mucho más. [Azure][27]  |  [docs][28] | [conexión a los canales][29] 

* **Cliente de Direct Line para JS**: Si desea utilizar el canal de Direct Line en Azure Bot Service y no usa el cliente de Web Chat, el cliente Direct Line para JS puede emplearse en la aplicación personalizada. Vaya a [Github][30] para obtener más información.

<a name="ABS-whats-new"></a>

* **Nuevo. Canal Direct Line Speech**: Estamos uniendo Bot Framework y los servicios de voz de Microsoft para proporcionar un canal que permite la transmisión bidireccional de voz y texto desde el cliente a la aplicación bot.  Para más información, consulte cómo agregar [el canal de voz al bot](https://docs.microsoft.com/azure/bot-service/directline-speech-bot?view=azure-bot-service-4.0).

[27]:https://azure.microsoft.com/services/bot-service/
[28]:https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0
[29]:https://docs.microsoft.com/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0
[30]:https://github.com/Microsoft/BotFramework-DirectLineJS/blob/master/README.md


## <a name="bot-framework-emulator"></a>Bot Framework Emulator
[Bot Framework Emulator][60] es una aplicación de escritorio multiplataforma que permite que los desarrolladores de bots prueben y depuren los bots mediante el SDK de Bot Framework. Puede usar Bot Framework Emulator para probar los bots que se ejecutan localmente en la máquina o para conectarse a los bots que se ejecutan de forma remota.

- [Descargar versión más reciente][61] | [Docs][62]

<a name="Emulator-whats-new"></a>
### <a name="bot-inspector-new-in-preview"></a>Bot Inspector (Nuevo. En versión preliminar)

Bot Framework Emulator ha publicado una versión beta de la nueva característica Bot Inspector. Proporciona una manera de depurar y probar los bots del SDK Bot Framework v4 en canales como Microsoft Teams, Slack, Cortana, Facebook Messenger o Skype, entre otros. Cuando tenga la conversación, los mensajes se reflejarán en Bot Framework Emulator, donde podrá inspeccionar los datos del mensaje que recibió el bot. Además, también se muestra una instantánea del estado del bot para cualquier turno determinado entre el canal y el bot. Más información sobre [Bot Inspector](https://github.com/Microsoft/BotFramework-Emulator/blob/master/content/CHANNELS.md).

[60]:https://github.com/Microsoft/BotFramework-Emulator#readme
[61]:https://github.com/Microsoft/BotFramework-Emulator/releases/latest
[62]:https://docs.microsoft.com/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0


## <a name="related-services"></a>Servicios relacionados

### <a name="language-understanding"></a>Language Understanding 
Un servicio basado en el aprendizaje automático para crear experiencias en lenguaje natural. Cree rápidamente modelos personalizados preparados para la empresa que puedan mejorar constantemente. [Language Understanding Service (LUIS)][30] permite que la aplicación entienda lo que una persona quiere en sus propias palabras.

<a name="LUIS-whats-new"></a>

- **Nuevo. Roles, entidades externas y entidades dinámicas**: LUIS ha agregado varias características que permiten a los desarrolladores extraer información más detallada del texto, por lo que los usuarios pueden ahora crear soluciones más inteligentes con menos esfuerzo. LUIS también extendió los roles a todos los tipos de entidades, lo que permite clasificar las mismas entidades con diferentes subtipos en función del contexto. Los desarrolladores ahora tienen un control más detallado de lo que pueden hacer con LUIS, incluida la capacidad de identificar y actualizar modelos en el entorno de ejecución mediante listas dinámicas y entidades externas. Las listas dinámicas se utilizan para agregar entidades a la lista en el momento de la predicción, lo que permite que la información específica del usuario coincida exactamente. Se utilizan extractores de entidades complementarios independientes con entidades externas, y esa información puede anexarse a LUIS como señales seguras para otros modelos.

- **Nuevo. Panel de análisis**: LUIS está iniciando un panel de análisis integral más detallado y visualmente rico. Su diseño fácil de usar destaca los problemas comunes a los que se enfrentan la mayoría de los usuarios al diseñar aplicaciones; para ello, proporciona explicaciones sencillas sobre cómo resolverlos para ayudar a los usuarios a comprender mejor la calidad de sus modelos, los posibles problemas de datos y la guía para adoptar los procedimientos recomendados.

[Docs][31] | [Incorporación de comprensión del lenguaje al bot][32] 

[18]:https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#readme
[19]:https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#readme
[30]:https://www.luis.ai
[31]:https://docs.microsoft.com/azure/cognitive-services/LUIS/Home
[32]:https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-luis?view=azure-bot-service-4.0&branch=pr-en-us-1325&tabs=csharp

### <a name="qna-maker"></a>QnA Maker
[QnA Maker][33] es un servicio de API basado en la nube que crea una capa de conversación, de preguntas y respuestas con los datos. Con QnA Maker se puede crear, entrenar y publicar un bot de preguntas y respuestas simple basado en direcciones URL de preguntas más frecuentes, documentos estructurados, manuales del producto o contenido editorial en cuestión de minutos.

<a name="QnA-whats-new"></a>

- **Nuevo. Canalización de extracción**: Ahora puede extraer información jerárquica de las direcciones URL, archivos y Sharepoint.
- **Nuevo. Inteligencia**: Modelos de clasificación contextuales, sugerencias de aprendizaje activo
- **Nuevo. Conversación**: Conversaciones de varios turnos en QnA Maker.

[Docs][34]  | [agregar qnamaker al bot][35] 

[33]:https://www.qnamaker.ai/
[34]:https://aka.ms/what-is-qnamaker
[35]:https://docs.microsoft.com/azure/bot-service/bot-builder-howto-qna?view=azure-bot-service-4.0&branch=pr-en-us-1325&tabs=cs

### <a name="speech-services"></a>Speech Services
[Los servicios de voz](https://docs.microsoft.com/azure/cognitive-services/speech-service/) permiten el audio en texto, traducir la voz y convertir el texto en voz con los servicios de voz unificados Con los servicios de voz, puede integrar el habla en su bot, crear palabras de activación personalizadas y crear en varios idiomas.

### <a name="adaptive-cards"></a>Tarjetas adaptables
Las [tarjetas adaptables](https://adaptivecards.io) son un estándar abierto para que los desarrolladores intercambien el contenido de las tarjetas de una manera común y coherente, y las utilizan los desarrolladores de Bot Framework para crear grandes experiencias conversacionales entre canales.

## <a name="additional-information"></a>Información adicional
- Visite la [página de GitHub](https://github.com/Microsoft/botframework/blob/master/whats-new.md#whats-new) para obtener más información.

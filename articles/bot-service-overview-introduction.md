---
title: Introducción a Azure Bot Service | Microsoft Docs
description: Obtenga información acerca de Bot Service, un servicio para compilar, conectar, probar, implementar, supervisar y administrar bots.
keywords: información general, introducción, SDK, esquema
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 10/08/2018
ms.openlocfilehash: 3ca80439a44ac7e715d19f8e47683ac9b5a5721a
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998882"
---
::: moniker range="azure-bot-service-3.0"

# <a name="about-azure-bot-service"></a>Acerca de Azure Bot Service

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Azure Bot Service proporciona herramientas para crear, probar, implementar y administrar bots inteligentes desde un único lugar. Mediante la plataforma modular y extensible que proporciona el SDK, los desarrolladores pueden aprovechar las plantillas para crear bots que proporcionen voz, reconocimiento del lenguaje, preguntas y respuestas, etc.  

## <a name="what-is-a-bot"></a>¿Qué es un bot?
Un bot es una aplicación con la que los usuarios interactúan de forma conversacional mediante texto, gráficos (tarjetas) o voz. Puede ser un diálogo sencillo de pregunta y respuesta, o un bot sofisticado que permita a los usuarios interactuar con los servicios de una forma inteligente mediante la coincidencia de patrones, el seguimiento de estados y técnicas de inteligencia artificial bien integradas con los servicios empresariales existentes. 

## <a name="building-a-bot"></a>Compilación de un bot 
Puede decidir usar su entorno de desarrollo o sus herramientas de línea de comandos favoritas para crear el bot en C# o Node.js. Se proporcionan herramientas para las distintas etapas de desarrollo de los bots que puede usar para compilar el bot y empezar a trabajar.    

![Información general sobre el bot](media/bot-service-overview.png) 

## <a name="plan"></a>Plan 
Antes de escribir código, revise las [directrices de diseño](bot-service-design-principles.md)  del bot para seguir los procedimientos recomendados e identificar las necesidades del bot. Puede crear un bot simple o incluir funcionalidades más sofisticadas como voz, reconocimiento del lenguaje, QnA o la posibilidad de extraer conocimientos a partir de distintos orígenes y proporcionar respuestas inteligentes.  

> [!TIP]
> Cree una cuenta de [Azure](https://portal.azure.com). 

## <a name="build-your-bot"></a>Compilación del bot 
El bot es un servicio web que implementa una interfaz de conversación y que se comunica con Bot Service. Puede crear esta solución en una amplia variedad de entornos y lenguajes, y le ofrecemos herramientas sencillas para empezar en Visual Studio o Yeoman, o directamente en Azure Portal. Mire a continuación algunas de las herramientas y servicios que puede usar.

> [!TIP]
> Cree un bot mediante [Azure Portal](bot-service-quickstart.md). Si es necesario, agregue componentes como: 
> - Language Understanding [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/home). 
> - Base de conocimiento de [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/home) para responder a las preguntas de los usuarios.  

## <a name="test-your-bot"></a>Prueba del bot 
Los bots son aplicaciones complejas con una gran cantidad de elementos diferentes que funcionan conjuntamente. Como cualquier otra aplicación compleja, esto puede provocar algunos errores interesantes o que el bot se comporte de forma diferente a la esperada. Antes de publicarlo, pruebe el bot.

> [!TIP] 
> - Pruebe el bot en[Chat en web](bot-service-manage-test-webchat.md) o pruebe el bot localmente con el [Emulador](bot-service-debug-emulator.md)

## <a name="publish"></a>Publicar 
Cuando esté listo, publique su bot en Azure o en su propio servicio web o centro de datos. Puede configurar la implementación continua que le permite desarrollar el bot de forma local y es útil si el bot se inserta en un control de código fuente como GitHub o Visual Studio Team Services. A medida que inserta los cambios en el repositorio de origen, los cambios se implementarán automáticamente en Azure.

> [!Tip]
> - [Descarga y reimplementación del código en Azure](bot-service-build-download-source-code.md)

## <a name="connect"></a>Conectar          
Conecte el bot a canales como Facebook, Messenger, Kik, Skype, Slack, Microsoft Teams, Telegram, mensajes de texto o SMS, Twilio, Cortana y Skype para aumentar las interacciones y llegar a más clientes.  
  
> [!TIP]
> - [Elija los canales que se van a agregar](bot-service-manage-channels.md)


## <a name="evaluate"></a>Evaluate 
Use los datos recopilados en Azure Portal para identificar oportunidades para mejorar las funcionalidades y el rendimiento de su bot. Puede obtener nivel de servicio y datos de instrumentación como tráfico, latencia e integraciones. Analytics proporciona también informes de nivel de conversación relativos a los datos del usuario, los mensajes y los canales.

> [!Tip]
> - [Recopilación de análisis](bot-service-manage-analytics.md) 


::: moniker-end

::: moniker range="azure-bot-service-4.0"

# <a name="about-azure-bot-service"></a>Acerca de Azure Bot Service

[!INCLUDE [pre-release-label](includes/pre-release-label.md)]

Azure Bot Service proporciona herramientas para crear, probar, implementar y administrar bots inteligentes desde un único lugar. Mediante el uso del marco modular y ampliable que proporciona el SDK, las herramientas, las plantillas y los desarrolladores de servicios de IA pueden crear bots que usen la voz, reconozcan el lenguaje natural, controlen las preguntas y respuestas, etc.

## <a name="what-is-a-bot"></a>¿Qué es un bot?
Los bots proporcionan una experiencia que hace que parezca menos que se usa un equipo y más que se trata con una persona (o al menos un robot inteligente). Se pueden usar para desplazar las tareas simples y repetitivas, como reservar una mesa en un restaurante o recopilar información de un perfil, a sistemas automatizados que puede que no requieran intervención humana directa. Los usuarios conversan con los bot mediante texto, tarjetas interactivas y la voz. Una interacción con un bot puede ser tanto una pregunta y una respuesta rápidas como una conversación sofisticada que proporciona acceso a servicios de forma inteligente.

Los bots se parecen mucho a las aplicaciones web modernas, ya que residen en Internet y usan las API para enviar y recibir mensajes. El contenido de un bot varía considerablemente en función de su tipo. El software para bot moderno se basa en una pila de tecnología y herramientas que proporcionan experiencias cada vez más complejas en una amplia variedad de plataformas. Sin embargo, un bot simple simplemente puede recibir un mensaje y devolvérselo al usuario con muy poco código implicado. 

Los bots puede hacer lo mismo que otros tipos de software (leer y escribir archivos, usar bases de datos y API, y realizar las tareas de cálculo habituales). Lo que hace que los bots sean únicos es su uso de mecanismos que generalmente se reservan para la comunicación entre humanos. 

Los bots suelen constar de los siguientes componentes:
* Un servidor web, que en la mayoría de los casos está disponible en la red pública de internet
* El SDK de Bot Builder y las herramientas de Bot Builder, que proporcionan una interfaz para el desarrollo de bots
* Azure Cognitive Services 
* Azure Storage

## <a name="building-a-bot"></a>Compilación de un bot 

Azure Bot Service ofrece un conjunto integrado de herramientas y servicios que facilita este proceso. Elija el entorno de desarrollo o las herramientas de línea de comandos que prefiera para crear el bot en C#, JavaScript o Typescript (pronto se podrá crear también en Java y Python) Se proporcionan herramientas para las distintas etapas de desarrollo de los bots que puede usar para compilar el bot y empezar a trabajar.

![Información general sobre el bot](media/bot-service-overview.png) 

### <a name="plan"></a>Plan
Al igual que con cualquier otro tipo de software, tener un conocimiento exhaustivo de los objetivos, los procesos y las necesidades de los usuarios es importante para el proceso de creación de un bot adecuado. Antes de escribir código, revise las [directrices de diseño](bot-service-design-principles.md)  del bot para seguir los procedimientos recomendados e identificar las necesidades del bot. Puede crear un bot simple o incluir funcionalidades sofisticadas, como la voz, el reconocimiento de lenguaje natural y la respuesta a preguntas.

### <a name="build"></a>Compilación
Un bot es un servicio web que implementa una interfaz de conversación y que se comunica con el servicio Bot Framework para enviar y recibir mensajes y eventos. Se pueden crear bots en cualquier número de entornos y lenguajes. El desarrollo de un bot puede comenzar en [Azure Portal](bot-service-quickstart.md), o bien se pueden usar [[C#](dotnet/bot-builder-dotnet-sdk-quickstart.md) | [JavaScript](javascript/bot-builder-javascript-quickstart.md)] para el desarrollo local.

Como parte de Azure Bot Service, ofrecemos componentes adicionales que se pueden usar para ampliar la funcionalidad de los bots

| Característica | DESCRIPCIÓN | Vínculo |
| --- | --- | --- |
| Agregar procesamiento de lenguaje natural | Habilite el bot para reconocer el lenguaje natural, los errores de ortografía, usar la voz y reconocer la intención del usuario | Uso de [LUIS](~/v4sdk/bot-builder-howto-v4-luis.md) 
| Responder preguntas | Agregue una base de conocimiento para responder preguntas que los usuarios formulan de forma más natural y conversacional | Uso de [QnA Maker](~/v4sdk/bot-builder-howto-qna.md) 
| Administrar varios modelos | Si usa más de un modelo, por ejemplo, para LUIS y para QnA Maker, determine de manera inteligente cuándo usar cada uno de ellos durante la conversación del bot | Herramienta de [distribución](~/v4sdk/bot-builder-tutorial-dispatch.md)|
| Agregar tarjetas y botones | Mejore la experiencia del usuario con medios distintos al texto, como gráficos, menús y tarjetas | Procedimientos para [agregar tarjetas](v4sdk/bot-builder-howto-add-media-attachments.md) |

> [!NOTE]
> La tabla anterior no es una lista completa. Explore los artículos de la izquierda, comenzando por el [envío de mensajes](~/v4sdk/bot-builder-howto-send-messages.md), para obtener más funcionalidades de bot.

Además, se ofrecen herramientas de línea de comandos que ayudan a crear, administrar y probar los recursos de los bots. Estas herramientas permiten administrar el archivo de configuración de un bot, configurar aplicaciones de LUIS, crear una base de conocimientos de QnA, simular una conversación, etc. Puede encontrar más información en el archivo [Léame](https://aka.ms/botbuilder-tools-readme) de las herramientas de línea de comandos.

También tiene acceso a una variedad de [ejemplos](https://github.com/microsoft/botbuilder-samples) que presentan muchas de las funcionalidades disponibles a través del SDK. Dichos ejemplos son magníficos para los desarrolladores que buscan un punto de partida con muchas más características.

### <a name="test"></a>Prueba 
Los bots son aplicaciones complejas con una gran cantidad de elementos diferentes que funcionan conjuntamente. Como cualquier otra aplicación compleja, esto puede provocar algunos errores interesantes o que el bot se comporte de forma diferente a la esperada. Antes de publicarlo, pruebe el bot. Hay varias maneras de probar los bots antes de liberarlos para su uso:

- Pruebe el bot localmente con el [emulador](bot-service-debug-emulator.md). Bot Framework Emulator es una aplicación independiente que no solo proporciona una interfaz de chat, sino también herramientas de depuración y consulta que ayudan a conocer el funcionamiento del bot.  El emulador se puede ejecutar en un entorno local junto con la aplicación del bot en desarrollo. 
 
- Pruebe el bot en la [Web](bot-service-manage-test-webchat.md). Una vez configurado a través de Azure Portal, también se puede acceder al bot a través de una interfaz de chat web, que es una excelente manera de conceder acceso al bot tanto a los evaluadores como a otras personas que no tienen acceso directo al código de ejecución del bot.

### <a name="publish"></a>Publicar 
Cuando esté listo para que el bot esté disponible en Internet, publíquelo a [Azure](bot-builder-howto-deploy-azure.md) o en su propio servicio web o centro de datos. Tener una dirección en la red pública de Internet es el primer paso para que el bot cobre vida en su sitio web o en los canales de chat.

### <a name="connect"></a>Conectar          
Conecte el bot a canales como Facebook, Messenger, Kik, Skype, Slack, Microsoft Teams, Telegram, mensajes de texto o SMS, Twilio, Cortana y Skype. Bot Framework realiza la mayor parte del trabajo necesario para enviar y recibir mensajes de todas estas plataformas (la aplicación del bot recibe un flujo de mensajes normalizado, independientemente del número y tipo de canales al que esté conectada). Para obtener información acerca de cómo agregar canales, consulte el tema de los [canales](bot-service-manage-channels.md).

### <a name="evaluate"></a>Evaluate 
Use los datos recopilados en Azure Portal para identificar oportunidades para mejorar las funcionalidades y el rendimiento de su bot. Puede obtener nivel de servicio y datos de instrumentación como tráfico, latencia e integraciones. Analytics proporciona también informes de nivel de conversación relativos a los datos del usuario, los mensajes y los canales. Para más información, consulte el artículo acerca de [cómo recopilar análisis](bot-service-manage-analytics.md).


## <a name="next-steps"></a>Pasos siguientes
Consulte estos [casos prácticos](https://azure.microsoft.com/services/bot-service/) de bots o haga clic en el vínculo siguiente para crear un bot.
> [!div class="nextstepaction"]
> [Creación de un bot](bot-service-quickstart.md)

::: moniker-end

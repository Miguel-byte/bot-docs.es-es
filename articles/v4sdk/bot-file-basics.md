---
title: Administración de los recursos con un archivo .bot | Microsoft Docs
description: Describe el propósito y el uso del archivo de bot.
keywords: archivo de bot, .bot, archivo .bot, msbot, recursos del bot, administrar recursos del bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 276b553a6990ed286acbf073825afa7c4656de32
ms.sourcegitcommit: 6c719b51c9e4e84f5642100a33fe346b21360e8a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/28/2018
ms.locfileid: "52452027"
---
# <a name="manage-resources-with-a-bot-file"></a>Administración de recursos con un archivo .bot

Los bots suelen consumir muchos servicios diferentes, como [LUIS.ai](https://luis.ai) o [QnaMaker.ai](https://qnamaker.ai). Cuando está desarrollando un bot, no hay ningún lugar uniforme para almacenar los metadatos acerca de los servicios que están en uso.  Esto impide la creación de herramientas que entiendan un bot como un todo.

Para solucionar este problema, hemos creado un **archivo .bot** para que actúe como el lugar para reunir todas las referencias de servicio en un solo lugar y habilitar las herramientas.  Por ejemplo, Bot Framework Emulator ([V4](https://aka.ms/Emulator-wiki-getting-started)) usa un archivo .bot para crear una vista unificada de los servicios conectados que consume el bot.  

Con un archivo .bot, puede registrar servicios como:

* **Localhost**: puntos de conexión del depurador local
* [**Azure Bot Service**](https://azure.microsoft.com/en-us/services/bot-service/): registros de Azure Bot Service.
* [**LUIS.AI**](https://www.luis.ai/): LUIS permite al bot comunicarse con las personas con lenguaje natural... 
* [**QnA Maker**](https://qnamaker.ai/): crear, entrenar y publicar un bot de preguntas y respuestas simple basado en direcciones URL de preguntas más frecuentes, documentos estructurados o contenido editorial en cuestión de minutos.
* [**Dispatch**](https://github.com/Microsoft/botbuilder-tools/tree/master/Dispatch): modelos para el envío mediante varios servicios.
* [**Azure Application Insights**](https://azure.microsoft.com/en-us/services/application-insights/): para las conclusiones y el análisis del bot.
* [**Azure Blob Storage**](https://azure.microsoft.com/en-us/services/storage/blobs/): para la persistencia del estado del bot. 
* [**Azure Cosmos DB**](https://azure.microsoft.com/en-us/services/cosmos-db/): servicio de base de datos multimodelo distribuido globalmente para conservar el estado del bot.

Aparte de estos, el bot podría depender de otros servicios personalizados. Puede aprovechar la funcionalidad de [servicio genérico](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/add-services.md) para conectar una configuración de servicio genérico.

## <a name="when-is-a-bot-file-created"></a>¿Cuando se crea un archivo .bot? 
- Si crea un bot con [Azure Bot Service](https://ms.portal.azure.com/#blade/Microsoft_Azure_Marketplace/GalleryResultsListBlade/selectedSubMenuItemId/%7B%22menuItemId%22%3A%22gallery%2FCognitiveServices_MP%2FBotService%22%2C%22resourceGroupId%22%3A%22%22%2C%22resourceGroupLocation%22%3A%22%22%2C%22dontDiscardJourney%22%3Afalse%2C%22launchingContext%22%3A%7B%22source%22%3A%5B%22GalleryFeaturedMenuItemPart%22%5D%2C%22menuItemId%22%3A%22CognitiveServices_MP%22%2C%22subMenuItemId%22%3A%22BotService%22%7D%7D), se crea automáticamente un archivo .bot con la lista de servicios conectados aprovisionados. El archivo .bot se cifra de forma predeterminada.
- Si crea un bot mediante una [plantilla](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) de Bot Builder SDK V4 para Visual Studio o mediante el [generador Yeoman](https://www.npmjs.com/package/generator-botbuilder) de Bot Builder, se crea un archivo .bot automáticamente. No se aprovisionan servicios conectados en este flujo y no se cifra el archivo de bot.
- Si está empezando con [BotBuilder-samples](https://github.com/Microsoft/botbuilder-samples), cada ejemplo del SDK Bot Builder V4 incluye un archivo .bot y el archivo .bot no está cifrado. 
- También puede crear un archivo de bot mediante la herramienta [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md).

## <a name="what-does-a-bot-file-look-like"></a>¿Qué aspecto tiene un archivo de bot? 
Eche un vistazo a un archivo [.bot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/sample-bot-file.json) de ejemplo.
Para obtener información sobre cómo cifrar y descifrar el archivo .bot, consulte [Secretos del bot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/bot-file-encryption.md).

## <a name="why-do-i-need-a-bot-file"></a>¿Por qué necesito un archivo .bot?
Un archivo .bot **no** es un requisito para crear bots con el SDK Bot Builder. También puede usar appsettings.json, web.config, env, un almacén de claves o cualquier mecanismo que le convenga para realizar un seguimiento de las referencias de servicio y las claves de las que depende el bot. Sin embargo, para probar el bot con el emulador, necesitará un archivo .bot. La buena noticia es que el emulador puede crear un archivo .bot para las pruebas. Para ello, inicie el emulador y haga clic en el vínculo **Crear una nueva configuración de bot** en la página de bienvenida. En el cuadro de diálogo que aparece, escriba un **nombre de bot** y una **dirección URL del punto de conexión**. A continuación, conéctese.

Las ventajas del uso de un archivo .bot son:
- Proporciona una manera estándar para el almacenamiento de recursos, independientemente del lenguaje o plataforma que use.   
- Bot Framework Emulator y las herramientas de la CLI se basan en ellos y funcionan correctamente para el seguimiento de los servicios conectados en un formato coherente (en un archivo .bot) 
- Las soluciones de herramientas elegantes en torno a la creación y la administración de servicios son más difíciles sin un esquema bien definido (archivo .bot).  

## <a name="using-bot-file-in-your-bot-builder-sdk-bot"></a>Uso del archivo .bot en el bot del SDK Bot Builder
Puede usar el archivo .bot para obtener información de configuración del servicio en el código del bot. La biblioteca de configuración de Bot Framework disponible para [C#](https://www.nuget.org/packages/Microsoft.Bot.Configuration) y [JS](https://www.npmjs.com/package/botframework-config) le permite cargar un archivo de bot y admite varios métodos para consultar y obtener la información de configuración del servicio adecuada.

## <a name="additional-resources"></a>Recursos adicionales
Consulte el archivo Léame de [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) para más información sobre el uso de un archivo de bot.

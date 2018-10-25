---
title: Bots de ejemplo para Bot Builder SDK para Node.js | Microsoft Docs
description: Explore una amplia selección de bots de ejemplo que pueden ayudarle a comenzar a desarrollar bots con Bot Builder SDK para Node.js.
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f22e9e6c27b36955aa09953b7fc5d67f97674181
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997762"
---
# <a name="bot-builder-sdk-for-nodejs-samples"></a>Ejemplos de Bot Builder SDK para Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

En estos ejemplos se describen bots basados en tareas en los que se muestra cómo aprovechar las características de Bot Builder SDK para Node.js. Puede usar los ejemplos para empezar a trabajar rápidamente en la compilación de bots excelentes con funcionalidades enriquecidas.

## <a name="get-the-samples"></a>Obtención de los ejemplos
Para obtener los ejemplos, clone el repositorio [BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) de GitHub mediante Git.

```cmd
git clone https://github.com/Microsoft/BotBuilder-Samples.git
cd BotBuilder-Samples
```

Los bots de ejemplo creados con Bot Builder SDK para Node.js se organizan en el directorio **Node**.

También puede ver los ejemplos en GitHub e implementarlos directamente en Azure.

## <a name="core"></a>Núcleo
En estos ejemplos se muestran las técnicas básicas para la creación de bots enriquecidos y completos.

Muestra | DESCRIPCIÓN
------------ | ------------- 
[Send Attachment](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-SendAttachment) | Bot de ejemplo que envía al usuario datos adjuntos de elementos multimedia sencillos (imágenes). 
[Receive Attachment](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-ReceiveAttachment) | Bot de ejemplo que recibe los datos adjuntos enviados por el usuario y los descarga. 
[Create New Conversation](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-CreateNewConversation)  | Bot de ejemplo que inicia una conversación nueva usando una dirección de usuario almacenada previamente.
[Get Members of a Conversation](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-GetConversationMembers) | Bot de ejemplo que recupera la lista de miembros de la conversación y detecta cuando cambia. 
[Direct Line](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-DirectLine) | Bot de ejemplo y cliente personalizado que se comunican entre sí usando Direct Line API. 
[Direct Line (WebSockets)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-DirectLineWebSockets) | Bot de ejemplo y cliente personalizado que se comunican entre sí mediante Direct Line API + WebSockets. 
[Multi Dialogs](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-MultiDialogs) | Bot de ejemplo que muestra varios tipos de cuadros de diálogo.
[State API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-State) | Bot de ejemplo sin estado que realiza el seguimiento del contexto de una conversación.
[Custom State API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-CustomState) | Bot de ejemplo sin estado que realiza el seguimiento del contexto de una conversación mediante un proveedor de almacenamiento personalizado.
[ChannelData](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-ChannelData) | Bot de ejemplo que envía metadatos nativos a Facebook mediante ChannelData.
[AppInsights](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-AppInsights) | Bot de ejemplo que registra datos de telemetría en una instancia de Application Insights.

## <a name="search"></a>Search
En este ejemplo se muestra cómo aprovechar Azure Search en bots controlados por datos.

Muestra | DESCRIPCIÓN
------------ | -------------
[Azure Search](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-Search) | Dos bots de ejemplo que ayudan al usuario a navegar por grandes cantidades de contenido.


## <a name="cards"></a>Cards
En estos ejemplos se muestra cómo enviar tarjetas enriquecidas en Bot Framework.

Muestra | DESCRIPCIÓN
------------ | -------------
[Rich Cards](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/cards-RichCards) | Bot de ejemplo que envía varios tipos de tarjetas enriquecidas.
[Carousel of Cards](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/cards-CarouselCards) | Bot de ejemplo que envía varias tarjetas enriquecidas dentro de un solo mensaje con el diseño de carrusel.

## <a name="intelligence"></a>Inteligencia
En estos ejemplos se muestra cómo agregar funcionalidades de inteligencia artificial a un bot con las API de Bing y Microsoft Cognitive Services.

Muestra | DESCRIPCIÓN
------------ | -------------
[LUIS](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-LUIS) | Bot de ejemplo que usa LuisDialog para integrarse con una aplicación LUIS.ai.
[Image Caption](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-ImageCaption) | Bot de ejemplo que obtiene un título de imagen mediante Vision API de Microsoft Cognitive Services.
[Speech To Text](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-SpeechToText)  | Bot de ejemplo que obtiene texto de audio mediante Bing Speech API.
[Similar Products](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-SimilarProducts) | Bot de ejemplo que busca productos visualmente similares con Bing Image Search API. 
[Zummer](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-Zummer) | Bot de ejemplo que busca artículos de Wikipedia mediante Bing Search API.

## <a name="reference-implementation"></a>Implementación de referencia
Este ejemplo está diseñado para presentar un escenario de principio a fin. Es una fuente excelente de fragmentos de código si está pensando en implementar características más complejas en el bot.


Muestra | DESCRIPCIÓN
------------ | -------------
[Contoso Flowers](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-ContosoFlowers) | Bot de ejemplo que usa numerosas características de Bot Framework.


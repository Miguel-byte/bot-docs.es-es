---
title: Bots de ejemplo para el SDK de Bot Builder para .NET | Microsoft Docs
description: Explore los bots de ejemplo que pueden ayudar a comenzar el desarrollo de bots con el SDK de Bot Builder para .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/13/2018
ms.openlocfilehash: 0ee2f062b66db8a09be844a6a2af33bb12a1d7fd
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000252"
---
# <a name="bot-builder-sdk-for-net-samples"></a>Ejemplos del SDK de Bot Builder para .NET

::: moniker range="azure-bot-service-3.0"

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

En estos ejemplos se describen bots basados en tareas en los que se muestra cómo aprovechar las características del SDK de Bot Builder para .NET. Puede usar los ejemplos para empezar a trabajar rápidamente en la compilación de bots excelentes con funcionalidades enriquecidas.

## <a name="get-the-samples"></a>Obtención de los ejemplos
Para obtener los ejemplos, clone el repositorio [BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) de GitHub mediante Git.

```cmd
git clone https://github.com/Microsoft/BotBuilder-Samples.git
cd BotBuilder-Samples
```

Los bots de ejemplo creados con el SDK de Bot Builder para .NET se organizan en el directorio **CSharp**.

También puede ver los ejemplos en GitHub e implementarlos directamente en Azure.

## <a name="core"></a>Núcleo
En estos ejemplos se muestran las técnicas básicas para la creación de bots enriquecidos y completos.

Muestra | DESCRIPCIÓN
------------ | ------------- 
[Send Attachment](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-SendAttachment) | Bot de ejemplo que envía al usuario datos adjuntos de elementos multimedia sencillos (imágenes). 
[Receive Attachment](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-ReceiveAttachment) | Bot de ejemplo que recibe los datos adjuntos enviados por el usuario y los descarga. 
[Create New Conversation](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-CreateNewConversation)  | Bot de ejemplo que inicia una conversación nueva usando una dirección de usuario almacenada previamente.
[Get Members of a Conversation](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GetConversationMembers) | Bot de ejemplo que recupera la lista de miembros de la conversación y detecta cuando cambia. 
[Direct Line](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-DirectLine) | Bot de ejemplo y cliente personalizado que se comunican entre sí usando Direct Line API. 
[Direct Line (WebSockets)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-DirectLineWebSockets) | Bot de ejemplo y cliente personalizado que se comunican entre sí mediante Direct Line API + WebSockets. 
[Multi Dialogs](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-MultiDialogs) | Bot de ejemplo que muestra varios tipos de cuadros de diálogo.
[State API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-State) | Bot de ejemplo sin estado que realiza el seguimiento del contexto de una conversación.
[Custom State API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-CustomState) | Bot de ejemplo sin estado que realiza el seguimiento del contexto de una conversación mediante un proveedor de almacenamiento personalizado.
[ChannelData](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-ChannelData) | Bot de ejemplo que envía metadatos nativos a Facebook mediante ChannelData.
[AppInsights](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-AppInsights) | Bot de ejemplo que registra datos de telemetría en una instancia de Application Insights.

## <a name="search"></a>Search
En este ejemplo se muestra cómo aprovechar Azure Search en bots controlados por datos.

Muestra | DESCRIPCIÓN
------------ | -------------
[Azure Search](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search) | Dos bots de ejemplo que ayudan al usuario a navegar por grandes cantidades de contenido.


## <a name="cards"></a>Cards
En estos ejemplos se muestra cómo enviar tarjetas enriquecidas en Bot Framework.

Muestra | DESCRIPCIÓN
------------ | -------------
[Rich Cards](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/cards-RichCards) | Bot de ejemplo que envía varios tipos de tarjetas enriquecidas.
[Carousel of Cards](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/cards-CarouselCards) | Bot de ejemplo que envía varias tarjetas enriquecidas dentro de un solo mensaje con el diseño de carrusel.

## <a name="intelligence"></a>Inteligencia
En estos ejemplos se muestra cómo agregar funcionalidades de inteligencia artificial a un bot con las API de Bing y Microsoft Cognitive Services.

Muestra | DESCRIPCIÓN
------------ | -------------
[LUIS](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-LUIS) | Bot de ejemplo que usa LuisDialog para integrarse con una aplicación LUIS.ai.
[Image Caption](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-ImageCaption) | Bot de ejemplo que obtiene un título de imagen mediante Vision API de Microsoft Cognitive Services.
[Speech To Text](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-SpeechToText)  | Bot de ejemplo que obtiene texto de audio mediante Bing Speech API.
[Similar Products](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-SimilarProducts) | Bot de ejemplo que busca productos visualmente similares con Bing Image Search API. 
[Zummer](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-Zummer) | Bot de ejemplo que busca artículos de Wikipedia mediante Bing Search API.

## <a name="reference-implementation"></a>Implementación de referencia
Este ejemplo está diseñado para presentar un escenario de principio a fin. Es una fuente excelente de fragmentos de código si está pensando en implementar características más complejas en el bot.


Muestra | DESCRIPCIÓN
------------ | -------------
[Contoso Flowers](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-ContosoFlowers) | Bot de ejemplo que usa numerosas características de Bot Framework.

::: moniker-end

::: moniker range="azure-bot-service-4.0"

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Los ejemplos del repositorio de ejemplos de Bot Builder describen bots basados en tareas en los que se muestra cómo aprovechar las características proporcionadas en el SDK para .NET. Puede usar los ejemplos para empezar a trabajar rápidamente en la compilación de bots excelentes con funcionalidades enriquecidas. Consulte el archivo [Léame](https://github.com/Microsoft/BotBuilder-Samples/blob/master/README.md) para obtener una lista de ejemplos e información adicional.

Para obtener los ejemplos, clone el repositorio [botbuilder-samples](https://github.com/Microsoft/botbuilder-samples) de GitHub mediante Git.
```cmd
git clone https://github.com/Microsoft/botbuilder-dotnet.git
```

::: moniker-end


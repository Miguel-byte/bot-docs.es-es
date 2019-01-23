---
title: Incorporación de datos adjuntos de tarjetas enriquecidas a mensajes | Microsoft Docs
description: Obtenga información sobre cómo agregar tarjetas enriquecidas a los mensajes mediante Bot Framework SDK para .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5a6fc63005797a1c645de7506a8f15df2dcd0557
ms.sourcegitcommit: b94361234816e6b95459f142add936732fc40344
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/15/2019
ms.locfileid: "54317680"
---
# <a name="add-rich-card-attachments-to-messages"></a>Incorporación de datos adjuntos de tarjetas enriquecidas a mensajes

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

Un intercambio de mensajes entre el usuario y el bot puede contener una o varias tarjetas enriquecidas que se presentan como una lista o un carrusel. La propiedad `Attachments` del objeto <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity</a> contiene una matriz de objetos <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Attachment</a> que presentan las tarjetas enriquecidas y los datos adjuntos con elementos multimedia dentro del mensaje. 

> [!NOTE]
> Para información sobre cómo agregar datos adjuntos con elementos multimedia a los mensajes, consulte [Incorporación de datos adjuntos con elementos multimedia a mensajes](bot-builder-dotnet-add-media-attachments.md).

## <a name="types-of-rich-cards"></a>Tipos de tarjetas enriquecidas

Bot Framework admite actualmente ocho tipos de tarjetas enriquecidas: 

| Tipo de tarjeta | DESCRIPCIÓN |
|----|----|
| <a href="/adaptive-cards/get-started/bots">Tarjeta adaptable</a> | Una tarjeta personalizable que puede contener cualquier combinación de texto, voz, imágenes, botones y campos de entrada. Consulte la [compatibilidad por canal](/adaptive-cards/get-started/bots#channel-status).  |
| [Tarjeta de animación][animationCard] | Una tarjeta que puede reproducir archivos GIF animados o vídeos cortos. |
| [Tarjeta de audio][audioCard] | Una tarjeta que se puede reproducir un archivo de audio. |
| [Tarjeta de imagen prominente][heroCard] | Una tarjeta que normalmente contiene una sola imagen grande, uno o varios botones y texto. |
| [Tarjeta de miniatura][thumbnailCard] | Una tarjeta que normalmente contiene una sola imagen miniatura, uno o varios botones y texto. |
| [Tarjeta de recepción][receiptCard] | Una tarjeta que permite que un bot proporcione una recepción al usuario. Normalmente, contiene la lista de elementos que se incluyen en el recibo, la información de impuestos y del total y texto adicional. |
| [Tarjeta de inicio de sesión][signinCard] | Una tarjeta que permite al bot solicitar el inicio de sesión del usuario. Normalmente contiene texto y uno o más botones en los cuales el usuario puede hacer clic para iniciar el proceso de inicio de sesión. |
| [Tarjeta de videollamada][videoCard] | Una tarjeta que puede reproducir vídeos. |

> [!TIP]
> Para mostrar varias tarjetas enriquecidas en formato de lista, establezca la propiedad `AttachmentLayout` de la actividad en "lista". Para mostrar varias tarjetas enriquecidas en formato de carrusel, establezca la propiedad `AttachmentLayout` de la actividad en "carrusel". Si el canal no admite el formato de carrusel, mostrará las tarjetas enriquecidas en el formato de lista, incluso si la propiedad `AttachmentLayout` especifica "carrusel".

## <a name="process-events-within-rich-cards"></a>Procesamiento de eventos dentro de tarjetas enriquecidas

Para procesar eventos dentro de tarjetas enriquecidas, defina los objetos `CardAction` para especificar qué debe ocurrir cuando el usuario hace clic en un botón o pulsa una sección de la tarjeta. Cada objeto `CardAction` contiene estas propiedades:

| Propiedad | Escriba | DESCRIPCIÓN | 
|----|----|----|
| Escriba | string | tipo de acción (uno de los valores especificados en la tabla siguiente) |
| Título | string | título del botón |
| Imagen | string | dirección URL de la imagen del botón |
| Valor | string | valor necesario para realizar el tipo de acción especificado |

> [!NOTE]
> Los botones dentro de las tarjetas adaptables no se crean mediante el uso de objetos `CardAction`, sino que mediante el esquema que definen las <a href="http://adaptivecards.io" target="_blank">tarjetas adaptables</a>. Consulte [Add an Adaptive Card to a message](#adaptive-card) (Incorporación de una tarjeta adaptable a un mensaje) para ver un ejemplo que muestra cómo agregar botones a una tarjeta adaptable.

En esta tabla se enumeran los valores válidos para `CardAction.Type` y se describe el contenido esperado de `CardAction.Value` para cada tipo:

| CardAction.Type | CardAction.Value | 
|----|----|
| openUrl | Dirección URL que se abrirá en el explorador integrado |
| imBack | Texto del mensaje para enviar al bot (desde el usuario que hizo clic en el botón o que pulsó la tarjeta). Este mensaje (del usuario al bot) será visible para todos los participantes de la conversación mediante la aplicación cliente que hospeda la conversación. |
| postBack | Texto del mensaje para enviar al bot (desde el usuario que hizo clic en el botón o que pulsó la tarjeta). Algunas aplicaciones cliente pueden mostrar este texto en la fuente del mensaje, donde estará visible para todos los participantes de la conversación. |
| llamada | Destino de una llamada telefónica con este formato: **tel:123123123123** |
| playAudio | Dirección URL del audio que se va a reproducir |
| playVideo | Dirección URL del vídeo que se va a reproducir |
| showImage | Dirección URL de la imagen que se va a mostrar |
| DownloadFile | Dirección URL del archivo que se va a descargar |
| signin | Dirección URL del flujo de OAuth que se va a iniciar |

## <a name="add-a-hero-card-to-a-message"></a>Incorporación de una tarjeta de imagen prominente a un mensaje

La tarjeta de imagen prominente normalmente contiene una sola imagen grande, uno o varios botones y texto. 

Este ejemplo de código muestra cómo crear un mensaje de respuesta que contiene tres tarjetas de imagen prominente presentadas en formato de carrusel: 

[!code-csharp[Add HeroCard attachment](../includes/code/dotnet-add-attachments.cs#addHeroCardAttachment)]

## <a name="add-a-thumbnail-card-to-a-message"></a>Incorporación de una tarjeta de miniatura a un mensaje

La tarjeta de miniatura normalmente contiene una sola imagen en miniatura, uno o varios botones y texto. 

Este ejemplo de código muestra cómo crear un mensaje de respuesta que contiene dos tarjetas de miniatura presentadas en formato de lista: 

[!code-csharp[Add ThumbnailCard attachment](../includes/code/dotnet-add-attachments.cs#addThumbnailCardAttachment)]

## <a name="add-a-receipt-card-to-a-message"></a>Incorporación de una tarjeta de recepción a un mensaje

La tarjeta de recepción permite que un bot proporcione una recepción al usuario. Normalmente, contiene la lista de elementos que se incluyen en el recibo, la información de impuestos y del total y texto adicional. 

Este ejemplo de código muestra cómo crear un mensaje de respuesta que contiene una tarjeta de recepción: 

[!code-csharp[Add ReceiptCard attachment](../includes/code/dotnet-add-attachments.cs#addReceiptCardAttachment)]

## <a name="add-a-sign-in-card-to-a-message"></a>Incorporación de una tarjeta de inicio de sesión a un mensaje

La tarjeta de inicio de sesión permite al bot solicitar el inicio de sesión del usuario. Normalmente contiene texto y uno o más botones en los cuales el usuario puede hacer clic para iniciar el proceso de inicio de sesión. 

Este ejemplo de código muestra cómo crear un mensaje de respuesta que contiene una tarjeta de inicio de sesión:

[!code-csharp[Add SignInCard attachment](../includes/code/dotnet-add-attachments.cs#addSignInCardAttachment)]

## <a id="adaptive-card"></a> Incorporación de una tarjeta adaptable a un mensaje

Una tarjeta adaptable puede contener cualquier combinación de texto, voz, imágenes, botones y campos de entrada. Las tarjetas adaptables se crean con el formato JSON especificado en <a href="http://adaptivecards.io" target="_blank">Adaptive Cards</a> (Tarjetas adaptables), que proporciona control total sobre el contenido y el formato de la tarjeta. 

Para crear una tarjeta adaptable con. NET, instale el paquete `AdaptiveCards` de NuGet. Luego, use la información del sitio <a href="http://adaptivecards.io" target="_blank">Adaptive Cards</a> (Tarjetas adaptables) a fin de comprender el esquema de la tarjeta adaptable, explorar los elementos de la tarjeta adaptable y ver ejemplos de JSON que puede usar para crear tarjetas de diferente composición y complejidad. Además, puede usar el visualizador interactivo para diseñar cargas útiles de la tarjeta adaptable y obtener una vista previa del resultado de la tarjeta.

En este ejemplo de código se muestra cómo crear un mensaje que contiene una tarjeta adaptable para un recordatorio del calendario: 

[!code-csharp[Add Adaptive Card attachment](../includes/code/dotnet-add-attachments.cs#addAdaptiveCardAttachment)]

La tarjeta resultante contiene tres bloques de texto, un campo de entrada (lista de opciones) y tres botones:

![Recordatorio del calendario de tarjeta adaptable](../media/adaptive-card-reminder.png)

## <a name="additional-resources"></a>Recursos adicionales

- [Preview features with the Channel Inspector][inspector] (Vista previa de las características con el Inspector de canales)
- <a href="http://adaptivecards.io" target="_blank">Adaptive Cards</a> (Tarjetas adaptables)
- [Activities overview](bot-builder-dotnet-activities.md) (Introducción a las actividades)
- [Creación de mensajes](bot-builder-dotnet-create-messages.md)
- [Incorporación de datos adjuntos con elementos multimedia a mensajes](bot-builder-dotnet-add-media-attachments.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Clase Activity</a>
- <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Clase Attachment</a>

[animationCard]: /dotnet/api/microsoft.bot.connector.animationcard

[audioCard]: /dotnet/api/microsoft.bot.connector.audiocard 

[heroCard]: /dotnet/api/microsoft.bot.connector.herocard 

[thumbnailCard]: /dotnet/api/microsoft.bot.connector.thumbnailcard 

[receiptCard]: /dotnet/api/microsoft.bot.connector.receiptcard 

[signinCard]: /dotnet/api/microsoft.bot.connector.signincard 

[videoCard]: /dotnet/api/microsoft.bot.connector.videocard

[inspector]: ../bot-service-channel-inspector.md

---
title: Incorporación de elementos multimedia a los mensajes | Microsoft Docs
description: Obtenga información sobre cómo agregar elementos multimedia a los mensajes mediante Bot Framework SDK.
keywords: multimedia, mensajes, imágenes, audio, vídeo, archivos, MessageFactory, tarjetas enriquecidas, mensajes, tarjetas adaptables, tarjeta de imagen prominente, acciones sugeridas
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7093f13d1958c741b497a50535eb70a255dfcbe8
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032581"
---
# <a name="add-media-to-messages"></a>Incorporación de elementos multimedia a los mensajes

[!INCLUDE[applies-to](../includes/applies-to.md)]

Un intercambio de mensajes entre el usuario y el bot puede contener datos adjuntos con elementos multimedia, como imágenes, vídeo, audio y archivos. Bot Framework SDK es compatible con la tarea de enviar mensajes enriquecidos al usuario. Para determinar el tipo de mensajes enriquecidos que admite un canal (Facebook, Skype, Slack, etc), consulte la documentación del canal para más información sobre las limitaciones.

Consulte cómo [diseñar la experiencia del usuario](../bot-service-design-user-experience.md) para obtener una lista de las tarjetas disponibles.

## <a name="send-attachments"></a>Envío de datos adjuntos

Para enviar el contenido de usuario como una imagen o un vídeo, puede agregar datos adjuntos o una lista de datos adjuntos a un mensaje.

Consulte cómo [diseñar la experiencia del usuario](../bot-service-design-user-experience.md) para obtener una lista de las tarjetas disponibles.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

La propiedad `Attachments` del objeto `Activity` contiene una matriz de objetos `Attachment` que representan los datos adjuntos con elementos multimedia y las tarjetas enriquecidas adjuntas al mensaje. Para agregar datos adjuntos con elementos multimedia a un mensaje, cree un objeto `Attachment` para la actividad `reply` (que se creó fuera de la actividad con `CreateReply()`) y establezca las propiedades `ContentType`, `ContentUrl` y `Name`.

El código fuente que se muestra a continuación se basa en el ejemplo [Handling Attachments](https://aka.ms/bot-attachments-sample-code) (sobre control de datos adjuntos).

Para crear el mensaje de respuesta, defina el texto y, a continuación, configure los datos adjuntos. La asignación de los datos adjuntos a la respuesta es la misma para cada tipo de archivo adjunto, aunque los distintos datos adjuntos se configuran y definen de manera diferente, tal como se muestra en los siguientes fragmentos de código. El código siguiente configura la respuesta para los datos adjuntos insertados:

**Bots/AttachmentsBot.cs** [!code-csharp[inline attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=108-109)]

A continuación, nos centramos en los tipos de datos adjuntos. Primero, datos adjuntos insertados:

**Bots/AttachmentsBot.cs** [!code-csharp[inline attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=165-176)]

Seguidamente, datos adjuntos cargados:

**Bots/AttachmentsBot.cs** [!code-csharp[uploaded attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=179-215)]

Por último, un archivo adjunto de Internet:

**Bots/AttachmentsBot.cs** [!code-csharp[online attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=218-227)]


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El código fuente que se muestra a continuación se basa en el ejemplo [JS Handling Attachments](https://aka.ms/bot-attachments-sample-code-js) (sobre control de datos adjuntos de JS).

Para usar datos adjuntos, incluya las siguientes bibliotecas en el bot:

**bots/attachmentsBot.js** [!code-javascript[attachments libraries](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=4)]

Para crear el mensaje de respuesta, defina el texto y, a continuación, configure los datos adjuntos. La asignación de los datos adjuntos a la respuesta es la misma para cada tipo de archivo adjunto, aunque los distintos datos adjuntos se configuran y definen de manera diferente, tal como se muestra en los siguientes fragmentos de código. El código siguiente configura la respuesta para los datos adjuntos insertados:

**bots/attachmentsBot.js** [!code-javascript[attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=119,128-129)]

Para enviar al usuario un único fragmento de contenido, como una imagen o un vídeo, puede enviar elementos multimedia de varias maneras diferentes. Primero, como datos adjuntos insertados:

**bots/attachmentsBot.js** [!code-javascript[inline attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=170-179)]

Seguidamente, datos adjuntos cargados:

**bots/attachmentsBot.js** [!code-javascript[uploaded attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=197-215)]

Por último, un archivo adjunto de Internet incluido en una dirección URL:

**bots/attachmentsBot.js** [!code-javascript[internet attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=184-191)]

---

Si los datos adjuntos son una imagen, un audio o un vídeo, el servicio del conector comunicará los datos adjuntos al canal de una manera que permita que el [canal](bot-builder-channeldata.md) presente esos datos adjuntos dentro de la conversación. Si los datos adjuntos son un archivo, la dirección URL del archivo se presentará como un hipervínculo dentro de la conversación.

## <a name="send-a-hero-card"></a>Envío de una tarjeta de imagen prominente

Además de los datos adjuntos de imagen o vídeo sencillos, puede adjuntar una **tarjeta de imagen prominente** que le permite combinar las imágenes y los botones en un objeto y los envían al usuario. Markdown es compatible con la mayoría de los campos de texto, pero la compatibilidad puede variar según el canal.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Para redactar un mensaje con un botón y una tarjeta de imagen prominente, puede adjuntar `HeroCard` a un mensaje. 

El código fuente que se muestra a continuación se basa en el ejemplo [Handling Attachments](https://aka.ms/bot-attachments-sample-code) (sobre control de datos adjuntos).

**Bots/AttachmentsBot.cs** [!code-csharp[Hero card](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=39-62)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Para redactar un mensaje con un botón y una tarjeta de imagen prominente, puede adjuntar `HeroCard` a un mensaje. 

El código fuente que se muestra a continuación se basa en el ejemplo [JS Handling Attachments](https://aka.ms/bot-attachments-sample-code-js) (sobre control de datos adjuntos de JS).

**bots/attachmentsBot.js** [!code-javascript[hero card](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=148-164)]

---

## <a name="process-events-within-rich-cards"></a>Procesamiento de eventos dentro de tarjetas enriquecidas

Para procesar eventos dentro de tarjetas enriquecidas, use los objetos de _acción de tarjeta_ para especificar qué debe ocurrir cuando el usuario hace clic en un botón o pulsa una sección de la tarjeta. Cada acción de la tarjeta tiene un _tipo_ y un _valor_.

Para que el funcionamiento sea correcto, asigne un tipo de acción a cada elemento en el que se puede hacer clic en la tarjeta. Esta tabla enumera y describe los tipos de acciones disponibles y lo que debería estar en la propiedad de valor asociada.

| Type | DESCRIPCIÓN | Valor |
| :---- | :---- | :---- |
| openUrl | Se abre una dirección URL en el explorador integrado. | Dirección URL que se va a abrir. |
| imBack | Envía un mensaje al bot y publica una respuesta visible en el chat. | Texto del mensaje para enviar. |
| postBack | Envía un mensaje al bot y puede no publicar una respuesta visible en el chat. | Texto del mensaje para enviar. |
| llamada | Inicia una llamada de teléfono. | Destino de una llamada telefónica con este formato: `tel:123123123123`. |
| playAudio | Reproduce audio. | La dirección URL del audio que se va a reproducir. |
| playVideo | Reproduce un vídeo. | La dirección URL del vídeo que se va a reproducir. |
| showImage | Muestra una imagen. | La dirección URL de la imagen que se va a mostrar. |
| DownloadFile | Descarga un archivo. | La dirección URL del archivo que se va a descargar. |
| signin | Inicia un proceso de inicio de sesión de OAuth. | La dirección URL del flujo de OAuth que se va a iniciar. |

## <a name="hero-card-using-various-event-types"></a>Tarjeta de imagen prominente a través de varios tipos de evento

El código siguiente muestra ejemplos que usan varios eventos de tarjeta enriquecida.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Para obtener ejemplos de todas las tarjetas disponibles, consulte el [ejemplo de tarjetas de C#](https://aka.ms/bot-cards-sample-code).

**Cards.cs** [!code-csharp[hero cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Cards.cs?range=27-40)]

**Cards.cs** [!code-csharp[cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Cards.cs?range=91-100)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Para obtener ejemplos de todas las tarjetas disponibles, consulte el [ejemplo de tarjetas de JS](https://aka.ms/bot-cards-js-sample-code).

**dialogs/mainDialog.js** [!code-javascript[hero cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=213-225)]

**dialogs/mainDialog.js** [!code-javascript[sign in cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=266-272)]

---

## <a name="send-an-adaptive-card"></a>Envío de una tarjeta adaptable
Las tarjetas adaptables y MessageFactory se utilizan para enviar mensajes enriquecidos, incluidos textos, imágenes, vídeo, audio y archivos para comunicarse con los usuarios. Sin embargo, hay algunas diferencias entre ellos. 

En primer lugar, solo algunos canales admiten tarjetas adaptables y aquellos canales que las admiten, podrían hacerlo de un modo parcial. Por ejemplo, si envía una tarjeta adaptable en Facebook, los botones no funcionarán, pero los textos y las imágenes funcionan bien. MessageFactory es simplemente una clase asistente en Bot Framework SDK para automatizar los pasos de creación y es compatible con la mayoría de los canales. 

En segundo lugar, una tarjeta adaptable entrega los mensajes en formato de tarjeta y el canal determina el diseño de la tarjeta. El formato de los mensajes que entrega MessageFactory depende del canal y no es necesariamente en formato de tarjeta, a menos que la tarjeta adaptable sea parte de los datos adjuntos. 

Para encontrar la información más reciente sobre la compatibilidad de canales de tarjetas adaptables, consulte <a href="http://adaptivecards.io/designer/">Diseñador de tarjetas adaptables</a>.

> [!NOTE]
> Debe probar esta característica con los canales que el bot usará para determinar si esos canales admiten las tarjetas adaptables.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Para usar las tarjetas adaptables, no olvide agregar el paquete de NuGet `AdaptiveCards`.

El código fuente que se muestra a continuación se basa en el ejemplo [Uso de tarjetas](https://aka.ms/bot-cards-sample-code):

**Cards.cs** [!code-csharp[adaptive cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Cards.cs?range=13-25)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Para usar las tarjetas adaptables, no olvide agregar el paquete de npm `adaptivecards`.

El código fuente que se muestra a continuación se basa en el ejemplo [Uso de tarjetas en JS](https://aka.ms/bot-cards-js-sample-code). 

En este caso, la tarjeta adaptable se almacena en su propio archivo y se incluye en el bot:

**resources/adaptiveCard.json** [!code-json[adaptive cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/resources/adaptiveCard.json)]

A continuación, se crea con CardFactory:

**dialogs/mainDialog.js** [!code-javascript[adaptive cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=177-179)]

---

## <a name="send-a-carousel-of-cards"></a>Envío de un carrusel de cartas

Los mensajes también pueden incluir varios datos adjuntos en un diseño de carrusel, que coloca los datos adjuntos en paralelo y permite que el usuario se desplace por ellos.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

El código fuente que se muestra a continuación se basa en el [Ejemplo de tarjetas](https://aka.ms/bot-cards-sample-code).

En primer lugar, se crea la respuesta y se definen los datos adjuntos como una lista.

**Dialogs/MainDialog.cs** [!code-csharp[carousel of cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Dialogs/MainDialog.cs?range=61-66)]

A continuación, agregue los datos adjuntos. Aquí se agregan uno cada vez, pero, si quiere, puede manipular la lista para agregar las tarjetas.

**Dialogs/MainDialog.cs** [!code-csharp[carousel of cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Dialogs/MainDialog.cs?range=105-113)]

Una vez que se han agregado los datos adjuntos, puede enviar la respuesta como cualquier otra.

**Dialogs/MainDialog.cs** [!code-csharp[carousel of cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Dialogs/MainDialog.cs?range=117-118)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El código fuente que se muestra a continuación se basa en el [Ejemplo de tarjetas en JS](https://aka.ms/bot-cards-js-sample-code).

Para enviar un carrusel de tarjetas, envíe una respuesta con los datos adjuntos como una matriz y el tipo de diseño definido como `Carousel`:

**dialogs/mainDialog.js** [!code-javascript[carousel of cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=104-116)]

---

<!-- TODO: Add a media card, such as video or audion. Revisit which examples we put here and link to the 06 through 08 samples. -->

## <a name="additional-resources"></a>Recursos adicionales

Consulte cómo [diseñar la experiencia del usuario](../bot-service-design-user-experience.md) para obtener una lista de las tarjetas disponibles.

Para más información sobre el esquema, consulte el [esquema de la tarjeta de Bot Framework](https://aka.ms/botSpecs-cardSchema) y la [sección sobre la actividad de mensajes](https://aka.ms/botSpecs-activitySchema#message-activity) del esquema de actividad de Bot Framework.

| Código de ejemplo | C# | JS |
| :------ | :----- | :---|
| Cards | [Ejemplo de C#](https://aka.ms/bot-cards-sample-code) | [Ejemplo de JS](https://aka.ms/bot-cards-js-sample-code) |
| Datos adjuntos | [Ejemplo de C#](https://aka.ms/bot-attachments-sample-code) | [Ejemplo de JS](https://aka.ms/bot-attachments-sample-code-js) |
| Acciones sugeridas | [Ejemplo de C#](https://aka.ms/SuggestedActionsCSharp) | [Ejemplo de JS](https://aka.ms/SuggestedActionsJS) |

Consulte el repositorio de ejemplos de Bot Builder en [GitHub](https://aka.ms/bot-samples-readme) para obtener ejemplos adicionales.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Agregar botones para guiar la acción del usuario](./bot-builder-howto-add-suggested-actions.md)

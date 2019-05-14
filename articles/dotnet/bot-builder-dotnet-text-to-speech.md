---
title: Incorporación de voz a los mensajes | Microsoft Docs
description: Obtenga información sobre cómo agregar voz a los mensajes mediante Bot Framework SDK para .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f93ab91281cf0f19be10898436dc41a6a1583c9a
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032979"
---
# <a name="add-speech-to-messages"></a>Incorporación de voz a los mensajes

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.js](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

Si está creando un bot para un canal habilitado para voz, como Cortana, puede construir mensajes que especifiquen el texto que dirá el bot. Para intentar influir en el estado del micrófono del cliente, puede especificar una [sugerencia de entrada](bot-builder-dotnet-add-input-hints.md) para indicar si el bot acepta, espera o ignora la entrada del usuario.

## <a name="specify-text-to-be-spoken-by-your-bot"></a>Especificación del texto que dirá el bot

Con Bot Framework SDK para .NET, hay varias maneras de especificar el texto que dirá el bot en un canal habilitado para voz. Puede establecer la propiedad `Speak` del [mensaje][IMessageActivity], llamar al método `IDialogContext.SayAsync()` o especificar las opciones de aviso `speak` y `retrySpeak` al enviar un mensaje con un aviso integrado.

### <a id="message-speak"></a> IMessageActivity.Speak

Si va a crear un [mensaje][IMessageActivity] y a configurar sus propiedades individuales, puede establecer la propiedad `Speak` del mensaje para especificar el texto que dirá el bot. El siguiente ejemplo de código crea un mensaje que especifica el texto que se mostrará y el texto que se dirá, e indica que el bot [acepta la entrada del usuario](bot-builder-dotnet-add-input-hints.md).

[!code-csharp[Set speak property](../includes/code/dotnet-text-to-speech.cs#Speak1)]

### <a id="say-async"></a> IDialogContext.SayAsync()

Si usa [diálogos](bot-builder-dotnet-dialogs.md), puede llamar al método `SayAsync()` para crear y enviar un mensaje que especifique el texto que se dirá, además del texto que se mostrará y otras opciones. En el ejemplo de código siguiente se crea un mensaje que especifica el texto que se mostrará y el texto que se hablará.

[!code-csharp[Call SayAsync()](../includes/code/dotnet-text-to-speech.cs#Speak2)]

### <a id="prompt-options"></a> Opciones de aviso

Con cualquiera de los avisos integrados, puede establecer las opciones `speak` y `retrySpeak` para especificar el texto que dirá el bot. En el ejemplo de código siguiente se crea un aviso que especifica el texto que se mostrará, el texto que se dirá inicialmente y el texto que se dirá después de esperar un tiempo los datos de entrada del usuario. Se usa el formato [SSML](#ssml) para indicar que la palabra "claro" se dirá con una cantidad moderada de énfasis.

[!code-csharp[Set Prompt options](../includes/code/dotnet-text-to-speech.cs#Speak3)]

## <a id="ssml"></a>Lenguaje de marcado de síntesis de voz (SSML)

Para especificar un texto para que lo hable el bot, puede asignarle una cadena con formato de lenguaje de marcado de síntesis de voz (SSML). SSML es un lenguaje de marcado basado en XML (y, por tanto, debe ser un XML válido) que le permite controlar diversas características de voz del bot como voz, velocidad, volumen, pronunciación, tono, etc. Para más información sobre SSML, consulte <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">Referencia del lenguaje de marcado de síntesis de voz</a>.

Al proporcionar la cadena con formato SSML, se puede omitir el elemento de contenedor exterior de SSML.

## <a name="input-hints"></a>Sugerencias de entrada

Cuando envía un mensaje en un canal habilitado para voz, puede intentar influir en el estado del micrófono del cliente e incluir una sugerencia de entrada para indicar si el bot acepta, espera o ignora la entrada del usuario. Para más información, consulte [Incorporación de sugerencias de entrada a los mensajes](bot-builder-dotnet-add-input-hints.md).

## <a name="sample-code"></a>Código de ejemplo 

Para un ejemplo completo en el que se muestra cómo crear un bot habilitado para voz mediante Bot Framework SDK para. NET, consulte el ejemplo <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp" target="_blank">Roller Skill</a> en GitHub.

## <a name="additional-resources"></a>Recursos adicionales

- [Creación de mensajes](bot-builder-dotnet-create-messages.md)
- [Incorporación de sugerencias de entrada a los mensajes](bot-builder-dotnet-add-input-hints.md)
- <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">Lenguaje de marcado de síntesis de voz (SSML)</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/demo-RollerSkill" target="_blank">Ejemplo de Roller Skill (GitHub)</a>
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Clase Activity</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">Interfaz IMessageActivity interface</a>
- <a href="/dotnet/api/microsoft.bot.builder.dialogs.internals.dialogcontext" target="_blank">Clase DialogContext</a>
- <a href="/dotnet/api/microsoft.bot.builder.dialogs.internals.prompt-2" target="_blank">Clase Prompt</a>

[IMessageActivity]: /dotnet/api/microsoft.bot.connector.imessageactivity


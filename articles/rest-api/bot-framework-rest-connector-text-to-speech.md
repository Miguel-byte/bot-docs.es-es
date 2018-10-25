---
title: Incorporación de voz a mensajes | Microsoft Docs
description: Obtenga información sobre cómo agregar voz a los mensajes mediante el servicio Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 2aac000b7e8dd52b00659ffecde5184df6c29991
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998692"
---
# <a name="add-speech-to-messages"></a>Incorporación de voz a los mensajes
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.js](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

Si está creando un bot para un canal habilitado para voz, como Cortana, puede construir mensajes que especifiquen el texto que dirá el bot. Para intentar influir en el estado del micrófono del cliente, puede especificar una [sugerencia de entrada](bot-framework-rest-connector-add-input-hints.md) para indicar si el bot acepta, espera o ignora la entrada del usuario.

## <a name="specify-text-to-be-spoken-by-your-bot"></a>Especificar el texto que dirá el bot

Para especificar el texto que dirá el bot en un canal habilitado para voz, establezca la propiedad `speak`dentro del objeto [Activity][Activity] que representa el mensaje. Puede establecer la propiedad `speak` en una cadena de texto sin formato o en una cadena con formato de <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">lenguaje de marcado de síntesis de voz (SSML)</a>, un lenguaje de marcado basado en XML que le permite controlar diversas características de la voz del bot, como la voz, la velocidad, el volumen, la pronunciación, el tono y mucho más. 

La siguiente solicitud envía un mensaje que especifica el texto que se mostrará y el texto que se dirá, e indica que el bot [espera la entrad del usuario](bot-framework-rest-connector-add-input-hints.md). Especifica la propiedad `speak` con el formato <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">SSML</a> para indicar que la palabra "sure" se debe decir con una cantidad moderada de énfasis. En esta solicitud de ejemplo, `https://smba.trafficmanager.net/apis` representa el URI de base; los URI de base para las solicitudes que emite el bot pueden ser diferentes. Para obtener más información sobre cómo establecer el URI de base, consulte [Referencia de la API](bot-framework-rest-connector-api-reference.md#base-uri).

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/5d5cdc723
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "Are you sure that you want to cancel this transaction?",
    "speak": "Are you <emphasis level='moderate'>sure</emphasis> that you want to cancel this transaction?",
    "inputHint": "expectingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="input-hints"></a>Sugerencias de entrada

Cuando envía un mensaje en un canal habilitado para voz, puede intentar influir en el estado del micrófono del cliente e incluir una sugerencia de entrada para indicar si el bot acepta, espera o ignora la entrada del usuario. Para más información, consulte [Incorporación de sugerencias de entrada a los mensajes](bot-framework-rest-connector-add-input-hints.md).

## <a name="additional-resources"></a>Recursos adicionales

- [Creación de mensajes](bot-framework-rest-connector-create-messages.md)
- [Envío y recepción de mensajes](bot-framework-rest-connector-send-and-receive-messages.md)
- [Incorporación de sugerencias de entrada a los mensajes](bot-framework-rest-connector-add-input-hints.md)
- <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">Lenguaje de marcado de síntesis de voz (SSML)</a>

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object

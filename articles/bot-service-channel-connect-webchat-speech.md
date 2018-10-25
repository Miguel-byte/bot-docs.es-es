---
title: Habilitación de voz en un chat en web | Microsoft Docs
description: Obtenga información sobre cómo habilitar la característica de voz en un control de chat en web para un bot conectado al canal Chat en web.
keywords: característica de voz, chat en web, voz, audio, micrófono
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f98391f05e962bfc8846818d8a236d16846c8c5c
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997064"
---
# <a name="enable-speech-in-web-chat"></a>Habilitación de voz en un chat en web
Puede habilitar una interfaz de voz en el control de chat en web. Los usuarios interactúan con la interfaz de voz utilizando el micrófono en el control de chat en web.

![Muestra de voz en el chat en web](~/media/bot-service-channel-webchat/webchat-sample-speech.png)

Si el usuario escribe en lugar de decir la respuesta, Chat en web desactiva la funcionalidad de voz y el bot solo da una respuesta textual en lugar de hablar en voz alta. Para volver a habilitar la respuesta hablada, el usuario puede usar el micrófono para responder al bot la próxima vez. Si el micrófono está aceptando entradas, aparecerá oscurecido o rellenado. Si está atenuado, el usuario hace clic en él para habilitarlo.

## <a name="prerequisites"></a>Requisitos previos

  Antes de ejecutar la muestra, debe tener un secreto o un token de línea directa para el bot que quiera ejecutar con el control de Chat en web. 
  * Consulte [Connect a bot to Direct Line](bot-service-channel-connect-directline.md) (Conectar un bot a la línea directa) para obtener información sobre cómo obtener un secreto de línea directa asociado al bot.
  * Consulte [Generate a Direct Line token](rest-api/bot-framework-rest-direct-line-3-0-authentication.md) (Generar un token de línea directa) para obtener información sobre cómo intercambiar el secreto de un token.

## <a name="customizing-web-chat-for-speech"></a>Personalizar el chat en web para la característica de voz
Para habilitar la funcionalidad de voz en Chat en web, debe personalizar el código de JavaScript que invoca el control de Chat en web. Puede probar el Chat en web habilitado para la característica de voz de forma local, mediante los siguientes pasos.

1. Descargue el [ejemplo index.html](https://aka.ms/web-chat-speech-sample). <!-- this aka.ms link needs to be updated if the sample location changes -->
2. Edite el código en `index.html` según el tipo de soporte de voz que desee agregar. Los tipos de implementaciones de voz se describen en [Habilitar servicios de voz](#enable-speech-services). 
3. Inicie un servidor web. Una forma de hacerlo es usar `npm http-server` en el símbolo del sistema Node.js.

   * Para instalar `http-server` de forma global para que pueda ejecutarse desde la línea de comandos, ejecute este comando:

     ```
     npm install http-server -g
     ```

   * Para iniciar un servidor web usando el puerto 8000, desde el directorio que contiene `index.html`, ejecute este comando:

     ```
     http-server -p 8000
     ```
4. Dirija el explorador a `http://localhost:8000/samples?parameters`. Por ejemplo, `http://localhost:8000/samples?s=YOURDIRECTLINESECRET` invoca el bot con un secreto de línea directa. Los parámetros que se pueden establecer en la cadena de consulta se describen en la tabla siguiente:

   | Parámetro | DESCRIPCIÓN |
   |-----------|-------------|
   | s | Secreto de línea directa. Consulte [Connect a bot to Direct Line](bot-service-channel-connect-directline.md) (Conectar un bot a la línea directa) para obtener información sobre cómo obtener un secreto de línea directa. |
   | t | Token de línea directa. Consulte [Generate a Direct Line token](rest-api/bot-framework-rest-direct-line-3-0-authentication.md) (Crear un token de línea directa) para obtener información sobre cómo generar este token. |
   | dominio | Opcional. Dirección URL de un punto de conexión de línea directa alternativo.  |
   | WebSocket | Opcional. Establecer en "true" para usar WebSocket para recibir mensajes. El valor predeterminado es `false`. |
   | userid | Opcional. Id. del usuario del bot.  |
   | nombre de usuario | Opcional. Nombre de usuario del usuario del bot.  |
   | botid | Opcional. Id. del bot. |
   | botname | Opcional. Nombre del bot. |


## <a name="enable-speech-services"></a>Habilitar servicios de voz
La personalización le permite agregar la funcionalidad de voz de cualquiera de las maneras siguientes:

* **Característica de voz que proporciona el explorador**: utilice la funcionalidad de voz integrada en el explorador web. En este momento, esta funcionalidad solo está disponible en el explorador Chrome.
* **Usar el servicio Bing Speech**: puede usar el servicio Bing Speech para proporcionar reconocimiento y síntesis de voz. Esta forma de acceso a la funcionalidad de voz es compatible con varios exploradores. En este caso, el procesamiento se realiza en un servidor en lugar de hacerse en el explorador.
* **Crear un servicio de voz personalizado**: puede crear sus propios componentes personalizados de reconocimiento y síntesis de voz.

### <a name="browser-provided-speech"></a>Característica de voz que proporciona el explorador

El siguiente código crea una instancia del reconocedor de voz y los componentes de síntesis de voz que vienen con el explorador. Este método de agregar la funcionalidad de voz no es compatible con todos los exploradores. 

> [!NOTE] 
> Google Chrome admite el reconocedor de voz del explorador. Sin embargo, Chrome puede bloquear el micrófono en los siguientes casos:
> * Si la dirección URL de la página que contiene el Chat en web comienza con `http://` en lugar de `https://`.
> * Si la dirección URL es un archivo local que usa el protocolo `file://` en lugar de `http://localhost:8000`.

[!code-js[Specify speech options to use in-browser speech (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#BrowserSpeech)]

### <a name="bing-speech-service"></a>Servicio Bing Speech

El siguiente código crea una instancia del reconocedor de voz y los componentes de síntesis de voz que usan el servicio Bing Speech. El reconocimiento y la generación de voz se llevan a cabo en el servidor. Este mecanismo se admite en varios exploradores. 

> [!TIP]
> Puede usar el desbloqueo del reconocimiento de voz para mejorar la precisión del reconocimiento de voz del bot si usa el servicio Bing Speech. Para obtener más información, consulte la entrada de blog [Speech Support in Bot Framework](https://blog.botframework.com/2017/06/26/Speech-To-Text) (compatibilidad con voz en Bot Framework).

[!code-js[Specify speech options to use the Bing Speech API (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#BingSpeech)]

#### <a name="use-the-bing-speech-service-with-a-token"></a>Usar el servicio de Bing Speech con un token

También tiene la opción de habilitar el reconocimiento de voz de Cognitive Services con un token. El token se genera en un back-end seguro mediante la clave de API.

En el ejemplo de código siguiente se muestra cómo se realiza la operación de captura tokens desde un back-end seguro para evitar la exposición de la clave de API.

[!code-js[Fetch a token to use with the Bing Speech API (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#FetchToken)]

### <a name="custom-speech-service"></a>Custom Speech Service

También puede proporcionar su propio reconocimiento de voz personalizado que implementa ISpeechRecognizer, o la síntesis de voz que implementa ISpeechSynthesis. 

[!code-js[Fetch a token to use with a custom speech service (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#CustomSpeechService)]

### <a name="pass-the-speech-options-to-web-chat"></a>Pasar las opciones de voz al Chat en web

El código siguiente pasa las opciones de voz al control de Chat en web:

[!code-js[Pass speech options to Web Chat (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#PassSpeechOptionsToWebChat)]

## <a name="next-steps"></a>Pasos siguientes
Ahora que puede habilitar la interacción de voz con el Chat en web, puede ver el método que usa el bot para crear los mensajes de voz y ajustar el estado del micrófono:
* [Agregar voz a los mensajes (C#)](dotnet/bot-builder-dotnet-text-to-speech.md)
* [Agregar voz a los mensajes (Node.js)](nodejs/bot-builder-nodejs-text-to-speech.md)

## <a name="additional-resources"></a>Recursos adicionales

* También puede [descargar el código fuente](https://github.com/Microsoft/BotFramework-WebChat) para el control de Chat en web en GitHub.
* Asimismo, la [documentación de Bing Speech API](https://docs.microsoft.com/azure/cognitive-services/speech/home) proporciona más información sobre Bing Speech API.


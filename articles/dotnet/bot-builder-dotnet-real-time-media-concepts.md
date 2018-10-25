---
title: Llamada multimedia en tiempo real con Skype | Microsoft Docs
description: Comprenda los conceptos clave en la creación de un bot que puede llevar a llamadas de audio y vídeo en tiempo real con Skype, mediante el SDK de Bot Builder para. NET.
author: ssulzer
ms.author: ssulzer
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: cf11a68c7ccc404506fdfe01dcf12c7c421f4e2b
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998349"
---
# <a name="real-time-media-calling-with-skype"></a>Llamada multimedia en tiempo real con Skype

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

La plataforma multimedia en tiempo real para bots agrega una nueva dimensión a cómo los bots pueden interactuar con los usuarios al permitir las modalidades de voz, vídeo y pantalla compartida en tiempo real. Esta plataforma ofrece la posibilidad de crear bots de entretenimiento, educativos y de ayuda atractivos e interactivos. Los usuarios se comunican con bots multimedia en tiempo real mediante Skype.

Esta es una funcionalidad avanzada que permite que el bot envíe y reciba contenido de voz y vídeo *trama a trama y fotograma a fotograma*. El bot tiene acceso "sin procesar" a las modalidades de voz, vídeo y pantalla compartida en tiempo real. Por ejemplo, a medida que habla el usuario, el bot recibe 50 tramas de audio por segundo, cada una de ellas con 20 milisegundos (ms) de audio. El bot puede realizar el reconocimiento de voz en tiempo real a medida que se reciben las tramas de audio, en lugar de tener que esperar una grabación después de que el usuario ha dejado de hablar. Junto con el audio, el bot también puede enviar al usuario vídeo con una resolución de alta definición de 30 fotogramas por segundo.

La plataforma multimedia en tiempo real para bots funciona con la nube de llamadas de Skype y se encarga de la configuración de llamadas y del establecimiento de sesiones multimedia, lo que permite que el bot entable una conversación de voz y (opcionalmente) de vídeo con el autor de una llamada en Skype. La plataforma proporciona un sencillo "socket" a modo de API para que el bot envíe y reciba elementos multimedia, y controla la codificación y decodificación en tiempo real de elementos multimedia mediante códecs como SILK para audio y H.264 para vídeo. Un bot multimedia en tiempo real puede participar también en videollamadas grupales de Skype con varios participantes.

En este artículo se presentan los conceptos clave relacionados con la creación de un bot que puede llevar a cabo llamadas de audio y vídeo en tiempo real con Skype y se proporcionan vínculos a recursos de interés para los desarrolladores.

## <a name="media-session"></a>Sesión multimedia
Cuando un bot multimedia en tiempo real responde a una llamada entrante de Skype, decide si se debe admitir las modalidades de audio y vídeo, o solo audio. Para cada modalidad compatible, el bot puede enviar y recibir elementos multimedia, recibir solo o enviar solo. Por ejemplo, puede que un bot quiera enviar y recibir audio, pero solo enviar vídeo (ya que no quiere recibir el vídeo del autor de la llamada en Skype). El conjunto de modalidades de audio y vídeo establecido entre el autor de la llamada de Skype y el bot se conoce como **sesión multimedia**.

Hay dos modalidades de vídeo admitidas: "vídeo principal" y "pantalla compartida". El vídeo principal transmite el vídeo que el bot genera, o reproduce, al autor de la llamada, y transmite el vídeo del autor de la llamada (normalmente desde la cámara web del usuario) al bot. La modalidad de pantalla compartida permite al autor de la llamada compartir su pantalla (como un vídeo) con el bot. El bot no puede enviar el vídeo en pantalla compartida al usuario.

Cuando se une a una **videollamada grupal** en Skype de varios participantes, el bot multimedia en tiempo real puede admitir la recepción de varias secuencias de vídeo principal a la vez. Esto permite al bot "ver" más de un participante en la videollamada grupal.

## <a name="frames-and-frame-rate"></a>Tramas/fotogramas y velocidad de trama/fotograma
Un bot multimedia en tiempo real interactúa directamente con las modalidades de audio y vídeo de una llamada de Skype. Esto significa que el bot envía o recibe elementos multimedia como una secuencia de **tramas/fotogramas**, donde cada una representa una unidad de contenido. Un segundo de audio se puede transmitir como una secuencia de 50 tramas, cada una de ellas con 20 milisegundos (ms) (1/50 milésimas de segundo) de contenido. Un segundo de vídeo se puede segmentar como una secuencia de 30 imágenes fijas, cada una de las cuales se verá durante justo 33 ms (1/30 milésimas de segundo) antes de que se muestre el siguiente fotograma de vídeo. El número de fotogramas transmitidos o representados por segundo se denomina **velocidad de fotograma**. "30fps" indica 30 fotogramas por segundo.

## <a name="audio-format"></a>Formato de audio
Cada segundo de audio se representa como 16 000 **muestras**, y cada una de ellas almacena 16 bits de datos. Una trama de audio de 20 ms contiene 320 muestras (640 bytes de datos).

## <a name="video-format"></a>Formato de vídeo
Existen varios formatos de vídeo admitidos. Las dos propiedades principales de un formato de vídeo son su **tamaño de fotograma** y su **formato de color**. Los tamaños de fotograma admitidos incluyen 640 x 360 ("360p"), 1280 x 720 ("720p") y 1920 x 1080 ("1080p"). Los formatos de color admitidos incluyen NV12 (12 bits por píxel) y RGB24 (24 bits por píxel).

Un fotograma de vídeo de "720p" contiene 921 600 píxeles (1280 veces 720). En el formato de color RGB24, cada píxel se representa como 3 bytes (24 bits) compuesto cada byte por componentes rojo, verde y azul. Por lo tanto, un único fotograma de vídeo RGB24 de 720p requiere 2 764 800 bytes de datos (921 600 veces 3 bytes/píxel). A una velocidad de fotograma de 30fps, enviar fotogramas de vídeo RGB24 de 720p significa procesar aproximadamente 80 MB/s de contenido (que en esencia se comprime con el códec de vídeo H.264 antes de la transmisión de red).

## <a name="active-and-dominant-speakers"></a>Interlocutores activos y dominantes
Cuando se une a una videollamada grupal en Skype que consta de varios participantes, el bot puede identificar qué participantes están hablando en ese momento. Los **interlocutores activos** identifican qué participantes se escuchan en cada trama de audio recibida. Los **interlocutores dominantes** identifican qué participantes son actualmente los más activos (o "dominantes") en la conversación grupal, aunque su voz pueda no oírse en la trama de audio. El conjunto de interlocutores dominantes puede cambiar a medida que diferentes participantes toman la palabra.

## <a name="video-subscription"></a>Suscripción de vídeo
En una llamada con un único autor de llamada de Skype, el bot recibe automáticamente el vídeo del autor de la llamada (si el bot está habilitado para recibir vídeo principal). En una videollamada grupal en Skype con varios participantes, el bot debe indicar a la plataforma multimedia en tiempo real qué participantes quiere ver. Una suscripción de vídeo es una solicitud del bot para recibir el vídeo de un participante. Mientras los participantes de una videollamada grupal llevan a cabo su conversación, un bot puede modificar sus suscripciones de vídeo deseadas basándose en las actualizaciones del conjunto de interlocutores dominantes.

## <a name="developer-resources"></a>Recursos para el desarrollador 

### <a name="sdks"></a>SDK

Para desarrollar un bot multimedia en tiempo real, debe instalar estos paquetes NuGet dentro de su proyecto de Visual Studio:

- [Bot Builder SDK para .NET](bot-builder-dotnet-overview.md)
- [Bot Builder Real-Time Media Calling for .NET](https://www.nuget.org/packages?q=Bot.Builder.RealTimeMediaCalling)
- [Biblioteca .NET Microsoft.Skype.Bots.Media](https://www.nuget.org/packages?q=Microsoft.Skype.Bots.Media)

### <a name="code-samples"></a>Ejemplos de código

El repositorio de GitHub [BotBuilder-RealTimeMediaCalling](https://github.com/Microsoft/BotBuilder-RealTimeMediaCalling) contiene ejemplos que muestran cómo compilar bots multimedia en tiempo real para Skype.

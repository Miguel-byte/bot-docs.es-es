---
title: Depuración del bot mediante archivos de transcripción | Microsoft Docs
description: Aprenda a usar un archivo de transcripción para ayudar a depurar el bot.
keywords: debugging, faq, transcript file, emulator
author: DanDev33
ms.author: v-dashel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservices: sdk
ms.date: 10/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2c493a27d7829e7c7be21b6ce70cb6f046dad616
ms.sourcegitcommit: 6c719b51c9e4e84f5642100a33fe346b21360e8a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/28/2018
ms.locfileid: "52452097"
---
# <a name="debug-your-bot-using-transcript-files"></a>Depuración del bot mediante archivos de transcripción
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Una de las claves para probar y depurar correctamente un bot es la capacidad para registrar y examinar el conjunto de condiciones que se producen al ejecutar el bot. En este artículo se trata la creación y el uso de un archivo de transcripción de bots para proporcionar un conjunto detallado de interacciones del usuario y respuestas del bot para pruebas y depuración.

## <a name="the-bot-transcript-file"></a>Archivo de transcripción de bot
Un archivo de transcripción de bot es un archivo JSON especializado que conserva las interacciones entre un usuario y el bot. Un archivo de transcripción conserva no solo el contenido de un mensaje, sino también detalles de la interacción como el identificador de usuario, el identificador del canal, el tipo de canal, las funciones de canal, la hora de la interacción, etc. Toda esta información puede usarse para ayudar a encontrar y resolver problemas al probar o depurar el bot. 

## <a name="creatingstoring-a-bot-transcript-file"></a>Creación o almacenamiento de un archivo de transcripción de bot
En este artículo se muestra cómo crear archivos de transcripción de bot mediante [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator) de Microsoft. Los archivos de transcripción también se pueden crear mediante programación; puede leer más sobre este enfoque [aquí](./bot-builder-howto-v4-storage.md#blob-transcript-storage). En este artículo se usará el código de ejemplo de Bot Framework para [EchoBot with Counter](https://aka.ms/EchoBot-With-Counter) que se ha modificado para solicitar el nombre y número de teléfono de un usuario, pero se puede usar cualquier código al que se pueda acceder mediante Bot Framework Emulator de Microsoft para crear un archivo de transcripción.

Para comenzar este proceso, asegúrese de que el código del bot que desea probar se está ejecutando en el entorno de desarrollo. Inicie Bot Framework Emulator, seleccione el botón _Open bot_ (Abrir bot) y conecte el emulador al archivo de _configuración del bot_ del código como se muestra en la imagen siguiente.

![conexión del emulador al código](./media/emulator_open_bot_configuration.png)

Después de conectar el emulador al código de ejecución, pruebe el código enviando interacciones simuladas de usuario al bot. En este ejemplo se ha pasado el nombre y número de teléfono del usuario. Después de haber escrito todas las interacciones de usuario que desea conservar, use Bot Framework Emulator para crear y guardar un archivo de transcripción que contenga esta conversación. 

Dentro de la pestaña _Live Chat_ (Chat en directo) (que se muestra a continuación), seleccione el botón _Save transcript_ (Guardar transcripción). 

![selección de save transcript (guardar transcripción)](./media/emulator_transcript_save.png)

Elija una ubicación y un nombre para el archivo de transcripción y, después, seleccione el botón Save (Guardar).

![transcripción guardada como ursula](./media/emulator_transcript_saveas_ursula.png)

Todas las interacciones del usuario y las respuestas de los bots que introdujo para probar el código con el emulador se han guardado en un archivo de transcripción que puede recargar más tarde para ayudar a depurar las interacciones entre un usuario y el bot.

## <a name="retrieving-a-bot-transcript-file"></a>Recuperación de un archivo de transcripción de bot
Para recuperar un archivo de transcripción de bot usando Bot Framework Emulator, seleccione el control de lista _TRANSCRIPTS_ (Transcripciones) en la sección _RESOURCES_ (Recursos) en la esquina superior izquierda del emulador, como se muestra a continuación. A continuación, seleccione el archivo de transcripción que se va a recuperar. En este ejemplo estamos recuperando el archivo de transcripción denominado "Ursula_User_transcript". Al seleccionar un archivo de transcripción, se cargará automáticamente toda la conversación conservada en una nueva pestaña denominada _Transcript_ (Transcripción).

![recuperación de transcripción guardada](./media/emulator_transcript_retrieve.png)

## <a name="debug-using-transcript-file"></a>Depuración con el archivo de transcripción
Con el archivo de transcripción cargado, ahora está listo para depurar las interacciones que ha capturado entre el usuario y el bot. Para ello, simplemente haga clic en cualquier evento o actividad registrado en la sección _LOG_ (Registro) que se muestra en el área inferior derecha del emulador. En el ejemplo que se muestra a continuación, seleccionamos la primera interacción del usuario cuando este ha enviado el mensaje "Hello". Al hacer esto, toda la información del archivo de transcripción referente a esta interacción específica se muestra en la ventana _INSPECTOR_ del emulador en formato JSON. Si vemos algunos de estos valores de abajo hacia arriba, vemos lo siguiente:
* El tipo de interacción era _mensaje_.
* El tiempo que se envió el mensaje.
* El texto sin formato enviado contiene "Hello".
* El mensaje se envió a nuestro bot.
* El identificador de usuario y la información.
* El identificador del canal, funcionalidades e información.

![depuración mediante la transcripción](./media/emulator_transcript_debug.png)

Este nivel detallado de información le permite seguir paso a paso las interacciones entre la entrada del usuario y la respuesta del bot, lo cual es útil para depurar situaciones en las que el bot no ha respondido de la manera esperada o no ha respondido al usuario en absoluto. Tener estos valores y un registro de los pasos que conducen a la interacción errónea le permite avanzar a través del código, buscar la ubicación donde el bot no responde como se esperaba y resolver esos problemas.

El uso de archivos de transcripción junto con Bot Framework Emulator es solo una de las muchas herramientas que puede utilizar para ayudarle a probar y depurar el código del bot y las interacciones del usuario. Para encontrar más formas de probar y depurar el bot, consulte los recursos adicionales que se muestran a continuación

## <a name="additional-resources"></a>Recursos adicionales
Para obtener información adicional sobre pruebas y depuración, consulte:
* [Directrices de prueba y depuración de los bots](./bot-builder-testing-debugging.md)
* [Depuración con Bot Framework Emulator](../bot-service-debug-emulator.md)
* [Depurar en Visual Studio](https://docs.microsoft.com/en-us/visualstudio/debugger/index)



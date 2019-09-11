---
title: Directrices de depuración | Microsoft Docs
description: Comprenda cómo depurar un bot.
keywords: debugging bots, botframework debugging
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 07/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a04e1acc37c488fcd7530b7df2dd8668d4cfdcc2
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299190"
---
# <a name="debugging-guidelines"></a>Directrices de depuración

[!INCLUDE[applies-to](../includes/applies-to.md)]

Los bots son aplicaciones complejas con una gran cantidad de elementos diferentes que funcionan conjuntamente. Como cualquier otra aplicación compleja, esto puede provocar algunos errores interesantes o que el bot se comporte de forma diferente a la esperada.

A veces, depurar un bot puede ser una tarea difícil. Cada desarrollador tiene su propia manera preferida de realizar esa tarea. Las directrices que se presentan a continuación son sugerencias que puede usar y que se aplican a la gran mayoría de los bots.

Después de comprobar si el bot funciona como le gustaría, el siguiente paso consiste en conectarlo a un canal. Para ello, puede implementar el bot en un servidor de almacenamiento provisional y crear su propio cliente de Direct Line para que el bot se conecte.
<!--IBTODO [Direct Line client](bot-builder-howto-direct-line.md)-->

La creación de su propio cliente le permite definir el funcionamiento interno del canal, así como probar específicamente cómo responde el bot a determinados intercambios de actividad. Una vez conectado al cliente, ejecute las pruebas para configurar el estado del bot y comprobar sus características. Si el bot usa una característica como la voz, el uso de estos canales puede ofrecer una manera de comprobar esa funcionalidad.

El uso del emulador y del Chat en web a través de Azure Portal puede proporcionar más información sobre el rendimiento del bot al interactuar con distintos canales.

La depuración del bot funciona de forma similar a otras aplicaciones multiproceso, con la capacidad de establecer puntos de interrupción o de usar características como la ventana inmediata. 

Los bots siguen un paradigma de programación controlado por eventos, que puede ser difícil de racionalizar si no está familiarizado con él. La idea de que el bot sea sin estado, multiproceso y que se encargue de las llamadas asincrónicas o de espera puede provocar errores inesperados. Aunque la depuración del bot funciona de forma similar a otras aplicaciones multiproceso, incluiremos algunas sugerencias, herramientas y recursos que le ayudarán.

## <a name="understanding-bot-activities-with-the-emulator"></a>Descripción de las actividades del bot con el emulador

El bot se ocupa de diferentes tipos de [actividades](bot-builder-basics.md#the-activity-processing-stack), además de la actividad normal de _mensajes_. Mediante el [emulador](../bot-service-debug-emulator.md), le mostraremos cuáles son esas actividades, cuándo se producen y qué información contienen. La información sobre esas actividades le ayudará a codificar el bot de forma eficaz y le permitirá comprobar si las actividades que el bot envía y recibe son las esperadas.

## <a name="saving-and-retrieving-user-interactions-with-transcripts"></a>Guardado y recuperación de las interacciones del usuario con transcripciones

El almacenamiento de transcripciones de blobs de Azure proporciona un recurso especializado en el que puede [almacenar y recuperar transcripciones](bot-builder-howto-v4-storage.md) que contengan interacciones entre los usuarios y el bot.  

Además, cuando las interacciones de entrada del usuario se han almacenado, puede usar el "_explorador de almacenamiento_" de Azure para ver manualmente los datos contenidos en las transcripciones almacenadas dentro del almacén de transcripciones de blobs. En el siguiente ejemplo se abre el "_explorador de almacenamiento_" de la configuración de "_mynewtestblobstorage_". Para abrir los datos de entrada del usuario guardados, seleccione:    Contenedor de blob > ChannelId > TranscriptId > ConversationId

![Examine_stored_transcript_text](./media/examine_transcript_text_in_azure.png)

Abre la entrada de conversación de usuario almacenada en formato JSON. La entrada de usuario se conserva junto con la clave "_text:_ ".

## <a name="how-middleware-works"></a>Funcionamiento del software intermedio

El [software intermedio](bot-builder-concept-middleware.md) puede no ser intuitivo al intentar usarlo por primera vez, especialmente con respecto a la continuación o al cortocircuito de la ejecución. El software intermedio puede ejecutarse en el borde inicial o final de un turno, con una llamada al delegado `next()` que dicte cuándo se pasa la ejecución a la lógica del bot. 

Si usa varios fragmentos de software intermedio, el delegado puede pasar la ejecución a otro fragmento de software intermedio si es así cómo está orientada la canalización. La información sobre [la canalización del software intermedio del bot](bot-builder-concept-middleware.md#the-bot-middleware-pipeline) puede ayudar a entender esta idea.

Si no se llama al delegado `next()`, se denomina [enrutamiento de cortocircuito](bot-builder-concept-middleware.md#short-circuiting). Esto sucede cuando el software intermedio satisface la actividad actual y determina que no es necesario pasar la ejecución. 

Comprender cuándo y por qué se produce el cortocircuito del software intermedio ayuda a indicar qué fragmento del software intermedio debe aparecer en primer lugar en la canalización. Además, la comprensión de lo que se puede esperar es especialmente importante para el software intermedio integrado que proporciona el SDK u otros desarrolladores. A algunos usuarios les resulta útil intentar crear su propio software intermedio primero para experimentar un poco antes de profundizar en el software intermedio integrado.

<!-- Snip: QnA was once implemented as middleware.
For example [QnA maker](bot-builder-howto-qna.md) is designed to handle certain interactions and short-circuit the pipeline when it does, which can be confusing when first learning how to use it.
-->

## <a name="understanding-state"></a>Información sobre el estado

El seguimiento del estado es una parte importante del bot, especialmente para las tareas complejas. En general, el procedimiento recomendado consiste en procesar actividades tan rápidamente como sea posible y permitir que el procesamiento se complete para que se conserve el estado. Las actividades se pueden enviar al bot casi al mismo tiempo, lo que puede provocar errores muy confusos debido a la arquitectura asincrónica.

Lo más importante es asegurarse de conservar el estado de forma que coincida con sus expectativas. En función de dónde se encuentre el estado conservado, los emuladores de almacenamiento de [Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/local-emulator) y [Azure Table Storage](https://docs.microsoft.com/azure/storage/common/storage-use-emulator) pueden ayudarle a comprobar dicho estado antes de usar el almacenamiento de producción.

## <a name="how-to-use-activity-handlers"></a>Uso de los controladores de actividad

Los controladores de actividad pueden introducir otro nivel de complejidad, específicamente que cada actividad se ejecute en un subproceso independiente (o roles de trabajo, dependiendo del lenguaje). En función de lo que hagan los controladores, esto puede causar problemas donde el estado actual no es el esperado.

El estado integrado se escribe al final de un turno, sin embargo, las actividades generadas por dicho turno se ejecutan independientemente de la canalización del turno. Con frecuencia esto no nos afecta, pero, si un controlador de actividad cambia de estado, es necesario que el estado escrito contenga ese cambio. En ese caso, la canalización del turno puede esperar a que la actividad finalice el procesamiento antes de completarse para garantizar que registra el estado correcto del turno.

El método _send activity_ y sus controladores plantean un problema único. Un simple llamada a _send activity_ desde el controlador _on send activities_ provoca una bifurcación infinita de subprocesos. Puede solucionar ese problema de diferentes formas, como anexando el mensaje adicional a la información saliente o escribiendo en otra ubicación, como la consola o un archivo, para evitar el bloqueo del bot.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Cómo hacer pruebas unitarias de bots](unit-test-bots.md)

## <a name="additional-resources"></a>Recursos adicionales

* [Depurar en Visual Studio](https://docs.microsoft.com/visualstudio/debugger/index)
* [Depurar, trazar y generar perfiles](https://docs.microsoft.com/dotnet/framework/debug-trace-profile/) para Bot Framework
* Usar [ConditionalAttribute](https://docs.microsoft.com/dotnet/api/system.diagnostics.conditionalattribute?view=netcore-2.0) para métodos que no quiere incluir en el código de producción
* Usar herramientas como [Fiddler](https://www.telerik.com/fiddler) para ver el tráfico de red
* [Solución de problemas generales](../bot-service-troubleshoot-bot-configuration.md) y otros artículos de solución de problemas en esa sección.

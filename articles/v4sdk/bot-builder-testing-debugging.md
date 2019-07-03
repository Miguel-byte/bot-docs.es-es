---
title: Directrices de prueba y depuración | Microsoft Docs
description: Comprenda cómo probar y depurar el bot.
keywords: principios de prueba, elementos ficticios, preguntas más frecuentes, niveles de prueba
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7b3380a409641bd7d406c877ebcbf83ddf15431c
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404448"
---
# <a name="testing-and-debugging-guidelines"></a>Directrices de prueba y depuración

[!INCLUDE[applies-to](../includes/applies-to.md)]

Los bots son aplicaciones complejas con una gran cantidad de elementos diferentes que funcionan conjuntamente. Como cualquier otra aplicación compleja, esto puede provocar algunos errores interesantes o que el bot se comporte de forma diferente a la esperada.

A veces, la prueba del bot y posteriormente su depuración puede ser una tarea difícil. Cada desarrollador tiene su propia manera preferida de realizar esa tarea. Las directrices que se presentan a continuación son sugerencias que puede usar y que se aplican a la gran mayoría de los bots.

## <a name="testing-your-bot"></a>Prueba del bot

Las directrices siguientes se presentan en tres **niveles** diferentes.  Cada nivel agrega complejidad y características a la prueba, por lo que le sugerimos que se conforme con un nivel antes de pasar al siguiente. De esta forma, podrá aislar y solucionar los problemas de nivel más bajo en primer lugar antes de agregar más complejidad.

Los procedimientos recomendados de la prueba tratarán diferentes ángulos siempre que sea aplicable. Pueden incluir seguridad, integración, direcciones URL con formato incorrecto, vulnerabilidades de seguridad de validación, códigos de estado HTTP, cargas JSON, valores null, etc. Si el bot se encarga de toda la información que afecta a la privacidad del usuario, esto es especialmente importante.

### <a name="level-1-use-mock-elements"></a>Nivel 1: uso de elementos ficticios

Lo que garantiza que cada fragmento de la aplicación, o bien, en este caso nuestro bot, funcione exactamente como debería, es el primer nivel de pruebas. Para lograr esto, puede usar elementos ficticios para las cosas que no está probando actualmente. Como referencia, este nivel puede considerarse generalmente como unidad y prueba de integración.

#### <a name="use-mock-elements-to-test-individual-sections"></a>Uso de elementos ficticios para probar secciones individuales

La simulación de tantos elementos como pueda permite un mejor aislamiento de la pieza que está probando. Los candidatos para los elementos ficticios incluyen el almacenamiento, el adaptador, el software intermedio, la canalización de actividades, los canales y cualquier otra cosa que no forme parte del bot directamente. También se podrían quitar ciertos aspectos temporalmente, como el software intermedio no implicado en la parte del bot que está probando, para aislar cada fragmento. Sin embargo, si va a probar su software intermedio, es posible que quiera simular el bot en su lugar.

La simulación de elementos puede asumir formas diferentes, desde el reemplazo de un elemento por otro objeto conocido a la implementación de una funcionalidad básica de Hola mundo. También puede consistir en la eliminación del elemento simplemente, en caso de que no sea necesario, así como en forzarlo a no hacer nada. 

Este método debe ejercitar funciones y métodos individuales en el bot. La prueba de métodos individuales se puede realizar a través de pruebas unitarias integradas, lo cual se recomienda, con su propia aplicación de prueba o conjunto de pruebas, o bien de forma manual si lo hace dentro del IDE. 

#### <a name="use-mock-elements-to-test-larger-features"></a>Uso de elementos ficticios para probar características mayores

Cuando esté satisfecho con el comportamiento de cada método, use estos elementos ficticios para probar características más completas en el bot. Esto demuestra como unas pocas capas funcionan conjuntamente para conversar con el usuario. 

Para ayudarle, se proporcionan diferentes herramientas. Por ejemplo, [Azure Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator) proporciona un canal emulado para comunicarse con el bot. Mediante el emulador, se obtiene una situación más compleja que la unidad y las pruebas de integración y, por lo tanto, también se pasa al siguiente nivel de pruebas.

### <a name="level-2-use-a-direct-line-client"></a>Nivel 2: Uso de un cliente de Direct Line

Después de comprobar si el bot funciona como le gustaría, el siguiente paso consiste en conectarlo a un canal. Para ello, puede implementar el bot en un servidor de almacenamiento provisional y crear su propio cliente de Direct Line para que el bot se conecte.
<!--IBTODO [Direct Line client](bot-builder-howto-direct-line.md)-->

La creación de su propio cliente le permite definir el funcionamiento interno del canal, así como probar específicamente cómo responde el bot a determinados intercambios de actividad. Una vez conectado al cliente, ejecute las pruebas para configurar el estado del bot y comprobar sus características. Si el bot usa una característica como la voz, el uso de estos canales puede ofrecer una manera de comprobar esa funcionalidad.

El uso del emulador y del Chat en web a través de Azure Portal puede proporcionar más información sobre el rendimiento del bot al interactuar con distintos canales.

### <a name="level-3-channel-tests"></a>Nivel 3: pruebas del canal

Una vez que esté seguro del rendimiento independiente del bot, es importante ver cómo funciona con distintos canales en los que estará disponible. 

La forma de conseguirlo puede variar considerablemente, desde su uso individual con distintos canales y exploradores al uso con una herramienta de terceros, como [Selenium](https://docs.seleniumhq.org/), para interactuar a través de un canal y extraer las respuestas del bot.

### <a name="other-testing"></a>Otras pruebas

Los diferentes tipos de pruebas pueden realizarse junto con los niveles anteriores o desde distintos ángulos, como pruebas de esfuerzo, pruebas de rendimiento o generación de perfiles de la actividad del bot. Visual Studio proporciona métodos para hacerlo de manera local, así como un [conjunto de herramientas](https://azure.microsoft.com/solutions/dev-test/) para probar la aplicación, mientras que [Azure Portal](https://portal.azure.com) proporciona información acerca del rendimiento del bot.

## <a name="debugging"></a>Depuración

La depuración del bot funciona de forma similar a otras aplicaciones multiproceso, con la capacidad de establecer puntos de interrupción o de usar características como la ventana inmediata. 

Los bots siguen un paradigma de programación controlado por eventos, que puede ser difícil de racionalizar si no está familiarizado con él. La idea de que el bot sea sin estado, multiproceso y que se encargue de las llamadas asincrónicas o de espera puede provocar errores inesperados. Aunque la depuración del bot funciona de forma similar a otras aplicaciones multiproceso, incluiremos algunas sugerencias, herramientas y recursos que le ayudarán.

### <a name="understanding-bot-activities-with-the-emulator"></a>Descripción de las actividades del bot con el emulador

El bot se ocupa de diferentes tipos de [actividades](bot-builder-basics.md#the-activity-processing-stack), además de la actividad normal de _mensajes_. Mediante el [emulador](../bot-service-debug-emulator.md), le mostraremos cuáles son esas actividades, cuándo se producen y qué información contienen. La información sobre esas actividades le ayudará a codificar el bot de forma eficaz y le permitirá comprobar si las actividades que el bot envía y recibe son las esperadas.

### <a name="saving-and-retrieving-user-interactions-with-transcripts"></a>Guardado y recuperación de las interacciones del usuario con transcripciones

El almacenamiento de transcripciones de blobs de Azure proporciona un recurso especializado en el que puede [almacenar y recuperar transcripciones](bot-builder-howto-v4-storage.md) que contengan interacciones entre los usuarios y el bot.  

Además, cuando las interacciones de entrada del usuario se han almacenado, puede usar el "_explorador de almacenamiento_" de Azure para ver manualmente los datos contenidos en las transcripciones almacenadas dentro del almacén de transcripciones de blobs. En el siguiente ejemplo se abre el "_explorador de almacenamiento_" de la configuración de "_mynewtestblobstorage_". Para abrir los datos de entrada del usuario guardados, seleccione:    Contenedor de blob > ChannelId > TranscriptId > ConversationId

![Examine_stored_transcript_text](./media/examine_transcript_text_in_azure.png)

Abre la entrada de conversación de usuario almacenada en formato JSON. La entrada de usuario se conserva junto con la clave "_text:_ ".

### <a name="how-middleware-works"></a>Funcionamiento del software intermedio

El [software intermedio](bot-builder-concept-middleware.md) puede no ser intuitivo al intentar usarlo por primera vez, especialmente con respecto a la continuación o al cortocircuito de la ejecución. El software intermedio puede ejecutarse en el borde inicial o final de un turno, con una llamada al delegado `next()` que dicte cuándo se pasa la ejecución a la lógica del bot. 

Si usa varios fragmentos de software intermedio, el delegado puede pasar la ejecución a otro fragmento de software intermedio si es así cómo está orientada la canalización. La información sobre [la canalización del software intermedio del bot](bot-builder-concept-middleware.md#the-bot-middleware-pipeline) puede ayudar a entender esta idea.

Si no se llama al delegado `next()`, se denomina [enrutamiento de cortocircuito](bot-builder-concept-middleware.md#short-circuiting). Esto sucede cuando el software intermedio satisface la actividad actual y determina que no es necesario pasar la ejecución. 

Comprender cuándo y por qué se produce el cortocircuito del software intermedio ayuda a indicar qué fragmento del software intermedio debe aparecer en primer lugar en la canalización. Además, la comprensión de lo que se puede esperar es especialmente importante para el software intermedio integrado que proporciona el SDK u otros desarrolladores. A algunos usuarios les resulta útil intentar crear su propio software intermedio primero para experimentar un poco antes de profundizar en el software intermedio integrado.

<!-- Snip: QnA was once implemented as middleware.
For example [QnA maker](bot-builder-howto-qna.md) is designed to handle certain interactions and short-circuit the pipeline when it does, which can be confusing when first learning how to use it.
-->

### <a name="understanding-state"></a>Información sobre el estado

El seguimiento del estado es una parte importante del bot, especialmente para las tareas complejas. En general, el procedimiento recomendado consiste en procesar actividades tan rápidamente como sea posible y permitir que el procesamiento se complete para que se conserve el estado. Las actividades se pueden enviar al bot casi al mismo tiempo, lo que puede provocar errores muy confusos debido a la arquitectura asincrónica.

Lo más importante es asegurarse de conservar el estado de forma que coincida con sus expectativas. En función de dónde se encuentre el estado conservado, los emuladores de almacenamiento de [Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/local-emulator) y [Azure Table Storage](https://docs.microsoft.com/azure/storage/common/storage-use-emulator) pueden ayudarle a comprobar dicho estado antes de usar el almacenamiento de producción.

### <a name="how-to-use-activity-handlers"></a>Uso de los controladores de actividad

Los controladores de actividad pueden introducir otro nivel de complejidad, específicamente que cada actividad se ejecute en un subproceso independiente (o roles de trabajo, dependiendo del lenguaje). En función de lo que hagan los controladores, esto puede causar problemas donde el estado actual no es el esperado.

El estado integrado se escribe al final de un turno, sin embargo, las actividades generadas por dicho turno se ejecutan independientemente de la canalización del turno. Con frecuencia esto no nos afecta, pero, si un controlador de actividad cambia de estado, es necesario que el estado escrito contenga ese cambio. En ese caso, la canalización del turno puede esperar a que la actividad finalice el procesamiento antes de completarse para garantizar que registra el estado correcto del turno.

El método _send activity_ y sus controladores plantean un problema único. Un simple llamada a _send activity_ desde el controlador _on send activities_ provoca una bifurcación infinita de subprocesos. Puede solucionar ese problema de diferentes formas, como anexando el mensaje adicional a la información saliente o escribiendo en otra ubicación, como la consola o un archivo, para evitar el bloqueo del bot.

## <a name="additional-resources"></a>Recursos adicionales

* [Depurar en Visual Studio](https://docs.microsoft.com/visualstudio/debugger/index)
* [Depurar, trazar y generar perfiles](https://docs.microsoft.com/dotnet/framework/debug-trace-profile/) para Bot Framework
* Usar [ConditionalAttribute](https://docs.microsoft.com/dotnet/api/system.diagnostics.conditionalattribute?view=netcore-2.0) para métodos que no quiere incluir en el código de producción
* Usar herramientas como [Fiddler](https://www.telerik.com/fiddler) para ver el tráfico de red
* [Repositorio de herramientas de bot](https://github.com/Microsoft/botbuilder-tools)
* Los marcos pueden ayudar con las pruebas, como [Moq](https://github.com/moq/moq4)
* [Solución de problemas generales](../bot-service-troubleshoot-bot-configuration.md) y otros artículos de solución de problemas en esa sección.

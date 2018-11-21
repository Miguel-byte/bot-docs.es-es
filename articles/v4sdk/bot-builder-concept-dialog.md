---
title: Diálogos en el SDK Bot Builder | Microsoft Docs
description: Se describe qué es un diálogo y cómo funciona en el SDK Bot Builder.
keywords: flujo de conversación, intención de reconocimiento, turno único, varios turnos, conversación de bot, diálogos, avisos, cascadas, conjunto de diálogos
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 9/22/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 88022c387d5f9ef7f645be74010aba3c676efadc
ms.sourcegitcommit: cb0b70d7cf1081b08eaf1fddb69f7db3b95b1b09
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/09/2018
ms.locfileid: "51332939"
---
# <a name="dialogs-library"></a>Biblioteca de diálogos

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

La administración de conversaciones a través del concepto de diálogo es fundamental para el SDK. Los objetos de diálogo procesan las actividades de entrada y generan las respuestas de salida. La lógica de negocios del bot se ejecuta directa o indirectamente dentro de clases de diálogo.

En tiempo de ejecución, las instancias de diálogo se organizan en una pila. El diálogo en la parte superior de la pila se conoce como ActiveDialog. El diálogo activo actual procesa la actividad de entrada. La pila se guarda entre cada turno de la conversación (que no está vinculado al tiempo y puede durar varios días). 

## <a name="dialog-lifecycle"></a>Ciclo de vida de los diálogos

Un diálogo implementa tres funciones principales:
- BeginDialog
- ContinueDialog
- ResumeDialog

En tiempo de ejecución, los diálogos y la clase DialogContext funcionan juntos para elegir el diálogo adecuado para gestionar la actividad. La clase DialogContext enlaza la pila de diálogos guardada, la actividad de entrada y la clase DialogSet. La clase DialogSet contiene diálogos que el bot puede llamar.

La interfaz de DialogContext refleja el concepto subyacente del comienzo y la continuación del diálogo. El patrón general para la aplicación es llamar siempre primero a ContinueDialog. Si no hay ninguna pila y, por tanto, no hay ningún diálogo ActiveDialog, la aplicación debe comenzar el diálogo que elija llamando a BeginDialog en DialogContext. De esta manera, la entrada de diálogo correspondiente de DialogSet se inserta en la pila (técnicamente es el identificador del diálogo que se agrega a la pila) y se delega entonces una llamada en BeginDialog en el objeto de diálogo específico. Si hubiera habido un diálogo ActiveDialog, simplemente se habría delegado la llamada en la función ContinueDialog de ese diálogo en el procesamiento y se le habría dado a este las propiedades guardadas asociadas.

Tenga en cuenta que una **función BeginDialog del diálogo** es código de inicialización y toma las propiedades de inicialización (llamadas "opciones" en el código) y que una **función ContinueDialog del diálogo** es el código que se ejecuta para continuar con la ejecución a la llegada de una actividad que sigue a la persistencia. Por ejemplo, imagine un diálogo que le hace una pregunta al usuario; la pregunta se haría en BeginDialog y la respuesta se esperaría en ContinueDialog.

Para permitir el anidamiento de diálogos (donde un diálogo tiene diálogos secundarios), hay un tipo adicional de continuación, que se conoce como reanudación. DialogContext llamará al método ResumeDialog en un diálogo principal cuando finalice un diálogo secundario.

Los avisos y las cascadas son ambos ejemplos concretos de diálogos que proporciona el SDK. Son muchos los escenarios que se crean mediante la composición de estas abstracciones; pero, en segundo plano, la lógica ejecutada es siempre el mismo comienzo, es decir, el patrón de continuación y reanudación aquí descrito. La implementación de una clase de diálogo desde el principio es un tema relativamente avanzado, pero se incluye una muestra en los [ejemplos](https://github.com/Microsoft/BotBuilder-samples).

La biblioteca **Dialog** del SDK Bot Builder incluye características integradas como _avisos_, _diálogos de cascada_ y _diálogos de componentes_ para ayudarle a administrar la conversación del bot. Puede usar avisos para pedir a los usuarios diferentes tipos de información, una cascada para combinar varios pasos en una secuencia y diálogos de componentes para empaquetar la lógica de diálogo en clases independientes que luego se pueden integrar en otros bots.
## <a name="waterfall-dialogs-and-prompts"></a>Diálogos y avisos de cascada

La biblioteca **Dialog** incluye un conjunto de tipos de avisos que puede usar para recopilar diversos tipos de entradas de usuario. Por ejemplo, para pedir a un usuario la entrada de texto, puede usar **TextPrompt**; para pedir al usuario un número, puede usar **NumberPrompt**; y para pedir una fecha y una hora, puede usar **DateTimePrompt**. Loa avisos son un tipo especial de diálogo. Para usar un aviso de un diálogo de cascada, agregue la cascada y el aviso al mismo conjunto de diálogos. 

Dada la naturaleza de la interacción solicitud-respuesta, para implementar un aviso se requieren al menos dos pasos en un diálogo de cascada: uno para enviar el aviso y otro para capturar y procesar la respuesta.  Si tiene un aviso adicional, en ocasiones puede combinar estos pasos mediante una única función para procesar primero la respuesta del usuario y luego iniciar el siguiente aviso.

`WaterfallDialog` es una implementación específica de un diálogo que se usa para recopilar información del usuario o guiar al usuario por una serie de tareas. Las tareas se implementan como una matriz de funciones, donde el resultado de la primera función se pasa como argumento a la función siguiente y así sucesivamente. Cada función representa normalmente un paso del proceso general. En cada paso, el bot pide al usuario que intervenga, espera una respuesta y, a continuación, pasa el resultado al paso siguiente. 

Los avisos y las cascadas son ambos diálogos, como se muestra en la siguiente jerarquía de clases. 

![clases de diálogo](media/bot-builder-dialog-classes.png)

Un diálogo de cascada se compone de una secuencia de pasos de cascada. Cada paso es un delegado asincrónico que toma un parámetro de _contexto de paso de cascada_ (`step`). El patrón es que lo último que se haga en un paso de la cascada sea comenzar un diálogo secundario (normalmente un aviso) o finalizar la cascada propiamente dicha. En el diagrama siguiente se muestra una secuencia de pasos de cascada y las operaciones de la pila que tienen lugar.

![Concepto de diálogo](media/bot-builder-dialog-concept.png)

Puede controlar un valor devuelto por un diálogo, bien dentro de un paso de la cascada de un diálogo o desde el controlador de turnos del bot.
Dentro de un paso de la cascada, el diálogo proporciona el valor devuelto en la propiedad _result_ del contexto del paso.
Por lo general, solo será necesario comprobar el estado del resultado de turnos del diálogo desde la lógica de turnos del bot.

## <a name="repeating-a-dialog"></a>Repetición de un diálogo

Para repetir un diálogo, use el método *replace dialog*. El método *replace dialog* del contexto de diálogo retira el diálogo actual de la pila, inserta el diálogo sustituto en la parte superior de la pila y comienza dicho diálogo. Puede usar este método para crear un bucle al reemplazar un diálogo por sí mismo. Tenga en cuenta que si tiene que conservar el estado interno del diálogo actual, deberá pasar información a la nueva instancia del diálogo en la llamada al método _replace dialog_ y, luego, inicializar el diálogo de la manera adecuada. Para acceder a las opciones pasadas al nuevo diálogo se puede usar la propiedad _options_ del contexto en cualquier paso del diálogo. Esta es una excelente manera de controlar un flujo de conversación complejo o de administrar los menús.

## <a name="branch-a-conversation"></a>Bifurcación de una conversación

El contexto del diálogo mantiene una _pila de diálogos_ y, para cada diálogo de la pila, lleva el seguimiento de cuál es el siguiente paso. Su método _begin dialog_ inserta un diálogo en la parte superior de la pila y su método _end dialog_ retira el diálogo superior de la pila.

Para iniciar un nuevo diálogo dentro del mismo conjunto de diálogos, un diálogo puede llamar al método _begin dialog_ del contexto del diálogo o proporcionar el identificador del nuevo diálogo, lo que convierte a este en el diálogo actualmente activo. El diálogo original está todavía en la pila, pero las llamadas al método _continue_ del contexto del diálogo solo se envían al diálogo que está arriba de la pila, el _diálogo activo_. Cuando un diálogo se retira de la pila, el contexto del diálogo se reanuda con el siguiente paso de la cascada en la pila donde se dejó el diálogo original.

Por lo tanto, puede crear una bifurcación dentro del flujo de conversación mediante la inclusión de un paso en un diálogo que puede elegir condicionalmente un diálogo para iniciar de un conjunto de diálogos disponibles.

## <a name="component-dialog"></a>Diálogo de componente
En ocasiones, querrá escribir un diálogo reutilizable para usar en distintos escenarios. Un ejemplo podría ser un diálogo de dirección que pide al usuario que proporcione valores para calle, ciudad y código postal. 

ComponentDialog proporciona un nivel de aislamiento porque tiene una clase DialogSet independiente. Al tener una clase DialogSet independiente, se evitan colisiones de nombres con el elemento principal que contiene el diálogo y se crea un entorno propio de ejecución de diálogos interno independiente (al crearse su propio DialogContext), al que se distribuye la actividad. Esta distribución secundaria significa que ha tenido la oportunidad de interceptar la actividad. Esto puede ser muy útil si quiere implementar características como "ayuda" y "cancelar".  Consulte el ejemplo de plantilla de bot de empresa. 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Uso de la biblioteca de diálogos para recopilar datos de entrada del usuario](bot-builder-prompts.md)

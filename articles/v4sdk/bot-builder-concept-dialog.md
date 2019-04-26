---
title: Diálogos en Bot Framework SDK | Microsoft Docs
description: Se describe qué es un diálogo y cómo funciona en Bot Framework SDK.
keywords: flujo de conversación, aviso, estado de diálogo, intención de reconocimiento, turno único, varios turnos, conversación de bot, diálogos, avisos, cascadas, conjunto de diálogos
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/18/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 36ccbb796c2cd014118d4ae1f426acd44aabed76
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904898"
---
# <a name="dialogs-library"></a>Biblioteca de diálogos

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

Los *diálogos* son un concepto central del SDK y proporcionan una forma útil de administrar conversaciones con los usuarios. Los diálogos son estructuras de un bot que actúan como funciones en el programa del bot; cada diálogo está diseñado para realizar una tarea específica, en un orden concreto. Puede especificar el orden de los diálogos individuales para guiar la conversación e invocarlos de maneras diferentes (unas veces, en respuesta a un usuario y otras en respuesta a estímulos externos o de otros diálogos).

La biblioteca de diálogos proporciona algunas características integradas, como *avisos* y *diálogos en cascada*, para facilitar la administración de la conversación del bot. Los [avisos](#prompts) se usan para solicitar distintos tipos de información, como texto, un número o una fecha. Los [diálogos en cascada](#waterfall-dialogs) puede combinar varios pasos en una sola secuencia, lo que permite al bot seguir fácilmente dicha secuencia predefinida y pasar información al paso siguiente.

<!-- When we have samples for building your own, add links and one liner about them -->

## <a name="dialogs-and-their-pieces"></a>Los diálogos y sus elementos

La biblioteca de diálogos tiene algunos elementos adicionales que hacen que sean más útiles. Además de los diferentes [tipos de diálogos](#dialog-types) que se tratan a continuación, la biblioteca contiene la idea de un *conjunto de diálogos*, el *contexto del diálogo*y el *resultado del diálogo* .

Los *conjuntos de diálogos* son, en su forma más simple, una colección de diálogos. Puede ser elementos como avisos, diálogos en cascadas o [diálogos de componentes](#component-dialog). Cada una de estas implementaciones de un diálogo y cada elemento de este se agrega al conjunto de diálogos con un identificador de cadena concreto. Cuando un bot desea iniciar un diálogo o un aviso determinado del conjunto de diálogos, utiliza el identificador de cadena para especificar el diálogo que se va a usar.

El *contexto del diálogo* contiene información que pertenece al diálogo y se usa para interactuar con un cuadro de diálogos desde dentro del controlador de turnos del bot. El contexto del diálogo incluye el contexto del turno actual, el diálogo principal y el [estado del diálogo](#dialog-state), que proporciona un método para conservar información en el diálogo. El contexto del diálogo permite iniciar un diálogo con su identificador de cadena o continuar el diálogo actual (por ejemplo, un diálogo en cascada que tiene varios pasos).

Cuando un diálogo finaliza, puede devolver un *resultado del diálogo* con información del diálogo. Dicha devolución se realiza para que el método de llamada pueda ver lo que sucedió en el diálogo y guardar la información en una ubicación persistente, si se desea.

## <a name="dialog-state"></a>Estado del diálogo

Los diálogos son un método de implementación de una conversación con varios turnos y, como tal, son un ejemplo de una característica del SDK que se basa en un estado persistente en varios turnos. Sin el estado de los diálogos el bot en qué parte del conjunto de diálogos se encuentra ni la información que ya ha recopilado.

Un bot basado en diálogos contiene normalmente una colección de conjuntos de diálogos como una variable de miembro en su implementación. Ese conjunto de diálogos se crea con un controlador para un objeto llamado descriptor de acceso que proporciona acceso a un estado persistente. Para más información sobre el estado dentro de los bots, consulte [Administración del estado](bot-builder-concept-state.md).

En el controlador del turnos del bot, este inicializa el subsistema del diálogo llamando a *create context* en el conjunto de diálogos, lo que devuelve un *contexto de diálogo*. Ese contexto de diálogo contiene la información necesaria que necesita el diálogo.

La creación de un contexto de diálogo requiere un estado, al que se accede con el descriptor de acceso proporcionado al crear el conjunto de diálogos. Con dicho descriptor de acceso el conjunto de diálogos puede obtener el estado de diálogo apropiado. Los detalles sobre los descriptores de acceso de estado se encuentran en [Guardado de los datos de usuario y la conversación](bot-builder-howto-v4-state.md).

## <a name="dialog-types"></a>Tipos de diálogo

Los diálogos pueden ser de varios tipos: avisos, diálogos en cascada y diálogos de componentes, como se muestra en esta jerarquía de clases.

![clases de diálogo](media/bot-builder-dialog-classes.png)

### <a name="prompts"></a>Mensajes

Los avisos, en la biblioteca de diálogos, proporcionan una forma fácil de pedir información al usuario y evaluar su respuesta. Por ejemplo, en el caso de un *aviso de número*, especifique la pregunta o la información que desee y el aviso comprobará si ha recibido una respuesta numérica válida. En caso afirmativo, la conversación puede continuar; en caso negativo, volverá a pedir al usuario una respuesta válida.

En un segundo plano, las preguntas consisten en un diálogo de dos pasos. En primer lugar, el aviso pide información y, en segundo lugar, devuelve el valor válido o se inicia desde el principio con un nuevo aviso.

A los avisos se les proporcionan *opciones de aviso* cuando se llama al aviso, que es el lugar en que se puede especificar el texto con el que se pide, el aviso de reintento si se produce un error de validación y las distintas opciones para responder al aviso.

Además, puede agregar una validación personalizada para el aviso al crearlo. Por ejemplo, supongamos que deseamos conocer el tamaño de una entidad mediante el símbolo numérico, pero dicho tamaño debe ser superior a 2 e inferior a 12. En primer lugar, el aviso comprueba si ha recibido un número válido y, después, se ejecuta la validación personalizada, en caso de que se haya proporcionado. Si se produce un error en la validación personalizada, volverá a pedir el usuario como antes.

Cuando se completa un aviso, este devuelve explícitamente el valor resultante que se ha pedido. Cuando se devuelve dicho valor, podemos tener la certeza de que ha pasado la validación de aviso integrada y cualquier validación personalizada adicional que se pueda haber proporcionado.

Para ver ejemplos acerca del uso de varios avisos, eche un vistazo a cómo usar la [biblioteca de diálogos para recopilar la entrada de usuario](bot-builder-prompts.md).

#### <a name="prompt-types"></a>Tipos de avisos

En un segundo plano, las preguntas consisten en un diálogo de dos pasos. En el primero, la pregunta solicita información. En el segundo, devuelve el valor válido, o se reinicia desde el principio con una nueva pregunta. La biblioteca de diálogos ofrece varios tipos de preguntas básicas, y cada uno de ellos se usa para recopilar un tipo de respuesta diferente. Las preguntas básicas pueden interpretar entradas en lenguaje natural como, por ejemplo, "diez" o "una docena" para un número, o "mañana" o "el viernes a las 10 am" para una fecha y hora.

| Prompt | DESCRIPCIÓN | Devuelve |
|:----|:----|:----|
| _Solicitud de archivos adjuntos_ | Pide uno o varios archivos adjuntos como, por ejemplo, documentos o imágenes. | Una colección de objetos _adjuntos_. |
| _Solicitud de elección_ | Pide que se realice una elección entre las diversas opciones de un conjunto. | Un objeto de _opción seleccionada_. |
| _Solicitud de confirmación_ | Solicita una confirmación. | Valor booleano. |
| _Solicitud de fecha y hora_ | Pregunta una fecha y una hora. | Una colección de objetos de _resolución de fecha y hora_. |
| _Solicitud de número_ | Solicita un número. | Un valor numérico. |
| _Solicitud de texto_ | Solicita la entrada de texto general. | Una cadena. |

Para solicitar una entrada al usuario, defina una pregunta mediante una de las clases integradas, como la _pregunta de texto_, y agréguela al conjunto de diálogos. Las preguntas tienen identificadores fijos que deben ser únicos dentro de un conjunto de diálogos. Puede tener un validador personalizado para cada pregunta y, para algunas preguntas, puede especificar una _configuración regional predeterminada_. 

#### <a name="prompt-locale"></a>Configuración regional de la pregunta

La configuración regional se usa para determinar el comportamiento específico del idioma de las solicitudes de **elección**, **confirmación**, **fecha y hora** y **número**. Para cualquier entrada del usuario, si el canal ha proporcionado una propiedad de _configuración regional_ en el mensaje del usuario, se usará esa. En caso contrario, si se establece la _configuración regional predeterminada_ de la pregunta, ya sea proporcionándola al llamar al constructor de la misma o estableciéndola posteriormente, esa será la que se use. Si no se proporciona ninguna de las dos, se utiliza el inglés ("en-us") como configuración regional. Nota: La configuración regional consiste en un código ISO 639 de 2, 3 o 4 caracteres que representa un idioma o familia de idiomas.

### <a name="waterfall-dialogs"></a>Diálogos en cascada

Un diálogo en cascada es una implementación concreta de un diálogo que normalmente se usa para recopilar información del usuario o guiarlo en una serie de tareas. Cada paso de la conversación se implementa como una función asincrónica que toma un parámetro del *contexto de pasos en cascada*, (`step`). En cada paso, el bot [pide al usuario que intervenga](bot-builder-prompts.md) (o puede empezar un diálogo secundario, que suele ser un aviso). espera una respuesta y, a continuación, pasa el resultado al paso siguiente. El resultado de la primera función se pasa como argumento a la función siguiente y así sucesivamente.

En el diagrama siguiente se muestra una secuencia de pasos de cascada y las operaciones de la pila que tienen lugar. En la sección acerca del [uso de diálogos](#using-dialogs) encontrará más información acerca del uso de la pila de diálogos.

![Concepto de diálogo](media/bot-builder-dialog-concept.png)

En los pasos en cascada, el contexto del diálogo en cascada se almacena en su *contexto de paso en cascada*. Este contexto es similar al contexto de diálogo, ya que proporciona acceso al estado y contexto del turno actual. Use el contexto de ambiente del paso de cascada para interactuar con un conjunto de diálogos desde un paso de cascada.

Puede controlar un valor devuelto de un diálogo, ya sea en un paso de una cascada de un diálogo o desde el controlador del turnos del bot, aunque generalmente solo necesita comprobar el estado del resultado de turnos del diálogo desde la lógica de turnos del bot.
Dentro de un paso de la cascada, el diálogo proporciona el valor devuelto en la propiedad _result_ del contexto del paso.

#### <a name="waterfall-step-context-properties"></a>Propiedades del contexto de un paso de una cascada

El contexto de un paso de una cascada contiene lo siguiente:

* *Opciones*: contiene información de entrada del diálogo.
* *Valores*: contiene información que se puede agregar al contexto y que se transmite a los pasos siguientes.
* *Resultado*: contiene el resultado del paso anterior.

Además, el *siguiente* método continúa hasta el paso siguiente del diálogo en cascada en el mismo turno, lo que permite que el bot omita un paso determinado si fuera necesario.

### <a name="component-dialog"></a>Diálogo de componente

A veces desea escribir un diálogo reutilizable que desea utilizar en diferentes escenarios, como un diálogo de dirección que pide al usuario que proporcione los valores de calle, ciudad y código postal.

El *diálogo de componente* proporciona una estrategia para crear diálogos independientes para controlar escenarios concretos, lo que divide un conjunto de diálogos grande en elementos más manejables. Cada una de estas piezas tiene su propio conjunto de diálogos y evita los conflictos de nombres con el conjunto de diálogos que lo contiene. Para más información al respecto, consulte [Reutilización de diálogos](bot-builder-compositcontrol.md).

## <a name="using-dialogs"></a>Uso de diálogos

El contexto del diálogo se puede usar para empezar un diálogo, continuarlo, reemplazarlo o finalizarlo. También puede cancelar todos los diálogos de la pila de diálogos.

Los diálogo pueden considerarse una pila mediante programación, lo que llamamos la *pila de diálogos*, con el controlador de turnos como lo que le ordena y que actúa como la reserva si la pila está vacía. El elemento superior de dicha pila se considera el *diálogo activo* y el contexto del diálogo dirige todas las entradas al cuadro de diálogo activo.

Cuando se inicia un diálogo, se inserta en la pila y pasa a ser el diálogo activo. Y no deja de serlo hasta que termina, se elimina mediante el método de [sustitución de diálogo](#repeating-a-dialog) u otro diálogo se inserta en la pila (mediante el controlador de turnos o el propio diálogo activo) y se convierte en el diálogo activo. Cuando termina el diálogo nuevo, desaparece de la pila y el siguiente se convierte en el diálogo activo. Esto permite la creación de ramas y bucles tal y como se describe a continuación.

### <a name="create-the-dialog-context"></a>Creación del contexto del diálogo

Para crear el contexto de un diálogo, es preciso llamar al método de *creación de contexto* de un conjunto de diálogos. Dicho método obtiene la propiedad del *estado del diálogo* del conjunto de diálogos y la usa para crear el contexto del diálogo. Luego, se usa el contexto del cuadro de diálogo para iniciar, continuar o controlar de cualquier otra forma los diálogos del conjunto.

El conjunto de diálogos requiere el uso de un *descriptor de acceso de la propiedad de estado* para acceder al estado del diálogo. Dicho descriptor se crea y se usa de la misma forma que los restantes descriptores de acceso del estado, ya que basa en su propia propiedad del estado de la conversación. Los detalles relativos a la administración del estado se pueden encontrar en el [tema relativo a la administración del estado](bot-builder-concept-state.md), mientras que el uso del estado del diálogo se muestra en el procedimiento referente al [flujo de conversaciones secuenciales](bot-builder-dialog-manage-conversation-flow.md).

### <a name="to-start-a-dialog"></a>Para iniciar un diálogo

Para iniciar un diálogo, use el *identificador del diálogo* que desea iniciar en los métodos de *comienzo de diálogo*, *aviso* o *sustitución de diálogo* del contexto del diálogo.

* El método begin dialog insertará el diálogo en la parte superior de la pila.
* El método replace dialog retira el diálogo actual de la pila e inserta el diálogo de reemplazo en la pila. Se cancela el cuadro de diálogo reemplazado y se elimina toda la información que esa instancia contenga.

Use el parámetro _options_ para pasar información a la nueva instancia del diálogo.
Para acceder a las opciones pasadas al nuevo diálogo se puede usar la propiedad *options* del contexto en cualquier paso del diálogo.
Para ver un ejemplo de código, consulte [Creación de un flujo de conversación avanzado con ramas y bucles](bot-builder-dialog-manage-complex-conversation-flow.md).

### <a name="to-continue-a-dialog"></a>Para continuar un diálogo

Para continuar un diálogo, llame al método para *continuar diálogo*. Dicho método continuará siempre el primer diálogo de la pila (el cuadro de diálogo activo), en caso de que haya alguno. Si el diálogo que se ha continuado finaliza, el control se pasa al contexto del primario que continúa en el mismo turno.

Use la propiedad *values* del contexto del paso para conservar el estado de un turno a otro.
Todos los valores agregados a esta colección en un turno anterior estará disponible en los turnos posteriores.
Para ver un ejemplo de código, consulte [Creación de un flujo de conversación avanzado con ramas y bucles](bot-builder-dialog-manage-complex-conversation-flow.md).

### <a name="to-end-a-dialog"></a>Para finalizar un diálogo

El método para *finalizar diálogo* finaliza un diálogo retirándolo de la pila y devuelve un resultado opcional al contexto primario (como por ejemplo el diálogo que lo llamó o el controlador de turnos del bot). Muy a menudo desde el propio diálogo se realiza la llamada para finalizar la instancia actual de sí mismo.

Este método se puede llamar desde cualquier lugar en que tenga un contexto de diálogo, pero aparecerá en el bot al que se llamó desde el diálogo activo actual.

> [!TIP]
> Un procedimiento recomendado es llamar al método de *finalización de diálogo* al final del diálogo.

### <a name="to-clear-all-dialogs"></a>Para borrar todos los diálogos

Si desea quitar todos los diálogos de la pila, puede borrar la pila de diálogos, para lo que debe llamar al método *cancel all dialogs* del contexto del diálogo.

### <a name="repeating-a-dialog"></a>Repetición de un diálogo

Puede reemplazar un diálogo por sí mismo y crear un bucle.
Esta es una excelente manera de tratar las [iteraciones complejas](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md) y una buena técnica para administrar menús.

> [!NOTE]
> Si necesita conservar el estado interno del diálogo actual, deberá pasar información a la nueva instancia del mismo en la llamada al método de *sustitución de diálogo* y, luego, inicializar el diálogo en consecuencia.

### <a name="branch-a-conversation"></a>Bifurcación de una conversación

El contexto del diálogo mantiene la pila del mismo y, en todos los diálogo de la pila, realiza un seguimiento de cuál es el paso siguiente. Su método de *comienzo de diálogo* crea un diálogo secundario y lo inserta al principio de la pila, mientras que su método de *finalización de diálogo* retira el diálogo de la pila. A este último método normalmente se le llama desde dentro del *diálogo que finaliza*.

Para iniciar un nuevo diálogo dentro del mismo conjunto de diálogos, un diálogo puede llamar al método *begin dialog* del contexto del diálogo o proporcionar el identificador del nuevo diálogo, lo que convierte a este en el diálogo actualmente activo. El diálogo original está todavía en la pila, pero las llamadas al método *continue* del contexto del diálogo solo se envían al diálogo que está arriba de la pila, el *diálogo activo*. Cuando un diálogo se retira de la pila, el contexto del diálogo se reanuda con el siguiente paso de la cascada en la pila donde se dejó el diálogo original.

Por lo tanto, puede crear una bifurcación dentro del flujo de conversación mediante la inclusión de un paso en un diálogo que puede elegir condicionalmente un diálogo para iniciar de un conjunto de diálogos disponibles.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Uso de la biblioteca de diálogos para recopilar datos de entrada del usuario](bot-builder-prompts.md)

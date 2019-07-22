---
title: Diferencias entre las versiones v3 y v4 del SDK para NodeJS | Microsoft Docs
description: Describe las diferencias entre las versiones v3 y v4 del SDK para NodeJS.
keywords: bot migration, dialogs, state
author: mmiele
ms.author: v-mimiel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/08/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 24725df2f109e73df142194c05ee27c2d8ba5da4
ms.sourcegitcommit: 4f78e68507fa3594971bfcbb13231c5bfd2ba555
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/17/2019
ms.locfileid: "68292215"
---
# <a name="differences-between-the-v3-and-v4-javascript-sdk"></a>Diferencias entre las versiones v3 y v4 del SDK para JavaScript

La versión v4 de Framework Bot SDK admite el mismo servicio Framework Bot subyacente que la versión v3. Sin embargo, la versión v4 es una refactorización de la versión anterior que ofrece a los desarrolladores más flexibilidad y control sobre sus bots. Algunos cambios importantes en el SDK son:

- Incorporación de un adaptador de bot.
  - Esto forma parte de la pila de procesamiento de la actividad.
  - Controla la autenticación de Bot Framework.
  - Administra el tráfico entrante y saliente entre un canal y el controlador de turnos del bot, que encapsula las llamadas a Bot Framework Connector.
  - Inicializa el contexto para cada turno.
  - Para más información, consulte cómo [funcionan los bots](../bot-builder-basics.md).
- Administración de estados refactorizada.
  - Los datos de estado ya no están disponibles automáticamente dentro de un bot.
  - Ahora, se administra mediante objetos de administración de estado y descriptores de acceso de propiedad.
  - Consulte [Administración del estado](../bot-builder-concept-state.md) para más información.
- Nueva biblioteca de diálogos
  - Los diálogos de la versión v3 tienen que volver a escribirse para la nueva biblioteca de diálogos.
  - Ya no existen componentes puntuables. Puede buscar comandos "globales", antes de pasar el control a los diálogos. En función de cómo haya diseñado su bot v4, podría estar en el controlador de mensajes o en un diálogo primario. Para obtener un ejemplo, consulte cómo [controlar las interrupciones de usuario](../bot-builder-howto-handle-user-interrupt.md).
  - Para más información, consulte la [biblioteca de diálogos](../bot-builder-concept-dialog.md).

## <a name="activity-processing"></a>Procesamiento de actividades

Al crear el adaptador para el bot, también proporciona un delegado de controlador de mensajes que recibirá las actividades entrantes de los canales y los usuarios. El adaptador crea un objeto de contexto de turno para cada actividad recibida. Pasa el objeto de contexto de turno al controlador de turnos del bot y luego elimina el objeto cuando el turno se completa.

El controlador de turnos puede recibir muchos tipos de actividades. En general, querrá reenviar solo las actividades de _mensaje_ a todos los diálogos que su bot contiene. Si deriva el bot desde `ActivityHandler`, el controlador de turnos del bot reenviará todas las actividades de mensajes a `OnMessage`. Reemplace este método para agregar la lógica de control de mensajes. Para más información acerca de los tipos de actividad, consulte el [esquema de actividades](https://github.com/Microsoft/botframework-sdk/blob/master/specs/botframework-activity/botframework-activity.md).

### <a name="handling-turns"></a>Control de turnos

En el control de un mensaje, use el contexto de turno para obtener información acerca de la actividad entrante y para enviar actividades al usuario:

|Actividad |DESCRIPCIÓN |
|:---|:---|
| Para obtener la actividad entrante | Obtenga la propiedad `Activity` del contexto de turno. |
| Para crear y enviar una actividad al usuario | Llame al método `SendActivity` del contexto de turno.<br/> Para más información, consulte cómo [enviar y recibir un mensaje de texto](../../rest-api/bot-framework-rest-direct-line-1-1-receive-messages.md) y cómo [agregar elementos multimedia a los mensajes](../bot-builder-howto-add-media-attachments.md). |

La clase `MessageFactory` proporciona algunos métodos auxiliares para las actividades de creación y formato.

### <a name="interruptions"></a>Interrupciones

Se administran en el bucle de mensajes del bot. Para obtener una descripción de cómo hacer esto con los diálogos de la versión v4, consulte cómo [controlar las interrupciones de usuario](../bot-builder-howto-handle-user-interrupt.md).

## <a name="state-management"></a>Administración de estados

En la versión v3, podría almacenar los datos de la conversación en el servicio Bot State, parte de la serie de servicios proporcionados por Bot Framework. Sin embargo, el servicio se retiró el 31 de marzo de 2018. A partir de la versión v4, las consideraciones de diseño sobre la administración de estado es igual que en cualquier aplicación web y hay una serie de opciones disponibles. El almacenamiento en caché en memoria y en el mismo proceso normalmente es la más sencilla; sin embargo, para aplicaciones de producción debería almacenar el estado de manera más permanente, como en una base de datos SQL o no SQL o como blobs.

La versión v4 no utiliza las propiedades `UserData`, `ConversationData` y `PrivateConversationData` ni contenedores de datos para administrar el estado.
Ahora, el estado se administra mediante objetos de administración de estado y descriptores de acceso de propiedad, tal y como se describe en [Administración del estado](../bot-builder-concept-state.md).

La versión v4 define las clases `UserState`, `ConversationState` y `PrivateConversationState` que administran el estado del bot. Deberá crear un descriptor de acceso de propiedad de estado para cada propiedad que desee conservar, en lugar de solo leer y escribir en un contenedor de datos predefinido.

### <a name="setting-up-state"></a>Configuración del estado

El estado debe configurarse en el archivo de punto de entrada de la aplicación, normalmente "index.js" o "app.js" en las aplicaciones de NodeJS. 

1. Inicialice uno o más objetos que implementan la interfaz `Storage` proporcionada por botbuilder-core. Representa la memoria auxiliar para los datos del bot.
    El SDK de la versión v4 proporciona algunas [capas de almacenamiento](../bot-builder-concept-state.md#storage-layer).
    También puede implementar las suyas propias para conectarse a otro tipo de almacén.
1. Después, cree y registre los objetos de [administración de estado](../bot-builder-concept-state.md#state-management) según sea necesario.
    Tiene los mismos ámbitos disponibles que en la versión v3 y puede crear otros si los necesita.
1. Por último, cree y registre los [descriptores de acceso de propiedad de estado](../bot-builder-concept-state.md#state-property-accessors) para las propiedades que su bot necesita.
    Dentro de un objeto de administración de estado, cada descriptor de acceso de propiedad requiere un nombre único.

### <a name="using-state"></a>Uso de estados

Utilice los descriptores de acceso de propiedad de estado para obtener y actualizar las propiedades, y use los objetos de administración de estado para escribir los cambios en el almacenamiento. Sabiendo que hay que tener en cuenta los problemas de simultaneidad, aquí le mostramos cómo realizar algunas tareas comunes.

| Tarea|DESCRIPCIÓN |
|:---|:---|
| Para crear un descriptor de acceso de propiedad de estado | Llame al método `createProperty` del objeto `BotState`. <br/>`BotState` es la clase base abstracta para la conversación, la conversación privada y el estado del usuario. |
| Para obtener el valor actual de una propiedad | Llame a `StatePropertyAccessor.get(TurnContext)`.<br/>Si no se ha establecido previamente ningún valor, usará el parámetro predeterminado para generar un valor. |
| Para conservar los cambios de estado en el almacenamiento | Llame a `BotState.saveChanges(TurnContext, boolean)` para cualquiera de los objetos de administración de estado en el que el estado haya cambiado antes de salir del controlador de turnos. |

### <a name="managing-concurrency"></a>Administrar la simultaneidad

Puede que el bot tenga que administrar la simultaneidad de estados. Para más información, consulte la sección [Guardar el estado](../bot-builder-concept-state.md#saving-state) de **Administración del estado** y la sección [Administración de la simultaneidad mediante eTags](../bot-builder-howto-v4-storage.md#manage-concurrency-using-etags) de **Escritura directa en el almacenamiento**.

## <a name="dialogs-library"></a>Biblioteca de diálogos

Estos son algunos de los cambios principales en los diálogos:

- La biblioteca de diálogos ahora es un paquete npm independiente: **botbuilder-dialogs**.
- El estado de los diálogos se administra mediante descriptores de acceso de propiedad de estado y la clase de estado `DialogState`.
  - Ahora, se conserva la propiedad de estado de diálogo de un turno a otro, en lugar del propio objeto de diálogo.
- Dentro de un turno, se crea un contexto de diálogo para un _conjunto de diálogos_.
  - Este contexto de diálogo encapsula la pila de diálogos. Esta información se conserva dentro de la propiedad de estado de diálogo.
- Ambas versiones contienen la clase abstracta `Dialog`; sin embargo, en v3 extiende la clase "ActionSet" mientras que en v4 extiende "Object".

### <a name="defining-dialogs"></a>Definición de diálogos

Aunque la versión v3 proporcionaba una manera flexible para implementar diálogos mediante la clase `Dialog`, esto significaba que debía implementar su propio código para características como la validación. En la versión v4, ahora hay clases de avisos que validan la entrada del usuario de modo automático, la restringen a un tipo específico (por ejemplo, un número entero) y preguntan al usuario de nuevo automáticamente hasta que proporcione una entrada válida. En general, esto significa menos código a escribir como desarrollador.

Ahora hay varias opciones para definir los diálogos:

|Tipo de diálogo| DESCRIPCIÓN |
|:---|:---|
| Un diálogo de componente, derivado de la clase `ComponentDialog`. | Permite encapsular el código del diálogo sin conflictos de nombres con los contextos externos. Consulte cómo [reutilizar los diálogos](../bot-builder-concept-dialog.md).|
| Un diálogo en cascada, una instancia de la clase `WaterfallDialog`. | Diseñado para trabajar con los diálogos de aviso, que solicitarán y validarán diversos tipos de datos de entrada del usuario. Una cascada automatiza la mayor parte del proceso, pero impone una cierta forma en el código del diálogo; consulte el [flujo de conversación secuencial](../bot-builder-dialog-manage-conversation-flow.md). |
| Un diálogo personalizado, derivado de la clase abstracta `Dialog`. | Esto ofrece la máxima flexibilidad en cuanto a cómo se comportan los diálogos, pero también necesita conocer mejor cómo se implementa la pila de diálogos. |

Cuando se crea un diálogo en cascada, se definen los pasos del diálogo en el constructor. El orden de los pasos ejecutados se sigue exactamente cómo se ha declarado y se mueve hacia delante automáticamente uno tras otro.

También puede crear flujos de control complejos mediante el uso de varios diálogos; consulte el [flujo de conversación avanzada](../bot-builder-dialog-manage-complex-conversation-flow.md).

Para acceder a un diálogo, deberá poner una instancia de él en un _conjunto de diálogos_ y, después, generar un _contexto de diálogo_ para ese conjunto. Debe proporcionar un descriptor de acceso de propiedad de estado de diálogo al crear el conjunto de diálogos. Esto permite al marco conservar el estado del diálogo de un turno al siguiente. En [Administración del estado](../bot-builder-concept-state.md) se describe cómo administrar el estado en la versióon v4.

### <a name="using-dialogs"></a>Uso de diálogos

Presentamos una lista de operaciones comunes en la versión v3 y cómo realizarlas en un diálogo en cascada. Tenga en cuenta que cada paso de un diálogo en cascada debe devolver un valor `DialogTurnResult`. Si no es así, la cascada podría terminar antes de tiempo.

| Operación | v3 | v4 |
|:---|:---|:---|
| Controlar el inicio del diálogo | Llame a `session.beginDialog` y pase el identificador del diálogo. | call `DialogContext.beginDialog` (llamar al 555-555-5555) |
| Envío de una actividad | Llame a `session.send`. | Llame a `TurnContext.sendActivity`.<br/>Use la propiedad `Context` del contexto del paso para obtener el contexto del turno (`step.context.sendActivity`).  |
| Esperar la respuesta de un usuario | Llame a un aviso desde dentro del paso de la cascada, por ejemplo, `builder.Prompts.text(session, 'Please enter your destination')`. Recupere la respuesta en el paso siguiente. | Espere a la devolución de `TurnContext.prompt` para iniciar un diálogo. Después, recupere el resultado en el paso siguiente de la cascada. |
| Controlar la continuación del diálogo | Automático | Agregue pasos adicionales a un diálogo en cascada o implemente `Dialog.continueDialog` |
| Indicar el final del procesamiento hasta el siguiente mensaje del usuario | Llame a `session.endDialog`. | Devuelva `Dialog.EndOfTurn`. |
| Iniciar un diálogo secundario | Llame a `session.beginDialog`. | Espere a la devolución del método `beginDialog` del contexto del paso.<br/>Si el diálogo secundario devuelve un valor, el valor está disponible en el paso siguiente de la cascada mediante la propiedad `Result` del contexto del paso. |
| Reemplazar el diálogo actual por un nuevo diálogo | Llame a `session.replaceDialog`. | Espere a la devolución de `ITurnContext.replaceDialog`. |
| Señalizar que el diálogo actual se ha completado | Llame a `session.endDialog`. | Espere a la devolución del método `endDialog` del contexto del paso. |
| Terminar un diálogo debido a un error | Llame a `session.pruneDialogStack`. | Inicie una excepción que se detecte en otro nivel del bot, finalice el paso con el estado `Cancelled` o llame a `cancelAllDialogs` del contexto del paso o diálogo. |

Otras notas sobre el código de la versión v4:

- En la versión v4, las distintas clases derivadas `Prompt` implementan los avisos al usuario como diálogos independientes en dos pasos. Consulte el procedimiento para [implementar un flujo de conversación secuencial][sequential-flow]..
- Use `DialogSet.createContext` para crear un contexto de diálogo para el turno actual.
- Dentro de un diálogo, use la propiedad `DialogContext.context` para obtener el contexto del turno actual.
- Los pasos de la cascada tienen un parámetro `WaterfallStepContext`, que deriva de `DialogContext`.
- Todas las clases específicas de los diálogos y los avisos derivan de la clase abstracta `Dialog`.
- Se asigna un identificador cuando se crea un diálogo de componente. En un conjunto de diálogos, se debe establecer un identificador único para cada diálogo del conjunto.

### <a name="passing-state-between-and-within-dialogs"></a>Paso del estado entre diálogos y dentro de un diálogo

En las secciones sobre el [estado de un diálogo](../bot-builder-concept-dialog.md#dialog-state), las [propiedades de contexto del paso de la cascada](../bot-builder-concept-dialog.md#waterfall-step-context-properties) y el [uso de los diálogos](../bot-builder-concept-dialog.md#using-dialogs) del artículo sobre la **biblioteca de diálogos** se describe cómo administrar el estado de los diálogos en la versión v4.

### <a name="get-user-response"></a>Obtener la respuesta del usuario

Puede obtener la actividad del usuario en un turno en el contexto del turno.

Para preguntar al usuario y recibir el resultado de un aviso:

- Agregue una instancia de aviso adecuada al conjunto de diálogos.
- Llame al aviso desde un paso de un diálogo en cascada.
- Recupere el resultado de la propiedad `Result` del contexto del paso en el paso siguiente.

## <a name="additional-resources"></a>Recursos adicionales

Consulte los siguientes recursos para más información.

| Tema | DESCRIPCIÓN |
| :--- | :--- |
|[Migración de un bot del SDK v3 para JavaScript a v4](https://docs.microsoft.com/en-us/azure/bot-service/migration/conversion-javascript?view=azure-bot-service-4.0)| Migración de un bot del SDK v3 para JavaScript a v4.|
| [Novedades de Bot Framework](https://docs.microsoft.com/en-us/azure/bot-service/what-is-new?view=azure-bot-service-4.0) | Principales mejoras y características de Bot Framework y Azure Bot Service|
|[Funcionamiento de los bots](../bot-builder-basics.md)|El mecanismo interno de un bot|
|[Administración de estados](../bot-builder-concept-state.md)|Abstracciones para facilitar la administración de estados|
|[Biblioteca de cuadros de diálogo](../bot-builder-concept-dialog.md)| Conceptos centrales para administrar una conversación|
|[Envío y recepción de mensajes de texto](../bot-builder-howto-send-messages.md)|Principal forma de comunicación de un bot con los usuarios|
|[Enviar elemento multimedia](../bot-builder-howto-add-media-attachments.md)|Archivos multimedia adjuntos, como imágenes, vídeo, audio y archivos| 
|[Flujo de conversación secuencial](../bot-builder-dialog-manage-conversation-flow.md)| Hacer preguntas es la principal forma de interacción de un bot con los usuarios.|
|[Guardar usuario y datos de conversación](../bot-builder-howto-v4-state.md)|Seguimiento de una conversación mientras está sin estado|
|[Flujo complejo](../bot-builder-dialog-manage-complex-conversation-flow.md)|Administración de flujos de conversación complejos |
|[Reutilización de diálogos](../bot-builder-compositcontrol.md)|Crear cuadros de diálogo independientes para controlar escenarios específicos|
|[Interrupciones](../bot-builder-howto-handle-user-interrupt.md)| Controlar las interrupciones para crear un bot sólido|
|[Esquema de la actividad](https://aka.ms/botSpecs-activitySchema)|Esquema para seres humanos y software automatizados|
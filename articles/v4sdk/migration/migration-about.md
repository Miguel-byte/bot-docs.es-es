---
title: Diferencias entre las versiones v3 y v4 del SDK | Microsoft Docs
description: Describe las diferencias entre las versiones v3 y v4 del SDK.
keywords: bot migration, formflow, dialogs, state
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6444c1e3ef1948b3e407df50255aaa7c5350bf34
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298993"
---
# <a name="differences-between-the-v3-and-v4-net-sdk"></a>Diferencias entre las versiones v3 y v4 del SDK para .NET

La versión v4 de Framework Bot SDK admite el mismo servicio Framework Bot subyacente que la versión v3. Sin embargo, la versión v4 es una refactorización de la versión anterior que ofrece a los desarrolladores más flexibilidad y control sobre sus bots. Algunos cambios importantes en el SDK son:

- Incorporación de un adaptador de bot. El adaptador es parte de la pila de procesamiento de la actividad.
  - El adaptador controla la autenticación de Bot Framework.
  - El adaptador administra el tráfico entrante y saliente entre un canal y el controlador de turnos del bot, que encapsula las llamadas a Bot Framework Connector.
  - El adaptador inicializa el contexto para cada turno.
  - Para más información, consulte [cómo funcionan los bots][about-bots].
- Administración de estados refactorizada.
  - Los datos de estado ya no están disponibles automáticamente dentro de un bot.
  - Ahora, el estado se administra mediante objetos de administración de estado y descriptores de acceso de propiedad.
  - Para más información, consulte la [administración del estado][about-state].
- Nueva biblioteca de diálogos.
  - Los diálogos de la versión v3 tendrá que volver a escribirse para la nueva biblioteca de diálogos.
  - Ya no existen componentes puntuables. Puede buscar comandos "globales", antes de pasar el control a los diálogos. En función de cómo haya diseñado su bot v4, podría estar en el controlador de mensajes o en un diálogo primario. Para obtener un ejemplo, consulte cómo [controlar las interrupciones de usuario][interruptions].
  - Para más información, consulte la [biblioteca de diálogos][about-dialogs].
- Compatibilidad con ASP.NET Core
  - Las plantillas para crear nuevos bots de C# tienen como destino la plataforma ASP.NET Core.
  - Puede seguir usando ASP.NET para sus bots, pero nuestro objetivo en la versión v4 es dar soporte a la plataforma ASP.NET Core.
  - Consulte la [introducción a ASP.NET Core](https://docs.microsoft.com/aspnet/core/) para más información acerca de esta plataforma.

## <a name="activity-processing"></a>Procesamiento de actividades

Al crear el adaptador para el bot, también proporciona un delegado de controlador de mensajes que recibirá las actividades entrantes de los canales y los usuarios. El adaptador crea un objeto de contexto de turno para cada actividad recibida. Pasa el objeto de contexto de turno al controlador de turnos del bot y luego elimina el objeto cuando el turno se completa.

El controlador de turnos puede recibir muchos tipos de actividades. En general, querrá reenviar solo las actividades de _mensaje_ a todos los diálogos que su bot contiene. Si deriva el bot desde `ActivityHandler`, el controlador de turnos del bot reenviará todas las actividades de mensajes a `OnMessageActivityAsync`. Reemplace este método para agregar la lógica de control de mensajes. Para más información acerca de los tipos de actividad, consulte el [esquema de actividades][].

### <a name="handling-turns"></a>Control de turnos

En el control de un mensaje, use el contexto de turno para obtener información acerca de la actividad entrante y para enviar actividades al usuario:

| | |
|-|-|
| Para obtener la actividad entrante | Obtenga la propiedad `Activity` del contexto de turno. |
| Para crear y enviar una actividad al usuario | Llame al método `SendActivityAsync` del contexto de turno.<br/>Para más información, consulte cómo [enviar y recibir un mensaje de texto][send-messages] y cómo [agregar elementos multimedia a los mensajes][send-media]. |

La clase `MessageFactory` proporciona algunos métodos auxiliares para las actividades de creación y formato.

### <a name="scorables-is-gone"></a>Los componentes puntuables han desaparecido

Se administran en el bucle de mensajes del bot. Para obtener una descripción de cómo hacer esto con los diálogos de la versión v4, consulte cómo [controlar las interrupciones de usuario][interruptions].

Los árboles de envío de componentes puntuables que admiten composición y los diálogos en cadena que admiten composición, como la _excepción predeterminada_, también han desaparecido. Una manera de reproducir esta funcionalidad es implementarla dentro de controlador de turnos del bot.

## <a name="state-management"></a>Administración de estados

En la versión v3, podría almacenar los datos de la conversación en el servicio Bot State, parte de la serie de servicios proporcionados por Bot Framework. Sin embargo, el servicio se ha retirado desde el 31 de marzo de 2018. A partir de la versión v4, las consideraciones de diseño sobre la administración de estado es igual que en cualquier aplicación web y hay una serie de opciones disponibles. El almacenamiento en caché en memoria y en el mismo proceso normalmente es la más sencilla; sin embargo, para aplicaciones de producción debería almacenar el estado de manera más permanente, como en una base de datos SQL o no SQL o como blobs.

La versión v4 no utiliza las propiedades `UserData`, `ConversationData` y `PrivateConversationData` ni contenedores de datos para administrar el estado.
Ahora, el estado se administra mediante objetos de administración de estado y descriptores de acceso de propiedad, tal y como se describe en [Administración del estado][about-state].

La versión v4 define las clases `UserState`, `ConversationState` y `PrivateConversationState` que administran el estado del bot. Deberá crear un descriptor de acceso de propiedad de estado para cada propiedad que desee conservar, en lugar de solo leer y escribir en un contenedor de datos predefinido.

### <a name="setting-up-state"></a>Configuración del estado

El estado se debe configurar como singletons siempre que sea posible, en **Startup.cs** para .NET Core o en **Global.asax.cs** para .NET Framework.

1. Inicialice uno o varios de los objetos `IStorage`. Representa la memoria auxiliar para los datos del bot.
    El SDK de la versión v4 proporciona algunas [capas de almacenamiento](../bot-builder-concept-state.md#storage-layer).
    También puede implementar las suyas propias para conectarse a otro tipo de almacén.
1. Después, cree y registre los objetos de [administración de estado](../bot-builder-concept-state.md#state-management) según sea necesario.
    Tiene los mismos ámbitos disponibles que en la versión v3 y puede crear otros si los necesita.
1. Después, cree y registre [descriptores de acceso de propiedad de estado](../bot-builder-concept-state.md#state-property-accessors) para las propiedades que su bot necesita.
    Dentro de un objeto de administración de estado, cada descriptor de acceso de propiedad requiere un nombre único.

### <a name="using-state"></a>Uso de estados

Puede usar la inserción de dependencias para acceder a ellas cuando el bot se crea.
(En ASP.NET, se crea una nueva instancia del bot o controlador de mensajes para cada turno). Utilice los descriptores de acceso de propiedad de estado para obtener y actualizar las propiedades, y use los objetos de administración de estado para escribir los cambios en el almacenamiento. Sabiendo que hay que tener en cuenta los problemas de simultaneidad, aquí le mostramos cómo realizar algunas tareas comunes.

| | |
|-|-|
| Para crear un descriptor de acceso de propiedad de estado | Llame a `BotState.CreateProperty<T>`.<br/>`BotState` es la clase base abstracta para la conversación, la conversación privada y el estado del usuario. |
| Para obtener el valor actual de una propiedad | Llame a `IStatePropertyAccessor<T>.GetAsync`.<br/>Si no se ha establecido previamente ningún valor, usará el parámetro predeterminado para generar un valor. |
| Para actualizar el valor actual de una propiedad almacenado en la memoria caché | Llame a `IStatePropertyAccessor<T>.SetAsync`.<br/>Esto solo actualiza la memoria caché y no la capa de almacenamiento de respaldo. |
| Para conservar los cambios de estado en el almacenamiento | Llame a `BotState.SaveChangesAsync` para cualquiera de los objetos de administración de estado en el que el estado haya cambiado antes de salir del controlador de turnos. |

### <a name="managing-concurrency"></a>Administrar la simultaneidad

Puede que el bot tenga que administrar la simultaneidad de estados. Para más información, consulte la sección [Guardar el estado](../bot-builder-concept-state.md#saving-state) de **Administración del estado** y la sección [Administración de la simultaneidad mediante eTags](../bot-builder-howto-v4-storage.md#manage-concurrency-using-etags) de **Escritura directa en el almacenamiento**.

## <a name="dialogs-library"></a>Biblioteca de diálogos

Estos son algunos de los cambios principales en los diálogos:

- La biblioteca de diálogos ahora es un paquete NuGet independiente: **Microsoft.Bot.Builder.Dialogs**.
- Ya no es necesario que las clases de diálogo sean serializables. El estado de los diálogos se administra mediante un descriptor de acceso de propiedad de estado `DialogState`.
  - Ahora, se conserva la propiedad de estado de diálogo de un turno a otro, en lugar del propio objeto de diálogo.
- La interfaz `IDialogContext` se sustituye por la clase `DialogContext`. Dentro de un turno, se crea un contexto de diálogo para un _conjunto de diálogos_.
  - Este contexto de diálogo encapsula la pila de diálogos (el antiguo marco de pila). Esta información se conserva dentro de la propiedad de estado de diálogo.
- La interfaz `IDialog` se sustituye por la clase abstracta `Dialog`.

### <a name="defining-dialogs"></a>Definición de diálogos

Aunque la versión v3 proporcionaba una manera flexible para implementar diálogos mediante la interfaz `IDialog`, esto significaba que debía implementar su propio código para características como la validación. En la versión v4, ahora hay clases de avisos que validan la entrada del usuario de modo automático, la restringen a un tipo específico (por ejemplo, un número entero) y preguntan al usuario de nuevo automáticamente hasta que proporcione una entrada válida. En general, esto significa menos código a escribir como desarrollador.

Ahora hay varias opciones para definir los diálogos:

| | |
|:--|:--|
| Un diálogo de componente, derivado de la clase `ComponentDialog`. | Permite encapsular el código del diálogo sin conflictos de nombres con los contextos externos. Consulte cómo [reutilizar los diálogos][reuse-dialogs]. |
| Un diálogo en cascada, una instancia de la clase `WaterfallDialog`. | Diseñado para trabajar con los diálogos de aviso, que solicitarán y validarán diversos tipos de datos de entrada del usuario. Una cascada automatiza la mayor parte del proceso, pero impone una cierta forma en el código del diálogo; consulte el [flujo de conversación secuencial][sequential-flow]. |
| Un diálogo personalizado, derivado de la clase abstracta `Dialog`. | Esto ofrece la máxima flexibilidad en cuanto a cómo se comportan los diálogos, pero también necesita conocer mejor cómo se implementa la pila de diálogos. |

En la versión v3, se usaba `FormFlow` para realizar un número determinado de pasos para una tarea. En la versión v4, el diálogo en cascada reemplaza a FormFlow. Cuando se crea un diálogo en cascada, se definen los pasos del diálogo en el constructor. El orden de los pasos ejecutados se sigue exactamente cómo se ha declarado y se mueve hacia delante automáticamente uno tras otro.

También puede crear flujos de control complejos mediante el uso de varios diálogos; consulte el [flujo de conversación avanzada][complex-flow].

Para acceder a un diálogo, deberá poner una instancia de él en un _conjunto de diálogos_ y, después, generar un _contexto de diálogo_ para ese conjunto. Debe proporcionar un descriptor de acceso de propiedad de estado de diálogo al crear el conjunto de diálogos. Esto permite al marco conservar el estado del diálogo de un turno al siguiente. En [Administración del estado][about-state] se describe cómo administrar el estado en la versióon v4.

### <a name="using-dialogs"></a>Uso de diálogos

Presentamos una lista de operaciones comunes en la versión v3 y cómo realizarlas en un diálogo en cascada. Tenga en cuenta que cada paso de un diálogo en cascada debe devolver un valor `DialogTurnResult`. Si no es así, la cascada podría terminar antes de tiempo.

| Operación | v3 | v4 |
|:---|:---|:---|
| Controlar el inicio del diálogo | Implemente `IDialog.StartAsync` | Haga que este sea el primer paso de un diálogo en cascada. |
| Envío de una actividad | Llame a `IDialogContext.PostAsync`. | Llame a `ITurnContext.SendActivityAsync`.<br/>Use la propiedad `Context` del contexto del paso para obtener el contexto de turno.  |
| Esperar la respuesta de un usuario | Use un parámetro `IAwaitable<IMessageActivity>` y llame a `IDialogContext.Wait`. | Espere a la devolución de `ITurnContext.PromptAsync` para iniciar un diálogo. Después, recupere el resultado en el paso siguiente de la cascada. |
| Controlar la continuación del diálogo | Llame a `IDialogContext.Wait`. | Agregue pasos adicionales a un diálogo en cascada o implemente `Dialog.ContinueDialogAsync` |
| Indicar el final del procesamiento hasta el siguiente mensaje del usuario | Llame a `IDialogContext.Wait`. | Devuelva `Dialog.EndOfTurn`. |
| Iniciar un diálogo secundario | Llame a `IDialogContext.Call`. | Espere a la devolución del método `BeginDialogAsync` del contexto del paso.<br/>Si el diálogo secundario devuelve un valor, el valor está disponible en el paso siguiente de la cascada mediante la propiedad `Result` del contexto del paso. |
| Reemplazar el diálogo actual por un nuevo diálogo | Llame a `IDialogContext.Forward`. | Espere a la devolución de `ITurnContext.ReplaceDialogAsync`. |
| Señalizar que el diálogo actual se ha completado | Llame a `IDialogContext.Done`. | Espere a la devolución del método `EndDialogAsync` del contexto del paso. |
| Terminar un diálogo debido a un error | Llame a `IDialogContext.Fail`. | Inicie una excepción que se detecte en otro nivel del bot, finalice el paso con el estado `Cancelled` o llame a `CancelAllDialogsAsync` del contexto del paso o diálogo.<br/>Tenga en cuenta que, en la versión v4, las excepciones dentro de un diálogo se propagan por la pila de C# en lugar de por la pila de diálogos. |

Otras notas sobre el código de la versión v4:

- En la versión v4, las distintas clases derivadas `Prompt` implementan los avisos al usuario como diálogos independientes en dos pasos. Consulte el procedimiento de [Implementación de flujo de conversación secuencial][sequential-flow].
- Use `DialogSet.CreateContextAsync` para crear un contexto de diálogo para el turno actual.
- Dentro de un diálogo, use la propiedad `DialogContext.Context` para obtener el contexto del turno actual.
- Los pasos de la cascada tienen un parámetro `WaterfallStepContext`, que deriva de `DialogContext`.
- Todas las clases específicas de los diálogos y los avisos derivan de la clase abstracta `Dialog`.
- Se asigna un identificador cuando se crea un diálogo de componente. En un conjunto de diálogos, se debe establecer un identificador único para cada diálogo del conjunto.

### <a name="passing-state-between-and-within-dialogs"></a>Paso del estado entre diálogos y dentro de un diálogo

En las secciones sobre el [estado de un diálogo](../bot-builder-concept-dialog.md#dialog-state), las [propiedades de contexto del paso de la cascada](../bot-builder-concept-dialog.md#waterfall-step-context-properties) y el [uso de los diálogos](../bot-builder-concept-dialog.md#using-dialogs) del artículo sobre la **biblioteca de diálogos** se describe cómo administrar el estado de los diálogos en la versión v4.

### <a name="iawaitable-is-gone"></a>IAwaitable ya no existe

Puede obtener la actividad del usuario en un turno en el contexto del turno.

Para preguntar al usuario y recibir el resultado de un aviso:

- Agregue una instancia de aviso adecuada al conjunto de diálogos.
- Llame al aviso desde un paso de un diálogo en cascada.
- Recupere el resultado de la propiedad `Result` del contexto del paso en el paso siguiente.

### <a name="formflow"></a>Formflow

En la versión v3, Formflow formaba parte del SDK para C#, pero no del SDK para JavaScript. No forma parte de la versión v4 del SDK, pero existe una versión de la comunidad para C#.

| Nombre del paquete NuGet | Repositorio de GitHub de la comunidad |
|-|-|
| Bot.Builder.Community.Dialogs.Formflow | [BotBuilderCommunity/botbuilder-community-dotnet/libraries/Bot.Builder.Community.Dialogs.FormFlow](https://github.com/BotBuilderCommunity/botbuilder-community-dotnet/tree/master/libraries/Bot.Builder.Community.Dialogs.FormFlow) |

## <a name="additional-resources"></a>Recursos adicionales

- [Migración de un bot de la versión v3 a la versión v4 del SDK para .NET](conversion-framework.md)

<!-- -->

[about-bots]: ../bot-builder-basics.md
[about-state]: ../bot-builder-concept-state.md
[about-dialogs]: ../bot-builder-concept-dialog.md

[send-messages]: ../bot-builder-howto-send-messages.md
[send-media]: ../bot-builder-howto-add-media-attachments.md

[sequential-flow]: ../bot-builder-dialog-manage-conversation-flow.md
[complex-flow]: ../bot-builder-dialog-manage-complex-conversation-flow.md
[reuse-dialogs]: ../bot-builder-compositcontrol.md
[interruptions]: ../bot-builder-howto-handle-user-interrupt.md

[esquema de actividades]: https://aka.ms/botSpecs-activitySchema
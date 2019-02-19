---
title: Diferencias entre las versiones v3 y v4 del SDK | Microsoft Docs
description: Describe las diferencias entre las versiones v3 y v4 del SDK.
keywords: bot migration, formflow, dialogs, state
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/11/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7e5440e7d47d88b7ff6827359e7eb621bce53e3c
ms.sourcegitcommit: 7f418bed4d0d8d398f824e951ac464c7c82b8c3e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2019
ms.locfileid: "56240497"
---
# <a name="differences-between-the-v3-and-v4-net-sdk"></a>Diferencias entre las versiones v3 y v4 del SDK para .NET

La versión v4 de Framework Bot SDK admite el mismo servicio Framework Bot subyacente que la versión v3. Sin embargo, la versión v4 es una refactorización de la versión anterior que ofrece a los desarrolladores más flexibilidad y control sobre sus bots. Algunos cambios importantes en el SDK son:

- Incorporación de un adaptador de bot. El adaptador es parte de la pila de procesamiento de la actividad.
  - El adaptador controla la autenticación de Bot Framework.
  - El adaptador administra el tráfico entrante y saliente entre un canal y el controlador de turnos del bot, que encapsula las llamadas a Bot Framework Connector.
  - El adaptador inicializa el contexto para cada turno.
  - Para más información, consulte [cómo funcionan los bots](../bot-builder-basics.md).
- Administración de estados refactorizada.
  - Los datos de estado ya no están disponibles automáticamente dentro de un bot.
  - Ahora, el estado se administra mediante objetos de administración de estado y descriptores de acceso de propiedad.
  - Para más información, consulte la [administración del estado](../bot-builder-concept-state.md).
- Nueva biblioteca de diálogos.
  - Los diálogos de la versión v3 tendrá que volver a escribirse para la nueva biblioteca de diálogos.
  - Ya no existen componentes puntuables. Puede buscar comandos "globales" en el controlador de turnos, antes de pasar el control a los diálogos.
  - Para más información, consulte la [biblioteca de diálogos](../bot-builder-concept-dialog.md).
- Compatibilidad con ASP.NET Core
  - Las plantillas para crear nuevos bots de C# tienen como destino la plataforma ASP.NET Core.
  - Puede seguir usando ASP.NET para sus bots, pero nuestro objetivo en la versión v4 es dar soporte a la plataforma ASP.NET Core.
  - Consulte la [introducción a ASP.NET Core](https://docs.microsoft.com/aspnet/core/) para más información acerca de esta plataforma.

## <a name="activity-processing"></a>Procesamiento de actividades

Al crear el adaptador para su bot, también proporciona un delegado de controlador de turnos que recibirá las actividades entrantes de los canales y los usuarios. El adaptador crea un objeto de contexto de turno para cada actividad recibida. Pasa el objeto de contexto de turno al controlador de turnos y luego elimina el objeto cuando el turno se completa.

El controlador de turnos puede recibir muchos tipos de actividades. En general, querrá reenviar solo las actividades de _mensaje_ a todos los diálogos que su bot contiene. Para más información acerca de los tipos de actividad, consulte el [esquema de actividades](https://aka.ms/botSpecs-activitySchema).

### <a name="handling-turns"></a>Control de turnos

El controlador de turnos debe coincidir con la firma de `BotCallbackHandler`:

```csharp
public delegate Task BotCallbackHandler(
    ITurnContext turnContext,
    CancellationToken cancellationToken);
```

Al controlar un turno, use el contexto de turno para obtener más información acerca de la actividad entrante y para enviar actividades al usuario:

| | |
|-|-|
| Para obtener la actividad entrante | Obtenga la propiedad `Activity` del contexto de turno. |
| Para crear y enviar una actividad al usuario | Llame al método `SendActivityAsync` del contexto de turno.<br/>Para más información, consulte cómo [enviar y recibir un mensaje de texto](../bot-builder-howto-send-messages.md) y [agregar multimedia a los mensajes](../bot-builder-howto-add-media-attachments.md). |

La clase `MessageFactory` proporciona algunos métodos auxiliares para las actividades de creación y formato.

### <a name="scorables-is-gone"></a>Los componentes puntuables han desaparecido

Se administran en el bucle de mensajes del bot. Para obtener una descripción de cómo hacer esto con los diálogos de la versión v4, consulte cómo [controlar las interrupciones de usuario](../bot-builder-howto-handle-user-interrupt.md).

Los árboles de envío de componentes puntuables que admiten composición y los diálogos en cadena que admiten composición, como la _excepción predeterminada_, también han desaparecido. Una manera de reproducir esta funcionalidad es implementarla dentro de controlador de turnos del bot.

## <a name="state-management"></a>Administración de estados

La versión v4 no utiliza las propiedades `UserData`, `ConversationData` y `PrivateConversationData` ni contenedores de datos para administrar el estado.
Ahora, el estado se administra mediante objetos de administración de estado y descriptores de acceso de propiedad, tal y como se describe en [Administración del estado](../bot-builder-concept-state.md).

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

Consulte cómo [guardar el estado](../bot-builder-concept-state.md#saving-state) para más información.

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

Ahora hay varias opciones para definir los diálogos:

- Un cuadro en cascada, una instancia de la clase `WaterfallDialog`.

  Esto funciona bien con los diálogos de aviso, que solicitarán y validarán diversos tipos de datos de entrada del usuario. Consulte cómo [pedir datos de entrada](../bot-builder-prompts.md).

  Esto automatiza la mayor parte del proceso, pero impone que el código se haga de una determinada manera; consulte el [flujo secuencial de conversación](../bot-builder-dialog-manage-conversation-flow.md). Sin embargo, puede crear otros flujos de control incorporando de varios diálogos a un conjunto de diálogos; consulte el [flujo de conversación avanzada](../bot-builder-dialog-manage-complex-conversation-flow.md).

- Un componente de diálogo, derivado de la clase `ComponentDialog`.

  Esto le permite encapsular el código del diálogo sin conflictos de nombres con los contextos externos. Consulte cómo [reutilizar los diálogos](../bot-builder-compositcontrol.md).

- Un diálogo personalizado, derivado de la clase abstracta `Dialog`.

  Esto ofrece la máxima flexibilidad en cuanto a cómo se comportan los diálogos, pero también necesita conocer mejor cómo se implementa la pila de diálogos.

Para acceder a un diálogo, deberá poner una instancia de él en un _conjunto de diálogos_ y, después, generar un _contexto de diálogo_ para ese conjunto.

Debe proporcionar un descriptor de acceso de propiedad de estado de diálogo al crear el conjunto de diálogos. Esto permite al marco conservar el estado del diálogo de un turno al siguiente. En [Administración del estado](../bot-builder-concept-state.md) se describe cómo administrar el estado en la versióon v4.

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

- En la versión v4, las distintas clases derivadas `Prompt` implementan los avisos al usuario como diálogos independientes en dos pasos. Consulte cómo [recopilar datos de entrada del usuario mediante un aviso de diálogo](../bot-builder-prompts.md).
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
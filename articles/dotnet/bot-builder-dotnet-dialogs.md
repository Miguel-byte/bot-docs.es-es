---
title: Introducción a los diálogos | Microsoft Docs
description: Aprenda a usar diálogos en el SDK de Bot Builder para Node.js para modelar conversaciones y administrar el flujo de conversación.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 943b206e4991c52f22928d2113977249ff9d9e04
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997582"
---
# <a name="dialogs-in-the-bot-builder-sdk-for-net"></a>Diálogos en el SDK de Bot Builder para .NET

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-dialogs.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-overview.md)

Al crear un bot mediante el SDK de Bot Builder para .NET, puede usar diálogos para modelar una conversación y administrar el [flujo de conversación](../bot-service-design-conversation-flow.md). Cada cuadro de diálogo es una abstracción que encapsula su propio estado en una clase de C# que implementa `IDialog`. Un diálogo puede estar formado por otros diálogos para maximizar la reutilización, y un contexto de diálogo mantiene la [pila de diálogos](../bot-service-design-conversation-flow.md#dialog-stack) que están activos en la conversación en cualquier momento dado. 

Una conversación que consta de diálogos se puede transferir entre varios equipos, lo que permite escalar la implementación del bot. Cuando usa diálogos en el SDK de Bot Builder para. NET, el estado de la conversación (la pila de diálogos y el estado de cada diálogo de la pila) se almacena automáticamente en el almacenamiento de [datos de estado](bot-builder-dotnet-state.md) de su elección. Esto permite que el código de servicio del bot sea sin estado, de forma muy parecida a una aplicación web que no necesita almacenar el estado de sesión en la memoria del servidor web. 

## <a name="echo-bot-example"></a>Ejemplo de repetición de bot

Considere este ejemplo de repetición de bot, que describe cómo cambiar el bot que se crea en el tutorial [Inicio rápido](bot-builder-dotnet-quickstart.md) para que use los diálogos para intercambiar mensajes con el usuario.

> [!TIP]
> Para seguir este ejemplo, use las instrucciones del tutorial [Inicio rápido](bot-builder-dotnet-quickstart.md) para crear un bot y, a continuación, actualice su archivo **MessagesController.cs**, como se describe a continuación.

### <a name="messagescontrollercs"></a>MessagesController.cs 

En el SDK de Bot Builder para .NET, la biblioteca de [Builder][builderLibrary] le permite implementar diálogos. Para acceder a las clases pertinentes, importe el espacio de nombres `Dialogs`.

[!code-csharp[Using statement](../includes/code/dotnet-dialogs.cs#usingStatement)]

A continuación, agregue esta clase `EchoDialog` a **MessagesController.cs** para representar la conversación. 

[!code-csharp[EchoDialog class](../includes/code/dotnet-dialogs.cs#echobot1)]

Luego, escriba la clase `EchoDialog` en el método `Post` mediante una llamada al método `Conversation.SendAsync`.

[!code-csharp[Post method](../includes/code/dotnet-dialogs.cs#echobot2)]

### <a name="implementation-details"></a>Detalles de la implementación 

El método `Post` está marcado como `async` porque Bot Builder usa las funciones de C# para controlar la comunicación asincrónica. Devuelve un objeto `Task`, que representa la tarea que es responsable de enviar las respuestas al mensaje pasado. Si se produce una excepción, el objeto `Task` que devuelve el método contendrá la información de la excepción. 

El método `Conversation.SendAsync` es clave para implementar diálogos con el SDK de Bot Builder para. NET. Sigue el <a href="https://en.wikipedia.org/wiki/Dependency_inversion_principle" target="_blank">principio de inversión de dependencia</a> y lleva a cabo estos pasos:

1. Crea una instancia de los componentes necesarios.  
2. Deserializa el estado de conversación (la pila de diálogos y el estado de cada diálogo de la pila) `IBotDataStore`.
3. Reanuda el proceso de conversación donde el bot se suspendió y espera un mensaje.
4. Envía las respuestas.
5. Serializa el estado de conversación actualizado y lo guarda de nuevo en `IBotDataStore`.

Cuando la conversación se inicia por primera vez, el diálogo no contiene estado, por lo que `Conversation.SendAsync` construye `EchoDialog` y llama a su método `StartAsync`. El método `StartAsync` llama a `IDialogContext.Wait` con el delegado de continuación para especificar el método que se debe invocar cuando se recibe un mensaje nuevo (`MessageReceivedAsync`). 

El método `MessageReceivedAsync` espera un mensaje, envía una respuesta y espera el siguiente mensaje. Cada vez que se llama a `IDialogContext.Wait`, el bot entra en un estado suspendido y se puede reiniciar en cualquier equipo que reciba el mensaje. 

Un bot que se crea mediante los ejemplos de código anteriores responderá a cada mensaje que el usuario envía devolviendo simplemente el mensaje del usuario con el texto delante "Dijo: ". Como el bot se crea mediante diálogos, puede evolucionar para admitir conversaciones más complejas sin tener que administrar explícitamente el estado.

## <a name="echo-bot-with-state-example"></a>Ejemplo de repetición de bot con estado

El ejemplo siguiente se basa en el anterior y se agrega la posibilidad de realizar el seguimiento del estado de los diálogos. Cuando se actualiza la clase `EchoDialog`, como se muestra en el ejemplo de código anterior, el bot responderá a cada mensaje que envía el usuario devolviéndolo con un número delante (`count`) seguido del texto "Dijo: ". El bot sigue aumentando `count` con cada respuesta, hasta que el usuario elige restablecer el recuento.

### <a name="messagescontrollercs"></a>MessagesController.cs 

[!code-csharp[EchoDialog class](../includes/code/dotnet-dialogs.cs#echobot3)]

### <a name="implementation-details"></a>Detalles de la implementación

Como en el primer ejemplo, se llama al método `MessageReceivedAsync` cuando se recibe un nuevo mensaje. Sin embargo, esta vez el método `MessageReceivedAsync` evalúa el mensaje del usuario antes de responder. Si el mensaje del usuario es "restablecer", el mensaje integrado `PromptDialog.Confirm` genera un diálogo secundario que pide al usuario que confirme el restablecimiento del recuento. El diálogo secundario tiene su propio estado privado que no interfiere con el estado del diálogo principal. Cuando el usuario responde al mensaje, el resultado del diálogo secundario se pasa al método `AfterResetAsync`, que envía un mensaje al usuario para indicar si se restableció o no el recuento y, a continuación, llama a `IDialogContext.Wait` con una continuación de vuelta a `MessageReceivedAsync` en el siguiente mensaje.

## <a name="dialog-context"></a>Contexto del diálogo

La interfaz `IDialogContext` que se pasa a cada método de diálogo proporciona acceso a los servicios que necesita un diálogo para guardar el estado y comunicarse con el canal. La interfaz `IDialogContext` consta de tres interfaces: [Internals.IBotData][iBotData], [Internals.IBotToUser][iBotToUser] e [Internals.IDialogStack][iDialogStack]. 

### <a name="internalsibotdata"></a>Internals.IBotData

La interfaz `Internals.IBotData` proporciona acceso a los datos de estado por usuario, por conversación y de conversación privada que mantiene Connector. Los datos de estado por usuario son útiles para almacenar datos de usuario que no están relacionados con una conversación específica, mientras que los datos por conversación resultan de utilidad para almacenar datos generales sobre una conversación; los datos de conversación privada son útiles para almacenar datos de usuario relacionados con una conversación en particular. 

### <a name="internalsibottouser"></a>Internals.IBotToUser

`Internals.IBotToUser` proporciona métodos para enviar un mensaje del bot al usuario. Se pueden enviar mensajes en línea con la respuesta a la llamada del método de API web o directamente mediante el [cliente de Connector](bot-builder-dotnet-connector.md#create-a-connector-client). Al enviar y recibir mensajes mediante el contexto del diálogo se garantiza que el estado `Internals.IBotData` se pasa a través de Connector.

### <a name="internalsidialogstack"></a>Internals.IDialogStack

`Internals.IDialogStack` proporciona métodos para administrar la [pila de diálogos](../bot-service-design-conversation-flow.md#dialog-stack). La mayor parte del tiempo, la pila de diálogos se administrará automáticamente. Sin embargo, puede haber casos donde desee administrar explícitamente la pila. Por ejemplo, puede que quiera llamar a un diálogo secundario y agregarlo al principio de la pila de diálogos, marcar el diálogo actual como completo (y así quitarlo de la pila de diálogos y devolver el resultado al diálogo anterior en la pila), suspender el diálogo actual hasta que llegue un mensaje del usuario o incluso restablecer toda la pila de diálogos.

## <a name="serialization"></a>Serialización

La pila de diálogos y el estado de todos los diálogos activos se serializan en [IBotDataBag][iBotDataBag] por usuario y por conversación. El blob serializado se conserva en los mensajes que el bot envía y recibe de [Connector](bot-builder-dotnet-concepts.md#connector). Para serializar una clase `Dialog`, debe incluir el atributo `[Serializable]`. Todas las implementaciones de `IDialog` de la biblioteca de [Builder][builderLibrary] están marcadas como serializables. 

Los [métodos de cadena](#dialog-chains) proporcionan una interfaz fluida para los diálogos que se puede usar en la sintaxis de consulta LINQ. La forma compilada de la sintaxis de consulta LINQ usa con frecuencia métodos anónimos. Si estos métodos anónimos no hacen referencia al entorno de variables locales, no tienen estado y son mínimamente serializables. Sin embargo, si el método anónimo captura cualquier variable local del entorno, el objeto de cierre resultante (generado por el compilador) no está marcado como serializable. En esta situación, Bot Builder produce una excepción `ClosureCaptureException` para identificar el problema.

Para usar la reflexión para serializar clases que no están marcadas como serializables, la biblioteca de Builder incluye una serialización suplente basada en la reflexión que se puede usar para registrarse con [Autofac][autofac].

[!code-csharp[Serialization](../includes/code/dotnet-dialogs.cs#serialization)]

## <a id="dialog-chains"></a> Cadenas de diálogos

Aunque puede administrar explícitamente la pila de diálogos activos mediante `IDialogStack.Call<R>` y `IDialogStack.Done<R>`, también puede administrarla implícitamente mediante estos métodos fluidos de [cadena][chain].


|           Método            |  Escriba   |                                 Notas                                  |
|-----------------------------|---------|------------------------------------------------------------------------|
|     Chain.Select<T, R>      |  LINQ   |           Admite "select" y "let" en la sintaxis de consulta LINQ.            |
|  Chain.SelectMany<T, C, R>  |  LINQ   |            Admite sucesivos "from" en la sintaxis de consulta LINQ.            |
|       Chain.Where<T>        |  LINQ   |                 Admite "where" en la sintaxis de consulta LINQ.                 |
|        Chain.From<T>        | Cadenas  |                Crea una instancia de un diálogo.                |
|       Chain.Return<T>       | Cadenas  |                Devuelve un valor constante en la cadena.                |
|         Chain.Do<T>         | Cadenas  |               Permite efectos secundarios dentro de la cadena.                |
|  Chain.ContinueWith<T, R>   | Cadenas  |                      Encadenamiento simple de diálogos.                       |
|       Chain.Unwrap<T>       | Cadenas  |                  Desempaqueta un diálogo anidado en un diálogo.                   |
| Chain.DefaultIfException<T> | Cadenas  | Acepta una excepción del resultado anterior y devuelve default(t). |
|        Chain.Loop<T>        | Rama  |                   Repite la cadena completa de diálogos.                   |
|        Chain.Fold<T>        | Rama  |   Incluye los resultados de una enumeración de diálogos en un único resultado.   |
|     Chain.Switch<T, R>      | Rama  |            Admite la bifurcación en diferentes cadenas de diálogos.            |
|     Chain.PostToUser<T>     | Message |                      Envía un mensaje al usuario.                      |
|     Chain.WaitToBot<T>      | Message |                    Espera un mensaje al bot.                     |
|    Chain.PostToChain<T>     | Message |              Inicia una cadena con un mensaje del usuario.              |

### <a name="examples"></a>Ejemplos 

La sintaxis de consulta LINQ usa el método `Chain.Select<T, R>`.

[!code-csharp[Chain.Select](../includes/code/dotnet-dialogs.cs#chain1)]

O el método `Chain.SelectMany<T, C, R>`.

[!code-csharp[Chain.SelectMany](../includes/code/dotnet-dialogs.cs#chain2)]

Los métodos `Chain.PostToUser<T>` y `Chain.WaitToBot<T>` envían mensajes del bot al usuario y viceversa.

[!code-csharp[Chain.PostToUser](../includes/code/dotnet-dialogs.cs#chain3)]

El método `Chain.Switch<T, R>` bifurca el flujo de diálogos de la conversación.

[!code-csharp[Chain.Switch](../includes/code/dotnet-dialogs.cs#chain4)]

Si `Chain.Switch<T, R>` devuelve un elemento `IDialog<IDialog<T>>` anidado, entonces el elemento `IDialog<T>` interno se puede desempaquetar con `Chain.Unwrap<T>`. Esto permite la bifurcación de conversaciones a diferentes rutas de acceso de diálogos encadenados, posiblemente con una longitud desigual. Este es un ejemplo más completo de bifurcación de diálogos escritos en el estilo de cadena fluida con administración implícita de la pila.

[!code-csharp[Chain.Switch](../includes/code/dotnet-dialogs.cs#chain5)]

## <a name="next-steps"></a>Pasos siguientes

Los diálogos administran el flujo de conversación entre un bot y un usuario. Un diálogo define cómo interactuar con un usuario. Un bot puede usar muchos diálogos organizados en pilas para guiar la conversación con el usuario. En la siguiente sección, vea cómo la pila de diálogos crece y decrece a medida que crea y descarta diálogos de la pila.

> [!div class="nextstepaction"]
> [Administración del flujo de conversación con diálogos](bot-builder-dotnet-manage-conversation-flow.md)


[builderLibrary]: /dotnet/api/microsoft.bot.builder.dialogs

[iBotData]: /dotnet/api/microsoft.bot.builder.dialogs.internals.ibotdata

[iBotToUser]: /dotnet/api/microsoft.bot.builder.dialogs.internals.ibottouser

[iDialogStack]: /dotnet/api/microsoft.bot.builder.dialogs.internals.idialogstack

[iBotDataBag]: /dotnet/api/microsoft.bot.builder.dialogs.ibotdatabag

[autofac]: /dotnet/api/microsoft.bot.builder.autofac.base

[chain]: /dotnet/api/microsoft.bot.builder.dialogs.chain

---
title: Implementación de un flujo de conversación secuencial | Microsoft Docs
description: Aprenda a administrar un flujo de conversación simple con diálogos en Bot Framework SDK.
keywords: flujo de conversación simple, flujo de conversación secuencial, diálogos, avisos, cascadas, conjunto de diálogos
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 07/05/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 27d7e5ee6edd4cedfb9d59b318d9a3765e2f0ad8
ms.sourcegitcommit: 9e1034a86ffdf2289b0d13cba2bd9bdf1958e7bc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2019
ms.locfileid: "69890608"
---
# <a name="implement-sequential-conversation-flow"></a>Implementación de un flujo de conversación secuencial

[!INCLUDE[applies-to](../includes/applies-to.md)]

La recopilación de información mediante la publicación de preguntas es una de las principales formas de interacción de un bot con los usuarios. La biblioteca de diálogos proporciona características integradas útiles, como las clases *prompt* que facilitan la formulación de preguntas y validan la respuesta para asegurarse de que coincide con un tipo de datos específico o cumple con las reglas de validación personalizadas.

Mediante la biblioteca Dialogs es posible administrar flujos de conversación simples y complejos. En una interacción simple, el bot ejecuta una secuencia fija de pasos y la conversación finaliza. En general, un diálogo es útil cuando el bot necesita recopilar información del usuario. En este tema se detalla cómo implementar un flujo de conversación simple mediante la creación de avisos y su llamada desde un diálogo en cascada.

> [!TIP]
> Para obtener ejemplos de cómo escribir sus propias preguntas sin usar la biblioteca de diálogos, vea el artículo [Creación de mensajes propios para recopilar datos de entrada del usuario](bot-builder-primitive-prompts.md).

## <a name="prerequisites"></a>Requisitos previos

- Conocimiento de los [conceptos básicos de bots][concept-basics], la [administración del estado][concept-state] y la [biblioteca de diálogos][concept-dialogs].
- Una copia del ejemplo de **aviso con varios turnos** en [**CSharp**][cs-sample] o [**JavaScript**][js-sample].

## <a name="about-this-sample"></a>Acerca de este ejemplo

En el ejemplo de aviso de varios turnos, usamos un diálogo en cascada, algunos avisos y un diálogos de componente para crear una interacción simple que formula al usuario una serie de preguntas. El código usa un diálogo para desplazarse por estos pasos:

| Pasos        | Tipo de aviso  |
|:-------------|:-------------|
| Pedir al usuario su modo de transporte | Aviso de elección |
| Pedir al usuario su nombre | Aviso de texto |
| Pedir al usuario si desea proporcionar su edad | Aviso de confirmación |
| Si responde Sí, solicitar su edad  | Aviso numérico con validación para que solo acepte edades mayores que 0 y menores de 150. |
| Preguntar si la información recopilada es correcta | Reutilización del aviso de confirmación |

Por último, si responde Sí, mostrar la información recopilada; de lo contrario, indicar al usuario que no se conservará su información.

## <a name="create-the-main-dialog"></a>Creación del diálogo principal

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Para usar diálogos, instale el paquete de NuGet **Microsoft.Bot.Builder.Dialogs**.

El bot interactúa con el usuario mediante `UserProfileDialog`. Cuando se crea la clase `DialogBot` del bot, se establece `UserProfileDialog` como su diálogo principal. El bot, a continuación, usa un método auxiliar `Run` para acceder al diálogo.

![Diálogo de perfil de usuario](media/user-profile-dialog.png)

**Dialogs\UserProfileDialog.cs**

Comenzamos por la creación de `UserProfileDialog`, que se deriva de la clase `ComponentDialog` y tiene 6 pasos.

En el constructor `UserProfileDialog`, se crean los pasos de cascada, los avisos y el diálogo en cascada y se agregan al conjunto de diálogos. Los avisos deben estar en el mismo conjunto de diálogos en el que se utilizan.

[!code-csharp[Constructor snippet](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=22-41)]

A continuación, implementamos los pasos que se usa el diálogo. Para usar un aviso, debe llamarlo desde un paso del diálogo y recuperar el resultado del aviso en el paso siguiente con `stepContext.Result`. En un segundo plano, las preguntas consisten en un diálogo de dos pasos. En primer lugar, el aviso pide una entrada; en segundo lugar, devuelve el valor válido o comienza de nuevo desde el principio con un nuevo aviso hasta que recibe una entrada válida.

Siempre debe devolver un valor no NULL de `DialogTurnResult` desde un paso de cascada. Si no lo hace, el diálogo podría no funcionar según lo previsto. Aquí se muestra la implementación de `NameStepAsync` en el diálogo en cascada.

[!code-csharp[Name step](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=56-61)]

En `AgeStepAsync`, especificamos un aviso de reintento cuando se produce un error al validar la entrada del usuario, ya sea porque tiene un formato que el aviso no puede analizar o porque se produce un error en un criterio de validación de la entrada. En este caso, si no se ha proporcionado un aviso de reintento, el aviso utilizará el texto del aviso inicial para volver a pedir la entrada al usuario.

[!code-csharp[Age step](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=74-93&highlight=10)]

**UserProfile.cs**

El modo de transporte, el nombre y la edad del usuario se guardan en una instancia de la clase `UserProfile`.

[!code-csharp[UserProfile class](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/UserProfile.cs?range=9-16)]

**Dialogs\UserProfileDialog.cs**

En el último paso, comprobamos el valor de `stepContext.Result` devuelto por el diálogo que se llama en el paso de cascada anterior. Si el valor devuelto es true, se usa el descriptor de acceso del perfil de usuario para obtener y actualizar el perfil de usuario. Para obtener el perfil de usuario, llamamos al método `GetAsync` y, a continuación, se establecen los valores de las propiedades `userProfile.Transport`, `userProfile.Name` y `userProfile.Age`. Por último, se resume la información del usuario antes de llamar a `EndDialogAsync`, que finaliza el diálogo. La finalización del diálogo lo extrae de la pila de diálogos y devuelve un resultado opcional al elemento primario del diálogo. El elemento primario es el diálogo o método que inició el diálogo que acaba de terminar.

[!code-csharp[SummaryStepAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=108-134&highlight=5-10,25-26)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Para usar diálogos, el proyecto debe instalar el paquete de npm **botbuilder-dialogs**.

El bot interactúa con el usuario mediante `UserProfileDialog`. Cuando se crea el elemento `DialogBot` del bot, se establece `UserProfileDialog` como su diálogo principal. El bot, a continuación, usa un método auxiliar `run` para acceder al diálogo.

![Diálogo de perfil de usuario](media/user-profile-dialog-js.png)

**dialogs\userProfileDialog.js**

Comenzamos por la creación de `UserProfileDialog`, que se deriva de la clase `ComponentDialog` y tiene 6 pasos.

En el constructor `UserProfileDialog`, se crean los pasos de cascada, los avisos y el diálogo en cascada y se agregan al conjunto de diálogos. Los avisos deben estar en el mismo conjunto de diálogos en el que se utilizan.

[!code-javascript[Constructor snippet](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=25-45)]

A continuación, implementamos los pasos que se usa el diálogo. Para usar un aviso, debe llamarlo desde un paso del diálogo y recuperar el resultado del aviso en el paso siguiente del contexto del paso, en este caso con `step.result`. En un segundo plano, las preguntas consisten en un diálogo de dos pasos. En primer lugar, el aviso pide una entrada; en segundo lugar, devuelve el valor válido o comienza de nuevo desde el principio con un nuevo aviso hasta que recibe una entrada válida.

Siempre debe devolver un valor no NULL de `DialogTurnResult` desde un paso de cascada. Si no lo hace, el diálogo podría no funcionar según lo previsto. Aquí se muestra la implementación de `nameStep` en el diálogo en cascada.

[!code-javascript[name step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=73-76)]

En `ageStep`, especificamos un aviso de reintento cuando se produce un error al validar la entrada del usuario, ya sea porque tiene un formato que el aviso no puede analizar o porque se produce un error en un criterio de validación de la entrada, como se especifica en el constructor anterior. En este caso, si no se ha proporcionado un aviso de reintento, el aviso utilizará el texto del aviso inicial para volver a pedir la entrada al usuario.

[!code-javascript[age step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=88-99&highlight=5)]

**userProfile.js**

El modo de transporte, el nombre y la edad del usuario se guardan en una instancia de la clase `UserProfile`.

[!code-javascript[user profile](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/userProfile.js?range=4-10)]

**dialogs\userProfileDialog.js**

En el último paso, comprobamos el valor de `step.result` devuelto por el diálogo que se llama en el paso de cascada anterior. Si el valor devuelto es true, se usa el descriptor de acceso del perfil de usuario para obtener y actualizar el perfil de usuario. Para obtener el perfil de usuario, llamamos al método `get` y, a continuación, se establecen los valores de las propiedades `userProfile.transport`, `userProfile.name` y `userProfile.age`. Por último, se resume la información del usuario antes de llamar a `endDialog`, que finaliza el diálogo. La finalización del diálogo lo extrae de la pila de diálogos y devuelve un resultado opcional al elemento primario del diálogo. El elemento primario es el diálogo o método que inició el diálogo que acaba de terminar.

[!code-javascript[summary step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=113-134&highlight=4-8,20-21)]

**Creación del método de extensión para ejecutar el diálogo en cascada**

Hemos definido un método auxiliar `run` dentro de `userProfileDialog` que se usará para crear y acceder al contexto del diálogo. En este caso, `accessor` es el descriptor de acceso de la propiedad de estado de la propiedad de estado del diálogo y `this` es el diálogo de componente de perfil de usuario. Puesto que los diálogos de componente definen un conjunto de diálogos interno, debemos crear un conjunto de diálogos externo que sea visible para el código del controlador de mensajes y usarlo para crear un contexto de diálogo.

El contexto del diálogo se crea mediante una llamada al método `createContext` y se usa para interactuar con el conjunto de diálogos desde dentro del controlador de turnos del bot. El contexto del diálogo incluye el contexto del turno actual, el diálogo primario y el estado del diálogo, que proporciona un método para conservar información en el diálogo.

El contexto del diálogo permite iniciar un diálogo con el identificador de cadena o continuar el diálogo actual (por ejemplo, un diálogo en cascada que tiene varios pasos). El contexto del diálogo se pasa a todos los diálogos y pasos de cascada del bot.

[!code-javascript[run method](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=53-62)]

---

## <a name="run-the-dialog"></a>Ejecución del diálogo

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\DialogBot.cs**

El controlador `OnMessageActivityAsync` utiliza el método `RunAsync` para iniciar o continuar el diálogo. En `OnTurnAsync`, usamos los objetos de administración de estado del bot para conservar los cambios de estado en el almacenamiento. (El método `ActivityHandler.OnTurnAsync` llama a los diversos métodos del controlador de actividades, como `OnMessageActivityAsync`. De este modo, se guarda el estado después de que finaliza el controlador de mensajes pero antes de la finalización del propio turno).

[!code-csharp[overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Bots/DialogBot.cs?range=33-48&highlight=5-7)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El controlador `onMessage` utiliza el método auxiliar para iniciar o continuar el diálogo. En `onDialog`, usamos los objetos de administración de estado del bot para conservar los cambios de estado en el almacenamiento. (El método `onDialog` se llama por última vez después de la ejecución de otros controladores definidos, como `onMessage`. De este modo, se guarda el estado después de que finaliza el controlador de mensajes pero antes de la finalización del propio turno).

**bots/dialogBot.js**

[!code-javascript[overrides](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/bots/dialogBot.js?range=24-38&highlight=11-13)]

---

## <a name="register-services-for-the-bot"></a>Registro de los servicios del bot

Este bot usa los siguientes _servicios_.

- Servicios básicos de un bot: un proveedor de credenciales, un adaptador y la implementación del bot.
- Servicios para administrar el estado: almacenamiento, estado del usuario y estado de la conversación.
- El diálogo que va a usar el bot.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs**

Registramos los servicios del bot en `Startup`. Estos servicios están disponibles para otras partes del código mediante la inserción de dependencias.

[!code-csharp[ConfigureServices](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Startup.cs?range=17-39)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

Registramos los servicios del bot en `index.js`.

[!code-javascript[overrides](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/index.js?range=18-46)]

---

> [!NOTE]
> El almacenamiento en memoria se usa solo con fines de prueba y no está pensado para su uso en producción.
> Asegúrese de usar un tipo de almacenamiento persistente para un bot de producción.

## <a name="to-test-the-bot"></a>Prueba del bot

1. Si aún no lo ha hecho, instale [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).
1. Ejecute el ejemplo localmente en la máquina.
1. Inicie el emulador, conéctese al bot y envíe mensajes como se muestra a continuación.

![Ejecución de ejemplo del diálogo de aviso de varios turnos](../media/emulator-v4/multi-turn-prompt.png)

## <a name="additional-information"></a>Información adicional

### <a name="about-dialog-and-bot-state"></a>Acerca del estado del diálogo y el bot

En este bot, definimos dos descriptores de acceso de propiedad de estado:

- Uno se crea en el estado de la conversación de la propiedad de estado del diálogo. El estado del diálogo realiza un seguimiento de dónde está el usuario dentro de los diálogos de un conjunto de diálogos y el contexto de diálogo lo actualiza, del mismo modo que cuando se llama a los métodos begin dialog o continue dialog.
- Uno creado en el estado de usuario de la propiedad de perfil de usuario. El bot lo utiliza para realizar el seguimiento de la información que tiene sobre el usuario y este estado se administra explícitamente en el código del diálogo.

Los métodos _get_ y _set_ del descriptor de acceso de propiedad de estado obtienen y establecen el valor de la propiedad en la memoria caché del objeto de administración de estado. La memoria caché se rellena la primera vez que se solicita el valor de una propiedad de estado en un turno, pero se debe guardar explícitamente. Para conservar los cambios en ambas propiedades de estado, llamamos al método _save changes_ del objeto de administración de estado correspondiente.

En este ejemplo se actualiza el estado del perfil de usuario desde el diálogo. Esta práctica puede funcionar para un bot simple, pero no funcionará si desea reutilizar un diálogo en varios bots.

Hay varias opciones para mantener independientes los pasos del diálogo y el estado del bot. Por ejemplo, una vez que el diálogo recopila toda la información, puede:

- Usar el método end dialog para proporcionar los datos recopilados como valor devuelto al contexto primario. Este puede ser el controlador de turnos del bot o un diálogo activo anterior en la pila de diálogos. Así se han diseñado las clases del aviso.
- Generar una solicitud a un servicio adecuado. Esto podría funcionar bien si el bot actúa como un front-end de un servicio de mayor tamaño.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Incorporación de comprensión lingüística natural al bot](bot-builder-howto-v4-luis.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-multi-prompts-sample
[js-sample]: https://aka.ms/js-multi-prompts-sample

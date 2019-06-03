---
title: Migrar un bot existente dentro del mismo proyecto de .NET Framework | Microsoft Docs
description: Tomamos un bot v3 existente y lo migramos al SDK v4, utilizando el mismo proyecto.
keywords: bot migration, formflow, dialogs, v3 bot
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: aca21d9af94f274936900f1d73c1b340272cd089
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215614"
---
# <a name="migrate-a-net-v3-bot-to-a-framework-v4-bot"></a>Migración de un bot de .NET v3 a un bot de Framework v4

En este artículo, convertiremos el bot de la versión v3 [ContosoHelpdeskChatBot](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V3) en un bot de la versión v4 _sin convertir el tipo de proyecto_. Seguirá siendo un proyecto de .NET Framework.
Esta conversión se divide en estos pasos:

1. Actualización e instalación de los paquetes NuGet
1. Actualización del archivo Global.asax.cs
1. Actualización de la clase MessagesController
1. Conversión de los cuadros de diálogo

El resultado de esta conversión es [ContosoHelpdeskChatBot de .NET Framework v4](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework).

Bot Framework SDK v4 se basa en las mismas API REST subyacentes que el SDK v3. Sin embargo, el SDK v4 es una refactorización de la versión anterior que ofrece a los desarrolladores más flexibilidad y control sobre sus bots. Algunos cambios importantes en el SDK son:

- El estado se administra mediante objetos de administración de estado y descriptores de acceso de propiedad.
- Ha cambiado el controlador de turnos y la forma de pasarle actividades.
- Ya no existen componentes puntuables. Puede buscar comandos "globales" en el controlador de turnos, antes de pasar el control a los diálogos.
- Una nueva biblioteca de diálogos muy diferente de la existente en la versión anterior. Deberá convertir los diálogos antiguos al nuevo sistema de diálogos con los diálogos de componente y en cascada, y la implementación de la comunidad de los diálogos Formflow para la versión v4.

Para más información sobre los cambios específicos, consulte las [diferencias entre las versiones v3 y v4 del SDK para .NET](migration-about.md).

## <a name="update-and-install-nuget-packages"></a>Actualización e instalación de los paquetes NuGet

1. Actualice **Microsoft.Bot.Builder.Azure** y **Microsoft.Bot.Builder.Integration.AspNet.WebApi** a la versión estable más reciente.

    Esto actualizará también los paquetes **Microsoft.Bot.Builder** y **Microsoft.Bot.Connector**, porque son dependencias.

1. Elimine el paquete **Microsoft.Bot.Builder.History**. No forma parte del SDK v4.
1. Agregue **Autofac.WebApi2**.

    Vamos a usarlo como ayuda para la inserción de dependencias en ASP.NET.

1. Agregue **Bot.Builder.Community.Dialogs.Formflow**.

    Esta es una biblioteca de la comunidad para crear diálogos de la versión v4 a partir de archivos de definición de Formflow v3. Tiene **Microsoft.Bot.Builder.Dialogs** como una de sus dependencias, así que también se instala.

Si compila en este momento, recibirá errores del compilador. Puede pasarlos por alto. Cuando hayamos terminado con la conversión, tendremos un código que funciona.

## <a name="update-your-globalasaxcs-file"></a>Actualización del archivo Global.asax.cs

Algunas de las técnicas de scaffolding han cambiado y, en la versión v4, tenemos que configurar partes de la [administración del estado](../bot-builder-concept-state.md) nosotros mismos. Por ejemplo, en la versión v4 se utiliza un adaptador de bot para controlar la autenticación y las actividades de reenvío al código del bot, y tenemos que declarar nuestras propiedades de estado por adelantado.

Vamos a crear una propiedad de estado para `DialogState`, que ahora es necesario para admitir los diálogos en la versión v4. Vamos a usar la inserción de dependencias para obtener la información necesaria para el controlador y el código del bot.

En **Global.asax.cs**:

1. Actualice las instrucciones `using`: [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Global.asax.cs?range=4-13)]

1. Quite estas líneas del método **Application_Start**: [!code-csharp[Removed lines](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V3/ContosoHelpdeskChatBot/Global.asax.cs?range=23-24)]

    E inserte esta línea: [!code-csharp[Reference BotConfig.Register](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Global.asax.cs?range=22)]

1. Quite el método **RegisterBotModules**, al que ya no se hace referencia.

1. Reemplace el método **BotConfig.UpdateConversationContainer** por este método **BotConfig.Register**, con el que registraremos los objetos necesarios para admitir la inserción de dependencias. Este bot no utiliza los estados de _usuario_ ni de _conversación privada_, por lo que solo se crea el objeto de administración del estado de la conversación.
    [!code-csharp[Define BotConfig.Register](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Global.asax.cs?range=31-61)]

## <a name="update-your-messagescontroller-class"></a>Actualización de la clase MessagesController

Aquí es donde el bot inicia un turno en v4, por lo que tiene que cambiar mucho. Excepto el propio controlador de turnos del bot, la mayor parte puede considerarse como reutilizable. En su archivo **Controllers\MessagesController.cs**:

1. Actualice las instrucciones `using`: [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Controllers/MessagesController.cs?range=4-8)]

1. Quite el atributo `[BotAuthentication]` de la clase. En la versión v4, el adaptador del bot controlará la autenticación.

1. Agregue estos campos y un constructor para inicializarlos. ASP.NET y Autofac utilizan la inserción de dependencias para obtener los valores de los parámetros. (Registramos los objetos adaptador y bot en **Global.asax.cs** para admitir esto). [!code-csharp[Fields and constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Controllers/MessagesController.cs?range=14-21)]

1. Reemplace el cuerpo del método **Post**. El adaptador se utiliza para llamar al bucle de mensajes del bot (controlador de turnos).
    [!code-csharp[Post method](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Controllers/MessagesController.cs?range=23-31)]

### <a name="delete-the-cancelscorable-and-globalmessagehandlersbotmodule-classes"></a>Eliminación de las clases CancelScorable y GlobalMessageHandlersBotModule

Como no existen componentes puntuables en la versión v4 y hemos actualizado el controlador de turnos para que reaccione a un mensaje `cancel`, podemos eliminar las clases **CancelScorable** (en **Dialogs\CancelScorable.cs**) y  **GlobalMessageHandlersBotModule**.

## <a name="create-your-bot-class"></a>Creación de la clase del bot

En v4, el controlador de turnos o la lógica del bucle de mensajes se encuentran principalmente en un archivo bot. Estamos derivando `ActivityHandler`, que define a los controladores para los tipos comunes de actividades.

1. Cree un archivo **Bots\DialogBots.cs**.

1. Actualice las instrucciones `using`: [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=4-8)]

1. Derive `DialogBot` de `ActivityHandler`y agregue un parámetro genérico para el diálogo.
    [!code-csharp[Class definition](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=19)]

1. Agregue estos campos y un constructor para inicializarlos. De nuevo, ASP.NET y Autofac utilizan la inserción de dependencias para obtener los valores de los parámetros.
    [!code-csharp[Fields and constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=21-28)]

1. Reemplace `OnMessageActivityAsync` para invocar nuestro diálogo principal. (Definiremos el método de extensión `Run` en breve). [!code-csharp[OnMessageActivityAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=38-47)]

1. Reemplace `OnTurnAsync` para guardar el estado de la conversación al final del turno. En v4, tenemos que hacer esto explícitamente para escribir el estado en la capa de persistencia. El método `ActivityHandler.OnTurnAsync` llama a los métodos específicos del controlador de actividades, en función del tipo de actividad recibida, por lo que guardamos el estado después de la llamada al método base.
    [!code-csharp[OnTurnAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=30-36)]

### <a name="create-the-run-extension-method"></a>Creación del método de extensión de ejecución

Estamos creando un método de extensión para consolidar el código necesario para ejecutar un diálogo de componentes desde el bot.

Cree un archivo **DialogExtensions.cs** e implemente un método de extensión `Run`.
[!code-csharp[The extension](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/DialogExtensions.cs?range=4-41)]

## <a name="convert-your-dialogs"></a>Conversión de los cuadros de diálogo

Vamos a hacer muchos cambios en los diálogos originales para migrarlos al SDK v4. Por ahora, no se preocupe por los errores del compilador. Se solucionarán cuando hayamos terminado la conversión.
Para no modificar el código original más de lo necesario, seguirá habiendo algunas advertencias del compilador después de terminar la migración.

Todos nuestros diálogos derivarán de `ComponentDialog`, en lugar de implementar la interfaz `IDialog<object>` de la versión v3.

Este bot tiene cuatro diálogos que necesitamos convertir:

| | |
|---|---|
| [RootDialog](#update-the-root-dialog) | Presenta opciones e inicia los otros diálogos. |
| [InstallAppDialog](#update-the-install-app-dialog) | Controla las solicitudes para instalar una aplicación en un equipo. |
| [LocalAdminDialog](#update-the-local-admin-dialog) | Controla las solicitudes de derechos de administrador del equipo local. |
| [ResetPasswordDialog](#update-the-reset-password-dialog) | Controla las solicitudes para restablecer la contraseña. |

Estos diálogos recopilan datos de entrada, pero no realizan ninguna de estas operaciones en el equipo.

### <a name="make-solution-wide-dialog-changes"></a>Cambios en los diálogos para toda la solución

1. En la solución completa, reemplace todas las apariciones de `IDialog<object>` por `ComponentDialog`.
1. En la solución completa, reemplace todas las apariciones de `IDialogContext` por `DialogContext`.
1. En cada clase de diálogo, quite el atributo `[Serializable]`.

El flujo de control y la mensajería dentro de los diálogos ya no se controlan del mismo modo, por lo que debemos revisarlos cuando convirtamos cada diálogo.

| Operación | Código de la versión v3 | Código de la versión v4 |
| :--- | :--- | :--- |
| Controlar el inicio del diálogo | Implemente `IDialog.StartAsync` | Haga que este sea el primer paso de un diálogo en cascada o implemente `Dialog.BeginDialogAsync` |
| Controlar la continuación del diálogo | Llame a `IDialogContext.Wait` | Agregue pasos adicionales a un diálogo en cascada o implemente `Dialog.ContinueDialogAsync` |
| Enviar un mensaje al usuario | Llame a `IDialogContext.PostAsync` | Llame a `ITurnContext.SendActivityAsync` |
| Iniciar un diálogo secundario | Llame a `IDialogContext.Call` | Llame a `DialogContext.BeginDialogAsync` |
| Señalizar que el diálogo actual se ha completado | Llame a `IDialogContext.Done` | Llame a `DialogContext.EndDialogAsync` |
| Obtener los datos de entrada del usuario | Use un parámetro `IAwaitable<IMessageActivity>` | Use un aviso desde dentro de una cascada o use `ITurnContext.Activity` |

Notas sobre el código de la versión v4:

- Dentro de un código de diálogo, use la propiedad `DialogContext.Context` para obtener el contexto de turno actual.
- Los pasos de la cascada tienen un parámetro `WaterfallStepContext`, que deriva de `DialogContext`.
- Todas las clases específicas de los diálogos y los avisos derivan de la clase abstracta `Dialog`.
- Se asigna un identificador cuando se crea un diálogo de componente. En un conjunto de diálogos, se debe establecer un identificador único para cada diálogo del conjunto.

### <a name="update-the-root-dialog"></a>Actualización del diálogo raíz

En este bot, de diálogo raíz pide al usuario que elija una opción de un conjunto de opciones y, a continuación, inicia un diálogo secundario en función de la opción elegida. Esto se repite en bucle durante la vigencia de la conversación.

- Podemos configurar el flujo principal como un diálogo en cascada, que es un concepto nuevo del SDK v4. Se ejecutará siguiendo unos pasos con un orden fijo. Para más información, consulte [Implementación de flujo de conversación secuencial](~/v4sdk/bot-builder-dialog-manage-conversation-flow.md).
- Ahora, los avisos se controlan mediante clases de aviso, que son diálogos secundarios cortos que piden datos de entrada, realizan algún procesamiento mínimo y la validación, y devuelven un valor. Para más información, consulte [Recopilación de datos de entrada del usuario mediante un aviso de diálogo](~/v4sdk/bot-builder-prompts.md).

En el archivo **Dialogs/RootDialog.cs**:

1. Actualice las instrucciones `using`: [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=4-10)]

1. Necesitamos convertir las opciones `HelpdeskOptions` de una lista de cadenas en una lista de opciones. Se usará con un aviso de opciones, que aceptará como entrada válida el número de opción (en la lista), el valor de la opción o cualquiera de los sinónimos de la opción.
    [!code-csharp[HelpDeskOptions](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=28-33)]

1. Agregue un constructor. Este código hace lo siguiente:
   - Cuando se crea cada instancia de un diálogo, se le asigna un identificador. El identificador de diálogo es parte del conjunto de diálogos al que se va a agregar el diálogo. Recuerde que el bot se inicializó con un objeto de diálogo en la clase **MessageController**. Cada `ComponentDialog` tiene su propio conjunto de diálogos interno, con su propio conjunto de identificadores de diálogo.
   - Agrega los demás diálogos, incluido el aviso de opciones, como diálogos secundarios. En este caso, estamos usando el nombre de la clase en cada identificador de diálogo.
   - Define un diálogo en cascada de tres pasos. Los implementaremos en un momento.
     - En primer lugar, el diálogo pide al usuario que elija la tarea que va a realizar.
     - Después, inicia el diálogo secundario asociado a esa opción.
     - Y, por último, se reinicia a sí mismo.
   - Cada paso de la cascada es un delegado que implementaremos ahora, y tomaremos el código existente del diálogo original siempre que podamos.
   - Al iniciar un diálogo de componente, se iniciará su _initial dialog_. De forma predeterminada, este es el primer diálogo secundario que se agrega a un diálogo de componente. Vamos a establecer explícitamente la propiedad `InitialDialogId`, lo que significa que no es necesario que el diálogo principal de la cascada sea el primero que se agregue al conjunto. Por ejemplo, si prefiere agregar los avisos primero, esto le permitiría hacerlo sin causar problemas en tiempo de ejecución.
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=35-49)]

1. Podemos eliminar el método **StartAsync**. Cuando comienza un diálogo de componente, inicia automáticamente su diálogo _inicial_. En este caso, es el diálogo en cascada que se define en el constructor. También se inicia automáticamente en su primer paso.

1. Eliminaremos los métodos **MessageReceivedAsync** y **ShowOptions** y los reemplazaremos por el primer paso de la cascada. Estos dos métodos saludarán al usuario y le pedirán que elija una de las opciones disponibles.
   - Aquí puede ver la lista de opciones y los mensajes de error y saludo que se proporcionan como opciones en la llamada a nuestro aviso de opciones.
   - No tenemos que especificar el método siguiente al que llamaremos en el diálogo, porque la cascada continuará con el paso siguiente cuando finalice el aviso de opciones.
   - El aviso de opciones se repetirá en bucle hasta que reciba una entrada válida o se cancele la pila completa del diálogo.
    [!code-csharp[PromptForOptionsAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=51-65)]

1. Podemos reemplazar **OnOptionSelected** por el segundo paso de nuestra cascada. Seguiremos iniciando un diálogo secundario en función de la entrada del usuario.
   - El aviso de opciones devuelve un valor `FoundChoice`. Esto se muestra en la propiedad `Result` del contexto del paso. La pila del diálogo trata todos los valores devueltos como objetos. Si el valor devuelto es uno de los diálogos, ya sabe de qué tipo de valor es el objeto. Consulte los [tipos de avisos](../bot-builder-concept-dialog.md#prompt-types) para obtener una lista de lo que devuelve cada tipo de aviso.
   - Como el aviso de opciones no iniciará una excepción, podemos quitar el bloque try-catch.
   - Necesitamos agregar una salida explícita para que este método siempre devuelva un valor adecuado. No se debería llegar nunca a este código pero, si se llega, permitirá que el diálogo "finalice correctamente".
    [!code-csharp[ShowChildDialogAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=67-102)]

1. Por último, reemplace el método **ResumeAfterOptionDialog** anterior por el último paso de nuestra cascada.
    - En lugar de finalizar el diálogo y devolver el número de vale, como hicimos en el diálogo original, vamos a reiniciar la cascada reemplazando la instancia original en la pila por una nueva instancia de sí mismo. Podemos hacerlo porque la aplicación original siempre pasa por alto el valor devuelto (el número de vale) y reinicia el diálogo raíz.
    [!code-csharp[ResumeAfterAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=104-138)]

### <a name="update-the-install-app-dialog"></a>Actualización del diálogo de instalación de la aplicación

El diálogo de instalación de la aplicación realiza algunas tareas lógicas que vamos a configurar como un diálogo en cascada de 4 pasos. La forma de factorizar el código existente en pasos de cascada es un ejercicio lógico para cada diálogo. En cada paso, se indica el método original del que procede el código.

1. Pide al usuario que especifique una cadena de búsqueda.
1. Consulta posibles coincidencias en una base de datos.
   - Si hay una coincidencia, se selecciona y continúa.
   - Si hay varias coincidencias, pide al usuario que elija una.
   - Si no hay ninguna coincidencia, el diálogo finaliza.
1. Pide al usuario que especifique la máquina donde se instalará la aplicación.
1. Escribe la información en una base de datos y envía un mensaje de confirmación.

En el archivo **Dialogs/InstallAppDialog.cs**:

1. Actualice las instrucciones `using`: [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=4-11)]

1. Defina una constante para la clave que usaremos para realizar el seguimiento de la información recopilada.
    [!code-csharp[Key ID](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=17-18)]

1. Agregue un constructor e inicialice el conjunto de diálogos del componente.
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=20-34)]

1. Podemos reemplazar **StartAsync** por el primer paso de nuestra cascada.
    - Como tenemos que administrar el estado nosotros mismos, haremos un seguimiento del estado del objeto de instalación de la aplicación en el diálogo.
    - El aviso que pide datos de entrada al usuario se convierte en una opción en la llamada al aviso.
    [!code-csharp[GetSearchTermAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=35-50)]

1. Podemos reemplazar **appNameAsync** y **multipleAppsAsync** por el segundo paso de la cascada.
    - Ahora obtenemos el resultado del aviso en lugar de tener que mirar el último mensaje del usuario.
    - La consulta de base de datos y las instrucciones if se organizan igual que en **appNameAsync**. El código de cada bloque de la instrucción if se ha actualizado para que funcione con los diálogos de la versión v4.
        - Si hay una coincidencia, actualizaremos el estado del diálogo y continuaremos con el paso siguiente.
        - Si tenemos varias coincidencias, usaremos nuestro aviso de opciones para pedir al usuario que elija una de las opciones de la lista. Esto significa que podemos eliminar **multipleAppsAsync**.
        - Si no tenemos coincidencias, finalizaremos este diálogo y devolveremos NULL al diálogo raíz.
    [!code-csharp[ResolveAppNameAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=52-91)]

1. Después de resolver la consulta, **appNameAsync** también pide al usuario el nombre del equipo. Capturaremos esa parte de la lógica en el paso siguiente de la cascada.
    - De nuevo, en la versión v4 tenemos que administrar el estado nosotros mismos. Lo único complicado aquí es que podemos llegar a este paso desde dos ramas diferentes de la lógica del paso anterior.
    - Pediremos el usuario un nombre de equipo con el mismo aviso de texto de antes, solo que esta vez ofrecemos opciones diferentes.
    [!code-csharp[GetMachineNameAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=93-114)]

1. La lógica de **machineNameAsync** se encapsula en el paso final de nuestra cascada.
    - Recuperamos el nombre del equipo del resultado del aviso de texto y actualizamos el estado del diálogo.
    - Vamos a quitar la llamada para actualizar la base de datos, porque el código auxiliar está en un proyecto diferente.
    - Después, enviamos el mensaje de confirmación al usuario y finalizamos el diálogo.
    [!code-csharp[SubmitRequestAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=116-135)]

1. Para simular la llamada de la base de datos, hemos simulado **getAppsAsync** para consultar una lista estática, en lugar de la base de datos.
    [!code-csharp[GetAppsAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=137-200)]

### <a name="update-the-local-admin-dialog"></a>Actualización del diálogo de administrador local

En la versión v3, este diálogo saludaba al usuario, iniciaba el diálogo Formflow y, después, guardaba el resultado en una base de datos. Esto se traduce fácilmente en una cascada de dos pasos.

1. Actualice las instrucciones `using`. Tenga en cuenta que este diálogo incluye un diálogo Formflow v3. En la versión v4, podemos usar la biblioteca Formflow de la comunidad.
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=4-8)]

1. Podemos quitar la propiedad de instancia de `LocalAdmin`, porque el resultado estará disponible en el estado del diálogo.

1. Agregue un constructor e inicialice el conjunto de diálogos del componente. El diálogo Formflow se crea de la misma manera. Simplemente lo agregamos al conjunto de diálogos de nuestro componente en el constructor.
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=14-23)]

1. Podemos reemplazar **StartAsync** por el primer paso de nuestra cascada. Ya hemos creamos el diálogo Formflow en el constructor y las otras dos instrucciones se traducen en esto. Tenga en cuenta que `FormBuilder` asigna el nombre de tipo del modelo como el identificador del diálogo generado, que es `LocalAdminPrompt` para este modelo.
    [!code-csharp[BeginFormflowAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=25-35)]

1. Podemos reemplazar **ResumeAfterLocalAdminFormDialog** por el segundo paso de nuestra cascada. Tenemos que obtener el valor que devuelve el contexto del paso y no una propiedad de instancia.
    [!code-csharp[SaveResultAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=37-50)]

1. **BuildLocalAdminForm** permanece casi igual, salvo que el diálogo Formflow no actualiza la propiedad de instancia.
    [!code-csharp[BuildLocalAdminForm](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=52-76)]

### <a name="update-the-reset-password-dialog"></a>Actualización del diálogo de restablecimiento de contraseña

En la versión v3, este diálogo saludaba al usuario, lo autorizaba con un código de acceso, producía un error o iniciaba el diálogo Formflow y, después, restablecía la contraseña. Esto se traduce bien en una cascada.

1. Actualice las instrucciones `using`. Tenga en cuenta que este diálogo incluye un diálogo Formflow v3. En la versión v4, podemos usar la biblioteca Formflow de la comunidad.
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=4-9)]

1. Agregue un constructor e inicialice el conjunto de diálogos del componente. El diálogo Formflow se crea de la misma manera. Simplemente lo agregamos al conjunto de diálogos de nuestro componente en el constructor.
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=15-25)]

1. Podemos reemplazar **StartAsync** por el primer paso de nuestra cascada. Ya hemos creado el diálogo Formflow en el constructor. Salvo eso, mantenemos la misma lógica y traducimos las llamadas de la versión v3 a sus equivalentes de la versión v4.
    [!code-csharp[BeginFormflowAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=27-45)]

1. **sendPassCode** se deja principalmente como un ejercicio. El código original se marca como comentario y el método devuelve true. Además, podemos volver a quitar la dirección de correo electrónico porque no se usa en el bot original.
    [!code-csharp[SendPassCode](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=47-81)]

1. **BuildResetPasswordForm** no tiene ningún cambio.

1. Podemos reemplazar **ResumeAfterLocalAdminFormDialog** por el segundo paso de la cascada y obtendremos el valor devuelto del contexto del paso. Hemos quitado la dirección de correo electrónico porque el diálogo original no hacía nada con ella, y hemos proporcionado un resultado ficticio en vez de consultar la base de datos. Mantenemos la misma lógica y traducimos las llamadas de la versión v3 a sus equivalentes de la versión v4.
    [!code-csharp[ProcessRequestAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=90-113)]

### <a name="update-models-as-necessary"></a>Actualización de los modelos

Tenemos que actualizar las instrucciones `using` en algunos de los modelos que hacen referencia a la biblioteca Formflow.

1. En `LocalAdminPrompt`, cámbielas por esto: [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Models/LocalAdminPrompt.cs?range=4)]

1. En `ResetPasswordPrompt`, cámbielas por esto: [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Models/ResetPasswordPrompt.cs?range=4-5)]

## <a name="update-webconfig"></a>Actualizar el archivo web.config

Marque como comentarios las claves de configuración de **MicrosoftAppId** y **MicrosoftAppPassword**. Esto le permitirá depurar el bot localmente sin necesidad de proporcionar estos valores en el emulador.

## <a name="run-and-test-your-bot-in-the-emulator"></a>Ejecución y prueba del bot en el emulador

En este momento, podremos ejecutar el bot localmente en IIS y conectarlo al emulador.

1. Ejecute el bot en IIS.
1. Inicie el emulador y conéctese al punto de conexión del bot (por ejemplo: **http://localhost:3978/api/messages** ).
    - Si es la primera vez que se ejecuta el bot, haga clic en **Archivo > Nuevo bot** y siga las instrucciones en pantalla. En caso contrario, haga clic en **Archivo > Abrir bot** para abrir un bot existente.
    - Vuelva a comprobar el valor del puerto en la configuración. Por ejemplo, si abre el bot en el explorador en `http://localhost:3979/`, establezca el punto de conexión del bot en `http://localhost:3979/api/messages` en el emulador.
1. Los cuatro diálogos deberían funcionar y puede establecer puntos de interrupción en los pasos de la cascada para comprobar cuál es el contexto y el estado del diálogo en ese momento.

## <a name="additional-resources"></a>Recursos adicionales

Temas conceptuales de la versión v4:

- [Funcionamiento de los bots](../bot-builder-basics.md)
- [Administración de estados](../bot-builder-concept-state.md)
- [Biblioteca de cuadros de diálogo](../bot-builder-concept-dialog.md)

Temas de procedimientos de la versión v4:

- [Envío y recepción de mensajes de texto](../bot-builder-howto-send-messages.md)
- [Guardar usuario y datos de conversación](../bot-builder-howto-v4-state.md)
- [Implementación de flujo de conversación secuencial](../bot-builder-dialog-manage-conversation-flow.md)

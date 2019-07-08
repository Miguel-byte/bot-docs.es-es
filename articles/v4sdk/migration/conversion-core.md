---
title: Migración de un bot existente en un nuevo proyecto de .NET Core | Microsoft Docs
description: Tomamos un bot de .NET v3 existente y lo migramos al SDK de .NET v4, utilizando un nuevo proyecto de .NET Core.
keywords: bot migration, formflow, dialogs, v3 bot
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 06/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d09b52760bf6ef15dac0205d9aef2d3906fcacfd
ms.sourcegitcommit: 41c8caf0e0c849beeeb50cdccf6dbc1ba7cce442
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/24/2019
ms.locfileid: "67344740"
---
# <a name="migrate-a-net-v3-bot-to-a-net-core-v4-bot"></a>Migración de un bot de .NET v3 a un bot de .NET Core v4

En este artículo, convertiremos el bot de la versión v3 [ContosoHelpdeskChatBot](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V3) en un bot de la versión v4 _en un nuevo proyecto de .NET Core_.
Esta conversión se divide en estos pasos:

1. Cree el nuevo proyecto mediante una plantilla.
1. Instale paquetes NuGet adicionales según sea necesario.
1. Personalice su bot, actualice el archivo Startup.cs y actualice la clase de controlador.
1. Actualice la clase del bot.
1. Copie y actualice los cuadros de diálogo y los modelos.
1. Último paso de portabilidad.

El resultado de esta conversión es [ContosoHelpdeskChatBot de .NET Core v4](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore).
Para migrar a un bot de .NET Framework v4 _sin convertir el tipo de proyecto_, consulte [Migración de un bot de .NET v3 a un bot de Framework v4](conversion-framework.md).

Bot Framework SDK v4 se basa en las mismas API REST subyacentes que el SDK v3. Sin embargo, el SDK v4 es una refactorización de la versión anterior que ofrece a los desarrolladores más flexibilidad y control sobre sus bots. Algunos cambios importantes en el SDK son:

- El estado se administra mediante objetos de administración de estado y descriptores de acceso de propiedad.
- Ha cambiado el controlador de turnos y la forma de pasarle actividades.
- Ya no existen componentes puntuables. Puede buscar comandos "globales" en el controlador de turnos, antes de pasar el control a los diálogos.
- Una nueva biblioteca de diálogos muy diferente de la existente en la versión anterior. Deberá convertir los diálogos antiguos al nuevo sistema de diálogos con los diálogos de componente y en cascada, y la implementación de la comunidad de los diálogos Formflow para la versión v4.

Para más información sobre los cambios específicos, consulte las [diferencias entre las versiones v3 y v4 del SDK para .NET](migration-about.md).

## <a name="create-the-new-project-using-a-template"></a>Crear el nuevo proyecto mediante una plantilla

Cree un nuevo proyecto para el bot.

1. Si aún no lo ha hecho, instale la [plantilla para C#](https://aka.ms/bot-vsix) de Bot Framework SDK v4.
1. Abra Visual Studio y cree un nuevo proyecto de bot de eco a partir de la plantilla. Asigne el nombre `ContosoHelpdeskChatBot` al proyecto.

## <a name="install-additional-nuget-packages"></a>Instalar paquetes NuGet adicionales

La plantilla instala la mayoría de los paquetes que necesitará, incluidos los paquetes **Microsoft.Bot.Builder** y **Microsoft.Bot.Connector**.

1. Agregue **Bot.Builder.Community.Dialogs.Formflow**.

    Esta es una biblioteca de la comunidad para crear diálogos de la versión v4 a partir de archivos de definición de Formflow v3. Tiene **Microsoft.Bot.Builder.Dialogs** como una de sus dependencias, así que también se instala.

1. Agregue **log4net** para admitir el registro.

## <a name="personalize-your-bot"></a>Personalizar el bot

1. Cambie el nombre del archivo del bot de **Bots\EchoBot.cs** a **Bots\DialogBot.cs** y cambie el nombre de la clase `EchoBot` a `DialogBot`.
1. Cambie el nombre del controlador de **Controllers\BotController.cs** a **Controllers\MessagesController.cs** y cambie el nombre de la clase `BotController` a `MessagesController`.

## <a name="update-your-startupcs-file"></a>Actualizar el archivo Startup.cs

El estado y la manera en que el bot recibe las actividades entrantes ha cambiado. Tenemos que configurar partes de la infraestructura de la [administración del estado](../bot-builder-concept-state.md) nosotros mismos en v4. Por ejemplo, en la versión v4 se utiliza un adaptador de bot para controlar la autenticación y las actividades de reenvío al código del bot, y tenemos que declarar nuestras propiedades de estado por adelantado.

Vamos a crear una propiedad de estado para `DialogState`, que ahora es necesario para admitir los diálogos en la versión v4. Vamos a usar la inserción de dependencias para obtener la información necesaria para el controlador y el código del bot.

En **Startup.cs**:

1. Actualice las instrucciones `using`: [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Startup.cs?range=4-13)]

1. Quite este constructor:
    ```csharp
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }
    ```

1. Quite la propiedad `Configuration`.

1. Actualice el método `ConfigureServices` con el código siguiente: [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Startup.cs?range=19-41)]

En este momento, van a producirse errores de tiempo de compilación. Los corregiremos en los pasos siguientes. 

## <a name="messagescontroller-class"></a>Clase MessagesController
Esta clase controla una solicitud. La inyección de dependencias proporcionará la implementación del adaptador e IBot en tiempo de ejecución. Esta clase de plantilla no se ha modificado. 

Aquí es donde el bot empieza un turno en v4, es bastante diferente del controlador de mensajes de v3. Excepto el propio controlador de turnos del bot, la mayor parte puede considerarse como reutilizable.

El controlador de turnos del bot se define en **Bots\DialogBot.cs**.

### <a name="ignore-the-cancelscorable-and-globalmessagehandlersbotmodule-classes"></a>Ignore las clases CancelScorable y GlobalMessageHandlersBotModule

Puesto que no existen componentes puntuables en v4, simplemente se ignoran estas clases. Actualizaremos el controlador de turnos para que reaccione ante un mensaje `cancel`.

## <a name="update-your-bot-class"></a>Actualizar la clase del bot

En v4, el controlador de turnos o la lógica del bucle de mensajes se encuentran principalmente en un archivo bot. Estamos derivando `ActivityHandler`, que define a los controladores para los tipos comunes de actividades.

1. Actualice el archivo **Bots\DialogBots.cs**.

1. Actualice las instrucciones `using`: [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=4-8)]

1. Actualice `DialogBot` para incluir un parámetro genérico para el diálogo.
    [!code-csharp[Class definition](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=19)]

1. Agregue estos campos y un constructor para inicializarlos. De nuevo, ASP.NET usa la inyección de dependencias para obtener los valores de los parámetros.
    [!code-csharp[Fields and constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=21-28)]

1. Actualice la implementación de `OnMessageActivityAsync` para invocar el cuadro de diálogo principal. (Definiremos el método de extensión `Run` en breve). [!code-csharp[OnMessageActivityAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=38-47)]

1. Actualice `OnTurnAsync` para guardar el estado de la conversación al final del turno. En v4, tenemos que hacer esto explícitamente para escribir el estado en la capa de persistencia. El método `ActivityHandler.OnTurnAsync` llama a los métodos específicos del controlador de actividades, en función del tipo de actividad recibida, por lo que guardamos el estado después de la llamada al método base.
    [!code-csharp[OnTurnAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=30-36)]

### <a name="create-the-run-extension-method"></a>Creación del método de extensión de ejecución

Estamos creando un método de extensión para consolidar el código necesario para ejecutar un diálogo de componentes desde el bot.

Cree un archivo **DialogExtensions.cs** e implemente un método de extensión `Run`.
[!code-csharp[The extension](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/DialogExtensions.cs?range=4-26)]

## <a name="copy-over-and-convert-your-dialogs"></a>Copiar y convertir los cuadros de diálogo

Vamos a hacer muchos cambios en los cuadros de diálogo v3 originales para migrarlos al SDK v4. Por ahora, no se preocupe por los errores del compilador. Se solucionarán cuando hayamos terminado la conversión.
Para no modificar el código original más de lo necesario, seguirá habiendo algunas advertencias del compilador después de terminar la migración.

Todos nuestros diálogos derivarán de `ComponentDialog`, en lugar de implementar la interfaz `IDialog<object>` de la versión v3.

Este bot tiene cuatro diálogos que necesitamos convertir:

| | |
|---|---|
| [RootDialog](#update-the-root-dialog) | Presenta opciones e inicia los otros diálogos. |
| [InstallAppDialog](#update-the-install-app-dialog) | Controla las solicitudes para instalar una aplicación en un equipo. |
| [LocalAdminDialog](#update-the-local-admin-dialog) | Controla las solicitudes de derechos de administrador del equipo local. |
| [ResetPasswordDialog](#update-the-reset-password-dialog) | Controla las solicitudes para restablecer la contraseña. |

No copie la clase `CancelScorable`, ya que no existen componentes puntuables. Puede buscar comandos _globales_ en el controlador de turnos, antes de pasar el control a los diálogos.

Estos diálogos recopilan datos de entrada, pero no realizan ninguna de estas operaciones en el equipo.

1. Cree una carpeta **Dialogs** en el proyecto.
1. Copie estos archivos del directorio de cuadros de diálogo del proyecto v3 en el nuevo directorio de cuadros de diálogo.
    - **InstallAppDialog.cs**
    - **LocalAdminDialog.cs**
    - **ResetPasswordDialog.cs**
    - **RootDialog.cs**

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

1. Actualice las instrucciones `using`: [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=4-10)]

1. Necesitamos convertir las opciones `HelpdeskOptions` de una lista de cadenas en una lista de opciones. Se usará con un aviso de opciones, que aceptará como entrada válida el número de opción (en la lista), el valor de la opción o cualquiera de los sinónimos de la opción.
    [!code-csharp[HelpDeskOptions](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=28-33)]

1. Agregue un constructor. Este código hace lo siguiente:
   - Cuando se crea cada instancia de un diálogo, se le asigna un identificador. El identificador de diálogo es parte del conjunto de diálogos al que se va a agregar el diálogo. Recuerde que el bot se inicializó con un objeto de diálogo en la clase `MessageController`. Cada `ComponentDialog` tiene su propio conjunto de diálogos interno, con su propio conjunto de identificadores de diálogo.
   - Agrega los demás diálogos, incluido el aviso de opciones, como diálogos secundarios. En este caso, estamos usando el nombre de la clase en cada identificador de diálogo.
   - Define un diálogo en cascada de tres pasos. Los implementaremos en un momento.
     - En primer lugar, el diálogo pide al usuario que elija la tarea que va a realizar.
     - Después, inicia el diálogo secundario asociado a esa opción.
     - Y, por último, se reinicia a sí mismo.
   - Cada paso de la cascada es un delegado que implementaremos ahora, y tomaremos el código existente del diálogo original siempre que podamos.
   - Al iniciar un diálogo de componente, se iniciará su _initial dialog_. De forma predeterminada, este es el primer diálogo secundario que se agrega a un diálogo de componente. Vamos a establecer explícitamente la propiedad `InitialDialogId`, lo que significa que no es necesario que el diálogo principal de la cascada sea el primero que se agregue al conjunto. Por ejemplo, si prefiere agregar los avisos primero, esto le permitiría hacerlo sin causar problemas en tiempo de ejecución.
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=35-49)]

1. Podemos eliminar el método `StartAsync`. Cuando comienza un diálogo de componente, inicia automáticamente su diálogo _inicial_. En este caso, es el diálogo en cascada que se define en el constructor. También se inicia automáticamente en su primer paso.

1. Eliminaremos los métodos `MessageReceivedAsync` y `ShowOptions`, y los reemplazaremos por el primer paso de la cascada. Estos dos métodos saludarán al usuario y le pedirán que elija una de las opciones disponibles.
   - Aquí puede ver la lista de opciones y los mensajes de error y saludo que se proporcionan como opciones en la llamada a nuestro aviso de opciones.
   - No tenemos que especificar el método siguiente al que llamaremos en el diálogo, porque la cascada continuará con el paso siguiente cuando finalice el aviso de opciones.
   - El aviso de opciones se repetirá en bucle hasta que reciba una entrada válida o se cancele la pila completa del diálogo.
    [!code-csharp[PromptForOptionsAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=51-65)]

1. Podemos reemplazar `OnOptionSelected` por el segundo paso de nuestra cascada. Seguiremos iniciando un diálogo secundario en función de la entrada del usuario.
   - El aviso de opciones devuelve un valor `FoundChoice`. Esto se muestra en la propiedad `Result` del contexto del paso. La pila del diálogo trata todos los valores devueltos como objetos. Si el valor devuelto es uno de los diálogos, ya sabe de qué tipo de valor es el objeto. Consulte los [tipos de avisos](../bot-builder-concept-dialog.md#prompt-types) para obtener una lista de lo que devuelve cada tipo de aviso.
   - Como el aviso de opciones no iniciará una excepción, podemos quitar el bloque try-catch.
   - Necesitamos agregar una salida explícita para que este método siempre devuelva un valor adecuado. No se debería llegar nunca a este código pero, si se llega, permitirá que el diálogo "finalice correctamente".
    [!code-csharp[ShowChildDialogAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=67-102)]

1. Por último, reemplace el método `ResumeAfterOptionDialog` anterior por el último paso de nuestra cascada.
    - En lugar de finalizar el diálogo y devolver el número de vale, como hicimos en el diálogo original, vamos a reiniciar la cascada reemplazando la instancia original en la pila por una nueva instancia de sí mismo. Podemos hacerlo porque la aplicación original siempre pasa por alto el valor devuelto (el número de vale) y reinicia el diálogo raíz.
    [!code-csharp[ResumeAfterAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=104-138)]

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

1. Actualice las instrucciones `using`: [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=4-11)]

1. Defina una constante para la clave que usaremos para realizar el seguimiento de la información recopilada.
    [!code-csharp[Key ID](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=17-18)]

1. Agregue un constructor e inicialice el conjunto de diálogos del componente.
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=20-33)]

1. Podemos reemplazar `StartAsync` por el primer paso de nuestra cascada.
    - Como tenemos que administrar el estado nosotros mismos, haremos un seguimiento del estado del objeto de instalación de la aplicación en el diálogo.
    - El aviso que pide datos de entrada al usuario se convierte en una opción en la llamada al aviso.
    [!code-csharp[GetSearchTermAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=35-50)]

1. Podemos reemplazar `appNameAsync` y `multipleAppsAsync` por el segundo paso de nuestra cascada.
    - Ahora obtenemos el resultado del aviso en lugar de tener que mirar el último mensaje del usuario.
    - La consulta de base de datos y las instrucciones if se organizan igual que en `appNameAsync`. El código de cada bloque de la instrucción if se ha actualizado para que funcione con los diálogos de la versión v4.
        - Si hay una coincidencia, actualizaremos el estado del diálogo y continuaremos con el paso siguiente.
        - Si tenemos varias coincidencias, usaremos nuestro aviso de opciones para pedir al usuario que elija una de las opciones de la lista. Esto significa que podemos eliminar `multipleAppsAsync`.
        - Si no tenemos coincidencias, finalizaremos este diálogo y devolveremos NULL al diálogo raíz.
    [!code-csharp[ResolveAppNameAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=52-91)]

1. Después de resolver la consulta, `appNameAsync` también pide al usuario el nombre del equipo. Capturaremos esa parte de la lógica en el paso siguiente de la cascada.
    - De nuevo, en la versión v4 tenemos que administrar el estado nosotros mismos. Lo único complicado aquí es que podemos llegar a este paso desde dos ramas diferentes de la lógica del paso anterior.
    - Pediremos el usuario un nombre de equipo con el mismo aviso de texto de antes, solo que esta vez ofrecemos opciones diferentes.
    [!code-csharp[GetMachineNameAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=93-114)]

1. La lógica de `machineNameAsync` se encapsula en el paso final de nuestra cascada.
    - Recuperamos el nombre del equipo del resultado del aviso de texto y actualizamos el estado del diálogo.
    - Vamos a quitar la llamada para actualizar la base de datos, porque el código auxiliar está en un proyecto diferente.
    - Después, enviamos el mensaje de confirmación al usuario y finalizamos el diálogo.
    [!code-csharp[SubmitRequestAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=116-135)]

1. Para simular la llamada de la base de datos, hemos simulado `getAppsAsync` para consultar una lista estática, en lugar de la base de datos.
    [!code-csharp[GetAppsAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=137-200)]

### <a name="update-the-local-admin-dialog"></a>Actualización del diálogo de administrador local

En la versión v3, este diálogo saludaba al usuario, iniciaba el diálogo Formflow y, después, guardaba el resultado en una base de datos. Esto se traduce fácilmente en una cascada de dos pasos.

1. Actualice las instrucciones `using`. Tenga en cuenta que este diálogo incluye un diálogo Formflow v3. En la versión v4, podemos usar la biblioteca Formflow de la comunidad.
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=4-8)]

1. Podemos quitar la propiedad de instancia de `LocalAdmin`, porque el resultado estará disponible en el estado del diálogo.

1. Agregue un constructor e inicialice el conjunto de diálogos del componente. El diálogo Formflow se crea de la misma manera. Simplemente lo agregamos al conjunto de diálogos de nuestro componente en el constructor.
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=14-23)]

1. Podemos reemplazar `StartAsync` por el primer paso de nuestra cascada. Ya hemos creamos el diálogo Formflow en el constructor y las otras dos instrucciones se traducen en esto. Tenga en cuenta que `FormBuilder` asigna el nombre de tipo del modelo como el identificador del diálogo generado, que es `LocalAdminPrompt` para este modelo.
    [!code-csharp[BeginFormflowAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=25-35)]

1. Podemos reemplazar `ResumeAfterLocalAdminFormDialog` por el segundo paso de nuestra cascada. Tenemos que obtener el valor que devuelve el contexto del paso y no una propiedad de instancia.
    [!code-csharp[SaveResultAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=37-50)]

1. `BuildLocalAdminForm` permanece casi igual, salvo que Formflow no actualiza la propiedad de instancia.
    [!code-csharp[BuildLocalAdminForm](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=52-76)]

### <a name="update-the-reset-password-dialog"></a>Actualización del diálogo de restablecimiento de contraseña

En la versión v3, este diálogo saludaba al usuario, lo autorizaba con un código de acceso, producía un error o iniciaba el diálogo Formflow y, después, restablecía la contraseña. Esto se traduce bien en una cascada.

1. Actualice las instrucciones `using`. Tenga en cuenta que este diálogo incluye un diálogo Formflow v3. En la versión v4, podemos usar la biblioteca Formflow de la comunidad.
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=4-9)]

1. Agregue un constructor e inicialice el conjunto de diálogos del componente. El diálogo Formflow se crea de la misma manera. Simplemente lo agregamos al conjunto de diálogos de nuestro componente en el constructor.
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=15-25)]

1. Podemos reemplazar `StartAsync` por el primer paso de nuestra cascada. Ya hemos creado el diálogo Formflow en el constructor. Salvo eso, mantenemos la misma lógica y traducimos las llamadas de la versión v3 a sus equivalentes de la versión v4.
    [!code-csharp[BeginFormflowAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=27-45)]

1. `sendPassCode` se deja principalmente como un ejercicio. El código original se marca como comentario y el método devuelve true. Además, podemos volver a quitar la dirección de correo electrónico porque no se usa en el bot original.
    [!code-csharp[SendPassCode](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=47-81)]

1. `BuildResetPasswordForm` no incluye cambios.

1. Podemos reemplazar `ResumeAfterResetPasswordFormDialog` por el segundo paso de la cascada y obtendremos el valor devuelto del contexto del paso. Hemos quitado la dirección de correo electrónico porque el diálogo original no hacía nada con ella, y hemos proporcionado un resultado ficticio en vez de consultar la base de datos. Mantenemos la misma lógica y traducimos las llamadas de la versión v3 a sus equivalentes de la versión v4.
    [!code-csharp[ProcessRequestAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=90-113)]

## <a name="copy-over-and-update-models-as-necessary"></a>Copiar y actualizar los modelos según sea necesario
Puede usar los mismos modelos v3 con la biblioteca de flujo de formularios de la comunidad v4. 

1. Cree una carpeta **Models** en el proyecto.
1. Copie estos archivos del directorio de modelos del proyecto v3 en el nuevo directorio de modelos.
    - **InstallApp.cs**
    - **LocalAdmin.cs**
    - **LocalAdminPrompt.cs**
    - **ResetPassword.cs**
    - **ResetPasswordPrompt.cs**

### <a name="update-using-statements"></a>Actualizar instrucciones using

Es necesario actualizar las instrucciones `using` en las clases de modelo, como se muestra a continuación.

1. En **InstallApps.cs**, cámbielas de esta manera: [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Models/InstallApp.cs?range=4-5)]

1. En **LocalAdmin.cs**, cámbielas de esta manera: [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Models/LocalAdmin.cs?range=4-5)]

1. En **LocalAdminPrompt.cs**, cámbielas de esta manera: [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Models/LocalAdminPrompt.cs?range=4)]

1. En **ResetPassword.cs**, cámbielas de esta manera: [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Models/ResetPassword.cs?range=4-5)]
    Además, elimine las instrucciones `using` dentro del espacio de nombres.

1. En **ResetPasswordPrompt.cs**, cámbielas de esta manera: [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Models/ResetPasswordPrompt.cs?range=4-5)]

### <a name="additional-changes"></a>Cambios adicionales

En **ResetPassword.cs**, cambie el tipo de valor devuelto de `MobileNumber` como sigue: [!code-csharp[MobileNumber](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Models/ResetPassword.cs?range=17)]

## <a name="final-porting-steps"></a>Últimos pasos de portabilidad 
Para completar el proceso de portabilidad, siga estos pasos:

1. Cree una clase `AdapterWithErrorHandler` para definir un adaptador que incluya un controlador de errores que pueda capturar las excepciones en las aplicaciones o middleware. El adaptador procesa y dirige las actividades entrantes a través de la canalización de software intermedio del bot a la lógica del bot y, luego, otra vez de vuelta. Use el código siguiente para crear la clase: [!code-csharp[MobileNumber](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/AdapterWithErrorHandler.cs?range=4-46)]
1. Modifique la página **wwwroot\default.htm** como considere oportuno.

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
- [Depuración con el emulador](../../bot-service-debug-emulator.md)
- [Adición de telemetría al bot](../bot-builder-telemetry.md)

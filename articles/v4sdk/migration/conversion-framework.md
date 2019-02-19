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
ms.date: 02/11/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1904bb09d8bd387cc5cec0d85f82df24d1f6ec9d
ms.sourcegitcommit: 7f418bed4d0d8d398f824e951ac464c7c82b8c3e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2019
ms.locfileid: "56240181"
---
# <a name="migrate-a-net-sdk-v3-bot-to-v4"></a>Migración de un bot de la versión v3 a la versión v4 del SDK para .NET

En este artículo, convertiremos el bot de la versión v3 [ContosoHelpdeskChatBot](https://github.com/Microsoft/intelligent-apps/tree/master/ContosoHelpdeskChatBot/ContosoHelpdeskChatBot) en un bot de la versión v4 _sin convertir el tipo de proyecto_. Seguirá siendo un proyecto de .NET Framework.
Esta conversión se divide en estos pasos:

1. Actualización e instalación de los paquetes NuGet
1. Actualización del archivo Global.asax.cs
1. Actualización de la clase MessagesController
1. Conversión de los cuadros de diálogo

<!--TODO: Link to the converted bot...[ContosoHelpdeskChatBot](https://github.com/EricDahlvang/intelligent-apps/tree/v4netframework/ContosoHelpdeskChatBot).-->

Bot Framework SDK v4 se basa en las mismas API REST subyacentes que el SDK v3. Sin embargo, el SDK v4 es una refactorización de la versión anterior que ofrece a los desarrolladores más flexibilidad y control sobre sus bots. Algunos cambios importantes en el SDK son:

- El estado se administra mediante objetos de administración de estado y descriptores de acceso de propiedad.
- Ha cambiado el controlador de turnos y la forma de pasarle actividades.
- Ya no existen componentes puntuables. Puede buscar comandos "globales" en el controlador de turnos, antes de pasar el control a los diálogos.
- Una nueva biblioteca de diálogos muy diferente de la existente en la versión anterior. Deberá convertir los diálogos antiguos al nuevo sistema de diálogos con los diálogos de componente y en cascada, y la implementación de la comunidad de los diálogos Formflow para la versión v4.

Para más información sobre los cambios específicos, consulte las [diferencias entre las versiones v3 y v4 del SDK para .NET](migration-about.md).

## <a name="update-and-install-nuget-packages"></a>Actualización e instalación de los paquetes NuGet

1. Actualice **Microsoft.Bot.Builder.Azure** a la última versión estable.

    Esto actualizará también los paquetes **Microsoft.Bot.Builder** y **Microsoft.Bot.Connector**, porque son dependencias.

1. Elimine el paquete **Microsoft.Bot.Builder.History**. No forma parte del SDK v4.
1. Agregue **Autofac.WebApi2**.

    Vamos a usarlo como ayuda para la inserción de dependencias en ASP.NET.

1. Agregue **Bot.Builder.Community.Dialogs.Formflow**.

    Esta es una biblioteca de la comunidad para crear diálogos de la versión v4 a partir de archivos de definición de Formflow v3. Tiene **Microsoft.Bot.Builder.Dialogs** como una de sus dependencias, así que también se instala.

Si compila en este momento, recibirá errores del compilador. Puede pasarlos por alto. Cuando hayamos terminado con la conversión, tendremos un código que funciona.

<!--
## Add a BotDataBag class

This file will contain wrappers to add a v3-style **IBotDataBag** to make dialog conversion simpler.

```csharp
using System.Collections.Generic;

namespace ContosoHelpdeskChatBot
{
    public class BotDataBag : Dictionary<string, object>, IBotDataBag
    {
        public bool RemoveValue(string key)
        {
            return base.Remove(key);
        }

        public void SetValue<T>(string key, T value)
        {
            this[key] = value;
        }

        public bool TryGetValue<T>(string key, out T value)
        {
            if (!ContainsKey(key))
            {
                value = default(T);
                return false;
            }

            value = (T)this[key];

            return true;
        }
    }

    public interface IBotDataBag
    {
        int Count { get; }

        void Clear();

        bool ContainsKey(string key);

        bool RemoveValue(string key);

        void SetValue<T>(string key, T value);

        bool TryGetValue<T>(string key, out T value);
    }
}
```
-->

## <a name="update-your-globalasaxcs-file"></a>Actualización del archivo Global.asax.cs

Algunas de las técnicas de scaffolding han cambiado y, en la versión v4, tenemos que configurar partes de la [administración del estado](/articles/v4sdk/bot-builder-concept-state.md) nosotros mismos. Por ejemplo, en la versión v4 se utiliza un adaptador de bot para controlar la autenticación y las actividades de reenvío al código del bot, y tenemos que declarar nuestras propiedades de estado por adelantado.

Vamos a crear una propiedad de estado para `DialogState`, que ahora es necesario para admitir los diálogos en la versión v4. Vamos a usar la inserción de dependencias para obtener la información necesaria para el controlador y el código del bot.

En **Global.asax.cs**:

1. Actualice las instrucciones `using`.
    ```csharp
    using System.Configuration;
    using System.Reflection;
    using System.Web.Http;
    using Autofac;
    using Autofac.Integration.WebApi;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Connector.Authentication;
    ```
1. Elimine estas líneas del método **Application_Start**.
    ```csharp
    BotConfig.UpdateConversationContainer();
    this.RegisterBotModules();
    ```
    E inserte esta línea.
    ```csharp
    GlobalConfiguration.Configure(BotConfig.Register);
    ```
1. Quite el método **RegisterBotModules**, al que ya no se hace referencia.
1. Reemplace el método **BotConfig.UpdateConversationContainer** por este método **BotConfig.Register**, con el que registraremos los objetos necesarios para admitir la inserción de dependencias.
    > [!NOTE]
    > Este bot no usa los estados _user_ (usuario) ni _private conversation_ (conversación privada). Las líneas para incluirlos se marcan como comentarios aquí.
    ```csharp
    public static void Register(HttpConfiguration config)
    {
        ContainerBuilder builder = new ContainerBuilder();
        builder.RegisterApiControllers(Assembly.GetExecutingAssembly());

        SimpleCredentialProvider credentialProvider = new SimpleCredentialProvider(
            ConfigurationManager.AppSettings[MicrosoftAppCredentials.MicrosoftAppIdKey],
            ConfigurationManager.AppSettings[MicrosoftAppCredentials.MicrosoftAppPasswordKey]);

        builder.RegisterInstance(credentialProvider).As<ICredentialProvider>();

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        // Create Conversation State object.
        // The Conversation State object is where we persist anything at the conversation-scope.
        ConversationState conversationState = new ConversationState(dataStore);
        builder.RegisterInstance(conversationState).As<ConversationState>();

        //var userState = new UserState(dataStore);
        //var privateConversationState = new PrivateConversationState(dataStore);

        // Create the dialog state property acccessor.
        IStatePropertyAccessor<DialogState> dialogStateAccessor
            = conversationState.CreateProperty<DialogState>(nameof(DialogState));
        builder.RegisterInstance(dialogStateAccessor).As<IStatePropertyAccessor<DialogState>>();

        IContainer container = builder.Build();
        AutofacWebApiDependencyResolver resolver = new AutofacWebApiDependencyResolver(container);
        config.DependencyResolver = resolver;
    }
    ```

## <a name="update-your-messagescontroller-class"></a>Actualización de la clase MessagesController

Aquí es donde se produce el controlador de turnos en la versión v4, por lo que tiene que cambiar mucho. Excepto el propio controlador de turnos, la mayor parte puede considerarse como reutilizable. En su archivo **Controllers\MessagesController.cs**:

1. Actualice las instrucciones `using`.
    ```csharp
    using System;
    using System.Net;
    using System.Net.Http;
    using System.Threading;
    using System.Threading.Tasks;
    using System.Web.Http;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Dialogs;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Bot.Schema;
    ```
1. Quite el atributo `[BotAuthentication]` de la clase. En la versión v4, el adaptador del bot controlará la autenticación.
1. Agregue estos campos. **ConversationState** administrará el estado en el ámbito de la conversación, y **IStatePropertyAccessor\<DialogState>** es necesario para admitir los diálogos en la versión v4.
    ```csharp
    private readonly ConversationState _conversationState;
    private readonly ICredentialProvider _credentialProvider;
    private readonly IStatePropertyAccessor<DialogState> _dialogData;

    private readonly DialogSet _dialogs;
    ```
1. Agregue un constructor a:
    - Inicialice los campos de instancia.
    - Use la inserción de dependencias en ASP.NET para obtener los valores de los parámetros. (Hemos registrado instancias de estas clases en **Global.asax.cs** para admitir esto).
    - Cree e inicialice un conjunto de diálogos, a partir del cual podemos crear el contexto de diálogo. (Debemos hacerlo explícitamente en la versión v4).
    ```csharp
    public MessagesController(
        ConversationState conversationState,
        ICredentialProvider credentialProvider,
        IStatePropertyAccessor<DialogState> dialogData)
    {
        _conversationState = conversationState;
        _dialogData = dialogData;
        _credentialProvider = credentialProvider;

        _dialogs = new DialogSet(dialogData);
        _dialogs.Add(new RootDialog(nameof(RootDialog)));
    }
    ```
1. Reemplace el cuerpo del método **Post**. Aquí es donde crearemos nuestro adaptador y lo usaremos para llamar al bucle de mensajes (controlador de turnos). Usamos `SaveChangesAsync` al final de cada turno para guardar los cambios de estado que el bot haya hecho.

    ```csharp
    public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
    {

        var botFrameworkAdapter = new BotFrameworkAdapter(_credentialProvider);

        var invokeResponse = await botFrameworkAdapter.ProcessActivityAsync(
            Request.Headers.Authorization?.ToString(),
            activity,
            OnTurnAsync,
            default(CancellationToken));

        if (invokeResponse == null)
        {
            return Request.CreateResponse(HttpStatusCode.OK);
        }
        else
        {
            return Request.CreateResponse(invokeResponse.Status);
        }
    }
    ```
1. Agregue un método **OnTurnAsync** que contiene el código del [controlador de turnos](/articles/v4sdk/bot-builder-basics.md#the-activity-processing-stack) del bot.
    > [!NOTE]
    > Ya no existen componentes puntuables como tal en la versión v4. Buscamos un mensaje `cancel` del usuario en el controlador de turnos del bot antes de continuar con uno de los diálogos activos.
    ```csharp
    protected async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken)
    {
        // We're only handling message activities in this bot.
        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            // Create the dialog context for our dialog set.
            DialogContext dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

            // Globally interrupt the dialog stack if the user sent 'cancel'.
            if (turnContext.Activity.Text.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
            {
                Activity reply = turnContext.Activity.CreateReply($"Ok restarting conversation.");
                await turnContext.SendActivityAsync(reply);
                await dc.CancelAllDialogsAsync();
            }

            try
            {
                // Continue the active dialog, if any. If we just cancelled all dialog, the
                // dialog stack will be empty, and this will return DialogTurnResult.Empty.
                DialogTurnResult dialogResult = await dc.ContinueDialogAsync();
                switch (dialogResult.Status)
                {
                    case DialogTurnStatus.Empty:
                        // There was no active dialog in the dialog stack; start the root dialog.
                        await dc.BeginDialogAsync(nameof(RootDialog));
                        break;

                    case DialogTurnStatus.Complete:
                        // The last dialog on the stack completed and the stack is empty.
                        await dc.EndDialogAsync();
                        break;

                    case DialogTurnStatus.Waiting:
                    case DialogTurnStatus.Cancelled:
                        // The active dialog is waiting for a response from the user, or all
                        // dialogs were cancelled and the stack is empty. In either case, we
                        // don't need to do anything here.
                        break;
                }
            }
            catch (FormCanceledException)
            {
                // One of the dialogs threw an exception to clear the dialog stack.
                await turnContext.SendActivityAsync("Cancelled.");
                await dc.CancelAllDialogsAsync();
                await dc.BeginDialogAsync(nameof(RootDialog));
            }
        }
    }
    ```
1. Como solo estamos controlando actividades de _mensaje_, podemos eliminar el método **HandleSystemMessage**.

### <a name="delete-the-cancelscorable-and-globalmessagehandlersbotmodule-classes"></a>Eliminación de las clases CancelScorable y GlobalMessageHandlersBotModule

Como no existen componentes puntuables en la versión v4 y hemos actualizado el controlador de turnos para que reaccione a un mensaje `cancel`, podemos eliminar las clases **CancelScorable** (en **Dialogs\CancelScorable.cs**) y  **GlobalMessageHandlersBotModule**.

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

- Use la propiedad `DialogContext.Context` para obtener el contexto del turno actual.
- Los pasos de la cascada tienen un parámetro `WaterfallStepContext`, que deriva de `DialogContext`.
- Todas las clases específicas de los diálogos y los avisos derivan de la clase abstracta `Dialog`.
- Se asigna un identificador cuando se crea un diálogo de componente. En un conjunto de diálogos, se debe establecer un identificador único para cada diálogo del conjunto.

### <a name="update-the-root-dialog"></a>Actualización del diálogo raíz

En este bot, de diálogo raíz pide al usuario que elija una opción de un conjunto de opciones y, a continuación, inicia un diálogo secundario en función de la opción elegida. Esto se repite en bucle durante la vigencia de la conversación.

- Podemos configurar el flujo principal como un diálogo en cascada, que es un concepto nuevo del SDK v4. Se ejecutará siguiendo unos pasos con un orden fijo. Para más información, consulte [Implementación de flujo de conversación secuencial](/articles/v4sdk/bot-builder-dialog-manage-conversation-flow).
- Ahora, los avisos se controlan mediante clases de aviso, que son diálogos secundarios cortos que piden datos de entrada, realizan algún procesamiento mínimo y la validación, y devuelven un valor. Para más información, consulte [Recopilación de datos de entrada del usuario mediante un aviso de diálogo](/articles/v4sdk/bot-builder-prompts.md).

En el archivo **Dialogs/RootDialog.cs**:

1. Actualice las instrucciones `using`.
    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Dialogs.Choices;
    ```
1. Necesitamos convertir las opciones `HelpdeskOptions` de una lista de cadenas en una lista de opciones. Se usará con un aviso de opciones, que aceptará como entrada válida el número de opción (en la lista), el valor de la opción o cualquiera de los sinónimos de la opción.
    ```csharp
    private static List<Choice> HelpdeskOptions = new List<Choice>()
    {
        new Choice(InstallAppOption) { Synonyms = new List<string>(){ "install" } },
        new Choice(ResetPasswordOption) { Synonyms = new List<string>(){ "password" } },
        new Choice(LocalAdminOption)  { Synonyms = new List<string>(){ "admin" } }
    };
    ```
1. Agregue un constructor. Este código hace lo siguiente:
   - Cuando se crea cada instancia de un diálogo, se le asigna un identificador. El identificador de diálogo es parte del conjunto de diálogos al que se va a agregar el diálogo. Recuerde que el bot tiene un conjunto de diálogos que se inicializó en la clase **MessageController**. Cada `ComponentDialog` tiene su propio conjunto de diálogos interno, con su propio conjunto de identificadores de diálogo.
   - Agrega los demás diálogos, incluido el aviso de opciones, como diálogos secundarios. En este caso, estamos usando el nombre de la clase en cada identificador de diálogo.
   - Define un diálogo en cascada de tres pasos. Los implementaremos en un momento.
     - En primer lugar, el diálogo pide al usuario que elija la tarea que va a realizar.
     - Después, inicia el diálogo secundario asociado a esa opción.
     - Y, por último, se reinicia a sí mismo.
   - Cada paso de la cascada es un delegado que implementaremos ahora, y tomaremos el código existente del diálogo original siempre que podamos.
   - Al iniciar un diálogo de componente, se iniciará su _initial dialog_. De forma predeterminada, este es el primer diálogo secundario que se agrega a un diálogo de componente. Para asignar otro diálogo secundario como diálogo inicial, se establece manualmente la propiedad `InitialDialogId` del componente.
    ```csharp
    public RootDialog(string id)
        : base(id)
    {
        AddDialog(new WaterfallDialog("choiceswaterfall", new WaterfallStep[]
            {
                PromptForOptionsAsync,
                ShowChildDialogAsync,
                ResumeAfterAsync,
            }));
        AddDialog(new InstallAppDialog(nameof(InstallAppDialog)));
        AddDialog(new LocalAdminDialog(nameof(LocalAdminDialog)));
        AddDialog(new ResetPasswordDialog(nameof(ResetPasswordDialog)));
        AddDialog(new ChoicePrompt("options"));
    }
    ```
1. Podemos eliminar el método **StartAsync**. Cuando comienza un diálogo de componente, inicia automáticamente su diálogo _inicial_. En este caso, es el diálogo en cascada que se define en el constructor. También se inicia automáticamente en su primer paso.
1. Eliminaremos los métodos **MessageReceivedAsync** y **ShowOptions** y los reemplazaremos por el primer paso de la cascada. Estos dos métodos saludarán al usuario y le pedirán que elija una de las opciones disponibles.
   - Aquí puede ver la lista de opciones y los mensajes de error y saludo que se proporcionan como opciones en la llamada a nuestro aviso de opciones.
   - No tenemos que especificar el método siguiente al que llamaremos en el diálogo, porque la cascada continuará con el paso siguiente cuando finalice el aviso de opciones.
   - El aviso de opciones se repetirá en bucle hasta que reciba una entrada válida o se cancele la pila completa del diálogo.
    ```csharp
    private async Task<DialogTurnResult> PromptForOptionsAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Prompt the user for a response using our choice prompt.
        return await stepContext.PromptAsync(
            "options",
            new PromptOptions()
            {
                Choices = HelpdeskOptions,
                Prompt = MessageFactory.Text(GreetMessage),
                RetryPrompt = MessageFactory.Text(ErrorMessage)
            },
            cancellationToken);
    }
    ```
1. Podemos reemplazar **OnOptionSelected** por el segundo paso de nuestra cascada. Seguiremos iniciando un diálogo secundario en función de la entrada del usuario.
   - El aviso de opciones devuelve un valor `FoundChoice`. Esto se muestra en la propiedad `Result` del contexto del paso. La pila del diálogo trata todos los valores devueltos como objetos. Si el valor devuelto es uno de los diálogos, ya sabe de qué tipo de valor es el objeto. Consulte los [tipos de avisos](/articles/v4sdk/bot-builder-concept-dialog.md#prompt-types) para obtener una lista de lo que devuelve cada tipo de aviso.
   - Como el aviso de opciones no iniciará una excepción, puede quitar el bloque try-catch.
   - Necesitamos agregar una salida explícita para que este método siempre devuelva un valor adecuado. No se debería llegar nunca a este código pero, si se llega, permitirá que el diálogo "finalice correctamente".
    ```csharp
    private async Task<DialogTurnResult> ShowChildDialogAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // string optionSelected = await userReply;
        string optionSelected = (stepContext.Result as FoundChoice).Value;

        switch (optionSelected)
        {
            case InstallAppOption:
                //context.Call(new InstallAppDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(InstallAppDialog),
                    cancellationToken);
            case ResetPasswordOption:
                //context.Call(new ResetPasswordDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(ResetPasswordDialog),
                    cancellationToken);
            case LocalAdminOption:
                //context.Call(new LocalAdminDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(LocalAdminDialog),
                    cancellationToken);
        }

        // We shouldn't get here, but fail gracefully if we do.
        await stepContext.Context.SendActivityAsync(
            "I don't recognize that option.",
            cancellationToken: cancellationToken);
        // Continue through to the next step without starting a child dialog.
        return await stepContext.NextAsync(cancellationToken: cancellationToken);
    }
    ```
1. Por último, reemplace el método **ResumeAfterOptionDialog** anterior por el último paso de nuestra cascada.
    - En lugar de finalizar el diálogo y devolver el número de vale, como hicimos en el diálogo original, vamos a reiniciar la cascada reemplazando la instancia original en la pila por una nueva instancia de sí mismo. Podemos hacerlo porque la aplicación original siempre pasa por alto el valor devuelto (el número de vale) y reinicia el diálogo raíz.
    ```csharp
    private async Task<DialogTurnResult> ResumeAfterAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        try
        {
            //var message = await userReply;
            var message = stepContext.Context.Activity;

            int ticketNumber = new Random().Next(0, 20000);
            //await context.PostAsync($"Thank you for using the Helpdesk Bot. Your ticket number is {ticketNumber}.");
            await stepContext.Context.SendActivityAsync(
                $"Thank you for using the Helpdesk Bot. Your ticket number is {ticketNumber}.",
                cancellationToken: cancellationToken);

            //context.Done(ticketNumber);
        }
        catch (Exception ex)
        {
            // await context.PostAsync($"Failed with message: {ex.Message}");
            await stepContext.Context.SendActivityAsync(
                $"Failed with message: {ex.Message}",
                cancellationToken: cancellationToken);

            // In general resume from task after calling a child dialog is a good place to handle exceptions
            // try catch will capture exceptions from the bot framework awaitable object which is essentially "userReply"
            logger.Error(ex);
        }

        // Replace on the stack the current instance of the waterfall with a new instance,
        // and start from the top.
        return await stepContext.ReplaceDialogAsync(
            "choiceswaterfall",
            cancellationToken: cancellationToken);
    }
    ```

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

1. Actualice las instrucciones `using`.
    ```csharp
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading;
    using System.Threading.Tasks;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Dialogs.Choices;
    ```
1. Defina constantes para los identificadores que vamos a usar para los avisos y los diálogos. Esto hace que el código del diálogo sea más fácil de mantener, porque la cadena que se va a usar se define en un solo lugar.
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainId = "mainDialog";
    private const string TextId = "textPrompt";
    private const string ChoiceId = "choicePrompt";
    ```
1. Defina constantes para las claves que vamos a usar para realizar el seguimiento del estado del diálogo.
    ```csharp
    // Set up keys for managing collected information.
    private const string InstallInfo = "installInfo";
    ```
1. Agregue un constructor e inicialice el conjunto de diálogos del componente. Esta vez, vamos a establecer explícitamente la propiedad `InitialDialogId`, lo que significa que no es necesario que el diálogo principal de la cascada sea el primero que se agregue al conjunto. Por ejemplo, si prefiere agregar los avisos primero, esto le permitiría hacerlo sin causar problemas en tiempo de ejecución.
    ```csharp
    public InstallAppDialog(string id)
        : base(id)
    {
        // Initialize our dialogs and prompts.
        InitialDialogId = MainId;
        AddDialog(new WaterfallDialog(MainId, new WaterfallStep[] {
            GetSearchTermAsync,
            ResolveAppNameAsync,
            GetMachineNameAsync,
            SubmitRequestAsync,
        }));
        AddDialog(new TextPrompt(TextId));
        AddDialog(new ChoicePrompt(ChoiceId));
    }
    ```
1. Podemos reemplazar **StartAsync** por el primer paso de nuestra cascada.
    - Como tenemos que administrar el estado nosotros mismos, haremos un seguimiento del estado del objeto de instalación de la aplicación en el diálogo.
    - El aviso que pide datos de entrada al usuario se convierte en una opción en la llamada al aviso.
    ```csharp
    private async Task<DialogTurnResult> GetSearchTermAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Create an object in dialog state in which to track our collected information.
        stepContext.Values[InstallInfo] = new InstallApp();

        // Ask for the search term.
        return await stepContext.PromptAsync(
            TextId,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("Ok let's get started. What is the name of the application? "),
            },
            cancellationToken);
    }
    ```
1. Podemos reemplazar **appNameAsync** y **multipleAppsAsync** por el segundo paso de la cascada.
    - Ahora obtenemos el resultado del aviso en lugar de tener que mirar el último mensaje del usuario.
    - La consulta de base de datos y las instrucciones if se organizan igual que en **appNameAsync**. El código de cada bloque de la instrucción if se ha actualizado para que funcione con los diálogos de la versión v4.
        - Si hay una coincidencia, actualizaremos el estado del diálogo y continuaremos con el paso siguiente.
        - Si tenemos varias coincidencias, usaremos nuestro aviso de opciones para pedir al usuario que elija una de las opciones de la lista. Esto significa que podemos eliminar **multipleAppsAsync**.
        - Si no tenemos coincidencias, finalizaremos este diálogo y devolveremos NULL al diálogo raíz.
    ```csharp
    private async Task<DialogTurnResult> ResolveAppNameAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the text prompt.
        var appname = stepContext.Result as string;

        // Query the database for matches.
        var names = await this.getAppsAsync(appname);

        if (names.Count == 1)
        {
            // Get our tracking information from dialog state and add the app name.
            var install = stepContext.Values[InstallInfo] as InstallApp;
            install.AppName = names.First();

            return await stepContext.NextAsync();
        }
        else if (names.Count > 1)
        {
            // Ask the user to choose from the list of matches.
            return await stepContext.PromptAsync(
                ChoiceId,
                new PromptOptions
                {
                    Prompt = MessageFactory.Text("I found the following applications. Please choose one:"),
                    Choices = ChoiceFactory.ToChoices(names),
                },
                cancellationToken);
        }
        else
        {
            // If no matches, exit this dialog.
            await stepContext.Context.SendActivityAsync(
                $"Sorry, I did not find any application with the name '{appname}'.",
                cancellationToken: cancellationToken);

            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
    }
    ```
1. Después de resolver la consulta, **appNameAsync** también pide al usuario el nombre del equipo. Capturaremos esa parte de la lógica en el paso siguiente de la cascada.
    - De nuevo, en la versión v4 tenemos que administrar el estado nosotros mismos. Lo único complicado aquí es que podemos llegar a este paso desde dos ramas diferentes de la lógica del paso anterior.
    - Pediremos el usuario un nombre de equipo con el mismo aviso de texto de antes, solo que esta vez ofrecemos opciones diferentes.
    ```csharp
    private async Task<DialogTurnResult> GetMachineNameAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the tracking info. If we don't already have an app name,
        // Then we used the choice prompt to get it in the previous step.
        var install = stepContext.Values[InstallInfo] as InstallApp;
        if (install.AppName is null)
        {
            install.AppName = (stepContext.Result as FoundChoice).Value;
        }

        // We now need the machine name, so prompt for it.
        return await stepContext.PromptAsync(
            TextId,
            new PromptOptions
            {
                Prompt = MessageFactory.Text(
                    $"Found {install.AppName}. What is the name of the machine to install application?"),
            },
            cancellationToken);
    }
    ```
1. La lógica de **machineNameAsync** se encapsula en el paso final de nuestra cascada.
    - Recuperamos el nombre del equipo del resultado del aviso de texto y actualizamos el estado del diálogo.
    - Vamos a quitar la llamada para actualizar la base de datos, porque el código auxiliar está en un proyecto diferente.
    - Después, enviamos el mensaje de confirmación al usuario y finalizamos el diálogo.
    ```csharp
    private async Task<DialogTurnResult> SubmitRequestAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            // Get the tracking info and add the machine name.
            var install = stepContext.Values[InstallInfo] as InstallApp;
            install.MachineName = stepContext.Context.Activity.Text;

            //TODO: Save to this information to the database.
        }

        await stepContext.Context.SendActivityAsync(
            $"Great, your request to install {install.AppName} on {install.MachineName} has been scheduled.",
            cancellationToken: cancellationToken);

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```
1. Para simular la base de datos, he actualizado **getAppsAsync** para consultar una lista estática, en lugar de la base de datos.
    ```csharp
    private async Task<List<string>> getAppsAsync(string Name)
    {
        List<string> names = new List<string>();

        // Simulate querying the database for applications that match.
        return (from app in AppMsis
            where app.ToLower().Contains(Name.ToLower())
            select app).ToList();
    }

    // Example list of app names in the database.
    private static readonly List<string> AppMsis = new List<string>
    {
        "µTorrent 3.5.0.44178",
        "7-Zip 17.1",
        "Ad-Aware 9.0",
        "Adobe AIR 2.5.1.17730",
        "Adobe Flash Player (IE) 28.0.0.105",
        "Adobe Flash Player (Non-IE) 27.0.0.130",
        "Adobe Reader 11.0.14",
        "Adobe Shockwave Player 12.3.1.201",
        "Advanced SystemCare Personal 11.0.3",
        "Auslogics Disk Defrag 3.6",
        "avast! 4 Home Edition 4.8.1351",
        "AVG Anti-Virus Free Edition 9.0.0.698",
        "Bonjour 3.1.0.1",
        "CCleaner 5.24.5839",
        "Chmod Calculator 20132.4",
        "CyberLink PowerDVD 17.0.2101.62",
        "DAEMON Tools Lite 4.46.1.328",
        "FileZilla Client 3.5",
        "Firefox 57.0",
        "Foxit Reader 4.1.1.805",
        "Google Chrome 66.143.49260",
        "Google Earth 7.3.0.3832",
        "Google Toolbar (IE) 7.5.8231.2252",
        "GSpot 2701.0",
        "Internet Explorer 903235.0",
        "iTunes 12.7.0.166",
        "Java Runtime Environment 6 Update 17",
        "K-Lite Codec Pack 12.1",
        "Malwarebytes Anti-Malware 2.2.1.1043",
        "Media Player Classic 6.4.9.0",
        "Microsoft Silverlight 5.1.50907",
        "Mozilla Thunderbird 57.0",
        "Nero Burning ROM 19.1.1005",
        "OpenOffice.org 3.1.1 Build 9420",
        "Opera 12.18.1873",
        "Paint.NET 4.0.19",
        "Picasa 3.9.141.259",
        "QuickTime 7.79.80.95",
        "RealPlayer SP 12.0.0.319",
        "Revo Uninstaller 1.95",
        "Skype 7.40.151",
        "Spybot - Search & Destroy 1.6.2.46",
        "SpywareBlaster 4.6",
        "TuneUp Utilities 2009 14.0.1000.353",
        "Unlocker 1.9.2",
        "VLC media player 1.1.6",
        "Winamp 5.56 Build 2512",
        "Windows Live Messenger 2009 16.4.3528.331",
        "WinPatrol 2010 31.0.2014",
        "WinRAR 5.0",
    };
    ```

### <a name="update-the-local-admin-dialog"></a>Actualización del diálogo de administrador local

En la versión v3, este diálogo saludaba al usuario, iniciaba el diálogo Formflow y, después, guardaba el resultado en una base de datos. Esto se traduce fácilmente en una cascada de dos pasos.

1. Actualice las instrucciones `using`. Tenga en cuenta que este diálogo incluye un diálogo Formflow v3. En la versión v4, podemos usar la biblioteca Formflow de la comunidad.
    ```csharp
    using System.Threading;
    using System.Threading.Tasks;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder.Dialogs;
    ```
1. Podemos quitar la propiedad de instancia de `LocalAdmin`, porque el resultado estará disponible en el estado del diálogo.
1. Defina constantes para los identificadores que vamos a usar para los diálogos. En la biblioteca de la comunidad, el identificador de diálogo construido siempre es el nombre de la clase que produce el diálogo.
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainDialog = "mainDialog";
    private static string AdminDialog { get; } = nameof(LocalAdminPrompt);
    ```
1. Agregue un constructor e inicialice el conjunto de diálogos del componente. El diálogo Formflow se crea de la misma manera. Simplemente lo agregamos al conjunto de diálogos de nuestro componente en el constructor.
    ```csharp
    public LocalAdminDialog(string dialogId) : base(dialogId)
    {
        InitialDialogId = MainDialog;

        AddDialog(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            BeginFormflowAsync,
            SaveResultAsync,
        }));
        AddDialog(FormDialog.FromForm(BuildLocalAdminForm, FormOptions.PromptInStart));
    }
    ```
1. Podemos reemplazar **StartAsync** por el primer paso de nuestra cascada. Ya hemos creamos el diálogo Formflow en el constructor y las otras dos instrucciones se traducen en esto.
    ```csharp
    private async Task<DialogTurnResult> BeginFormflowAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        await stepContext.Context.SendActivityAsync("Great I will help you request local machine admin.");

        // Begin the Formflow dialog.
        return await stepContext.BeginDialogAsync(AdminDialog, cancellationToken: cancellationToken);
    }
    ```
1. Podemos reemplazar **ResumeAfterLocalAdminFormDialog** por el segundo paso de nuestra cascada. Tenemos que obtener el valor que devuelve el contexto del paso y no una propiedad de instancia.
    ```csharp
    private async Task<DialogTurnResult> SaveResultAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the Formflow dialog when it ends.
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            var admin = stepContext.Result as LocalAdminPrompt;

            //TODO: Save to this information to the database.
        }

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```
1. **BuildLocalAdminForm** permanece casi igual, salvo que el diálogo Formflow no actualiza la propiedad de instancia.
    ```csharp
    // Nearly the same as before.
    private IForm<LocalAdminPrompt> BuildLocalAdminForm()
    {
        //here's an example of how validation can be used in form builder
        return new FormBuilder<LocalAdminPrompt>()
            .Field(nameof(LocalAdminPrompt.MachineName),
            validate: async (state, value) =>
            {
                ValidateResult result = new ValidateResult { IsValid = true, Value = value };
                //add validation here

                //this.admin.MachineName = (string)value;
                return result;
            })
            .Field(nameof(LocalAdminPrompt.AdminDuration),
            validate: async (state, value) =>
            {
                ValidateResult result = new ValidateResult { IsValid = true, Value = value };
                //add validation here

                //this.admin.AdminDuration = Convert.ToInt32((long)value) as int?;
                return result;
            })
            .Build();
    }
    ```

### <a name="update-the-reset-password-dialog"></a>Actualización del diálogo de restablecimiento de contraseña

En la versión v3, este diálogo saludaba al usuario, lo autorizaba con un código de acceso, producía un error o iniciaba el diálogo Formflow y, después, restablecía la contraseña. Esto se traduce bien en una cascada.

1. Actualice las instrucciones `using`. Tenga en cuenta que este diálogo incluye un diálogo Formflow v3. En la versión v4, podemos usar la biblioteca Formflow de la comunidad.
    ```csharp
    using System;
    using System.Threading;
    using System.Threading.Tasks;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder.Dialogs;
    ```
1. Defina constantes para los identificadores que vamos a usar para los diálogos. En la biblioteca de la comunidad, el identificador de diálogo construido siempre es el nombre de la clase que produce el diálogo.
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainDialog = "mainDialog";
    private static string ResetDialog { get; } = nameof(ResetPasswordPrompt);
    ```
1. Agregue un constructor e inicialice el conjunto de diálogos del componente. El diálogo Formflow se crea de la misma manera. Simplemente lo agregamos al conjunto de diálogos de nuestro componente en el constructor.
    ```csharp
    public ResetPasswordDialog(string dialogId) : base(dialogId)
    {
        InitialDialogId = MainDialog;

        AddDialog(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            BeginFormflowAsync,
            ProcessRequestAsync,
        }));
        AddDialog(FormDialog.FromForm(BuildResetPasswordForm, FormOptions.PromptInStart));
    }
    ```
1. Podemos reemplazar **StartAsync** por el primer paso de nuestra cascada. Ya hemos creado el diálogo Formflow en el constructor. Salvo eso, mantenemos la misma lógica y traducimos las llamadas de la versión v3 a sus equivalentes de la versión v4.
    ```csharp
    private async Task<DialogTurnResult> BeginFormflowAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        await stepContext.Context.SendActivityAsync("Alright I will help you create a temp password.");

        // Check the passcode and fail out or begin the Formflow dialog.
        if (sendPassCode(stepContext))
        {
            return await stepContext.BeginDialogAsync(ResetDialog, cancellationToken: cancellationToken);
        }
        else
        {
            //here we can simply fail the current dialog because we have root dialog handling all exceptions
            throw new Exception("Failed to send SMS. Make sure email & phone number has been added to database.");
        }
    }
    ```
1. **sendPassCode** se deja principalmente como un ejercicio. El código original se marca como comentario y el método devuelve true. Además, podemos volver a quitar la dirección de correo electrónico porque no se usa en el bot original.
    ```csharp
    private bool sendPassCode(DialogContext context)
    {
        //bool result = false;

        //Recipient Id varies depending on channel
        //refer ChannelAccount class https://docs.botframework.com/en-us/csharp/builder/sdkreference/dd/def/class_microsoft_1_1_bot_1_1_connector_1_1_channel_account.html#a0b89cf01fdd73cbc00a524dce9e2ad1a
        //as well as Activity class https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html
        //int passcode = new Random().Next(1000, 9999);
        //Int64? smsNumber = 0;
        //string smsMessage = "Your Contoso Pass Code is ";
        //string countryDialPrefix = "+1";

        // TODO: save PassCode to database
        //using (var db = new ContosoHelpdeskContext())
        //{
        //    var reset = db.ResetPasswords.Where(r => r.EmailAddress == email).ToList();
        //    if (reset.Count >= 1)
        //    {
        //        reset.First().PassCode = passcode;
        //        smsNumber = reset.First().MobileNumber;
        //        result = true;
        //    }

        //    db.SaveChanges();
        //}

        // TODO: send passcode to user via SMS.
        //if (result)
        //{
        //    result = Helper.SendSms($"{countryDialPrefix}{smsNumber.ToString()}", $"{smsMessage} {passcode}");
        //}

        //return result;
        return true;
    }
    ```
1. **BuildResetPasswordForm** no tiene ningún cambio.
1. Podemos reemplazar **ResumeAfterLocalAdminFormDialog** por el segundo paso de la cascada y obtendremos el valor devuelto del contexto del paso. Hemos quitado la dirección de correo electrónico porque el diálogo original no hacía nada con ella, y hemos proporcionado un resultado ficticio en vez de consultar la base de datos. Mantenemos la misma lógica y traducimos las llamadas de la versión v3 a sus equivalentes de la versión v4.
    ```csharp
    private async Task<DialogTurnResult> ProcessRequestAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the Formflow dialog when it ends.
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            var prompt = stepContext.Result as ResetPasswordPrompt;
            int? passcode;

            // TODO: Retrieve the passcode from the database.
            passcode = 1111;

            if (prompt.PassCode == passcode)
            {
                string temppwd = "TempPwd" + new Random().Next(0, 5000);
                await stepContext.Context.SendActivityAsync(
                    $"Your temp password is {temppwd}",
                    cancellationToken: cancellationToken);
            }
        }

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```

### <a name="update-models-as-necessary"></a>Actualización de los modelos

Tenemos que actualizar las instrucciones `using` en algunos de los modelos que hacen referencia a la biblioteca Formflow.

1. En `LocalAdminPrompt`, cámbielas por esto:
    ```csharp
    using Bot.Builder.Community.Dialogs.FormFlow;
    ```
1. En `ResetPasswordPrompt`, cámbielas por esto:
    ```csharp
    using Bot.Builder.Community.Dialogs.FormFlow;
    using System;
    ```

## <a name="update-webconfig"></a>Actualizar el archivo web.config

Marque como comentarios las claves de configuración de **MicrosoftAppId** y **MicrosoftAppPassword**. Esto le permitirá depurar el bot localmente sin necesidad de proporcionar estos valores en el emulador.

## <a name="run-and-test-your-bot-in-the-emulator"></a>Ejecución y prueba del bot en el emulador

En este momento, podremos ejecutar el bot localmente en IIS y conectarlo al emulador.

1. Ejecute el bot en IIS.
1. Inicie el emulador y conéctese al punto de conexión del bot (por ejemplo: **http://localhost:3978/api/messages**).
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
- [Recopilación de datos de entrada del usuario mediante una solicitud de diálogo](../bot-builder-prompts.md)

<!-- TODO:
- The conceptual piece
- The migration to a .NET Core project
-->

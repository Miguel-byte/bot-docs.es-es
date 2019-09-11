---
title: Referencia rápida de la migración de la versión v3 a la v4 de .NET | Microsoft Docs
description: Esquema de las principales diferencias del SDK de Bot Framework para .NET en las versiones v3 y v4.
keywords: .net, bot migration, dialogs, v3 bot
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3ede676cd1a09566b42dc49cc3258aa6e42cbe05
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298951"
---
# <a name="net-migration-quick-reference"></a>Referencia rápida de la migración de .NET

El SDK BotBuilder para .NET v4 introduce varios cambios fundamentales que afectan a cómo se crean los bots. El propósito de esta guía es proporcionar una referencia rápida para resaltar las diferencias comunes al realizar tareas en los SDK de las versiones v3 y v4.

- La forma en que la información pasa entre un bot y los canales ha cambiado. En v3, hemos utilizado el objeto _Conversation_ y el método _SendAsync_ para procesar un mensaje y Autofac se ha utilizado ampliamente para cargar varias dependencias. En v4, se usan los objetos _Adapter_ y _TurnContext_ para procesar un mensaje y se puede usar la biblioteca de inserción de dependencias que prefiera.

- Además, los diálogos y las instancias de los bot se han desacoplado aún más. En v3, los diálogos se incorporaron al SDK principal y la pila se controló internamente y los diálogos secundarios se cargaron con los métodos _Call_ y _Forward_. En v4, ahora pasa los diálogos a instancias de bot como argumentos, proporcionando una mayor flexibilidad de composición y control del desarrollador de la pila de diálogos, y los diálogos secundarios se cargan con los métodos _BeginDialogAsync_ y _ReplaceDialogAsync_.

- Además, la versión v4 proporciona una clase `ActivityHandler`, que ayuda a automatizar el control de diferentes tipos de actividades, tales como _mensajes_, _actualización de conversación_ y actividades de _eventos_.

Estas mejoras dan como resultado cambios en la sintaxis para el desarrollo de bots en .NET, especialmente en la creación de objetos de bot, la definición de diálogos y la codificación de la lógica del control de eventos.

En el resto de este tema se comparan las construcciones de la versión v3 de SDK de Bot Framework para .NET con su equivalente de la versión v4.

## <a name="to-process-incoming-messages"></a>Para procesar los mensajes entrantes

### <a name="v3"></a>v3

```cs
[BotAuthentication]
public class MessagesController : ApiController
{
    public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
    {
        if (activity.GetActivityType() == ActivityTypes.Message)
        {
            await Conversation.SendAsync(activity, () => new Dialogs.RootDialog());
        }

        return Request.CreateResponse(HttpStatusCode.OK);
    }
}
```

### <a name="v4"></a>v4

```cs
[Route("api/messages")]
[ApiController]
public class BotController : ControllerBase
{
    private readonly IBotFrameworkHttpAdapter Adapter;
    private readonly IBot Bot;

    public BotController(IBotFrameworkHttpAdapter adapter, IBot bot)
    {
        Adapter = adapter;
        Bot = bot;
    }

    [HttpPost]
    public async Task PostAsync()
    {
        await Adapter.ProcessAsync(Request, Response, Bot);
    }
}
```

## <a name="to-send-a-message-to-a-user"></a>Para enviar un mensaje a un usuario

### <a name="v3"></a>v3

```cs
await context.PostAsync("Hello and welcome to the help desk bot.");
```

### <a name="v4"></a>v4

```cs
await turnContext.SendActivityAsync("Hello and welcome to the help desk bot.");
```

## <a name="to-load-a-root-dialog"></a>Para cargar un diálogo raíz

### <a name="v3"></a>v3

```cs
await Conversation.SendAsync(activity, () => new Dialogs.RootDialog());
```

### <a name="v4"></a>v4

```cs
// Create a DialogExtensions class with a Run method.
public static class DialogExtensions
{
    public static async Task Run(
        this Dialog dialog,
        ITurnContext turnContext,
        IStatePropertyAccessor<DialogState> accessor,
        CancellationToken cancellationToken)
    {
        var dialogSet = new DialogSet(accessor);
        dialogSet.Add(dialog);

        var dialogContext = await dialogSet.CreateContextAsync(turnContext, cancellationToken);

        var results = await dialogContext.ContinueDialogAsync(cancellationToken);
        if (results.Status == DialogTurnStatus.Empty)
        {
            await dialogContext.BeginDialogAsync(dialog.Id, null, cancellationToken);
        }
    }
}

// Call it from the ActivityHandler's OnMessageActivityAsync override
protected override async Task OnMessageActivityAsync(
    ITurnContext<IMessageActivity> turnContext,
    CancellationToken cancellationToken)
{
    // Run the Dialog with the new message Activity.
    await Dialog.Run(
        turnContext,
        ConversationState.CreateProperty<DialogState>("DialogState"),
        cancellationToken);
}
```

## <a name="to-start-a-child-dialog"></a>Para iniciar un diálogo secundario

### <a name="v3"></a>v3

```cs
context.Call(new NextDialog(), this.ResumeAfterNextDialog);
```

o

```cs
await context.Forward(new NextDialog(), this.ResumeAfterNextDialog, message);
```

### <a name="v4"></a>v4

```cs
dialogContext.BeginDialogAsync("<child-dialog-id>", options);
```

o

```cs
dialogContext.ReplaceDialogAsync("<child-dialog-id>", options);
```

## <a name="to-end-a-dialog"></a>Para finalizar un diálogo

### <a name="v3"></a>v3

```cs
context.Done(ReturnValue);
```

### <a name="v4"></a>v4

```cs
await context.EndDialogAsync(ReturnValue);
```

## <a name="to-prompt-a-user-for-input"></a>Para solicitar una entrada del usuario

### <a name="v3"></a>v3

```cs
PromptDialog.Choice(
    context,
    this.OnOptionSelected,
    Options, PromptMessage,
    ErrorMessage,
    3,
    PromptStyle.PerLine);
```

### <a name="v4"></a>v4

```cs
// In the dialog's constructor, register the prompt, and waterfall steps.
AddDialog(new TextPrompt(nameof(TextPrompt)));
AddDialog(new WaterfallDialog(nameof(WaterfallDialog), new WaterfallStep[]
{
    FirstStepAsync,
    SecondStepAsync,
}));

// The initial child Dialog to run.
InitialDialogId = nameof(WaterfallDialog);

// ...

// In the first step, invoke the prompt.
private async Task<DialogTurnResult> FirstStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    return await stepContext.PromptAsync(
        nameof(TextPrompt),
        new PromptOptions { Prompt = MessageFactory.Text("Please enter your destination.") },
        cancellationToken);
}

// In the second step, retrieve the Result from the stepContext.
private async Task<DialogTurnResult> SecondStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    var destination = (string)stepContext.Result;
}
```

## <a name="to-save-information-to-dialog-state"></a>Para guardar la información en el estado del diálogo

### <a name="v3"></a>v3

Todos los diálogos, y sus campos, se serializaban automáticamente en V3.

### <a name="v4"></a>v4

```cs
// StepContext values are auto-serialized in V4, and scoped to the dialog.
stepContext.values.destination = destination;
```

## <a name="to-write-changes-in-state-to-the-persistance-layer"></a>Para escribir los cambios de estado en la capa de persistencia

### <a name="v3"></a>v3

De forma predeterminada, los datos de estado se guardan automáticamente al final del turno.

### <a name="v4"></a>v4

```cs
// You now must explicitly save state changes before the end of the turn.
await this.conversationState.saveChanges(context, false);
await this.userState.saveChanges(context, false);
```

## <a name="to-create-and-register-state-storage"></a>Para crear y registrar el almacenamiento de estado

### <a name="v3"></a>v3

```cs
// Autofac was used internally by the sdk, and state was automatic.
Conversation.UpdateContainer(
builder =>
{
    builder.RegisterModule(new AzureModule(Assembly.GetExecutingAssembly()));
    var store = new InMemoryDataStore();
    builder.Register(c => store)
        .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
        .AsSelf()
        .SingleInstance();
});
```

### <a name="v4"></a>v4

```cs
// Create the storage we'll be using for User and Conversation state.
// In-memory storage is great for testing purposes.
services.AddSingleton<IStorage, MemoryStorage>();

// Create the user state (used in this bot's Dialog implementation).
services.AddSingleton<UserState>();

// Create the conversation state (used by the Dialog system itself).
services.AddSingleton<ConversationState>();

// The dialog that will be run by the bot.
services.AddSingleton<MainDialog>();

// Create the bot as a transient. In this case the ASP.NET controller is expecting an IBot.
services.AddTransient<IBot, DialogBot>();

// In the bot's ActivityHandler implementation, call SaveChangesAsync after the OnTurnAsync completes.
public class DialogBot : ActivityHandler
{
    protected readonly Dialog Dialog;
    protected readonly BotState ConversationState;
    protected readonly BotState UserState;

    public DialogBot(ConversationState conversationState, UserState userState, Dialog dialog)
    {
        ConversationState = conversationState;
        UserState = userState;
    }

    public override async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        await base.OnTurnAsync(turnContext, cancellationToken);

        // Save any state changes that might have ocurred during the turn.
        await ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
        await UserState.SaveChangesAsync(turnContext, false, cancellationToken);
    }

    protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
    {
        // Run the dialog, passing in the message activity for this turn.
        await Dialog.Run(turnContext, ConversationState.CreateProperty<DialogState>("DialogState"), cancellationToken);
    }
}
```

## <a name="to-catch-an-error-thrown-from-a-dialog"></a>Para detectar un error producido en un diálogo

### <a name="v3"></a>v3

```cs
// Create a custom IPostToBot implementation to catch exceptions.
public sealed class CustomPostUnhandledExceptionToUser : IPostToBot
{
    private readonly IPostToBot inner;
    private readonly IBotToUser botToUser;
    private readonly ResourceManager resources;
    private readonly System.Diagnostics.TraceListener trace;

    public CustomPostUnhandledExceptionToUser(IPostToBot inner, IBotToUser botToUser, ResourceManager resources, System.Diagnostics.TraceListener trace)
    {
        SetField.NotNull(out this.inner, nameof(inner), inner);
        SetField.NotNull(out this.botToUser, nameof(botToUser), botToUser);
        SetField.NotNull(out this.resources, nameof(resources), resources);
        SetField.NotNull(out this.trace, nameof(trace), trace);
    }

    async Task IPostToBot.PostAsync(IActivity activity, CancellationToken token)
    {
        try
        {
            await this.inner.PostAsync(activity, token);
        }
        catch (Exception ex)
        {
            try
            {
                // Log exception and send custom error message here.
                await this.botToUser.PostAsync("custom error message");
            }
            catch (Exception inner)
            {
                this.trace.WriteLine(inner);
            }

            throw;
        }
    }
}

// Register this using AutoFac, replacing the default PostUnhandledExceptionToUser.
builder
  .RegisterType<CustomPostUnhandledExceptionToUser>()
  .Keyed<IPostToBot>(typeof(PostUnhandledExceptionToUser));
```

### <a name="v4"></a>v4

```cs
// Provide an error handler in your implementation of the BotFrameworkHttpAdapter.
public class AdapterWithErrorHandler : BotFrameworkHttpAdapter
{
    public AdapterWithErrorHandler(
        ICredentialProvider credentialProvider,
        ILogger<BotFrameworkHttpAdapter> logger,
        ConversationState conversationState = null)
        : base(credentialProvider)
    {
        OnTurnError = async (turnContext, exception) =>
        {
            // Log any leaked exception from the application.
            logger.LogError($"Exception caught : {exception.Message}");

            // Send a catch-all apology to the user.
            await turnContext.SendActivityAsync("Sorry, it looks like something went wrong.");

            if (conversationState != null)
            {
                try
                {
                    // Delete the conversation state for the current conversation, to prevent the
                    // bot from getting stuck in a error-loop caused by being in a bad state.
                    // Conversation state is similar to "cookie-state" in a web page.
                    await conversationState.DeleteAsync(turnContext);
                }
                catch (Exception e)
                {
                    logger.LogError(
                        $"Exception caught on attempting to Delete ConversationState : {e.Message}");
                }
            }
        };
    }
}
```

## <a name="to-process-different-activity-types"></a>Para procesar diferentes tipos de actividad

### <a name="v3"></a>v3

```cs
// Within your MessageController, check the message type.
string messageType = activity.GetActivityType();
if (messageType == ActivityTypes.Message)
{
    await Conversation.SendAsync(activity, () => new Dialogs.RootDialog());
}
else if (messageType == ActivityTypes.DeleteUserData)
{
}
else if (messageType == ActivityTypes.ConversationUpdate)
{
}
else if (messageType == ActivityTypes.ContactRelationUpdate)
{
}
else if (messageType == ActivityTypes.Typing)
{
}
```

### <a name="v4"></a>v4

```cs
// In the bot's ActivityHandler implementation, override relevant methods.

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    // Handle message activities here.
}

protected override Task OnConversationUpdateActivityAsync(ITurnContext<IConversationUpdateActivity> turnContext, CancellationToken cancellationToken)
{
    // Handle conversation update activities in general here.
}

protected override Task OnEventActivityAsync(ITurnContext<IEventActivity> turnContext, CancellationToken cancellationToken)
{
    // Handle event activities in general here.
}
```

## <a name="to-log-all-activities"></a>Para registrar todas las actividades

### <a name="v3"></a>v3

Se usaba [IActivityLogger](https://docs.microsoft.com/dotnet/api/microsoft.bot.builder.history.iactivitylogger).

```csharp
builder.RegisterType<ActivityLoggerImplementation>().AsImplementedInterfaces().InstancePerDependency(); 

public class ActivityLoggerImplementation : IActivityLogger
{
    async Task IActivityLogger.LogAsync(IActivity activity)
    {
        // Store the activity.
    }
}
```

### <a name="v4"></a>v4

Use [ITranscriptLogger](https://docs.microsoft.com/dotnet/api/microsoft.bot.builder.itranscriptlogger).

```csharp
var transcriptMiddleware = new TranscriptLoggerMiddleware(new TranscriptLoggerImplementation(Configuration.GetSection("StorageConnectionString").Value));
adapter.Use(transcriptMiddleware);

public class TranscriptLoggerImplementation : ITranscriptLogger
{
    async Task ITranscriptLogger.LogActivityAsync(IActivity activity)
    {
        // Store the activity.
    }
}
```

## <a name="to-add-bot-state-storage"></a>Para agregar almacenamiento para el estado del bot

La interfaz para almacenar los _datos del usuario_, los _datos de la conversación_ y los _datos de la conversación privada_ ha cambiado.

### <a name="v3"></a>v3

El estado se conserva mediante la implementación de `IBotDataStore`, y se inserta en el sistema de estado del diálogo del SDK mediante Autofac.  Microsoft proporcionaba las clases `MemoryStorage`, `DocumentDbBotDataStore`, `TableBotDataStore` y `SqlBotDataStore` en [Microsoft.Bot.Builder.Azure](https://github.com/Microsoft/BotBuilder-Azure/).

Se usaba [IBotDataStore<BotData> ](https://docs.microsoft.com/dotnet/api/microsoft.bot.builder.dialogs.internals.ibotdatastore-1?view=botbuilder-dotnet-3.0) para conservar los datos.

```csharp
Task<bool> FlushAsync(IAddress key, CancellationToken cancellationToken);
Task<T> LoadAsync(IAddress key, BotStoreType botStoreType, CancellationToken cancellationToken);
Task SaveAsync(IAddress key, BotStoreType botStoreType, T data, CancellationToken cancellationToken);
```

```csharp
var dbPath = ConfigurationManager.AppSettings["DocDbPath"];
var dbKey = ConfigurationManager.AppSettings["DocDbKey"];
var docDbUri = new Uri(dbPath);
var storage = new DocumentDbBotDataStore(docDbUri, dbKey);
builder.Register(c => storage)
                .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
                .AsSelf()
                .SingleInstance();
```

### <a name="v4"></a>v4

La capa de almacenamiento utiliza la interfaz `IStorage`; especifique el objeto de capa de almacenamiento al crear cada objeto de administración del estado del bot, por ejemplo, `UserState`, `ConversationState`, o `PrivateConversationState`. El objeto de administración del estado proporciona las claves de la capa de almacenamiento subyacente y también actúa como administrador de propiedades. Por ejemplo, use `IPropertyManager.CreateProperty<T>(string name)` para crear un descriptor de acceso de propiedad de estado.  Estos descriptores de acceso de propiedad se usan para almacenar y recuperar los valores del almacenamiento subyacente del bot.

Use [IStorage](https://docs.microsoft.com/dotnet/api/microsoft.bot.builder.istorage?view=botbuilder-dotnet-stable) para conservar los datos.

```csharp
Task DeleteAsync(string[] keys, CancellationToken cancellationToken = default(CancellationToken));
Task<IDictionary<string, object>> ReadAsync(string[] keys, CancellationToken cancellationToken = default(CancellationToken));
Task WriteAsync(IDictionary<string, object> changes, CancellationToken cancellationToken = default(CancellationToken));
```

```csharp
var storageOptions = new CosmosDbStorageOptions()
{
    AuthKey = configuration["cosmosKey"],
    CollectionId = configuration["cosmosCollection"],
    CosmosDBEndpoint = new Uri(configuration["cosmosPath"]),
    DatabaseId = configuration["cosmosDatabase"]
};

IStorage dataStore = new CosmosDbStorage(storageOptions);
var conversationState = new ConversationState(dataStore);
services.AddSingleton(conversationState);

```

## <a name="to-use-form-flow"></a>Para usar FormFlow

### <a name="v3"></a>v3

Se incluyó [Microsoft.Bot.Builder.FormFlow](https://docs.microsoft.com/dotnet/api/microsoft.bot.builder.formflow?view=botbuilder-dotnet-3.0) en el SDK Bot Builder principal.

### <a name="v4"></a>v4

Ahora, [Bot.Builder.Community.Dialogs.FormFlow](https://www.nuget.org/packages/Bot.Builder.Community.Dialogs.FormFlow/) es una biblioteca de la comunidad de Bot Builder.  El código fuente está disponible en el [repositorio](https://github.com/BotBuilderCommunity/botbuilder-community-dotnet/tree/develop/libraries/Bot.Builder.Community.Dialogs.FormFlow) de la comunidad.

## <a name="to-use-luisdialog"></a>Para usar LuisDialog

### <a name="v3"></a>v3

Se incluyó [Microsoft.Bot.Builder.Dialogs.LuisDialog](https://docs.microsoft.com/dotnet/api/microsoft.bot.builder.dialogs.luisdialog-1?view=botbuilder-dotnet-3.0) en el SDK Bot Builder principal.

### <a name="v4"></a>v4

Ahora, [Bot.Builder.Community.Dialogs.Luis](https://www.nuget.org/packages/Bot.Builder.Community.Dialogs.Luis/) es una biblioteca de la comunidad de Bot Builder.  El código fuente está disponible en el [repositorio](https://github.com/BotBuilderCommunity/botbuilder-community-dotnet/tree/develop/libraries/Bot.Builder.Community.Dialogs.Luis) de la comunidad.

## <a name="to-use-qna-maker"></a>Para usar QnA Maker

### <a name="v3"></a>v3

```csharp
[Serializable]
[QnAMaker("QnAEndpointKey", "QnAKnowledgebaseId", <ScoreThreshold>, <TotalResults>, "QnAEndpointHostName")]
public class SimpleQnADialog : QnAMakerDialog
{
}
```

### <a name="v4"></a>v4

```csharp
public class QnABot : ActivityHandler
{
  private readonly IConfiguration _configuration;
  private readonly ILogger<QnABot> _logger;
  private readonly IHttpClientFactory _httpClientFactory;

  public QnABot(IConfiguration configuration, ILogger<QnABot> logger, IHttpClientFactory httpClientFactory)
  {
    _configuration = configuration;
    _logger = logger;
    _httpClientFactory = httpClientFactory;
  }

  protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
  {
    var httpClient = _httpClientFactory.CreateClient();

    var qnaMaker = new QnAMaker(new QnAMakerEndpoint
    {
      KnowledgeBaseId = _configuration["QnAKnowledgebaseId"],
      EndpointKey = _configuration["QnAEndpointKey"],
      Host = _configuration["QnAEndpointHostName"]
    },
    null,
    httpClient);

    _logger.LogInformation("Calling QnA Maker");

    // The actual call to the QnA Maker service.
    var response = await qnaMaker.GetAnswersAsync(turnContext);
    if (response != null && response.Length > 0)
    {
      await turnContext.SendActivityAsync(MessageFactory.Text(response[0].Answer), cancellationToken);
    }
    else
    {
      await turnContext.SendActivityAsync(MessageFactory.Text("No QnA Maker answers were found."), cancellationToken);
    }
  }
}
```

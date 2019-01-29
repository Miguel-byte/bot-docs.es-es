---
title: Obtención de notificaciones de los bots | Microsoft Docs
description: Descripción de cómo enviar mensajes de notificación
keywords: mensaje proactivo, mensaje de notificación, notificación de bot,
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7a9a2e4f30d1e9b293e51a921afce57d243376d7
ms.sourcegitcommit: c6ce4c42fc56ce1e12b45358d2c747fb77eb74e2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/22/2019
ms.locfileid: "54453969"
---
# <a name="get-notification-from-bots"></a>Obtención de notificaciones de los bots

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

Normalmente, cada mensaje que envía un bot al usuario se relaciona directamente con la anterior entrada del usuario.
En algunos casos, puede que un bot tenga que enviar al usuario un mensaje que no está relacionado directamente con el tema actual de la conversación o con el último mensaje que envió el usuario. Este tipo de mensajes se llaman _mensajes proactivos_.

## <a name="proactive-messages"></a>Mensajes proactivos

Los mensajes proactivos pueden ser útiles en diversos escenarios. Si un bot establece un temporizador o un recordatorio, deberá notificar al usuario cuando llegue la hora. O bien, si un bot recibe una notificación desde un evento externo, es posible que deba comunicar esa información al usuario inmediatamente. Por ejemplo, si el usuario ha solicitado anteriormente al bot que supervise el precio de un producto, el bot puede alertar al usuario si el precio del producto ha descendido un 20 %. O bien, si un bot necesita algo de tiempo para compilar una respuesta a la pregunta del usuario, puede informar al usuario del retraso y permitir que la conversación continúe mientras tanto. Cuando el bot termine de compilar la respuesta a la pregunta, compartirá esta información con el usuario.

Al implementar mensajes proactivos en el bot:

- No envíe varios mensajes proactivos dentro de un intervalo corto de tiempo. Algunos canales imponen restricciones sobre la frecuencia con que un bot puede enviar mensajes al usuario, y deshabilitarán el bot si se infringen tales restricciones.
- No envíe mensajes proactivos a usuarios que no hayan interactuado anteriormente con el bot o que no hayan solicitado contacto con el bot por otros medios como correo electrónico o SMS.

Un mensaje proactivo ad hoc es el tipo más simple de mensaje proactivo. El bot simplemente interpone el mensaje en la conversación cada vez que se desencadena, sin tener en cuenta si el usuario está implicado actualmente en otro tema de conversación con el bot, y no intentará cambiar la conversación de ninguna manera.

Para controlar las notificaciones más fácilmente, considere otras formas de integrar la notificación en el flujo de conversación, como establecer una marca en el estado de la conversación o agregar la notificación a una cola.

## <a name="prerequisites"></a>Requisitos previos

- Descripción de los [conceptos básicos de bot](bot-builder-basics.md) e información sobre la [administración del estado](bot-builder-concept-state.md).
- Una copia del **ejemplo de mensajes proactivos** en [C#](https://aka.ms/proactive-sample-cs) o [JS](https://aka.ms/proactive-sample-js). Este ejemplo se utiliza para explicar la mensajería proactiva de este artículo. 

## <a name="about-the-sample-code"></a>Acerca del código de ejemplo

El ejemplo de mensajes proactivos modela tareas de usuario que pueden tardar una cantidad de tiempo no determinada. El bot almacena información sobre la tarea, indica al usuario que el bot regresará cuando la tarea termine y deja que la conversación continúe. Cuando la tarea se completa, el bot envía el mensaje de confirmación de forma proactiva en la conversación original.

## <a name="define-job-data-and-state"></a>Definición del estado y los datos de trabajo

En este escenario, se va a realizar un seguimiento de los trabajos arbitrarios que diferentes usuarios pueden crear en distintas conversaciones. Necesitaremos almacenar información de cada trabajo, incluyendo una referencia de conversación y un identificador de trabajo. Necesitaremos:

- La referencia de la conversación para poder enviar el mensaje proactivo a la conversación correcta.
- Una forma de identificar los trabajos. En este ejemplo, se usa una marca de tiempo simple.
- Necesitaremos almacenar el estado del trabajo de forma independiente a la del estado de la conversación o del usuario.

Ampliaremos el _estado del bot_ para definir nuestro propio objeto de administración de estado para todo el bot. Bot Framework usa la _clave de almacenamiento_ y el contexto del turno para conservar y recuperar el estado. Consulte [Administración del estado](bot-builder-concept-state.md) para más información.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Tenemos que definir clases para el estado y los datos de trabajos. También es necesario registrar nuestro bot y configurar un descriptor de acceso de las propiedades de estado para el registro del trabajo.

### <a name="define-a-class-for-job-data"></a>Definición de una clase para los datos de trabajo

La clase `JobLog` realiza un seguimiento de los datos del trabajo, indexados por el número de trabajo (la marca de tiempo). La clase `JobLog` realiza el seguimiento de todos los trabajos pendientes.  Cada trabajo se identifica mediante una clave única. `JobData` describe el estado de un trabajo y se define como una clase interna de un diccionario.

```csharp
public class JobLog : Dictionary<long, JobLog.JobData>
{
    public class JobData
    {
        // Gets or sets the time-stamp for the job.
        public long TimeStamp { get; set; } = 0;

        // Gets or sets a value indicating whether indicates whether the job has completed.
        public bool Completed { get; set; } = false;

        // Gets or sets the conversation reference to which to send status updates.
        public ConversationReference Conversation { get; set; }
    }
}
```

### <a name="define-a-state-management-class"></a>Definición de una clase de administración del estado

La clase `JobState` administra el estado del trabajo independientemente del estado de la conversación o el usuario.

```csharp
using Microsoft.Bot.Builder;

/// A BotState for managing bot state for "bot jobs".
public class JobState : BotState
{
    // The key used to cache the state information in the turn context.
    private const string StorageKey = "ProactiveBot.JobState";

    // Initializes a new instance of the JobState class.
    public JobState(IStorage storage)
        : base(storage, StorageKey)
    {
    }

    // Gets the storage key for caching state information.
    protected override string GetStorageKey(ITurnContext turnContext) => StorageKey;
}
```

### <a name="register-the-bot-and-required-services"></a>Registro del bot y servicios necesarios

El archivo **Startup.cs** registra el bot y los servicios asociados.

El método `ConfigureServices` registra el bot y el servicio del punto de conexión, incluido el control de errores y la administración de estados. También registra el descriptor de acceso de estado del trabajo.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // The Memory Storage used here is for local bot debugging only. When the bot
    // is restarted, everything stored in memory will be gone.
    IStorage dataStore = new MemoryStorage();
    // ...

    // Create Job State object.
    // The Job State object is where we persist anything at the job-scope.
    // Note: It's independent of any user or conversation.
    var jobState = new JobState(dataStore);

    // Make it available to our bot
    services.AddSingleton(sp => jobState);

    // ...      
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Un bot necesita un sistema de almacenamiento de estado para mantener el diálogo y el estado de usuario entre los mensajes, que en este caso se define con el proveedor de almacenamiento en memoria.

```javascript
// index.js

const memoryStorage = new MemoryStorage();
const botState = new BotState(memoryStorage, () => 'proactiveBot.botState');

// Create the main dialog, which serves as the bot's main handler.
const bot = new ProactiveBot(botState, adapter);

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (turnContext) => {
        // Route the message to the bot's main handler.
        await bot.onTurn(turnContext);
    });
});

// ...
```

---

## <a name="define-the-bot"></a>Definición del bot

El usuario puede pedir al bot que cree y ejecute un trabajo en su lugar. Un servicio de trabajo independiente podría enviar una notificación al bot cada vez que se termina un trabajo. El bot está diseñado para:

- Crear un trabajo en respuesta a un mensaje `run` o `run job` del usuario.
- Mostrar todos los trabajos registrados en respuesta a un mensaje `show` o `show jobs` del usuario.
- Completar un trabajo en respuesta a un evento de _trabajo completado_ que identifica un trabajo completado.
- Simular un evento de trabajo completado en respuesta a un mensaje `done <jobIdentifier>`.
- Enviar un mensaje proactivo al usuario, mediante la conversación original, cuando finalice el trabajo.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

El bot incluye varios aspectos:

- código de inicialización
- un controlador de turnos
- métodos para crear y completar los trabajos

### <a name="declare-the-class"></a>Declaración de la clase

Cada interacción del usuario crea una instancia de la clase `ProactiveBot`. El proceso de crear un servicio cada vez que se necesita se conoce como servicio de duración transitoria. Los objetos que son costosos de construir o que tienen una duración más allá de un turno único deben administrarse con cuidado.

```csharp
namespace Microsoft.BotBuilderSamples
{
    public class ProactiveBot : IBot
    {
        // The name of events that signal that a job has completed.
        public const string JobCompleteEventName = "jobComplete";

        public const string WelcomeText = "Type 'run' or 'run job' to start a new job.\r\n" +
                                          "Type 'show' or 'show jobs' to display the job log.\r\n" +
                                          "Type 'done <jobNumber>' to complete a job.";
    }
}
```

### <a name="add-initialization-code"></a>Incorporación del código de inicialización

```csharp
private readonly JobState _jobState;
private readonly IStatePropertyAccessor<JobLog> _jobLogPropertyAccessor;

public ProactiveBot(JobState jobState, EndpointService endpointService)
{
    _jobState = jobState ?? throw new ArgumentNullException(nameof(jobState));
    _jobLogPropertyAccessor = _jobState.CreateProperty<JobLog>(nameof(JobLog));

    //...
}

```

### <a name="add-a-turn-handler"></a>Incorporación de un controlador de turnos

El adaptador reenvía las actividades al controlador de turnos, que inspecciona el tipo `Activity` y llama al método adecuado. Cada bot debe implementar un controlador de turnos.

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type != ActivityTypes.Message)
    {
        // Handle non-message activities.
        await OnSystemActivityAsync(turnContext);
    }
    else
    {
        // Get the job log.
        // The job log is a dictionary of all outstanding jobs in the system.
        JobLog jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());

        // Get the user's text input for the message.
        var text = turnContext.Activity.Text.Trim().ToLowerInvariant();
        switch (text)
        {
            case "run":
            case "run job":

                // Start a virtual job for the user.
                JobLog.JobData job = CreateJob(turnContext, jobLog);

                // Set the new property
                await _jobLogPropertyAccessor.SetAsync(turnContext, jobLog);

                // Now save it into the JobState
                await _jobState.SaveChangesAsync(turnContext);

                await turnContext.SendActivityAsync(
                    $"We're starting job {job.TimeStamp} for you. We'll notify you when it's complete.");

                break;

            case "show":
            case "show jobs":
                // Display information for all jobs in the log.
                // ...
                break;

            default:
                // Check whether this is simulating a job completed event.
                string[] parts = text?.Split(' ', StringSplitOptions.RemoveEmptyEntries);
                if (parts != null && parts.Length == 2
                    && parts[0].Equals("done", StringComparison.InvariantCultureIgnoreCase)
                    && long.TryParse(parts[1], out long jobNumber))
                {
                    if (!jobLog.TryGetValue(jobNumber, out JobLog.JobData jobInfo))
                    {
                        await turnContext.SendActivityAsync($"The log does not contain a job {jobInfo.TimeStamp}.");
                    }
                    else if (jobInfo.Completed)
                    {
                        await turnContext.SendActivityAsync($"Job {jobInfo.TimeStamp} is already complete.");
                    }
                    else
                    {
                        await turnContext.SendActivityAsync($"Completing job {jobInfo.TimeStamp}.");

                        // Send the proactive message.
                        await CompleteJobAsync(turnContext.Adapter, AppId, jobInfo);
                    }
                }

                break;
        }

        if (!turnContext.Responded)
        {
            await turnContext.SendActivityAsync(WelcomeText);
        }
    }
}

private static async Task SendWelcomeMessageAsync(ITurnContext turnContext)
{
    foreach (var member in turnContext.Activity.MembersAdded)
    {
        if (member.Id != turnContext.Activity.Recipient.Id)
        {
            await turnContext.SendActivityAsync($"Welcome to SuggestedActionsBot {member.Name}.\r\n{WelcomeText}");
        }
    }
}
```

### <a name="handle-non-message-activities"></a>Control de las actividades que no son mensajes

En un evento de trabajo terminado, marque el trabajo como terminado y notifique al usuario.

```csharp
private async Task OnSystemActivityAsync(ITurnContext turnContext)
{
    if (turnContext.Activity.Type is ActivityTypes.Event)
    {
        var jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());
        var activity = turnContext.Activity.AsEventActivity();
        if (activity.Name == JobCompleteEventName
            && activity.Value is long timestamp
            && jobLog.ContainsKey(timestamp)
            && !jobLog[timestamp].Completed)
        {
            await CompleteJobAsync(turnContext.Adapter, AppId, jobLog[timestamp]);
        }
    }
    else if (turnContext.Activity.Type is ActivityTypes.ConversationUpdate)
    {
        if (turnContext.Activity.MembersAdded.Any())
        {
            await SendWelcomeMessageAsync(turnContext);
        }
    }
}
```

### <a name="add-job-creation-and-completion-methods"></a>Incorporación de métodos de creación y finalización de trabajos

Para iniciar un trabajo, el bot crea el trabajo y registra la información sobre él y la conversación actual en el registro del trabajo. Cuando el bot recibe un evento de trabajo completado en cualquier conversación, valida el identificador de trabajo antes de llamar al código para completar el trabajo.

El código para completar el trabajo obtiene el registro del trabajo del estado y, a continuación, marca el trabajo como completo y envía un mensaje proactivo, mediante el método `ContinueConversationAsync` del adaptador.

- La llamada al método continue conversation pide al canal que inicie un turno independiente del usuario.
- El adaptador ejecuta la devolución de llamada asociada en lugar del controlador OnTurn normal del bot. Este turno tiene su propio contexto de turno, del que se recupera la información de estado y se envía el mensaje proactivo al usuario.

```csharp
// Creates and "starts" a new job.
private JobLog.JobData CreateJob(ITurnContext turnContext, JobLog jobLog)
{
    JobLog.JobData jobInfo = new JobLog.JobData
    {
        TimeStamp = DateTime.Now.ToBinary(),
        Conversation = turnContext.Activity.GetConversationReference(),
    };

    jobLog[jobInfo.TimeStamp] = jobInfo;

    return jobInfo;
}
```

### <a name="sends-a-proactive-message-to-the-user"></a>Envía un mensaje proactivo al usuario.

```csharp
private async Task CompleteJobAsync(
    BotAdapter adapter,
    string botId,
    JobLog.JobData jobInfo,
    CancellationToken cancellationToken = default(CancellationToken))
{
    await adapter.ContinueConversationAsync(botId, jobInfo.Conversation, CreateCallback(jobInfo), cancellationToken);
}
```

### <a name="creates-the-turn-logic-to-use-for-the-proactive-message"></a>Crea la lógica de turnos que se usará para el mensaje proactivo.

```csharp
private BotCallbackHandler CreateCallback(JobLog.JobData jobInfo)
{
    return async (turnContext, token) =>
    {
        // Get the job log from state, and retrieve the job.
        JobLog jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());

        // Perform bookkeeping.
        jobLog[jobInfo.TimeStamp].Completed = true;

        // Set the new property
        await _jobLogPropertyAccessor.SetAsync(turnContext, jobLog);

        // Now save it into the JobState
        await _jobState.SaveChangesAsync(turnContext);

        // Send the user a proactive confirmation message.
        await turnContext.SendActivityAsync($"Job {jobInfo.TimeStamp} is complete.");
    };
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El bot se define en **bot.js** y presenta varios aspectos:

- código de inicialización
- un controlador de turnos
- métodos para crear y completar los trabajos

### <a name="declare-the-class-and-add-initialization-code"></a>Declaración de la clase e incorporación del código de inicialización

```javascript
const { ActivityTypes, TurnContext } = require('botbuilder');

const JOBS_LIST = 'jobs';

class ProactiveBot {
    constructor(botState, adapter) {
        this.botState = botState;
        this.adapter = adapter;

        this.jobsList = this.botState.createProperty(JOBS_LIST);
    }

    // ...
};

// Helper function to check if object is empty.
function isEmpty(obj) {
    for (var key in obj) {
        if (obj.hasOwnProperty(key)) {
            return false;
        }
    }
    return true;
};

module.exports.ProactiveBot = ProactiveBot;
```

### <a name="the-turn-handler"></a>El controlador de turnos

Los métodos `onTurn` y `showJobs` se definen en la clase `ProactiveBot`. `onTurn` controla las entradas de los usuarios. También recibiría actividades de evento desde un hipotético sistema de procesamiento de trabajos. `showJobs` da formato al registro de trabajo y lo envía.

```javascript
/**
    *
    * @param {TurnContext} turnContext A TurnContext object representing an incoming message to be handled by the bot.
    */
async onTurn(turnContext) {
    // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
    if (turnContext.activity.type === ActivityTypes.Message) {
        const utterance = (turnContext.activity.text || '').trim().toLowerCase();
        var jobIdNumber;

        // If user types in run, create a new job.
        if (utterance === 'run') {
            await this.createJob(turnContext);
        } else if (utterance === 'show') {
            await this.showJobs(turnContext);
        } else {
            const words = utterance.split(' ');

            // If the user types done and a Job Id Number,
            // we check if the second word input is a number.
            if (words[0] === 'done' && !isNaN(parseInt(words[1]))) {
                jobIdNumber = words[1];
                await this.completeJob(turnContext, jobIdNumber);
            } else if (words[0] === 'done' && (words.length < 2 || isNaN(parseInt(words[1])))) {
                await turnContext.sendActivity('Enter the job ID number after "done".');
            }
        }

        if (!turnContext.responded) {
            await turnContext.sendActivity(`Say "run" to start a job, or "done <job>" to complete one.`);
        }
    } else if (turnContext.activity.type === ActivityTypes.Event && turnContext.activity.name === 'jobCompleted') {
        jobIdNumber = turnContext.activity.value;
        if (!isNaN(parseInt(jobIdNumber))) {
            await this.completeJob(turnContext, jobIdNumber);
        }
    }

    await this.botState.saveChanges(turnContext);
}

// Show a list of the pending jobs
async showJobs(turnContext) {
    const jobs = await this.jobsList.get(turnContext, {});
    if (Object.keys(jobs).length) {
        await turnContext.sendActivity(
            '| Job number &nbsp; | Conversation ID &nbsp; | Completed |<br>' +
            '| :--- | :---: | :---: |<br>' +
            Object.keys(jobs).map((key) => {
                return `${ key } &nbsp; | ${ jobs[key].reference.conversation.id.split('|')[0] } &nbsp; | ${ jobs[key].completed }`;
            }).join('<br>'));
    } else {
        await turnContext.sendActivity('The job log is empty.');
    }
}
```

### <a name="logic-to-start-a-job"></a>Lógica para iniciar un trabajo

El método `createJob` se define dentro de la clase `ProactiveBot`. Crea y registra nuevos trabajos para el usuario. En teoría, también reenviaría esta información al sistema de procesamiento de trabajos.

```javascript
// Save job ID and conversation reference.
async createJob(turnContext) {
    // Create a unique job ID.
    var date = new Date();
    var jobIdNumber = date.getTime();

    // Get the conversation reference.
    const reference = TurnContext.getConversationReference(turnContext.activity);

    // Get the list of jobs. Default it to {} if it is empty.
    const jobs = await this.jobsList.get(turnContext, {});

    // Try to find previous information about the saved job.
    const jobInfo = jobs[jobIdNumber];

    try {
        if (isEmpty(jobInfo)) {
            // Job object is empty so we have to create it
            await turnContext.sendActivity(`Need to create new job ID: ${ jobIdNumber }`);

            // Update jobInfo with new info
            jobs[jobIdNumber] = { completed: false, reference: reference };

            try {
                // Save to storage
                await this.jobsList.set(turnContext, jobs);
                // Notify the user that the job has been processed
                await turnContext.sendActivity('Successful write to log.');
            } catch (err) {
                await turnContext.sendActivity(`Write failed: ${ err.message }`);
            }
        }
    } catch (err) {
        await turnContext.sendActivity(`Read rejected. ${ err.message }`);
    }
}
```

### <a name="logic-to-complete-a-job"></a>Lógica para completar un trabajo

El método `completeJob` se define dentro de la clase `ProactiveBot`. Realiza algo de contabilidad y envía el mensaje proactivo al usuario (en la conversación original del usuario) para avisarle de que su trabajo se ha terminado.

```javascript
async completeJob(turnContext, jobIdNumber) {
    // Get the list of jobs from the bot's state property accessor.
    const jobs = await this.jobsList.get(turnContext, {});

    // Find the appropriate job in the list of jobs.
    let jobInfo = jobs[jobIdNumber];

    // If no job was found, notify the user of this error state.
    if (isEmpty(jobInfo)) {
        await turnContext.sendActivity(`Sorry no job with ID ${ jobIdNumber }.`);
    } else {
        // Found a job with the ID passed in.
        const reference = jobInfo.reference;
        const completed = jobInfo.completed;

        // If the job is not yet completed and conversation reference exists,
        // use the adapter to continue the conversation with the job's original creator.
        if (reference && !completed) {
            // Since we are going to proactively send a message to the user who started the job,
            // we need to create the turnContext based on the stored reference value.
            await this.adapter.continueConversation(reference, async (proactiveTurnContext) => {
                // Complete the job.
                jobInfo.completed = true;
                // Save the updated job.
                await this.jobsList.set(turnContext, jobs);
                // Notify the user that the job is complete.
                await proactiveTurnContext.sendActivity(`Your queued job ${ jobIdNumber } just completed.`);
            });

            // Send a message to the person who completed the job.
            await turnContext.sendActivity('Job completed. Notification sent.');
        } else if (completed) { // The job has already been completed.
            await turnContext.sendActivity('This job is already completed, please start a new job.');
        };
    };
};
```

---

## <a name="test-your-bot"></a>Prueba del bot

Ejecute el bot localmente y abra dos ventanas del emulador. Si necesita instrucciones paso a paso, consulte el archivo [Léame](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/16.proactive-messages/README.md).

1. Tenga en cuenta que el identificador de conversación es diferente en las dos ventanas.
1. En la primera ventana, escriba `run` un par de veces para iniciar varios trabajos.
1. En la segunda ventana, escriba `show` para ver una lista de los trabajos del registro.
1. En la segunda ventana, escriba `done <jobNumber>`, donde `<jobNumber>` es uno de los números de trabajo del registro, sin los corchetes angulares. (El código del bot está diseñado para interpretar esto como si fuese un evento jobComplete).
1. Tenga en cuenta que el bot envía un mensaje proactivo al usuario en la primera ventana.

La conversación podría tener el siguiente aspecto desde la perspectiva del usuario:

![Sesión de emulador del usuario](~/v4sdk/media/how-to-proactive/user.png)

Y tiene este aspecto desde la perspectiva del sistema de trabajo simulado:

![Sesión de emulador del sistema de trabajo](~/v4sdk/media/how-to-proactive/job-system.png)

## <a name="additional-resources"></a>Recursos adicionales
Consulte ejemplos adicionales en C# y JS en [GitHub](https://github.com/Microsoft/BotBuilder-Samples/blob/master/readme.md).

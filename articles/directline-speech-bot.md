---
title: Desarrollo de un bot de DirectLine Speech | Microsoft Docs
description: Desarrollo de un bot de DirectLine Speech
keywords: desarrollo de un bot de DirectLine Speech, bot de voz
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 07/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 8294ca4b58c2a50d55bdfd9a81cc2c6fb57f3922
ms.sourcegitcommit: d493caf74b87b790c99bcdaddb30682251e3fdd4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/26/2019
ms.locfileid: "71279006"
---
# <a name="use-direct-line-speech-in-your-bot"></a>Uso de Direct Line Speech en el bot

[!INCLUDE [applies-to-v4](includes/applies-to.md)]

Direct Line Speech usa una nueva funcionalidad de streaming basada en WebSocket de Bot Framework para intercambiar mensajes entre el canal de Direct Line Speech y el bot. Después de habilitar el canal de Direct Line Speech en Azure Portal, deberá actualizar el bot para escuchar y aceptar estas conexiones de WebSocket. Estas instrucciones explican cómo hacerlo.

## <a name="add-additional-nuget-packages"></a>Adición de paquetes NuGet adicionales

Para Direct Line Speech (versión preliminar) hay paquetes de NuGet adicionales que deberá agregar al bot.

- **Microsoft.Bot.Builder.StreamingExtensions** 4.5.1-preview1

También instalará el paquete siguiente:

- **Microsoft.Bot.StreamingExtensions** 4.5.1-preview1

Si no los encuentra al principio, asegúrese de que ha incluido los paquetes de versión preliminar en la búsqueda.

## <a name="set-the-speak-field-on-activities-you-want-spoken-to-the-user"></a>Establecer el campo Speak (Hablar) en las actividades que quiera que se hablen con el usuario

Debe establecer el campo Speak (Hablar) de cualquier actividad que se envíe desde el bot y que quiera que se hable con el usuario.

```cs
public IActivity Speak(string message)
{
    var activity = MessageFactory.Text(message);
    string body = @"<speak version='1.0' xmlns='https://www.w3.org/2001/10/synthesis' xml:lang='en-US'>
        <voice name='Microsoft Server Speech Text to Speech Voice (en-US, JessaRUS)'>" +
        $"{message}" + "</voice></speak>";
    activity.Speak = body;
    return activity;
}
```

En el fragmento de código siguiente se muestra cómo usar la función *Speak* anterior:

```cs
protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    await turnContext.SendActivityAsync(Speak($"Echo: {turnContext.Activity.Text}"), cancellationToken);
}
```

## <a name="option-1-update-your-net-core-bot-code-_if-your-bot-has-a-botcontrollercs_"></a>Opción 1: Actualización del código del bot de .NET Core _si el bot tiene un elemento BotController.cs_

Cuando se crea un nuevo bot desde Azure Portal mediante una de las plantillas, como EchoBot, obtendrá un bot que incluye un controlador MVC de ASP.NET que expone un punto de conexión POST único. Estas instrucciones explican cómo expandirlo para exponer también un punto de conexión que acepte el punto de conexión de streaming de WebSocket, que es un punto de conexión GET.

1. Abra BotController.cs en la carpeta Controllers de la solución

2. Busque el método PostAsync en la clase y actualice su decoración de [HttpPost] a [HttpGet, HttpPost]:

    ```cs
    [HttpPost, HttpGet]
    public async Task PostAsync()
    {
        await _adapter.ProcessAsync(Request, Response, _bot);
    }
    ```

3. Guarde y cierre BotController.cs

4. Abra Startup.cs en la raíz de la solución.

5. Agregue un nuevo espacio de nombres:

    ```cs
    using Microsoft.Bot.Builder.StreamingExtensions;
    ```

6. En el método ConfigureServices, reemplace el uso de AdapterWithErrorHandler por WebSocketEnabledHttpAdapter en la llamada a services.AddSingleton apropiada:

    ```cs
    public void ConfigureServices(IServiceCollection services)
    {
        ...

        // Create the Bot Framework Adapter.
        services.AddSingleton<IBotFrameworkHttpAdapter, WebSocketEnabledHttpAdapter>();

        services.AddTransient<IBot, EchoBot>();

        ...
    }
    ```

7. Todavía en Startup.cs, navegue a la parte inferior del método Configure. Antes de la llamada a `app.UseMvc()`, agregue una llamada a `app.UseWebSockets()`. Esto es importante, ya que el orden de estas llamadas a _use_ importa. El final del método debería parecerse al siguiente:

    ```cs
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        ...

        app.UseDefaultFiles();
        app.UseStaticFiles();
        app.UseWebSockets();
        app.UseMvc();

        ...
    }
    ```

8. El resto del código del bot sigue siendo el mismo.

## <a name="option-2-update-your-net-core-bot-code-_if-your-bot-uses-addbot-and-usebotframework-instead-of-a-botcontroller_"></a>Opción 2: Actualización del código del bot de .NET Core _si el bot usa AddBot y UseBotFramework en lugar de BotController_

Si ha creado un bot con una versión v4 de Bot Builder SDK anterior a la versión 4.3.2, es probable que el bot no incluya un elemento BotController sino que use los métodos AddBot() y UseBotFramework() en el archivo Startup.cs para exponer el punto de conexión POST en el que el bot recibe los mensajes. Para exponer el nuevo punto de conexión de streaming, deberá agregar un elemento BotController y quitar los métodos AddBot() y UseBotFramework(). Estas instrucciones le guiarán en los cambios que deben realizarse.

1. Agregue un nuevo controlador MVC al proyecto del bot mediante la adición de un archivo llamado BotController.cs. Agregue el código del controlador a este archivo:

    ```cs
    [Route("api/messages")]
    [ApiController]
    public class BotController : ControllerBase
    {
        private readonly IBotFrameworkHttpAdapter _adapter;
        private readonly IBot _bot;

        public BotController(IBotFrameworkHttpAdapter adapter, IBot bot)
        {
            _adapter = adapter;
            _bot = bot;
        }

        [HttpPost, HttpGet]
        public async Task ProcessMessageAsync()
        {
            await _adapter.ProcessAsync(Request, Response, _bot);
        }
    }
    ```

2. En el archivo Startup.cs, busque el método Configure. Quite la línea UseBotFramework() y asegúrese de tener estas líneas:

    ```cs
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        ...

        app.UseDefaultFiles();
        app.UseStaticFiles();
        app.UseWebSockets();
        app.UseMvc();

        ...
    }
    ```

3. También en el archivo Startup.cs, busque el método ConfigureServices. Quite la línea AddBot() y asegúrese de tener estas líneas:

    ```cs
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

        services.AddSingleton<ICredentialProvider, ConfigurationCredentialProvider>();

        services.AddSingleton<IChannelProvider, ConfigurationChannelProvider>();

        // Create the Bot Framework Adapter.
        services.AddSingleton<IBotFrameworkHttpAdapter, WebSocketEnabledHttpAdapter>();

        // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
        services.AddTransient<IBot, EchoBot>();
    }
    ```

4. El resto del código del bot sigue siendo el mismo.

## <a name="additional-information"></a>Información adicional

- Para ver un ejemplo completo de cómo crear y usar un bot habilitado para voz, consulte [Tutorial: Habilitación del bot con voz mediante el SDK de Speech](https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/tutorial-voice-enable-your-bot-speech-sdk).

- Para más información sobre cómo trabajar con actividades, consulte [cómo funcionan los bots](v4sdk/bot-builder-basics.md) y [cómo enviar y recibir mensajes de texto](v4sdk/bot-builder-howto-send-messages.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Conexión de un bot con Direct Line Speech](./bot-service-channel-connect-directlinespeech.md)

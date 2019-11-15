---
title: Desarrollo de un bot de DirectLine Speech | Microsoft Docs
description: Desarrollo de un bot de DirectLine Speech
keywords: desarrollo de un bot de DirectLine Speech, bot de voz
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/01/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4fc2eb84751f64a1ca1493515ccb231a0a78bbd4
ms.sourcegitcommit: 490810d278d1c8207330b132f28a5eaf2b37bd07
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/05/2019
ms.locfileid: "73592252"
---
# <a name="use-direct-line-speech-in-your-bot"></a>Uso de Direct Line Speech en el bot

[!INCLUDE [applies-to-v4](includes/applies-to.md)]

Direct Line Speech usa una nueva funcionalidad de streaming basada en WebSocket de Bot Framework para intercambiar mensajes entre el canal de Direct Line Speech y el bot. Después de habilitar el canal de Direct Line Speech en Azure Portal, deberá actualizar el bot para escuchar y aceptar estas conexiones de WebSocket. Estas instrucciones explican cómo hacerlo.  

## <a name="step-1-upgrade-to-the-46-sdk"></a>Paso 1: Actualización al SDK 4.6 

Para Direct Line Speech, asegúrese de que usa la versión 4.6 o superior de SDK Bot Builder. 

## <a name="step-2-update-your-net-core-bot-codeif-your-bot-uses-addbot-and-usebotframework-instead-of-a-botcontroller"></a>Paso 2: Actualice el código del bot de .NET Core si el bot usa AddBot y UseBotFramework en lugar de un elemento BotController. 

Si ha creado un bot con una versión v4 de Bot Builder SDK anterior a la versión 4.3.2, es probable que el bot no incluya un elemento BotController sino que use los métodos AddBot() y UseBotFramework() en el archivo Startup.cs para exponer el punto de conexión POST en el que el bot recibe los mensajes. Para exponer el nuevo punto de conexión de streaming, deberá agregar un elemento BotController y quitar los métodos AddBot() y UseBotFramework(). Estas instrucciones le guiarán en los cambios que deben realizarse. Si ya tiene estos cambios, continúe con el paso siguiente. 

Agregue un nuevo controlador MVC al proyecto del bot mediante la adición de un archivo llamado BotController.cs. Agregue el código del controlador a este archivo: 

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

En el archivo **Startup.cs**, busque el método Configure. Elimine la línea de `UseBotFramework()` y asegúrese de que tiene estas líneas para `UseWebSockets`: 

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

También en el archivo Startup.cs, busque el método ConfigureServices. Elimine la línea de `AddBot()` y asegúrese de que tiene líneas para agregar el elemento `IBot` y un elemento `BotFrameworkHttpAdapter`: 

```cs

public void ConfigureServices(IServiceCollection services) 
{ 
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1); 
    services.AddSingleton<ICredentialProvider, ConfigurationCredentialProvider>(); 
    services.AddSingleton<IChannelProvider, ConfigurationChannelProvider>(); 
    
    // Create the Bot Framework Adapter. 
    services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkHttpAdapter>(); 

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot. 
    services.AddTransient<IBot, EchoBot>(); 
} 
```

El resto del código del bot sigue siendo el mismo. 

## <a name="step3-ensure-websockets-are-enabled"></a>Paso 3: Asegurarse de que los WebSockets están habilitados 

Cuando se crea un nuevo bot desde Azure Portal mediante una de las plantillas, como EchoBot, obtendrá un bot que incluye un controlador MVC de ASP.NET que expone un punto de conexión GET y POST único y que también utilizará WebSockets. En estas instrucciones se explica cómo agregar estos elementos al bot si lo está actualizando o si no creó el bot a partir de una plantilla. 

Abra **BotController.cs** en la carpeta Controllers de la solución. 

Busque el método `PostAsync` de la clase y actualice su decoración de [HttpPost] a [HttpPost, HttpGet]: 

```cs

[HttpPost, HttpGet] 
public async Task PostAsync() 
{ 
    await _adapter.ProcessAsync(Request, Response, _bot); 
} 
```

Guarde y cierre BotController.cs 

Abra **Startup.cs** en la carpeta raíz de la solución. 

En Startup.cs, vaya a la parte inferior del método Configure. Antes de la llamada a  `app.UseMvc()`, agregue una llamada a  `app.UseWebSockets()`. Esto es importante, ya que el orden de estas llamadas a use importa. El final del método debería parecerse al siguiente: 

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
El resto del código del bot sigue siendo el mismo. 

 

Paso 4: Opcionalmente, puede establecer el campo Speak en Activities para personalizar lo que se habla al usuario. 

De forma predeterminada, se hablarán todos los mensajes enviados mediante Direct Line Speech al usuario.  

Opcionalmente, puede personalizar cómo se habla el mensaje estableciendo el campo Speak de cualquier actividad enviada desde el bot: 

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

En el fragmento de código siguiente se muestra cómo usar la función Speak anterior: 

```cs

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken) 
{ 
    await turnContext.SendActivityAsync(Speak($"Echo: {turnContext.Activity.Text}"), cancellationToken); 
} 
``` 

## <a name="additional-information"></a>Información adicional 

- Para ver un ejemplo completo de cómo crear y usar un bot habilitado para voz, consulte  [Tutorial: Habilitación del bot con voz mediante el SDK de Speech](https://docs.microsoft.com/azure/cognitive-services/speech-service/tutorial-voice-enable-your-bot-speech-sdk). 

- Para más información sobre cómo trabajar con actividades, consulte  [Cómo funcionan los bots](https://docs.microsoft.com/azure/bot-service/bot-builder-basics) y  [Cómo enviar y recibir mensajes de texto https://docs.microsoft.com/azure/bot-service/bot-builder-howto-send-messages?view=azure-bot-service-4.0](). 

 

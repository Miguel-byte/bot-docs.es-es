---
title: Conexión de un bot a WeChat | Microsoft Docs
description: Obtenga información sobre cómo configurar la conexión de un bot a WeChat.
keywords: WeChat, Tencent, bot channel, WeChat App, WeChat bot, App ID, App Secret, credentials
author: seaen
manager: kamrani
ms.topic: article
ms.author: egorn
ms.service: bot-service
ms.date: 11/01/2019
ms.openlocfilehash: aee02ba1707f08fbcb4479b37edd065fd28efd8f
ms.sourcegitcommit: 4751c7b8ff1d3603d4596e4fa99e0071036c207c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/02/2019
ms.locfileid: "73443093"
---
# <a name="connect-a-bot-to-wechat"></a>Conexión de un bot a WeChat

Puede configurar el bot para comunicarse con personas mediante la Plataforma oficial de cuentas de WeChat.

## <a name="download-wechat-adapter-for-bot-framework"></a>Descarga del adaptador de WeChat para Bot Framework

El adaptador de WeChat para Microsoft Bot Framework es un adaptador de código abierto de GitHub. [Descarga del adaptador de WeChat para Bot Framework](https://github.com/microsoft/BotFramework-WeChat/)

## <a name="create-a-wechat-account"></a>Creación de una cuenta de WeChat

Para configurar un bot para que se comunique mediante WeChat, debe crear una cuenta oficial de WeChat en la [Plataforma oficial de cuentas de WeChat](https://mp.weixin.qq.com/?lang=en_US) y, después, conectar el bot a la aplicación. Actualmente solo se admite la cuenta de servicio.

### <a name="change-your-prefer-language"></a>Cambio del idioma preferido

Puede cambiar el idioma de visualización que prefiera antes del inicio de sesión.

 ![Cambio de idioma](./media/channels/wechat-change-language.png)

### <a name="register-a-service-account"></a>Registro de una cuenta de servicio

WeChat debe comprobar la cuenta de servicio real, no se pueden habilitar webhooks antes de que se compruebe la cuenta. Para crear su propia cuenta de servicio, siga las instrucciones descritas [aquí](https://kf.qq.com/product/weixinmp.html#hid=87).
Para abreviar, solo tiene que hacer clic en el botón Registrarme ahora que se encuentra en la parte superior, seleccionar la cuenta de servicio y seguir las instrucciones.

 ![Registro de cuenta](./media/channels/wechat-register-account.png)

### <a name="sandbox-account"></a>Cuenta de espacio aislado

Si solo quiere probar la integración entre WeChat y el bot, puede usar una cuenta de espacio aislado en lugar de crear la cuenta de servicio. Más información sobre la creación de una [cuenta de espacio aislado](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login).

## <a name="enable-wechat-adapter-to-bot"></a>Habilitación del adaptador de WeChat en el bot

El proyecto del bot es un proyecto normal de Bot Framework SDK V4. Antes de poder iniciarlo, debe asegurarse de que puede ejecutar el bot. Descargue el [Adaptador de WeChat para Bot Framework](https://github.com/microsoft/BotFramework-WeChat/).

### <a name="prerequisites"></a>Requisitos previos

    .NET Core SDK (version 2.2.x)

### <a name="add-reference-to-wechat-adapter-source"></a>Adición de la referencia al origen del adaptador de WeChat

Haga referencia directamente al proyecto del adaptador de WeChat o agregue ~/BotFramework-WeChat/libraries/csharp_dotnetcore/outputpackages como origen de NuGet local.

### <a name="inject-wechat-adapter-in-your-bot-startupcs"></a>Inserción del adaptador de WeChat en el archivo Startup.cs del bot

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);

    // Create the storage we'll be using for User and Conversation state. (Memory is great for testing purposes.)
    services.AddSingleton<IStorage, MemoryStorage>();

    // Create the User state. (Used in this bot's Dialog implementation.)
    services.AddSingleton<UserState>();

    // Create the Conversation state. (Used by the Dialog system itself.)
    services.AddSingleton<ConversationState>();

    // Load WeChat settings.
    var wechatSettings = new WeChatSettings();
    Configuration.Bind("WeChatSettings", wechatSettings);
    services.AddSingleton<WeChatSettings>(wechatSettings);

    // Configure hosted serivce.
    services.AddSingleton<IBackgroundTaskQueue, BackgroundTaskQueue>();
    services.AddHostedService<QueuedHostedService>();
    services.AddSingleton<WeChatHttpAdapter>();

    // The Dialog that will be run by the bot.
    services.AddSingleton<MainDialog>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```

### <a name="update-your-bot-controller"></a>Actualización del controlador del bot

```csharp
[Route("api/messages")]
[ApiController]
public class BotController : ControllerBase
{  
    private readonly IBot _bot;
    private readonly WeChatHttpAdapter _weChatHttpAdapter;
    private readonly string Token;
    public BotController(IBot bot, WeChatHttpAdapter weChatAdapter)
    {
        _bot = bot;
        _weChatHttpAdapter = weChatAdapter;
    }

    [HttpPost("/WeChat")]
    [HttpGet("/WeChat")]
    public async Task PostWeChatAsync([FromQuery] SecretInfo secretInfo)
    {
        // Delegate the processing of the HTTP POST to the adapter.
        // The adapter will invoke the bot.
        await _weChatHttpAdapter.ProcessAsync(Request, Response, _bot, secretInfo);
    }
}
```

### <a name="setup-appsettingsjson"></a>Configuración de appsettings.json

Debe configurar el archivo appsettings.json antes de iniciar el bot; puede encontrar lo necesario a continuación.

```json
"WeChatSettings": {
    "UploadTemporaryMedia": true,
    "PassiveResponseMode": false,
    "Token": "",
    "EncodingAESKey": "",
    "AppId": "",
    "AppSecret": ""
}
```

#### <a name="service-account"></a>Cuenta de servicio

Si ya tiene una cuenta de servicio y está lista para su implementación, puede encontrar los elementos **AppID**, **AppSecret**, **EncodingAESKey** y **Token** en las configuraciones básicas de la barra de navegación izquierda, como se muestra a continuación.

No olvide configurar la lista de direcciones IP permitidas; de lo contrario, WeChat no aceptará la solicitud.

 ![Consola de la cuenta de servicio](./media/channels/wechat-serviceaccount-console.png)

#### <a name="sandbox-account"></a>Cuenta de espacio aislado

La cuenta de espacio aislado no tiene elemento **EncodingAESKey**, el mensaje de WeChat no se cifró, deje EncodingAESKey en blanco. Aquí solo tiene tres parámetros, **appID**, **appsecret** y **Token**.

 ![Cuenta de espacio aislado](./media/channels/wechat-sandbox-account.png)

### <a name="start-bot-and-set-endpoint-url"></a>Inicio del bot y establecimiento de la dirección URL del punto de conexión

Ahora puede establecer el back-end del bot. Antes de hacerlo, debe iniciar el bot antes de guardar la configuración; WeChat le enviará una solicitud para comprobar la dirección URL.
Establezca el punto de conexión en este patrón: **https://your_end_point/WeChat** , o bien establezca la configuración personal tal y como ha hecho en BotController.cs

 ![sandbox_account2](./media/channels/wechat-sandbox-account-2.png)

### <a name="subscribe-your-official-account"></a>Suscripción a la cuenta oficial

Puede encontrar un código QR para suscribirse a la cuenta de prueba como en WeChat.

 ![subscribe](./media/channels/wechat-subscribe.png)

## <a name="test-through-wechat"></a>Prueba mediante WeChat

Todo esta listo, puede probarlo en el cliente de WeChat. Puede probar el bot de ejemplo en la carpeta de pruebas. Este bot de ejemplo incluye el adaptador de WeChat y se integra con el bot de eco y el bot de tarjetas.

 ![chat](./media/channels/wechat-chat.png)

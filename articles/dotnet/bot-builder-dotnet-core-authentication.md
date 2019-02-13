---
redirect_url: /bot-framework/bot-builder-tutorial-authentication
ms.openlocfilehash: 6dd1013e870fb749919f272e82b60ee633cddadb
ms.sourcegitcommit: c7d2e939ec71f46f48383c750fddaf6627b6489d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/07/2019
ms.locfileid: "55783374"
---
<a name="--"></a><!--
---
título: Autenticación de actividades con .NET Core | Microsoft Docs descripción: Obtenga información sobre cómo autenticar las actividades de bots con .NET Core.
author: v-ducvo ms.author: v-ducvo manager: kamrani ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date: 12/13/17 monikerRange: 'azure-bot-service-3.0'
---

# <a name="authenticating-activities-using-net-core"></a>Autenticación de actividades con .NET Core

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Si decide desarrollar el bot con [.NET Core](/dotnet/core/index), puede usar el [Bot Framework Connector](bot-builder-dotnet-connector.md) para enviar y recibir mensajes de [actividad](https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html) desde el bot. Para usar el servicio Connector, debe configurar el modelo de autenticación adecuado para la versión del marco de destino.

Bot Framework Connector.AspNetCore admite las siguientes versiones de ASP.NET:
* Para .NET Core v1.1/AspNetCore1.x, use **Microsoft.Bot.Connector.AspNetCore 1.x**
* Para .NET Core v2.0/AspNetCore2.x, use **Microsoft.Bot.Connector.AspNetCore 2.x**

En este artículo se muestra cómo configurar el modelo de autenticación para el marco específico que el bot tiene como destino.

## <a name="prerequisites"></a>Requisitos previos

* [Visual Studio 2017](https://www.visualstudio.com/downloads/)
* [.NET Core](https://www.microsoft.com/net/download/windows). Instale la versión de .NET Core de destino (p. ej.: .NET Core v1.1 o .NET Core v2.0).
* [Registre un bot](~/bot-service-quickstart-registration.md). Registre el bot para obtener un AppID y una contraseña necesarios para el proceso de autenticación.

## <a name="create-a-net-core-project"></a>Creación de un proyecto de .NET Core

Para crear el proyecto de .NET Core, realice lo siguiente:

1. Abra Visual Studio 2017 y haga clic en **Archivo > Nuevo > Proyecto…**
2. Expanda el nodo **Visual C#** y haga clic en **.NET Core**.
3. Elija el tipo de proyecto **Aplicación web ASP.NET Core** y rellene la información del proyecto (p. ej.: los campos Nombre, Ubicación y Nombre de la solución).
4. Haga clic en **OK**.
5. Asegúrese de que el proyecto tenga como destino *.NET Core* y la versión de *ASP.NET Core* que desee. Por ejemplo, la captura de pantalla siguiente muestra que el proyecto tiene como destino **.NET Core** y **ASP.NET Core 2.0**:

![Crear un proyecto de ASP.NET Core v2.0](~/media/dotnet-core-authentication/create-asp-net-core-2x-project.png)
 
  > [!NOTE]
  > Si el destino es **ASP.NET Core 2.0**, asegúrese de que la versión instalada en nuestra máquina sea al menos 2.x y posterior.

6. Elija el tipo de proyecto **API web**.
7. Haga clic en **Aceptar** para crear el proyecto.

## <a name="download-the-nuget-package"></a>Descarga del paquete NuGet

Para usar el **Bot Framework Connector**, instale el paquete NuGet adecuado para su versión de .NET Core. Realice los pasos siguientes para instalar el paquete NuGet:

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el nombre del proyecto y **Administrar paquetes NuGet…**
2. Haga clic en **Examinar** y busque **Connector.ASPNetCore**. 
3. Elija la versión que desee establecer como destino. Por ejemplo, la captura de pantalla siguiente muestra la versión **2.0.0.3** seleccionada. Para instalar la versión 1.1.3.2, haga clic en la lista desplegable **Versión** y elija la versión **1.1.3.2** en su lugar.
![Paquete NuGet para ASP.NET Core versión 2.0.0.3](~/media/dotnet-core-authentication/nuget-package-net-core-version.png)
4. Haga clic en **Instalar**.

## <a name="update-the-appsettingsjson"></a>Actualización de appsettings.json

El Bot Framework Connector requiere su AppID y contraseña para autenticar el bot. Puede establecer estos valores en el archivo **appsettings.json** de su proyecto de aplicación web.

> [!NOTE]
> Para buscar **AppID** y **AppPassword** del bot, consulte [MicrosoftAppID y MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

### <a name="appsettingsjson-for-net-core-v11"></a>appsettings.json para .NET Core v1.1:

```json
{
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "MicrosoftAppId": "<MicrosoftAppId>",
  "MicrosoftAppPassword": "<MicrosoftAppPassword>"
}
```

### <a name="appsettingsjson-for-net-core-v20"></a>appsettings.json para .NET Core v2.0:

```json
{
  "Logging": {
    "IncludeScopes": false,
    "Debug": {
      "LogLevel": {
        "Default": "Warning"
      }
    },
    "Console": {
      "LogLevel": {
        "Default": "Warning"
      }
    }
  },
  "MicrosoftAppId": "<MicrosoftAppId>",
  "MicrosoftAppPassword": "<MicrosoftAppPassword>"
}
```

## <a name="update-the-startupcs-class"></a>Actualización de la clase startup.cs

Actualice la clase **startup.cs**. Según la versión del proyecto, actualice el fragmento de código adecuado para permitir que la autenticación funcione.

### <a name="startupcs-for-net-core-v11"></a>startup.cs para .NET Core v1.1

```cs
public class Startup
    {
        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        public IConfigurationRoot Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);

            services.AddSingleton(typeof(ICredentialProvider), new StaticCredentialProvider(
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppIdKey)?.Value,
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppPasswordKey)?.Value));

            // Add framework services.
            services.AddMvc(options =>
            {
                options.Filters.Add(typeof(TrustServiceUrlAttribute));
            });
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IServiceProvider serviceProvider, IHostingEnvironment env, ILoggerFactory loggerFactory)
        {
            loggerFactory.AddConsole(Configuration.GetSection("Logging"));
            loggerFactory.AddDebug();

            app.UseStaticFiles();

            ICredentialProvider credentialProvider = serviceProvider.GetService<ICredentialProvider>();

            app.UseBotAuthentication(credentialProvider);

            app.UseMvc();
        }
    }
```

### <a name="startupcs-for-net-core-v20"></a>startup.cs para .NET Core v2.0:

```cs
public class Startup
    {
        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        public IConfiguration Configuration { get; }
        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);

            var credentialProvider = new StaticCredentialProvider(Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppIdKey)?.Value,
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppPasswordKey)?.Value);

            services.AddAuthentication(
                    options =>
                    {
                        options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                        options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
                    }
                )
                .AddBotAuthentication(credentialProvider);

            services.AddSingleton(typeof(ICredentialProvider), credentialProvider);

            services.AddMvc(options =>
            {
                options.Filters.Add(typeof(TrustServiceUrlAttribute));
            });
        }


        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            app.UseStaticFiles();
            app.UseAuthentication();
            app.UseMvc();
        }
    }
```

## <a name="create-the-messagescontrollercs-class"></a>Creación de la clase MessagesController.cs

Desde el **Explorador de soluciones** agregue una nueva clase vacía denominada **MessagesController.cs**. En la clase **MessagesController.cs**, actualice el método **Post** con el código siguiente. Esto permitirá que el bot envíe y reciba mensajes para el usuario.

```cs
private IConfiguration configuration;

public MessagesController(IConfiguration configuration)
{
    this.configuration = configuration;
}


[Authorize(Roles = "Bot")]
[HttpPost]
public async Task<OkResult> Post([FromBody] Activity activity)
{
    if (activity.Type == ActivityTypes.Message)
    {
        //MicrosoftAppCredentials.TrustServiceUrl(activity.ServiceUrl);
        var appCredentials = new MicrosoftAppCredentials(configuration);
        var connector = new ConnectorClient(new Uri(activity.ServiceUrl), appCredentials);

        // return our reply to the user
        var reply = activity.CreateReply("HelloWorld");
        await connector.Conversations.ReplyToActivityAsync(reply);
    }
    else
    {
        //HandleSystemMessage(activity);
    }
    return Ok();
}
```
-->

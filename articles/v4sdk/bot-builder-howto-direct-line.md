---
title: Creación de un bot y un cliente de Direct Line | Microsoft Docs
description: Obtenga información sobre cómo crear un cliente y un bot de Direct Line con V4 del SDK de Bot Builder para .NET.
keywords: bot de direct line, cliente de direct line, canal personalizado, basado en consola, publicar
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/16/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c13733af0f6b26654952a0aab190f6d8cb06d059
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999982"
---
# <a name="create-a-direct-line-bot-and-client"></a>Creación de un bot y un cliente de Direct Line

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Los bots de Direct Line de Microsoft Bot Framework son bots que pueden funcionar con un cliente personalizado de su propio diseño. Los bots de Direct Line son increíblemente parecidos a los bots normales. Simplemente no necesitan utilizar los canales proporcionados.

Los clientes de Direct Line se pueden escribir para que sean lo que quiera. Puede escribir un cliente de Android, un cliente de iOS o incluso una aplicación cliente basada en consola.

En este tema obtendrá información sobre cómo crear e implementar un bot de Direct Line y cómo crear y ejecutar una aplicación cliente de Direct Line basada en consola.

## <a name="create-your-bot"></a>Creación del bot

Cada lenguaje sigue una ruta de acceso diferente para crear el bot. JavaScript crea el bot en Azure y luego modifica el código, mientras que C# crea el bot localmente y luego lo publica en Azure, pero ambos son métodos válidos y se pueden usar con cualquiera de estos lenguajes. Si quiere obtener más información sobre cómo publicar el bot en Azure, eche un vistazo a [deploying your bot in Azure](../bot-builder-howto-deploy-azure.md) (Implementación de un bot en Azure).

# <a name="ctabcscreatebot"></a>[C#](#tab/cscreatebot)

### <a name="create-the-solution-in-visual-studio"></a>Creación de la solución en Visual Studio

Para crear la solución para el bot de Direct Line, en Visual Studio 2015 o una versión posterior:

1. Cree una nueva **aplicación web ASP.NET Core** en **Visual C#** > **.NET Core**.

1. Escriba **DirectLineBotSample** como nombre y haga clic en **Aceptar**.

1. Asegúrese de que **.NET Core** y **ASP.NET Core 2.0** estén seleccionados, elija la plantilla de proyecto **Vacía** y, después, haga clic en **Aceptar**.

#### <a name="add-dependencies"></a>Adición de dependencias

1. En el **Explorador de soluciones**, haga clic con el botón derecho en **Dependencias** y, después, en **Administrar paquetes NuGet**.

1. Haga clic en **Examinar** y, a continuación, asegúrese de que la casilla **Incluir versión preliminar** esté activada.

1. Busque e instale los siguientes paquetes NuGet:
    - Microsoft.Bot.Builder
    - Microsoft.Bot.Builder.Core.Extensions
    - Microsoft.Bot.Builder.Integration.AspNet.Core
    - Microsoft.Rest.ClientRuntime
    - Newtonsoft.Json

### <a name="create-the-appsettingsjson-file"></a>Creación del archivo appsettings.json

El archivo appsettings.json contendrá el id. de la aplicación de Microsoft, la contraseña de la aplicación y la cadena de conexión de datos. Como este bot no almacenará la información de estado, la cadena de conexión de datos permanecerá vacía. Y, si solo usa Bot Framework Emulator, todas ellas pueden estar vacías.

Para crear un archivo **appsettings.json**:

1. Haga clic con el botón derecho en el proyecto **DirectLineBotSample** y elija **Agregar** > **Nuevo elemento**.

1. En **ASP.NET Core**, haga clic en **Archivo de configuración de ASP.NET**.

1. Escriba **appsettings.json** en el nombre y haga clic en **Agregar**.

Reemplace el contenido del archivo appsettings.json por el código siguiente:

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

    "MicrosoftAppId": "",
    "MicrosoftAppPassword": "",
    "DataConnectionString": ""
}
```

### <a name="edit-startupcs"></a>Edición de Startup.cs

Reemplace el contenido del archivo Startup.cs por el código siguiente:

**Nota**: **DirectBot** se definirá en el paso siguiente, por lo que puede pasar por alto el subrayado rojo.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace DirectLineBotSample
{
    public class Startup
    {
        public IConfiguration Configuration { get; }

        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);
            services.AddBot<DirectBot>(options =>
            {
                options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
            });
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseDefaultFiles();
            app.UseStaticFiles();
            app.UseBotFramework();
        }
    }
}
```

### <a name="create-the-directbot-class"></a>Creación de la clase DirectBot

La clase DirectBot contiene la mayor parte de la lógica para este bot.

1. Haga clic en el **DirectLineBotSample** > **Agregar** > **Nuevo elemento**.

1. En **ASP.NET Core**, haga clic en **Clase**.

1. Escriba **DirectBot.cs** en el nombre y haga clic en **Agregar**.

Reemplace el contenido del archivo DirectBot.cs por el código siguiente:

```csharp
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

namespace DirectLineBotSample
{
    public class DirectBot : IBot
    {
        public Task OnTurn(ITurnContext context)
        {
            // Respond to the various activity types.
            switch (context.Activity.Type)
            {
                case ActivityTypes.Message:
                    // Respond to the incoming text message.
                    RespondToMessage(context);
                    break;

                case ActivityTypes.ConversationUpdate:
                    break;

                case ActivityTypes.ContactRelationUpdate:
                    break;

                case ActivityTypes.Typing:
                    break;

                case ActivityTypes.DeleteUserData:
                    break;
            }

            return Task.CompletedTask;
        }

        /// <summary>
        /// Responds to the incoming message by either sending a hero card, an image, 
        /// or echoing the user's message.
        /// </summary>
        /// <param name="context">The context of this conversation.</param>
        private void RespondToMessage(ITurnContext context)
        {
            switch (context.Activity.Text.Trim().ToLower())
            {
                case "hi":
                case "hello":
                case "help":
                    // Send the user an instruction message.
                    context.SendActivity("Welcome to the Bot to showcase the DirectLine API. " +
                    "Send \"Show me a hero card\" or \"Send me a BotFramework image\" to see how the " +
                    "DirectLine client supports custom channel data. Any other message will be echoed.");
                    break;

                case "show me a hero card":
                    // Create the hero card.
                    HeroCard heroCard = new HeroCard()
                    {
                        Title = "Sample Hero Card",
                        Text = "Displayed in the DirectLine client"
                    };

                    // Attach the hero card to a new activity.
                    context.SendActivity(MessageFactory.Attachment(heroCard.ToAttachment()));
                    break;

                case "send me a botframework image":
                    // Create the image attachment.
                    Attachment imageAttachment = new Attachment()
                    {
                        ContentType = "image/png",
                        ContentUrl = "https://docs.microsoft.com/en-us/bot-framework/media/how-it-works/architecture-resize.png",
                    };

                    // Attach the image attachment to a new activity.
                    context.SendActivity(MessageFactory.Attachment(imageAttachment));
                    break;

                default:
                    // No command was encountered. Echo the user's message.
                    context.SendActivity($"You said \"{context.Activity.Text}\"");
                    break;
            }
        }
    }
}
```

**Nota:** No es necesario cambiar el archivo Program.cs.

Para comprobar que todo está en orden, presione **F6** para compilar el proyecto. No debería haber advertencias ni errores.

### <a name="publish-the-bot-from-visual-studio"></a>Publicación del bot desde Visual Studio

1. En el Explorador de soluciones de Visual Studio, haga clic con el botón derecho en el proyecto **DirectLineBotSample** y luego en **Establecer como proyecto de inicio**.

    ![Página de publicación de Visual Studio](media/bot-builder-howto-direct-line/visual-studio-publish-page.png)

1. Vuelva a hacer clic con el botón derecho en **DirectLineBotSample** y luego en **Publicar**. Se muestra la página **Publicar** de Visual Studio.

1. Si ya tiene un perfil de publicación para este bot, selecciónelo y haga clic en el botón **Publicar** y, después, vaya a la sección siguiente.

1. Para crear un nuevo perfil de publicación, seleccione el icono **Microsoft Azure App Service**.

1. Seleccione **Crear nuevo**.

1. Haga clic en el botón **Publicar** . Se muestra el cuadro de diálogo **Create App Service** (Crear una instancia de App Service).

    ![Cuadro de diálogo Create App Service (Crear una instancia de App Service)](media/bot-builder-howto-direct-line/create-app-service-dialog.png)

    - En Nombre de la aplicación, asígnele un nombre que pueda encontrar más adelante. Por ejemplo, puede agregar su nombre de correo electrónico al principio del nombre de la aplicación.

    - Compruebe si está usando la suscripción correcta.

    - En Grupo de recursos, compruebe que usa el grupo de recursos correcto. El grupo de recursos puede encontrarse en la hoja **Introducción** del bot. **Nota:** Un grupo de recursos incorrecto es difícil de corregir.

    - Compruebe si está usando el plan correcto de App Service.
    
1. Haga clic en el botón **Crear**. Visual Studio inicia la implementación del bot.

Una vez publicado el bot, aparecerá un explorador con el punto de conexión de dirección URL del bot.

### <a name="create-bot-channels-registration-bot-on-microsoft-azure"></a>Creación de un bot de registro de canales de bot en Microsoft Azure

El bot de Direct Line puede hospedarse en cualquier plataforma. En este ejemplo, el bot se hospedará en Microsoft Azure. 

Para crear el bot en Microsoft Azure:

1. En Microsoft Azure Portal, haga clic en **Crear un recurso** y, a continuación, busque "Bot Channels Registration" (Registro de canales de bot).

1. Haga clic en **Create**(Crear). Aparece la hoja Bot Channels Registration (Registro de canales de bot).

    ![Hoja Bot Channels Registration (Registro de canales de bot), con los campos de nombre del bot, suscripción, grupo de recursos, ubicación, plan de tarifa y punto de conexión de mensajería, entre otros.](media/bot-builder-howto-direct-line/bot-service-registration-blade.png)

1. En la hoja Bot Channels Registration (Registro de canales de bot), escriba el **nombre del bot**, la **suscripción**, el **grupo de recursos**, la **ubicación** y el **plan de tarifa**.

1. Deje el **Punto de conexión de mensajería** en blanco. Este valor se rellenará más adelante.

1. Haga clic en **Id. y contraseña de aplicación de Microsoft** y, a continuación, haga clic en **Creación automática del id. y la contraseña de la aplicación**.

1. Seleccione la casilla **Anclar al panel**.

1. Haga clic en el botón **Crear**.

La implementación tardará unos minutos, pero, una vez hecho esto, debería aparecer en el panel.

### <a name="update-the-appsettingsjson-file"></a>Actualización del archivo appsettings.json

1. Haga clic en el bot en el panel o, para ir al nuevo recurso, haga clic en **Todos los recursos** y busque el nombre de su registro de canales de bot.

1. Haga clic en **Descripción general**.

1. Copie el nombre del grupo de recursos en la cadena **botId** en **Program.cs** del proyecto **DirectLineClientSample**.

1. Haga clic en **Configuración** y, a continuación, haga clic en **Administrar** cerca del campo **Microsoft App ID** (Id. de la aplicación de Microsoft).

1. Copie el id. de la aplicación y péguelo en el campo **"MicrosoftAppId"** del archivo **appsettings.json**.

1. Haga clic en el botón **Generar nueva contraseña**.

1. Copie la nueva contraseña y péguela en el campo **"MicrosoftAppPassword"** del archivo **appsettings.json**.

### <a name="set-the-messaging-endpoint"></a>Establecimiento del punto de conexión de mensajería

Para agregar el punto de conexión al bot:

1. Copie la dirección URL del explorador que apareció después de publicar el bot.

    ![Hoja de configuración del registro de canales de bot, con el punto de conexión de mensajería resaltado](media/bot-builder-howto-direct-line/bot-channels-registration-settings.png)

1. Aparece la hoja **Configuración** del bot.

1. Pegue la dirección en el **punto de conexión de mensajería**.

1. Edite la dirección para que empiece por "https://" y termine por "/api/messages". Por ejemplo, si la dirección copiada del explorador es "http://v-royhar-dlbot-directlinebotsample20180329044602.azurewebsites.net", modifíquela a "https://v-royhar-dlbot-directlinebotsample20180329044602.azurewebsites.net/api/messages".

1. Haga clic en el botón **Guardar** en la hoja de configuración.

**NOTA:** Si se produce un error de publicación, es posible que deba detener el bot en Azure para poder publicar los cambios en el bot.

1. Busque el nombre de App Service (se encuentra entre "https://" y ".azurewebsites.net").

1. En "Todos los recursos" de Azure Portal, busque ese recurso.

1. Haga clic en el recurso de App Service. Aparece la hoja de App Service.

1. Haga clic en el botón **Detener** en la hoja de App Service.

1. En Visual Studio, vuelva a publicar el bot.

1. Haga clic en el botón **Inicio** en la hoja de App Service.

### <a name="test-your-bot-in-webchat"></a>Prueba del bot en el chat en web

Para comprobar que el bot funciona, compruébelo en el Chat en web:

1. En la hoja Bot Channels Registration (Registro de canales del bot) del bot, haga clic en **Configuración** y, a continuación, en **Probar en Chat en web**.

1. Escriba "Hola". El bot debería responder con un mensaje de bienvenida.

1. Escriba "Mostrarme una tarjeta de imagen prominente". El bot debería mostrar una tarjeta de imagen prominente.

1. Escriba "Enviarme una imagen de botframework". El bot debería mostrar una imagen de la documentación de Bot Framework.

1. Escriba cualquier otra cosa y el bot debería responder con "Ha dicho" y el mensaje entre comillas.

### <a name="troubleshooting"></a>solución de problemas

Si el bot no funciona, compruebe o vuelva a escribir el id. y la contraseña de aplicación de Microsoft. Aunque lo haya copiado con anterioridad, compruebe el id. de la aplicación de Microsoft en la hoja de configuración de Bot Channels Registration (Registro de canales del bot) con el valor del campo **"MicrosoftAppId"** del archivo appsettings.json.

Compruebe la contraseña o cree y use una nueva: 

1. Haga clic en **Administrar** junto al campo **Microsoft App ID** (Id. de la aplicación de Microsoft) en la hoja Bot Channels Registration (Registro de canales del bot).

1. Inicie sesión en el Portal de registro de aplicaciones.

1. Compruebe si las tres primeras letras de la contraseña coinciden con el campo **"MicrosoftAppPassword"** del archivo **appsettings.json**. 

1. Si los valores no coinciden, genere una nueva contraseña y almacene ese valor en el campo **"MicrosoftAppPassword"** del archivo **appsettings.json**.

# <a name="javascripttabjscreatebot"></a>[JavaScript](#tab/jscreatebot)

### <a name="create-web-app-bot-on-microsoft-azure"></a>Creación de una aplicación web en Microsoft Azure

Para crear el bot en Microsoft Azure: 

1. En Microsoft Azure Portal, haga clic en **Crear un recurso** y, a continuación, busque "Bot de aplicación web".

1. Haga clic en **Create**(Crear). Aparece la hoja Bot de aplicación web.

![Registro del bot de aplicación web](media/bot-builder-howto-direct-line/web-app-bot-registration.png)

1. En la hoja Bot de aplicación web, escriba el **nombre del bot**, la **suscripción**, el **grupo de recursos**, la **ubicación**, el **plan de tarifa** y el **nombre de la aplicación**.

1. Seleccione la plantilla de bot que quiere utilizar. **Versión de SDK: SDK v4** y **lenguaje de SDK: Node.js** 

1. Seleccione la plantilla básica de versión preliminar de v4. 

1. Elija el **plan de App Service/ubicación**, **Azure Storage** y la **ubicación de Application Insights**.

1. Seleccione la casilla Anclar al panel.

1. Haga clic en el botón Crear.

1. Espere a que se implemente el bot. Como seleccionó la casilla Anclar al panel, el bot aparecerá en el panel.

### <a name="edit-your-direct-line-bot"></a>Edición del bot de Direct Line

Ahora vamos a agregar algunas características más al bot de eco.

1. En la hoja Bot de aplicación web, haga clic en Crear. 

1. Haga clic en Descargar el archivo ZIP. 

1. Extraiga el archivo ZIP en un directorio local.

1. Vaya a la carpeta extraída y abra los archivos de origen en su IDE favorito.

1. En el archivo app.js, copie y pegue el código siguiente.

```javascript
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License.

const { BotFrameworkAdapter, MessageFactory, CardFactory } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

// Responds to the incoming message by either sending a hero card, an image, 
// or echoing the user's message.

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        if (context.activity.type === 'message') {
            const text = (context.activity.text || '').trim().toLowerCase()

            switch(text){
                case 'hi':
                case "hello":
                case "help":
                    // Send the user an instruction message.
                    return context.sendActivity("Welcome to the Bot to showcase the DirectLine API. " +
                    "Send \"Show me a hero card\" or \"Send me a BotFramework image\" to see how the " +
                    "DirectLine client supports custom channel data. Any other message will be echoed.");
                    break;

                case 'show me a hero card':
                    // Create the hero card.
                    const message = MessageFactory.attachment(
                        CardFactory.heroCard(   
                        'Sample Hero Card', //cards title
                        'Displayed in the DirectLine client' //cards text
                        )
                    );
                    return context.sendActivity(message);
                    break;
                    
                case 'send me a botframework image':
                    // Create the image attachment.
                    const imageOrVideoMessage = MessageFactory.contentUrl('https://docs.microsoft.com/en-us/azure/bot-service/media/how-it-works/architecture-resize.png', 'image/png')
                    return context.sendActivity(imageOrVideoMessage);
                    break;
                
                default:
                    // No command was encountered. Echo the user's message.
                    return context.sendActivity(`You said ${context.activity.text}`);
                    break;
                    
            }
        }
    });
});

```

Cuando esté listo, puede volver a publicar los orígenes en Azure. Siga estos pasos para obtener información sobre cómo volver a publicar el bot en [Azure](../bot-service-build-download-source-code.md)

---


## <a name="create-the-console-client-app"></a>Creación de la aplicación cliente de consola

# <a name="ctabcsclientapp"></a>[C#](#tab/csclientapp)

La aplicación cliente de consola opera en dos subprocesos. El subproceso principal acepta la entrada del usuario y envía mensajes al bot. El subproceso secundario sondea el bot una vez por segundo para recuperar los mensajes del bot y, a continuación, muestra los mensajes recibidos.

Para crear un proyecto de consola:

1. En el Explorador de soluciones, haga clic con el botón derecho en **Solution 'DirectLineBotSample'** (solución "DirectLineBotSample").

1. Elija **Agregar** > **Nuevo proyecto**.

1. En **Visual C#** > **Escritorio clásico de Windows*, elija **Aplicación de consola (.NET Framework)**.
 
    **Nota:** No elija **Aplicación de consola (.NET Core)**.

1. En el nombre, escriba **DirectLineClientSample**.

1. Haga clic en **OK**.

### <a name="add-the-nuget-packages-to-the-console-app"></a>Agregue los paquetes NuGet a la aplicación de consola.

1. Haga clic con el botón secundario en **Referencias**.

1. Haga clic en **Administración de paquetes de NuGet**.

1. Haga clic en **Examinar**. Asegúrese de que la casilla **Incluir versión preliminar** esté activada.

1. Busque e instale los siguientes paquetes NuGet:
    - Microsoft.Bot.Connector.DirectLine (v3.0.2)
    - Newtonsoft.Json

### <a name="edit-the-programcs-file"></a>Edición del archivo Program.cs

Reemplace el contenido del archivo **Program.cs** de DirectLineClientSample por el código siguiente:

```csharp
using System;
using System.Diagnostics;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Bot.Connector.DirectLine;
using Newtonsoft.Json;

namespace DirectLineClientSample
{
    class Program
    {
        // ************
        // Replace the following values with your Direct Line secret and the name of your bot resource ID.
        //*************
        private static string directLineSecret = "*** Replace with Direct Line secret ***";
        private static string botId = "*** Replace with the resource name of your bot ***";

        // This gives a name to the bot user.
        private static string fromUser = "DirectLineClientSampleUser";

        static void Main(string[] args)
        {
            StartBotConversation().Wait();
        }


        /// <summary>
        /// Drives the user's conversation with the bot.
        /// </summary>
        /// <returns></returns>
        private static async Task StartBotConversation()
        {
            // Create a new Direct Line client.
            DirectLineClient client = new DirectLineClient(directLineSecret);

            // Start the conversation.
            var conversation = await client.Conversations.StartConversationAsync();

            // Start the bot message reader in a separate thread.
            new System.Threading.Thread(async () => await ReadBotMessagesAsync(client, conversation.ConversationId)).Start();

            // Prompt the user to start talking to the bot.
            Console.Write("Type your message (or \"exit\" to end): ");

            // Loop until the user chooses to exit this loop.
            while (true)
            {
                // Accept the input from the user.
                string input = Console.ReadLine().Trim();

                // Check to see if the user wants to exit.
                if (input.ToLower() == "exit")
                {
                    // Exit the app if the user requests it.
                    break;
                }
                else
                {
                    if (input.Length > 0)
                    {
                        // Create a message activity with the text the user entered.
                        Activity userMessage = new Activity
                        {
                            From = new ChannelAccount(fromUser),
                            Text = input,
                            Type = ActivityTypes.Message
                        };

                        // Send the message activity to the bot.
                        await client.Conversations.PostActivityAsync(conversation.ConversationId, userMessage);
                    }
                }
            }
        }


        /// <summary>
        /// Polls the bot continuously and retrieves messages sent by the bot to the client.
        /// </summary>
        /// <param name="client">The Direct Line client.</param>
        /// <param name="conversationId">The conversation ID.</param>
        /// <returns></returns>
        private static async Task ReadBotMessagesAsync(DirectLineClient client, string conversationId)
        {
            string watermark = null;

            // Poll the bot for replies once per second.
            while (true)
            {
                // Retrieve the activity set from the bot.
                var activitySet = await client.Conversations.GetActivitiesAsync(conversationId, watermark);
                watermark = activitySet?.Watermark;

                // Extract the activies sent from our bot.
                var activities = from x in activitySet.Activities
                                 where x.From.Id == botId
                                 select x;

                // Analyze each activity in the activity set.
                foreach (Activity activity in activities)
                {
                    // Display the text of the activity.
                    Console.WriteLine(activity.Text);

                    // Are there any attachments?
                    if (activity.Attachments != null)
                    {
                        // Extract each attachment from the activity.
                        foreach (Attachment attachment in activity.Attachments)
                        {
                            switch (attachment.ContentType)
                            {
                                // Display a hero card.
                                case "application/vnd.microsoft.card.hero":
                                    RenderHeroCard(attachment);
                                    break;

                                // Display the image in a browser.
                                case "image/png":
                                    Console.WriteLine($"Opening the requested image '{attachment.ContentUrl}'");
                                    Process.Start(attachment.ContentUrl);
                                    break;
                            }
                        }
                    }

                    // Redisplay the user prompt.
                    Console.Write("\nType your message (\"exit\" to end): ");
                }

                // Wait for one second before polling the bot again.
                await Task.Delay(TimeSpan.FromSeconds(1)).ConfigureAwait(false);
            }
        }


        /// <summary>
        /// Displays the hero card on the console.
        /// </summary>
        /// <param name="attachment">The attachment that contains the hero card.</param>
        private static void RenderHeroCard(Attachment attachment)
        {
            const int Width = 70;
            // Function to center a string between asterisks.
            Func<string, string> contentLine = (content) => string.Format($"{{0, -{Width}}}", string.Format("{0," + ((Width + content.Length) / 2).ToString() + "}", content));

            // Extract the hero card data.
            var heroCard = JsonConvert.DeserializeObject<HeroCard>(attachment.Content.ToString());

            // Display the hero card.
            if (heroCard != null)
            {
                Console.WriteLine("/{0}", new string('*', Width + 1));
                Console.WriteLine("*{0}*", contentLine(heroCard.Title));
                Console.WriteLine("*{0}*", new string(' ', Width));
                Console.WriteLine("*{0}*", contentLine(heroCard.Text));
                Console.WriteLine("{0}/", new string('*', Width + 1));
            }
        }
    }
}
```

Para comprobar que todo está en orden, presione **F6** para compilar el proyecto. No debería haber advertencias ni errores.

# <a name="javascripttabjsclientapp"></a>[JavaScript](#tab/jsclientapp)

### <a name="create-a-direct-line-client"></a>Creación de un cliente de Direct Line 

Ahora que ha implementado el bot de aplicación web, podemos crear un cliente de Direct Line.

```
    md DirectLineClient
    cd DirectLineClient
    npm init
```

A continuación, instale estos paquetes de npm.

```
    npm install --save open
    npm install --save request
    npm install --save request-promise
    npm install --save swagger-client
```

Cree un archivo **app.js**. Copie y pegue el código siguiente en este archivo.

Se obtendrá directLineSecret en el paso siguiente.
```javascript
var Swagger = require('swagger-client');
var open = require('open');
var rp = require('request-promise');

// config items
var pollInterval = 1000;
// Change the Direct Line Secret to your own 
var directLineSecret = 'your secret here';
var directLineClientName = 'DirectLineClient';
var directLineSpecUrl = 'https://docs.botframework.com/en-us/restapi/directline3/swagger.json';

var directLineClient = rp(directLineSpecUrl)
    .then(function (spec) {
        // client
        return new Swagger({
            spec: JSON.parse(spec.trim()),
            usePromise: true
        });
    })
    .then(function (client) {
        // add authorization header to client
        client.clientAuthorizations.add('AuthorizationBotConnector', new Swagger.ApiKeyAuthorization('Authorization', 'Bearer ' + directLineSecret, 'header'));
        return client;
    })
    .catch(function (err) {
        console.error('Error initializing DirectLine client', err);
    });

// once the client is ready, create a new conversation
directLineClient.then(function (client) {
    client.Conversations.Conversations_StartConversation()                          // create conversation
        .then(function (response) {
            return response.obj.conversationId;
        })                            // obtain id
        .then(function (conversationId) {
            sendMessagesFromConsole(client, conversationId);                        // start watching console input for sending new messages to bot
            pollMessages(client, conversationId);                                   // start polling messages from bot
        })
        .catch(function (err) {
            console.error('Error starting conversation', err);
        });
});

// Read from console (stdin) and send input to conversation using DirectLine client
function sendMessagesFromConsole(client, conversationId) {
    var stdin = process.openStdin();
    process.stdout.write('Command> ');
    stdin.addListener('data', function (e) {
        var input = e.toString().trim();
        if (input) {
            // exit
            if (input.toLowerCase() === 'exit') {
                return process.exit();
            }

            // send message
            client.Conversations.Conversations_PostActivity(
                {
                    conversationId: conversationId,
                    activity: {
                        textFormat: 'plain',
                        text: input,
                        type: 'message',
                        from: {
                            id: directLineClientName,
                            name: directLineClientName
                        }
                    }
                }).catch(function (err) {
                    console.error('Error sending message:', err);
                });

            process.stdout.write('Command> ');
        }
    });
}

// Poll Messages from conversation using DirectLine client
function pollMessages(client, conversationId) {
    console.log('Starting polling message for conversationId: ' + conversationId);
    var watermark = null;
    setInterval(function () {
        client.Conversations.Conversations_GetActivities({ conversationId: conversationId, watermark: watermark })
            .then(function (response) {
                watermark = response.obj.watermark;                                 // use watermark so subsequent requests skip old messages
                return response.obj.activities;
            })
            .then(printMessages);
    }, pollInterval);
}

// Helpers methods
function printMessages(activities) {
    if (activities && activities.length) {
        // ignore own messages
        activities = activities.filter(function (m) { return m.from.id !== directLineClientName });

        if (activities.length) {
            process.stdout.clearLine();
            process.stdout.cursorTo(0);

            // print other messages
            activities.forEach(printMessage);

            process.stdout.write('Command> ');
        }
    }
}

function printMessage(activity) {
    if (activity.text) {
        console.log(activity.text);
    }

    if (activity.attachments) {
        activity.attachments.forEach(function (attachment) {
            switch (attachment.contentType) {
                case "application/vnd.microsoft.card.hero":
                    renderHeroCard(attachment);
                    break;

                case "image/png":
                    console.log('Opening the requested image ' + attachment.contentUrl);
                    open(attachment.contentUrl);
                    break;
            }
        });
    }
}

function renderHeroCard(attachment) {
    var width = 70;
    var contentLine = function (content) {
        return ' '.repeat((width - content.length) / 2) +
            content +
            ' '.repeat((width - content.length) / 2);
    }

    console.log('/' + '*'.repeat(width + 1));
    console.log('*' + contentLine(attachment.content.title) + '*');
    console.log('*' + ' '.repeat(width) + '*');
    console.log('*' + contentLine(attachment.content.text) + '*');
    console.log('*'.repeat(width + 1) + '/');
}
```

---

## <a name="configure-the-direct-line-channel"></a>Configuración del canal de Direct Line

La adición del canal de Direct Line transforma este bot en un bot de Direct Line.

Para configurar el canal de Direct Line:

![Hoja Conexión a los canales con el botón de canal de Direct Line resaltado.](media/bot-builder-howto-direct-line/direct-line-channel-button.png)

1. En la hoja **Bot Channels Registration** (Registro de canales de bot), haga clic en **Canales**. Aparecerá la hoja **Conexión a los canales**.

    ![Hoja Configure Direct Line (Configuración de Direct Line) que muestra ambas claves secretas.](media/bot-builder-howto-direct-line/configure-direct-line.png)

1. En la hoja **Conexión a los canales**, haga clic en el botón **Configure Direct Line channel** (Configurar canal de Direct Line). 

1. Si la opción **3.0** no está activada, coloque una marca de verificación en la casilla de verificación.

1. Haga clic en **Mostrar** para una clave secreta por lo menos.

1. Copie una de las claves secretas y péguela en la aplicación cliente, en la cadena **directLineSecret**.

1. Haga clic en Done (Listo).

## <a name="run-the-client-app"></a>Ejecución de la aplicación cliente

# <a name="ctabcsrunclient"></a>[C#](#tab/csrunclient)

El bot ya está listo para comunicarse con la aplicación cliente de consola de Direct Line. Para ejecutar la aplicación de consola, haga lo siguiente:

1. En Visual Studio, haga clic con el botón derecho en el proyecto **DirectLineClientSample** y seleccione **Set as StartUp Project**. (Establecer como proyecto de inicio).

1. Examine el archivo **Program.cs** en el proyecto **DirectLineClientSample**.

    - Compruebe que el código secreto de Direct Line está en la cadena **directLineSecret**.

1. Si **directLineSecret** no es correcto, vaya a Azure Portal, haga clic en el bot de Direct Line, en **Canales**, luego en **Configure Direct Line channel** (Configurar canal de Direct Line) (o **Editar**), mostrar una clave y luego cópiela en la cadena **directLineSecret**.

    - Compruebe que el grupo de recursos está en la cadena **botId**.

1. Si no lo está, desde la hoja **Introducción**, copie y pegue el **grupo de recursos** en la cadena **botId**.

1. Pulse F5 para iniciar la depuración.

Se iniciará la aplicación cliente de consola. Para probar la aplicación: 

1. Escriba "hola". El bot debería mostrar "Ha dicho 'Hola'".

1. Escriba "Mostrarme una tarjeta de imagen prominente". El bot debería mostrar una tarjeta de imagen prominente.

1. Escriba "Enviarme una imagen de botframework". El bot iniciará un explorador para mostrar una imagen de la documentación de Bot Framework.

1. Escriba cualquier otra cosa y el bot debería responder con "Ha dicho" y el mensaje entre comillas.

# <a name="javascripttabjsrunclient"></a>[JavaScript](#tab/jsrunclient)

Para ejecutar el ejemplo, deberá ejecutar la aplicación DirectLineClient.

1. Abra una consola CMD y CD en el directorio DirectLineClient.

1. Ejecute `node app.js`

Para probar los tipos de mensajes personalizados en la consola del cliente, muéstreme una tarjeta de imagen prominente o envíeme una imagen de botframework y verá el resultado siguiente.

![resultado](media/bot-builder-howto-direct-line/outcome.png)

---


### <a name="next-steps"></a>Pasos siguientes

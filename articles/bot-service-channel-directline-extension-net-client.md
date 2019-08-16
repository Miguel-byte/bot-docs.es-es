---
title: Creación de un cliente de .NET para conectarse a la extensión de App Service para Direct Line
titleSuffix: Bot Service
description: Cliente de .NET en C# para conectarse a la extensión de App Service para Direct Line
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 07/25/2019
ms.openlocfilehash: 62fda46569c6134f798b4d253a0676a037fdfa0f
ms.sourcegitcommit: 6a83b2c8ab2902121e8ee9531a7aa2d85b827396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/09/2019
ms.locfileid: "68866456"
---
# <a name="create-net-client-to-connect-to-direct-line-app-service-extension"></a>Creación de un cliente de .NET para conectarse a la extensión de App Service para Direct Line

En este artículo se describe cómo crear un cliente de .NET en C# que se conecta a la extensión de App Service para Direct Line.

## <a name="gather-your-direct-line-extension-keys"></a>Recopilación de las claves de la extensión Direct Line

1. En el explorador, vaya a [Azure Portal](https://portal.azure.com/).
1. En Azure Portal, busque el recurso de **Azure Bot Service**.
1. Haga clic en **Canales** para configurar los canales del bot.
1. Si aún no está habilitado, haga clic en el canal de **Direct Line** para habilitarlo. 
1. Si ya está habilitado, en la tabla Conectar a canales, haga clic en el vínculo **Editar** en la fila de Direct Line.
1. Desplácese a la sección Sitios. Normalmente, hay un Sitio predeterminado a menos que se haya eliminado o cambiado de nombre.
1. Haga clic en el vínculo **Mostrar** para mostrar una de las claves y, a continuación, copie su valor.

    ![Claves de la extensión de App Service](./media/channels/direct-line-extension-extension-keys-net-client.png)

> [!NOTE]
> Este valor es el secreto de cliente de Direct Line que se usa para conectarse a la extensión de App Service para Direct Line. Puede crear sitios adicionales si lo desea y usar también esos valores de secreto.

## <a name="add-the-preview-nuget-package-source"></a>Adición del origen del paquete de Nuget de la versión preliminar

Los paquetes de NuGet de la versión preliminar necesarios para crear un cliente de Direct Line en C# se pueden encontrar en una fuente de NuGet.

1. En Visual Studio, vaya al elemento de menú **Herramientas->Opciones**.
1. Seleccione el elemento **Administrador de paquetes de NuGet->Orígenes del paquete**.
1. Haga clic en el botón + para agregar un nuevo origen del paquete con estos valores:
    - Nombre: DL ASE Preview
    - Origen: https://botbuilder.myget.org/F/experimental/api/v3/index.json
1. Haga clic en el botón **Actualizar** para guardar los valores.
1. Haga clic en **Aceptar** para salir de la configuración de orígenes del paquete.

## <a name="create-a-c-direct-line-client"></a>Creación de un cliente de Direct Line en C#

Las interacciones con la extensión de App Service para Direct Line se producen de manera diferente al modo tradicional de Direct Line porque la mayoría de las comunicaciones se producen mediante un *WebSocket*. El cliente de Direct Line actualizado incluye clases auxiliares para abrir y cerrar un *WebSocket*, enviar comandos a través del WebSocket y recibir las actividades del bot. En esta sección se describe cómo crear un cliente simple en C# para interactuar con un bot.

1. Cree un nuevo proyecto de aplicación de consola de .NET Core 2.2 en Visual Studio.
1. Agregue el **cliente de Direct Line de NuGet** al proyecto.
    - Haga clic en Dependencias en el árbol de soluciones.
    - Seleccione **Administrar paquetes de NuGet**.
    - Cambie el origen del paquete por el que ha definido anteriormente (DL ASE Preview).
    - Busque el paquete *Microsoft.Bot.Connector.Directline* versión v3.0.3-Preview1 o posterior.
    - Haga clic en **Instalar paquete**.
1. Cree un cliente y genere un token con un secreto. Este paso equivale a crear cualquier otro cliente de Direct Line en C# excepto por el punto de conexión que necesita usar en el bot, anexado con la ruta de acceso **.bot/** , como se muestra a continuación. No olvide la barra **/** final.

    ```csharp
    string endpoint = "https://<YOUR_BOT_HOST>.azurewebsites.net/.bot/";
    string secret = "<YOUR_BOT_SECRET>";

    var tokenClient = new DirectLineClient(
        new Uri(endpoint),
        new DirectLineClientCredentials(secret));
    var conversation = await tokenClient.Tokens.GenerateTokenForNewConversationAsync();
    ```

1. Una vez que tenga una referencia de conversación de la generación de un token, puede usar este identificador de conversación para abrir un WebSocket con la nueva propiedad `StreamingConversations` en `DirectLineClient`. Para ello, debe crear una devolución de llamada que se invocará cuando el bot quiera enviar `ActivitySets` al cliente:

    ```csharp
    public static void ReceiveActivities(ActivitySet activitySet)
    {
        if (activitySet != null)
        {
            foreach (var a in activitySet.Activities)
            {
                if (a.Type == ActivityTypes.Message && a.From.Id.Contains("bot"))
                {
                    Console.WriteLine($"<Bot>: {a.Text}");
                }
            }
        }
    }
    ```

1. Ahora está listo para abrir el WebSocket en la propiedad `StreamingConversations` mediante el token de la conversación, `conversationId` y la devolución de llamada `ReceiveActivities`:

    ```csharp
    var client = new DirectLineClient(
        new Uri(endpoint),
        new DirectLineClientCredentials(conversation.Token));

    await client.StreamingConversations.ConnectAsync(
        conversation.ConversationId,
        ReceiveActivities);
    ```

1. Ahora se puede usar el cliente para iniciar una conversación y enviar `Activities` al bot:

    ```csharp

    var startConversation = await client.StreamingConversations.StartConversationAsync();
    var from = new ChannelAccount() { Id = "123", Name = "Fred" };
    var message = Console.ReadLine();

    while (message != "end")
    {
        try
        {
            var response = await client.StreamingConversations.PostActivityAsync(
                startConversation.ConversationId,
                new Activity()
                {
                    Type = "message",
                    Text = message,
                    From = from
                });
        }
        catch (OperationException ex)
        {
            Console.WriteLine(
                $"OperationException when calling PostActivityAsync: ({ex.StatusCode})");
        }
        message = Console.ReadLine();
    }
    ```

---
title: Administración de datos de estado personalizado con Azure Cosmos DB | Microsoft Docs
description: Aprenda a guardar y a recuperar datos de estado mediante Azure Cosmos DB con Bot Framework SDK para .NET
author: kaiqb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 19b44fbb09af69bde803b9b884d58108c248ebe5
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224840"
---
# <a name="manage-custom-state-data-with-azure-cosmos-db-for-net"></a>Administración de datos de estado personalizado con Azure Cosmos DB para Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

En este artículo, implementará Azure Cosmos DB para almacenar y administrar los datos de estado del bot. El servicio de estado de Connector predeterminado que usan los bots no es adecuado para el entorno de producción. Deberá usar las [extensiones de Azure](https://github.com/Microsoft/BotBuilder-Azure) disponibles en GitHub o implementar un cliente de estado personalizado mediante la plataforma de almacenamiento de datos de su elección. Estas son algunas de las razones para usar el almacenamiento de estado personalizado:
 - Mayor rendimiento de la API de estado (más control sobre el rendimiento)
 - Latencia más baja para la distribución geográfica
 - Control sobre dónde se almacenan los datos
 - Acceso a los datos de estado reales
 - Almacenamiento de más de 32 kb de datos.
 
## <a name="prerequisites"></a>Requisitos previos
Necesitará:
 - [Cuenta de Microsoft Azure](https://azure.microsoft.com/en-us/free/)
 - [Visual Studio 2015 o posterior](https://www.visualstudio.com/)
 - [Paquete NuGet de Azure para Bot Builder](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/)
 - [Paquete NuGet Autofac Web Api2](https://www.nuget.org/packages/Autofac.WebApi2/)
 - [Bot Framework Emulator](~/bot-service-debug-emulator.md)
 
## <a name="create-azure-account"></a>Creación de una cuenta de Azure
Si no tiene una cuenta de Azure, haga clic [aquí](https://azure.microsoft.com/en-us/free/) para registrarse y obtener una gratuita.

## <a name="set-up-the-azure-cosmos-db-database"></a>Configuración de la base de datos de Azure Cosmos DB
1. Una vez que ha iniciado sesión en Azure Portal, haga clic en *Nuevo* para crear una base de datos de **Azure Cosmos DB**. 
2. Haga clic en **Bases de datos**. 
3. Busque **Azure Cosmos DB** y haga clic en **Crear**.
4. Rellene los campos. Para el campo **API**, seleccione **SQL (DocumentDB)**. Una vez rellenados todos los campos, haga clic en el botón **Crear** en la parte inferior de la pantalla para implementar la nueva base de datos. 
5. Tras la implementación de la nueva base de datos, vaya a la base de datos. Haga clic en **Claves de acceso** para encontrar las claves y las cadenas de conexión. El bot usará esta información para llamar al servicio de almacenamiento para guardar los datos de estado.

## <a name="install-nuget-packages"></a>Instalación de paquetes NuGet
1. Abra un proyecto de bot de C# existente o cree uno con la plantilla Bot en Visual Studio. 
2. Instale los siguientes paquetes NuGet:
   - Microsoft.Bot.Builder.Azure
   - Autofac.WebApi2

## <a name="add-connection-string"></a>Agregar cadena de conexión 
Agregue las siguientes entradas al archivo Web.config:
```XML
<add key="DocumentDbUrl" value="Your DocumentDB URI"/>
<add key="DocumentDbKey" value="Your DocumentDB Key"/>
```
Reemplazará el valor por el URI y la clave principal encontrados en su instancia de Azure Cosmos DB. Guarde el archivo Web.config.

## <a name="modify-your-bot-code"></a>Modificación del código de bot
Para usar el almacenamiento de **Azure Cosmos DB**, agregue las siguientes líneas de código al archivo **Global.asax.cs** de su bot dentro del método **Application_Start ()**.

```cs
using System;
using Autofac;
using System.Web.Http;
using System.Configuration;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;

namespace SampleApp
{
    public class WebApiApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            GlobalConfiguration.Configure(WebApiConfig.Register);
            var uri = new Uri(ConfigurationManager.AppSettings["DocumentDbUrl"]);
            var key = ConfigurationManager.AppSettings["DocumentDbKey"];
            var store = new DocumentDbBotDataStore(uri, key);

            Conversation.UpdateContainer(
                        builder =>
                        {
                            builder.Register(c => store)
                                .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
                                .AsSelf()
                                .SingleInstance();

                            builder.Register(c => new CachingBotDataStore(store, CachingBotDataStoreConsistencyPolicy.ETagBasedConsistency))
                                .As<IBotDataStore<BotData>>()
                                .AsSelf()
                                .InstancePerLifetimeScope();

                        });

        }
    }
}
```

Guarde el archivo Global.asax.cs. Ahora está listo para probar el bot con el emulador.

## <a name="run-your-bot-app"></a>Ejecución de la aplicación bot
Ejecute el bot en Visual Studio; el código que agregó creará la tabla **botdata** personalizada en Azure.

## <a name="connect-your-bot-to-the-emulator"></a>Conexión del bot al emulador
En este momento, el bot se ejecuta de forma local. A continuación, inicie el emulador y, después, conéctese al bot en el emulador:
1. Escriba http://localhost:port-number/api/messages en la barra de dirección, donde port-number coincide con el número de puerto mostrado en el explorador donde se ejecuta la aplicación. Por ahora puede dejar en blanco los campos <strong>Microsoft App ID</strong> (Id. de aplicación de Microsoft) y <strong>Microsoft App Password</strong> (Contraseña de aplicación de Microsoft). Más adelante obtendrá esta información al [registrar el bot](~/bot-service-quickstart-registration.md).
2. Haga clic en **Conectar**. 
3. Para probar su bot, escriba unos cuantos mensajes en el emulador. 

## <a name="view-state-data-on-azure-portal"></a>Vista de los datos de estado en Azure Portal
Para ver los datos de estado, inicie sesión en Azure Portal y vaya a la base de datos. Haga clic en **Explorador de datos (versión preliminar)** para verificar que se está guardando la información de estado del bot. 

## <a name="next-steps"></a>Pasos siguientes
En este artículo, ha usado Cosmos DB para guardar y administrar datos del bot. A continuación, aprenda cómo modelar el flujo de la conversación con diálogos.

> [!div class="nextstepaction"]
> [Administración del flujo de conversación](bot-builder-dotnet-manage-conversation-flow.md)

## <a name="additional-resources"></a>Recursos adicionales
Si no está familiarizado con los contenedores de inversión de control y el patrón de inserción de dependencias usados en el código anterior, visite el sitio de [Autofac](http://autofac.readthedocs.io/en/latest/) para obtener información. 

También puede descargar un [ejemplo](https://github.com/Microsoft/BotBuilder-Azure/tree/master/CSharp/Samples/DocumentDb) de GitHub para saber más sobre el uso de Cosmos DB para administrar el estado. 

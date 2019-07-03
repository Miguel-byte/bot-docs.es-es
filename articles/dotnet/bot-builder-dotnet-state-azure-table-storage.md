---
title: Administración de datos de estado personalizado con Azure Table Storage | Microsoft Docs
description: Aprenda a guardar y a recuperar datos de estado mediante Azure Table Storage con Bot Framework SDK para .NET
author: kaiqb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 05ab8f9148cd520be266d9340cd65a9903bf6e23
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405620"
---
# <a name="manage-custom-state-data-with-azure-table-storage-for-net"></a>Administración de datos de estado personalizado con Azure Table Storage para .NET

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

En este artículo, implementará Azure Table Storage para almacenar y administrar los datos de estado del bot. El servicio de estado de Connector predeterminado que usan los bots no es adecuado para el entorno de producción. Deberá usar las [extensiones de Azure](https://github.com/Microsoft/BotBuilder-Azure) disponibles en GitHub o implementar un cliente de estado personalizado mediante la plataforma de almacenamiento de datos de su elección. Estas son algunas de las razones para usar el almacenamiento de estado personalizado:
 - Mayor rendimiento de la API de estado (más control sobre el rendimiento)
 - Latencia más baja para la distribución geográfica
 - Control sobre dónde se almacenan los datos
 - Acceso a los datos de estado reales
 - Almacenamiento de más de 32 kb de datos.

## <a name="prerequisites"></a>Requisitos previos
Necesitará:
 - [Cuenta de Microsoft Azure](https://azure.microsoft.com/free/)
 - [Visual Studio 2015 o posterior](https://www.visualstudio.com/)
 - [Paquete NuGet de Azure para Bot Builder](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/)
 - [Paquete NuGet Autofac Web Api2](https://www.nuget.org/packages/Autofac.WebApi2/)
 - [Bot Framework Emulator](https://emulator.botframework.com/)
 - [Explorador de Azure Storage](http://storageexplorer.com/)
 
## <a name="create-azure-account"></a>Creación de una cuenta de Azure
Si no tiene una cuenta de Azure, haga clic [aquí](https://azure.microsoft.com/free/) para registrarse y obtener una gratuita.

## <a name="set-up-the-azure-table-storage-service"></a>Configuración del servicio de Azure Table Storage
1. Una vez que ha iniciado sesión en Azure Portal, haga clic en **Nuevo** para crear un servicio de Azure Table Storage. 
2. Busque la **cuenta de almacenamiento** que implementa la tabla de Azure. 
3. Rellene los campos, haga clic en el botón **Crear** en la parte inferior de la pantalla para implementar el nuevo servicio de almacenamiento. Una vez implementado el nuevo servicio de almacenamiento, mostrará las características y opciones que tiene a su disposición.
4. Seleccione la pestaña **Claves de acceso** a la izquierda y copie la cadena de conexión para su uso posterior. El bot usará esta cadena de conexión para llamar al servicio de almacenamiento para guardar los datos de estado.

## <a name="install-nuget-packages"></a>Instalación de paquetes NuGet
1. Abra un proyecto de bot de C# existente o cree uno con la plantilla Bot de C# en Visual Studio. 
2. Instale los siguientes paquetes NuGet:
   - Microsoft.Bot.Builder.Azure
   - Autofac.WebApi2

## <a name="add-connection-string"></a>Agregar cadena de conexión 
Agregue la siguiente entrada en el archivo Web.config: 
```XML
  <connectionStrings>
    <add name="StorageConnectionString"
    connectionString="YourConnectionString"/>
  </connectionStrings>
```
Reemplace "YourConnectionString" con la cadena de conexión a Azure Table Storage que guardó anteriormente. Guarde el archivo Web.config.

## <a name="modify-your-bot-code"></a>Modificación del código del bot
En el archivo Global.asax.cs, agregue las siguientes instrucciones `using`:
```cs
using Autofac;
using System.Configuration;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
```
En el método `Application_Start()`, cree una instancia de la clase `TableBotDataStore`. La clase `TableBotDataStore` implementa la interfaz `IBotDataStore<BotData>`. La interfaz `IBotDataStore` le permite invalidar la conexión predeterminada al servicio de estado de Connector.
 ```cs
 var store = new TableBotDataStore(ConfigurationManager.ConnectionStrings["StorageConnectionString"].ConnectionString);
 ```
Registre el servicio como se muestra a continuación:
 ```cs
 Conversation.UpdateContainer(
            builder =>
            {
                builder.Register(c => store)
                          .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
                          .AsSelf()
                          .SingleInstance();

                builder.Register(c => new CachingBotDataStore(store,
                           CachingBotDataStoreConsistencyPolicy
                           .ETagBasedConsistency))
                           .As<IBotDataStore<BotData>>()
                           .AsSelf()
                           .InstancePerLifetimeScope();

                
            });
 ```
Guarde el archivo Global.asax.cs.

## <a name="run-your-bot-app"></a>Ejecución de la aplicación bot
Ejecute el bot en Visual Studio; el código que agregó creará la tabla **botdata** personalizada en Azure.

## <a name="connect-your-bot-to-the-emulator"></a>Conexión del bot al emulador
En este momento, el bot se ejecuta de forma local. A continuación, inicie el emulador y, después, conéctese al bot en el emulador:
1. Escriba http://localhost:port-number/api/messages en la barra de dirección, donde port-number coincide con el número de puerto mostrado en el explorador donde se ejecuta la aplicación. Por ahora puede dejar en blanco los campos <strong>Microsoft App ID</strong> (Id. de aplicación de Microsoft) y <strong>Microsoft App Password</strong> (Contraseña de aplicación de Microsoft). Más adelante obtendrá esta información al [registrar el bot](~/bot-service-quickstart-registration.md).
2. Haga clic en **Conectar**. 
3. Para probar su bot, escriba unos cuantos mensajes en el emulador. 

## <a name="view-data-in-azure-table-storage"></a>Visualización de datos en Azure Table Storage
Para ver los datos de estado, abra el **Explorador de Storage** y conéctese a Azure con sus credenciales de Azure Portal o conéctese directamente a la tabla mediante el nombre y la clave de almacenamiento y después vaya al nombre de la tabla.  

## <a name="next-steps"></a>Pasos siguientes
En este artículo, implementó Azure Table Storage para guardar y administrar los datos del bot. A continuación, obtenga información sobre cómo modelar el flujo de la conversación con diálogos.

> [!div class="nextstepaction"]
> [Administración del flujo de conversación](bot-builder-dotnet-manage-conversation-flow.md)


## <a name="additional-resources"></a>Recursos adicionales

Si no está familiarizado con los contenedores de inversión de control y el patrón de inserción de dependencias usados en el código anterior, vaya al sitio de [Autofac](http://autofac.readthedocs.io/en/latest/) para obtener información. 

También puede descargar un ejemplo completo de [Azure Table Storage](https://github.com/Microsoft/BotBuilder-Azure/tree/master/CSharp/Samples/AzureTable) de GitHub.

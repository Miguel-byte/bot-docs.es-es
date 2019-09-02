---
title: Uso del estado de usuario de .NET v3 en un bot de la versión v4 | Microsoft Docs
description: Cómo usar el estado de usuario de la versión v3 en un ejemplo de un bot de la versión v4
keywords: Csharp, bot migration, v3 bot
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/21/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1bae27b9cf2bdc6a53a5c55fe7458802729160c1
ms.sourcegitcommit: 008aa6223aef800c3abccda9a7f72684959ce5e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2019
ms.locfileid: "70026704"
---
# <a name="using-net-v3-user-state-in-a-v4-bot"></a>Uso del estado de usuario de .NET v3 en un bot de la versión v4

En este artículo se muestra un ejemplo de cómo un bot de la versión v4 puede realizar operaciones de lectura, escritura y eliminación en la información de un estado de usuario de la versión v3.
El bot mantiene un estado de conversación mediante `MemoryStorage` para realizar un seguimiento de la conversación y dirigirla, al tiempo que se formulan preguntas al usuario.  Mantiene el **estado de usuario** en el formato de la versión v3 para realizar el seguimiento de las respuestas del usuario mediante una clase `IStorage` personalizada denominada `V3V4Storage`.  Uno de los argumentos de esta clase es `IBotDataStore`. La base de código del SDK de la versión v3 se copió en `Bot.Builder.Azure.V3V4` y contiene los tres proveedores de almacenamiento de esta versión del SDK (Azure SQL, Azure Table y Cosmos DB).  El objetivo es permitir que el **estado de usuario** existente de la versión v3 se introduzca en un bot migrado de la versión v4.

El ejemplo de código se puede encontrar [aquí](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/CSharp/V4StateBotFromV3Providers).

## <a name="prerequisites"></a>Requisitos previos

- [.NET Core SDK](https://dotnet.microsoft.com/download) versión 2.1

    ```bash
    # determine dotnet version
    dotnet --version
    ```

## <a name="setup"></a>Configuración

1. Clonación del repositorio

    ```bash
    git clone https://github.com/microsoft/botbuilder-samples.git
    ```

1. En un terminal, vaya a `MigrationV3V4/CSharp/V4StateBotFromV3Providers`

    ```bash
    cd BotBuilder-Samples/MigrationV3V4/CSharp/V4StateBotFromV3Providers
    ```

1. Ejecute el bot desde un terminal o desde Visual Studio, elija la opción A o B.

    - Desde un terminal

        ```bash
        # run the bot
        dotnet run
        ```

    - O desde Visual Studio

        - Inicie Visual Studio, seleccione Archivo -> Abrir -> Proyecto o Solución.
        - Vaya a la carpeta `BotBuilder-Samples/MigrationV3V4/CSharp/V4StateBotFromV3Providers`.
        - Seleccione el archivo `V4StateBot.sln`.
        - Presione F5 para ejecutar el proyecto


## <a name="storage-provider-setup"></a>Configuración del proveedor de almacenamiento

Se supone que tiene un almacén de estados existente de la versión v3 configurado y en uso. En este caso, la configuración de este ejemplo para que use el almacén de estados existente consiste simplemente en agregar la información de conexión del proveedor de almacenamiento al archivo `web.config`, tal y como se muestra a continuación.

- Cosmos DB

```json
  "v3CosmosEndpoint": "https://yourcosmosdb.documents.azure.com:443/",
  "v3CosmosKey": "YourCosmosDbKey",
  "v3CosmosDatataseName": "v3botdb",
  "v3CosmosCollectionName": "v3botcollection",
```

- Azure SQL

```json
 "ConnectionStrings": {
    "SqlBotData": "Server=YourServer;Initial Catalog=BotData;Persist Security Info=False;User ID=YourUserName;Password=YourUserPassword;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=True;Connection Timeout=30;"
  },
```

- tabla de Azure

```json
 "ConnectionStrings": {
    "AzureTable": "DefaultEndpointsProtocol=https;AccountName=YourAccountName;AccountKey=YourAccountKey;EndpointSuffix=core.windows.net"
  },
```

- Establecimiento del proveedor de almacenamiento del bot

    Abra el archivo `Startup.cs` en la raíz del proyecto `V4V3StateBot`. Hacia la mitad del archivo (líneas 52-76 aproximadamente) verá las configuraciones para cada proveedor de almacenamiento. Estos leen los valores de configuración del archivo web.config. 

    [!code-csharp[Storage configuration](~/../botbuilder-samples/MigrationV3V4/CSharp/V4StateBotFromV3Providers/V4V3StateBot/Startup.cs?range=52-76)]

    Especifique el proveedor de almacenamiento que desea que use el bot quitando la marca de comentario de las líneas correspondientes de la instancia de su elección. Una vez que el proveedor esté configurado correctamente, asegúrese de que la clase de proveedor se pasa a `V3V4Storage` (líneas 72-75 aproximadamente). 

    [!code-csharp[Storage provider](~/../botbuilder-samples/MigrationV3V4/CSharp/V4StateBotFromV3Providers/V4V3StateBot/Startup.cs?range=72-75)]

    De forma predeterminada, está establecido Cosmos DB (anteriormente denominado Document DB). Los valores posibles son:

    ```bash
    documentDbBotDataStore
    tableBotDataStore
    tableBotDataStore2
    sqlBotDataStore
    ```

- Inicie la aplicación. 

## <a name="v3v4-storage-and-state-classes"></a>Clases de estados y almacenamiento en las versiones v3 y v4

### <a name="v3v4storage"></a>V3V4Storage

La clase `V3V4Storage` contiene la funcionalidad principal de asignación de almacenamiento. Implementa la interfaz `IStorage` de la versión v4 y asigna los métodos del proveedor de almacenamiento (lectura, escritura y eliminación) de nuevo a las clases del proveedor de almacenamiento de la versión v3 para que el estado del usuario con formato v3 se pueda usar en un bot de la versión v4.

### <a name="v3v4state"></a>V3V4State

Esta clase se hereda de la clase `BotState` de la versión v4 y utiliza una clave del tipo v3 (`IAddress`). Esto permite lecturas, escrituras y eliminaciones en el almacenamiento de la versión v3 de la misma manera que ha funcionado siempre el almacenamiento de estados en esta versión.


## <a name="testing-the-bot-using-bot-framework-emulator"></a>Prueba del bot con Bot Framework Emulator

[Bot Framework Emulator][5] es una aplicación de escritorio que permite que los desarrolladores de bots prueben y depuren sus bots en localhost o mediante la ejecución remota mediante un túnel.

- Instale Bot Framework Emulator, versión 4.3.0 o posterior desde [aquí][6].


### <a name="connect-to-the-bot-using-bot-framework-emulator"></a>Conexión al bot con Bot Framework Emulator

- Inicie Bot Framework Emulator.
- Archivo -> Open Bot (Abrir bot)
- Escriba la dirección URL de un bot: `http://localhost:3978/api/messages`


## <a name="further-reading"></a>Lecturas adicionales

- [Introducción a Azure Bot Service][21]
- [Bot State][7]
- [Escritura directa en el almacenamiento][8]
- [Administración del estado de la conversación y del usuario][9]

[3]: https://aka.ms/botframework-emulator
[5]: https://github.com/microsoft/botframework-emulator
[6]: https://github.com/Microsoft/BotFramework-Emulator/releases
[7]: https://docs.microsoft.com/azure/bot-service/bot-builder-storage-concept
[8]: https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-storage?tabs=csharp
[9]: https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-state?tabs=csharp
[21]: https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0
[40]: https://aka.ms/azuredeployment

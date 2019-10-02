---
title: Uso del estado de usuario de JavaScript v3 en un bot de la versión v4 | Microsoft Docs
description: Cómo usar el estado de usuario de la versión v3 en un ejemplo de un bot de la versión v4
keywords: JavaScript, bot migration, v3 bot
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/14/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4697ecf47464114de68ec6c0d872b45ff1ee5e54
ms.sourcegitcommit: d493caf74b87b790c99bcdaddb30682251e3fdd4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/26/2019
ms.locfileid: "71278980"
---
<!-- This article is on hold -->

# <a name="using-javascript-v3-user-state-in-a-v4-bot"></a>Uso del estado de usuario de JavaScript v3 en un bot de la versión v4

En este artículo se muestra un ejemplo de cómo un bot de la versión v4 puede realizar operaciones de lectura, escritura y eliminación en la información de un estado de usuario de la versión v3.

El ejemplo de código se puede encontrar [aquí](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot).

> [!NOTE]
> Un bot mantiene un **estado de conversación** para realizar un seguimiento de la conversación y dirigirla, y formular preguntas al usuario. Mantiene el **estado del usuario** para realizar el seguimiento de las respuestas del usuario.

## <a name="prerequisites"></a>Requisitos previos

- npm, versión 6.9.0 o posterior (necesaria para admitir el alias de paquetes).

- Node.js, versión 10.14.1 o posterior.

    Para comprobar la versión instalada en el equipo, en un terminal, ejecute el comando siguiente:

    ```bash
    # determine node version
    node --version
    ```

## <a name="setup"></a>Configuración

1. Clonación del repositorio

    ```bash
    git clone https://github.com/microsoft/botbuilder-samples.git
    ```

1. En un terminal, vaya a `BotBuilder-Samples/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot`

    ```bash
    cd BotBuilder-Samples/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot
    ```

1. Ejecute `npm install` en las siguientes ubicaciones:

    ```bash
    root
    /V4V3StorageMapper
    /V4V3UserState
    ```

1. Ejecute ``npm run build`` o ``tsc`` para compilar los módulos `StorageMapper` y `UserState` en las siguientes ubicaciones:

    ```bash
    /V4V3StorageMapper
    /V4V3UserState
    ```

1. Configuración de la base de datos

    1. Copie el contenido del archivo `.env.example`.
    1. Cree un nuevo archivo denominado `.env` y pegue el contenido anterior. 
    1. Rellene los valores de los proveedores de almacenamiento.
        Tenga en cuenta que el *nombre de usuario*, la *contraseña* y la *información de host* se pueden encontrar en Azure Portal en la sección del proveedor de almacenamiento determinado como, por ejemplo, *Cosmos DB*, *Table Storage* o *SQL Database*. El usuario define los nombres de tablas y las colecciones.
  
1. Establecimiento del proveedor de almacenamiento del bot

    1. Abra el archivo `index.js` en la raíz del proyecto. Al principio del archivo (líneas 38-98 aproximadamente) verá las configuraciones para cada proveedor de almacenamiento, como se indica en los comentarios. Leen los valores de configuración del archivo `.env` a través de `process.env` de Node. El siguiente fragmento de código muestra cómo configurar la instancia de SQL Database.

        [!code-javascript[Storage configuration](~/../botbuilder-samples/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot/index.js?range=77-92)]

    1. Especifique el proveedor de almacenamiento que desea que use el bot pasando la instancia del cliente de almacenamiento que prefiera al adaptador `StorageMapper` (línea 107 aproximadamente).  

        [!code-javascript[StorageMapper](~/../botbuilder-samples/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot/index.js?range=105-107)]

        La configuración predeterminada es *Cosmos DB*. Los valores posibles son:

        ```bash
            cosmosStorageClient
            tableStorage
            sqlStorage
        ```

1. Inicie la aplicación. En la raíz del proyecto, ejecute el comando siguiente:

    ```bash
    npm run start
    ```

## <a name="adapter-classes"></a>Clases de adaptador

### <a name="v4v3storagemapper"></a>V4V3StorageMapper

La clase `StorageMapper` contiene la funcionalidad principal del adaptador. Implementa la interfaz de almacenamiento de la versión v4 y asigna los métodos del proveedor de almacenamiento (lectura, escritura y eliminación) de nuevo a las clases del proveedor de almacenamiento de la versión v3 para que el estado del usuario con formato v3 se pueda usar en un bot de la versión v4.

### <a name="v4v3userstate"></a>V4V3UserState

Esta clase extiende la clase `BotState` de la versión v4 (`botbuilder-core`) para que use una clave del tipo v3, lo cual permite las operaciones de lectura, escritura y eliminación en el almacenamiento de la versión v3.

## <a name="testing-the-bot-using-bot-framework-emulator"></a>Prueba del bot con Bot Framework Emulator

[Bot Framework Emulator][5] es una aplicación de escritorio que permite probar y depurar un bot en localhost o mediante la ejecución remota mediante un túnel.

- Instale Bot Framework Emulator, versión 4.3.0 o posterior desde [aquí][6].

### <a name="connect-to-the-bot-using-bot-framework-emulator"></a>Conexión al bot con Bot Framework Emulator

1. Inicie Bot Framework Emulator.
1. Escriba la siguiente dirección URL (punto de conexión): `http://localhost:3978/api/messages`.

### <a name="testing-steps"></a>Pasos de pruebas

1. Abra el bot en el emulador y envíe un mensaje. Proporcione su nombre cuando se le solicite.
1. Una vez acabado el turno, envíe otro mensaje al bot.
1. Compruebe que no se le vuelve a solicitar su nombre. El bot debería leerlo del almacenamiento y reconocer que ya se le ha solicitado.
1. El bot debe devolver su mensaje.
1. Vaya al proveedor de almacenamiento en Azure y compruebe que su nombre está almacenado como datos de usuario en la base de datos.

## <a name="deploy-the-bot-to-azure"></a>Implementación del bot en Azure

Para más información sobre la implementación de un bot en Azure, consulte [Implementación de un bot en Azure][40] para obtener una lista completa de las instrucciones de implementación.

## <a name="further-reading"></a>Lecturas adicionales

- [Introducción a Azure Bot Service][21]
- [Bot State][7]
- [Escritura directa en el almacenamiento][8]
- [Administración del estado de la conversación y del usuario][9]
- [Restify][30]
- [dotenv][31]

[3]: https://aka.ms/botframework-emulator
[5]: https://github.com/microsoft/botframework-emulator
[6]: https://github.com/Microsoft/BotFramework-Emulator/releases
[7]: https://docs.microsoft.com/azure/bot-service/bot-builder-storage-concept
[8]: https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-storage?tabs=javascript
[9]: https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-state?tabs=javascript
[21]: https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0
[30]: https://www.npmjs.com/package/restify
[31]: https://www.npmjs.com/package/dotenv
[40]: https://aka.ms/azuredeployment

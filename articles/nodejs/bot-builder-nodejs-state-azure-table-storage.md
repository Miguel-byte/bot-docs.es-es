---
title: Administración de datos de estado personalizado con Azure Table Storage | Microsoft Docs
description: Aprenda a guardar y a recuperar datos de estado mediante Azure Table Storage con el SDK de Bot Builder para Node.js.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5bf308c440e08cb3c9d4730212fbba3053de459d
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998345"
---
# <a name="manage-custom-state-data-with-azure-table-storage-for-nodejs"></a>Administración de datos de estado personalizado con Azure Table Storage para Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

En este artículo, implementará Azure Table Storage para almacenar y administrar los datos de estado del bot. El servicio de estado de Connector predeterminado que usan los bots no es adecuado para el entorno de producción. Deberá usar las [extensiones de Azure](https://www.npmjs.com/package/botbuilder-azure) disponibles en GitHub o implementar un cliente de estado personalizado mediante la plataforma de almacenamiento de datos de su elección. Estas son algunas de las razones para usar el almacenamiento de estado personalizado:

- mayor rendimiento de la API de estado (más control sobre el rendimiento)
- latencia más baja para la distribución geográfica
- control sobre dónde se almacenan los datos (p. ej.: oeste de Estados Unidos frente a este de Estados Unidos)
- acceso a los datos de estado reales
- base de datos de estado no compartida con otros bots
- almacenamiento de más de 32 kb

## <a name="prerequisites"></a>Requisitos previos

- [Node.js](https://nodejs.org/en/).
- [Bot Framework Emulator](~/bot-service-debug-emulator.md).
- Debe tener un bot de Node.js. Si no tiene uno, vaya a la sección sobre [cómo crear un bot](bot-builder-nodejs-quickstart.md). 
- [Explorador de Storage](http://storageexplorer.com/).

## <a name="create-azure-account"></a>Creación de una cuenta de Azure
Si no tiene una cuenta de Azure, haga clic [aquí](https://azure.microsoft.com/en-us/free/) para registrarse y obtener una gratuita.

## <a name="set-up-the-azure-table-storage-service"></a>Configuración del servicio de Azure Table Storage
1. Una vez que ha iniciado sesión en Azure Portal, haga clic en **Nuevo** para crear un servicio de Azure Table Storage. 
2. Busque la **cuenta de almacenamiento** que implementa la tabla de Azure. Haga clic en **Crear** para comenzar a crear la cuenta de almacenamiento. 
3. Rellene los campos, haga clic en el botón **Crear** en la parte inferior de la pantalla para implementar el nuevo servicio de almacenamiento. 
4. Después de implementar el nuevo servicio de almacenamiento, vaya a la cuenta de almacenamiento que acaba de crear. Puede encontrarla en la hoja **Cuentas de almacenamiento**.
4. Seleccione **Claves de acceso** y copie la clave para usarla más adelante. El bot usará el **nombre de la cuenta de almacenamiento** y la **clave** para llamar al servicio de almacenamiento para guardar los datos de estado.

## <a name="install-botbuilder-azure-module"></a>Instalación del módulo botbuilder-azure

Para instalar el módulo `botbuilder-azure` desde un símbolo del sistema, vaya al directorio del bot y ejecute el siguiente comando npm:

```nodejs
npm install --save botbuilder-azure
```

## <a name="modify-your-bot-code"></a>Modificación del código del bot

Para usar el almacenamiento de **Azure Table**, agregue las siguientes líneas de código al archivo **app.js** del bot.

1. Reclame el módulo recién instalado.

   ```javascript
   var azure = require('botbuilder-azure'); 
   ```

2. Configure las opciones de conexión a Azure.
   ```javascript
   // Table storage
   var tableName = "Table-Name"; // You define
   var storageName = "Table-Storage-Name"; // Obtain from Azure Portal
   var storageKey = "Azure-Table-Key"; // Obtain from Azure Portal
   ```
   Los valores `storageName` y `storageKay` se pueden encontrar en el menú **Claves de acceso** de Azure Table. Si `tableName` no existe en Azure Table, se crea automáticamente.

3. Use el módulo `botbuilder-azure` para crear dos objetos para conectarse a la tabla de Azure. En primer lugar, cree una instancia de `AzureTableClient` pasando los valores de configuración de conexión. A continuación, cree una instancia de `AzureBotStorage` pasando el objeto `AzureTableClient`. Por ejemplo: 

   ```javascript
   var azureTableClient = new azure.AzureTableClient(tableName, storageName, storageKey);

   var tableStorage = new azure.AzureBotStorage({gzipData: false}, azureTableClient);
   ```

4. Especifique que quiere usar su base de datos personalizada en lugar del almacenamiento en memoria y agregue información de sesión a la base de datos. Por ejemplo: 

   ```javascript
   var bot = new builder.UniversalBot(connector, function (session) {
        // ... Bot code ...

        // capture session user information
        session.userData = {"userId": session.message.user.id, "jobTitle": "Senior Developer"};

        // capture conversation information  
        session.conversationData[timestamp.toISOString().replace(/:/g,"-")] = session.message.text;

        // save data
        session.save();
   })
   .set('storage', tableStorage);
   ```
Ahora está listo para probar el bot con el emulador.

## <a name="run-your-bot-app"></a>Ejecución de la aplicación bot

Desde un símbolo del sistema, vaya al directorio del bot y ejecute el bot con el siguiente comando:

```nodejs
node app.js
```

## <a name="connect-your-bot-to-the-emulator"></a>Conexión del bot al emulador

En este momento, el bot se ejecuta de forma local. Inicie el emulador y, después, conéctese al bot desde él:

1. Escriba <strong>http://localhost:port-number/api/messages</strong> en la barra de direcciones del emulador, donde port-number coincide con el número de puerto mostrado en el explorador donde se ejecuta la aplicación. Por ahora puede dejar en blanco los campos <strong>Microsoft App ID</strong> (Id. de aplicación de Microsoft) y <strong>Microsoft App Password</strong> (Contraseña de aplicación de Microsoft). Más adelante obtendrá esta información al [registrar el bot](~/bot-service-quickstart-registration.md).
2. Haga clic en **Conectar**.
3. Para probar el bot, envíe a su bot un mensaje. Interactúe con el bot como haría normalmente. Cuando haya terminado, vaya al **Explorador de Storage** y vea los datos de estado guardados.

## <a name="view-data-in-storage-explorer"></a>Visualización de los datos en el Explorador de Storage

Para ver los datos de estado, abra el **Explorador de Storage** y conéctese a Azure con sus credenciales de Azure Portal o conéctese directamente a la tabla mediante `storageName` y `storageKey` y navegue hasta su `tableName`. 

![Captura de pantalla del Explorador de Storage con filas de tabla de datos de bot](~/media/bot-builder-nodejs-state-azure-table-storage/bot-builder-nodejs-state-azure-table-storage-query.png)

Un registro de la conversación en la columna **datos** tiene esta apariencia:

```JSON
{
    "2018-05-15T18-23-48.780Z": "I'm the second user",
    "2018-05-15T18-23-55.120Z": "Do you know what time it is?",
    "2018-05-15T18-24-12.214Z": "I'm looking for information about the new process."
}
```

## <a name="next-step"></a>Paso siguiente

Ahora que tiene control total sobre los datos de estado del bot, echemos un vistazo a cómo puede usarlos para administrar mejor el flujo de conversación.

> [!div class="nextstepaction"]
> [Administración de flujo de conversación](bot-builder-nodejs-dialog-manage-conversation-flow.md)

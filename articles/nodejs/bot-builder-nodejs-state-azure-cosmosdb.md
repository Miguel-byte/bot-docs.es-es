---
title: Administración de datos de estado personalizado con Azure Cosmos DB | Microsoft Docs
description: Aprenda a guardar y a recuperar datos de estado mediante Azure Cosmos DB con Bot Builder SDK para Node.js.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0c0d91a7ec9fd1d72c7c51c042b0f52e28798778
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998122"
---
# <a name="manage-custom-state-data-with-azure-cosmos-db-for-nodejs"></a>Administración de datos de estado personalizado con Azure Cosmos DB para Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

En este artículo, implementará el almacenamiento de Cosmos DB para almacenar y administrar los datos de estado del bot. El servicio de estado de Connector predeterminado que usan los bots no es adecuado para el entorno de producción. Deberá usar las [extensiones de Azure](https://www.npmjs.com/package/botbuilder-azure) disponibles en GitHub o implementar un cliente de estado personalizado mediante la plataforma de almacenamiento de datos de su elección. Estas son algunas de las razones para usar el almacenamiento de estado personalizado:

- mayor rendimiento de la API de estado (más control sobre el rendimiento)
- latencia más baja para la distribución geográfica
- control sobre dónde se almacenan los datos (p. ej.: oeste de Estados Unidos frente a este de Estados Unidos)
- acceso a los datos de estado reales
- base de datos de estado no compartida con otros bots
- almacenamiento de más de 32 kb

## <a name="prerequisites"></a>Requisitos previos

- [Node.js](https://nodejs.org/en/).
- [Bot Framework Emulator](~/bot-service-debug-emulator.md)
- Debe tener un bot de Node.js. Si no tiene uno, vaya a la sección sobre [cómo crear un bot](bot-builder-nodejs-quickstart.md). 

## <a name="create-azure-account"></a>Creación de una cuenta de Azure
Si no tiene una cuenta de Azure, haga clic [aquí](https://azure.microsoft.com/en-us/free/) para registrarse y obtener una gratuita.

## <a name="set-up-the-azure-cosmos-db-database"></a>Configuración de la base de datos de Azure Cosmos DB
1. Una vez que ha iniciado sesión en Azure Portal, haga clic en *Nuevo* para crear una base de datos de **Azure Cosmos DB**. 
2. Haga clic en **Bases de datos**. 
3. Busque **Azure Cosmos DB** y haga clic en **Crear**.
4. Rellene los campos. Para el campo **API**, seleccione **SQL (DocumentDB)**. Una vez rellenados todos los campos, haga clic en el botón **Crear** en la parte inferior de la pantalla para implementar la nueva base de datos. 
5. Tras la implementación de la nueva base de datos, vaya a la base de datos. Haga clic en **Claves de acceso** para encontrar las claves y las cadenas de conexión. El bot usará esta información para llamar al servicio de almacenamiento para guardar los datos de estado.

## <a name="install-botbuilder-azure-module"></a>Instalación del módulo botbuilder-azure

Para instalar el módulo `botbuilder-azure` desde un símbolo del sistema, vaya al directorio del bot y ejecute el siguiente comando npm:

```nodejs
npm install --save botbuilder-azure
```

## <a name="modify-your-bot-code"></a>Modificación del código del bot

Para usar la base de datos de **Azure Cosmos DB**, agregue las siguientes líneas de código al archivo **app.js** del bot.

1. Reclame el módulo recién instalado.

   ```javascript
   var azure = require('botbuilder-azure'); 
   ```

2. Configure las opciones de conexión a Azure.
   ```javascript
   var documentDbOptions = {
       host: 'Your-Azure-DocumentDB-URI', 
       masterKey: 'Your-Azure-DocumentDB-Key', 
       database: 'botdocs',   
       collection: 'botdata'
   };
   ```
   Los valores `host` y `masterKey` se pueden encontrar en el menú **Claves** de la base de datos. Si las entradas `database` y `collection` no existen en la base de datos de Azure, se crearán automáticamente.

3. Use el módulo `botbuilder-azure` para crear dos objetos para conectarse a la base de datos de Azure. En primer lugar, cree una instancia de `DocumentDBClient` pasando los valores de configuración de conexión (que se define como `documentDbOptions` a partir de lo anterior). A continuación, cree una instancia de `AzureBotStorage` pasando el objeto `DocumentDBClient`. Por ejemplo: 
   ```javascript
   var docDbClient = new azure.DocumentDbClient(documentDbOptions);

   var cosmosStorage = new azure.AzureBotStorage({ gzipData: false }, docDbClient);
   ```

4. Especifique que quiere usar su base de datos personalizada en lugar del almacenamiento en memoria. Por ejemplo: 

   ```javascript
   var bot = new builder.UniversalBot(connector, function (session) {
        // ... Bot code ...
   })
   .set('storage', cosmosStorage);
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
3. Para probar el bot, envíe a su bot un mensaje. Interactúe con el bot como haría normalmente. Cuando haya terminado, vaya a **Explorador de Storage** y vea los datos de estado guardados.

## <a name="view-state-data-on-azure-portal"></a>Vista de los datos de estado en Azure Portal

Para ver los datos de estado, inicie sesión en Azure Portal y vaya a la base de datos. Haga clic en **Explorador de datos (versión preliminar)** para verificar que se está guardando la información de estado del bot.

## <a name="next-step"></a>Paso siguiente

Ahora que tiene control total sobre los datos de estado del bot, echemos un vistazo a cómo puede usarlos para administrar mejor el flujo de conversación.

> [!div class="nextstepaction"]
> [Administración de flujo de conversación](bot-builder-nodejs-dialog-manage-conversation-flow.md)

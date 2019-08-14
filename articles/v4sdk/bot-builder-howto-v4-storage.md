---
title: Escritura directa en el almacenamiento | Microsoft Docs
description: Obtenga información sobre cómo escribir directamente en el almacenamiento con Bot Framework SDK para .NET.
keywords: almacenamiento, lectura y escritura, almacenamiento en memoria, eTag
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b3743f798377de5bb1279e2af1b0124682a5575f
ms.sourcegitcommit: 6a83b2c8ab2902121e8ee9531a7aa2d85b827396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/09/2019
ms.locfileid: "68970700"
---
# <a name="write-directly-to-storage"></a>Escritura directa en el almacenamiento

[!INCLUDE[applies-to](../includes/applies-to.md)]

Puede leer y escribir directamente en el objeto de almacenamiento sin usar middleware ni un objeto de contexto. Esto puede ser adecuado para los datos que usa el bot para conservar una conversación o los que provienen de un origen externo al flujo de conversación del bot. En este modelo de almacenamiento de datos, los datos se leen directamente desde el almacenamiento en lugar de usar un administrador de estados. En los ejemplos de código de este artículo, se muestra cómo leer y escribir datos en el almacenamiento con **almacenamiento en memoria**, **Cosmos DB**, **Blob Storage** y el **almacén de transcripciones de blobs de Azure**. 

## <a name="prerequisites"></a>Requisitos previos
- Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.
- Conocimientos sobre el artículo: Creación de un bot de forma local para [dotnet](https://aka.ms/bot-framework-www-c-sharp-quickstart) o [nodeJS](https://aka.ms/bot-framework-www-node-js-quickstart).
- Plantilla de Bot Framework SDK v4 para [C#](https://aka.ms/bot-vsix) o [nodeJS](https://nodejs.org) y [yeoman](http://yeoman.io).

## <a name="about-this-sample"></a>Acerca de este ejemplo
El código de ejemplo de este artículo comienza con la estructura de un bot de eco básico y luego se amplía la funcionalidad de ese bot mediante la incorporación de código adicional (que se proporciona a continuación). Este código ampliado permite crear una lista para conservar las entradas del usuario tal como se reciben. En cada turno, se devuelve al usuario la lista completa de sus entradas. A continuación, la estructura de datos que contiene esta lista de entradas se guarda en el almacenamiento al final de ese turno. Se describen varios tipos de almacenamiento a medida que se va agregando funcionalidad adicional a este código de ejemplo.

## <a name="memory-storage"></a>Almacenamiento en memoria

Bot Framework SDK le permite almacenar las entradas del usuario mediante el almacenamiento en memoria. El almacenamiento en memoria se usa solo con fines de prueba y no está pensado para su uso en producción. Los tipos de almacenamiento persistente, como el almacenamiento de base de datos, son los más adecuados para los bots de producción. Asegúrese de establecer el almacenamiento en Cosmos DB o Blob Storage antes de publicar el bot.

#### <a name="build-a-basic-bot"></a>Creación de un bot básico

El resto de este tema se basa en un bot de eco. El código de ejemplo de un bot de eco puede compilarse localmente. Para ello, siga las instrucciones de inicio rápido para compilar un [bot de eco de C#](https://aka.ms/bot-framework-www-c-sharp-quickstart) o un [bot de eco de JS](https://aka.ms/bot-framework-www-node-js-quickstart).

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBot.cs**
```csharp
using System;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Linq;
using System.Threading;

// Represents a bot saves and echoes back user input.
public class EchoBot : ActivityHandler
{
   // Create local Memory Storage.
   private static readonly MemoryStorage _myStorage = new MemoryStorage();

   // Create cancellation token (used by Async Write operation).
   public CancellationToken cancellationToken { get; private set; }

   // Class for storing a log of utterances (text of messages) as a list.
   public class UtteranceLog : IStoreItem
   {
      // A list of things that users have said to the bot
      public List<string> UtteranceList { get; } = new List<string>();

      // The number of conversational turns that have occurred        
      public int TurnNumber { get; set; } = 0;

      // Create concurrency control where this is used.
      public string ETag { get; set; } = "*";
   }
     
   // Echo back user input.
   protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
   {
      // preserve user input.
      var utterance = turnContext.Activity.Text;  
      // make empty local logitems list.
      UtteranceLog logItems = null;
          
      // see if there are previous messages saved in storage.
      try
      {
         string[] utteranceList = { "UtteranceLog" };
         logItems = _myStorage.ReadAsync<UtteranceLog>(utteranceList).Result?.FirstOrDefault().Value;
      }
      catch
      {
         // Inform the user an error occured.
         await turnContext.SendActivityAsync("Sorry, something went wrong reading your stored messages!");
      }
         
      // If no stored messages were found, create and store a new entry.
      if (logItems is null)
      {
         // add the current utterance to a new object.
         logItems = new UtteranceLog();
         logItems.UtteranceList.Add(utterance);
         // set initial turn counter to 1.
         logItems.TurnNumber++;

         // Show user new user message.
         await turnContext.SendActivityAsync($"{logItems.TurnNumber}: The list is now: {string.Join(", ", logItems.UtteranceList)}");

         // Create Dictionary object to hold received user messages.
         var changes = new Dictionary<string, object>();
         {
            changes.Add("UtteranceLog", logItems);
         }
         try
         {
            // Save the user message to your Storage.
            await _myStorage.WriteAsync(changes, cancellationToken);
         }
         catch
         {
            // Inform the user an error occured.
            await turnContext.SendActivityAsync("Sorry, something went wrong storing your message!");
         }
      }
      // Else, our Storage already contained saved user messages, add new one to the list.
      else
      {
         // add new message to list of messages to display.
         logItems.UtteranceList.Add(utterance);
         // increment turn counter.
         logItems.TurnNumber++;
         
         // show user new list of saved messages.
         await turnContext.SendActivityAsync($"{logItems.TurnNumber}: The list is now: {string.Join(", ", logItems.UtteranceList)}");
         
         // Create Dictionary object to hold new list of messages.
         var changes = new Dictionary<string, object>();
         {
            changes.Add("UtteranceLog", logItems);
         };
         
         try
         {
            // Save new list to your Storage.
            await _myStorage.WriteAsync(changes,cancellationToken);
         }
         catch
         {
            // Inform the user an error occured.
            await turnContext.SendActivityAsync("Sorry, something went wrong storing your message!");
         }
      }
      ...  // OnMessageActivityAsync( )
   }
}

```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El bot necesita incluir un paquete adicional para usar el archivo de configuración .env. Si aún no está instalado, obtenga el paquete de dotnet de npm:

```powershell
npm install --save dotenv
```

**bot.js**
```javascript
const { ActivityHandler, MemoryStorage } = require('botbuilder');
const restify = require('restify');

// Add memory storage.
var storage = new MemoryStorage();

// Process incoming requests - adds storage for messages.
class MyBot extends ActivityHandler {
    constructor() {
        super();
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        this.onMessage(async turnContext => { console.log('this gets called (message)'); 
        await turnContext.sendActivity(`You said '${ turnContext.activity.text }'`); 
        // Save updated utterance inputs.
        await logMessageText(storage, turnContext);
    });
        this.onConversationUpdate(async turnContext => { console.log('this gets called (conversation update)'); 
        await turnContext.sendActivity('[conversationUpdate event detected]'); });
    }
}

// This function stores new user messages. Creates new utterance log if none exists.
async function logMessageText(storage, turnContext) {
    let utterance = turnContext.activity.text;
    // debugger;
    try {
        // Read from the storage.
        let storeItems = await storage.read(["UtteranceLogJS"])
        // Check the result.
        var UtteranceLogJS = storeItems["UtteranceLogJS"];
        if (typeof (UtteranceLogJS) != 'undefined') {
            // The log exists so we can write to it.
            storeItems["UtteranceLogJS"].turnNumber++;
            storeItems["UtteranceLogJS"].UtteranceList.push(utterance);
            // Gather info for user message.
            var storedString = storeItems.UtteranceLogJS.UtteranceList.toString();
            var numStored = storeItems.UtteranceLogJS.turnNumber;

            try {
                await storage.write(storeItems)
                turnContext.sendActivity(`${numStored}: The list is now: ${storedString}`);
            } catch (err) {
                turnContext.sendActivity(`Write failed of UtteranceLogJS: ${err}`);
            }
        }
        else{
            turnContext.sendActivity(`Creating and saving new utterance log`);
            var turnNumber = 1;
            storeItems["UtteranceLogJS"] = { UtteranceList: [`${utterance}`], "eTag": "*", turnNumber }
            // Gather info for user message.
            var storedString = storeItems.UtteranceLogJS.UtteranceList.toString();
            var numStored = storeItems.UtteranceLogJS.turnNumber;

            try {
                await storage.write(storeItems)
                turnContext.sendActivity(`${numStored}: The list is now: ${storedString}`);
            } catch (err) {
                turnContext.sendActivity(`Write failed: ${err}`);
            }
        }
    }
    catch (err){
        turnContext.sendActivity(`Read rejected. ${err}`);
    }
}

module.exports.MyBot = MyBot;

```
---

### <a name="start-your-bot"></a>Inicio del bot
Ejecute el bot localmente.

### <a name="start-the-emulator-and-connect-your-bot"></a>Inicio del emulador y conexión del bot
- Instale [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme). A continuación, inicie el emulador y, después, conéctese al bot en el emulador:

1. Haga clic en el vínculo **Create new bot configuration** (Crear configuración de bot) en la pestaña de bienvenida del emulador. 
2. Rellene los campos para conectarse al bot con la información de la página web que aparece cuando inicia el bot.

### <a name="interact-with-your-bot"></a>Interacción con el bot
Envíe un mensaje al bot. El bot mostrará la lista de mensajes que ha recibido.

![Bot de almacenamiento de pruebas del emulador](./media/emulator-direct-storage-test.png)

## <a name="using-cosmos-db"></a>Uso de Cosmos DB
Ahora que ha usado el almacenamiento en memoria, actualizaremos el código para usar Azure Cosmos DB. Cosmos DB es la base de datos multimodelo de distribución global de Microsoft. Azure Cosmos DB permite escalar de forma elástica e individual el rendimiento y el almacenamiento en cualquiera de las regiones geográficas de Azure. Ofrece garantía de rendimiento, latencia, disponibilidad y coherencia con Acuerdos de Nivel de Servicio (SLA) integrales. 

### <a name="set-up"></a>Instalación
Para utilizar Cosmos DB en el bot, es necesario configurar algunas cosas antes de meternos en el código. Para obtener una descripción detallada sobre la base de datos de Cosmos DB y la creación de aplicaciones acceda a esta documentación para [Cosmos DB dotnet](https://aka.ms/Bot-framework-create-dotnet-cosmosdb) o [Cosmos DB nodejs](https://aka.ms/Bot-framework-create-nodejs-cosmosdb).

### <a name="create-your-database-account"></a>Creación de la cuenta de base de datos

1. En una nueva ventana del explorador, inicie sesión en [Azure Portal](http://portal.azure.com).

![Creación de una base de datos de Cosmos DB](./media/create-cosmosdb-database.png)

2. Haga clic en **Crear un recurso > Bases de datos > Azure Cosmos DB**

![Página Nueva cuenta de Cosmos DB](./media/cosmosdb-new-account-page.png)

3. En la **página Nueva cuenta**, proporcione información sobre la **suscripción** y el **grupo de recursos**. Cree un nombre único para el campo **Nombre de cuenta**. Este se convertirá finalmente en parte del nombre de la dirección URL de acceso a los datos. Para **API**, seleccione **Core(SQL)** y proporcione una **ubicación** cercana para mejorar los tiempos de acceso a los datos.
4. Después, haga clic en **Revisar + crear**.
5. Una vez que la validación se haya completado correctamente, haga clic en **Crear**.

La cuenta tarda unos minutos en crearse. Espere a que el portal muestre la página "Enhorabuena. Se ha creado su cuenta de Azure Cosmos DB".

### <a name="add-a-collection"></a>Agregar una colección

![Adición de una colección de Cosmos DB](./media/add_database_collection.png)

1. Haga clic en **Configuración > Nueva colección**. El área **Agregar colección** se muestra en el extremo derecho, pero es posible que haya que desplazarse hacia la derecha para verlo. Debido a actualizaciones recientes de Cosmos DB, asegúrese de agregar una clave de partición única: _/id_. Esta clave evita errores de consultas entre particiones.

![Cosmos DB](./media/cosmos-db-sql-database.png)

2. La nueva colección de base de datos, "bot-cosmos-sql-db" con el identificador de colección "bot-storage". Usaremos estos valores en el ejemplo de código siguiente.

![Claves de Cosmos DB](./media/comos-db-keys.png)

3. El identificador URI del punto de conexión y la clave están disponibles en la pestaña **Claves** de la configuración de la base de datos. Necesitará estos valores para configurar el código más adelante en este artículo. 

### <a name="add-configuration-information"></a>Adición de la información de configuración
Los datos de configuración para agregar el almacenamiento de Cosmos DB son cortos y sencillos; puede agregar una configuración adicional con estos mismos métodos a medida que el bot se vuelva más complejo. En este ejemplo se usan los nombres de la base de datos y de la colección de Cosmos DB del ejemplo anterior.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBot.cs**
```csharp
public class EchoBot : ActivityHandler
{
   private const string CosmosServiceEndpoint = "<your-cosmos-db-URI>";
   private const string CosmosDBKey = "<your-cosmos-db-account-key>";
   private const string CosmosDBDatabaseName = "bot-cosmos-sql-db";
   private const string CosmosDBCollectionName = "bot-storage";
   ...
   
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Agregue la siguiente información al archivo `.env`.

**.env**
```javascript
DB_SERVICE_ENDPOINT="<your-Cosmos-db-URI>"
AUTH_KEY="<your-cosmos-db-account-key>"
DATABASE="<bot-cosmos-sql-db>"
COLLECTION="<bot-storage>"
```
---

#### <a name="installing-packages"></a>Instalación de paquetes
Asegúrese de que tiene los paquetes necesarios para Cosmos DB.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Puede agregar referencias a botbuilder-azure en el proyecto mediante npm.
>**Nota**: este paquete de npm se basa en una instalación de Python existente en la máquina de desarrollo. Si no ha instalado Python anteriormente, puede encontrar los recursos de instalación para la máquina aquí: [Python.org](https://www.python.org/downloads/)

```powershell
npm install --save botbuilder-azure 
```

Si aún no está instalado, obtenga el paquete de dotnet de npm para acceder a la configuración del archivo `.env`.

```powershell
npm install --save dotenv
```
---

### <a name="implementation"></a>Implementación 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

El código de ejemplo siguiente se ejecuta con el mismo código de bot que el ejemplo de [almacenamiento en memoria](#memory-storage) anterior.
El siguiente fragmento de código muestra una implementación de almacenamiento de Cosmos DB para "_myStorage_", que sustituye al almacenamiento en memoria local. El almacenamiento en memoria se marca como comentario y se sustituye por una referencia a Cosmos DB.

**EchoBot.cs**
```csharp

using System;
...
using Microsoft.Bot.Builder.Azure;
...
public class EchoBot : ActivityHandler
{
   // Create local Memory Storage - commented out.
   // private static readonly MemoryStorage _myStorage = new MemoryStorage();

   // Replaces Memory Storage with reference to Cosmos DB.
   private static readonly CosmosDbStorage _myStorage = new CosmosDbStorage(new CosmosDbStorageOptions
   {
      AuthKey = CosmosDBKey,
      CollectionId = CosmosDBCollectionName,
      CosmosDBEndpoint = new Uri(CosmosServiceEndpoint),
      DatabaseId = CosmosDBDatabaseName,
   });
   
   ...
}

```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El siguiente código de ejemplo es similar al de [almacenamiento en memoria](#memory-storage), pero con algunos pequeños cambios.

Se requiere `CosmosDbStorage` de `botbuilder-azure` y configurar dotenv para leer el archivo `.env`.

**bot.js**

```javascript
const { CosmosDbStorage } = require("botbuilder-azure");
```
Marque como comentario el almacenamiento en memoria y reemplácelo por una referencia a Cosmos DB.

**bot.js**
```javascript
// initialized to access values in .env file.
const ENV_FILE = path.join(__dirname, '.env');
require('dotenv').config({ path: ENV_FILE });

// Create local Memory Storage - commented out.
// var storage = new MemoryStorage();

// Create access to CosmosDb Storage - this replaces local Memory Storage.
var storage = new CosmosDbStorage({
    serviceEndpoint: process.env.DB_SERVICE_ENDPOINT, 
    authKey: process.env.AUTH_KEY, 
    databaseId: process.env.DATABASE,
    collectionId: process.env.COLLECTION
})

```
---

## <a name="start-your-bot"></a>Inicio del bot
Ejecute el bot localmente.

## <a name="test-your-bot-with-bot-framework-emulator"></a>Prueba del bot mediante Bot Framework Emulator
Inicie Bot Framework Emulator y conéctese al bot:

1. Haga clic en el vínculo **Create new bot configuration** (Crear configuración de bot) en la pestaña de bienvenida del emulador. 
2. Rellene los campos para conectarse al bot con la información de la página web que aparece cuando inicia el bot.

## <a name="interact-with-your-bot"></a>Interacción con el bot
Envíe un mensaje al bot y este enumerará los mensajes que ha recibido.
![Emulador en ejecución](./media/emulator-direct-storage-test.png)


### <a name="view-your-data"></a>Consulta de los datos
Después de ejecutar el bot y guardar la información, podemos ver los datos almacenados en Azure Portal en la pestaña **Explorador de datos**. 

![Ejemplo del Explorador de datos](./media/data_explorer.PNG)


## <a name="using-blob-storage"></a>Uso de Blob Storage 
Azure Blob Storage es la solución de almacenamiento de objetos de Microsoft para la nube. Blob Storage está optimizado para el almacenamiento de cantidades masivas de datos no estructurados, como texto o datos binarios.

### <a name="create-your-blob-storage-account"></a>Creación de la cuenta de Blob Storage
Para utilizar Blob Storage en el bot, es necesario configurar algunas cosas antes de meternos en el código.
1. En una nueva ventana del explorador, inicie sesión en [Azure Portal](http://portal.azure.com).

![Creación de un almacenamiento de blobs](./media/create-blob-storage.png)

2. Haga clic en **Crear un recurso > Almacenamiento > Cuenta de almacenamiento: blob, archivo, tabla, cola**

![Página Nueva cuenta de Blob Storage](./media/blob-storage-new-account.png)

3. En la página **Nueva cuenta**, escriba un **Nombre** para la cuenta de almacenamiento, seleccione **Blob Storage** en **Tipo de cuenta** y proporcione la información de **Ubicación**, **Grupo de recursos** y **Suscripción**.  
4. Después, haga clic en **Revisar + crear**.
5. Una vez que la validación se haya completado correctamente, haga clic en **Crear**.

### <a name="create-blob-storage-container"></a>Creación de un contenedor de Blob Storage
Una vez creada la cuenta de Blob Storage, abra esta cuenta 
1. seleccionando el recurso.
2. Abra ahora mediante el Explorador de Storage (versión preliminar).

![Creación de un contenedor de Blob Storage](./media/create-blob-container.png)

3. Haga clic con el botón derecho en CONTENEDORES DE BLOBS y seleccione _Crear contenedor de blobs_.
4. Agregue un nombre. Usará este nombre para el valor "your-blob-storage-container-name" para proporcionar acceso a la cuenta de Blob Storage.

#### <a name="add-configuration-information"></a>Adición de la información de configuración
Busque las claves de Blob Storage que necesita para configurar el almacenamiento de blobs del bot, tal y como se indicó anteriormente:
1. En Azure Portal, abra la cuenta de Blob Storage y seleccione **Configuración > Claves de acceso**.

![Búsqueda de las claves de Blob Storage](./media/find-blob-storage-keys.png)

Se usará la _cadena de conexión_ key1 para el valor "your-blob-storage-container-name" para proporcionar acceso a la cuenta de Blob Storage.

#### <a name="installing-packages"></a>Instalación de paquetes
Si no se han instalado anteriormente para usar Cosmos DB, instale los siguientes paquetes.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Agregue referencias a botbuilder-azure en el proyecto mediante npm.
>**Nota**: este paquete de npm se basa en una instalación de Python existente en la máquina de desarrollo. Si no ha instalado Python anteriormente, puede encontrar los recursos de instalación para la máquina aquí: [Python.org](https://www.python.org/downloads/)

```powershell
npm install --save botbuilder-azure 
```

Si aún no está instalado, obtenga el paquete de dotnet de npm para acceder a la configuración del archivo `.env`.

```powershell
npm install --save dotenv
```
---

### <a name="implementation"></a>Implementación 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBot.cs**
```csharp
using Microsoft.Bot.Builder.Azure;
```
Actualice la línea de código que apunta "_myStorage_" a la cuenta de Blob Storage existente.

**EchoBot.cs**
```csharp
private static readonly AzureBlobStorage _myStorage = new AzureBlobStorage("<your-blob-storage-account-string>", "<your-blob-storage-container-name>");
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Agregue la siguiente información al archivo `.env`.

**.env**
```javascript
BLOB_NAME="<your-blob-storage-container-name>"
BLOB_STRING="<your-blob-storage-account-string>"
```

Actualice el archivo `bot.js` como sigue. Se requiere `BlobStorage` de `botbuilder-azure`

**bot.js**
```javascript
const { BlobStorage } = require("botbuilder-azure");
```

Si no agregó código para cargar el archivo `.env` para implementar el almacenamiento de Cosmos DB, agréguelo aquí.

```javascript
// initialized to access values in .env file.
const ENV_FILE = path.join(__dirname, '.env');
require('dotenv').config({ path: ENV_FILE });
```
Ahora, actualice el código para que apunte el "_almacenamiento_" hacia la cuenta de Blob Storage ya existente, marcando como comentario las definiciones de almacenamiento anteriores y agregando lo siguiente.

**bot.js**
```javascript
var storage = new BlobStorage({
    containerName: process.env.BLOB_NAME,
    storageAccountOrConnectionString: process.env.BLOB_STRING
});
```
---

Una vez que el almacenamiento se establece para que apunte a la cuenta de Blob Storage, el código del bot almacena y recupera los datos desde Blob Storage.

## <a name="start-your-bot"></a>Inicio del bot
Ejecute el bot localmente.

## <a name="start-the-emulator-and-connect-your-bot"></a>Inicio del emulador y conexión del bot
A continuación, inicie el emulador y, después, conéctese al bot en el emulador:

1. Haga clic en el vínculo **Create new bot configuration** (Crear configuración de bot) en la pestaña de bienvenida del emulador. 
2. Rellene los campos para conectarse al bot con la información de la página web que aparece cuando inicia el bot.

## <a name="interact-with-your-bot"></a>Interacción con el bot
Envíe un mensaje al bot y este mostrará una lista de los mensajes que recibe.

![Bot de almacenamiento de pruebas del emulador](./media/emulator-direct-storage-test.png)

### <a name="view-your-data"></a>Consulta de los datos
Después de ejecutar el bot y guardar la información, lo podemos ver en la pestaña **Explorador de Storage** de Azure Portal.

## <a name="blob-transcript-storage"></a>Almacenamiento de transcripciones de blobs
El almacenamiento de transcripciones de blobs de Azure ofrece una opción de almacenamiento especializado que le permite guardar y recuperar con facilidad las conversaciones del usuario en forma de una transcripción grabada. El almacenamiento de transcripciones de blobs de Azure es especialmente útil para capturar automáticamente las entradas de usuario para examinarlas al tiempo que se depura el rendimiento del bot.

### <a name="set-up"></a>Instalación
El almacenamiento de transcripciones de blobs de Azure puede usar la misma cuenta de Blob Storage que creó siguiendo los pasos detallados en las secciones "_Creación de la cuenta de Blob Storage_" y "_Adición de la información de configuración_" anteriores. Ahora se agrega un contenedor para almacenar las transcripciones

![Creación de un contenedor de transcripciones](./media/create-blob-transcript-container.png)

1. Abra la cuenta de Azure Blob Storage.
1. Haga clic en _Explorador de Storage_.
1. Haga clic con el botón derecho en _CONTENEDORES DE BLOB_ y seleccione _Crear contenedor de blobs_.
1. Escriba un nombre para el contenedor de transcripciones y, después, seleccione _Aceptar_. (Hemos escrito mybottranscripts)

### <a name="implementation"></a>Implementación 
El siguiente código conecta el puntero de almacenamiento de transcripciones `_myTranscripts` a la nueva cuenta de almacenamiento de transcripciones de blobs de Azure. Para crear este vínculo con un nuevo nombre de contenedor, <your-blob-transcript-container-name>, cree un nuevo contenedor en Blob Storage para almacenar los archivos de transcripción.

**echoBot.cs**
```csharp
using Microsoft.Bot.Builder.Azure;

public class EchoBot : ActivityHandler
{
   ...
   
   private readonly AzureBlobTranscriptStore _myTranscripts = new AzureBlobTranscriptStore("<your-blob-transcript-storage-account-string>", "<your-blob-transcript-container-name>");
   
   ...
}

```

### <a name="store-user-conversations-in-azure-blob-transcripts"></a>Almacenamiento de las conversaciones del usuario en transcripciones de blobs de Azure
Después de tener disponible un contenedor de blobs para almacenar las transcripciones, puede comenzar a conservar las conversaciones de los usuarios con el bot. Estas conversaciones se pueden utilizar más adelante como una herramienta de depuración para ver cómo interactúan los usuarios con el bot. Cada vez que se elige la opción _Reiniciar conversación_ del emulador se inicia la creación de una nueva lista de conversación de transcripción. El siguiente código conserva las entradas de la conversación del usuario en un archivo de transcripciones almacenado.
- La transcripción actual se guarda mediante `LogActivityAsync`.
- Las transcripciones guardadas se recuperan mediante `ListTranscriptsAsync`.
En este ejemplo de código, el identificador de cada transcripción almacenada se guarda en una lista llamada "storedTranscripts". Esta lista se usa más adelante para administrar el número de transcripciones de blobs almacenadas que se conservan.

**echoBot.cs**
```csharp

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    await _myTranscripts.LogActivityAsync(turnContext.Activity);

    List<string> storedTranscripts = new List<string>();
    PagedResult<Microsoft.Bot.Builder.TranscriptInfo> pagedResult = null;
    var pageSize = 0;
    do
    {
       pagedResult = await _myTranscripts.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
       pageSize = pagedResult.Items.Count();

       // transcript item contains ChannelId, Created, Id.
       // save the channelIds found by "ListTranscriptsAsync" to a local list.
       foreach (var item in pagedResult.Items)
       {
          storedTranscripts.Add(item.Id);
       }
    } while (pagedResult.ContinuationToken != null);
    
    ...
}

```

### <a name="manage-stored-blob-transcripts"></a>Administración de las transcripciones de blobs almacenadas
Aunque las transcripciones almacenadas se pueden usar como una herramienta de depuración, con el tiempo, el número de transcripciones almacenadas puede crecer más de lo que desea conservar. El código adicional que se incluye a continuación usa `DeleteTranscriptAsync` para eliminarlas todas excepto los tres últimos elementos de transcripción recuperados del almacén de transcripciones del blob.

**echoBot.cs**
```csharp

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    await _myTranscripts.LogActivityAsync(turnContext.Activity);

    List<string> storedTranscripts = new List<string>();
    PagedResult<Microsoft.Bot.Builder.TranscriptInfo> pagedResult = null;
    var pageSize = 0;
    do
    {
       pagedResult = await _myTranscripts.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
       pageSize = pagedResult.Items.Count();

       // transcript item contains ChannelId, Created, Id.
       // save the channelIds found by "ListTranscriptsAsync" to a local list.
       foreach (var item in pagedResult.Items)
       {
          storedTranscripts.Add(item.Id);
       }
    } while (pagedResult.ContinuationToken != null);
    
    // Manage the size of your transcript storage.
    for (int i = 0; i < pageSize; i++)
    {
       // Remove older stored transcripts, save just the last three.
       if (i < pageSize - 3)
       {
          string thisTranscriptId = storedTranscripts[i];
          try
          {
             await _myTranscripts.DeleteTranscriptAsync("emulator", thisTranscriptId);
           }
           catch (System.Exception ex)
           {
              await turnContext.SendActivityAsync("Debug Out: DeleteTranscriptAsync had a problem!");
              await turnContext.SendActivityAsync("exception: " + ex.Message);
           }
       }
    }
    ...
}

```

El vínculo siguiente proporciona más información sobre el [almacenamiento de transcripciones de blobs de Azure](https://docs.microsoft.com/dotnet/api/microsoft.bot.builder.azure.azureblobtranscriptstore) 

## <a name="additional-information"></a>Información adicional

### <a name="manage-concurrency-using-etags"></a>Administración de la simultaneidad mediante eTags
En nuestro ejemplo de código de bot, establecemos la propiedad `eTag` de cada elemento `IStoreItem` en `*`. Cosmos DB utiliza el miembro `eTag` (etiqueta de entidad) del objeto de almacén para administrar la simultaneidad. La etiqueta `eTag` indica a la base de datos qué hacer si otra instancia del bot ha cambiado el objeto en el mismo almacenamiento en el que está escribiendo su bot. 

<!-- define optimistic concurrency -->

#### <a name="last-write-wins---allow-overwrites"></a>La última escritura tiene prioridad: permitir la sobrescritura
Un valor de propiedad `eTag` de asterisco (`*`) indica que la última escritura tiene prioridad. Al crear un nuevo almacén de datos, puede establecer la etiqueta `eTag` de una propiedad en `*` para indicar que no ha guardado previamente los datos que está escribiendo, o que desea que el último escritor sobrescriba cualquier propiedad guardada previamente. Si la simultaneidad no es un problema para el bot, establezca la propiedad `eTag` en `*` para habilitar la sobrescritura para los datos que escriba.

#### <a name="maintain-concurrency-and-prevent-overwrites"></a>Mantenimiento de la simultaneidad y evitación de la sobrescritura
Cuando almacene datos en Cosmos DB, use un valor distinto de `*` en la propiedad `eTag` si desea evitar el acceso simultáneo a una propiedad y sobrescribir los cambios de otra instancia del bot. El bot recibe una respuesta de error con el mensaje `etag conflict key=` cuando intenta guardar los datos de estado, y la etiqueta `eTag` no tiene el mismo valor que `eTag` en el almacenamiento. <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

De forma predeterminada, el almacén de Cosmos DB comprueba la igualdad de la propiedad `eTag` de un objeto de almacenamiento cada vez que un bot escribe en ese elemento y, a continuación, lo actualiza a un nuevo valor único después de cada escritura. Si la propiedad `eTag` en la escritura no coincide con la `eTag` en el almacenamiento, significa que otro subproceso o bot ha cambiado los datos. 

Por ejemplo, supongamos que desea que su bot edite una nota guardada, pero no quiere que sobrescriba los cambios que haya realizado otra instancia del bot. Si otra instancia del bot ha realizado modificaciones, lo que quiere es que el usuario edite la versión con las actualizaciones más recientes.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

Primero, cree una clase que implemente `IStoreItem`.

**EchoBot.cs**
```csharp
public class Note : IStoreItem
{
    public string Name { get; set; }
    public string Contents { get; set; }
    public string ETag { get; set; }
}
```

A continuación, cree una nota inicial mediante la creación de un objeto de almacenamiento y agregue el objeto a su almacén.

**EchoBot.cs**
```csharp
// create a note for the first time, with a non-null, non-* ETag.
var note = new Note { Name = "Shopping List", Contents = "eggs", ETag = "x" };

var changes = Dictionary<string, object>();
{
    changes.Add("Note", note);
};
await NoteStore.WriteAsync(changes, cancellationToken);
```

A continuación, acceda y actualice la nota más adelante, manteniendo su `eTag` que usted lee desde el almacén.

**EchoBot.cs**
```csharp
var note = NoteStore.ReadAsync<Note>("Note").Result?.FirstOrDefault().Value;

if (note != null)
{
    note.Contents += ", bread";
    var changes = new Dictionary<string, object>();
    {
         changes.Add("Note1", note);
    };
    await NoteStore.WriteAsync(changes, cancellationToken);
}
```

Si se ha actualizado la nota en el almacén antes de que haya escrito los cambios, la llamada a `Write` iniciará una excepción.


### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Agregue una función auxiliar al final del bot que escriba una nota de ejemplo en un almacén de datos.
Primero, cree un objeto `myNoteData`.

**bot.js**
```javascript
// Helper function for writing a sample note to a data store
async function createSampleNote(storage, context) {
    var myNoteData = {
        name: "Shopping List",
        contents: "eggs",
        // If any Note file is already stored, the eTag field
        // must be set to "*" in order to allow writing without first reading the stored eTag
        // otherwise you'll likely get an exception indicating an eTag conflict. 
        eTag: "*"
    }
}
```

Dentro de la función auxiliar `createSampleNote`, inicialice un objeto `changes`, agréguele las *notas* y, a continuación, escríbalo en el almacenamiento.

**bot.js**
```javascript
// Write the note data to the "Note" key
var changes = {};
changes["Note"] = myNoteData;
// Creates a file named Note, if it doesn't already exist.
// specifying eTag= "*" will overwrite any existing contents.
// The act of writing to the file automatically updates the eTag property
// The first time you write to Note, the eTag is changed from *, and file contents will become:
//    {"name":"Shopping List","contents":"eggs","eTag":"1"}
try {
     await storage.write(changes);
     var list = changes["Note"].contents;
     await context.sendActivity(`Successful created a note: ${list}`);
} catch (err) {
     await context.sendActivity(`Could not create note: ${err}`);
}
```

Se accede a la función auxiliar desde dentro de la lógica del bot agregando la siguiente llamada:

**bot.js**
```javascript
// Save a note with etag.
await createSampleNote(storage, turnContext);
```

Una vez creado, para recuperar y actualizar la nota más adelante, creamos otra función auxiliar a la que se puede acceder cuando el usuario escribe "actualizar nota".

**bot.js**
```javascript
async function updateSampleNote(storage, context) {
    try {
        // Read in a note
        var note = await storage.read(["Note"]);
        console.log(`note.eTag=${note["Note"].eTag}\n note=${JSON.stringify(note)}`);
        // update the note that we just read
        note["Note"].contents += ", bread";
        console.log(`Updated note=${JSON.stringify(note)}`);

        try {
             await storage.write(note); // Write the changes back to storage
             var list = note["Note"].contents;
             await context.sendActivity(`Successfully updated note: ${list}`);
        } catch (err) {            
             console.log(`Write failed: ${err}`);
        }
    }
    catch (err) {
        await context.sendActivity(`Unable to read the Note: ${err}`);
    }
}
```

Se accede a esta función auxiliar desde dentro de la lógica del bot agregando la siguiente llamada:

**bot.js**
```javascript
// Update a note with etag.
await updateSampleNote(storage, turnContext);
```

Si otro usuario ha almacenado la nota en el almacén antes de que intentara reescribir los cambios, el valor `eTag` ya no coincidirá y la llamada a `write` generará una excepción.

---

Para mantener la simultaneidad, siempre es necesario leer una propiedad desde el almacenamiento y luego modificar la propiedad que se ha leído, de esta manera `eTag` se mantiene. Si lee datos de usuario desde el almacén, la respuesta contendrá la propiedad de eTag. Si cambia los datos y escribe datos actualizados en el almacén, la solicitud debe incluir la propiedad de eTag que especifica el mismo valor que leyó anteriormente. Sin embargo, escribir un objeto con su `eTag` establecida en `*` permitirá que la escritura sobrescriba cualquier otro cambio.

## <a name="next-steps"></a>Pasos siguientes
Ahora que sabe cómo leer lectura y escritura directamente desde el almacenamiento, eche un vistazo a cómo puede usar el Administrador de estado para que lo haga por usted.

> [!div class="nextstepaction"]
> [Guardar el estado mediante las propiedades de usuario y de conversación](bot-builder-howto-v4-state.md)


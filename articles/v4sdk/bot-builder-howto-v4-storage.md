---
title: Escritura directa en el almacenamiento | Microsoft Docs
description: Obtenga información sobre cómo escribir directamente en el almacenamiento con Bot Framework SDK para .NET.
keywords: almacenamiento, lectura y escritura, almacenamiento en memoria, eTag
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/13/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 268ac62c68d2157a50d3b20cddf7e8f54cccaefe
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/21/2019
ms.locfileid: "56591206"
---
# <a name="write-directly-to-storage"></a>Escritura directa en el almacenamiento

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Puede leer y escribir directamente en el objeto de almacenamiento sin usar middleware ni un objeto de contexto. Esto puede ser adecuado para los datos que usa el bot, que provienen de un origen externo al flujo de conversación del bot. Por ejemplo, suponga que el bot permite al usuario solicitar el informe meteorológico y el bot lo recupera para una fecha concreta, leyéndolo de una base de datos externa. El contenido de la base de datos meteorológicos no depende de la información del usuario ni del contexto de la conversación, por lo que se podría leer directamente desde el almacenamiento en lugar de usar el administrador de estado. En los ejemplos de código de este artículo, se muestra cómo leer y escribir datos en el almacenamiento con **almacenamiento en memoria**, **Cosmos DB**, **Blob Storage** y el **almacén de transcripciones de blobs de Azure**. 

## <a name="prerequisites"></a>Requisitos previos
- Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/en-us/free/) antes de empezar.
- Instalación de Bot Framework [Emulator](https://aka.ms/Emulator-wiki-getting-started)

## <a name="memory-storage"></a>Almacenamiento en memoria

Primero crearemos un bot que lea y escriba datos en el almacenamiento en memoria. El almacenamiento en memoria se usa solo con fines de prueba y no está pensado para su uso en producción. Asegúrese de establecer el almacenamiento en Cosmos DB o Blob Storage antes de publicar el bot.

#### <a name="build-a-basic-bot"></a>Creación de un bot básico

El resto de este tema se basa en un bot de eco. El código de ejemplo del bot de eco para crear este proyecto puede encontrarse aquí: [Ejemplo de C#](https://aka.ms/cs-echobot-sample) o aquí [Ejemplo de JS](https://aka.ms/js-echobot-sample). Puede usar Bot Framework Emulator para conectarse al bot, conversar con él y probarlo. En el ejemplo siguiente se agregan todos los mensajes del usuario a una lista. La estructura de datos que contiene la lista se guarda en el almacenamiento.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**MyBot.cs**
```csharp
using System;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Linq;
using System.Threading;

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

// Every Conversation turn for our Bot calls this method.
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{     
   if (turnContext.Activity.Type == ActivityTypes.Message)
   {
      // Replace the two lines of code from original MyBot code with the following:
      
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
   }
}

```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**
```javascript
const { BotFrameworkAdapter, ConversationState, MemoryStorage } = require('botbuilder');
const restify = require('restify');

// Create server.
// let server = restify.createServer();
// server.listen(process.env.port || process.env.PORT || 3978, function () {
//    console.log(`${server.name} listening to ${server.url}`);
// });

// Create adapter.
// const adapter = new BotFrameworkAdapter({
//    appId: process.env.MICROSOFT_APP_ID,
//    appPassword: process.env.MICROSOFT_APP_PASSWORD
//  });

// Add memory storage.
var storage = new MemoryStorage();

// const conversationState = new ConversationState(storage);
// adapter.use(conversationState);

// Listen for incoming requests - adds storage for messages.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {

        if (context.activity.type === 'message') {
            // Route to main dialog.
            await myBot.onTurn(context);
            // Save updated utterance inputs.
            await logMessageText(storage, context);
        }
        else {
            // Just route to main dialog.
            await myBot.onTurn(context);
        } 
    });
});

// This function stores new user messages. Creates new utterance log if none exists.
async function logMessageText(storage, context) {
    let utterance = context.activity.text;
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
                context.sendActivity(`${numStored}: You stored: ${storedString}`);
            } catch (err) {
                context.sendActivity(`Write failed of UtteranceLogJS: ${err}`);
            }

         } else {
            context.sendActivity(`Creating and saving new utterance log`);
            var turnNumber = 1;
            storeItems["UtteranceLogJS"] = { UtteranceList: [`${utterance}`], "eTag": "*", turnNumber }
            // Gather info for user message.
            var storedString = storeItems.UtteranceLogJS.UtteranceList.toString();
            var numStored = storeItems.UtteranceLogJS.turnNumber;

            try {
                await storage.write(storeItems)
                context.sendActivity(`${numStored}: You stored: ${storedString}`);
            } catch (err) {
                context.sendActivity(`Write failed: ${err}`);
            }
        }
    } catch (err) {
        context.sendActivity(`Read rejected. ${err}`);
    };
}

```

---

### <a name="start-your-bot"></a>Inicio del bot
Ejecute el bot localmente.

### <a name="start-the-emulator-and-connect-your-bot"></a>Inicio del emulador y conexión del bot
A continuación, inicie el emulador y, después, conéctese al bot en el emulador:

1. Haga clic en el vínculo **Open bot** (Abrir bot) de la pestaña de bienvenida del emulador. 
2. Seleccione el archivo .bot ubicado en el directorio donde se creó el proyecto.

### <a name="interact-with-your-bot"></a>Interacción con el bot
Envíe un mensaje al bot y este enumerará los mensajes que ha recibido.
![Emulador en ejecución](../media/emulator-v4/emulator-running.png)

 
## <a name="using-cosmos-db"></a>Uso de Cosmos DB
Ahora que ha usado el almacenamiento en memoria, actualizaremos el código para usar Azure Cosmos DB. Cosmos DB es la base de datos multimodelo de distribución global de Microsoft. Azure Cosmos DB permite escalar de forma elástica e individual el rendimiento y el almacenamiento en cualquiera de las regiones geográficas de Azure. Ofrece garantía de rendimiento, latencia, disponibilidad y coherencia con Acuerdos de Nivel de Servicio (SLA) integrales. 

### <a name="set-up"></a>Instalación
Para utilizar Cosmos DB en el bot, es necesario configurar algunas cosas antes de meternos en el código.

#### <a name="create-your-database-account"></a>Creación de la cuenta de base de datos
1. En una nueva ventana del explorador, inicie sesión en [Azure Portal](http://portal.azure.com).
2. Haga clic en **Crear un recurso > Bases de datos > Azure Cosmos DB**
3. En la página **Nueva cuenta**, proporcione un nombre único en el campo **ID**. En **API**, seleccione **SQL** y proporcione la información de **Suscripción**, **Ubicación** y **Grupo de recursos**.
4. A continuación, haga clic en **Crear**.

La cuenta tarda unos minutos en crearse. Espere a que el portal muestre la página "Enhorabuena. Se ha creado su cuenta de Azure Cosmos DB".

##### <a name="add-a-collection"></a>Agregar una colección
1. Haga clic en **Configuración > Nueva colección**. El área **Agregar colección** se muestra en el extremo derecho, pero es posible que haya que desplazarse hacia la derecha para verlo. Debido a actualizaciones recientes de Cosmos DB, asegúrese de agregar una clave de partición única: _/id_. Esta clave evita errores de consultas entre particiones.

![Adición de una colección de Cosmos DB](./media/add_database_collection.png)

2. La nueva colección de base de datos, "bot-cosmos-sql-db" con el identificador de colección "bot-storage". Usaremos estos valores en el ejemplo de código siguiente.
 -
![Cosmos DB](./media/cosmos-db-sql-database.png)

3. El identificador URI del punto de conexión y la clave están disponibles en la pestaña **Claves** de la configuración de la base de datos. Necesitará estos valores para configurar el código más adelante en este artículo. 

![Claves de Cosmos DB](./media/comos-db-keys.png)

#### <a name="add-configuration-information"></a>Adición de la información de configuración
Los datos de configuración para agregar el almacenamiento de Cosmos DB son cortos y sencillos; puede agregar una configuración adicional con estos mismos métodos a medida que el bot se vuelva más complejo. En este ejemplo se usan los nombres de la base de datos y de la colección de Cosmos DB del ejemplo anterior.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**MyBot.cs**
```csharp
private const string CosmosServiceEndpoint = "<your-cosmos-db-URI>";
private const string CosmosDBKey = "<your-cosmos-db-account-key>";
private const string CosmosDBDatabaseName = "bot-cosmos-sql-db";
private const string CosmosDBCollectionName = "bot-storage";
```


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Agregue la siguiente información al archivo `.env`.

**.env**
```javascript
ACTUAL_SERVICE_ENDPOINT=<your database URI>
ACTUAL_AUTH_KEY=<your database key>
DATABASE=bot-cosmos-sql-db
COLLECTION=bot-storage
```
---

#### <a name="installing-packages"></a>Instalación de paquetes
Asegúrese de que tiene los paquetes necesarios para Cosmos DB.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Puede agregar referencias a botbuilder-azure en el proyecto mediante npm: -->


```powershell
npm install --save botbuilder-azure 
```

El bot necesita incluir un paquete adicional para usar el archivo de configuración .env. En primer lugar, obtenga el paquete de dotnet de npm:

```powershell
npm install --save dotenv
```

---

### <a name="implementation"></a>Implementación 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

El código de ejemplo siguiente se ejecuta con el mismo código de bot que el ejemplo de [almacenamiento en memoria](#memory-storage) anterior.
El siguiente fragmento de código muestra una implementación de almacenamiento de Cosmos DB para "_myStorage_", que sustituye al almacenamiento en memoria local.

**MyBot.cs**
```csharp
using Microsoft.Bot.Builder.Azure;

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
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El siguiente código de ejemplo es similar al de [almacenamiento en memoria](#memory-storage), pero con algunos pequeños cambios.

Se requiere `CosmosDbStorage` de botbuilder-azure y configurar dotenv para leer el archivo `.env`.

**index.js**
```javascript
const { CosmosDbStorage } = require("botbuilder-azure");
```
Marque como comentario el almacenamiento en memoria y reemplácelo por una referencia a Cosmos DB.

**index.js**
```javascript
// Create local Memory Storage - commented out.
// var storage = new MemoryStorage();

// Create access to Cosmos DB storage.
//Add CosmosDB 
const storage = new CosmosDbStorage({
    serviceEndpoint: process.env.ACTUAL_SERVICE_ENDPOINT, 
    authKey: process.env.ACTUAL_AUTH_KEY, 
    databaseId: process.env.DATABASE,
     collectionId: process.env.COLLECTION
})

```

---

## <a name="start-your-bot"></a>Inicio del bot
Ejecute el bot localmente.

## <a name="start-the-emulator-and-connect-your-bot"></a>Inicio del emulador y conexión del bot
A continuación, inicie el emulador y, después, conéctese al bot en el emulador:

1. Haga clic en el vínculo **Open bot** (Abrir bot) de la pestaña de bienvenida del emulador. 
2. Seleccione el archivo .bot ubicado en el directorio donde se creó el proyecto.

## <a name="interact-with-your-bot"></a>Interacción con el bot
Envíe un mensaje al bot y este enumerará los mensajes que ha recibido.
![Emulador en ejecución](../media/emulator-v4/emulator-running.png)


### <a name="view-your-data"></a>Consulta de los datos
Después de ejecutar el bot y guardar la información, podemos ver los datos almacenados en Azure Portal en la pestaña **Explorador de datos**. 

![Ejemplo del Explorador de datos](./media/data_explorer.PNG)

### <a name="manage-concurrency-using-etags"></a>Administración de la simultaneidad mediante eTags
En nuestro ejemplo de código de bot, establecemos la propiedad `eTag` de cada elemento `IStoreItem` en `*`. Cosmos DB utiliza el miembro `eTag` (etiqueta de entidad) del objeto de almacén para administrar la simultaneidad. La etiqueta `eTag` indica a la base de datos qué hacer si otra instancia del bot ha cambiado el objeto en el mismo almacenamiento en el que está escribiendo su bot. 

<!-- define optimistic concurrency -->

#### <a name="last-write-wins---allow-overwrites"></a>La última escritura tiene prioridad: permitir la sobrescritura
Un valor de propiedad `eTag` de asterisco (`*`) indica que la última escritura tiene prioridad. Al crear un nuevo almacén de datos, puede establecer la etiqueta `eTag` de una propiedad en `*` para indicar que no ha guardado previamente los datos que está escribiendo, o que desea que el último escritor sobrescriba cualquier propiedad guardada previamente. Si la simultaneidad no es un problema para el bot, establezca la propiedad `eTag` en `*` para habilitar la sobrescritura para los datos que escriba.

#### <a name="maintain-concurrency-and-prevent-overwrites"></a>Mantenimiento de la simultaneidad y evitación de la sobrescritura
Cuando almacene datos en Cosmos DB, use un valor distinto de `*` en la propiedad `eTag` si desea evitar el acceso simultáneo a una propiedad y sobrescribir los cambios de otra instancia del bot. El bot recibe una respuesta de error con el mensaje `etag conflict key=` cuando intenta guardar los datos de estado, y la etiqueta `eTag` no tiene el mismo valor que `eTag` en el almacenamiento. <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

De forma predeterminada, el almacén de Cosmos DB comprueba la igualdad de la propiedad `eTag` de un objeto de almacenamiento cada vez que un bot escribe en ese elemento y, a continuación, lo actualiza a un nuevo valor único después de cada escritura. Si la propiedad `eTag` en la escritura no coincide con la `eTag` en el almacenamiento, significa que otro subproceso o bot ha cambiado los datos. 

Por ejemplo, supongamos que desea que su bot edite una nota guardada, pero no quiere que sobrescriba los cambios que haya realizado otra instancia del bot. Si otra instancia del bot ha realizado modificaciones, lo que quiere es que el usuario edite la versión con las actualizaciones más recientes.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Primero, cree una clase que implemente `IStoreItem`.

```csharp
public class Note : IStoreItem
{
    public string Name { get; set; }
    public string Contents { get; set; }
    public string ETag { get; set; }
}
```

A continuación, cree una nota inicial mediante la creación de un objeto de almacenamiento y agregue el objeto a su almacén.

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


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Agregue una función auxiliar al final del bot que escriba una nota de ejemplo en un almacén de datos.
Primero, cree un objeto `myNoteData`.

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
    await context.sendActivity('Successful created a note.');
} catch (err) {
    await context.sendActivity(`Could not create note: ${err}`);
}
```

A continuación, para acceder y actualizar la nota más adelante, creamos otra función auxiliar a la que se puede tener acceso cuando el usuario escribe "actualizar nota".

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
            await context.sendActivity('Successfully updated to note.');
        } catch (err) {            console.log(`Write failed: ${err}`);
        }
    }
    catch (err) {
        await context.sendActivity(`Unable to read the Note: ${err}`);
    }
}
```

Si se ha actualizado la nota en el almacén antes de que haya escrito los cambios, la llamada a `write` iniciará una excepción.

---

Para mantener la simultaneidad, siempre es necesario leer una propiedad desde el almacenamiento y luego modificar la propiedad que se ha leído, de esta manera `eTag` se mantiene. Si lee datos de usuario desde el almacén, la respuesta contendrá la propiedad de eTag. Si cambia los datos y escribe datos actualizados en el almacén, la solicitud debe incluir la propiedad de eTag que especifica el mismo valor que leyó anteriormente. Sin embargo, escribir un objeto con su `eTag` establecida en `*` permitirá que la escritura sobrescriba cualquier otro cambio.

## <a name="using-blob-storage"></a>Uso de Blob Storage 
Azure Blob Storage es la solución de almacenamiento de objetos de Microsoft para la nube. Blob Storage está optimizado para el almacenamiento de cantidades masivas de datos no estructurados, como texto o datos binarios.

### <a name="create-your-blob-storage-account"></a>Creación de la cuenta de Blob Storage
Para utilizar Blob Storage en el bot, es necesario configurar algunas cosas antes de meternos en el código.
1. En una nueva ventana del explorador, inicie sesión en [Azure Portal](http://portal.azure.com).
2. Haga clic en **Crear un recurso > Almacenamiento > Cuenta de almacenamiento: blob, archivo, tabla, cola**
3. En la página **Nueva cuenta**, escriba un **Nombre** para la cuenta de almacenamiento, seleccione **Blob Storage** en **Tipo de cuenta** y proporcione la información de **Ubicación**, **Grupo de recursos** y **Suscripción**.  
4. A continuación, haga clic en **Crear**.

![Creación de un almacenamiento de blobs](./media/create-blob-storage.png)

#### <a name="add-configuration-information"></a>Adición de la información de configuración

Busque las claves de Blob Storage que necesita para configurar el almacenamiento de blobs del bot, tal y como se indicó anteriormente:
1. En Azure Portal, abra la cuenta de Blob Storage y seleccione **Configuración > Claves de acceso**.

![Búsqueda de las claves de Blob Storage](./media/find-blob-storage-keys.png)

Ahora usaremos dos de estas claves para proporcionar acceso a nuestro código a la cuenta de Blob Storage.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder.Azure;
```
Actualice la línea de código que apunta "_myStorage_" a la cuenta de Blob Storage existente.

```csharp


private static readonly AzureBlobStorage _myStorage = new AzureBlobStorage("<your-blob-storage-account-string>", "<your-blob-storage-container-name>");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
```javascript
const mystorage = new BlobStorage({
   <youy_containerName>,
   <your_storageAccountOrConnectionString>,
   <your_storageAccessKey>
})
```
---

Una vez que "_myStorage_" se establece para que apunte a la cuenta de Blob Storage, el código del bot almacena y recupera los datos desde Blob Storage.

## <a name="start-your-bot"></a>Inicio del bot
Ejecute el bot localmente.

## <a name="start-the-emulator-and-connect-your-bot"></a>Inicio del emulador y conexión del bot
A continuación, inicie el emulador y, después, conéctese al bot en el emulador:

1. Haga clic en el vínculo **Open bot** (Abrir bot) de la pestaña de bienvenida del emulador. 
2. Seleccione el archivo .bot ubicado en el directorio donde se creó el proyecto.

## <a name="interact-with-your-bot"></a>Interacción con el bot
Envíe un mensaje al bot y este enumerará los mensajes que ha recibido.
![Emulador en ejecución](../media/emulator-v4/emulator-running.png)

### <a name="view-your-data"></a>Consulta de los datos
Después de ejecutar el bot y guardar la información, lo podemos ver en la pestaña **Explorador de Storage** de Azure Portal.

## <a name="blob-transcript-storage"></a>Almacenamiento de transcripciones de blobs
El almacenamiento de transcripciones de blobs de Azure ofrece una opción de almacenamiento especializado que le permite guardar y recuperar con facilidad las conversaciones del usuario en forma de una transcripción grabada. El almacenamiento de transcripciones de blobs de Azure es especialmente útil para capturar automáticamente las entradas de usuario para examinarlas durante la depuración del rendimiento del bot.

### <a name="set-up"></a>Instalación
El almacenamiento de transcripciones de blobs de Azure puede usar la misma cuenta de Blob Storage que creó siguiendo los pasos detallados en las secciones "_Creación de la cuenta de Blob Storage_" y "_Adición de la información de configuración_" anteriores. Para este análisis, hemos agregado un nuevo contenedor de blobs, "_mybottranscripts_". 

### <a name="implementation"></a>Implementación 
El siguiente código conecta el puntero de almacenamiento de transcripciones "_transcriptStore_" a la nueva cuenta de almacenamiento de transcripciones de blobs de Azure. El código fuente para almacenar las conversaciones de usuario mostrado aquí se basa en el [historial de conversaciones](https://aka.ms/bot-history-sample-code) de ejemplo. 

```csharp
// In Startup.cs
using Microsoft.Bot.Builder.Azure;

// Enable the conversation transcript middleware.
blobStore = new AzureBlobTranscriptStore(blobStorageConfig.ConnectionString, storageContainer);
var transcriptMiddleware = new TranscriptLoggerMiddleware(blobStore);
options.Middleware.Add(transcriptMiddleware);

// In ConversationHistoryBot.cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Schema;

private readonly AzureBlobTranscriptStore _transcriptStore;

/// <param name="transcriptStore">Injected via ASP.NET dependency injection.</param>
public ConversationHistoryBot(AzureBlobTranscriptStore transcriptStore)
{
    _transcriptStore = transcriptStore ?? throw new ArgumentNullException(nameof(transcriptStore));
}

```

### <a name="store-user-conversations-in-azure-blob-transcripts"></a>Almacenamiento de las conversaciones del usuario en transcripciones de blobs de Azure
Después de tener disponible un contenedor de blobs para almacenar las transcripciones, puede comenzar a conservar las conversaciones de los usuarios con el bot. Estas conversaciones se pueden utilizar más adelante como una herramienta de depuración para ver cómo interactúan los usuarios con el bot. El siguiente código conserva las entradas de conversación del usuario cuando activity.text recibe el mensaje de entrada _!history_.


```csharp
/// <summary>
/// Every Conversation turn for our EchoBot will call this method. 
/// </summary>
/// <param name="turnContext">A <see cref="ITurnContext"/> containing all the data needed
/// for processing this conversation turn. </param>        
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    var activity = turnContext.Activity;
    if (activity.Type == ActivityTypes.Message)
    {
        if (activity.Text == "!history")
        {
           // Download the activities from the Transcript (blob store) when a request to upload history arrives.
           var connectorClient = turnContext.TurnState.Get<ConnectorClient>(typeof(IConnectorClient).FullName);
           // Get all the message type activities from the Transcript.
           string continuationToken = null;
           var count = 0;
           do
           {
               var pagedTranscript = await _transcriptStore.GetTranscriptActivitiesAsync(activity.ChannelId, activity.Conversation.Id, continuationToken);
               var activities = pagedTranscript.Items
                  .Where(a => a.Type == ActivityTypes.Message)
                  .Select(ia => (Activity)ia)
                  .ToList();
               
               var transcript = new Transcript(activities);

               await connectorClient.Conversations.SendConversationHistoryAsync(activity.Conversation.Id, transcript, cancellationToken: cancellationToken);

               continuationToken = pagedTranscript.ContinuationToken;
           }
           while (continuationToken != null);
```

### <a name="find-all-stored-transcripts-for-your-channel"></a>Búsqueda de todas las transcripciones almacenadas del canal
Para ver qué datos se han almacenado, puede usar el código siguiente para buscar mediante programación los elementos "_ConversationID_" de todas las transcripciones que ha almacenado. Al usar el emulador para probar el código, seleccione "_Start Over_" (Volver a empezar) para comenzar una transcripción nueva con un nuevo "_ConversationID_".

```csharp
List<string> storedTranscripts = new List<string>();
PagedResult<Transcript> pagedResult = null;
var pageSize = 0;
do
{
    pagedResult = await _transcriptStore.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
    
    // transcript item contains ChannelId, Created, Id.
    // save the converasationIds (Id) found by "ListTranscriptsAsync" to a local list.
    foreach (var item in pagedResult.Items)
    {
         // Make sure we store an unescaped conversationId string.
         var strConversationId = item.Id;
         storedTranscripts.Add(Uri.UnescapeDataString(strConversationId));
    }
} while (pagedResult.ContinuationToken != null);
```

### <a name="retrieve-user-conversations-from-azure-blob-transcript-storage"></a>Recuperación de las conversaciones del usuario desde el almacenamiento de transcripciones de blobs de Azure
Una vez que las transcripciones de interacción del bot se han almacenado en el almacén de transcripciones de blobs de Azure, se pueden recuperar mediante programación para realizar pruebas o depuración mediante el método de AzureBlobTranscriptStorage, "_GetTranscriptActivities_". El fragmento de código siguiente recupera todas las transcripciones de entradas de usuario que se han recibido y almacenado en las últimas 24 horas de cada transcripción almacenada.


```csharp
var numTranscripts = storedTranscripts.Count();
for (int i = 0; i < numTranscripts; i++)
{
    PagedResult<IActivity> pagedActivities = null;
    do
    {
        string thisConversationId = storedTranscripts[i];
        // Find all inputs in the last 24 hours.
        DateTime yesterday = DateTime.Now.AddDays(-1);
        // Retrieve iActivities for this transcript.
        pagedActivities = await _myTranscripts.GetTranscriptActivitiesAsync("emulator", thisConversationId, pagedActivities?.ContinuationToken, yesterday);
        foreach (var item in pagedActivities.Items)
        {
            // View as message and find value for key "text" :
            var thisMessage = item.AsMessageActivity();
            var userInput = thisMessage.Text;
         }
    } while (pagedActivities.ContinuationToken != null);
}
```

### <a name="remove-stored-transcripts-from-azure-blob-transcript-storage"></a>Eliminación de las transcripciones almacenadas del almacenamiento de transcripciones de blobs de Azure
Una vez que haya terminado de utilizar los datos de entrada de usuario para probar o depurar el bot, las transcripciones almacenadas se pueden eliminar mediante programación del almacén de transcripciones de blobs de Azure. El siguiente fragmento de código elimina todas las transcripciones almacenadas del almacén de transcripciones del bot.


```csharp
for (int i = 0; i < numTranscripts; i++)
{
   // Remove all stored transcripts except the last one found.
   if (i > 0)
   {
       string thisConversationId = storedTranscripts[i];    
       await _transcriptStore.DeleteTranscriptAsync("emulator", thisConversationId);
    }
}
```


El vínculo siguiente proporciona más información sobre el [almacenamiento de transcripciones de blobs de Azure](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.azure.azureblobtranscriptstore)  

## <a name="next-steps"></a>Pasos siguientes
Ahora que sabe cómo leer lectura y escritura directamente desde el almacenamiento, eche un vistazo a cómo puede usar el Administrador de estado para que lo haga por usted.

> [!div class="nextstepaction"]
> [Guardar el estado mediante las propiedades de usuario y de conversación](bot-builder-howto-v4-state.md)


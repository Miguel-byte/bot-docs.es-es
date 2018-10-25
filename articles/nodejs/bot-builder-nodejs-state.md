---
title: Administración de datos de estado | Microsoft Docs
description: Obtenga información sobre cómo guardar y recuperar datos de estado con Bot Builder SDK para Node.js.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 519f188a29db2b9b37061ee0eff5361a63d2acac
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999992"
---
# <a name="manage-state-data"></a>Administración de datos de estado

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-state.md)
> - [Node.js](../nodejs/bot-builder-nodejs-state.md)

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

## <a name="in-memory-data-storage"></a>Almacenamiento de datos en memoria

El almacenamiento de datos en memoria solo está pensado para realizar pruebas. Este tipo de almacenamiento es volátil y temporal. Los datos se borran cada vez que se reinicia el bot. Para usar el almacenamiento en memoria con fines de prueba, deberá hacer dos cosas. En primer lugar, cree una nueva instancia del almacenamiento en memoria:

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
```

A continuación, establézcalo en el bot cuando cree el **UniversalBot**:

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
var bot = new builder.UniversalBot(connector, [..waterfall steps..])
                    .set('storage', inMemoryStorage); // Register in-memory storage 
```

Puede usar este método para establecer su propio almacenamiento de datos personalizado o usar cualquiera de las *extensiones de Azure*.

## <a name="manage-custom-data-storage"></a>Administración del almacenamiento de datos personalizado

Por motivos de seguridad y rendimiento en el entorno de producción, puede implementar su propio almacenamiento de datos o considerar la posibilidad de implementar una de las siguientes opciones de almacenamiento de datos:

1. [Administración de datos de estado con Cosmos DB](bot-builder-nodejs-state-azure-cosmosdb.md)

2. [Administración de datos de estado con Table Storage](bot-builder-nodejs-state-azure-table-storage.md)

Con cualquiera de estas opciones de las [extensiones de Azure](https://www.npmjs.com/package/botbuilder-azure), el mecanismo para establecer y conservar los datos con Bot Framework SDK para Node.js es el mismo que el del almacenamiento de datos en memoria.

## <a name="storage-containers"></a>Contenedores de almacenamiento

En Bot Builder SDK para Node.js, el objeto `session` expone las siguientes propiedades para almacenar datos de estado.

| Propiedad | Ámbito | DESCRIPCIÓN |
| ---- | ---- | ---- |
| [`userData`][userDataURL] | Usuario | Contiene datos que se guardan para el usuario en el canal especificado. Estos datos se conservan a lo largo de varias conversaciones. |
| [`privateConversationData`][privateConversationDataURL] | Conversation | Contiene datos que se guardan para el usuario en el contexto de una conversación determinada en el canal especificado. Estos datos son privados para el usuario actual y se conservarán únicamente para la conversación actual. La propiedad se borra cuando la conversación finaliza o se llama explícitamente a `endConversation`. |
| [`conversationData`][conversationDataURL] | Conversation | Contiene datos que se guardan en el contexto de una conversación determinada en el canal especificado. Estos datos se comparten con todos los usuarios que participan en la conversación y se conservarán únicamente para la conversación actual. La propiedad se borra cuando la conversación finaliza o se llama explícitamente a `endConversation`. |
| [`dialogData`][dialogDataURL] | Diálogo | Contiene datos que se guardan únicamente para el diálogo actual. Cada diálogo mantiene su propia copia de esta propiedad. La propiedad se borra cuando el diálogo se elimina de la pila de diálogos. |

Estas cuatro propiedades corresponden a los cuatro contenedores de almacenamiento de datos que se pueden usar para almacenar los datos. Las propiedades que use para almacenar los datos dependerán del ámbito adecuado para los datos que almacena, la naturaleza de los mismos y cuánto tiempo desea conservar los datos. Por ejemplo, si necesita almacenar datos de usuario que estarán disponibles a lo largo de varias conversaciones, considere el uso de la propiedad `userData`. Si necesita almacenar temporalmente los valores de las variables locales dentro del ámbito de un diálogo, considere el uso de la propiedad `dialogData`. Si necesita almacenar temporalmente datos que deben ser accesibles a lo largo de varios diálogos, considere el uso de la propiedad `conversationData`.

## <a name="data-persistence"></a>Persistencia de los datos

De forma predeterminada, los datos que se almacenan utilizando las propiedades `userData`, `privateConversationData` y `conversationData` se conservan después de que finalice la conversación. Si no desea que los datos se conserven en el contenedor `userData`, establezca la marca `persistUserData` en **false**. Si no desea que los datos se conserven en el contenedor `conversationData`, establezca la marca `persistConversationData` en **false**. 

```javascript
// Do not persist userData
bot.set(`persistUserData`, false);

// Do not persist conversationData
bot.set(`persistConversationData`, false);
```

> [!NOTE]
> No se puede deshabilitar la persistencia de datos para el contenedor `privateConversationData`; siempre se conservan.

## <a name="set-data"></a>Establecimiento de los datos

Puede almacenar objetos JavaScript simples si los guarda directamente en un contenedor de almacenamiento. Para un objeto complejo como `Date`, considere convertirlo a `string`. Esto es porque los datos de estado se serializan y se almacenan como JSON. Los ejemplos de código siguientes muestran cómo almacenar datos primitivos, una matriz, un mapa de objetos y un objeto `Date` complejo. 

**Almacenamiento de datos primitivos**

```javascript
session.userData.userName = "Kumar Sarma";
session.userData.userAge = 37;
session.userData.hasChildren = true;
```

**Almacenamiento de una matriz**

```javascript
session.userData.profile = ["Kumar Sarma", "37", "true"];
```

**Almacenamiento de un mapa de objetos**

```javascript
session.userData.about = {
    "Profile": {
        "Name": "Kumar Sarma",
        "Age": 37,
        "hasChildren": true
    },
    "Job": {
        "Company": "Contoso",
        "StartDate": "June 8th, 2010",
        "Title": "Developer"
    }
}
```
**Almacenamiento de fecha y hora** 

Para un objeto JavaScript complejo, debe convertirlo en una cadena antes de guardarlo en el contenedor de almacenamiento. 

```javascript 
var startDate = builder.EntityRecognizer.resolveTime([results.response]); 

// Date as string: "2017-08-23T05:00:00.000Z" 
session.userdata.start = startDate.toISOString(); 
``` 

### <a name="saving-data"></a>Guardar datos

Los datos que se crean en cada contenedor de almacenamiento permanecerán en memoria hasta que se guarde el contenedor. Bot Builder SDK para Node.js envía los datos al servicio `ChatConnector` por lotes para que se guarden cuando haya mensajes que enviar. Para guardar los datos que existan en los contenedores de almacenamiento sin enviar ningún mensaje, puede llamar manualmente al método [`save`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#save). Si no llama al método `save`, los datos que existan en los contenedores de almacenamiento se guardarán como parte del procesamiento por lotes.

```javascript
session.userData.favoriteColor = "Red";
session.userData.about.job.Title = "Senior Developer"; 
session.save();
```

## <a name="get-data"></a>Obtener los datos

Para acceder a los datos guardados en un contenedor de almacenamiento determinado, basta con hacer referencia a la propiedad correspondiente. Los ejemplos de código siguientes muestran cómo acceder a datos que se almacenaron como datos primitivos, una matriz, un mapa de objetos y un objeto Date complejo.

**Acceso a datos primitivos**

```javascript
var userName = session.userData.userName;
var userAge = session.userData.userAge;
var hasChildren = session.userData.hasChildren;
```

**Acceso a una matriz**

```javascript
var userProfile = session.userData.userProfile;

session.send("User Profile:");
for(int i = 0; i < userProfile.length, i++){
    session.send(userProfile[i]);
}
```

**Acceso a un mapa de objetos**

```javascript
var about = session.userData.about;

session.send("User %s works at %s.", about.Profile.Name, about.Job.Company);
```

**Acceso a un objeto Date** 

Recupere los datos de fecha como una cadena y, a continuación, conviértala en un objeto Date de JavaScript. 

```javascript 
// startDate as a JavaScript Date object. 
var startDate = new Date(session.userdata.start); 
``` 

## <a name="delete-data"></a>Eliminación de datos

De forma predeterminada, los datos que se almacenan en el contenedor `dialogData` se borran cuando se elimina un diálogo de la pila de diálogos. Del mismo modo, los datos que se almacenan en los contenedores `conversationData` y `privateConversationData` se borran cuando se llama al método `endConversation`. Sin embargo, para eliminar los datos almacenados en el contenedor `userData`, debe eliminarlo explícitamente.

Para borrar explícitamente los datos almacenados en cualquiera de los contenedores de almacenamiento, simplemente restablezca el contenedor como se muestra en el siguiente ejemplo de código. 

```javascript
// Clears data stored in container.
session.userData = {}; 
session.privateConversationData = {};
session.conversationData = {};
session.dialogData = {};
```

No establezca nunca un contenedor de datos en `null` ni lo elimine del objeto `session`, ya que hacerlo provocará errores la próxima vez que intente acceder al contenedor. Además, es posible que desee llamar manualmente a `session.save();` después de borrar manualmente un contenedor en memoria para borrar los datos correspondientes que se han guardado anteriormente.

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe cómo administrar los datos de estado de usuario, echemos un vistazo a cómo puede usarlos para administrar mejor el flujo de conversación.

> [!div class="nextstepaction"]
> [Administración del flujo de conversación](bot-builder-nodejs-dialog-manage-conversation-flow.md)

## <a name="additional-resources"></a>Recursos adicionales
- [Petición de datos de entrada al usuario](bot-builder-nodejs-dialog-prompt.md)

[userDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata
[conversationDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#conversationdata
[privateConversationDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#privateconversationdata
[dialogDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#dialogdata

[ChatConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html

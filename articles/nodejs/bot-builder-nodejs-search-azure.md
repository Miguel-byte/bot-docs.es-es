---
title: Creación de experiencias controladas por datos con Azure Search | Microsoft Docs
description: Obtenga información sobre cómo crear experiencias controladas por datos con Azure Search y ayude a los usuarios a navegar por grandes cantidades de contenido en un bot con el SDK de Bot Builder para Node.js y Azure Search.
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 204fb5d8f4838c78d771bfad5c0ed6511b27932b
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000002"
---
# <a name="create-data-driven-experiences-with-azure-search"></a>Creación de experiencias controladas por datos con Azure Search 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-search-azure.md)
> - [Node.js](../nodejs/bot-builder-nodejs-search-azure.md)

Puede agregar [Azure Search][search] al bot para ayudar al usuario a navegar por grandes cantidades de contenido y crear una experiencia de exploración controlada por datos para los usuarios del bot.

Azure Search es un servicio de Azure que ofrece búsqueda de palabras clave, lingüística integrada, puntuación personalizada, navegación por facetas y mucho más. Azure Search también puede indexar el contenido desde diversos orígenes, incluidos Azure SQL DB, DocumentDB, Blob Storage y Table Storage. Es compatible con la indexación "push" para otros orígenes de datos y puede abrir archivos PDF, documentos de Office y otros formatos que contengan datos no estructurados. Una vez recopilado, el contenido se coloca en un índice de Azure Search, al que luego el bot puede consultar.

## <a name="install-dependencies"></a>Instalación de dependencias

Desde un símbolo del sistema, vaya al directorio del proyecto del bot e instale los módulos siguientes con el Administrador de paquetes para Node (NPM):

* [bluebird](https://www.npmjs.com/package/bluebird)
* [lodash](https://www.npmjs.com/package/lodash)
* [Solicitud](https://www.npmjs.com/package/request)

## <a name="prerequisites"></a>Requisitos previos

Se **requiere** lo siguiente: 
- Una suscripción de Azure y una clave principal de Azure Search. Puede encontrarlo en Azure Portal.
- Copiar la biblioteca [SearchDialogLibrary](https://github.com/Microsoft/botBuilder-Samples/tree/master/Node/demo-Search/SearchDialogLibrary) en el directorio del proyecto del bot. Esta biblioteca contiene diálogos generales para la búsqueda del usuario, pero se pueden personalizar según sea necesario para adaptarlos al bot. 

- Copiar la biblioteca [SearchProviders](https://github.com/Microsoft/botBuilder-Samples/tree/master/Node/demo-Search/SearchProviders) en el directorio del proyecto del bot. Esta biblioteca contiene todos los componentes necesarios para crear una solicitud y enviarla a Azure Search.

## <a name="connect-to-the-azure-service"></a>Conexión al servicio de Azure 

En el archivo de programa principal del bot (por ejemplo, app.js), cree las rutas de acceso de referencia a las dos bibliotecas que instaló anteriormente. 

```javascript
var SearchLibrary = require('./SearchDialogLibrary');
var AzureSearch = require('./SearchProviders/azure-search');
```

Agregue el código de ejemplo siguiente al bot. En el objeto `AzureSearch`, pase su propia configuración de Azure Search al método `.create`. En tiempo de ejecución, esto enlazará el bot con el servicio de Azure Search y esperará una consulta de usuario completada en forma de `Promise`.  

```javascript
// Azure Search
var azureSearchClient = AzureSearch.create('Your-Azure-Search-Service-Name', 'Your-Azure-Search-Primary-Key', 'Your-Azure-Search-Service-Index');
var ResultsMapper = SearchLibrary.defaultResultsMapper(ToSearchHit);
```

 La referencia `azureSearchClient` crea el modelo de Azure Search, que pasa la configuración de autorización del servicio de Azure desde la configuración `.env` del bot. 
 `ResultsMapper` analiza el objeto de respuesta de Azure y asigna los datos tal como se definen en el método `ToSearchHit`. Para ver un ejemplo de implementación de este método, consulte [Después de que Azure Search responde](#after-azure-search-responds).

## <a name="register-the-search-library"></a>Registro de la biblioteca de búsqueda
Puede personalizar los diálogos de búsqueda directamente en el propio módulo `SearchLibrary`. `SearchLibrary` realiza la mayoría del trabajo pesado, incluida la llamada a Azure Search. 

Agregue el código siguiente en el archivo de programa principal del bot para registrar la biblioteca de diálogos de búsqueda con el bot. 

```javascript
bot.library(SearchLibrary.create({
    multipleSelection: true,
    search: function (query) { return azureSearchClient.search(query).then(ResultsMapper); },
    refiners: ['refiner1', 'refiner2', 'refiner3'], // customize your own refiners 
    refineFormatter: function (refiners) {
        return _.zipObject(
            refiners.map(function (r) { return 'By ' + _.capitalize(r); }),
            refiners);
    }
}));
```
`SearchLibrary` no solo almacena todos los diálogos relacionados con la búsqueda, sino que también toma la consulta del usuario para enviarla a Azure Search. Deberá definir sus propios refinadores en la matriz `refiners` para especificar las entidades que desea permitir que el usuario restrinja o filtre los resultados de la búsqueda.  

## <a name="create-a-search-dialog"></a>Creación de un diálogo de búsqueda

Puede elegir estructurar los diálogos como quiera. El único requisito para configurar un diálogo de Azure Search es invocar el método `.begin` del objeto `SearchLibrary`, pasando el objeto `session` generado por el SDK de Bot Builder. 

```javascript
function (session) {
        // Trigger Azure Search dialogs 
        SearchLibrary.begin(session);
    },
    function (session, args) {
        // Process selected search results
        session.send(
            'Search Completed!',
            args.selection.map(  ); // format your response 
    }
```
Para más información sobre los diálogos, consulte [Manage a conversation with dialogs](bot-builder-nodejs-dialog-manage-conversation.md) (Administración de una conversación con diálogos).

## <a name="after-azure-search-responds"></a>Después de que Azure Search responde 

Una vez que se resuelve una instancia correcta de Azure Search, ahora debe almacenar los datos que quiere desde el objeto de respuesta y mostrarlos de manera significativa para el usuario.

> [!TIP]
> Considere incluir el [módulo util][NodeUtil]. Lo ayudará a dar formato a la respuesta de Azure Search y a asignarla.

En el archivo de programa principal del bot, cree un método `ToSearchHit`. Este método devuelve un objeto que da formato a los datos relevantes que necesita de la respuesta de Azure. El código siguiente muestra cómo puede definir sus propios parámetros en el método `ToSearchHit`. 
 
 ```javascript
 function ToSearchHit(azureResponse) {
     return {
         // define your own parameters 
         key: azureResponse.id,
         title: azureResponse.title,
         description: azureResponse.description,
         imageUrl: azureResponse.thumbnail
     };
 }
```
Una vez hecho esto, todo lo que tiene que hacer es mostrar los datos al usuario. 

 En el archivo **index.js** del proyecto **SearchDialogLibrary**, el método `searchHitAsCard` analiza cada respuesta de Azure Search y crea un nuevo objeto de tarjeta para mostrar al usuario. Los campos definidos en el método `ToSearchHit` desde el archivo de programa principal del bot se debe sincronizar con las propiedades en el método `searchHitAsCard`. 

A continuación se muestra cómo y dónde se usan los parámetros definidos del método `ToSearchHit` para generar una interfaz de usuario de los datos adjuntos de la tarjeta para presentarlos al usuario. 

```javascript
function searchHitAsCard(showSave, searchHit) {
    var buttons = showSave
        ? [new builder.CardAction().type('imBack').title('Save').value(searchHit.key)]
        : [];

    var card = new builder.HeroCard()
        .title(searchHit.title) 
        .buttons(buttons);

    if (searchHit.description) {
        card.subtitle(searchHit.description);
    }

    if (searchHit.imageUrl) {
        card.images([new builder.CardImage().url(searchHit.imageUrl)]);
    }

    return card;
}
```

## <a name="sample-code"></a>Código de ejemplo

Para ver dos ejemplos completos que muestran cómo admitir Azure Search con bots usando el SDK de Bot Builder para Node.js, consulte el [ejemplo de bot de bienes inmuebles](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-Search/RealEstateBot) o el [ejemplo de bot de listas de trabajos](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-Search/JobListingBot) en GitHub. 

## <a name="additional-resources"></a>Recursos adicionales

* [Azure Search][search]
* [Nodo Util][NodeUtil]
* [Diálogos](bot-builder-nodejs-dialog-manage-conversation.md)

[NodeUtil]: https://nodejs.org/api/util.html
[search]: /azure/search/search-what-is-azure-search

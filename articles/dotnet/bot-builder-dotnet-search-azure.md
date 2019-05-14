---
title: Creación de experiencias controladas por datos con Azure Search | Microsoft Docs
description: Aprenda a crear experiencias controladas por datos con Azure Search y ayude a los usuarios a explorar grandes cantidades de contenido en un bot con Bot Framework SDK para .NET y Azure Search.
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 1/28/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e0ffb9c24b5e85b0eb1afdd885654e4864e65939
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032920"
---
# <a name="create-data-driven-experiences-with-azure-search"></a>Creación de experiencias controladas por datos con Azure Search 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-search-azure.md)
> - [Node.js](../nodejs/bot-builder-nodejs-search-azure.md)

Puede agregar [Azure Search](https://azure.microsoft.com/en-us/services/search/) a un bot para ayudar a los usuarios a navegar por grandes cantidades de contenido y crear una experiencia de exploración controlada por datos.

Azure Search es un servicio de Azure que ofrece búsqueda de palabras clave, lingüística integrada, puntuación personalizada, navegación por facetas y mucho más. Azure Search también puede indexar el contenido desde diversos orígenes, incluidos Azure SQL DB, DocumentDB, Blob Storage y Table Storage. Es compatible con la indexación "push" para otros orígenes de datos y puede abrir archivos PDF, documentos de Office y otros formatos que contengan datos no estructurados. Una vez recopilado, el contenido se coloca en un índice de Azure Search, al que luego el bot puede consultar.

## <a name="prerequisites"></a>Requisitos previos

Instale el paquete de Nuget [Microsoft.Azure.Search](https://www.nuget.org/packages/Microsoft.Azure.Search/4.0.0-preview) en el proyecto de bot.

Los siguientes tres proyectos de C# son necesarios en la solución del bot. Estos proyectos proporcionan funcionalidad adicional para los bots y Azure Search. Bifurque los proyectos de [GitHub](https://aka.ms/v3-cs-search-demo) o descargue el código fuente directamente.

- El proyecto **Search.Azure** define la llamada al servicio de Azure.
- El proyecto **Search.Contracts** define los modelos de datos e interfaces genéricos para controlar los datos.
- El proyecto **Search.Dialogs** incluye diversos cuadros de diálogo genéricos de Bot Builder usados para consultas en Azure Search.

## <a name="configure-azure-search-settings"></a>Configuración de las opciones de Azure Search

Configure las opciones de Azure Search en el archivo **Web.config** del proyecto con sus propias credenciales de Azure Search en los campos de valor. El constructor en la clase `AzureSearchClient` usará estos valores para registrar y enlazar el bot con el servicio de Azure.

```xml
<appSettings>
    <add key="SearchDialogsServiceName" value="Azure-Search-Service-Name" /> <!-- replace value field with Azure Service Name --> 
    <add key="SearchDialogsServiceKey" value="Azure-Search-Service-Primary-Key" /> <!-- replace value field with Azure Service Key --> 
    <add key="SearchDialogsIndexName" value="Azure-Search-Service-Index" /> <!-- replace value field with your Azure Search Index --> 
</appSettings>
```

## <a name="create-a-search-dialog"></a>Creación de un cuadro de diálogo de búsqueda

En el proyecto del bot, cree una nueva clase `AzureSearchDialog` para llamar al servicio de Azure en el bot. Esta nueva clase debe heredar la clase `SearchDialog` del proyecto **Search.Dialogs**, que controla la mayoría del trabajo pesado. La invalidación `GetTopRefiners()` permite a los usuarios afinar o filtrar los resultados de la búsqueda sin tener que volver a iniciar la búsqueda desde el principio, lo que mantiene el estado del objeto de búsqueda. Puede agregar sus propios refinadores personalizados en la matriz `TopRefiners` para permitir que los usuarios filtren o restrinjan los resultados de la búsqueda. 

```cs
[Serializable]
public class AzureSearchDialog : SearchDialog
{
    private static readonly string[] TopRefiners = { "refiner1", "refiner2", "refiner3" }; // define your own custom refiners 

    public AzureSearchDialog(ISearchClient searchClient) : base(searchClient, multipleSelection: true)
    {
    }

    protected override string[] GetTopRefiners()
    {
        return TopRefiners;
    }
}
```

## <a name="define-the-response-data-model"></a>Definición del modelo de datos de respuesta

La clase **SearchHit.cs** dentro del proyecto `Search.Contracts` define los datos pertinentes que deben analizarse a partir de la respuesta de Azure Search. Para el bot, las únicas inclusiones obligatorias son la declaración de IDictionary `PropertyBag` y la creación en el constructor. Puede definir todas las demás propiedades de esta clase en relación con las necesidades de su bot. 

```cs
[Serializable]
public class SearchHit
{
    public SearchHit()
    {
        this.PropertyBag = new Dictionary<string, object>();
    }

    public IDictionary<string, object> PropertyBag { get; set; }

    // customize the fields below as needed 
    public string Key { get; set; }

    public string Title { get; set; }

    public string PictureUrl { get; set; }

    public string Description { get; set; }
}
```

## <a name="after-azure-search-responds"></a>Después de que Azure Search responde 

Tras una consulta correcta al servicio de Azure, el resultado de la búsqueda deberá analizarse para recuperar los datos pertinentes para que el bot los muestre al usuario. Para habilitar esta opción, deberá crear una clase `SearchResultMapper`. El objeto `GenericSearchResult` creado en el constructor define una lista y diccionario para almacenar los resultados y las facetas, respectivamente, después de que cada resultado se analice con respecto a los modelos de datos de su bot. 

Sincronice las propiedades en el método `ToSearchHit` para que coincidan con el modelo de datos en **SearchHit.cs**. El método `ToSearchHit` se ejecutará y generará un nuevo `SearchHit` para cada resultado que se encuentre en la respuesta.  

```cs
public class SearchResultMapper : IMapper<DocumentSearchResult, GenericSearchResult>
{
    public GenericSearchResult Map(DocumentSearchResult documentSearchResult)
    {
        var searchResult = new GenericSearchResult();

        searchResult.Results = documentSearchResult.Results.Select(r => ToSearchHit(r)).ToList();
        searchResult.Facets = documentSearchResult.Facets?.ToDictionary(kv => kv.Key, kv => kv.Value.Select(f => ToFacet(f)));

        return searchResult;
    }

    private static GenericFacet ToFacet(FacetResult facetResult)
    {
        return new GenericFacet
        {
            Value = facetResult.Value,
            Count = facetResult.Count.Value
        };
    }

    private static SearchHit ToSearchHit(SearchResult hit)
    {
        return new SearchHit
        {
            // custom properties defined in SearchHit.cs 
            Key = (string)hit.Document["id"],
            Title = (string)hit.Document["title"],
            PictureUrl = (string)hit.Document["thumbnail"],
            Description = (string)hit.Document["description"]
        };
    }
}
```
Una vez que se analicen y se almacenen los resultados, la información deberá mostrarse al usuario. La clase `SearchHitStyler` deberá administrarse para que se adapte al modelo de datos de la clase `SearchHit`. Por ejemplo, las propiedades `Title`, `PictureUrl` y `Description` de la clase **SearchHit.cs** se utilizan en el código de ejemplo para crear adjuntos de una nueva tarjeta. El código siguiente crea un nuevo archivo adjunto de tarjeta para cada objeto `SearchHit` en la lista de resultados `GenericSearchResult` para mostrar al usuario.   

```cs
[Serializable]
public class SearchHitStyler : PromptStyler
{
    public override void Apply<T>(ref IMessageActivity message, string prompt, IReadOnlyList<T> options, IReadOnlyList<string> descriptions = null)
    {
        var hits = options as IList<SearchHit>;
        if (hits != null)
        {
            var cards = hits.Select(h => new ThumbnailCard
            {
                Title = h.Title,
                Images = new[] { new CardImage(h.PictureUrl) },
                Buttons = new[] { new CardAction(ActionTypes.ImBack, "Pick this one", value: h.Key) },
                Text = h.Description
            });

            message.AttachmentLayout = AttachmentLayoutTypes.Carousel;
            message.Attachments = cards.Select(c => c.ToAttachment()).ToList();
            message.Text = prompt;
        }
        else
        {
            base.Apply<T>(ref message, prompt, options, descriptions);
        }
    }
}
```
Los resultados de la búsqueda se muestran al usuario, y Azure Search se habrá agregado correctamente al bot.

## <a name="samples"></a>Ejemplos

Para obtener dos ejemplos completos que muestran cómo admitir Azure Search con bots que utilizan Bot Framework SDK para. NET, consulte [Ejemplo de bot de bienes inmuebles](https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/demo-Search/RealEstateBot) o [Ejemplo de bot de listas de trabajos](https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/demo-Search/JobListingBot) en GitHub. 

## <a name="additional-resources"></a>Recursos adicionales

- [Azure Search][search]
- [Introducción a los diálogos](bot-builder-dotnet-dialogs.md)

[search]: /azure/search/search-what-is-azure-search

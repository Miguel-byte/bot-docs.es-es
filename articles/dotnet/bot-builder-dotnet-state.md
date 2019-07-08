---
title: Administración de datos de estado | Microsoft Docs
description: Obtenga información sobre cómo guardar y recuperar datos de estado con Bot Framework SDK para .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: fc4fe669c57c53b8b53a98bf495ed858094ef4b6
ms.sourcegitcommit: 697a577d72aaf91a0834d4b4c2ef5aa11291f28f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/01/2019
ms.locfileid: "67496632"
---
# <a name="manage-state-data"></a>Administración de datos de estado

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-state.md)
> - [Node.js](../nodejs/bot-builder-nodejs-state.md)

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

## <a name="in-memory-data-storage"></a>Almacenamiento de datos en memoria

El almacenamiento de datos en memoria solo está pensado para realizar pruebas. Este tipo de almacenamiento es volátil y temporal. Los datos se borran cada vez que se reinicia el bot. Para usar el almacenamiento en la memoria para hacer pruebas, necesitará: 

Instale los siguientes paquetes NuGet: 
- Microsoft.Bot.Builder.Azure
- Autofac.WebApi2

En el método **Application_Start**, cree una nueva instancia del almacenamiento en memoria y registre el nuevo almacén de datos:

```cs
// Global.asax file

var store = new InMemoryDataStore();

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
GlobalConfiguration.Configure(WebApiConfig.Register);

```

Puede usar este método para establecer su propio almacenamiento de datos personalizado o usar cualquiera de las *extensiones de Azure*.

## <a name="manage-custom-data-storage"></a>Administración del almacenamiento de datos personalizado

Por motivos de seguridad y rendimiento en el entorno de producción, puede implementar su propio almacenamiento de datos o considerar la posibilidad de implementar una de las siguientes opciones de almacenamiento de datos:

1. [Administración de datos de estado con Cosmos DB](bot-builder-dotnet-state-azure-cosmosdb.md)

2. [Administración de datos de estado con Table Storage](bot-builder-dotnet-state-azure-table-storage.md)

Con cualquiera de estas opciones de las [extensiones de Azure](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/), el mecanismo para establecer y conservar los datos a través del SDK de Bot Framework para .NET es el mismo que el del almacenamiento de datos en memoria.

## <a name="bot-state-methods"></a>Métodos de estado del bot

En esta tabla se enumeran los métodos que se pueden usar para administrar los datos de estado.

| Método | Ámbito | Objetivo |                                                
|----|----|----|
| `GetUserData` | Usuario | Obtener datos de estado, guardados previamente, del usuario de un canal específico. |
| `GetConversationData` | Conversation | Obtener datos de estado, guardados previamente, de la conversación de un canal específico. |
| `GetPrivateConversationData` | Usuario y de conversación | Obtener datos de estado, guardados previamente, del usuario de una conversación de un canal específico. |
| `SetUserData` | Usuario | Guardar datos de estado del usuario de un canal específico. |
| `SetConversationData` | Conversation | Guardar datos de estado de la conversación de un canal específico. <br/><br/>**Nota**: Como el método `DeleteStateForUser` no elimina los datos que se han almacenado mediante el método `SetConversationData`, NO debe usar este método para almacenar información personal identificable (PII) de un usuario. |
| `SetPrivateConversationData` | Usuario y de conversación | Guardar datos de estado del usuario en una conversación de un canal específico. |
| `DeleteStateForUser` | Usuario | Eliminar datos de estado del usuario, que se hayan almacenado previamente mediante el método `SetUserData` o `SetPrivateConversationData`. <br/><br/>**Nota**: El bot debe llamar a este método cuando reciba una actividad de tipo [deleteUserData](bot-builder-dotnet-activities.md#deleteuserdata) o una actividad de tipo [contactRelationUpdate](bot-builder-dotnet-activities.md#contactrelationupdate) que indique que el bot ha sido eliminado de la lista de contactos del usuario. |

Si el bot guarda los datos de estado mediante uno de los métodos "**Set...Data**", los mensajes futuros que reciba el bot en el mismo contexto contendrán esos datos, a los que el bot puede acceder con el correspondiente método "**Get...Data**".

## <a name="useful-properties-for-managing-state-data"></a>Propiedades útiles para administrar los datos de estado

Cada objeto [Activity][Activity] contiene propiedades que usará para administrar datos de estado.

| Propiedad | DESCRIPCIÓN | Caso de uso |
|----|----|----|
| `From` | Identifica un usuario en un canal de forma única. | Almacenar y recuperar datos de estado que estén asociados a un usuario. |
| `Conversation` | Identifica una conversación de forma única. | Almacenar y recuperar datos de estado que estén asociados a una conversación. |
| `From` y `Conversation` | Identifica un usuario y una conversación de forma única. | Almacenar y recuperar datos de estado que estén asociados a un usuario específico dentro del contexto de una conversación concreta. |

> [!NOTE]
> Puede usar estos valores de propiedad como claves, incluso si decide almacenar los datos de estado en su propia base de datos, en lugar de usar el almacén de datos de estado de Bot Framework.

## <a name="handle-concurrency-issues"></a>Administrar los problemas de simultaneidad

El bot puede recibir una respuesta de error con el código de estado HTTP **412 Error de condición previa** cuando intenta guardar datos de estado, si es que otra instancia del bot ha cambiado los datos. Puede diseñar su bot para que tenga en cuenta este escenario, tal como se muestra en el siguiente ejemplo de código.

[!code-csharp[Handle exception saving state](../includes/code/dotnet-state.cs#handleException)]

## <a name="additional-resources"></a>Recursos adicionales

- [Bot Framework troubleshooting guide](../bot-service-troubleshoot-general-problems.md) (Guía de solución de problemas de Bot Framework)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Referencia de Bot Framework SDK para .NET</a>

[Activity]: https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html

---
title: Administración de datos de estado | Microsoft Docs
description: Obtenga información sobre cómo almacenar y recuperar datos de estado con el servicio Bot State.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: b0d5ca6893d70a73bc005a949ef6cc2518d3862f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000032"
---
# <a name="manage-state-data"></a>Administración de datos de estado

El servicio Bot State permite que el bot almacene y recupere datos de estado asociados a un usuario, una conversación o un usuario específico en el contexto de una conversación específica. Puede almacenar hasta 32 KB de datos para cada usuario en un canal, cada conversación en un canal y cada usuario en el contexto de una conversación en un canal. Los datos de estado se pueden usar para muchos propósitos, como determinar dónde se dejó una conversación anterior o simplemente saludar por su nombre a un usuario cuando vuelva. Si almacena las preferencias del usuario, puede usar esa información para personalizar la conversación la próxima vez que chatee. Por ejemplo, podría alertar al usuario sobre un artículo de noticias que trate un tema que le interese, o bien cuando haya una cita disponible. 

> [!IMPORTANT]
> No se recomienda la API del servicio de estado de Bot Framework para entornos de producción y es posible que se encuentre en desuso en una versión futura. Se recomienda actualizar el código del bot para que use el almacenamiento en memoria para realizar pruebas o usar una de las **extensiones de Azure** para bots de producción. Para obtener más información, consulte el tema **Administración de datos de estado** para la implementación de [.NET](~/dotnet/bot-builder-dotnet-state.md) o [Node](~/nodejs/bot-builder-nodejs-state.md).

## <a id="concurrency"></a> Simultaneidad de datos

Para controlar la simultaneidad de datos que se almacenan con el servicio de Bot State, el marco usa la etiqueta de entidad (ETag) para las solicitudes `POST`. El marco no utiliza los encabezados estándar para las etiquetas ETag. En su lugar, el cuerpo de la solicitud y respuesta especifica la ETag con la propiedad `eTag`. 

Por ejemplo, si emite una solicitud `GET` para recuperar datos del usuario del almacén, la respuesta contendrá la propiedad `eTag`. Si cambia los datos y emite una solicitud `POST` para guardar los datos actualizados en el almacén, la solicitud puede incluir la propiedad `eTag` que especifica el mismo valor que recibió anteriormente en la respuesta de `GET`. Si la ETag especificada en la solicitud `POST` coincide con el valor actual del almacén, el servidor guardará los datos del usuario y responderá con **HTTP 200 Success** (HTTP 200 Correcto) y especificará un nuevo valor de `eTag` en el cuerpo de la respuesta. Si la ETag especificada en la solicitud `POST` no coincide con el valor actual del almacén, el servidor responderá con **HTTP 412 Precondition Failed** (HTTP 412 Error en la condición previa) para indicar que los datos del usuario en el almacén han cambiado desde que los guardó o recuperó por última vez.

> [!TIP]
> Una respuesta de `GET` siempre incluirá una propiedad `eTag`, pero no es necesario especificar dicha `eTag` propiedad en las solicitudes `GET`. Un valor de propiedad `eTag` de asterisco (`*`) indica que no ha guardado previamente los datos para la combinación especificada de canal, usuario y conversación.
>
> La especificación de la propiedad `eTag` en las solicitudes `POST` es opcional. 
> Si la simultaneidad no es un problema para el bot, no incluya la propiedad `eTag` en las solicitudes `POST`. 

## <a id="save-user-data"></a> Almacenamiento de los datos del usuario

Para guardar los datos de estado de un usuario en un canal específico, emita esta solicitud:

```http
POST /v3/botstate/{channelId}/users/{userId}
```

En este URI de solicitud, reemplace **{channelId}** por el id. del canal y **{userId}** por el id. del usuario en dicho canal. Las propiedades `channelId` y `from` de cualquier mensaje que el bot haya recibido del usuario con anterioridad incluirán estos identificadores. También puede almacenar en caché estos valores en una ubicación segura para que pueda acceder a los datos del usuario en el futuro sin tener que extraerlos de un mensaje.

Establezca el cuerpo de la solicitud en un objeto [BotData][BotData], donde la propiedad `data` especifique los datos que quiere guardar. Si usa etiquetas de entidad para el [control de simultaneidad](#concurrency), establezca la propiedad `eTag` en la ETag que recibió en la respuesta la última vez que guardó o recuperó los datos del usuario (lo que sea la más reciente). Si no usa etiquetas de entidad para la simultaneidad, no incluya la propiedad `eTag` en la solicitud.

> [!NOTE]
> Esta solicitud `POST` sobrescribirá los datos del usuario en el almacén solo si la ETag especificada coincide con la del servidor, o bien si no se especificó ninguna ETag en la solicitud.

### <a name="request"></a>Solicitud 

En el ejemplo siguiente se muestra una solicitud que guarda los datos de un usuario en un canal específico. En esta solicitud de ejemplo, `https://smba.trafficmanager.net/apis` representa el URI base; los URI base para las solicitudes que emite el bot pueden ser diferentes. Para obtener más información sobre cómo establecer el URI base, consulte [API Reference](bot-framework-rest-connector-api-reference.md#base-uri) (Referencia de la API).

```http
POST https://smba.trafficmanager.net/apis/v3/botstate/abcd1234/users/12345678
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "data": [
        {
            "trail": "Lake Serene",
            "miles": 8.2,
            "difficulty": "Difficult",
        },
        {
            "trail": "Rainbow Falls",
            "miles": 6.3,
            "difficulty": "Moderate",
        }
    ],
    "eTag": "a1b2c3d4"
}
```

### <a name="response"></a>Response 

La respuesta contendrá un objeto [BotData][BotData] con un nuevo valor `eTag`.

## <a name="get-user-data"></a>Obtención de los datos del usuario

Para obtener datos de estado guardados previamente del usuario en un canal específico, emita esta solicitud:

```http
GET /v3/botstate/{channelId}/users/{userId}
```

En este URI de solicitud, reemplace **{channelId}** por el id. del canal y **{userId}** por el id. del usuario en dicho canal. Las propiedades `channelId` y `from` de cualquier mensaje que el bot haya recibido del usuario con anterioridad incluirán estos identificadores. También puede almacenar en caché estos valores en una ubicación segura para que pueda acceder a los datos del usuario en el futuro sin tener que extraerlos de un mensaje.

### <a name="request"></a>Solicitud

En el ejemplo siguiente se muestra una solicitud que obtiene los datos que se han guardado previamente del usuario. En esta solicitud de ejemplo, `https://smba.trafficmanager.net/apis` representa el URI base; los URI base para las solicitudes que emite el bot pueden ser diferentes. Para obtener más información sobre cómo establecer el URI base, consulte [API Reference](bot-framework-rest-connector-api-reference.md#base-uri) (Referencia de la API).

```http
GET https://smba.trafficmanager.net/apis/v3/botstate/abcd1234/users/12345678
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

### <a name="response"></a>Response

La respuesta contendrá un objeto [BotData][BotData], donde la propiedad `data` contiene los datos que ha guardado previamente para el usuario y la propiedad `eTag` contiene la ETag que puede usar en una solicitud posterior para guardar los datos del usuario. Si no ha guardado previamente los datos del usuario, la propiedad `data` será `null` y la propiedad `eTag` contendrá un asterisco (`*`).

En este ejemplo se muestra la respuesta a la solicitud `GET`:

```json
{
    "data": [
        {
            "trail": "Lake Serene",
            "miles": 8.2,
            "difficulty": "Difficult",
        },
        {
            "trail": "Rainbow Falls",
            "miles": 6.3,
            "difficulty": "Moderate",
        }
    ],
    "eTag": "xyz12345"
}
```

## <a name="save-conversation-data"></a>Almacenamiento de los datos de una conversación

Para guardar los datos de estado de una conversación en un canal específico, emita esta solicitud:

```http
POST /v3/botstate/{channelId}/conversations/{conversationId}
```

En este URI de solicitud, reemplace **{channelId}** por el id. del canal y **{conversationId}** por el id. de la conversación. Las propiedades `channelId` y `conversation` de cualquier mensaje que el bot haya enviado o recibido con anterioridad en el contexto de la conversación incluirán estos identificadores. También puede almacenar en caché estos valores en una ubicación segura para que pueda acceder a los datos de la conversación en el futuro sin tener que extraerlos de un mensaje.

Establezca el cuerpo de la solicitud en un objeto [BotData][BotData], según se describe en [Almacenamiento de los datos del usuario](#save-user-data).

> [!IMPORTANT]
> Dado que la operación [Eliminación de los datos del usuario](#delete-user-data) no elimina los datos que se han almacenado mediante la operación **Almacenamiento de los datos de una conversación**, NO debe utilizar esta operación para almacenar información de identificación personal de un usuario.

### <a name="response"></a>Response

La respuesta contendrá un objeto [BotData][BotData] con un nuevo valor `eTag`.

## <a name="get-conversation-data"></a>Obtención de los datos de una conversación

Para obtener datos de estado guardados previamente de una conversación en un canal específico, emita esta solicitud:

```http
GET /v3/botstate/{channelId}/conversations/{conversationId}
```

En este URI de solicitud, reemplace **{channelId}** por el id. del canal y **{conversationId}** por el id. de la conversación. Las propiedades `channelId` y `conversation` de cualquier mensaje que el bot haya enviado o recibido con anterioridad en el contexto de la conversación incluirán estos identificadores. También puede almacenar en caché estos valores en una ubicación segura para que pueda acceder a los datos de la conversación en el futuro sin tener que extraerlos de un mensaje.

### <a name="response"></a>Response

La respuesta contendrá un objeto [BotData][BotData] con un nuevo valor `eTag`.

## <a name="save-private-conversation-data"></a>Almacenamiento de los datos de una conversación privada

Para guardar los datos de estado de un usuario en el contexto de una conversación específica, emita esta solicitud:

```http
POST /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId}
```

En este URI de solicitud, reemplace **{channelId}** por el id. del canal y **{conversationId}** por el id. de la conversación, así como **{userId}** por el id. del usuario en ese canal. Las propiedades `channelId`, `conversation` y `from` de cualquier mensaje que el bot haya recibido del usuario con anterioridad en el contexto de la conversación incluirán estos identificadores. También puede almacenar en caché estos valores en una ubicación segura para que pueda acceder a los datos de la conversación en el futuro sin tener que extraerlos de un mensaje.

Establezca el cuerpo de la solicitud en un objeto [BotData][BotData], según se describe en [Almacenamiento de los datos del usuario](#save-user-data).

### <a name="response"></a>Response

La respuesta contendrá un objeto [BotData][BotData] con un nuevo valor `eTag`.

## <a name="get-private-conversation-data"></a>Obtención de los datos de una conversación privada

Para obtener datos de estado guardados previamente de un usuario dentro del contexto de una conversación específica, emita esta solicitud: 

```http
GET /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId}
```

En este URI de solicitud, reemplace **{channelId}** por el id. del canal y **{conversationId}** por el id. de la conversación, así como **{userId}** por el id. del usuario en ese canal. Las propiedades `channelId`, `conversation` y `from` de cualquier mensaje que el bot haya recibido del usuario con anterioridad en el contexto de la conversación incluirán estos identificadores. También puede almacenar en caché estos valores en una ubicación segura para que pueda acceder a los datos de la conversación en el futuro sin tener que extraerlos de un mensaje.

### <a name="response"></a>Response

La respuesta contendrá un objeto [BotData][BotData] con un nuevo valor `eTag`.

## <a name="delete-user-data"></a>Eliminación de los datos de usuario

Para eliminar los datos de estado de un usuario en un canal específico, emita esta solicitud:

```http
DELETE /v3/botstate/{channelId}/users/{userId}
```
En este URI de solicitud, reemplace **{channelId}** por el id. del canal y **{userId}** por el id. del usuario en dicho canal. Las propiedades `channelId` y `from` de cualquier mensaje que el bot haya recibido del usuario con anterioridad incluirán estos identificadores. También puede almacenar en caché estos valores en una ubicación segura para que pueda acceder a los datos del usuario en el futuro sin tener que extraerlos de un mensaje.

> [!IMPORTANT]
> Esta operación elimina los datos almacenados anteriormente del usuario mediante la operación [Almacenamiento de los datos del usuario](#save-user-data) o [Almacenamiento de los datos de una conversación privada](#save-private-conversation-data). NO elimina los datos que se han almacenado previamente mediante el uso de la operación [Almacenamiento de los datos de una conversación](#save-conversation-data). Por lo tanto, NO debe utilizar la operación **Almacenamiento de los datos de una conversación** para almacenar información de identificación personal de un usuario.

El bot debe ejecutar la operación **Eliminación de los datos del usuario** cuando recibe una [actividad][Activity] de tipo [deleteUserData](bot-framework-rest-connector-activities.md#deleteuserdata) o una de tipo [contactRelationUpdate](bot-framework-rest-connector-activities.md#contactrelationupdate) que indica que el bot se ha quitado de la lista de contactos del usuario.

## <a name="additional-resources"></a>Recursos adicionales

- [Conceptos clave](bot-framework-rest-connector-concepts.md)
- [Activities overview](bot-framework-rest-connector-activities.md) (Introducción a las actividades)

[BotData]: bot-framework-rest-connector-api-reference.md#botdata-object
[Activity]: bot-framework-rest-connector-api-reference.md#activity-object

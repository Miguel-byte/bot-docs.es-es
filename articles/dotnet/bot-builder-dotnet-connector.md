---
title: Envío y recepción de actividades | Microsoft Docs
description: Obtenga información sobre cómo intercambiar información con un usuario a través de diversos canales con el servicio Connector mediante Bot Framework SDK para. NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 4df2dcc8857c2af9a69c18e6acf8c8d064e1e043
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405689"
---
# <a name="send-and-receive-activities"></a>Envío y recepción de actividades

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Bot Framework Connector proporciona una única API de REST que permite que un bot se comunique a través de varios canales, como Skype, correo electrónico, Slack y mucho más. Facilita la comunicación entre el bot y el usuario mediante la retransmisión de mensajes del bot al canal y viceversa. 

En este artículo se describe cómo usar el conector con Bot Framework SDK para .NET con el fin de intercambiar información entre el bot y el usuario en un canal. 

> [!NOTE]
> Si bien es posible construir un bot solo con las técnicas que se describen en este artículo, Bot Framework SDK proporciona características adicionales como [diálogos](bot-builder-dotnet-dialogs.md) y [FormFlow](bot-builder-dotnet-formflow.md) que pueden optimizar el proceso de administrar el estado y el flujo de una conversación para simplificar la incorporación de Cognitive Services, como Language Understanding.

## <a name="create-a-connector-client"></a>Creación de un cliente de conector

La clase [ConnectorClient][ConnectorClient] incluye los métodos que un bot usa para comunicarse con un usuario en un canal. Cuando el bot recibe un objeto <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity</a> desde el conector, debe usar el objeto `ServiceUrl` especificado para esa actividad para crear el cliente de conector que usará posteriormente para generar una respuesta. 

[!code-csharp[Create connector client](../includes/code/dotnet-send-and-receive.cs#createConnectorClient)]

> [!TIP]
> Dado que es posible que el punto de conexión de un canal no sea estable, el bot debe dirigir las comunicaciones al punto de conexión que el conector especifica en el objeto `Activity`, siempre que sea posible (en lugar de basarse en un punto de conexión almacenado en caché). 
>
> Si el bot debe iniciar la conversación, puede usar un punto de conexión almacenado en caché para el canal especificado (debido a que no habrá ningún objeto `Activity` entrante en ese escenario), pero deberá actualizar a menudo los puntos de conexión almacenados en caché. 

## <a id="create-reply"></a> Creación de una respuesta

El conector usa un objeto [Activity](bot-builder-dotnet-activities.md) para pasar información de un lado a otro entre el bot y el canal (usuario). Cada actividad contiene información utilizada para enrutar el mensaje al destino adecuado junto con información sobre quién creó el mensaje (propiedad `From`), cuál es el contexto del mensaje y quién es el destinatario del mismo (propiedad `Recipient`).

Cuando el bot recibe una actividad del conector, la propiedad `Recipient` de la actividad entrante especifica la identidad del bot en esa conversación. Como algunos canales (por ejemplo, Slack) asignan una identidad nueva al bot cuando se agrega a una conversación, el bot siempre debe usar el valor de la propiedad `Recipient` de la actividad entrante como el valor de la propiedad `From` en su respuesta.

Si bien el mismo usuario puede crear e inicializar el objeto `Activity` saliente desde cero, el SDK de Bot Builder proporciona una manera más sencilla de crear una respuesta. Con el método `CreateReply` de la actividad entrante, simplemente debe especificar el texto del mensaje para la respuesta y la actividad saliente se crea con las propiedades `Recipient`, `From` y `Conversation` que se rellenan automáticamente.

[!code-csharp[Create reply](../includes/code/dotnet-send-and-receive.cs#createReply)]

## <a name="send-a-reply"></a>Envío de una respuesta

Una vez que crea una respuesta, puede enviarla si llama al método `ReplyToActivity` del cliente de conector. El conector entregará la respuesta a través de la semántica de canal correspondiente. 

[!code-csharp[Send reply](../includes/code/dotnet-send-and-receive.cs#sendReply)]

> [!TIP]
> Si el bot responde el mensaje de un usuario, use siempre el método `ReplyToActivity`.

## <a name="send-a-non-reply-message"></a>Envío de un mensaje (de no respuesta) 

Si el bot es parte de una conversación, puede enviar un mensaje que no es una respuesta directa a ningún mensaje del usuario si llama al método `SendToConversation`. 

[!code-csharp[Send non-reply message](../includes/code/dotnet-send-and-receive.cs#sendNonReplyMessage)]

Puede usar el método `CreateReply` para inicializar el mensaje nuevo (que establecería automáticamente las propiedades `Recipient`, `From` y `Conversation` del mensaje). Como alternativa, puede usar el método `CreateMessageActivity` para crear el nuevo mensaje y establecer todos los valores de propiedad usted mismo.

> [!NOTE]
> Bot Framework no impone ninguna restricción en el número de mensajes que puede enviar un bot. Sin embargo, la mayoría de los canales aplican valores de limitación para impedir que los bots envíen un gran número de mensajes en un breve período de tiempo. Además, si el bot envía varios mensajes en una sucesión rápida, puede que el canal no siempre procese los mensajes en el orden correcto.

## <a name="start-a-conversation"></a>Inicio de una conversación

Puede haber ocasiones en las que su bot debe iniciar una conversación con uno o varios usuarios. Puede iniciar una conversación llamando al método `CreateDirectConversation` (para una conversación privada con un solo usuario) o al método `CreateConversation` (para una conversación grupal con varios usuarios) para recuperar un objeto`ConversationAccount`. Luego, cree el mensaje y envíelo mediante una llamada al método `SendToConversation`. Para usar el método `CreateDirectConversation` o el método `CreateConversation`, primero debe [crear el cliente de conector](#create-a-connector-client) mediante el uso de la dirección URL de servicio del canal de destino (que podría recuperar desde la caché, si se conservó a partir de mensajes anteriores). 

> [!NOTE]
> No todos los canales admiten conversaciones de grupo. Para determinar si un canal admite las conversaciones grupales, consulte la documentación del canal.

Este ejemplo de código utiliza el método `CreateDirectConversation` para crear una conversación privada con un solo usuario.

[!code-csharp[Start private conversation](../includes/code/dotnet-send-and-receive.cs#startPrivateConversation)]

Este ejemplo de código utiliza el método `CreateConversation` para crear una conversación grupal con varios usuarios.

[!code-csharp[Start group conversation](../includes/code/dotnet-send-and-receive.cs#startGroupConversation)]

## <a name="additional-resources"></a>Recursos adicionales

- [Activities overview](bot-builder-dotnet-activities.md) (Introducción a las actividades)
- [Creación de mensajes](bot-builder-dotnet-create-messages.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Referencia de Bot Framework SDK para .NET</a>
- <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Clase Activity</a>
- <a href="/dotnet/api/microsoft.bot.connector.connectorclient" target="_blank">Clase ConnectorClient</a>

[ConnectorClient]: /dotnet/api/microsoft.bot.connector.connectorclient

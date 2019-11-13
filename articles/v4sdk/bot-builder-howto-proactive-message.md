---
title: Envío de notificaciones activas a los usuarios | Microsoft Docs
description: Descripción de cómo enviar mensajes de notificación
keywords: mensaje proactivo, mensaje de notificación, notificación de bot,
author: jonathanfingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 774738186127bff1e680d905d208b69097402d8e
ms.sourcegitcommit: 312a4593177840433dfee405335100ce59aac347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/12/2019
ms.locfileid: "73933581"
---
# <a name="send-proactive-notifications-to-users"></a>Envío de notificaciones activas a los usuarios

[!INCLUDE[applies-to](../includes/applies-to.md)]

Normalmente, cada mensaje que envía un bot al usuario se relaciona directamente con la anterior entrada del usuario.
En algunos casos, puede que un bot tenga que enviar al usuario un mensaje que no está relacionado directamente con el tema actual de la conversación o con el último mensaje que envió el usuario. Este tipo de mensajes se llaman _mensajes proactivos_.

Los mensajes proactivos pueden ser útiles en diversos escenarios. Por ejemplo, si el usuario ha solicitado anteriormente al bot que supervise el precio de un producto, el bot puede alertar al usuario si el precio del producto ha descendido un 20 %. O bien, si un bot necesita algo de tiempo para compilar una respuesta a la pregunta del usuario, puede informar al usuario del retraso y permitir que la conversación continúe mientras tanto. Cuando el bot termine de compilar la respuesta a la pregunta, compartirá esta información con el usuario.

Al implementar mensajes proactivos en un bot, no enviar varios mensajes proactivos en un breve lapso de tiempo. Algunos canales imponen restricciones sobre la frecuencia con que un bot puede enviar mensajes al usuario, y deshabilitarán el bot si se infringen tales restricciones.

Un mensaje proactivo ad hoc es el tipo más simple de mensaje proactivo. El bot simplemente interpone el mensaje en la conversación cada vez que se desencadena, sin tener en cuenta si el usuario está implicado actualmente en otro tema de conversación con el bot, y no intentará cambiar la conversación de ninguna manera.

Para controlar las notificaciones más fácilmente, considere otras formas de integrar la notificación en el flujo de conversación, como establecer una marca en el estado de la conversación o agregar la notificación a una cola.

## <a name="prerequisites"></a>Requisitos previos

- Comprender los [conceptos básicos de bot](bot-builder-basics.md).
- Una copia del ejemplo de mensajes proactivos en **[C#](https://aka.ms/proactive-sample-cs) o [JavaScript](https://aka.ms/proactive-sample-js)** . Este ejemplo se utiliza para explicar la mensajería proactiva de este artículo.

## <a name="about-the-proactive-sample"></a>Acerca del ejemplo proactivo

El ejemplo tiene un bot y un controlador adicional que se usa para enviar mensajes proactivos al bot, como se muestra en la siguiente ilustración.

![Bot proactivo](media/proactive-sample-bot.png)

## <a name="retrieve-and-store-conversation-reference"></a>Recuperación y almacenamiento de la referencia de la conversación

Cuando el emulador se conecta al bot, este recibe dos actividades de actualización de conversación. En el controlador de actividad de actualización de conversación del bot, la referencia de la conversación se recupera y almacena en un diccionario, como se muestra a continuación.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\ProactiveBot.cs**

[!code-csharp[OnConversationUpdateActivityAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/16.proactive-messages/Bots/ProactiveBot.cs?range=26-37&highlight=3-4,9)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/proactiveBot.js**

[!code-javascript[onConversationUpdateActivity](~/../botbuilder-samples/samples/javascript_nodejs/16.proactive-messages/bots/proactiveBot.js?range=13-17&highlight=2)]

[!code-javascript[onConversationUpdateActivity](~/../botbuilder-samples/samples/javascript_nodejs/16.proactive-messages/bots/proactiveBot.js?range=41-44&highlight=2-3)]

---

Nota: En un escenario real, las referencias de conversación persistirían en una base de datos, en lugar de usar un objeto en memoria.

La referencia de la conversación tiene una propiedad _conversation_ que describe la conversación en el que existe la actividad. La conversación tiene una propiedad _user_ que enumera los usuarios que participan en la conversación, y una propiedad _service URL_ que los canales usan para indicar la dirección URL a la que se pueden enviar las respuestas a la actividad actual. Para enviar mensajes proactivos a los usuarios se necesita una referencia de conversación válida.

## <a name="send-proactive-message"></a>Envío de mensajes proactivos

El segundo controlador, _notificar_, es responsable de enviar el mensaje proactivo al bot. Siga estos pasos para generar un mensaje proactivo.

1. Recupere la referencia de la conversación a la que va a enviar el mensaje proactivo.
1. Llame al método para _continuar la conversación_ del adaptador, que proporciona la referencia de la conversación y el delegado de control de turnos que se va a usar. El método continuar conversación genera un contexto de turnos para la conversación a la que se hace referencia y, después, llama al delegado del controlador de turnos especificado.
1. En el delegado, use el contexto de turnos para enviar el mensaje proactivo.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Controllers\NotifyController .cs**

Cada vez que se solicita la página de notificación del bot, el controlador de notificación recupera las referencias de conversación del diccionario.
Luego, el controlador usa los métodos `ContinueConversationAsync` y `BotCallback` para enviar el mensaje proactivo.

[!code-csharp[Notify logic](~/../botbuilder-samples/samples/csharp_dotnetcore/16.proactive-messages/Controllers/NotifyController.cs?range=17-60&highlight=28,40-43)]

Para enviar un mensaje proactivo, el adaptador requiere un identificador de la aplicación para el bot. En un entorno de producción, puede usar el identificador de la aplicación del bot. En un entorno de prueba local, puede usar cualquier GUID. Si actualmente no está asignado ningún identificador de la aplicación al bot, el controlador de notificación genera automáticamente un identificador de marcador de posición que se usará para la llamada.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

Cada vez que se solicita la página `/api/notify` del servidor, este recupera las referencias de conversación del diccionario.
Luego, usa el método `continueConversation` para enviar el mensaje proactivo.
El parámetro para `continueConversation` es una función que sirve controlador de turnos del bot para este turno.

[!code-javascript[Notify logic](~/../botbuilder-samples/samples/javascript_nodejs/16.proactive-messages/index.js?range=68-80&highlight=4-6)]

---

## <a name="test-your-bot"></a>Prueba del bot

1. Si aún no lo ha hecho, instale [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).
1. Ejecute el ejemplo localmente en la máquina.
1. Inicie el emulador y conéctelo al bot.
1. Cargue la página de api o notificación del bot. De esa forma se generará un mensaje proactivo en el emulador.

## <a name="additional-information"></a>Información adicional

Además del ejemplo que se usa en este artículo, hay otros ejemplos disponibles en C# y JS en [GitHub](https://github.com/Microsoft/BotBuilder-Samples/).

### <a name="avoiding-401-unauthorized-errors"></a>Evitar errores del tipo 401 "no autorizado" 

De forma predeterminada, el SDK de BotBuilder agrega `serviceUrl` a la lista de nombres de host de confianza si BotAuthentication autentica la solicitud entrante. Dichos nombres se mantienen en una caché. Si el bot se reinicia, los usuarios que esperan un mensaje automático no podrán recibirlo, a menos que hayan enviado un mensaje al bot otra vez después de que se haya reiniciado. 

Para evitar este problema, debe agregar manualmente `serviceUrl` a la lista de nombres de host de confianza mediante el uso de: 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp 
MicrosoftAppCredentials.TrustServiceUrl(serviceUrl); 
``` 

Para la mensajería proactiva, `serviceUrl` es la dirección URL del canal que el destinatario del mensaje automático usa y se puede encontrar en `Activity.ServiceUrl`. 

Deseará agregar el código anterior inmediatamente antes del código que envía el mensaje automático. En el [ejemplo de mensajes proactivos](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/16.proactive-messages), lo colocaría en `NotifyController.cs` justo antes de `await turnContext.SendActivityAsync("proactive hello");`.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```js
MicrosoftAppCredentials.trustServiceUrl(serviceUrl);
```

Para la mensajería proactiva, `serviceUrl` es la dirección URL del canal que el destinatario del mensaje automático usa y se puede encontrar en `activity.serviceUrl`.

Deseará agregar el código anterior inmediatamente antes del código que envía el mensaje automático. En el [ejemplo de mensajes proactivos](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/16.proactive-messages), lo colocaría en `index.js` justo antes de `await turnContext.sendActivity('proactive hello');`.

---

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Implementación de flujo de conversación secuencial](bot-builder-dialog-manage-conversation-flow.md)

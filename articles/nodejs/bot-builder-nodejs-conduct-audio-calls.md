---
title: Realización de llamadas de audio | Microsoft Docs
description: Obtenga información sobre cómo realizar llamadas de audio con Skype en un bot mediante Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 9251307cb77cfb240e88c7ae44b13fe7e7fa2cdb
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998642"
---
# <a name="support-audio-calls-with-skype"></a>Compatibilidad de las llamadas de audio con Skype

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Skype admite una característica avanzada denominada Bots de llamadas.  Cuando se habilita, los usuarios pueden realizar una llamada de voz al bot e interactuar con él mediante la respuesta interactiva de voz (IVR).  El SDK de Bot Builder para Node.js incluye un [SDK de llamadas][calling_sdk] especial que los desarrolladores pueden usar para agregar características de llamadas al bot de chat.   

El SDK de llamadas es muy similar al [SDK de chat][chat_sdk]. Tienen clases similares, comparten construcciones comunes e incluso puede usar el SDK de chat para enviar un mensaje al usuario con el que está en una llamada.  Los dos SDK están diseñados para ejecutarse en paralelo, pero, aunque son similares, existen algunas diferencias importantes y, por lo general, debe evitar mezclar las clases de las dos bibliotecas.  

## <a name="create-a-calling-bot"></a>Creación de un bot de llamadas
En el ejemplo de código siguiente se muestra cómo el programa "Hola mundo" de un chat de llamada tiene un aspecto muy similar a un bot de chat normal. 

```javascript
var restify = require('restify');
var calling = require('botbuilder-calling');

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log(`${server.name} listening to ${server.url}`); 
});

// Create calling bot
var connector = new calling.CallConnector({
    callbackUrl: 'https://<your host>/api/calls',
    appId: '<your bots app id>',
    appPassword: '<your bots app password>'
});
var bot = new calling.UniversalCallBot(connector);
server.post('/api/calls', connector.listen());

// Add root dialog
bot.dialog('/', function (session) {
    session.send('Watson... come here!');
});
```

> [!NOTE]
> Para buscar los valores **AppID** y **AppPassword** del bot, consulte [MicrosoftAppID and MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword) (MicrosoftAppID y MicrosoftAppPassword).

Actualmente, el emulador no admite los bots de llamadas de prueba. Para probar el bot, debe realizar la mayoría de los pasos necesarios para publicar el bot.  También deberá usar un cliente de Skype para interactuar con el bot. 

### <a name="enable-the-skype-channel"></a>Habilitación del canal de Skype
[Registre el bot](../bot-service-quickstart-registration.md) y habilite el canal de Skype. Deberá proporcionar un punto de conexión de mensajería al registrar el bot. Se recomienda que empareje el bot de llamadas con un bot de chat para que el punto de conexión del bot de chat coincida con lo que colocaría normalmente en ese campo.  Si solo va a registrar un bot de llamadas, simplemente puede pegar el punto de conexión de llamadas en ese campo.  

Para habilitar la característica de llamadas real, deberá entrar en el canal de Skype del bot y activar la característica de llamadas. Se le proporcionará un campo en el que copiar el punto de conexión de llamadas. Asegúrese de usar el vínculo de https ngrok para la parte del host del punto de conexión de llamadas.

Durante el registro del bot, se le asignará un id. y una contraseña de aplicación, que debe pegar en la configuración del conector del bot Hola mundo. También deberá tomar el vínculo completo de llamadas y pegarlo para la configuración de callbackUrl.

### <a name="add-bot-to-contacts"></a>Adición del bot a los contactos
En la página de registro del bot en el portal para desarrolladores, verá el botón **Add to Skype** (Agregar a Skype) situado junto al canal de Skype de bots. Haga clic en el botón para agregar el bot a su lista de contactos de Skype.  Una vez hecho, usted (y cualquier usuario al que le haya proporcionado el vínculo de unión) podrá comunicarse con el bot.

### <a name="test-your-bot"></a>Prueba del bot
Puede probar el bot con un cliente de Skype. Debe ver que el icono de llamada está iluminado al hacer clic en la entrada de contacto de los bots (es posible que tenga que buscar el bot para verlo).  El icono de llamada puede tardar unos minutos en iluminarse si ha agregado llamadas a un bot existente.  

Si presiona el botón de llamada, debe marcar el bot y debe escucharlo hablar "Watson... come here!" ("Watson, ven aquí.") y, después, colgar.

## <a name="calling-basics"></a>Conceptos básicos de las llamadas
Las clases [UniversalCallBot](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.universalcallbot) y [CallConnector](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callconnector) le permiten crear un bot de llamadas en gran parte de la misma manera que lo haría con un bot de chat. Agregue diálogos al bot prácticamente idénticos a los [diálogos de chat](bot-builder-nodejs-manage-conversation-flow.md). Puede agregar [cascadas](bot-builder-nodejs-prompts.md) al bot. Hay un objeto de sesión, la clase [CallSession](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession), que contiene los métodos [answer()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#answer), [hangup()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#hangup) y [reject()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#reject) para administrar la llamada actual. En general, no es necesario preocuparse por tales métodos de control de llamadas, aunque CallSession tiene una lógica para administrar automáticamente la llamada. Si realiza una acción, como el envío de un mensaje o una llamada a un símbolo del sistema integrado, la sesión responderá automáticamente a la llamada. También colgará o rechazará automáticamente la llamada si llama a [endConversation()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#endconversation) o detecta que ha dejado de responder a las preguntas del autor de llamada (no llamó a un símbolo del sistema integrado).

Otra diferencia entre los bots de chat y de llamadas es que, mientras los bots de chat suelen enviar mensajes, tarjetas y teclados a un usuario, un bot de llamadas se encarga de las acciones y los resultados. Los bots de llamadas de Skype son necesarios para crear [flujos de trabajo](http://docs.botframework.com/en-us/node/builder/calling-reference/interfaces/_botbuilder_d_.iworkflow) que constan de una o varias [acciones](http://docs.botframework.com/en-us/node/builder/calling-reference/interfaces/_botbuilder_d_.iaction).  Se trata de otro aspecto sobre el que, en la práctica, no tiene que preocuparse demasiado, ya que el SDK de llamadas de Bot Builder administrará la mayoría. El método [CallSession.send()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#send) le permite pasar acciones o cadenas que se convertirán en [PlayPromptActions](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.playpromptaction).  La sesión contiene una lógica de procesamiento por lotes automática para combinar varias acciones en un único flujo de trabajo que se envía al servicio de llamadas a fin de que pueda llamar sin ningún riesgo a Send() varias veces.  Asimismo, debe confiar en las [solicitudes](bot-builder-nodejs-prompts.md) integradas del SDK para recopilar información del usuario a medida que se procesan todos los resultados.  

[calling_sdk]: http://docs.botframework.com/en-us/node/builder/calling-reference/modules/_botbuilder_d_
[chat_sdk]: http://docs.botframework.com/en-us/node/builder/chat-reference/modules/_botbuilder_d_

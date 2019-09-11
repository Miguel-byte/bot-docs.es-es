---
title: Intercambio de información mediante el control web | Microsoft Docs
description: Obtenga información sobre cómo intercambiar información entre el bot y una página web mediante Bot Framework SDK para Node.js.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 52f57cba5824deb01b176363880084760a95a41b
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299924"
---
# <a name="use-the-backchannel-mechanism"></a>Use el mecanismo Backchannel

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Introduction to backchannel mechanism](../includes/snippet-backchannel.md)]

## <a name="walk-through"></a>Tutorial

El control del chat web de código abierto tiene acceso a la API de Direct Line mediante una clase de JavaScript denominada <a href="https://github.com/microsoft/botframework-DirectLinejs" target="_blank">DirectLineJS</a>. El control puede crear su propia instancia de Direct Line o puede compartir una con la página de hospedaje. Si el control comparte una instancia de Direct Line con la página de hospedaje, el control y la página podrán enviar y recibir actividades. En el siguiente diagrama se muestra la arquitectura de alto nivel de un sitio web que es compatible con la funcionalidad de bot mediante el control (de chat) web de código abierto y la API de Direct Line. 

![Backchannel](../media/designing-bots/patterns/back-channel.png)

### <a name="sample-code"></a>Código de ejemplo 

En este ejemplo, el bot y una página web utilizarán el mecanismo Backchannel para intercambiar información que no es visible para el usuario. El bot solicitará que la página web cambie su color de fondo y la página web notificará al bot cuando el usuario haga clic en un botón de la página. 

> [!NOTE]
> Los fragmentos de código en este artículo vienen del <a href="https://github.com/Microsoft/BotFramework-WebChat/blob/master/samples/backchannel/index.html" target="_blank">ejemplo de Backchannel</a> y del <a href="https://github.com/ryanvolum/backChannelBot" target="_blank">bot de Backchannel</a>. 

#### <a name="client-side-code"></a>Código del lado cliente

En primer lugar, la página web crea un objeto **DirectLine**.

```javascript
var botConnection = new BotChat.DirectLine(...);
```

A continuación, comparte el objeto **DirectLine** cuando crea la instancia de WebChat.

```javascript
BotChat.App({
    botConnection: botConnection,
    user: user,
    bot: bot
}, document.getElementById("BotChatGoesHere"));
```

Cuando el usuario hace clic en un botón de la página web, esta envía una actividad de tipo "evento" para notificar al bot que se presionó el botón.

```javascript
const postButtonMessage = () => {
    botConnection
        .postActivity({type: "event", value: "", from: {id: "me" }, name: "buttonClicked"})
        .subscribe(id => console.log("success"));
    }
```

> [!TIP]
> Use los atributos `name` y `value` para comunicarle al bot cualquier información que pueda necesitar con el fin de interpretar correctamente el evento o para responder al mismo. 

Por último, la página web también está a la escucha de un evento específico del bot.
En este ejemplo, la página web está a la escucha de una actividad donde type ="event" y name="changeBackground". Cuando recibe este tipo de actividad, cambia el color de fondo de la página web al `value` especificado por la actividad. 

```javascript
botConnection.activity$
    .filter(activity => activity.type === "event" && activity.name === "changeBackground")
    .subscribe(activity => changeBackgroundColor(activity.value))
```

#### <a name="server-side-code"></a>Código del lado servidor

El <a href="https://github.com/ryanvolum/backChannelBot" target="_blank">bot de Backchannel</a> crea un evento mediante el uso de una función auxiliar.

```javascript
var bot = new builder.UniversalBot(connector, 
    function (session) {
        var reply = createEvent("changeBackground", session.message.text, session.message.address);
        session.endDialog(reply);
    }
);

const createEvent = (eventName, value, address) => {
    var msg = new builder.Message().address(address);
    msg.data.type = "event";
    msg.data.name = eventName;
    msg.data.value = value;
    return msg;
}
```

Del mismo modo, el bot también escucha los eventos desde el cliente. En este ejemplo, si el bot recibe un evento con `name="buttonClicked"`, enviará un mensaje al usuario para que diga "veo que hizo clic en un botón".

```javascript
bot.on("event", function (event) {
    var msg = new builder.Message().address(event.address);
    msg.data.textLocale = "en-us";
    if (event.name === "buttonClicked") {
        msg.data.text = "I see that you clicked a button.";
    }
    bot.send(msg);
})
```

## <a name="additional-resources"></a>Recursos adicionales

- [Direct Line API][directLineAPI]
- <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Control Chat en web de Microsoft Bot Framework</a>
- <a href="https://aka.ms/v3-js-backchannel-sample" target="_blank">Ejemplo de Backchannel</a>
- <a href="https://github.com/ryanvolum/backChannelBot" target="_blank">Bot de Backchannel</a>

[directLineAPI]: https://docs.botframework.com/restapi/directline3/#navtitle

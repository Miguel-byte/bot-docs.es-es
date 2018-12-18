---
title: Conexión de un bot al canal Chat en web | Microsoft Docs
description: Obtenga información sobre cómo usar el control Chat en web en la página web para un bot conectado al canal Chat en web.
keywords: chat en web, canal de bot, página web, clave secreta, IFrame, HTML
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/04/2018
ms.openlocfilehash: 6c3cc52514988172e5c2cc85168aef401fbc1f22
ms.sourcegitcommit: 91156d0866316eda8d68454a0c4cd74be5060144
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/07/2018
ms.locfileid: "53010500"
---
# <a name="connect-a-bot-to-web-chat"></a>Conexión de un bot a un Chat en web

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

Cuando se [crea un bot](bot-service-quickstart.md) con Bot Service, el canal Chat en web se configura automáticamente. El canal Chat en web incluye el control Chat en web, que proporciona la capacidad para que los usuarios interactúen con el bot directamente en una página web.

![Ejemplo de chat en web](./media/bot-service-channel-webchat/create-a-bot.png)

El canal Chat en web del portal de Framework Bot contiene todo lo que necesita para insertar el control Chat en web en una página web. Lo único que debe hacer para usar el control Chat en web es obtener la clave secreta del bot e insertar el control en una página web.

## <a id="step-1"></a> Obtener la clave secreta del bot

1. Abra el bot en [Azure Portal](http://portal.azure.com) y haga clic en la hoja **Canales**.

2. Haga clic en **Editar** en el canal **Chat en web**.  
![Canal Chat en web](./media/bot-service-channel-webchat/bot-service-channel-list.png)

3. En **Claves secretas**, haga clic en **Mostrar** para la primera clave.  
![Clave secreta](./media/bot-service-channel-webchat/secret-key.png)

4. Copia la **clave secreta** y el **código para insertar**.

5. Haga clic en **Done**(Listo).

## <a name="embed-the-web-chat-control-in-your-website"></a>Insertar el control Chat en web en su sitio web

Puede insertar el control Chat en web en su sitio web mediante una de las siguientes dos opciones.

### <a name="option-1---keep-your-secret-hidden-exchange-your-secret-for-a-token-and-generate-the-embed"></a>Opción 1: Mantener el secreto oculto, intercambiar el secreto por un token y generar la inserción

Use esta opción si puede ejecutar una solicitud de servidor a servidor para intercambiar el secreto de chat en web por un token temporal, y si quiere hacer que sea difícil para otros desarrolladores insertar el bot en sus sitios web. Aunque el uso de esta opción no impedirá en absoluto que otros desarrolladores inserten el bot en sus sitios web, hace que sea difícil hacerlo.

Para intercambiar el secreto por un token y generar la inserción, siga los siguientes pasos:

1. Emita una solicitud **GET** para `https://webchat.botframework.com/api/tokens` y transfiera su secreto de chat en web a través del encabezado `Authorization`. El encabezado `Authorization` utiliza el esquema `BotConnector` e incluye el secreto, como se muestra en la siguiente solicitud de ejemplo.

2. La respuesta a su solicitud **GET** contendrá el token (entre comillas) que se puede usar para iniciar una conversación al representar el control Chat en web dentro de un **IFrame**. Un token es válido solo para una conversación. Para iniciar otra conversación, debe generar un nuevo token.

3. Dentro del `iframe` **Código para insertar** que ha copiado del canal WebChat del portal de Bot Framework (como se describe en la sección [Obtener la clave secreta del bot](#step-1) anterior), cambie el parámetro `s=` por `t=` y reemplace "YOUR_SECRET_HERE" por su token.

> [!NOTE]
> Los tokens se renovarán automáticamente antes de que expiren. 

##### <a name="example-request"></a>Solicitud de ejemplo

```requestGET https://webchat.botframework.com/api/tokens Authorization: BotConnector YOUR_SECRET_HERE
```

##### Example response 

```response
"IIbSpLnn8sA.dBB.MQBhAFMAZwBXAHoANgBQAGcAZABKAEcAMwB2ADQASABjAFMAegBuAHYANwA.bbguxyOv0gE.cccJjH-TFDs.ruXQyivVZIcgvosGaFs_4jRj1AyPnDt1wk1HMBb5Fuw"
```

##### <a name="example-iframe-using-token"></a>IFrame de ejemplo (con el token)

```html
<iframe src="https://webchat.botframework.com/embed/YOUR_BOT_ID?t=YOUR_TOKEN_HERE"></iframe>
```

##### <a name="example-html-code"></a>Código HTML de ejemplo
```html
<!DOCTYPE html>
<html>
<body>
  <iframe id="chat" style="width: 400px; height: 400px;" src=''></iframe>
</body>
<script>

    var xhr = new XMLHttpRequest();
    xhr.open('GET', "https://webchat.botframework.com/api/tokens", true);
    xhr.setRequestHeader('Authorization', 'BotConnector ' + 'YOUR SECRET HERE');
    xhr.send();
    xhr.onreadystatechange = processRequest;

    function processRequest(e) {
      if (xhr.readyState == 4  && xhr.status == 200) {
        var response = JSON.parse(xhr.responseText);
        document.getElementById("chat").src="https://webchat.botframework.com/embed/lucas-direct-line?t="+response
      }
    }

  </script>
</html>
```

### <a id="option-2"></a> Opción 2: Insertar el control Chat en web en su sitio web mediante el secreto

Use esta opción si quiere permitir que otros desarrolladores inserten fácilmente el bot en sus sitios web. 

> [!WARNING]
> Si usa esta opción, otros desarrolladores podrán insertar el bot en sus sitios web simplemente copiando el código para insertar.

Para insertar el bot en su sitio web indicando el secreto en la etiqueta `iframe`, siga los siguientes pasos:

1. Copie el **Código para insertar** `iframe` desde el canal Chat en web en el portal de Bot Framework (como se describe en la sección [Obtener la clave secreta del bot](#step-1) anterior).

2. En ese **Código para insertar**, reemplace "YOUR_SECRET_HERE" por la **clave secreta** que copió de la misma página.

##### <a name="example-iframe-using-secret"></a>IFrame de ejemplo (con el secreto)

```html
<iframe src="https://webchat.botframework.com/embed/YOUR_BOT_ID?s=YOUR_SECRET_HERE"></iframe>
```

## <a name="style-the-web-chat-control"></a>Estilo del control de chat web

Puede cambiar el tamaño del control Chat en web mediante el uso del atributo `style` de `iframe` para especificar los valores `height` y `width`.

```html
<iframe style="height:480px; width:402px" src="... SEE ABOVE ..."></iframe>
```

![Cliente de control de chat](./media/chatwidget-client.png)

## <a name="additional-resources"></a>Recursos adicionales

También puede [descargar el código fuente](https://aka.ms/BotFramework-WebChat-V4) para el control de Chat en web en GitHub.

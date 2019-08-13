---
title: Uso de WebChat con la extensión de App Service para Direct Line
titleSuffix: Bot Service
description: Uso de WebChat con la extensión de App Service para Direct Line
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 07/25/2019
ms.openlocfilehash: 5e74627530e77f4ae5f1f8ec1ae36dc5b07959a6
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2019
ms.locfileid: "68757747"
---
## <a name="use-webchat-with-the-direct-line-app-service-extension"></a>Uso de WebChat con la extensión de App Service para Direct Line

En este artículo se describe cómo usar WebChat con la extensión de App Service para Direct Line

### <a name="get-your-direct-line-secret"></a>Obtención del secreto de Direct Line

El primer paso es encontrar el secreto de Direct Line. Puede hacerlo según las instrucciones descritas en [Conexión de un bot a Direct Line](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-channel-connect-directline?view=azure-bot-service-4.0).

### <a name="get-the-preview-version-of-directlinejs"></a>Obtención de la versión preliminar de DirectLineJS
La versión preliminar de DirectLineJS se puede encontrar aquí: https://github.com/Jeffders/DirectLineAppServiceExtensionPreview/tree/master/libraries

### <a name="integrate-webchat-client"></a>Integración del cliente de WebChat

En términos generales, el enfoque es el mismo que antes. Con la excepción de que se ha creado una nueva versión de **WebChat** que admite el tráfico bidireccional de **WebSocket** y que en lugar de conectarse a https://directline.botframework.com/ se conecta directamente al bot hospedado.
La dirección URL de Direct Line para el bot será `https://<your_app_service>.azurewebsites.net/.bot/`, donde la extensión `/.bot/` es el **punto de conexión** de Direct Line en la instancia de App Service.
Si bien puede configurar su propio nombre de dominio, debe anexar la ruta de acceso `/.bot/` para acceder a las API REST de Direct Line.

1. Intercambie el secreto por un token según las instrucciones del artículo [Autenticación](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-authentication?view=azure-bot-service-4.0). Sin embargo, en lugar de obtener un token en esta ubicación: `https://directline.botframework.com/v3/directline/tokens/generate`, genere el token directamente desde su extensión de App Service para Direct Line en esta ubicación: `https://<your_app_service>.azurewebsites.net/.bot/v3/directline/tokens/generate`.  

1. Una vez que tenga un token, puede actualizar la página web que usa WebChat con estos cambios:

```html
<!DOCTYPE html>
<html lang="en-US">
<head>
    <title>Direct Line Streaming Sample</title>
    <script src="~/directLine.js"></script>
    <script src="https://cdn.botframework.com/botframework-webchat/master/webchat.js"></script>
    <style>
        html, body {
            height: 100%
        }

        body {
            margin: 0
        }

        #webchat,

        #webchat > * {
            height: 100%;
            width: 100%;
        }
    </style>
</head>

<body>
    <div id="webchat" role="main"></div>
    <script>
        const activityMiddleware = () => next => card => {
            if (card.activity.type === 'trace') {
                // Return false means, don't render the trace activities
                return () => false;
            } else {
                return children => next(card)(children);
            }
        };


        var dl = new DirectLine.DirectLine({
            secret: '<your token>',
            domain: 'https://<your_site>.azurewebsites.net/.bot/v3/directline',
            webSocket: true,
            conversationId: '<your conversation id>'
        });
        window.WebChat.renderWebChat({
            activityMiddleware,
            directLine: dl,
            userID: '<your generated user id>'
        }, document.getElementById('webchat'));
    </script>
</body>
</html>

```

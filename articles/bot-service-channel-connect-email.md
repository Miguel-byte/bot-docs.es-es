---
title: Conexión de un bot al correo electrónico de Office 365 | Microsoft Docs
description: Aprenda a configurar un bot para enviar y recibir correo electrónico con Office 365.
keywords: Office 365, canales de bots, correo electrónico, credenciales de correo electrónico, azure portal, correo electrónico personalizado
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/10/2018
ms.openlocfilehash: 93270dd6211d8aef1ff44fb8e272855df2058b8a
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997212"
---
# <a name="connect-a-bot-to-office-365-email"></a>Conexión de un bot al correo electrónico de Office 365

Los bots pueden comunicarse con los usuarios mediante el correo electrónico de Office 365 además de otros [canales](~/bot-service-manage-channels.md). Cuando se configura un bot para acceder a una cuenta de correo electrónico, recibe un mensaje cuando llega un nuevo correo. El bot puede responder entonces tal y como indica su lógica de negocios. Por ejemplo, el bot podría enviar un correo electrónico de respuesta para confirmar un correo electrónico que se recibió con el mensaje "¡Hola! Le agradecemos su pedido. Comenzaremos a procesarlo inmediatamente".

> [!WARNING]
> Es una infracción del [Código de conducta](https://www.botframework.com/Content/Microsoft-Bot-Framework-Preview-Online-Services-Agreement.htm) de Bot Framework crear "spambots", lo que incluye bots que envían correo masivo no deseado ni solicitado.

## <a name="configure-email-credentials"></a>Configuración de las credenciales de correo electrónico

Para conectar un bot al canal de correo electrónico, debe escribir las credenciales de Office 365 en la configuración de canal de correo electrónico.
No se admite la autenticación federada con un proveedor que reemplace a AAD.

> [!NOTE]
> No use sus propias cuentas de correo electrónico personal con bots, ya que todos los mensajes enviados a esa cuenta de correo electrónico se reenviarán al bot. Como consecuencia, el bot puede enviar una respuesta al remitente de manera inadecuada. Por este motivo, los bots solo deben usar cuentas de correo electrónico de Office 365 dedicadas.

Para agregar el canal de correo electrónico, abra el bot en [Azure Portal](https://portal.azure.com/), haga clic en la hoja **Canales** y luego en **Correo electrónico**. Escriba sus credenciales de correo electrónico válidas y haga clic en **Guardar**.

![Escribir las credenciales de correo electrónico](~/media/bot-service-channel-connect-email/bot-service-channel-connect-email-credentials.png)

El canal de correo electrónico solo funciona actualmente con Office 365. Otros servicios de correo electrónico no se admiten actualmente.

## <a name="customize-emails"></a>Personalización de los mensajes de correo electrónico

El canal de correo electrónico admite el envío de propiedades personalizadas para crear correos electrónicos personalizados más avanzados mediante la propiedad `channelData`.

[!INCLUDE [Email channelData table](~/includes/snippet-channelData-email.md)]

El mensaje de ejemplo siguiente muestra un archivo JSON que incluye estas propiedades `channelData`.

```json
{
    "type": "message",
    "locale": "en-Us",
    "channelID": "email",
    "from": { "id": "mybot@mydomain.com", "name": "My bot"},
    "recipient": { "id": "joe@otherdomain.com", "name": "Joe Doe"},
    "conversation": { "id": "123123123123", "topic": "awesome chat" },
    "channelData":
    {
        "htmlBody": "<html><body style = /"font-family: Calibri; font-size: 11pt;/" >This is more than awesome.</body></html>",
        "subject": "Super awesome message subject",
        "importance": "high",
        "ccRecipients": "Yasemin@adatum.com;Temel@adventure-works.com"
    }
}
```

::: moniker range="azure-bot-service-3.0"
Para más información sobre el uso de `channelData`, consulte el ejemplo de [Node.js](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-ChannelData) o la documentación de [.NET](~/dotnet/bot-builder-dotnet-channeldata.md).
::: moniker-end

::: moniker range="azure-bot-service-4.0"
Para más información acerca del uso de `channelData`, consulte [implementación de la funcionalidad específica de un canal](~/v4sdk/bot-builder-channeldata.md).
::: moniker-end

## <a name="additional-resources"></a>Recursos adicionales

<!-- Put whole list in monikers, even though it's just the second item that needs to be different. -->
::: moniker range="azure-bot-service-3.0"
* Conexión de un bot a [canales](~/bot-service-manage-channels.md)
* [Implementación de una funcionalidad específica de canal](dotnet/bot-builder-dotnet-channeldata.md) con Bot Builder SDK para .NET
* Uso de [Channel Inspector](bot-service-channel-inspector.md) para ver cómo un canal representa una característica determinada de la aplicación de bot
::: moniker-end
::: moniker range="azure-bot-service-4.0"
* Conexión de un bot a [canales](~/bot-service-manage-channels.md)
* [Implementación de una funcionalidad específica de canal](~/v4sdk/bot-builder-channeldata.md) con Bot Builder SDK para .NET
* Uso de [Channel Inspector](bot-service-channel-inspector.md) para ver cómo un canal representa una característica determinada de la aplicación de bot
::: moniker-end

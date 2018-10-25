---
title: Protección del bot | Microsoft Docs
description: Obtenga información sobre cómo proteger el bot mediante el uso de la autenticación de Bot Framework y HTTPS.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: deb184483bf7e0963f827b20377291ab971c1516
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997343"
---
# <a name="secure-your-bot"></a>Protección del bot

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Su bot puede conectarse a muchos canales de comunicación diferentes (Skype, SMS, correo electrónico, etc.) a través del servicio de Bot Framework Connector. En este artículo se describe cómo proteger su bot con la autenticación Bot Framework y HTTPS.

## <a name="use-https-and-bot-framework-authentication"></a>Usar la autenticación de Bot Framework y HTTPS

Para asegurarse de que solo el [conector](bot-builder-dotnet-concepts.md#connector) de Bot Framework Connector pueda acceder al punto de conexión del bot, configure el punto de conexión del bot para usar únicamente HTTPS y habilite la autenticación de Bot Framework [registrando](~/bot-service-quickstart-registration.md) el bot para que adquiera su appID y contraseña. 

## <a name="configure-authentication-for-your-bot"></a>Configurar la autenticación para el bot

Especifique el appID y la contraseña del bot en el archivo web.config del bot. 

> [!NOTE]
> Para buscar el **AppID** y **AppPassword** del bot, consulte la sección [MicrosoftAppID and MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword) (MicrosoftAppID y MicrosoftAppPassword).

```xml
<appSettings>
    <add key="MicrosoftAppId" value="_appIdValue_" />
    <add key="MicrosoftAppPassword" value="_passwordValue_" />
</appSettings>
```

A continuación, utilice el atributo `[BotAuthentication]` para especificar las credenciales de autenticación al usar el SDK de Bot Builder para .NET para crear su bot. 

Para usar las credenciales de autenticación que se almacenan en el archivo web.config, especifique el atributo `[BotAuthentication]` sin parámetros.

[!code-csharp[Use botAuthentication attribute](../includes/code/dotnet-security.cs#attribute1)]

Para usar otros valores para las credenciales de autenticación, especifique el atributo `[BotAuthentication]` e inserte esos valores.

[!code-csharp[Use botAuthentication attribute with parameters](../includes/code/dotnet-security.cs#attribute2)]

## <a name="additional-resources"></a>Recursos adicionales

- [Bot Builder SDK para .NET](bot-builder-dotnet-overview.md)
- [Key concepts in the bot Builder SDK for .NET](bot-builder-dotnet-concepts.md) (Conceptos clave del SDK de Bot Builder para .NET)
- [Register a bot with the Bot Framework](~/bot-service-quickstart-registration.md) (Registrar un bot con Bot Framework)

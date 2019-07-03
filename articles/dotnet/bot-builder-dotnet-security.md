---
title: Protección del bot | Microsoft Docs
description: Obtenga información sobre cómo proteger el bot mediante el uso de la autenticación de Bot Framework y HTTPS.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8741dee2724d5e3c1e650b86957aabec5716da59
ms.sourcegitcommit: 697a577d72aaf91a0834d4b4c2ef5aa11291f28f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/01/2019
ms.locfileid: "67496639"
---
# <a name="secure-your-bot"></a>Protección del bot

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Su bot puede conectarse a muchos canales de comunicación diferentes (Skype, SMS, correo electrónico, etc.) a través del servicio de Bot Framework Connector. En este artículo se describe cómo proteger su bot con la autenticación Bot Framework y HTTPS.

## <a name="use-https-and-bot-framework-authentication"></a>Usar la autenticación de Bot Framework y HTTPS

Para asegurarse de que solo el [conector](bot-builder-dotnet-concepts.md#connector) de Bot Framework Connector pueda acceder al punto de conexión del bot, configure el punto de conexión del bot para usar únicamente HTTPS y habilite la autenticación de Bot Framework [registrando](~/bot-service-quickstart-registration.md) el bot para que adquiera su appID y contraseña. 

## <a name="configure-authentication-for-your-bot"></a>Configurar la autenticación para el bot

Especifique el appID y la contraseña del bot en el archivo web.config del bot. 

> [!NOTE]
> Para buscar los valores **AppID** y **AppPassword** del bot, consulte [MicrosoftAppID y MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

```xml
<appSettings>
    <add key="MicrosoftAppId" value="_appIdValue_" />
    <add key="MicrosoftAppPassword" value="_passwordValue_" />
</appSettings>
```

A continuación, utilice el atributo `[BotAuthentication]` para especificar las credenciales de autenticación al usar Bot Framework SDK para .NET para crear el bot. 

Para usar las credenciales de autenticación que se almacenan en el archivo web.config, especifique el atributo `[BotAuthentication]` sin parámetros.

[!code-csharp[Use botAuthentication attribute](../includes/code/dotnet-security.cs#attribute1)]

Para usar otros valores para las credenciales de autenticación, especifique el atributo `[BotAuthentication]` e inserte esos valores.

[!code-csharp[Use botAuthentication attribute with parameters](../includes/code/dotnet-security.cs#attribute2)]

## <a name="additional-resources"></a>Recursos adicionales

- [SDK de Bot Framework para .NET](bot-builder-dotnet-overview.md)
- [Key concepts in the bot Builder SDK for .NET](bot-builder-dotnet-concepts.md) (Conceptos clave del SDK de Bot Builder para .NET)
- [Register a bot with the Bot Framework](~/bot-service-quickstart-registration.md) (Registrar un bot con Bot Framework)

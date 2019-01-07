---
title: Creación de un bot con Bot Builder SDK para .NET | Microsoft Docs
description: Cree un bot con Bot Builder SDK para .NET, un marco de trabajo eficaz para la creación de bots.
keywords: SDK Bot Builder, creación de un bot, guía de inicio rápido, .NET, introducción, bot de C#
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/19/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d40b203ccd044992c026a592d5f86b0881754a41
ms.sourcegitcommit: f7a8f05fc05ff4a7212a437d540485bf68831604
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/21/2018
ms.locfileid: "53735925"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-net"></a>Creación de un bot con Bot Builder SDK para .NET
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Esta guía de inicio rápido le orienta en el desarrollo de un bot con la plantilla de C# y, a continuación, con la prueba con Bot Framework Emulator.

## <a name="prerequisites"></a>Requisitos previos
- Visual Studio [2017](https://www.visualstudio.com/downloads)
- Plantilla de Bot Builder SDK v4 para [C#](https://aka.ms/bot-vsix)
- Bot Framework [Emulator](https://aka.ms/Emulator-wiki-getting-started)
- Conocimientos sobre [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) y la programación asincrónica en [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index)

## <a name="create-a-bot"></a>Creación de un bot
Instale la plantilla BotBuilderVSIX.vsix que descargó en la sección de requisitos previos.

En Visual Studio, cree un nuevo proyecto de bot con la plantilla Echo Bot V4 de Bot Builder.

![Proyecto de Visual Studio](../media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> Si es necesario, cambie el tipo de compilación del proyecto a ``.Net Core 2.1``; si es necesario, actualice los [paquetes NuGet](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio).

Gracias a la plantilla, el proyecto contiene todo el código necesario para crear el bot en esta guía de inicio rápido. Realmente no necesita escribir ningún código adicional.

## <a name="start-your-bot-in-visual-studio"></a>Inicio del bot en Visual Studio

Al hacer clic en el botón de ejecución, Visual Studio compila la aplicación, la implementa en localhost e inicia el explorador web para mostrar la página `default.htm` de la aplicación. En este momento, el bot se ejecuta de forma local.

## <a name="start-the-emulator-and-connect-your-bot"></a>Inicio del emulador y conexión del bot

A continuación, inicie el emulador y, después, conéctese al bot en el emulador:

1. Haga clic en el vínculo **Open bot** (Abrir bot) de la pestaña de bienvenida del emulador. 
2. Seleccione el archivo .bot ubicado en el directorio donde se creó la solución de Visual Studio.

## <a name="interact-with-your-bot"></a>Interacción con el bot

Envíe un mensaje al bot y este responderá con un mensaje.

![Ejecución del emulador](../media/emulator-v4/emulator-running.png)

> [!NOTE]
> Si observa que el mensaje no se puede enviar, puede que deba reiniciar la máquina dado que ngrok no ha obtenido aún los privilegios necesarios en el sistema (solo debe hacerse una vez).

## <a name="additional-resources"></a>Recursos adicionales

Consulte [Tunelización (ngrok)](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-(ngrok)) para más información sobre cómo conectarse a un bot hospedado de forma remota.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Funcionamiento de los bots](../v4sdk/bot-builder-basics.md) 

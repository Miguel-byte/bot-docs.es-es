---
ms.openlocfilehash: 04f9101d0cf29618fb7d50e126c008190064a831
ms.sourcegitcommit: 3e3c9986b95532197e187b9cc562e6a1452cbd95
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2019
ms.locfileid: "65199446"
---
## <a name="prerequisites"></a>Requisitos previos
- Visual Studio [2017 o cualquier versión posterior](https://www.visualstudio.com/downloads)
- Plantilla de Bot Framework SDK v4 para [C#](https://aka.ms/bot-vsix)
- [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)
- Conocimientos de [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) y [programación asincrónica en C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index)

## <a name="create-a-bot"></a>Creación de un bot
Instale la plantilla BotBuilderVSIX.vsix que descargó en la sección de requisitos previos.

En Visual Studio, cree un proyecto de bot mediante la plantilla **Echo Bot (Bot Framework v4)**.

![Proyecto de Visual Studio](~/media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> Si es necesario, cambie el tipo de compilación del proyecto a ``.Net Core 2.1``. Si es necesario, actualice los [paquetes de NuGet](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio) de `Microsoft.Bot.Builder`.

Gracias a la plantilla, el proyecto contiene todo el código necesario para crear el bot en esta guía de inicio rápido. Realmente no necesita escribir ningún código adicional.

## <a name="start-your-bot-in-visual-studio"></a>Inicio del bot en Visual Studio

Al hacer clic en el botón de ejecución, Visual Studio compila la aplicación, la implementa en localhost e inicia el explorador web para mostrar la página `default.htm` de la aplicación. En este momento, el bot se ejecuta de forma local.

## <a name="start-the-emulator-and-connect-your-bot"></a>Inicio del emulador y conexión del bot

A continuación, inicie el emulador y, después, conéctese al bot en el emulador:

1. Haga clic en el vínculo **Create a new bot configuration** (Crear configuración de bot) en la pestaña de bienvenida del emulador. 
2. Rellene los campos del bot y haga clic en **Save and connect** (Guardar y conectar).

## <a name="interact-with-your-bot"></a>Interacción con el bot

Envíe un mensaje al bot y este responderá con un mensaje.

![Ejecución del emulador](~/media/emulator-v4/emulator-running.png)

> [!NOTE]
> Si observa que el mensaje no se puede enviar, puede que deba reiniciar la máquina dado que ngrok no ha obtenido aún los privilegios necesarios en el sistema (solo debe hacerse una vez).

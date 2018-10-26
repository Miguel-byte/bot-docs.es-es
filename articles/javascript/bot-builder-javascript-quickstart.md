---
title: Creación de un bot con el SDK de Bot Builder para JavaScript | Microsoft Docs
description: Cree un bot de forma rápida con el SDK de Bot Builder para JavaScript.
keywords: inicio rápido, sdk de bot builder, introducción
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3e721c9142ffb1511ef926f5b1caca782e0919ed
ms.sourcegitcommit: 54ed5000c67a5b59e23b667547565dd96c7302f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/13/2018
ms.locfileid: "49315091"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-javascript"></a>Creación de un bot con el SDK de Bot Builder para JavaScript

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Este inicio rápido le guía por la creación de un bot mediante el generador Bot Builder de Yeoman y el SDK de Bot Builder para JavaScript, y después se prueba con el emulador de Bot Framework. 

## <a name="prerequisites"></a>Requisitos previos

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.js](https://nodejs.org/)
- [Yeoman](http://yeoman.io/), que puede usar un generador para crear un bot de forma automática.
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator)
- Conocimientos sobre [restify](http://restify.com/) y programación asincrónica en JavaScript.

> [!NOTE]
> En algunas instalaciones, el paso de instalación de restify genera un error relacionado con node-gyp.
> Si este es el caso, intente ejecutar `npm install -g windows-build-tools`.

## <a name="create-a-bot"></a>Creación de un bot

Abra un símbolo del sistema con privilegios elevados, cree un directorio e inicialice el paquete para el bot.

```bash
md myJsBots
cd myJsBots
```

Asegúrese de que la versión de npm está actualizada.
```bash
npm i npm
```

Después, instale Yeoman y el generador para JavaScript.

```bash
npm install -g yo
npm install -g generator-botbuilder
```

Luego, use el generador para crear un bot de eco.

```bash
yo botbuilder
```

Yeoman le solicitará alguna información con la que se va a crear el bot.

- Escriba un nombre para el bot.
- Escriba una descripción.
- Elija el lenguaje para el bot, ya sea `JavaScript` o `TypeScript`.
- Elija la plantilla `Echo`.

Gracias a la plantilla, el proyecto contiene todo el código necesario para crear el bot en esta guía de inicio rápido. Realmente no necesita escribir ningún código adicional.

> [!NOTE]
> Para un bot básico necesitará un modelo de lenguaje de LUIS. Puede crear uno en [luis.ai](https://www.luis.ai). Después de crear el modelo, actualice el archivo .bot. El archivo del bot deb tener un aspecto similar a [este](../v4sdk/bot-builder-service-file.md). 

## <a name="start-your-bot"></a>Inicio del bot

En Powershell/Bash, cambie el directorio al que creó para el bot e inícielo con `npm start`. En este momento, el bot se ejecuta de forma local.

## <a name="start-the-emulator-and-connect-your-bot"></a>Inicio del emulador y conexión del bot
1. Inicie el emulador.
2. Haga clic en el vínculo **Open bot** (Abrir bot) de la pestaña de bienvenida del emulador.
3. Seleccione el archivo .bot ubicado en el directorio donde se creó el proyecto.

Envíe un mensaje al bot y este responderá con un mensaje.
![Emulador en ejecución](../media/emulator-v4/emulator-running.png)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Funcionamiento de los bots](../v4sdk/bot-builder-basics.md) 

---
title: Depuración de un bot | Microsoft Docs
description: Obtenga información sobre cómo depurar un bot creado con Bot Service.
author: v-ducvo
ms.author: kamrani
keywords: Bot Framework SDK, depurar bot, probar bot, emulador de bot, emulador
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/26/2019
ms.openlocfilehash: 27af606506eb19b98a327e276a00201ac417c7e4
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298180"
---
# <a name="debug-a-bot"></a>Depuración de un bot

En este artículo se describe cómo depurar el bot mediante un entorno de desarrollo integrado (IDE), como Visual Studio o Visual Studio Code, y el emulador de Bot Framework. Aunque puede usar estos métodos para depurar cualquier bot localmente, en este artículo se usa un [bot de C#](~/dotnet/bot-builder-dotnet-sdk-quickstart.md) o uno de [Javascript](~/javascript/bot-builder-javascript-quickstart.md) que se creó en el inicio rápido.

> [!NOTE]
> En este artículo, se usa Bot Framework Emulator para enviar y recibir mensajes desde el bot durante la depuración. Si busca otras maneras de depurar el bot mediante Bot Framework Emulator, lea el artículo [Depuración con Bot Framework Emulator](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0). 

## <a name="prerequisites"></a>Requisitos previos 
- Descargue e instale el [emulador de Bot Framework](https://aka.ms/Emulator-wiki-getting-started).
- Descargue e instale [Visual Studio Code](https://code.visualstudio.com) o [Visual Studio](https://www.visualstudio.com/downloads) (Community Edition o superior).

<!-- ### Debug a JavaScript bot using command-line and emulator

To run a JavaScript bot using the command line and testing the bot with the emulator, do the following:
1. From the command line, change directory to your bot project directory.
1. Start the bot by running the command **node app.js**.
1. Start the emulator and connect to the bot's endpoint (e.g.: **http://localhost:3978/api/messages**). If this is the first time you are running 
the bot then click **File > New Bot** and follow the instructions on screen. Otherwise, click **File > Open Bot** to open an existing bot. 
Since this bot is running locally on your computer, you can leave the **MicrosoftAppId** and **MicrosoftAppPassword** fields blank. 
For more information, see [Debug with the Emulator](bot-service-debug-emulator.md).
1. From the emulator, send your bot a message (e.g.: send the message "Hi"). 
1. Use the **Inspector** and **Log** panels on the right side of the emulator window to debug your bot. For example, clicking on any of the messages bubble (e.g.: the "Hi" message bubble in the screenshot below) will show you the detail of that message in the **Inspector** panel. You can use it to view requests and responses as messages are exchanged between the emulator and the bot. Alternatively, you can click on any of the linked text in the **Log** panel to view the details in the **Inspector** panel.


   ![Inspector panel on the Emulator](~/media/bot-service-debug-bot/emulator_inspector.png) -->

## <a name="debug-a-javascript-bot-using-breakpoints-in-visual-studio-code"></a>Depurar un bot de JavaScript mediante puntos de interrupción en Visual Studio Code

En Visual Studio Code, se pueden establecer puntos de interrupción y ejecutar el bot en modo de depuración para recorrer el código. Para establecer puntos de interrupción en VS Code, siga estos pasos:

1. Inicie VS Code y abra la carpeta del proyecto de bot.
2. En la barra de menús, haga clic en **Depurar** y en **Iniciar depuración**. Si se le pide que seleccione un motor de tiempo de ejecución para ejecutar el código, seleccione **Node.js**. En este momento, el bot se ejecuta de forma local. 
3. Haga clic en el archivo **.js** y establezca los puntos de interrupción según sea necesario. En VS Code, puede establecer puntos de interrupción si mantiene el ratón sobre la columna a la izquierda de los números de línea. Aparecerá un pequeño punto de color rojo. Si hace clic en el punto, se establece el punto de interrupción. Si vuelve a hacer clic en el punto, el punto de interrupción se quita.
   ![Establecer punto de interrupción en VS Code](~/media/bot-service-debug-bot/breakpoint-set.png)
4. Inicie Bot Framework Emulator y conéctese al bot como se describe en el artículo [Depuración con Bot Framework Emulator](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0). 
5. Desde el emulador, envíe un mensaje al bot (por ejemplo, envíe el mensaje "Hola"). La ejecución se detendrá en la línea donde ha colocado el punto de interrupción.
   ![Depuración en VS Code](~/media/bot-service-debug-bot/breakpoint-caught.png)

## <a name="debug-a-c-bot-using-breakpoints-in-visual-studio"></a>Depurar un bot de C# mediante puntos de interrupción en Visual Studio

En Visual Studio (VS), se pueden establecer puntos de interrupción y ejecutar el bot en modo de depuración para recorrer el código. Para establecer puntos de interrupción en VS, siga estos pasos:

1. Navegue hasta la carpeta del bot y abra el archivo **.sln**. La solución se abrirá en VS.
2. Desde la barra de menús, haga clic en **Compilar** y en **Compilar solución**.
3. En el **Explorador de soluciones**, haga clic en el archivo **.cs** y establezca puntos de interrupción según sea necesario. Este archivo define la lógica principal del bot. En VS, puede establecer puntos de interrupción si mantiene el ratón sobre la columna a la izquierda de los números de línea. Aparecerá un pequeño punto de color rojo. Si hace clic en el punto, se establece el punto de interrupción. Si vuelve a hacer clic en el punto, el punto de interrupción se quita.
4. En el menú, haga clic en **Depurar** y en **Iniciar depuración**. En este momento, el bot se ejecuta de forma local. 

<!--
   > [!NOTE]
   > If you get the "Value cannot be null" error, check to make sure your **Table Storage** setting is valid.
   > The **EchoBot** is default to using **Table Storage**. To use Table Storage in your bot, you need the table *name* and *key*. If you do not have a Table Storage instance ready, you can create one or for testing purposes, you can comment out the code that uses **TableBotDataStore** and uncomment the line of code that uses **InMemoryDataStore**. The **InMemoryDataStore** is intended for testing and prototyping only.
-->
   ![Establecer punto de interrupción en VS](~/media/bot-service-debug-bot/breakpoint-set-vs.png)

5. Inicie el emulador de Bot Framework y conéctese al bot como se describe en la sección anterior. 
6. Desde el emulador, envíe un mensaje al bot (por ejemplo, envíe el mensaje "¡Hi" [¡Hola!]). La ejecución se detendrá en la línea donde ha colocado el punto de interrupción.
   ![Depuración en VS](~/media/bot-service-debug-bot/breakpoint-caught-vs.png)
::: moniker range="azure-bot-service-3.0" 

## <a id="debug-csharp-serverless"></a> Depurar un bot de Functions de C\# de plan de consumo

El entorno de C\# sin servidor de plan de consumo de Bot Service tiene más en común con Node.js que una aplicación C\# típica porque requiere un host en tiempo de ejecución, al igual que el motor de nodos. En Azure, el runtime forma parte del entorno de hospedaje en la nube, pero ese entorno se debe replicar localmente en el escritorio. 

### <a name="prerequisites"></a>Requisitos previos

Antes de poder depurar un bot de C# de plan de consumo, se deben completar estas tareas.

- Descargue el código fuente del bot (desde Azure), como se describe en [Configuración de la implementación continua](bot-service-continuous-deployment.md).
- Descargue e instale el [emulador de Bot Framework](https://aka.ms/Emulator-wiki-getting-started).
- Instale la <a href="https://www.npmjs.com/package/azure-functions-cli" target="_blank">CLI de Azure Functions</a>.
- Instale la <a href="https://github.com/dotnet/cli" target="_blank">CLI de DotNet</a>.
  
Si quiere poder depurar el código mediante puntos de interrupción en Visual Studio 2017, también debe completar estas tareas.
  
- Descargue e instale <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017</a> (Community Edition o superior).
- Descargue e instale la <a href="https://visualstudiogallery.msdn.microsoft.com/e6bf6a3d-7411-4494-8a1e-28c1a8c4ce99" target="_blank">extensión Command Task Runner de Visual Studio</a>.

> [!NOTE]
> En la actualidad no se admite Visual Studio Code.

### <a name="debug-a-consumption-plan-c-functions-bot-using-the-emulator"></a>Depurar un bot de Functions de C# de plan de consumo con el emulador

La manera más sencilla de depurar el bot de forma local consiste en iniciarlo y, después, conectarse a él desde el emulador de Bot Framework. 
En primer lugar, abra un símbolo del sistema y vaya hasta la carpeta del repositorio donde se encuentra el archivo **project.json**. Después, ejecute el comando `dotnet restore` para restaurar los distintos paquetes a los que se hace referencia en el bot.

> [!NOTE]
> En Visual Studio 2017 se cambia la manera en que Visual Studio controla las dependencias. Mientras que en Visual Studio 2015 se usa **project.json** para controlar las dependencias, en Visual Studio 2017 se usa un modelo de **.csproj** cuando se carga en Visual Studio. Si usa Visual Studio 2017, <a href="https://aka.ms/bf-debug-project">descargue este archivo **.csproj**</a> a la carpeta **/messages** del repositorio antes de ejecutar el comando `dotnet restore`.

![Símbolo del sistema](~/media/bot-service-debug-bot/csharp-azureservice-debug-envconfig.png)

A continuación, ejecute `debughost.cmd` para cargar e iniciar el bot. 

![debughost.cmd ejecutado en el símbolo del sistema](~/media/bot-service-debug-bot/csharp-azureservice-debug-debughost.png)

En este momento, el bot se ejecuta de forma local. En la ventana de consola, copie el punto de conexión en el que escucha debughost (en este ejemplo, `http://localhost:3978`). Después, inicie el emulador de Bot Framework y pegue el punto de conexión en la barra de direcciones del emulador. Para este ejemplo, también se debe anexar `/api/messages` al punto de conexión. Como para la depuración local no se necesita la seguridad, puede dejar en blanco los campos **Id. de aplicación de Microsoft** y **Contraseña de aplicación de Microsoft**. Haga clic en **Conectar** para establecer una conexión con el bot mediante el punto de conexión especificado.

![Configuración del emulador](~/media/bot-service-debug-bot/mac-azureservice-emulator-config.png)

Después de conectar el emulador al bot, envíe un mensaje al bot escribiendo algún texto en el cuadro de texto que se encuentra en la parte inferior de la ventana del emulador (es decir, en la esquina inferior izquierda donde aparece **Escriba el mensaje...** ). Con los paneles **Registro** e **Inspector** del lado derecho de la ventana del emulador, puede ver las solicitudes y respuestas como mensajes que se intercambian entre el emulador y el bot.

![Pruebas a través del emulador](~/media/bot-service-debug-bot/mac-azureservice-debug-emulator.png)

Además, puede ver los detalles de registro en la ventana de consola.

![Ventana de consola](~/media/bot-service-debug-bot/csharp-azureservice-debug-debughostlogging.png)

::: moniker-end

## <a name="additional-resources"></a>Recursos adicionales

- Consulte [Solución de problemas generales](bot-service-troubleshoot-bot-configuration.md) y otros artículos de solución de problemas en esa sección.
- Consulte la sección [Depuración con el emulador](bot-service-debug-emulator.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Depuración del bot mediante archivos de transcripción](v4sdk/bot-builder-debug-transcript.md).

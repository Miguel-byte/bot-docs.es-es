---
title: Depuración de un bot creado con Bot Service | Microsoft Docs
description: Obtenga información sobre cómo depurar un bot creado con Bot Service.
author: v-ducvo
ms.author: v-ducvo
keywords: SDK de Bot Builder, implementación continua, App Service, emulador
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/13/2018
ms.openlocfilehash: 3312aa3508a7d9e57f1adc3e36c9d0325cdf6df6
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997552"
---
# <a name="debug-a-bot-service-bot"></a>Depuración de un bot de Bot Service

En este artículo se describe cómo depurar el bot mediante un entorno de desarrollo integrado (IDE), como Visual Studio o Visual Studio Code, y el emulador de Bot Framework. Aunque estos métodos se pueden usar para depurar cualquier bot localmente, en este artículo se usa el **EchoBot** creado en el artículo [Creación de un bot con Bot Service](bot-service-quickstart.md).

## <a name="debug-a-javascript-bot"></a>Depurar un bot de JavaScript

Siga los pasos descritos en esta sección para depurar un bot escrito en JavaScript.

### <a name="prerequisites"></a>Requisitos previos

Antes de poder depurar un bot de JavaScript, debe completar estas tareas.

- Descargue el código fuente del bot (desde Azure), tal y como se describe en [Descarga del código fuente del bot](bot-service-build-download-source-code.md).
- Descargue e instale el [emulador de Bot Framework](https://github.com/Microsoft/BotFramework-Emulator/releases).
- Descargue e instale un editor de código como <a href="https://code.visualstudio.com" target="_blank">Visual Studio Code</a>.

### <a name="debug-a-javascript-bot-using-command-line-and-emulator"></a>Depurar un bot de JavaScript mediante la línea de comandos y el emulador

Para ejecutar un bot de JavaScript mediante la línea de comandos y probarlo con el emulador, siga estos pasos:
1. Desde la línea de comandos, cambie el directorio al del proyecto del bot.
2. Ejecute el comando **node app.js** para iniciar el bot.
3. Inicie el emulador y conéctese al punto de conexión del bot (por ejemplo: **http://localhost:3978/api/messages**). Si es la primera vez que se ejecuta el bot, haga clic en **Archivo > Nuevo bot** y siga las instrucciones en pantalla. En caso contrario, haga clic en **Archivo > Abrir bot** para abrir un bot existente. Como este bot se ejecuta localmente en el equipo, puede dejar en blanco los campos **MSA app ID** (Id. de aplicación de MSA) y **MSA app password** (Contraseña de aplicación de MSA). Para obtener más información, vea [Depuración con el emulador](bot-service-debug-emulator.md).
4. Desde el emulador, envíe un mensaje al bot (por ejemplo, envíe el mensaje "¡Hi" [¡Hola!]). 
5. Use los paneles **Inspector** y **Registro** del lado derecho de la ventana del emulador para depurar el bot. Por ejemplo, al hacer clic en cualquiera de las burbujas de mensaje (como la burbuja de mensaje "Hi" (Hola) en la captura de pantalla siguiente), se mostrarán los detalles de ese mensaje en el panel **Inspector**. Puede usarlo para ver las solicitudes y respuestas como mensajes que se intercambian entre el emulador y el bot. Como alternativa, puede hacer clic en cualquier parte del texto vinculado en el panel **Registro** para ver los detalles en el panel **Inspector**.

   ![Panel Inspector en el emulador](~/media/bot-service-debug-bot/emulator_inspector.png)

### <a name="debug-a-javascript-bot-using-breakpoints-in-visual-studio-code"></a>Depurar un bot de JavaScript mediante puntos de interrupción en Visual Studio Code

Con un IDE como Visual Studio Code, se pueden establecer puntos de interrupción y ejecutar el bot en modo de depuración para recorrer el código. Para establecer puntos de interrupción en VS Code, siga estos pasos:

1. Inicie VS Code y abra la carpeta del proyecto de bot.
2. En la barra de menús, haga clic en **Depurar** y en **Iniciar depuración**. Si se le pide que seleccione un motor de tiempo de ejecución para ejecutar el código, seleccione **Node.js**. En este momento, el bot se ejecuta de forma local. 

   > [!NOTE]
   > Si recibe el error "El valor no puede ser nulo", asegúrese de que la configuración de **Table Storage** sea válida.
   > **EchoBot** usa **Table Storage** de forma predeterminada. Para usar Table Storage en el bot, se necesitan el *nombre* y la *clave* de la tabla. Si no tiene una instancia de Table Storage lista, puede crear una o, con fines de prueba, puede comentar el código que usa **TableBotDataStore** y quitar la marca de comentario de la línea de código en la que se usa **InMemoryDataStore**. **InMemoryDataStore** solo está destinado para la creación de prototipos y pruebas.

3. Establezca los puntos de interrupción según sea necesario. En VS Code, puede establecer puntos de interrupción si mantiene el ratón sobre la columna a la izquierda de los números de línea. Aparecerá un pequeño punto de color rojo. Si hace clic en el punto, se establece el punto de interrupción. Si vuelve a hacer clic en el punto, el punto de interrupción se quita.

   ![Establecer punto de interrupción en VS Code](~/media/bot-service-debug-bot/breakpoint-set.png)

4. Inicie el emulador de Bot Framework y conéctese al bot como se describe en la sección anterior. 
5. Desde el emulador, envíe un mensaje al bot (por ejemplo, envíe el mensaje "¡Hi" [¡Hola!]). La ejecución se detendrá en la línea donde ha colocado el punto de interrupción.

   ![Depuración en VS Code](~/media/bot-service-debug-bot/breakpoint-caught.png)


## <a name="debug-a-c-bot"></a>Depuración de un bot de C#

Siga los pasos descritos en esta sección para depurar un bot escrito en C#.


### <a name="prerequisites"></a>Requisitos previos

Antes de poder depurar un bot de C# de aplicación web, debe completar estas tareas.

- Descargue el código fuente del bot (desde Azure), tal y como se describe en [Descarga del código fuente del bot](bot-service-build-download-source-code.md).
- Descargue e instale el [emulador de Bot Framework](https://github.com/Microsoft/BotFramework-Emulator/releases).
- Descargue e instale <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017</a> (Community Edition o superior).

### <a name="debug-a-c-bot-using-breakpoints-in-visual-studio"></a>Depurar un bot de C# mediante puntos de interrupción en Visual Studio

Con un IDE como Visual Studio (VS), se pueden establecer puntos de interrupción y ejecutar el bot en modo de depuración para recorrer el código. Para establecer puntos de interrupción en VS, siga estos pasos:

1. Navegue hasta la carpeta del bot y abra el archivo **.sln**. La solución se abrirá en VS.
2. Desde la barra de menús, haga clic en **Compilar** y en **Compilar solución**.
3. Desde el **Explorador de soluciones**, expanda la carpeta **Cuadros de diálogo** y haga clic en **EchoDialog.cs**. Este archivo define la lógica principal del bot.
4. En la barra de menús, haga clic en **Depurar** y en **Iniciar depuración**. En este momento, el bot se ejecuta de forma local. 

   > [!NOTE]
   > Si recibe el error "El valor no puede ser nulo", asegúrese de que la configuración de **Table Storage** sea válida.
   > **EchoBot** usa **Table Storage** de forma predeterminada. Para usar Table Storage en el bot, se necesitan el *nombre* y la *clave* de la tabla. Si no tiene una instancia de Table Storage lista, puede crear una o, con fines de prueba, puede comentar el código que usa **TableBotDataStore** y quitar la marca de comentario de la línea de código en la que se usa **InMemoryDataStore**. **InMemoryDataStore** solo está destinado para la creación de prototipos y pruebas.

5. Establezca los puntos de interrupción según sea necesario. En VS, puede establecer puntos de interrupción si mantiene el ratón sobre la columna a la izquierda de los números de línea. Aparecerá un pequeño punto de color rojo. Si hace clic en el punto, se establece el punto de interrupción. Si vuelve a hacer clic en el punto, el punto de interrupción se quita.

   ![Establecer punto de interrupción en VS](~/media/bot-service-debug-bot/breakpoint-set-vs.png)

6. Inicie el emulador de Bot Framework y conéctese al bot como se describe en la sección anterior. 
7. Desde el emulador, envíe un mensaje al bot (por ejemplo, envíe el mensaje "¡Hi" [¡Hola!]). La ejecución se detendrá en la línea donde ha colocado el punto de interrupción.

   ![Depuración en VS](~/media/bot-service-debug-bot/breakpoint-caught-vs.png)

::: moniker range="azure-bot-service-3.0" 

## <a id="debug-csharp-serverless"></a> Depurar un bot de Functions de C\# de plan de consumo

El entorno de C\# sin servidor de plan de consumo de Bot Service tiene más en común con Node.js que una aplicación C\# típica porque requiere un host en tiempo de ejecución, al igual que el motor de nodos. En Azure, el runtime forma parte del entorno de hospedaje en la nube, pero ese entorno se debe replicar localmente en el escritorio. 

### <a name="prerequisites"></a>Requisitos previos

Antes de poder depurar un bot de C# de plan de consumo, se deben completar estas tareas.

- Descargue el código fuente del bot (desde Azure), como se describe en [Configuración de la implementación continua](bot-service-continuous-deployment.md).
- Descargue e instale el [emulador de Bot Framework](https://github.com/Microsoft/BotFramework-Emulator/releases).
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

Después de conectar el emulador al bot, envíe un mensaje al bot escribiendo algún texto en el cuadro de texto que se encuentra en la parte inferior de la ventana del emulador (es decir, en la esquina inferior izquierda donde aparece **Escriba el mensaje...**). Con los paneles **Registro** e **Inspector** del lado derecho de la ventana del emulador, puede ver las solicitudes y respuestas como mensajes que se intercambian entre el emulador y el bot.

![Pruebas a través del emulador](~/media/bot-service-debug-bot/mac-azureservice-debug-emulator.png)

Además, puede ver los detalles de registro en la ventana de consola.

![Ventana de consola](~/media/bot-service-debug-bot/csharp-azureservice-debug-debughostlogging.png)

::: moniker-end

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Depuración con el emulador](bot-service-debug-emulator.md).

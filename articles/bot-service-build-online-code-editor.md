---
title: Compilación de un bot con el editor de código en línea de Azure | Microsoft Docs
description: Obtenga información sobre cómo compilar el bot mediante el editor de código en línea en Bot Service.
keywords: editor de código en línea, azure portal, bot de functions
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: tools
ms.date: 09/21/2018
ms.openlocfilehash: ba8e87073f888f21afb806221108bd6b5391744c
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997242"
---
# <a name="edit-a-bot-with-online-code-editor"></a>Edición de un bot con el editor de código en línea

Puede usar el editor de código en línea para compilar el bot sin necesidad de un IDE. En este tema se mostrará cómo abrir el código del bot en el editor de código en línea. 

## <a name="edit-bot-source-code-in-online-code-editor"></a>Edición del código fuente del bot en el editor de código en línea

Para editar el código fuente de un bot en el editor de código en línea, haga lo siguiente para el tipo específico de bot que tiene.

### <a name="web-app-bot"></a>Bot de aplicación web
1. Inicie sesión en [Azure Portal](http://portal.azure.com) y abra la hoja para el bot.
2. En la sección **Administración de bots**, haga clic en **Crear**.
3. Haga clic en **Open online code editor** (Abrir el editor de código en línea). El código del bot se abrirá en una nueva ventana del explorador. 

   ![Abrir el editor de código en línea](~/media/azure-bot-build/open-online-code-editor.png)

   La estructura del archivo en el directorio **WWWRoot** será distinta en función del lenguaje del bot. Por ejemplo, si tiene un bot de C#, es posible que el directorio **WWWRoot** tenga este aspecto:

   ![Estructura de archivos de C#](~/media/azure-bot-build/cs-wwwroot-structure.png)

   Si tiene un bot de Node.js, es posible que el directorio **WWWRoot** tenga este aspecto:

   ![Estructura de archivos de Node.js](~/media/azure-bot-build/node-wwwroot-structure.png)

4. Modifique el código. Por ejemplo, para bots de C#, tiene un archivo .cs. En el caso de los bots de Node.js, puede empezar con el archivo App.js.

   > [!NOTE]
   > Si bien puede modificar el código de los archivos de origen actuales del proyecto, no es posible crear archivos de origen nuevos con el editor de código en línea. Para agregar nuevos archivos de origen al bot, deberá [descargar el proyecto de origen](bot-service-build-download-source-code.md), agregar los archivos y volver a publicar los cambios en Azure.

5. Para los bots de C# que se encuentran en un **plan de consumo** y todos los bots de Node.js, el bot se guarda automáticamente. 

6. En el caso de los bots de C# que se encuentren en un plan de **App Service**, abra la hoja **Consola** y envíe el comando **build.cmd**. 

   ![Compilación del proyecto en la hoja Consola](~/media/azure-bot-build/cs-console-build-cmd.png)
 
   > [!NOTE]
   > Si este comando no se compila, intente reiniciar el servicio de aplicación del bot e intente volver a realizar la compilación. Para reiniciar el servicio de aplicación, en la hoja del bot, haga clic en **All App service settings** (Toda la configuración de App Service) y haga clic en el botón **Reiniciar**.
   > ![Reinicio de una aplicación web](~/media/azure-bot-build/open-online-code-editor-restart-appservice.png)

7. Vuelva a Azure Portal y haga clic en **Test in Web Chat** (Probar en Chat en web) para probar los cambios. Si ya tiene abierto el Chat web para este bot, haga clic en **Start over** (Volver a empezar) para ver los cambios nuevos.

### <a name="functions-bot"></a>Bot de Functions

1. Inicie sesión en [Azure Portal](http://portal.azure.com) y abra la hoja para el bot.
2. En la sección **Administración de bots**, haga clic en **Crear**.
3. Haga clic en **Open this bot in Azure Functions** (Abrir este bot en Azure Functions). Esta acción abrirá el bot con la interfaz de usuario de <a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a>. 
4. Modifique el código. Por ejemplo, actualice el código de los mensajes de la función. La captura de pantalla siguiente muestra el código de los mensajes de un bot de Functions para Node.js.

   ![Editor de código de mensajes del bot de Functions](~/media/azure-bot-build/functions-messages-code.png)

5. Guarde los cambios del código.
6. Vuelva a Azure Portal y haga clic en **Test in Web Chat** (Probar en Chat en web) para probar los cambios. Si ya tiene abierto el Chat web para este bot, haga clic en **Start over** (Volver a empezar) para ver los cambios nuevos.

## <a name="next-steps"></a>Pasos siguientes
Ahora que sabe cómo editar el código del bot con el editor de código en línea, también puede compilar localmente el bot mediante el IDE de su preferencia.

> [!div class="nextstepaction"]
> [Descarga del código fuente del bot](bot-service-build-download-source-code.md)

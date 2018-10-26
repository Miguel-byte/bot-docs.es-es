---
title: Descarga y reimplementación del código fuente de un bot | Microsoft Docs
description: Aprenda a descargar y publicar un bot de Bot Service.
keywords: descargar código fuente, volver a implementar, implementar, archivo zip, publicar
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/26/2018
ms.openlocfilehash: ee7a7a9f1b4c06f8ad762f750099383e218d98f2
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326432"
---
# <a name="download-and-redeploy-bot-code"></a>Descarga y reimplementación del código de bot
Azure Bot Service le permite descargar el proyecto de código fuente completo del bot, de forma que pueda trabajar localmente con el IDE que prefiera. Cuando haya terminado de actualizar el código, puede volver a publicar los cambios en Azure Portal. Se mostrará cómo descargar el código mediante Azure Portal y `az` cli. También se describirá cómo volver a implementar el código de bot actualizado mediante Visual Studio y `az` cli. Puede elegir el método que más le convenga.

## <a name="prerequisites"></a>Requisitos previos
- Instalación de la [CLI de Azure](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest)
- Instale la extensión az botservice mediante el comando `az extension add -n botservice`.

### <a name="download-code-using-the-azure-portal"></a>Descarga del código mediante Azure Portal
Para descargar el código desde [Azure Portal](https://portal.azure.com), lleve a cabo los siguientes pasos:
1. Abra la hoja del bot.
1. En la sección **Bot management** (Administración de bots), haga clic en **Compilar**.
1. En **Download source code** (Descargar el código fuente), haga clic en **Download zip file** (Descargar archivo ZIP).
1. Espere a que Azure prepare el URI de descarga y, luego, haga clic en **Download zip file** (Descargar archivo ZIP) en la notificación.
1. Guarde y extraiga el archivo ZIP en un directorio local.

Si tiene un bot de C#, actualice el archivo `appsettings.json` para que incluya la información del archivo .bot, como se muestra a continuación:

```
{
  "botFilePath": "yourbasicBot.bot",
  "botFileSecret": "ukxxxxxxxxxxxs="
}
```
`botFilePath` hace referencia al nombre del bot, solo tiene que reemplazar "yourbasicBot.bot" por su propio nombre de bot. Para obtener la clave `botFileSecret`, consulte el artículo [Bot File Encryption](https://aka.ms/bot-file-encryption) (Cifrado de archivos .bot) sobre la generación de una clave para el bot.


Si tiene un bot de Node.js, agregue un archivo `.env` con las siguientes entradas:
```
botFilePath=yourbasicBot.bot
botFileSecret=ukxxxxxxxxxxxxs=
```

A continuación, realice cambios en los orígenes: puede editar los archivos de origen existentes o agregar otros nuevos al proyecto. Pruebe el código con el emulador. Cuando esté listo para volver a implementar el código modificado en Azure Portal, siga las instrucciones siguientes.

### <a name="publish-code-using-visual-studio"></a>Publicación del código con Visual Studio
1. En Visual Studio, haga clic con el botón derecho en el nombre del proyecto y haga clic en **Publicar**. Se abre la ventana **Publicar**.

![Publicación de Azure](~/media/azure-bot-build/azure-csharp-publish.png)

2. Seleccione el perfil del proyecto.
3. Copie la contraseña que se muestra en el archivo _publish.cmd_ del proyecto.
4. Haga clic en **Publicar**.
5. Cuando se le solicite, escriba la contraseña que copió en el paso 3.   

Después de configurar su proyecto, los cambios del proyecto se publicarán en Azure. 

A continuación, se examinará la descarga y reimplementación del código mediante la herramienta `az` cli.

### <a name="download-code-using-azure-cli"></a>Descarga del código mediante la CLI de Azure

En primer lugar, inicie sesión en Azure Portal mediante la herramienta az cli.

```azcli
az login
```

Se le pedirá un código de autenticación temporal único. Para iniciar sesión, use un explorador web y visite [Inicio de sesión del dispositivo](https://microsoft.com/devicelogin) de Microsoft, y pegue el código proporcionado por la CLI para continuar.

Para descargar el código mediante `az` cli, use el siguiente comando:
```azcli
az bot download --name "my-bot-name" --resource-group "my-resource-group"`
```
Después de descargar el código, realice lo siguiente:
- Si tiene un bot de C#, actualice el archivo appsettings.json para que incluya la información del archivo .bot, como se muestra a continuación:

```
{
  "botFilePath": "yourbasicBot.bot",
  "botFileSecret": "ukxxxxxxxxxxxs="
}
```

- Si tiene un bot de Node.js, agregue un archivo .env con las siguientes entradas:

```
botFilePath=yourbasicBot.bot
botFileSecret=ukxxxxxxxxxxxxs=
```

A continuación, realice cambios en los orígenes: puede editar los archivos de origen existentes o agregar otros nuevos al proyecto. Pruebe el código con el emulador. Cuando esté listo para volver a implementar el código modificado en Azure Portal, siga las instrucciones siguientes.

### <a name="login-to-azure-cli-by-running-the-following-command"></a>Inicie sesión en la CLI de Azure mediante la ejecución del comando siguiente.
Si ya ha iniciado sesión, puede omitir este paso.

```azcli
az login
```
Se le pedirá un código de autenticación temporal único. Para iniciar sesión, use un explorador web y visite [Inicio de sesión del dispositivo](https://microsoft.com/devicelogin) de Microsoft, y pegue el código proporcionado por la CLI para continuar.

### <a name="publish-code-using-azure-cli"></a>Publicación del código mediante la CLI de Azure
Para volver a publicar el código en Azure mediante `az` cli, use el siguiente comando:
```azcli
az bot publish --name "my-bot-name" --resource-group "my-resource-group" --code-dir <path to directory> 
```

Puede usar la opción `code-dir` para indicar qué directorio usar. Si no se proporciona, el comando `az bot publish` usará el directorio local para publicar.

## <a name="next-steps"></a>Pasos siguientes
Ahora que sabe cómo volver a cargar los cambios en Azure, puede configurar una implementación continua para el bot.

> [!div class="nextstepaction"]
> [Configuración de la implementación continua](bot-service-build-continuous-deployment.md)

---
title: Tutorial de creación e implementación de un bot básico | Microsoft Docs
description: Obtenga información sobre cómo crear un bot básico y, a continuación, implementarlo en Azure.
keywords: bot de eco, implementar, Azure, tutorial
author: Ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 1/9/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7927ab97dc88657a198c8f1d8e56bcb1ddf0fabe
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/07/2019
ms.locfileid: "57568242"
---
# <a name="tutorial-create-and-deploy-a-basic-bot"></a>Tutorial: Creación e implementación de un bot básico

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Este tutorial le guiará en la creación de un bot básico con Bot Framework SDK y su implementación en Azure. Si ya ha creado un bot básico que se ejecuta localmente, puede ir directamente a la sección [Implementación del bot](#deploy-your-bot).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un bot de eco básico
> * Ejecutarlo e interactuar con él localmente
> * Publicar el bot

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

[!INCLUDE [dotnet quickstart](~/includes/quickstart-dotnet.md)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [javascript quickstart](~/includes/quickstart-javascript.md)]

---

## <a name="deploy-your-bot"></a>Implementación del bot

Ahora, implemente el bot en Azure.

### <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]

### <a name="login-to-azure-cli-and-set-your-subscription"></a>Inicio de sesión en la CLI de Azure y establecimiento de la suscripción

Ya ha creado y probado localmente un bot y ahora quiere implementarlo en Azure.

[!INCLUDE [az login snippet](~/includes/deploy/snippet-az-login.md)]

### <a name="create-a-web-app-bot"></a>Creación de un bot de aplicación web

Si no dispone de un grupo de recursos para publicar el bot, cree uno:

[!INCLUDE [az create group snippet](~/includes/deploy/snippet-az-create-group.md)]

[!INCLUDE [az create web app snippet](~/includes/deploy/snippet-create-web-app.md)]

Antes de continuar, lea las instrucciones que se le aplican en función del tipo de cuenta de correo electrónico que usa para iniciar sesión en Azure.

#### <a name="msa-email-account"></a>Cuenta de correo electrónico MSA

Si va a usar una cuenta de correo electrónico [MSA](https://en.wikipedia.org/wiki/Microsoft_account), tendrá que crear el identificador de la aplicación y la contraseña en el portal de registro de aplicación que va a usar con el comando `az bot create`.

[!INCLUDE [create bot msa snippet](~/includes/deploy/snippet-create-bot-msa.md)]

#### <a name="business-or-school-account"></a>Cuenta empresarial o educativa

[!INCLUDE [create bot snippet](~/includes/deploy/snippet-create-bot.md)]

### <a name="download-the-bot-from-azure"></a>Descarga del bot de Azure

A continuación, descargue el bot que acaba de crear. 
[!INCLUDE [download bot snippet](~/includes/deploy/snippet-download-bot.md)]

[!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

### <a name="decrypt-the-downloaded-bot-file-and-use-in-your-project"></a>Descifrado del archivo .bot descargado y su uso en el proyecto

La información confidencial del archivo .bot está cifrada, por lo que vamos a descifrarla para poder usarla con facilidad. 

En primer lugar, vaya al directorio de bot que descargó.

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### <a name="test-your-bot-locally"></a>Prueba local del bot

En este punto, el bot debería funcionar del mismo modo que lo hacía con el archivo `.bot` antiguo. Asegúrese de que funciona según lo esperado con el nuevo archivo `.bot`.

Ahora debería ver un punto de conexión de *producción* en el emulador. De no ser así, probablemente todavía está usando el archivo `.bot` antiguo.

### <a name="publish-your-bot-to-azure"></a>Publicación del bot en Azure

<!-- TODO: re-encrypt your .bot file? -->

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

Ahora, debería poder probar el bot en el chat en web.

## <a name="additional-resources"></a>Recursos adicionales

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Adición de servicios al bot](bot-builder-tutorial-add-qna.md)


---
title: Implementación de un bot mediante la CLI de Azure | Microsoft Docs
description: Implemente su bot en la nube de Azure.
keywords: implementar bot, implementación de azure, publicar bot, bot de implementación de az, bot de implementación de visual studio, publicación de msbot, clonado de msbot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 01/07/2019
ms.openlocfilehash: 3ebc13cf9e2d111d716d081c36f125d28a441811
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360745"
---
# <a name="deploy-your-bot-using-azure-cli"></a>Implementación de un bot mediante la CLI de Azure

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

Después de que haya creado el bot y lo haya probado localmente, puede implementarlo en Azure para que sea posible acceder a él desde cualquier lugar. La implementación de un bot en Azure conllevará el pago de los servicios que se usen. El artículo acerca de la [facturación y administración de costos](https://docs.microsoft.com/en-us/azure/billing/) sirve de ayuda para conocer la facturación de Azure, para supervisar el uso y los costos, y para administrar cuentas y suscripciones.

En dicho artículo, podrá ver cómo implementar bots de C# y JavaScript en Azure mediante la cli de `az` y `msbot`. Es conveniente leerlo antes de seguir los pasos, con el fin de tener un conocimiento completo de lo que implica la implementación de un bot.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]


## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>Implementación de los bots de JavaScript y C# mediante az cli

Ya ha creado y probado localmente un bot y ahora quiere implementarlo en Azure. En estos pasos se supone que ha creado los recursos de Azure necesarios.

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

### <a name="decrypt-the-downloaded-bot-file-and-use-in-your-project"></a>Descifrado del archivo .bot descargado y su uso en el proyecto

La información confidencial del archivo .bot está cifrada.

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### <a name="update-the-bot-file"></a>Actualización del archivo .bot

Si el bot usa LUIS, QnA Maker o servicios de distribución, deberá agregar referencias a ellos en el archivo .bot. En caso contrario, puede omitir este paso.

1. Abra el bot en Bot Framework Emulator con el nuevo archivo .bot. No es necesario que el bot se esté ejecutando localmente.
1. En el panel **BOT EXPLORER** (Explorador de bots), expanda la sección **SERVICES** (Servicios).
1. Para agregar referencias a las aplicaciones de LUIS, haga clic en el signo más (+) situado a la derecha de **SERVICES** (Servicios).
   1. Seleccione **Add Language Understanding (LUIS)** (Agregar Language Understanding (LUIS).
   1. Si se le solicita que inicie sesión en la cuenta de Azure, hágalo.
   1. Presenta una lista de las aplicaciones de LUIS a las que tiene acceso. Seleccione las del bot.
1. Para agregar referencias a una base de conocimiento de QnA Maker, haga clic en el signo más (+) situado a la derecha de **SERVICES** (Servicios).
   1. Seleccione **Add QnA Maker** (Agregar QnA Maker).
   1. Si se le solicita que inicie sesión en la cuenta de Azure, hágalo.
   1. Presenta una lista de las bases de conocimiento a las que tiene acceso. Seleccione las del bot.
1. Para agregar referencias a los modelos de distribución, haga clic en el signo más (+) situado a la derecha de **SERVICES** (Servicios).
   1. Seleccione **Add Dispatch** (Agregar distribución).
   1. Si se le solicita que inicie sesión en la cuenta de Azure, hágalo.
   1. Presenta una lista de modelos de distribución a los que tiene acceso. Seleccione los del bot.

### <a name="test-your-bot-locally"></a>Prueba local del bot

En este punto, el bot debería funcionar del mismo modo que lo hacía con el archivo .bot antiguo. Asegúrese de que funciona según lo esperado con el nuevo archivo .bot.

### <a name="publish-your-bot-to-azure"></a>Publicación del bot en Azure

<!-- TODO: re-encrypt your .bot file? -->

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

## <a name="additional-resources"></a>Recursos adicionales

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Configuración de la implementación continua](bot-service-build-continuous-deployment.md)

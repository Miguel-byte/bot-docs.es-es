---
title: Tutorial de creación e implementación de un bot básico | Microsoft Docs
description: Obtenga información sobre cómo crear un bot básico y, a continuación, implementarlo en Azure.
keywords: bot de eco, implementar, Azure, tutorial
author: Ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e9a7158183171d45ba60cdb1507af9f717ecd51b
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386119"
---
# <a name="tutorial-create-and-deploy-a-basic-bot"></a>Tutorial: Creación e implementación de un bot básico

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

Este tutorial le guiará en la creación de un bot básico con el SDK de Bot Framework y su implementación en Azure. Si ya ha creado un bot básico que se ejecuta localmente, puede ir directamente a la sección [Implementación del bot](#deploy-your-bot).

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

### <a name="prerequisites"></a>Requisitos previos
[!INCLUDE [deploy prerequisite](~/includes/deploy/snippet-prerequisite.md)]

### <a name="prepare-for-deployment"></a>Preparación de la implementación
[!INCLUDE [deploy prepare intro](~/includes/deploy/snippet-prepare-deploy-intro.md)]

#### <a name="1-login-to-azure"></a>1. Inicio de sesión en Azure
[!INCLUDE [deploy az login](~/includes/deploy/snippet-az-login.md)]

#### <a name="2-set-the-subscription"></a>2. Establecimiento de la suscripción
[!INCLUDE [deploy az subscription](~/includes/deploy/snippet-az-set-subscription.md)]

#### <a name="3-create-an-app-registration"></a>3. Creación de un registro de aplicación
[!INCLUDE [deploy create app registration](~/includes/deploy/snippet-create-app-registration.md)]

#### <a name="4-deploy-via-arm-template"></a>4. Implementación de una plantilla de Resource Manager
Puede implementar el bot en un grupo de recursos nuevo o en uno ya existente. Elija la opción que mejor funcione en su caso. 
##### <a name="deploy-via-arm-template-with-new-resource-group"></a>**Implementación mediante la plantilla de ARM (con un grupo de recursos nuevo)**
[!INCLUDE [ARM with new resourece group](~/includes/deploy/snippet-ARM-new-resource-group.md)]

##### <a name="deploy-via-arm-template-with-existing-resource-group"></a>**Implementación mediante la plantilla de ARM (con un grupo de recursos ya existente)**
[!INCLUDE [ARM with existing resourece group](~/includes/deploy/snippet-ARM-existing-resource-group.md)]

#### <a name="5-prepare-your-code-for-deployment"></a>5. Preparación del código para la implementación
##### <a name="retrieve-or-create-necessary-iiskudu-files"></a>**Recuperación o creación de los archivos IIS/Kudu necesarios**
[!INCLUDE [retrieve or create IIS/Kudu files](~/includes/deploy/snippet-IIS-Kudu-files.md)]

##### <a name="zip-up-the-code-directory-manually"></a>**Compresión del directorio de código manualmente**
[!INCLUDE [zip up code](~/includes/deploy/snippet-zip-code.md)]

### <a name="deploy-code-to-azure"></a>Implementación del código en Azure
[!INCLUDE [deploy code to Azure](~/includes/deploy/snippet-deploy-code-to-az.md)]

### <a name="test-in-web-chat"></a>Probar en Chat en web
[!INCLUDE [test in web chat](~/includes/deploy/snippet-test-in-web-chat.md)]

## <a name="additional-resources"></a>Recursos adicionales

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Uso de QnA Maker en el bot para responder preguntas](bot-builder-tutorial-add-qna.md)

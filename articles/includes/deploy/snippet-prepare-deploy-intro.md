---
ms.openlocfilehash: db7c4b447c5a27fe2047cfa41e65fac0d3e3a86c
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/05/2019
ms.locfileid: "70385987"
---
Cuando crea un bot mediante la [plantilla de Visual Studio](https://docs.microsoft.com/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0) o de [Yeoman](https://docs.microsoft.com/azure/bot-service/javascript/bot-builder-javascript-quickstart?view=azure-bot-service-4.0), el código fuente que se genera incluye una carpeta `deploymentTemplates` con plantillas de Resource Manager. El proceso de implementación que se documenta aquí utiliza las plantillas de Resource Manager para aprovisionar los recursos necesarios para el bot en Azure mediante la CLI de Azure. 

> [!NOTE]
> Con la versión SDK Bot Framework 4.3, hemos dejado _en desuso_ un archivo .bot. En su lugar, usamos un archivo appsettings.json o .env para administrar los recursos del bot. Para más información sobre cómo migrar la configuración del archivo .bot al archivo appsettings.json o .env, consulte [Administración de recursos del bot](https://docs.microsoft.com/azure/bot-service/bot-file-basics?view=azure-bot-service-4.0).

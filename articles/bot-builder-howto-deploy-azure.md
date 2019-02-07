---
title: Implementación de un bot de C# mediante Visual Studio | Microsoft Docs
description: Implemente su bot en la nube de Azure.
keywords: implementar bot, implementación de azure, publicar bot, bot de implementación de az, bot de implementación de visual studio, publicación de msbot, clonado de msbot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/08/2018
ms.openlocfilehash: eb559418bc2925ec6fb64902086dede50e485414
ms.sourcegitcommit: 32615b88e4758004c8c99e9d564658a700c7d61f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/04/2019
ms.locfileid: "55712019"
---
# <a name="deploy-your-c-bot-using-visual-studio"></a>Implementación de un bot de C# mediante Visual Studio

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

Después de que haya creado el bot y lo haya probado localmente, puede implementarlo en Azure para que sea posible acceder a él desde cualquier lugar. La implementación de un bot en Azure conllevará el pago de los servicios que se usen. El artículo acerca de la [facturación y administración de costos](https://docs.microsoft.com/en-us/azure/billing/) sirve de ayuda para conocer la facturación de Azure, para supervisar el uso y los costos, y para administrar cuentas y suscripciones.

En este artículo, le mostraremos cómo implementar un bot de C# mediante Visual Studio y Azure Portal. Es conveniente leerlo antes de seguir los pasos, con el fin de tener un conocimiento completo de lo que implica la implementación de un bot.

## <a name="prerequisites"></a>Requisitos previos
- Instale [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started).
- Conocimientos del archivo [.bot](v4sdk/bot-file-basics.md).

## <a name="deploy-your-bot-in-app-service"></a>Implementación del bot en App Service
En primer lugar, va a implementar el bot en Azure desde Visual Studio en App Service. A continuación, configurará el bot con Azure Bot Service mediante el registro de canales de bots.

**Nota: Si el nombre del proyecto de Visual Studio tiene espacios, los pasos de implementación que se describen a continuación no funcionarán.**

En la ventana Explorador de soluciones, haga clic con el botón derecho en el nodo del proyecto y seleccione Publicar.

![configuración de publicación](media/azure-bot-quickstarts/getting-started-publish-setting.png)

2. En el cuadro diálogo para la selección de un destino de publicación, asegúrese de que **App Service** esté seleccionado a la izquierda y que **Crear nuevo** esté seleccionado a la derecha.

3. Haga clic en el botón Publicar.

4. En la esquina superior derecha del cuadro de diálogo, asegúrese de que se muestra el identificador de usuario correcto para su suscripción de Azure.

![publicación principal](media/azure-bot-quickstarts/getting-started-publish-main.png)

5. Escriba la información de nombre de la aplicación, suscripción, grupo de recursos y plan de hospedaje.

6. Cuando esté listo, haga clic en Crear. Este proceso puede tardar unos minutos en completarse.

7. Una vez finalizado, se abrirá un explorador web que muestra la dirección URL pública del bot.

8. Realice una copia de esta dirección (será algo como https://<yourbotname>.azurewebsites.net/).

> [!NOTE] 
> Deberá usar la versión HTTPS de la dirección URL al registrar el bot. Azure proporciona compatibilidad con SSL en Azure App Service.

## <a name="create-your-bot-channels-registration"></a>Creación del registro de canales de bot
Con el bot implementado en Azure, deberá registrarlo en Azure Bot Service.

1. Acceda a Azure Portal en https://portal.azure.com.

2. Inicie sesión con la misma identidad que usó anteriormente desde Visual Studio para publicar su bot.

3. Haga clic en Crear un recurso.

4. En el campo Search the Marketplace (Buscar en Marketplace), escriba Bot Channels Registration y presione Entrar.

5. En la lista devuelta, elija Bot Channels Registration:

![Publicar](media/azure-bot-quickstarts/getting-started-bot-registration.png)

6. Haga clic en Crear en la hoja que se abre.

7. Proporcione un nombre para el bot.

8. Elija la misma suscripción donde implementó el código del bot.

9. Seleccione el grupo de recursos existente que establecerá la ubicación.

10. Puede elegir el plan de tarifa F0 para desarrollo y pruebas.

11. Escriba la dirección URL de su bot. Asegúrese de que comienza por HTTPS y de que agrega /api/messages, por ejemplo https://yourbotname.azurewebsites.net/api/messages.

12. Desactive por ahora Application Insights.

13. Haga clic en el identificador y la contraseña de aplicación de Microsoft.

14. En la hoja Nuevo, haga clic en Crear nuevo.

15. En la nueva hoja que aparece a la derecha, haga clic "Create App ID in the App Registration Portal" (Crear el identificador de aplicación en el portal de registro de aplicaciones), que se abre en una nueva pestaña del explorador.

![msa de bot](media/azure-bot-quickstarts/getting-started-msa.png)

16. En la nueva pestaña, realice una copia del identificador de aplicación y guárdela en alguna parte. 

17. Haga clic en el botón Generate an app password to continue (Generar una contraseña de aplicación para continuar).

18. Se abre un cuadro de diálogo de explorador que le proporciona la contraseña de la aplicación. Esta será la única vez que la reciba. Copie y guarde esta contraseña en algún lugar donde pueda recuperarla más adelante.

19. Cuando esté satisfecho con la contraseña guardada, haga clic en Aceptar.

20. Simplemente cierre la pestaña del explorador y vuelva a la pestaña de Azure Portal.

21. Pegue el identificador y la contraseña de aplicación en los campos correctos y haga clic en Aceptar.

22. Ahora haga clic en Crear para configurar el registro de canales. Este proceso puede tardar algunos segundos o minutos en completarse.

## <a name="update-your-bots-application-settings"></a>Actualización de la configuración de la aplicación del bot
Para que el bot se autentique en Azure Bot Service, debe agregar dos valores de configuración a la configuración de la aplicación del bot en Azure App Service. 

1. Haga clic en App Services. Escriba el nombre del bot en el cuadro de texto de suscripciones. A continuación, haga clic en el nombre del bot en la lista.

![app service](media/azure-bot-quickstarts/getting-started-app-service.png)

2. En la lista de opciones a la izquierda de las opciones del bot, busque Application Settings (Configuración de la aplicación) en la sección Settings (Configuración) y haga clic en ella.

![identificador del bot](media/azure-bot-quickstarts/getting-started-app-settings-1.png)

3. Desplácese hasta que encuentre la sección Application settings (Configuración de la aplicación).

![msa de bot](media/azure-bot-quickstarts/getting-started-app-settings-2.png)

4. Haga clic en Add new setting (Agregar nueva configuración).

5. Escriba **MicrosoftAppId** como nombre y el identificador de la aplicación como valor.

6. Haga clic en Add new setting (Agregar nueva configuración).

7. Escriba **MicrosoftAppPassword** como nombre y su contraseña como valor.

8. Haga clic en el botón Save (Guardar) de la parte superior.

## <a name="test-your-bot-in-production"></a>Prueba del bot en producción
En este momento, puede probar el bot desde Azure mediante el cliente de Chat en web integrado.

1. Vuelva al grupo de recursos en Azure Portal

2. Abra el bot.

3. En **Administración de bots**, seleccione **Probar en Chat en web**.

![probar en Chat en web](media/azure-bot-quickstarts/getting-started-test-webchat.png)

4. Escriba un mensaje como `Hi` y presione ENTRAR. El bot devolverá `Turn 1: You sent Hi`.

---

## <a name="additional-resources"></a>Recursos adicionales

Cuando se implementa un bot, estos recursos normalmente se crean en Azure Portal:

| Recursos      | DESCRIPCIÓN |
|----------------|-------------|
| Bot de aplicación web | Bot de Azure Bot Service que se implementa en Azure App Service.|
| [App Service](https://docs.microsoft.com/en-us/azure/app-service/)| Permite crear y hospedar aplicaciones web.|
| [plan de App Service](https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)| Define un conjunto de recursos de proceso para que una aplicación web se ejecute.|
| [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview)| Proporciona las herramientas necesarias para recopilar y analizar datos de telemetría.|
| [Cuenta de almacenamiento](https://docs.microsoft.com/en-us/azure/storage/common/storage-introduction)| Proporciona almacenamiento en la nube altamente disponible, seguro, duradero, escalable y redundante.|

Si no está familiarizado con el grupo de recursos de Azure, consulte este tema de [terminología](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#terminology).

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Configuración de la implementación continua](bot-service-build-continuous-deployment.md)

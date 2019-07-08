---
title: Creación de un bot con Bot Service | Microsoft Docs
description: Obtenga información sobre cómo crear un bot con Bot Service, un entorno de desarrollo de bots integrado y dedicado.
keywords: Inicio rápido, crear bot, servicio de bots, bot de aplicación web
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 05/31/2019
ms.openlocfilehash: 9ef15b09c8517db9e8b1f13f72172f09f9fb23eb
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464617"
---
# <a name="create-a-bot-with-azure-bot-service"></a>Creación de un bot con Azure Bot Service

::: moniker range="azure-bot-service-3.0"

> [!NOTE]
> **A partir del 10 de junio de 2019, no se podrán crear bots con el SDK v3 en Azure Portal. Se recomienda a los clientes que creen sus bots con el [SDK v4](https://docs.microsoft.com/azure/bot-service/bot-service-quickstart?view=azure-bot-service-4.0) a partir de ahora. Encontrará más información sobre la compatibilidad a largo plazo del SDK V3 [aquí](https://docs.microsoft.com/azure/bot-service/bot-service-resources-bot-framework-faq?view=azure-bot-service-3.0#bot-framework-sdk-version-3-lifetime-support)** . 

Bot Service ofrece los componentes principales para la creación de bots, incluido Bot Framework SDK para desarrollar bots y Bot Framework para conectar bots a canales. Bot Service ofrece cinco plantillas entre las que puede elegir al crear sus bots con compatibilidad con .NET y Node.js. En este tema, aprenda a usar Bot Service para crear un nuevo bot que usa Bot Framework SDK.

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure
Inicie sesión en [Azure Portal](http://portal.azure.com).

## <a name="create-a-new-bot-service"></a>Creación de un nuevo servicio de bots

1. Haga clic en el vínculo **Crear un nuevo recurso** que se encuentra en la esquina superior izquierda de Azure Portal y, a continuación, seleccione **IA + Machine Learning > Web App Bot** (Bot de aplicación web). 

2. Se abrirá una nueva hoja con información sobre **Web App Bot**.  

3. En la hoja **Bot Service**, indique la información solicitada sobre el bot según se especifica en la tabla debajo de la imagen.  <br/>
   ![Creación de la hoja Web App Bot](./media/azure-bot-quickstarts/sdk-create-bot-service-blade.png) (bot de aplicación web)

   | Configuración | Valor sugerido | DESCRIPCIÓN |
   | ---- | ---- | ---- |
   | **Nombre del bot** | Nombre para mostrar del bot | Nombre para mostrar del bot que aparece en los canales y directorios. Este nombre se puede cambiar en cualquier momento. |
   | **Suscripción** | Su suscripción | Seleccione la suscripción de Azure que quiere usar. |
   | **Grupo de recursos** | myResourceGroup | Puede crear un [grupo de recursos](/azure/azure-resource-manager/resource-group-overview#resource-groups) o elegir uno existente. |
   | **Ubicación** | Ubicación predeterminada | Seleccione la ubicación geográfica del grupo de recursos. Puede elegir cualquier ubicación de la lista, aunque a menudo es mejor elegir la más cercana al cliente. No se puede cambiar la ubicación una vez creado el bot. |
   | **Plan de tarifa** | F0 | Seleccione un plan de tarifa. Puede actualizar el plan de tarifa en cualquier momento. Para más información, consulte [Precios de Azure Bot Service](https://azure.microsoft.com/pricing/details/bot-service/). |
   | **Nombre de la aplicación** | Un nombre único | Nombre único de la dirección URL del bot. Por ejemplo, si el nombre del bot es *myawesomebot*, la dirección URL del bot será `http://myawesomebot.azurewebsites.net`. El nombre solo debe usar caracteres alfanuméricos y de subrayado. Hay un límite de 35 caracteres para este campo. No se puede cambiar el nombre de la aplicación una vez creado el bot. |
   | **Plantilla de bot** | Básica | Elija **C#** o **Node.js** y seleccione la plantilla **básica** para este inicio rápido y, a continuación, haga clic en **Seleccionar**. La plantilla básica crea un bot de eco. [Obtenga más información](bot-service-concept-templates.md) sobre las plantillas. |
   | **Plan de App Service/Ubicación** | Su plan de App Service  | Seleccione una ubicación para el [plan de App Service](https://azure.microsoft.com/pricing/details/app-service/plans/). Puede elegir cualquier ubicación de la lista, aunque a menudo es mejor elegir la más cercana al cliente. (No disponible para el bot de Functions). |
   | **Application Insights** | Por | Decida si quiere **Activar** o **Desactivar** [Application Insights](/bot-framework/bot-service-manage-analytics). Si selecciona **Activar**, también debe especificar una ubicación regional. Su elección de ubicación puede ser cualquier ubicación de la lista, aunque con frecuencia es mejor elegir la misma ubicación que la del servicio de bot. |
   | **Id. y contraseña de aplicación de Microsoft** | Creación automática del id. y contraseña de la aplicación | Use esta opción si tiene que escribir manualmente un id. y contraseña de aplicación de Microsoft. En caso contrario, se crearán automáticamente un id. y contraseña nuevos de aplicación de Microsoft en el proceso de creación del bot. Al crear un registro de aplicación manualmente para Bot Service, asegúrese de que los tipos de cuenta compatibles se han establecido en "Cuentas en cualquier directorio organizativo" o "Cuentas en cualquier directorio organizativo y cuentas Microsoft personales (por ejemplo, Skype, Xbox, Outlook.com)".  |

   > [!NOTE]
   > 
   > Si está creando un **bot de Functions**, no verá ningún campo de **Plan de App Service/Ubicación**. En su lugar, verá el campo *Plan de hospedaje*. En ese caso, elija un [Plan de hospedaje](bot-service-overview-readme.md#hosting-plans).

4. Haga clic en **Crear** para crear el servicio e implementar el bot en la nube. Este proceso puede tardar varios minutos.

Para confirmar que el bot se ha implementado, active las **Notificaciones**. Las notificaciones cambiarán de **Implementación en curso...** a **Implementación correcta**. Haga clic en el botón **Ir al recurso** para abrir la hoja de recursos del bot.

 > [!TIP] 
 > Por motivos de rendimiento, los **bot de Functions** que ejecutan plantillas de Node.js ya se han empaquetado con la herramienta *Azure Functions Pack*. La herramienta *Azure Functions Pack* toma todos los módulos de Node.js y los combina en un archivo *.js.
 > Para más información, consulte el artículo [Azure Functions Pack](https://github.com/Azure/azure-functions-pack) (Paquete de Azure Functions).
 
## <a name="test-the-bot"></a>Probar el bot
Una vez ha creado el bot, pruébelo en [Chat en web](bot-service-manage-test-webchat.md). Escriba un mensaje y el bot debería responder.

## <a name="next-steps"></a>Pasos siguientes

En este tema, ha aprendido a crear un bot de Functions o un bot de aplicación web **básico** mediante el uso de Bot Service y ha comprobado la funcionalidad del bot con el control Chat en web de Azure. Ahora, obtenga información sobre cómo administrar el bot y empezar a trabajar con su código fuente.

> [!div class="nextstepaction"]
> [Administración de un bot](bot-service-manage-overview.md)

::: moniker-end

::: moniker range="azure-bot-service-4.0"

[!INCLUDE [applies-to-v4](includes/applies-to.md)]

Azure Bot Service proporciona los componentes principales para crear bots, incluido Bot Framework SDK para desarrollar bots y Bot Service para conectar los bots con los canales. En el tema, podrá elegir entre plantilla .NET o Node.js para crear un bot mediante Bot Framework SDK v4.

[!INCLUDE [Azure vs local development](~/includes/snippet-quickstart-paths.md)]

## <a name="prerequisites"></a>Requisitos previos

- Cuenta de [Azure](http://portal.azure.com)

### <a name="create-a-new-bot-service"></a>Creación de un nuevo servicio de bots

1. Inicie sesión en [Azure Portal](http://portal.azure.com/).
1. Haga clic en el vínculo **Crear un recurso** que se encuentra en la esquina superior izquierda de Azure Portal y, a continuación, seleccione **AI + Machine Learning**>**Web App Bot**. 

![crear bot](~/media/azure-bot-quickstarts/abs-create-blade.png)

2. Se abrirá una *nueva hoja* con información sobre **Web App Bot**.  

3. En la hoja **Bot Service**, indique la información solicitada sobre el bot según se especifica en la tabla debajo de la imagen.  <br/>
 ![Creación de la hoja Web App Bot](~/media/azure-bot-quickstarts/sdk-create-bot-service-blade.png) (bot de aplicación web)

 | Configuración | Valor sugerido | DESCRIPCIÓN |
 | ---- | ---- | ---- |
 | **Nombre del bot** | Nombre para mostrar del bot | Nombre para mostrar del bot que aparece en los canales y directorios. Este nombre se puede cambiar en cualquier momento. |
 | **Suscripción** | Su suscripción | Seleccione la suscripción de Azure que quiere usar. |
 | **Grupo de recursos** | myResourceGroup | Puede crear un [grupo de recursos](/azure/azure-resource-manager/resource-group-overview#resource-groups) o elegir uno existente. |
 | **Ubicación** | Ubicación predeterminada | Seleccione la ubicación geográfica del grupo de recursos. Puede elegir cualquier ubicación de la lista, aunque a menudo es mejor elegir la más cercana al cliente. No se puede cambiar la ubicación una vez creado el bot. |
 | **Plan de tarifa** | F0 | Seleccione un plan de tarifa. Puede actualizar el plan de tarifa en cualquier momento. Para más información, consulte [Precios de Azure Bot Service](https://azure.microsoft.com/pricing/details/bot-service/). |
 | **Nombre de la aplicación** | Un nombre único | Nombre único de la dirección URL del bot. Por ejemplo, si el nombre del bot es *myawesomebot*, la dirección URL del bot será `http://myawesomebot.azurewebsites.net`. El nombre solo debe usar caracteres alfanuméricos y de subrayado. Hay un límite de 35 caracteres para este campo. No se puede cambiar el nombre de la aplicación una vez creado el bot. |
 | **Plantilla de bot** | Bot de eco | Elija **SDK v4**. Seleccione C# o Node.js para esta guía de inicio rápido y, a continuación, haga clic en **Seleccionar**.  
 | **Plan de App Service/Ubicación** | Su plan de App Service  | Seleccione una ubicación para el [plan de App Service](https://azure.microsoft.com/pricing/details/app-service/plans/). Su elección de ubicación puede ser cualquier ubicación de la lista, aunque con frecuencia es mejor elegir la misma ubicación que la del servicio de bot. |
 | **Application Insights** | Por | Decida si quiere **Activar** o **Desactivar** [Application Insights](/bot-framework/bot-service-manage-analytics). Si selecciona **Activar**, también debe especificar una ubicación regional. Su elección de ubicación puede ser cualquier ubicación de la lista, aunque con frecuencia es mejor elegir la misma ubicación que la del servicio de bot. |
 | **Id. y contraseña de aplicación de Microsoft** | Creación automática del id. y contraseña de la aplicación | Use esta opción si tiene que escribir manualmente un id. y contraseña de aplicación de Microsoft. En caso contrario, se crearán automáticamente un id. y contraseña nuevos de aplicación de Microsoft en el proceso de creación del bot. Al crear un registro de aplicación manualmente para Bot Service, asegúrese de que los tipos de cuenta compatibles se han establecido en "Cuentas en cualquier directorio organizativo" o "Cuentas en cualquier directorio organizativo y cuentas Microsoft personales (por ejemplo, Skype, Xbox, Outlook.com)". |

4. Haga clic en **Crear** para crear el servicio e implementar el bot en la nube. Este proceso puede tardar varios minutos.

Para confirmar que el bot se ha implementado, active las **Notificaciones**. Las notificaciones cambiarán de **Implementación en curso...** a **Implementación correcta**. Haga clic en el botón **Ir al recurso** para abrir la hoja de recursos del bot.

Una vez ha creado el bot, pruébelo en Chat en web. 

## <a name="test-the-bot"></a>Probar el bot
En la sección **Administración del bot**, haga clic en **Probar en Chat en web**. Azure Bot Service cargará el control Chat en web y se conectará al bot. 

![Prueba del Chat en web de Azure](./media/azure-bot-quickstarts/azure-webchat-test.png)

Escriba un mensaje y el bot debería responder.

## <a name="download-code"></a>Descarga de código
El código se puede descargar para trabajar en el localmente. 
1. En la sección **Bot Management** (Administración de bots), haga clic en **Build** (Compilar). 
1. Haga clic en el vínculo **Download Bot source code** (Descargar código fuente del bot) del panel derecho. 
1. Siga las indicaciones para descargar el código y, después, descomprima la carpeta.
    1. [!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

## <a name="next-steps"></a>Pasos siguientes
Después de descargar el código, puede continuar desarrollando el bot localmente en el equipo. Una vez que pruebe su bot y esté listo para cargar el código del bot en Azure Portal, siga las instrucciones del tema [Configuración de la implementación continua](./bot-service-build-continuous-deployment.md) para actualizar automáticamente el código después de realizar cambios.

::: moniker-end

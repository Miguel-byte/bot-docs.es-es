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
ms.date: 09/18/2018
ms.openlocfilehash: b5b02773ab71801132f2a73f81123588e7ddfcdb
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645695"
---
::: moniker range="azure-bot-service-3.0"


# <a name="create-a-bot-with-azure-bot-service"></a>Creación de un bot con Azure Bot Service

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Bot Service ofrece los componentes principales para la creación de bots, incluido Bot Builder SDK para desarrollar bots y Bot Framework para conectar bots a canales. Bot Service ofrece cinco plantillas entre las que puede elegir al crear sus bots con compatibilidad con .NET y Node.js. En este tema, aprenda a usar Bot Service para crear un nuevo bot que usa el SDK de Bot Builder.

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure
Inicie sesión en [Azure Portal](http://portal.azure.com).

> [!TIP]
> Si aún no tiene una suscripción, puede registrarse para obtener una <a href="https://azure.microsoft.com/en-us/free/" target="_blank">cuenta gratuita</a>.

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
   | **Plan de tarifa** | F0 | Seleccione un plan de tarifa. Puede actualizar el plan de tarifa en cualquier momento. Para más información, consulte [Precios de Azure Bot Service](https://azure.microsoft.com/en-us/pricing/details/bot-service/). |
   | **Nombre de la aplicación** | Un nombre único | Nombre único de la dirección URL del bot. Por ejemplo, si el nombre del bot es *myawesomebot*, la dirección URL del bot será `http://myawesomebot.azurewebsites.net`. El nombre solo debe usar caracteres alfanuméricos y de subrayado. Hay un límite de 35 caracteres para este campo. No se puede cambiar el nombre de la aplicación una vez creado el bot. |
   | **Plantilla de bot** | Básica | Elija **C#** o **Node.js** y seleccione la plantilla **básica** para este inicio rápido y, a continuación, haga clic en **Seleccionar**. La plantilla básica crea un bot de eco. [Obtenga más información](bot-service-concept-templates.md) sobre las plantillas. |
   | **Plan de App Service/Ubicación** | Su plan de App Service  | Seleccione una ubicación para el [plan de App Service](https://azure.microsoft.com/en-us/pricing/details/app-service/plans/). Puede elegir cualquier ubicación de la lista, aunque a menudo es mejor elegir la más cercana al cliente. (No disponible para el bot de Functions). |
   | **Azure Storage** | Su cuenta de almacenamiento de Azure | Puede crear una cuenta de almacenamiento de datos o usar una existente. De forma predeterminada, el bot usará [Table Storage](/azure/storage/common/storage-introduction#table-storage). |
   | **Application Insights** | Por | Decida si quiere **Activar** o **Desactivar** [Application Insights](/bot-framework/bot-service-manage-analytics). Si selecciona **Activar**, también debe especificar una ubicación regional. Su elección de ubicación puede ser cualquier ubicación de la lista, aunque con frecuencia es mejor elegir la misma ubicación que la del servicio de bot. |
   | **Id. y contraseña de aplicación de Microsoft** | Creación automática del id. y contraseña de la aplicación | Use esta opción si tiene que escribir manualmente un id. y contraseña de aplicación de Microsoft. En caso contrario, se crearán automáticamente un id. y contraseña nuevos de aplicación de Microsoft en el proceso de creación del bot. |

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

# <a name="create-a-bot-with-azure-bot-service"></a>Creación de un bot con Azure Bot Service
[!INCLUDE [pre-release-label](includes/pre-release-label.md)]

Azure Bot Service ofrece los componentes principales para la creación de bots, incluido el SDK de Bot Builder para desarrollar bots y el servicio de bots para conectar los bots con canales. En el tema, podrá elegir una plantilla .NET o Node.js para crear un bot con el SDK Bot Builder v4.

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure
Inicie sesión en [Azure Portal](http://portal.azure.com).

> [!TIP]
> Si aún no tiene una suscripción, puede registrarse para obtener una <a href="https://azure.microsoft.com/en-us/free/" target="_blank">cuenta gratuita</a>.

## <a name="create-a-new-bot-service"></a>Creación de un nuevo servicio de bots

1. Haga clic en el vínculo **Crear un nuevo recurso** que se encuentra en la esquina superior izquierda de Azure Portal y, a continuación, seleccione **IA + Machine Learning > Web App Bot** (Bot de aplicación web). 

2. Se abrirá una nueva hoja con información sobre **Web App Bot**.  

3. En la hoja **Bot Service**, indique la información solicitada sobre el bot según se especifica en la tabla debajo de la imagen.  <br/>
 ![Creación de la hoja Web App Bot](~/media/azure-bot-quickstarts/sdk-create-bot-service-blade.png) (bot de aplicación web)

 | Configuración | Valor sugerido | DESCRIPCIÓN |
 | ---- | ---- | ---- |
 | **Nombre del bot** | Nombre para mostrar del bot | Nombre para mostrar del bot que aparece en los canales y directorios. Este nombre se puede cambiar en cualquier momento. |
 | **Suscripción** | Su suscripción | Seleccione la suscripción de Azure que quiere usar. |
 | **Grupo de recursos** | myResourceGroup | Puede crear un [grupo de recursos](/azure/azure-resource-manager/resource-group-overview#resource-groups) o elegir uno existente. |
 | **Ubicación** | Ubicación predeterminada | Seleccione la ubicación geográfica del grupo de recursos. Puede elegir cualquier ubicación de la lista, aunque a menudo es mejor elegir la más cercana al cliente. No se puede cambiar la ubicación una vez creado el bot. |
 | **Plan de tarifa** | F0 | Seleccione un plan de tarifa. Puede actualizar el plan de tarifa en cualquier momento. Para más información, consulte [Precios de Azure Bot Service](https://azure.microsoft.com/en-us/pricing/details/bot-service/). |
 | **Nombre de la aplicación** | Un nombre único | Nombre único de la dirección URL del bot. Por ejemplo, si el nombre del bot es *myawesomebot*, la dirección URL del bot será `http://myawesomebot.azurewebsites.net`. El nombre solo debe usar caracteres alfanuméricos y de subrayado. Hay un límite de 35 caracteres para este campo. No se puede cambiar el nombre de la aplicación una vez creado el bot. |
 | **Plantilla de bot** | Bot de eco | Elija **SDK v4**. Seleccione C# o Node.js para esta guía de inicio rápido y, a continuación, haga clic en **Seleccionar**.  
 | **Plan de App Service/Ubicación** | Su plan de App Service  | Seleccione una ubicación para el [plan de App Service](https://azure.microsoft.com/en-us/pricing/details/app-service/plans/). Su elección de ubicación puede ser cualquier ubicación de la lista, aunque con frecuencia es mejor elegir la misma ubicación que la del servicio de bot. |
 | **Azure Storage** | Su cuenta de almacenamiento de Azure | Puede crear una cuenta de almacenamiento de datos o usar una existente. De forma predeterminada, el bot usará [Table Storage](/azure/storage/common/storage-introduction#table-storage). |
 | **Application Insights** | Por | Decida si quiere **Activar** o **Desactivar** [Application Insights](/bot-framework/bot-service-manage-analytics). Si selecciona **Activar**, también debe especificar una ubicación regional. Su elección de ubicación puede ser cualquier ubicación de la lista, aunque con frecuencia es mejor elegir la misma ubicación que la del servicio de bot. |
 | **Id. y contraseña de aplicación de Microsoft** | Creación automática del id. y contraseña de la aplicación | Use esta opción si tiene que escribir manualmente un id. y contraseña de aplicación de Microsoft. En caso contrario, se crearán automáticamente un id. y contraseña nuevos de aplicación de Microsoft en el proceso de creación del bot. |

4. Haga clic en **Crear** para crear el servicio e implementar el bot en la nube. Este proceso puede tardar varios minutos.

Para confirmar que el bot se ha implementado, active las **Notificaciones**. Las notificaciones cambiarán de **Implementación en curso...** a **Implementación correcta**. Haga clic en el botón **Ir al recurso** para abrir la hoja de recursos del bot.

Una vez ha creado el bot, pruébelo en Chat en web. 

## <a name="test-the-bot"></a>Probar el bot
En la sección **Administración del bot**, haga clic en **Probar en Chat en web**. Azure Bot Service cargará el control Chat en web y se conectará al bot. 

![Prueba del Chat en web de Azure](./media/azure-bot-quickstarts/azure-webchat-test.png)

Escriba un mensaje y el bot debería responder.

## <a name="next-steps"></a>Pasos siguientes

En este tema, ha aprendido a crear una aplicación web de bot de **eco** mediante el uso de Azure Bot Service y ha comprobado la funcionalidad del bot con el control Chat en web integrado. Ahora, obtenga información sobre cómo administrar el bot y empezar a trabajar con su código fuente.

> [!div class="nextstepaction"]
> [Administración de un bot](bot-service-manage-overview.md)

::: moniker-end

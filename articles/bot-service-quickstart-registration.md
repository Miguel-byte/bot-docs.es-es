---
title: Creación de Bot Channels Registration con Bot Service | Microsoft Docs
description: Obtenga información sobre cómo registrar un bot existente en Bot Service.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0d13a27ef81660c6702f5a09487b731bc891c750
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405804"
---
# <a name="register-a-bot-with-bot-service"></a>Registro de un bot en Bot Service



Si ya tiene un bot hospedado en otra parte y le gustaría usar Bot Service para conectarlo a otros canales, deberá registrar su bot en el Servicio de bots. En este tema, aprenderá a registrar el bot en Bot Service mediante la creación de un servicio de bots **Bot Channels Registration**.

> [!IMPORTANT] 
> Solo deberá registrar su bot si no está hospedado en Azure. Si [creó un bot](bot-service-quickstart.md) a través de Azure Portal, el bot ya está registrado en Bot Service.

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure
Inicie sesión en [Azure Portal](http://portal.azure.com).

> [!TIP]
> Si aún no tiene una suscripción, puede registrarse para obtener una <a href="https://azure.microsoft.com/free/" target="_blank">cuenta gratuita</a>.

## <a name="create-a-bot-channels-registration"></a>Creación de un Bot Channels Registration
Necesita un servicio de bots **Bot Channels Registration** para poder usar la funcionalidad de Bot Service. Un bot de registro le permite conectar el bot a los canales.

Para crear un **Bot Channels Registration**, haga lo siguiente:

1. Haga clic en el botón **Nuevo** de la esquina superior izquierda de Azure Portal y seleccione **IA y Cognitive Services > Bot Channels Registration** (Registro de canales de bot). 

2. Se abrirá una nueva hoja con información sobre ell **Bot Channels Registration**. Haga clic en el botón **Crear** para iniciar el proceso de creación. 

3. En la hoja **Servicio de bots**, indique la información solicitada sobre el bot tal como se especifica en la tabla debajo de la imagen.  <br/>
   ![Hoja de creación de registro de bot](~/media/azure-bot-quickstarts/registration-create-bot-service-blade.png)


   |                    Configuración                     |         Valor sugerido         |                                                                                                  DESCRIPCIÓN                                                                                                  |
   |------------------------------------------------|---------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
   |           <strong>Nombre del bot</strong>            |     Nombre para mostrar del bot     |                                                  Nombre para mostrar del bot que aparece en los canales y directorios. Este nombre se puede cambiar en cualquier momento.                                                  |
   |         <strong>Suscripción</strong>          |        Su suscripción        |                                                                                Seleccione la suscripción de Azure que quiere usar.                                                                                 |
   |        <strong>Grupo de recursos</strong>         |         myResourceGroup         |                                 Puede crear un [grupo de recursos](/azure/azure-resource-manager/resource-group-overview#resource-groups) o elegir uno existente.                                  |
   |                    Location                    |             Oeste de EE. UU.             |                                                        Elija una ubicación cerca de donde se implementa el bot o cerca de otros servicios a los que tendrá acceso su bot.                                                         |
   |         <strong>Plan de tarifa</strong>          |               F0                |             Seleccione un plan de tarifa. Puede actualizar el plan de tarifa en cualquier momento. Para más información, consulte [Precios de Azure Bot Service](https://azure.microsoft.com/pricing/details/bot-service/).              |
   |      <strong>Punto de conexión de mensajería</strong>       |               URL               |                                                                               Escriba la dirección URL de punto de conexión de mensajería del bot.                                                                                |
   |     <strong>Application Insights</strong>      |               Por                | Decida si quiere <strong>Activar</strong> o <strong>Desactivar</strong> [Application Insights](bot-service-manage-analytics.md). Si selecciona <strong>Activar</strong>, también debe especificar una ubicación regional. |
   | <strong>Id. y contraseña de la aplicación de Microsoft</strong> | Creación automática del id. y contraseña de la aplicación |              Use esta opción si tiene que escribir manualmente un id. y contraseña de aplicación de Microsoft. En caso contrario, un Id. y contraseña nuevos de aplicación de Microsoft se crearán automáticamente en el proceso de creación del bot.               |


4. Haga clic en **Crear** para crear el servicio y registrar punto de conexión de mensajería del bot.

Para confirmar que se ha creado el registro, consulte las **Notificaciones**. Las notificaciones cambiarán de **Implementación en curso...** a **Implementación correcta**. Haga clic en el botón **Ir al recurso** para abrir la hoja de recursos del bot. 

## <a name="bot-channels-registration-password"></a>Contraseña de Bot Channels Registration

El servicio de bots **Bot Channels Registration** no tiene asociado un servicio de aplicación. Por este motivo, este servicio de bots solo tiene un *MicrosoftAppID*. Debe generar manualmente la contraseña y guardarla usted mismo. Necesitará esta contraseña si desea probar el bot con el [emulador](bot-service-debug-emulator.md).

Para generar un MicrosoftAppPassword, haga lo siguiente:

1. En la hoja **Configuración**, haga clic en **Administrar**. Este es el vínculo que aparecen junto al **Id. de aplicación de Microsoft**. Este vínculo abrirá una ventana donde puede generar una nueva contraseña. <br/>
  ![Vínculo Administrar en la hoja Configuración](~/media/azure-bot-quickstarts/registration-settings-manage-link.png)

2. Haga clic en **Generar nueva contraseña**. Esto generará una nueva contraseña para el bot. Copie esta contraseña y guárdela en un archivo. Esta es la única vez que verá esta contraseña. Si no guarda la contraseña completa, deberá repetir el proceso para crear una nueva contraseña en caso de que la necesite más adelante. <br/>
  ![Generar contraseña de aplicación de Microsoft](~/media/azure-bot-quickstarts/registration-generate-app-password.png)

## <a name="update-the-bot"></a>Actualización del bot

Si usa Bot Framework SDK para Node.js, establezca las variables de entorno siguientes:

* MICROSOFT_APP_ID
* MICROSOFT_APP_PASSWORD

Si usa Bot Framework SDK para. NET, establezca los siguientes valores de clave en el archivo web.config:

* MicrosoftAppId
* MicrosoftAppPassword

## <a name="test-the-bot"></a>Probar el bot

Una vez ha creado el servicio de bots, [pruébelo en Chat en web](bot-service-manage-test-webchat.md). Escriba un mensaje y el bot debería responder.

## <a name="next-steps"></a>Pasos siguientes

En este tema, ha aprendido cómo registrar su bot hospedado en Bot Service. El siguiente paso es obtener información sobre cómo administrar Bot Service.

> [!div class="nextstepaction"]
> [Administración de un bot](bot-service-manage-overview.md)


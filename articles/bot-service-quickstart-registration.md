---
title: Creación de Bot Channels Registration con Bot Service | Microsoft Docs
description: Obtenga información sobre cómo registrar un bot existente en Bot Service.
ms.author: kamrani
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 014f5c998fcb9d322439ca8b0e0bf2ba5f9f0679
ms.sourcegitcommit: 0b647dc6716b0c06f04ee22ebdd7b53039c2784a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2019
ms.locfileid: "70076513"
---
# <a name="register-a-bot-with-azure-bot-service"></a>Registro de un bot con Azure Bot Service

En este tema se muestra cómo crear un recurso de **Azure Bot Service** para registrar el bot. Lo necesitará si el bot se hospeda en otro lugar y desea que esté disponible en Azure y que se conecte a los canales de Azure Bot Service.

Esto le permite compilar, conectarse y administrar su bot para interactuar con los usuarios, dondequiera que estos se encuentren, mediante Cortana, Skype, Messenger y muchos otros servicios.

> [!IMPORTANT] 
> Solo deberá registrar su bot si no está hospedado en Azure. Si [creó un bot](v4sdk/abs-quickstart.md) mediante Azure Portal, el bot ya está registrado en el servicio.

## <a name="create-a-registration-resource"></a>Creación de un recurso de registro

1. En el explorador, vaya a [Azure Portal](https://ms.portal.azure.com).

    > [!TIP]
    > Si no tiene una suscripción, puede registrarse para obtener una <a href="https://azure.microsoft.com/free/" target="_blank">cuenta gratuita</a>.

1. En el panel izquierdo, haga clic en **Crear un recurso**.
1. En el cuadro de selección del panel derecho, escriba *bot*. Y, en la lista desplegable, seleccione **Registro de canales de bot** o **Bot de aplicación web** según su aplicación.
Para **Bot de aplicación web**, siga los pasos descritos en el artículo: [Creación de un bot con Azure Bot Service](v4sdk/abs-quickstart.md). Va a crear un bot en Azure que se registra automáticamente con Azure Bot Service.
1. Haga clic en el botón **Crear** para iniciar el proceso.
1. En la hoja **Bot Service**, indique la información solicitada sobre el bot según se especifica en la tabla debajo de la imagen.  

   ![Hoja de creación de registro de bot](media/azure-bot-quickstarts/registration-create-bot-service-blade.png)

   |Configuración |Valor sugerido|DESCRIPCIÓN|
   |---|---|--|
   |**Nombre del bot** <img width="300px">|Nombre para mostrar del bot|Nombre para mostrar del bot que aparece en los canales y directorios. Este nombre se puede cambiar en cualquier momento.|
   |**Suscripción**|Su suscripción|Seleccione la suscripción de Azure que quiere usar.|
   |**Grupo de recursos**|myResourceGroup|Puede crear un [grupo de recursos](/azure/azure-resource-manager/resource-group-overview#resource-groups) o elegir uno existente.|
   |**Ubicación**|Oeste de EE. UU.|Elija una ubicación cerca de donde se implementa el bot o cerca de otros servicios a los que tendrá acceso su bot.|
   |**Plan de tarifa**|F0|Seleccione un plan de tarifa. Puede actualizar el plan de tarifa en cualquier momento. Para más información, consulte [Precios de Azure Bot Service](https://azure.microsoft.com/pricing/details/bot-service/).|
   |**Punto de conexión de mensajería**|URL|Escriba la dirección URL de punto de conexión de mensajería del bot.|
   |**Application Insights**|Por| Decida si quiere **Activar** o **Desactivar** [Application Insights](bot-service-manage-analytics.md). Si selecciona **Activar**, también debe especificar una ubicación regional. |
   |**Id. y contraseña de la aplicación de Microsoft**| Creación automática del id. y contraseña de la aplicación |Use esta opción si tiene que escribir manualmente un id. y contraseña de aplicación de Microsoft. Consulte la siguiente sección [Registro de aplicación manual](#manual-app-registration). En caso contrario, se crearán automáticamente un identificador y contraseña de aplicación de Microsoft en el proceso de registro. |

    > [!IMPORTANT]
    > No olvide escribir la dirección URL del punto de conexión de mensajería del bot.

1. Haga clic en el botón **Crear**. Espere a que el recurso se cree. Aparecerá en la lista de recursos.

### <a name="get-registration-password"></a>Obtención de la contraseña de registro

Una vez completado el registro, Azure Active Directory asigna un identificador de aplicación único al registro y se le dirige a la página *Información general* de la aplicación.

Para obtener la contraseña, siga los pasos que se describen a continuación.

1. En lista de recursos, haga clic en el recurso de Azure App Service que acaba de crear.
1. En el panel derecho, en la hoja de recursos, haga clic en **Configuración**. Aparece la página *Configuración* del recurso.
1. En la página Configuración, copie el **identificador de aplicación de Microsoft** generado y guárdelo en un archivo.
1. Haga clic en el vínculo **Administrar** junto al *identificador de aplicación de Microsoft*.

    ![Hoja de creación de registro de bot](media/azure-bot-quickstarts/bot-channels-registration-app-settings.png)

1. En la página de *Certificados y secretos* que aparece, haga clic en el botón **Nuevo secreto de cliente**.

    ![Hoja de creación de registro de bot](media/azure-bot-quickstarts/bot-channels-registration-app-secrets.png)

1. Agregue la descripción, seleccione la fecha de expiración y haga clic en el botón **Agregar**.

    ![Hoja de creación de registro de bot](media/azure-bot-quickstarts/bot-channels-registration-app-secrets-create.png)

    Esto generará una nueva contraseña para el bot. Copie esta contraseña y guárdela en un archivo. Esta es la única vez que verá esta contraseña. Si no guarda la contraseña completa, deberá repetir el proceso para crear una nueva contraseña en caso de que la necesite más adelante.

## <a name="manual-app-registration"></a>Registro de aplicación manual

Un registro manual es necesario en situaciones como:

- No puede realizar registros en su organización y necesita otra entidad para crear el identificador de aplicación para el bot que está compilando.
- Deberá crear manualmente su propio identificador de aplicación (y contraseña).

Consulte [Preguntas más frecuentes sobre el registro de aplicación](bot-service-resources-bot-framework-faq.md#app-registration).

> [!IMPORTANT]
> En la sección *Admite tipos de cuenta*, debe elegir uno de los 2 tipos de multiinquilino, es decir: *Cuentas en cualquier directorio organizativo (todas las instancias de Azure AD: multiinquilino)* o *Cuentas en cualquier directorio organizativo (todas las instancias de Azure AD: multiinquilino) y cuentas personales de Microsoft (por ejemplo, Skype, Xbox, Outlook.com)* , al crear la aplicación ya que, de lo contrario, el bot no funcionará. Para más información, consulte [Registro de una aplicación nueva mediante Azure Portal](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app#register-a-new-application-using-the-azure-portal).

## <a name="update-the-bot"></a>Actualización del bot

Si usa Bot Framework SDK para. NET, establezca los siguientes valores de clave en el archivo web.config:

- `MicrosoftAppId = <appId>`
- `MicrosoftAppPassword = <appSecret>`

Si usa Bot Framework SDK para Node.js, establezca las variables de entorno siguientes:

- `MICROSOFT_APP_ID = <appId>`
- `MICROSOFT_APP_PASSWORD = <appSecret>`

## <a name="test-the-bot"></a>Probar el bot

Una vez ha creado el servicio de bots, [pruébelo en Chat en web](bot-service-manage-test-webchat.md). Escriba un mensaje y el bot debería responder.

## <a name="next-steps"></a>Pasos siguientes

En este tema, ha aprendido cómo registrar su bot hospedado en Bot Service. El siguiente paso es obtener información sobre cómo administrar Bot Service.

> [!div class="nextstepaction"]
> [Administración de un bot](bot-service-manage-overview.md)

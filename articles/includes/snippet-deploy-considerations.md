---
ms.openlocfilehash: 4e2ab87f509c29c33a27810668445975e31f6ac4
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563843"
---
## <a name="application-settings-and-messaging-endpoint"></a>Configuración de la aplicación y punto de conexión de mensajería

### <a name="verify-application-settings"></a>Verificación de la configuración de la aplicación

Para que el bot funcione correctamente en la nube, debe asegurarse de que la configuración de la aplicación sea correcta. Si tiene un **Id. de aplicación** y una **contraseña**, actualice los valores `Microsoft AppId` y `Microsoft App Password` en la configuración de la aplicación como parte del proceso de implementación. Para buscar **AppID** y **AppPassword** del bot, consulte [MicrosoftAppID y MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

> [!TIP]
> [!INCLUDE [Application configuration settings](~/includes/snippet-tip-bot-config-settings.md)]

Si aún no ha registrado el bot en Bot Framework y, por tanto, no dispone de un **Id. de aplicación** y una **contraseña**, puede implementar el bot con valores de marcadores de posición temporales para estas configuraciones.
Más adelante, después de [registrar el bot](~/bot-service-quickstart-registration.md), actualice la configuración de la aplicación implementada con los valores de **Id. de aplicación** y **contraseña** generados para el bot durante el registro.

### <a id="messagingEndpoint"></a> Verificación del punto de conexión de mensajería

El bot implementado debe tener un **punto de conexión de mensajería** que pueda recuperar los mensajes del servicio Bot Framework Connector.

> [!NOTE]
> Al implementar el bot en Azure, SSL se configurará automáticamente para la aplicación, lo que permite habilitar el **punto de conexión de mensajería** que Bot Framework requiere.
> Si realiza la implementación en otro servicio en la nube, asegúrese de verificar que la aplicación está configurada para SSL, a fin de que el bot disponga de un **punto de conexión de mensajería**.

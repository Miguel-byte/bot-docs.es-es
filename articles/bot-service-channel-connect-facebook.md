---
title: Conexión de un bot a Facebook Messenger | Microsoft Docs
description: Obtenga información sobre cómo configurar la conexión de un bot a Facebook Messenger.
keywords: Facebook Messenger, canal de bot, aplicación de Facebook, id. de aplicación, secreto de aplicación, bot de Facebook, credenciales
manager: kamrani
ms.topic: article
ms.author: kamrani
ms.service: bot-service
ms.date: 10/28/2019
ms.openlocfilehash: ba9f0ec8f4b6f0e2f5a7069ab456f96b303a4c92
ms.sourcegitcommit: 4751c7b8ff1d3603d4596e4fa99e0071036c207c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/02/2019
ms.locfileid: "73441635"
---
# <a name="connect-a-bot-to-facebook"></a>Conexión de un bot a Facebook

El bot se puede conectar a Facebook Messenger y Facebook Workplace para comunicarse con usuarios de estas plataformas. En el siguiente tutorial se muestra cómo conectar un bot con estos dos canales.

> [!NOTE]
> La UI de Facebook puede aparecer de forma ligeramente diferente dependiendo de la versión que use.

## <a name="connect-a-bot-to-facebook-messenger"></a>Conexión de un bot a Facebook Messenger

> [!NOTE]
> A partir del 16 de diciembre de 2019, Workplace by Facebook cambia el modelo de seguridad de las integraciones personalizadas.  Las integraciones actuales creadas con Microsoft Bot Framework deben actualizarse para usar el adaptador de Bot Framework (disponible en JavaScript/node.js) e implementarse mediante una aplicación web en Azure.  Los nuevos bots de área de trabajo que se desarrollen con Microsoft Bot Framework también deben usar el adaptador de Facebook de JavaScript. Obtenga más información acerca de cómo [usar el adaptador de Facebook](https://aka.ms/botframework-workplace-adapter). Las instrucciones siguientes solo funcionarán hasta el 16 de diciembre de 2019.

Para obtener más información acerca del desarrollo de Facebook Messenger, consulte la [documentación de la plataforma de Messenger](https://developers.facebook.com/docs/messenger-platform). Puede que quiera revisar la [lista de verificación previa al lanzamiento](https://developers.facebook.com/docs/messenger-platform/product-overview/launch#app_public), el [tutorial de inicio rápido](https://developers.facebook.com/docs/messenger-platform/guides/quick-start) y la [guía de configuración](https://developers.facebook.com/docs/messenger-platform/guides/setup) de Facebook.

Para configurar un bot para que se comunique con Facebook Messenger, habilite Facebook Messenger en una página de Facebook y, a continuación, conecte el bot.

### <a name="copy-the-page-id"></a>Copia del id. de página

Se accede al bot a través de una página de Facebook.

1. [Cree una nueva página de Facebook](https://www.facebook.com/bookmarks/pages) o vaya a una página existente.

1. Abra la página **Sobre** de la página de Facebook y, a continuación, copie y guarde el **id. de página**.

### <a name="create-a-facebook-app"></a>Creación de una aplicación de Facebook

1. En el explorador, vaya a [Crear una nueva aplicación de Facebook](https://developers.facebook.com/quickstarts/?platform=web).
1. Escriba el nombre de la aplicación y haga clic en el botón **Crear nuevo id. de aplicación de Facebook**.

    ![Crear aplicación](media/channels/fb-create-messenger-bot-app.png)

1. En el cuadro de diálogo que aparece, escriba su dirección de correo electrónico y haga clic en el botón **Crear id. de aplicación**.

    ![Creación de un id. de aplicación](media/channels/fb-create-messenger-bot-app-id.png)

1. Realice los pasos que le indique el asistente.

1. Escriba la información de comprobación necesaria y haga clic en el botón **Omitir Inicio rápido** en la parte superior derecha.

1. En el panel izquierdo de la ventana que se muestra a continuación, expanda *Configuración* y haga clic en **Básico**.

1. En el panel derecho, copie y guarde el **id. de aplicación** y el **secreto de aplicación**.

    ![Copiar identificador y secreto de la aplicación](media/channels/fb-messenger-bot-get-appid-secret.png)

1. En el panel izquierdo, en *Configuración*, haga clic en **Avanzado**.

1. En el panel derecho, establezca el control deslizante **Permitir el acceso de la API a la configuración de la aplicación** en **Sí**.

    ![Copiar identificador y secreto de la aplicación](media/channels/fb-messenger-bot-api-settings.png)

1. En la página inferior derecha, haga clic en el botón **Guardar cambios**.

### <a name="enable-messenger"></a>Habilitación de Messenger

1. En el panel izquierdo, haga clic en **Panel**.
1. En el panel derecho, desplácese hacia abajo y, en el cuadro **Messenger**, haga clic en el botón **Configurar**. La entrada de Messenger se muestra en la sección *PRODUCTOS* del panel izquierdo.  

    ![Habilitación de Messenger](media/channels/fb-messenger-bot-enable-messenger.png)

### <a name="generate-a-page-access-token"></a>Generación de un token de acceso a la página

1. En el panel izquierdo, en la entrada de Messenger, haga clic en **Configuración**.
1. En el panel derecho, desplácese hacia abajo y, en la sección **Generación de tokens**, seleccione la página de destino.

    ![Habilitación de Messenger](media/channels/fb-messenger-bot-select-messenger-page.png)

1. Haga clic en el botón **Editar permisos** para conceder a la aplicación pages_messaging permiso para generar un token de acceso.
1. Siga las instrucciones del asistente. En el último paso, acepte la configuración predeterminada y haga clic en el botón **Listo**. Al final, se genera un **token de acceso a la página**.

    ![Permisos de Messenger](media/channels/fb-messenger-bot-permissions.png)

1. Copie y guarde el **token de acceso a la página**.

### <a name="enable-webhooks"></a>Habilitación de webhooks

Para enviar mensajes y otros eventos desde el bot a Facebook Messenger, debe habilitar la integración de webhooks. En este punto, vamos a dejar pendientes los pasos de configuración de Facebook; volveremos más tarde a ellos.

1. En el explorador, abra una nueva ventana y vaya a [Azure portal](https://portal.azure.com/). 

1. En la lista de recursos, haga clic en el registro de recursos del bot y, en la hoja correspondiente, haga clic en **Canales**.

1. En el panel derecho, haga clic en el icono de **Facebook**.

1. En el asistente, escriba la información de Facebook almacenada en los pasos anteriores. Si la información es correcta, en la parte inferior del asistente debería ver la **dirección URL de devolución de llamada** y el **token de comprobación**. Cópielos y almacénelos.  

    ![configuración del canal de fb messenger](media/channels/fb-messenger-bot-config-channel.PNG)

1. Haga clic en el botón **Save** (Guardar).


1. Volvamos a la configuración de Facebook. En el panel derecho, desplácese hacia abajo y, en la sección **Webhooks**, haga clic en el botón **Suscribirse a eventos**. Esto es para reenviar eventos de mensajería de Facebook Messenger al bot.

    ![Habilitación de webhooks](media/channels/fb-messenger-bot-webhooks.PNG)

1. En el cuadro de diálogo que aparece, escriba la **dirección URL de devolución de llamada** y compruebe los valores del **token de comprobación** almacenados anteriormente. En **Campos de suscripción**, seleccione *message\_deliveries*, *messages*, *messaging\_options* y *messaging\_postbacks*.

    ![Configurar webhooks](media/channels/fb-messenger-bot-config-webhooks.png)

1. Haga clic en el botón **Comprobar y guardar**.
1. Seleccione la página de Facebook para suscribirse al webhook. Haga clic en el botón **Suscribirse**.

    ![Página Configurar webhooks](media/channels/fb-messenger-bot-config-webhooks-page.PNG)

### <a name="submit-for-review"></a>Envío para revisión

Facebook requiere una dirección URL de la directiva de privacidad y una dirección URL de las Condiciones del servicio en su página de configuración de aplicación básica. La página [Código de conducta](https://investor.fb.com/corporate-governance/code-of-conduct/default.aspx) contiene vínculos a recursos de terceros para ayudar a crear una directiva de privacidad. La página [Condiciones de uso](https://www.facebook.com/terms.php) contiene los términos de ejemplo que le ayudarán a crear un documento de condiciones del servicio adecuado.

Al finalizar el bot, Facebook tiene su propio [proceso de revisión](https://developers.facebook.com/docs/messenger-platform/app-review) para las aplicaciones que se publican en Messenger. El bot se probará para asegurarse de que es compatible con las [políticas de la plataforma](https://developers.facebook.com/docs/messenger-platform/policy-overview) de Facebook.

### <a name="make-the-app-public-and-publish-the-page"></a>Publicación de la aplicación y la página

> [!NOTE]
> Una aplicación se encuentra en [modo de desarrollo](https://developers.facebook.com/docs/apps/managing-development-cycle) hasta que no se publica. Los complementos y las funcionalidades de API solo funcionarán para los administradores, los desarrolladores y los evaluadores.

Una vez que la revisión se considere correcta, en el panel de aplicaciones en Revisión de la aplicación, establezca la aplicación en Pública.
Asegúrese de que la página de Facebook asociada a este bot esté publicada. El estado aparece en la configuración de las páginas.

## <a name="connect-a-bot-to-facebook-workplace"></a>Conexión de un bot a Facebook Workplace

> [!NOTE]
> A partir del 16 de diciembre de 2019, Workplace by Facebook cambia el modelo de seguridad de las integraciones personalizadas.  Las integraciones actuales creadas con Microsoft Bot Framework V4 en JavaScript y Node.js deben actualizarse para usar el [Adaptador Facebook for Workplace](https://aka.ms/botframework-workplace-adapter) de Bot Framework e implementarse con Azure Web Apps para seguir en funcionamiento después de esa fecha. Los nuevos bots de Microsoft Bot Framework que se dirigen al área de trabajo también deben desarrollarse con ese adaptador.

Facebook Workplace es una versión de Facebook orientada a la empresa que permite a los empleados conectarse y colaborar fácilmente. Contiene vídeos en directo, fuentes de noticias, grupos, Messenger, reacciones, búsquedas y publicaciones populares. También admite:

- Análisis e integraciones. Un panel con análisis, integración, inicio de sesión único y proveedores de identidades que las compañías usan para integrar Workplace con sus sistemas de TI existentes.
- Grupos de varias empresas. Espacios compartidos en los que los empleados de distintas organizaciones pueden trabajar juntos y colaborar.

Consulte el [servicio de ayuda Workplace](https://workplace.facebook.com/help/work/) para aprender sobre Facebook Workplace y la [documentación para desarrolladores de Workplace](https://developers.facebook.com/docs/workplace) para instrucciones sobre cómo desarrollar para Facebook Workplace.

Para usar Facebook Workplace con el bot, debe crear una cuenta de Workplace y una integración personalizada para conectar el bot.

### <a name="create-a-workplace-premium-account"></a>Creación de una cuenta de Workplace Premium

1. Envíe una aplicación a [Workplace](https://www.facebook.com/workplace) en nombre de su empresa.
1. Una vez aprobada la aplicación, recibirá un correo electrónico en el que se le invitará a unirse. La respuesta puede tardar un rato.
1. En la invitación del correo electrónico, haga clic en **Empezar**.
1. Escriba su información de perfil.
    > [!TIP]
    > Establézcase como administrador del sistema. Recuerde que solo los administradores del sistema pueden crear integraciones personalizadas.
1. Haga clic en **Preview Profile** (Vista previa del perfil) y compruebe que la información es correcta.
1. Acceda a *Prueba gratuita*.
1. Crea una **contraseña**.
1. Haga clic en **Invitar a compañeros de trabajo** para invitar a los empleados a iniciar sesión. Los empleados a los que ha invitado se convertirán en miembros en cuanto inicien sesión. Pasarán por un proceso de inicio de sesión parecido al que se describe en estos pasos.

### <a name="create-a-custom-integration"></a>Creación de una integración personalizada

Cree una [integración personalizada](https://developers.facebook.com/docs/workplace/custom-integrations-new) para Workplace con los pasos que se describen a continuación. Al crear una integración personalizada se crean una aplicación con permisos definidos y una página de tipo bot que solo se ven en la comunidad de Workplace.

1. En el **panel de administración**, abra la pestaña **Integraciones**.
1. Haga clic en el botón **Create your own custom App** (Crear una aplicación personalizada propia).

    ![Integración en Workplace](media/channels/fb-integration.png)

1. Elija un nombre para mostrar y una imagen de perfil para la aplicación. Esta información se compartirá con la página de tipo bot.
1. Establezca **Allow API Access to App Settings** (Permitir el acceso de la API a la configuración de la aplicación) en "Sí".
1. Copie el identificador, el secreto y el token de la aplicación que aparecen y guárdelos con seguridad.

    ![Claves de Workplace](media/channels/fb-keys.png)

1. Ahora ha terminado de crear la integración personalizada. Encontrará la página de tipo bot en la comunidad de Workplace, como se muestra a continuación.

    ![Página de Workplace](media/channels/fb-page.png)

### <a name="provide-facebook-credentials"></a>Aprovisionamiento de credenciales de Facebook

En Azure Portal, copie los valores de **Facebook App ID** (Id. de aplicación de Facebook), **Facebook App Secret** (Secreto de aplicación de Facebook) y **Page Access Token** (Token de acceso a página) que copió de Facebook Workplace anteriormente. En lugar de un valor de pageID tradicional, use los números que siguen al nombre de la integración en la página **Acerca de**. Los webhooks se pueden conectar con las credenciales que se muestran en Azure, de manera parecida a conectar un bot a Facebook Messenger.

### <a name="submit-for-review"></a>Envío para revisión
Consulte la sección **Conexión de un bot a Facebook Messenger** y la [documentación para desarrolladores de Workplace](https://developers.facebook.com/docs/workplace) para más información.

### <a name="make-the-app-public-and-publish-the-page"></a>Publicación de la aplicación y la página
Consulte la sección**Conexión de un bot a Facebook Messenger** para más información.

## <a name="setting-the-api-version"></a>Configuración de la versión de API

Si recibe una notificación de Facebook sobre una determinada versión de Graph API en desuso, vaya a la [página para desarrolladores de Facebook](https://developers.facebook.com). Vaya a la sección **Configuración de la aplicación** del bot y a **Configuración > Avanzada > Upgrade API version (Actualizar versión de API)** y, después, cambie la opción **Upgrade All Calls (Actualizar todas las llamadas)** a 3.0.

![Actualización de la versión de API](media/channels/fb-version-upgrade.png)

## <a name="see-also"></a>Otras referencias

- **Código de ejemplo**. Use el bot de ejemplo <a href="https://aka.ms/facebook-events" target="_blank">Facebook-events</a> para explorar la comunicación del bot con Facebook Messenger.

- **Disponible como adaptador**. Este canal también está [disponible como un adaptador](https://botkit.ai/docs/v4/platforms/facebook.html). Para ayudarle a elegir entre un adaptador y un canal, consulte [Adaptadores disponibles actualmente](bot-service-channel-additional-channels.md#currently-available-adapters).

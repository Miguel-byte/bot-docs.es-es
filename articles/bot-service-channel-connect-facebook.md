---
title: Conexión de un bot a Facebook Messenger | Microsoft Docs
description: Obtenga información sobre cómo configurar la conexión de un bot a Facebook Messenger.
keywords: Facebook Messenger, canal de bot, aplicación de Facebook, id. de aplicación, secreto de aplicación, bot de Facebook, credenciales
author: RobStand
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/12/2018
ms.openlocfilehash: 57a3efd36ddae5c52a2d791b87ed4fa6a96d5e8a
ms.sourcegitcommit: 152760771214865b9c7d0ed481acfba05bdc44dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2019
ms.locfileid: "58655525"
---
# <a name="connect-a-bot-to-facebook"></a>Conexión de un bot a Facebook

El bot se puede conectar a Facebook Messenger y Facebook Workplace para comunicarse con usuarios de estas plataformas. En el siguiente tutorial se muestra cómo conectar un bot con estos dos canales paso a paso.

> [!NOTE]
> La UI de Facebook puede aparecer de forma ligeramente diferente dependiendo de la versión que use.

## <a name="connect-a-bot-to-facebook-messenger"></a>Conexión de un bot a Facebook Messenger

Para obtener más información acerca del desarrollo de Facebook Messenger, consulte la [documentación de la plataforma de Messenger](https://developers.facebook.com/docs/messenger-platform). Puede que quiera revisar la [lista de verificación previa al lanzamiento](https://developers.facebook.com/docs/messenger-platform/product-overview/launch#app_public), el [tutorial de inicio rápido](https://developers.facebook.com/docs/messenger-platform/guides/quick-start) y la [guía de configuración](https://developers.facebook.com/docs/messenger-platform/guides/setup) de Facebook.

Para configurar un bot para que se comunique con Facebook Messenger, habilite Facebook Messenger en una página de Facebook y, a continuación, conecte el bot a la aplicación.

### <a name="copy-the-page-id"></a>Copia del id. de página

Se accede al bot a través de una página de Facebook. [Cree una nueva página de Facebook](https://www.facebook.com/bookmarks/pages) o vaya a una página existente.

* Abra la página **Sobre** de la página de Facebook y, a continuación, copie y guarde el **id. de página**.

### <a name="create-a-facebook-app"></a>Creación de una aplicación de Facebook

[Cree una nueva aplicación de Facebook](https://developers.facebook.com/quickstarts/?platform=web) en la página y genere un id. y un secreto de aplicación.

![Creación de un id. de aplicación](~/media/channels/FB-CreateAppId.png)

* Copie y guarde el **id. de aplicación** y el **secreto de aplicación**.

![Guardar el id. y el secreto de aplicación](~/media/channels/FB-get-appid.png)

Establezca el control deslizante "Allow API Access to App Settings" (Permitir el acceso de la API a la configuración de la aplicación) en "Sí".

![Configuración de la aplicación](~/media/bot-service-channel-connect-facebook/api_settings.png)

### <a name="enable-messenger"></a>Habilitación de Messenger

Habilite Facebook Messenger en la nueva aplicación de Facebook.

![Habilitación de Messenger](~/media/channels/FB-AddMessaging1.png)

### <a name="generate-a-page-access-token"></a>Generación de un token de acceso a la página

En el panel **Generación de tokens** de la sección de Messenger, seleccione la página de destino. Se generará un token de acceso a la página.

* Copie y guarde el **token de acceso a la página**.

![Generación del token](~/media/channels/FB-generateToken.png)

### <a name="enable-webhooks"></a>Habilitación de webhooks

Haga clic en **Configurar webhooks** para reenviar eventos de mensajería de Facebook Messenger al bot.

![Habilitación del webhook](~/media/channels/FB-webhook.png)

### <a name="provide-webhook-callback-url-and-verify-token"></a>Aprovisionamiento de la dirección URL de devolución de llamada de webhook y comprobación del token

En [Azure Portal](https://portal.azure.com/), abra el bot, haga clic en la pestaña **Canales** y, a continuación, haga clic en **Facebook Messenger**.

* Copie los valores **Dirección URL de devolución de llamada** y **Comprobar token** desde el portal.

![Copia de valores](~/media/channels/fb-callbackVerify.png)

1. Vuelva a Facebook Messenger y pegue los valores **Dirección URL de devolución de llamada** y **Comprobar token**.

2. En **Campos de suscripción**, seleccione *message\_deliveries*, *messages*, *messaging\_options* y *messaging\_postbacks*.

3. Haga clic en **Comprobar y guardar**.

![Configuración del webhook](~/media/channels/FB-webhookConfig.png)

4. Subscriba el webhook a la página de Facebook.

![Suscripción del webhook](~/media/bot-service-channel-connect-facebook/subscribe-webhook.png)


### <a name="provide-facebook-credentials"></a>Aprovisionamiento de credenciales de Facebook

En Azure Portal, pegue los valores de **Identificador de aplicación de Facebook**, **Secreto de aplicación de Facebook**, **Identificador de página** y **Token de acceso a la página** copiados de Facebook Messenger anteriormente. Puede usar el mismo bot en varias páginas de Facebook mediante la adición de identificadores de página y tokens de acceso adicionales.

![Escribir credenciales](~/media/channels/fb-credentials2.png)

### <a name="submit-for-review"></a>Envío para revisión

Facebook requiere una dirección URL de la directiva de privacidad y una dirección URL de las Condiciones del servicio en su página de configuración de aplicación básica. La página [Código de conducta](https://investor.fb.com/corporate-governance/code-of-conduct/default.aspx) contiene vínculos a recursos de terceros para ayudar a crear una directiva de privacidad. La página [Condiciones de uso](https://www.facebook.com/terms.php) contiene los términos de ejemplo que le ayudarán a crear un documento de condiciones del servicio adecuado.

Al finalizar el bot, Facebook tiene su propio [proceso de revisión](https://developers.facebook.com/docs/messenger-platform/app-review) para las aplicaciones que se publican en Messenger. El bot se probará para asegurarse de que es compatible con las [políticas de la plataforma](https://developers.facebook.com/docs/messenger-platform/policy-overview) de Facebook.

### <a name="make-the-app-public-and-publish-the-page"></a>Publicación de la aplicación y la página

> [!NOTE]
> Una aplicación se encuentra en [modo de desarrollo](https://developers.facebook.com/docs/apps/managing-development-cycle) hasta que no se publica. Los complementos y las funcionalidades de API solo funcionarán para los administradores, los desarrolladores y los evaluadores.

Una vez que la revisión se considere correcta, en el panel de aplicaciones en Revisión de la aplicación, establezca la aplicación en Pública.
Asegúrese de que la página de Facebook asociada a este bot esté publicada. El estado aparece en la configuración de las páginas.

## <a name="connect-a-bot-to-facebook-workplace"></a>Conexión de un bot a Facebook Workplace

Consulte el [servicio de ayuda Workplace](https://workplace.facebook.com/help/work/) para aprender sobre Facebook Workplace y la [documentación para desarrolladores de Workplace](https://developers.facebook.com/docs/workplace) para instrucciones sobre cómo desarrollar para Facebook Workplace.

Para configurar un bot para que se comunique mediante Facebook Workplace, cree una integración personalizada y conecte el bot con ella.

### <a name="create-a-facebook-workplace-premium-account"></a>Creación de una cuenta de Facebook Workplace Premium

Siga [estas](https://www.facebook.com/workplace) instrucciones para crear una cuenta de Facebook Workplace Premium y establecerse como administrador del sistema. Tenga en cuenta que solo el administrador del sistema de Workplace puede crear integraciones personalizadas.

### <a name="create-a-custom-integration"></a>Creación de una integración personalizada

Al crear una integración personalizada se crean una aplicación con permisos definidos y una página de tipo bot que solo se ven en la comunidad de Workplace.

Cree una [integración personalizada](https://developers.facebook.com/docs/workplace/custom-integrations-new) para Workplace con los siguientes pasos:

- En el **panel de administración**, abra la pestaña **Integraciones**.
- Haga clic en el botón **Create your own custom App** (Crear una aplicación personalizada propia).

![Integración en Workplace](~/media/channels/fb-integration.png)

- Elija un nombre para mostrar y una imagen de perfil para la aplicación. Esta información se compartirá con la página de tipo bot.
- Establezca **Allow API Access to App Settings** (Permitir el acceso de la API a la configuración de la aplicación) en "Sí".
- Copie el identificador, el secreto y el token de la aplicación que aparecen y guárdelos con seguridad.

![Claves de Workplace](~/media/channels/fb-keys.png)

Ahora ha terminado de crear la integración personalizada. Encontrará la página de tipo bot en la comunidad de Workplace, como se muestra a continuación.

![Página de Workplace](~/media/channels/fb-page.png)

### <a name="provide-facebook-credentials"></a>Aprovisionamiento de credenciales de Facebook

En Azure Portal, copie los valores de **Facebook App ID** (Id. de aplicación de Facebook), **Facebook App Secret** (Secreto de aplicación de Facebook) y **Page Access Token** (Token de acceso a página) que copió de Facebook Workplace anteriormente. En lugar de un valor de pageID tradicional, use los números que siguen al nombre de la integración en la página **Acerca de**. Los webhooks se pueden conectar con las credenciales que se muestran en Azure, de manera parecida a conectar un bot a Facebook Messenger.

### <a name="submit-for-review"></a>Envío para revisión
Consulte la sección **Conexión de un bot a Facebook Messenger** y la [documentación para desarrolladores de Workplace](https://developers.facebook.com/docs/workplace) para más información.

### <a name="make-the-app-public-and-publish-the-page"></a>Publicación de la aplicación y la página
Consulte la sección**Conexión de un bot a Facebook Messenger** para más información.

## <a name="setting-the-api-version"></a>Configuración de la versión de API

Si recibe una notificación de Facebook sobre una determinada versión de Graph API en desuso, vaya a la [página para desarrolladores de Facebook](https://developers.facebook.com). Vaya a la sección **Configuración de la aplicación** del bot y a **Configuración > Avanzada > Upgrade API version (Actualizar versión de API)** y, después, cambie la opción **Upgrade All Calls (Actualizar todas las llamadas)** a 3.0.

![Actualización de la versión de API](~/media/channels/facebook-version-upgrade.png)

## <a name="sample-code"></a>Código de ejemplo

Para más información, con el bot de ejemplo <a href="https://aka.ms/facebook-events" target="_blank">Facebook-events</a> puede explorar la comunicación del bot con Facebook Messenger.

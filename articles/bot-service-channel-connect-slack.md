---
title: Conexión de un bot a Slack | Microsoft Docs
description: Obtenga información sobre cómo configurar la conexión de un bot a Slack.
keywords: conectar un bot, canal de bot, bot de Slack, aplicación de mensajería Slack
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/09/2019
ms.openlocfilehash: 3573103e1d1c55e3ad648ad68d84674a98b397f7
ms.sourcegitcommit: 8161753641368567f239e24a35ad61768acccd8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/10/2019
ms.locfileid: "54202569"
---
# <a name="connect-a-bot-to-slack"></a>Conexión de un bot a Slack

Puede configurar el bot para que se comunique con las personas mediante la aplicación de mensajería Slack.

## <a name="create-a-slack-application-for-your-bot"></a>Creación de una aplicación de Slack para el bot

Inicie sesión en [Slack](https://slack.com/signin) y vaya al canal de [creación de una aplicación de Slack](https://api.slack.com/apps).

![Configurar el bot](~/media/channels/slack-NewApp.png)

## <a name="create-an-app-and-assign-a-development-slack-team"></a>Creación de una aplicación y asignación de un equipo de desarrollo de Slack

Escriba un nombre de aplicación y seleccione un equipo de desarrollo de Slack. Si aún no es miembro de un equipo de desarrollo de Slack, [cree uno o únase a uno](https://slack.com/).

![Creación de una aplicación](~/media/channels/slack-CreateApp.png)

Haga clic en **Create app** (Crear aplicación). Slack creará la aplicación y generará un id. de cliente y un secreto de cliente.

## <a name="add-a-new-redirect-url"></a>Adición de una nueva dirección URL de redireccionamiento

A continuación, agregará una nueva dirección URL de redireccionamiento.

1. Seleccione la pestaña **OAuth & Permissions** (OAuth y permisos).
2. Haga clic en **Add a new Redirect URL** (Adición de una nueva dirección URL de redireccionamiento).
3. Escriba https://slack.botframework.com.
4. Haga clic en **Agregar**.
5. Haga clic en **Save URLs** (Guardar direcciones URL).

![Agregar dirección URL de redireccionamiento](~/media/channels/slack-RedirectURL.png)

## <a name="create-a-slack-bot-user"></a>Creación de un usuario de bot de Slack

Agregar un usuario de bot le permite asignar un nombre de usuario al bot y elegir si siempre se muestra como en línea.

1. Seleccione la pestaña **Bot Users** (Usuarios de bot).
2. Haga clic en **Add a Bot User** (Agregar un usuario de bot).

![Crear bot](~/media/channels/slack-CreateBot.png)

Haga clic en **Add Bot User** para validar la configuración, haga clic en **Always Show My Bot as Online** (Mostrar siempre mi bot como en línea) para **activar** y, a continuación, haga clic en **Guardar cambios**.

![Crear bot](~/media/channels/slack-CreateApp-AddBotUser.png)

## <a name="subscribe-to-bot-events"></a>Suscribirse a eventos de bot

Siga estos pasos para suscribirse a seis eventos determinados de bot. Si se suscribe a eventos de bot, se notificará a la aplicación acerca de las actividades de los usuarios en la dirección URL que especifique.

> [!TIP]
> El identificador de bot es el nombre del bot. Para buscar el identificador de un bot, visite [https://dev.botframework.com/bots](https://dev.botframework.com/bots), elija un bot y registre el nombre del bot.

1. Seleccione la pestaña **Suscripciones a eventos**.
2. Haga clic en **Habilitar eventos** para **activar**.
3. En **URL de solicitud**, escriba `https://slack.botframework.com/api/Events/{YourBotHandle}`, donde `{YourBotHandle}` es el identificador del bot, sin las llaves. El identificador de bot utilizado en este ejemplo es **ContosoBot**.

   ![Suscripción a eventos: superior](~/media/channels/slack-SubscribeEvents-a.png)

4. En **Subscribe to Bot Events** (suscribirse a eventos de Bot), haga clic en **Add Bot User Event** (Agregar evento de usuario de bot).
5. En la lista de eventos, seleccione estos seis tipos de evento:
    * `member_joined_channel`
    * `member_left_channel`
    * `message.channels`
    * `message.groups`
    * `message.im`
    * `message.mpim`

   ![Suscripción a eventos: intermedio](~/media/channels/slack-SubscribeEvents-b.png)

6. Haga clic en **Guardar cambios**.

   ![Suscripción a eventos: inferior](~/media/channels/slack-SubscribeEvents-c.png)

## <a name="add-and-configure-interactive-messages-optional"></a>Adición y configuración de mensajes interactivos (opcional)

Si el bot usará funcionalidades específicas de Slack, como botones, siga estos pasos:

1. Seleccione la pestaña **Interactive Components** (Componentes interactivos) y haga clic en **Enable Interactive Components** (Habilitar componentes interactivos).
2. Escriba `https://slack.botframework.com/api/Actions` como **Dirección URL de la solicitud**.
3. Haga clic en el botón **Save changes** (Guardar cambios).

![Habilitar mensajes](~/media/channels/slack-MessageURL.png)

## <a name="gather-credentials"></a>Obtención de credenciales

Seleccione la pestaña **Información básica** y desplácese hasta la sección **Credenciales de la aplicación**.
Se muestran el id. de cliente, el secreto de cliente y el token de comprobación necesario para la configuración de un bot de Slack.

![Obtención de credenciales](~/media/channels/slack-AppCredentials.png)

## <a name="submit-credentials"></a>Envío de credenciales

En otra ventana del explorador, vuelva al sitio de Bot Framework en `https://dev.botframework.com/`.

1. Seleccione **Mis bots** y elija el bot que desea que se conecte con Slack.
2. En la sección **Channels** (Canales), haga clic en el icono de Slack.
3. En la sección **Enter your Slack credentials** (Escriba sus credenciales de Slack), pegue las credenciales de la aplicación desde el sitio web de Slack en los campos correspondientes.
4. La **Landing Page URL** (Dirección URL de la página de aterrizaje) es opcional. Puede omitirla o cambiarla.
5. Haga clic en **Save**(Guardar).

![Enviar las credenciales](~/media/channels/slack-SubmitCredentials.png)

Siga las instrucciones para autorizar el acceso de la aplicación Slack a su equipo de desarrollo de Slack.

## <a name="enable-the-bot"></a>Habilitación del bot

En la página Configure Slack (Configurar Slack), confirme que el control deslizante junto al botón Guardar esté establecido en **Habilitado**.
El bot está configurado para comunicarse con los usuarios en Slack.

## <a name="create-an-add-to-slack-button"></a>Creación de un botón Agregar a Slack

Slack proporciona el código HTML que se puede usar para ayudar a los usuarios de Slack a encontrar su bot en la sección *Add the Slack button* (Agregar el botón de Slack) de [esta página](https://api.slack.com/docs/slack-button).
Para usar este código HTML con el bot, reemplace el valor de href (comienza con `https://`) por la dirección URL que se encuentra en configuración del canal de Slack del bot.
Siga estos pasos para obtener la dirección URL de reemplazo.

1. En [https://dev.botframework.com/bots](https://dev.botframework.com/bots), haga clic en el bot.
2. Haga clic en **Channels** (Canales), haga clic con el botón derecho en la entrada denominada **Slack** y haga clic en **Copy link** (Copiar vínculo). Esta dirección URL ahora está en el Portapapeles.
3. Pegue esta dirección URL desde el Portapapeles en el código HTML proporcionado para el botón de Slack. Esta dirección URL reemplaza el valor de href proporcionado por Slack para este bot.

Los usuarios autorizados pueden hacer clic en el botón **Agregar a Slack** proporcionado por este código HTML modificado para comunicarse con el bot en Slack.

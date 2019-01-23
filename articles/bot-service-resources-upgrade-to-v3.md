---
title: Actualización de un bot a Bot Framework API v3 | Microsoft Docs
description: Aprenda a actualizar su bot a Bot Framework API v3.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 8d9b2ea2e2133c86428b537427433f9dd15216ee
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225950"
---
# <a name="upgrade-your-bot-to-bot-framework-api-v3"></a>Actualización de un bot a Bot Framework API v3

En el evento Build 2016, Microsoft anunció Microsoft Bot Framework y su iteración inicial de Bot Connector API, junto con los SDK de Bot Builder y Bot Connector. Desde entonces, hemos estado recopilando sus comentarios y trabajando activamente para mejorar la API REST y los SDK.

En julio de 2016, se lanzó Bot Framework API v3 y Bot Framework API v1 ha quedado en desuso. Los bots que usan API v1 han dejado de funcionar en Skype en diciembre de 2016 y en todos los canales restantes el 23 de febrero de 2017. Si ha creado un bot con API v1 y quiere que funcione de nuevo, debe actualizar a la API v3 siguiendo las instrucciones de este artículo. Para asegurarse de que comprende el proceso de actualización de fin a fin, lea este artículo completamente antes de comenzar. 

## <a name="step-1-get-your-app-id-and-password-from-the-bot-framework-portal"></a>Paso 1: Obtención del identificador y la contraseña de la aplicación del portal de Bot Framework

Inicie sesión en el [portal de Bot Framework](https://dev.botframework.com/), haga clic en **My bots** (Mis bots) y seleccione su bot para abrir su panel. A continuación, haga clic en el vínculo **SETTINGS** (Configuración) que se encuentra parte izquierda de la página en **Bot Management** (Administración de bots). 

En la sección **Configuration** (Configuración) de la página de configuración, examine el contenido del campo **Microsoft App ID** (Id. de aplicación de Microsoft) y continúe con los pasos siguientes.

<!-- TODO: Remove this 
### Case 1: App ID field is already populated

If the **App ID** field is already populated, complete these steps:
-->

1. Haga clic en **Manage Microsoft App ID and password** (Administrar id. y contraseña de aplicación de Microsoft).  
![Configuración](./media/upgrade/manage-app-id.png)

2. Haga clic en **Generate New Password** (Generar nueva contraseña).  
![Generación de nueva contraseña](./media/upgrade/generate-new-password.png)

3. Copie y guarde la nueva contraseña junto con el identificador de aplicación de MSA; necesitará estos valores en el futuro.  
![Contraseña nueva](./media/upgrade/new-password-generated.png)

Otro método para recuperar el **identificador de aplicación de Microsoft y la contraseña** puede realizarse siguiendo estas [instrucciones](https://blog.botframework.com/2018/07/03/find-your-azure-bots-appid-and-appsecret/).

<!-- TODO: These steps are no longer valid. AppID will always be generated, confirmed with Support Engineers
### Case 2: App ID field is empty

If the **App ID** field is empty, complete these steps:

1. Click **Create Microsoft App ID and password**.  
   ![Create App ID and password](~/media/upgrade/generate-appid-and-password.png)
   > [!IMPORTANT]
   > Do not select the **Version 3.0** radio button yet. You will do this later, after you have [updated your bot code](#update-code).</div>

2. Click **Generate a password to continue**.  
   ![Generate app password](~/media/upgrade/generate-a-password-to-continue.png)

3. Copy and save the new password along with the MSA App Id; you will need these values in the future.  
   ![New password](~/media/upgrade/new-password-generated.png)

4. Click **Finish and go back to Bot Framework**.  
   ![Finish and go back to Portal](~/media/upgrade/finish-and-go-back-to-bot-framework.png)

5. Back on the bot settings page in the Bot Framework Portal, scroll to the bottom of the page and click **Save changes**.  
   ![Save changes](~/media/upgrade/save-changes.png)
-->

## <a id="update-code"></a> Paso 2: Actualización del código del bot a la versión 4.0

Los bots de la versión 1 ya no son compatibles. Para actualizar el bot, deberá crear uno nuevo en la versión 3 en su lugar. Si desea conservar alguno de sus códigos antiguos, tendrá que migrar el código manualmente.

La solución más sencilla consiste en volver a crear el bot con el nuevo [SDK](https://docs.microsoft.com/en-us/azure/bot-service/?view=azure-bot-service-4.0) e implementarlo. 

Si desea conservar el código anterior, siga estos pasos:

1. Cree una aplicación de bot.
2. Copie el código antiguo en la nueva aplicación de bot.
3. Actualice el SDK a la versión más reciente mediante el administrador de paquetes Nuget.
4. Corrija los errores que aparecen y haga referencia al nuevo [SDK](https://docs.microsoft.com/en-us/azure/bot-service/?view=azure-bot-service-4.0).
5. Implemente el bot en Azure siguiendo estas [instrucciones](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-deploy-azure?view=azure-bot-service-4.0).

<!-- TODO: Remove outdated code 
To update your bot code to version 3.0, complete these steps:

1. Update to the latest version of the [Bot Framework SDK](https://github.com/Microsoft/BotBuilder) for your bot's language.
2. Update your code to apply the necessary changes, according the guidance below.
3. Use the [Bot Framework Emulator](~/bot-service-debug-emulator.md) to test your bot locally and then in the cloud.

The following sections describe the key differences between API v1 and API v3. After you have updated your code to API v3, you can finish the upgrade process by [updating your bot settings](#step-3) in the Bot Framework Portal.
-->

### <a name="botbuilder-and-connector-are-now-one-sdk"></a>BotBuilder y Connector son ahora un SDK

En lugar de tener que instalar distintos SDK para Builder y Connector mediante varios paquetes NuGet (o módulos NPM), ahora puede obtener ambas bibliotecas en un único Bot Framework SDK:

- Bot Framework SDK para .NET: Paquete NuGet `Microsoft.Bot.Builder`
- Bot Framework SDK para Node.js: Módulo NPM `botbuilder`

El SDK independiente `Microsoft.Bot.Connector` está ahora obsoleto y ya no es objeto de mantenimiento.

### <a name="message-is-now-activity"></a>Message es ahora Activity

El objeto `Message` se ha reemplazado por el objeto `Activity` en API v3. El tipo de actividad más común es **message**, pero hay otros tipos de actividades que se pueden usar para comunicar distintos tipos de información a un bot o canal. Para más información sobre los mensajes, consulte [Creación de mensajes](~/dotnet/bot-builder-dotnet-create-messages.md) y [Envío y recepción de actividades](~/dotnet/bot-builder-dotnet-connector.md).

### <a name="activity-types--events"></a>Eventos y tipos de actividad

Algunos eventos han cambiado de nombre o se han refactorizado en API v3. Además, se ha agregado una nueva enumeración `ActivityTypes` al conector para eliminar la necesidad de recordar tipos de actividad específicos.

- El tipo de actividad `conversationUpdate` reemplaza Bot/Usuario Agregado/Quitado A/De Conversación por un único método.
- El nuevo tipo de actividad `typing` permite a su bot indicar que está compilando una respuesta y saber cuándo el usuario está escribiendo una respuesta.
- El nuevo tipo de actividad `contactRelationUpdate` permite a su bot saber si se ha agregado a la lista de contactos del usuario o quitado de esta.

Cuando el bot recibe una actividad `conversationUpdate`, la propiedad `MembersRemoved` y la propiedad `MembersAdded` indican quién se agregó a la conversación o se quitó de esta. Cuando el bot recibe una actividad `contactRelationUpdate`, la propiedad `Action` indica si el usuario agregó el bot a la lista de contactos o lo quitó de esta. Para más información sobre los tipos de actividad, consulte [Introducción a las actividades](~/dotnet/bot-builder-dotnet-activities.md).

### <a name="addressing-messages"></a>Direccionamiento de mensajes

El lugar donde se especifica la información de remitente, destinatario y canal dentro de un mensaje ha cambiado ligeramente en API v3:

|Campo de API v1 | Campo de API v3|
|--------|--------|
| Objeto `From` | Objeto `From` |
| Objeto `To` | Objeto `Recipient` |
| Propiedad `ChannelConversationID` | Objeto `Conversation`|
| Propiedad `ChannelId` | Propiedad `ChannelId` |

Para más información sobre el direccionamiento de mensajes, consulte [Envío y recepción de actividades](~/dotnet/bot-builder-dotnet-connector.md).

### <a name="sending-replies"></a>Envío de respuestas

En Bot Framework API v3, todas las respuestas al usuario se enviarán de manera asincrónica mediante una solicitud HTTP iniciada por separado, en lugar de en línea con HTTP POST para el mensaje entrante al bot. Dado que no se devuelve ningún mensaje en línea al usuario a través de Connector, el tipo de valor devuelto del método POST de su bot será `HttpResponseMessage`. Esto significa que el bot no "devuelve" de manera sincrónica la cadena que desea enviar al usuario, sino que envía un mensaje de respuesta en cualquier punto del código en lugar de tener que contestar en respuesta al POST entrante. Ambos métodos enviarán un mensaje a una conversación:

- `SendToConversation`
- `ReplyToConversation`

El método `SendToConversation` anexará el mensaje especificado al final de la conversación, mientras que el método `ReplyToConversation` agregará (si las conversaciones lo admiten) el mensaje especificado como una respuesta directa a un mensaje anterior en la conversación. Para más información sobre estos métodos, consulte [Envío y recepción de actividades](~/dotnet/bot-builder-dotnet-connector.md).

### <a name="starting-conversations"></a>Inicio de conversaciones

En Bot Framework API v3, puede iniciar una conversación bien con el nuevo método `CreateDirectConversation` para iniciar una conversación privada con un único usuario, o con el nuevo método `CreateConversation` para iniciar una conversación en grupo con varios usuarios. Para más información sobre cómo iniciar conversaciones, consulte [Envío y recepción de actividades](~/dotnet/bot-builder-dotnet-connector.md#start-a-conversation).

### <a name="attachments-and-options"></a>Opciones y datos adjuntos

Bot Framework API v3 introduce una implementación más sólida de datos adjuntos y tarjetas. El tipo `Options` ya no se admite en API v3 y en su lugar se ha reemplazado por las tarjetas. Para más información sobre cómo agregar datos adjuntos a mensajes mediante. NET, consulte [Incorporación de datos adjuntos de elementos multimedia a mensajes](~/dotnet/bot-builder-dotnet-add-media-attachments.md) e [Incorporación de tarjetas enriquecidas a mensajes](~/dotnet/bot-builder-dotnet-add-rich-card-attachments.md).

### <a name="bot-data-storage-bot-state"></a>Almacenamiento de datos de bot (estado del bot)

En Bot Framework API v1, la API para administrar los datos de estado del bot se incluyó en la API de mensajería. En Bot Framework API v3, estas API son distintas. Ahora, debe usar el servicio Bot State para obtener datos de estado (en lugar de suponer que se incluirán en el objeto `Message`) y para almacenar datos de estado (en lugar de pasarlos como parte del objeto `Message`). Para información sobre cómo administrar los datos de estado del bot mediante el servicio Bot State, consulte [Administración de datos de estado](~/dotnet/bot-builder-dotnet-state.md).

> [!IMPORTANT]
> No se recomienda State Service API de Bot Framework para entornos de producción y es posible que se encuentre en desuso en una versión futura. Se recomienda actualizar el código del bot para que use el almacenamiento en memoria para realizar pruebas o usar una de las **extensiones de Azure** para bots de producción. Para más información, consulte el tema **Administración de datos de estado** para la implementación de [.NET](~/dotnet/bot-builder-dotnet-state.md) o [Node](~/nodejs/bot-builder-nodejs-state.md).

### <a name="webconfig-changes"></a>Cambios de Web.config

Bot Framework API v1 almacenaba las propiedades de autenticación con estas claves en **Web.Config**:

- `AppID`
- `AppSecret`

Bot Framework API v3 almacena las propiedades de autenticación con estas claves en **Web.Config**:

- `MicrosoftAppID`
- `MicrosoftAppPassword`

## <a id="step-3"></a> Paso 3: Implementación del bot actualizado en Azure.

Después de haber actualizado el código del bot a API v3, basta con implementar el bot en Azure mediante estas [instrucciones](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-deploy-azure?view=azure-bot-service-4.0). Como la versión 1 ya no es compatible, todos los bots utilizarán automáticamente la API de la versión 3 cuando se implemente en los servicios Azure.

<!-- TODO: Documentation set for removal 
1. Sign in to the [Bot Framework Portal](https://dev.botframework.com/).

2. Click **My bots** and select your bot to open its dashboard. 

3. Click the **SETTINGS** link that is located near the top-right corner of the page. 

4. Under **Version 3.0** within the **Configuration** section, paste your bot's endpoint into the **Messaging endpoint** field.  
![Version 3 configuration](~/media/upgrade/paste-new-v3-enpoint-url.png)

5. Select the **Version 3.0** radio button.  
![Select version 3.0](~/media/upgrade/switch-to-v3-endpoint.png)

6. Scroll to the bottom of the page and click **Save changes**.  
![Save changes](~/media/upgrade/save-changes.png)
-->
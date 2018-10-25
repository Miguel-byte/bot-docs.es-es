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
ms.openlocfilehash: 6ee7120536d42257dde2ed1411df32d807268e33
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000024"
---
# <a name="upgrade-your-bot-to-bot-framework-api-v3"></a>Actualización de un bot a Bot Framework API v3

En el evento Build 2016, Microsoft anunció Microsoft Bot Framework y su iteración inicial de Bot Connector API, junto con los SDK de Bot Builder y Bot Connector. Desde entonces, hemos estado recopilando sus comentarios y trabajando activamente para mejorar la API REST y los SDK.

En julio de 2016, se lanzó Bot Framework API v3 y Bot Framework API v1 ha quedado en desuso. Los bots que usan API v1 han dejado de funcionar en Skype en diciembre de 2016 y en todos los canales restantes el 23 de febrero de 2017. Si ha creado un bot con API v1 y quiere que funcione de nuevo, debe actualizar a la API v3 siguiendo las instrucciones de este artículo. Para asegurarse de que comprende el proceso de actualización de fin a fin, lea este artículo completamente antes de comenzar. 

## <a name="step-1-get-your-app-id-and-password-from-the-bot-framework-portal"></a>Paso 1: Obtención del identificador y la contraseña de la aplicación del portal de Bot Framework

Inicie sesión en el [portal de Bot Framework](https://dev.botframework.com/), haga clic en **My bots** (Mis bots) y seleccione su bot para abrir su panel. A continuación, haga clic en el vínculo **SETTINGS** (CONFIGURACIÓN) que se encuentra cerca de la esquina superior derecha de la página. 

En la sección **Configuration** (Configuración) de la página de configuración, examine el contenido del campo **App ID** (Id. de aplicación) y continúe con los pasos siguientes según si este campo está o no ya rellenado.

### <a name="case-1-app-id-field-is-already-populated"></a>Caso 1: El campo de identificador de aplicación está ya rellenado

Si el campo **App ID** (Id. de aplicación) está ya rellenado, realice estos pasos:

1. Haga clic en **Manage Microsoft App ID and password** (Administrar id. y contraseña de aplicación de Microsoft).  
![Configuración](~/media/upgrade/manage-app-id.png)

2. Haga clic en **Generate New Password** (Generar nueva contraseña).  
![Generación de nueva contraseña](~/media/upgrade/generate-new-password.png)

3. Copie y guarde la nueva contraseña junto con el identificador de aplicación de MSA; necesitará estos valores en el futuro.  
![Contraseña nueva](~/media/upgrade/new-password-generated.png)

### <a name="case-2-app-id-field-is-empty"></a>Caso 2: El campo de identificador de aplicación está vacío

Si el campo **App ID** (Id. de aplicación) está vacío, realice estos pasos:

1. Haga clic en **Create Microsoft App ID and password** (Crear id. y contraseña de aplicación de Microsoft).  
   ![Creación de identificador y contraseña de aplicación](~/media/upgrade/generate-appid-and-password.png)
   > [!IMPORTANT]
   > No seleccione aún el botón de radio **Versión 3.0** (Versión 3.0). Lo hará más adelante, una vez que haya [actualizado su código de bot](#update-code).</div>

2. Haga clic en **Generate a password to continue** (Generar una contraseña para continuar).  
   ![Generación de la contraseña de aplicación](~/media/upgrade/generate-a-password-to-continue.png)

3. Copie y guarde la nueva contraseña junto con el identificador de aplicación de MSA; necesitará estos valores en el futuro.  
   ![Contraseña nueva](~/media/upgrade/new-password-generated.png)

4. Haga clic en **Finish and go back to Bot Framework** (Finalizar y volver a Bot Framework).  
   ![Finalización y vuelta al portal](~/media/upgrade/finish-and-go-back-to-bot-framework.png)

5. De nuevo en la página de configuración del bot en el portal de Bot Framework, desplácese al final de la página y haga clic en **Save changes** (Guardar cambios).  
   ![Guardar cambios](~/media/upgrade/save-changes.png)

## <a id="update-code"></a> Paso 2: Actualización del código de bot a la versión 3.0

Para actualizar el código de bot a la versión 3.0, siga estos pasos:

1. Actualice a la versión más reciente del [SDK de Bot Builder](https://github.com/Microsoft/BotBuilder) correspondiente al lenguaje de su bot.
2. Actualice el código para aplicar los cambios necesarios, según las instrucciones siguientes.
3. Use [Bot Framework Emulator](~/bot-service-debug-emulator.md) para probar el bot localmente y luego en la nube.

En las secciones siguientes se describen las principales diferencias entre API v1 y API v3. Después de haber actualizado el código a API v3, [actualice la configuración de bot](#step-3) en el portal de Bot Framework para finalizar el proceso de actualización.

### <a name="botbuilder-and-connector-are-now-one-sdk"></a>BotBuilder y Connector son ahora un SDK

En lugar de tener que instalar distintos SDK para Builder y Connector mediante varios paquetes NuGet (o módulos NPM), ahora puede obtener ambas bibliotecas en un único SDK de Bot Builder:

- SDK de Bot Builder para. NET: paquete NuGet `Microsoft.Bot.Builder`
- SDK de Bot Builder para Node.js: módulo de NPM `botbuilder`

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

## <a id="step-3"></a> Paso 3: Actualización de la configuración del bot en el portal de Bot Framework

Después de haber actualizado el código de bot a API v3 y haberlo implementado en la nube, realice los siguientes pasos para finalizar el proceso de actualización: 

1. Inicie sesión en el [portal de Bot Framework](https://dev.botframework.com/).

2. Haga clic en **My bots** (Mis bots) y seleccione el bot para abrir su panel. 

3. Haga clic en el vínculo **SETTINGS** (CONFIGURACIÓN) que se encuentra cerca de la esquina superior derecha de la página. 

4. En **Version 3.0** (Versión 3.0) dentro de la sección **Configuration** (Configuración), pegue el punto de conexión del bot en el campo **Messaging endpoint** (Punto de conexión de mensajería).  
![Configuración de la versión 3](~/media/upgrade/paste-new-v3-enpoint-url.png)

5. Seleccione el botón de radio **Versión 3.0** (Versión 3.0).  
![Selección de la versión 3.0](~/media/upgrade/switch-to-v3-endpoint.png)

6. Desplácese a la parte inferior de la página y haga clic en **Save changes** (Guardar cambios).  
![Guardar cambios](~/media/upgrade/save-changes.png)
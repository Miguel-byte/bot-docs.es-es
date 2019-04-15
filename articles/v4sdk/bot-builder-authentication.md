---
title: Incorporación de autenticación al bot mediante Azure Bot Service | Microsoft Docs
description: Obtenga información sobre cómo usar las características de autenticación de Azure Bot Service para agregar el inicio de sesión único al bot.
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 04/09/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 27c97d257261a6f3b9d867503aee40382b685e20
ms.sourcegitcommit: 562dd44e38abacaa31427da5675da556a970cf11
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/10/2019
ms.locfileid: "59477108"
---
# <a name="add-authentication-to-your-bot-via-azure-bot-service"></a>Incorporación de autenticación al bot mediante Azure Bot Service

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Azure Bot Service y el SDK v4 incluyen nuevas funciones de autenticación de bot, que proporcionan características para facilitar el desarrollo de un bot que autentica a los usuarios en varios proveedores de identidades como Azure AD (Azure Active Directory), GitHub, Uber y así sucesivamente. Estas actualizaciones también pueden mejorar la experiencia del usuario porque eliminan la _comprobación del código mágico_ para algunos clientes.

Antes de esto, era necesario que el bot incluyera controladores de OAuth y vínculos de inicio de sesión, que almacenara los identificadores de cliente de destino y los secretos, y que realizara la administración de los tokens de usuario. El bot pediría al usuario que inicie sesión en un sitio web que, a continuación, generaría un _código mágico_ que el usuario podría utilizar para comprobar su identidad.

Ahora, ya no es necesario que los desarrolladores de bots hospeden controladores de OAuth o administren el ciclo de vida de los tokens, ya que todo esto se puede realizar mediante Azure Bot Service.

Las características incluyen:

- Mejoras en los canales para admitir características de autenticación nuevas, como las bibliotecas WebChat y DirectLineJS nuevas para eliminar la necesidad de la comprobación de código mágico de seis dígitos.
- Mejoras en Azure Portal para agregar, eliminar y configurar opciones de conexión a distintos proveedores de identidades de OAuth.
- Compatibilidad con una variedad de proveedores de identidades estándar como Azure AD (los puntos de conexión v1 y v2), GitHub y otros.
- Actualizaciones en Bot Framework SDK para C# y Node.js con el fin de poder recuperar los tokens, crear OAuthCards y controlar eventos de TokenResponse.
- Ejemplos de cómo crear un bot que se autentique en Azure AD.

Puede extrapolar a partir de los pasos descritos en este artículo para agregar estas características a un bot existente. Estos bots de ejemplo muestran las nuevas características de autenticación.

> [!NOTE]
> Las características de autenticación también funcionan con BotBuilder v3. Sin embargo, en este artículo solo se describe código de ejemplo de v4.

### <a name="about-this-sample"></a>Acerca de este ejemplo

Deberá crear un recurso de Azure Bot Service y una nueva aplicación de Azure AD (v1 o v2) para que el bot pueda acceder a Office 365. El recurso de bot registra las credenciales del bot; necesita estas credenciales para probar las características de autenticación, aunque ejecute el código del bot localmente.

> [!IMPORTANT]
> Cuando se registra un bot en Azure, se le asigna una aplicación de Azure AD. Sin embargo, esta aplicación protege el acceso del canal al bot.
Necesita una aplicación AAD adicional para cada aplicación que desee que el bot autentique en nombre del usuario.

En este artículo se crea un bot de ejemplo que se conecta a Microsoft Graph mediante un token de Azure AD v1 o v2. También se describe cómo crear y registrar la aplicación de Azure AD asociada. Como parte de este proceso, deberá usar código del repositorio [Microsoft-Samples/BotBuilder](https://github.com/Microsoft/BotBuilder-Samples) de GitHub. En este artículo se detallan estos procesos.

- **Creación de un recurso de bot**
- **Creación de una aplicación de Azure AD**
- **Registro de la aplicación de Azure AD con el bot**
- **Preparar el código de ejemplo del bot**

Cuando haya terminado, tendrá un bot ejecutándose localmente que puede responder a una serie de tareas simples en una aplicación de Azure AD, como las de comprobar y enviar un correo electrónico, o bien mostrar quién es usted y quién es el administrador. Para ello, el bot usará un token de una aplicación de Azure AD con la biblioteca Microsoft.Graph. No es necesario publicar su bot para probar las características de inicio de sesión de OAuth; sin embargo, su bot necesitará un identificador y una contraseña de aplicación de Azure válidos.

Estas características de autenticación funcionan también con otros tipos de bots. Sin embargo, en este tutorial se usa un bot solo de registro.

### <a name="web-chat-and-direct-line-considerations"></a>Consideraciones de Web Chat y Direct Line

<!-- Summarized from: https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/ -->

Hay un par de problemas de seguridad importantes que hay que tener en cuenta cuando se usa la autenticación de Azure Bot Service con Web Chat.

1. Evite la suplantación, por la que un atacante hace creer a un bot que es otra persona. En Web Chat, un atacante puede suplantar a otra persona cambiando el identificador de usuario de su instancia de Web Chat.

    Para evitarlo, cree un identificador de usuario que no se pueda adivinar. Al habilitar las opciones de autenticación mejoradas en el canal Direct Line, Azure Bot Service puede detectar y rechazar cualquier cambio de identificador de usuario. En los mensajes de Direct Line al bot, el identificador de usuario siempre será igual al identificador con el que inicializó Web Chat. Tenga en cuenta que esta característica requiere que el identificador de usuario empiece por `dl_`.

1. Asegúrese de que el usuario correcto ha iniciado sesión. El usuario tiene dos identidades: la identidad en un canal y la identidad con el proveedor de identidades. En Web Chat, Azure Bot Service puede garantizar que el proceso de inicio de sesión se ha completado en la misma sesión de explorador que Web Chat.

    Para habilitar esta protección, inicie Web Chat con un token de Direct Line que contiene una lista de dominios de confianza que pueden hospedar el cliente de Web Chat del bot. A continuación, especifique estáticamente la lista de dominios de confianza (origen) en la página de configuración de Direct Line.

Use el punto de conexión REST `/v3/directline/tokens/generate` de Direct Line para generar un token para la conversación y especifique el identificador de usuario en la carga de solicitud. Para ver un ejemplo de código, consulte la entrada de blog [Enhanced Direct Line Authentication Features](https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/) (Características de autenticación mejoradas de Direct Line).

<!-- The eventual article about this should talk about the tokens/generate endpoint and its parameters: user, trustedOrigins, and [maybe] eTag.
Sample payload
{
  "user": {
    "id": "string",
    "name": "string",
    "aadObjectId": "string",
    "role": "string"
  },
  "trustedOrigins": [
    "string"
  ],
  "eTag": "string"
}
 -->

## <a name="prerequisites"></a>Requisitos previos

- Conocimientos de los [conceptos básicos de los bots][concept-basics] y de la [administración del estado][concept-state].
- Conocimientos de desarrollo de Azure y OAuth 2.0.
- Visual Studio 2017 o versiones posteriores, Node.js, npm y git.
- Uno de estos ejemplos.

| Muestra | Versión de BotBuilder | Muestra |
|:---|:---:|:---|
| **Autenticación de bots** en [**CSharp**][cs-auth-sample] o [**JavaScript**][js-auth-sample] | v4 | Compatibilidad con OAuthCard |
| **Autenticación de bots de MSGraph** en [**CSharp**][cs-msgraph-sample] o [**JavaScript**][js-msgraph-sample] | v4 |  Compatibilidad de Microsoft Graph API con OAuth 2 |

## <a name="create-your-bot-resource-on-azure"></a>Creación de un recurso de bot en Azure

Cree un **Registro de canales de bot** mediante [Azure Portal](https://portal.azure.com/).

## <a name="create-and-register-an-azure-ad-application"></a>Creación y registro de una aplicación de Azure AD

Necesita una aplicación de Azure AD que el bot pueda usar para conectarse a Microsoft Graph API.

Para este bot se pueden usar puntos de conexión v1 o v2 de Azure AD.
Para obtener información sobre las diferencias entre los puntos de conexión v1 y v2, vea la [comparación entre v1 y v2](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare) y la [introducción al punto de conexión v2.0 de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview).

### <a name="create-your-azure-ad-application"></a>Creación de una aplicación de Azure AD

Siga estos pasos para crear una aplicación de Azure AD. Puede usar los puntos de conexión v1 o v2 con la aplicación que cree.

> [!TIP]
> Deberá crear y registrar la aplicación de Azure AD en un inquilino en el que tenga derechos de administrador.

1. Abra el panel [Azure Active Directory][azure-aad-blade] en Azure Portal.
    Si no está en el inquilino correcto, haga clic en **Cambiar directorio** para cambiar el inquilino. (Para obtener instrucciones sobre cómo crear un inquilino, consulte [Acceso al portal y creación de un inquilino](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-access-create-new-tenant)).
1. Abra el panel **Registros de aplicaciones**.
1. En el panel **Registros de aplicaciones**, haga clic en **Nuevo registro de aplicaciones**.
1. Rellene los campos obligatorios y cree el registro de aplicaciones.

   1. Asigne un nombre a la aplicación.
   1. Establezca **Tipo de aplicación** en **Aplicación web o API**.
   1. Establezca **URL de inicio de sesión** en `https://token.botframework.com/.auth/web/redirect`.
   1. Haga clic en **Create**(Crear).

      - Una vez creada, se muestra en un panel de **Aplicación registrada**.
      - Anote el **Id. de aplicación**. Usará este valor más adelante como _Id. de cliente_ al registrar la aplicación de Azure AD con el bot.

1. Haga clic en **Configuración** para configurar la aplicación.
1. Haga clic en **Claves** para abrir el panel **Claves**.

   1. En **Contraseñas**, cree una clave `BotLogin`.
   1. Establezca su **Duración** en **Nunca expira**.
   1. Haga clic en **Guardar** y registre el valor de clave. Usará este valor más adelante como _Secreto de cliente_ al registrar la aplicación de Azure AD con el bot.
   1. Cierre el panel **Claves**.

1. Haga clic en **Permisos necesarios** para abrir el panel **Permisos necesarios**.

   1. Haga clic en **Agregar**.
   1. Haga clic en **Seleccionar una API**, seleccione **Microsoft Graph** y haga clic en **Seleccionar**.
   1. Haga clic en **Seleccionar permisos**. Elija los permisos delegados que la aplicación va a utilizar.

      > [!NOTE]
      > Los permisos marcados como **Requiere un administrador** requerirán un usuario y un administrador de inquilinos para iniciar sesión, por lo que evite usarlos para el bot.

      Seleccione los permisos delegados siguientes de Microsoft Graph:
      - Leer los perfiles básicos de todos los usuarios
      - Permite leer el correo del usuario.
      - Enviar correo como un usuario
      - Iniciar sesión y leer el perfil de usuario
      - Ver el perfil básico de los usuarios
      - Ver la dirección de correo electrónico de los usuarios

   1. Haga clic en **Seleccionar** y después en **Listo**.
   1. Cierre el panel **Permisos necesarios**.

Ahora tiene una aplicación v1 de Azure AD configurada.

### <a name="register-your-azure-ad-application-with-your-bot"></a>Registro de la aplicación de Azure AD con el bot

El paso siguiente consiste en registrar la aplicación de Azure AD que acaba de crear.

# [<a name="azure-ad-v1"></a>Azure AD v1](#tab/aadv1)

1. Vaya hasta la página de recursos del bot en [Azure Portal](http://portal.azure.com/).
1. Haga clic en **Configuración**.
1. En **Configuración de conexión de OAuth**, cerca de la parte inferior de la página, haga clic en **Agregar configuración**.
1. Rellene el formulario de la siguiente manera:

    1. Para **Nombre**, escriba un nombre para la conexión. Este nombre se usará en el código del bot.
    1. En **Proveedor de servicios**, seleccione **Azure Active Directory**. Después de seleccionar esta opción, se mostrarán los campos específicos de Azure AD.
    1. Para **Id. de cliente**, escriba el identificador de aplicación que registró para la aplicación v1 de Azure AD.
    1. Para **Secreto de cliente**, escriba la clave que registró para la clave `BotLogin` de la aplicación.
    1. En **Tipo de concesión**, escriba `authorization_code`.
    1. Para **Dirección URL de inicio de sesión**, escriba `https://login.microsoftonline.com`.
    1. Para **Id. de inquilino**, escriba el identificador de inquilino de Azure Active Directory, por ejemplo `microsoft.com` o `common`.

       Este será el inquilino asociado a los usuarios que se pueden autenticar. Para permitir que cualquier usuario se autentique a sí mismo mediante el bot, use el inquilino `common`.

    1. En **URL de recursos**, escriba `https://graph.microsoft.com/`.
    1. Deje **Ámbitos** en blanco.

1. Haga clic en **Save**(Guardar).

> [!NOTE]
> Estos valores permiten que la aplicación acceda a datos de Office 365 a través de Microsoft Graph API.

# [<a name="azure-ad-v2"></a>Azure AD v2](#tab/aadv2)

1. Vaya a la página de registro de canales del bot en [Azure Portal](http://portal.azure.com/).
1. Haga clic en **Configuración**.
1. En **Configuración de conexión de OAuth**, cerca de la parte inferior de la página, haga clic en **Agregar configuración**.
1. Rellene el formulario de la siguiente manera:

    1. Para **Nombre**, escriba un nombre para la conexión. Lo usará en el código del bot.
    1. En **Proveedor de servicios**, seleccione **Azure Active Directory v2**. Después de seleccionar esta opción, se mostrarán los campos específicos de Azure AD.
    1. Para **Id. de cliente**, escriba el identificador de aplicación v2 de Azure AD del registro de la aplicación.
    1. Para **Secreto de cliente**, escriba la contraseña de aplicación v2 de Azure AD del registro de la aplicación.
    1. Para **Id. de inquilino**, escriba el identificador de inquilino de Azure Active Directory, por ejemplo `microsoft.com` o `common`.

        Este será el inquilino asociado a los usuarios que se pueden autenticar. Para permitir que cualquier usuario se autentique a sí mismo mediante el bot, use el inquilino `common`.

    1. En **Ámbitos**, escriba los nombres de los permisos que eligió en el registro de la aplicación: `Mail.Read Mail.Send openid profile User.Read User.ReadBasic.All`.

        > [!NOTE]
        > Para Azure AD v2, **Ámbitos** toma una lista de valores que distingue mayúsculas de minúsculas, separados por espacios.

1. Haga clic en **Save**(Guardar).

> [!NOTE]
> Estos valores permiten que la aplicación acceda a datos de Office 365 a través de Microsoft Graph API.

---

### <a name="test-your-connection"></a>Prueba de la conexión

1. Haga clic en la entrada de la conexión para abrir la conexión que acaba de crear.
1. Haga clic en **Probar conexión** en la parte superior del panel **Configuración de conexión del proveedor de servicios**.
1. La primera vez, se debería abrir una pestaña del explorador nueva con los permisos que solicita la aplicación y en la que se le pide que acepte.
1. Haga clic en **Aceptar**.
1. Esto debería redirigirle a la página **Test Connection to \<<your-connection-name> Succeeded** (Resultado satisfactorio de la prueba de conexión a "<nombreDeLaConexión>").

Ahora puede usar este nombre de conexión en el código del bot para recuperar los tokens de usuario.

## <a name="prepare-the-bot-sample-code"></a>Preparar el código de ejemplo del bot

Dependiendo del ejemplo que ha elegido, trabajará con C# o con Node.

| Muestra | Versión de BotBuilder | Muestra |
|:---|:---:|:---|
| **Autenticación de bots** en [**CSharp**][cs-auth-sample] o [**JavaScript**][js-auth-sample] | v4 | Compatibilidad con OAuthCard |
| **Autenticación de bots de MSGraph** en [**CSharp**][cs-msgraph-sample] o [**JavaScript**][js-msgraph-sample] | v4 |  Compatibilidad de Microsoft Graph API con OAuth 2 |

1. Haga clic en uno de los vínculos de ejemplo anteriores y clone el repositorio de GitHub.
1. Siga las instrucciones de la página Léame de GitHub para saber cómo ejecutar ese bot en particular (C# o Node).
1. Si usa el ejemplo Bot-Authentication para C#:

    1. Establezca la variable `ConnectionName` del archivo `AuthenticationBot.cs` en el valor que ha usado al configurar el valor de conexión de OAuth 2.0 del bot.
    1. Establezca el valor `appId` del archivo `BotConfiguration.bot` en el identificador de aplicación del bot.
    1. Establezca el valor `appPassword` del archivo `BotConfiguration.bot` en el secreto del bot.

1. Si usa el ejemplo Bot-Authentication para Node/JS:

    1. Establezca la variable `CONNECTION_NAME` del archivo `bot.js` en el valor que ha usado al configurar el valor de conexión de OAuth 2.0 del bot.
    1. Establezca el valor `appId` del archivo `bot-authentication.bot` en el identificador de aplicación del bot.
    1. Establezca el valor `appPassword` del archivo `bot-authentication.bot` en el secreto del bot.

    > [!IMPORTANT]
    > En función de los caracteres del secreto, es posible que sea necesario aplicar secuencias de escape XML a la contraseña. Por ejemplo, cualquier símbolo de Y comercial (&) tendrá que codificarse como `&amp;`.

    ```json
    {
        "name": "BotAuthentication",
        "secretKey": "",
        "services": [
            {
            "appId": "",
            "id": "http://localhost:3978/api/messages",
            "type": "endpoint",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages",
            "name": "BotAuthentication"
            }
        ]
    }
    ```

    Si no sabe cómo obtener los valores **Id. de aplicación de Microsoft** y **Contraseña de aplicación de Microsoft**, busque en **Configuración de la aplicación** del Azure App Service que se haya aprovisionado para el bot en Azure Portal.

    > [!NOTE]
    > Ahora podría publicar el código de este bot en la suscripción de Azure (si hace clic con el botón derecho en el proyecto y selecciona **Publicar**), pero para este tutorial no es necesario. Tendría que establecer una configuración de publicación que usara la aplicación y el plan de hospedaje que usó durante la configuración del bot en Azure Portal.

## <a name="use-the-emulator-to-test-your-bot"></a>Usar el emulador para probar el bot

Deberá instalar el [Emulador de bot](https://github.com/Microsoft/BotFramework-Emulator) para probar el bot de forma local. Puede usar el emulador v3 o v4.

1. Inicie el bot (con o sin depuración).
1. Anote el número de puerto de host local de la página. Necesitará esta información para interactuar con el bot.
1. Inicie el emulador.
1. Conéctese al bot. Asegúrese de que la configuración del bot usa el **Identificador de aplicación de Microsoft** y la **contraseña de aplicación de Microsoft** al utilizar la autenticación
1. Asegúrese de que en la configuración del emulador **Usar un código de verificación de inicio de sesión para OAuthCards** está activada y **ngrok** está habilitado para que Azure Bot Service pueda devolver el token al emulador cuando esté disponible.

   Si todavía no ha configurado la conexión, proporcione la dirección y el id. de aplicación y la contraseña de Microsoft del bot. Agregue `/api/messages` a la dirección URL del bot. La dirección URL se parecerá a esta `http://localhost:portNumber/api/messages`.

1. Escriba `help` para ver una lista de los comandos disponibles para el bot y probar las características de autenticación.
1. Una vez que haya iniciado sesión, no es necesario que vuelva a proporcionar las credenciales hasta que cierre la sesión.
1. Para cerrar la sesión y cancelar la autenticación, escriba `signout`.

<!--To restart completely from scratch you also need to:
1. Navigate to the **AppData** folder for your account.
1. Go to the **Roaming/botframework-emulator** subfolder.
1. Delete the **Cookies** and **Coolies-journal** files.
-->

> [!NOTE]
> La autenticación del bot requiere el uso de Bot Connector Service. El servicio accede a la información de registro de los canales del bot, motivo por el que es necesario establecer el punto de conexión de mensajería del bot en el portal. La autenticación también requiere el uso de HTTPS, por lo que fue necesario crear una dirección de reenvío HTTPS para el bot que se ejecuta localmente.

<!--The following is necessary for WebChat:
1. Use the **ngrok** command-line tool to get a forwarding HTTPS address for your bot.
   - For information on how to do this, see [Debug any Channel locally using ngrok](https://blog.botframework.com/2017/10/19/debug-channel-locally-using-ngrok/).
   - Any time you exit **ngrok**, you will need to redo this and the following step before starting the Emulator.
1. On the Azure Portal, go to the **Settings** blade for your bot.
   1. In the **Configuration** section, change the **Messaging endpoint** to the HTTPS forwarding address generated by **ngrok**.
   1. Click **Save** to save your change.
-->

## <a name="notes-on-the-token-retrieval-flow"></a>Notas sobre el flujo de recuperación de tokens

Cuando un usuario le pide al bot que haga algo para lo que es necesario que el usuario haya iniciado sesión, el bot puede usar `OAuthPrompt` para iniciar la recuperación de un token para una conexión determinada. `OAuthPrompt` crea un flujo de recuperación de tokens que consta de:

1. Comprobación de que Azure Bot Service ya tiene un token para el usuario y la conexión actuales. Si hay un token, este se devuelve.
1. Si Azure Bot Service no tiene un token almacenado en caché, se crea un elemento `OAuthCard` que es un botón de inicio de sesión en el que el usuario puede hacer clic.
1. Después de que el usuario hace clic en el botón de inicio de sesión `OAuthCard`, Azure Bot Service envía al bot el token del usuario directamente o presenta al usuario un código de autenticación de 6 dígitos para entrar en la ventana de chat.
1. Si se presenta al usuario un código de autenticación, el bot intercambia este código de autenticación por el token del usuario.

El siguiente par de fragmentos de código se toman del elemento `OAuthPrompt` que muestra cómo funcionan estos pasos en el aviso.

### <a name="check-for-a-cached-token"></a>Buscar un token en caché

En este código, en primer lugar el bot realiza una comprobación rápida para determinar si Azure Bot Service ya tiene un token para el usuario (que se identifica mediante el remitente de la actividad actual) y el nombre de la conexión dado (que es el nombre de conexión que se usa en la configuración). Azure Bot Service tendrá ya un token en caché o no. La llamada a GetUserTokenAsync realiza esta comprobación rápida. Si Azure Bot Service tiene un token y lo devuelve, el token se puede usar inmediatamente. Si Azure Bot Service no tiene un token, este método devolverá NULL. En este caso, el bot puede enviar una OAuthCard personalizada para que el usuario inicie sesión.

# [<a name="c"></a>C#](#tab/csharp)

```csharp
// First ask Bot Service if it already has a token for this user
var token = await adapter.GetUserTokenAsync(turnContext, connectionName, null, cancellationToken).ConfigureAwait(false);
if (token != null)
{
    // use the token to do exciting things!
}
else
{
    // If Bot Service does not have a token, send an OAuth card to sign in
}
```

# [<a name="javascript"></a>JavaScript](#tab/javascript)

```javascript
public async getUserToken(context: TurnContext, code?: string): Promise<TokenResponse|undefined> {
    // Get the token and call validator
    const adapter: any = context.adapter as any; // cast to BotFrameworkAdapter
    return await adapter.getUserToken(context, this.settings.connectionName, code);
}
```

---

### <a name="send-an-oauthcard-to-the-user"></a>Envío de OAuthCard al usuario

Puede personalizar la OAuthCard con el texto y el texto de botón que quiera. Las partes importantes son las siguientes:

- Establecer `ContentType` en `OAuthCard.ContentType`.
- Establecer la propiedad `ConnectionName` en el nombre de la conexión que se quiere usar.
- Incluir un botón con un elemento `CardAction` de `Type` `ActionTypes.Signin`; tenga en cuenta que no es necesario especificar ningún valor para el vínculo de inicio de sesión.

Al final de esta llamada, el bot debe "esperar a que el token" regrese. Esta espera tiene lugar en la secuencia de actividad principal porque es posible que el usuario tenga que hacer muchas cosas para iniciar sesión.

# [<a name="c"></a>C#](#tab/csharp)

```csharp
private async Task SendOAuthCardAsync(ITurnContext turnContext, IMessageActivity message, CancellationToken cancellationToken = default(CancellationToken))
{
    if (message.Attachments == null)
    {
        message.Attachments = new List<Attachment>();
    }

    message.Attachments.Add(new Attachment
    {
        ContentType = OAuthCard.ContentType,
        Content = new OAuthCard
        {
            Text = "Please sign in",
            ConnectionName = connectionName,
            Buttons = new[]
            {
                new CardAction
                {
                    Title = "Sign In",
                    Text = "Sign In",
                    Type = ActionTypes.Signin,
                },
            },
        },
    });

    await turnContext.SendActivityAsync(message, cancellationToken).ConfigureAwait(false);
}
```

# [<a name="javascript"></a>JavaScript](#tab/javascript)

```javascript
private async sendOAuthCardAsync(context: TurnContext, prompt?: string|Partial<Activity>): Promise<void> {
    // Initialize outgoing message
    const msg: Partial<Activity> =
        typeof prompt === 'object' ? {...prompt} : MessageFactory.text(prompt, undefined, InputHints.ExpectingInput);
    if (!Array.isArray(msg.attachments)) { msg.attachments = []; }

    const cards: Attachment[] = msg.attachments.filter((a: Attachment) => a.contentType === CardFactory.contentTypes.oauthCard);
    if (cards.length === 0) {
        // Append oauth card
        msg.attachments.push(CardFactory.oauthCard(
            this.settings.connectionName,
            this.settings.title,
            this.settings.text
        ));
    }

    // Send prompt
    await context.sendActivity(msg);
}
```

---

### <a name="wait-for-a-tokenresponseevent"></a>Esperar a TokenResponseEvent

En este código, el bot espera un elemento `TokenResponseEvent` (a continuación se muestra más información sobre cómo se enruta este elemento a la pila de diálogos). Primero, el método `WaitForToken` determina si este evento se ha enviado. Si se ha enviado, el bot lo puede usar. En caso contrario, el método `RecognizeTokenAsync` toma el texto que se haya enviado al bot y lo pasa a `GetUserTokenAsync`. El motivo es que algunos clientes (como WebChat) no necesitan el código de comprobación y pueden enviar el token directamente en `TokenResponseEvent`. Otros clientes requerirán el código mágico (como Facebook o Slack). Azure Bot Service presentará a estos clientes un código mágico de seis dígitos y pedirá al usuario que lo escriba en la ventana de chat. Aunque no es lo más adecuado, es el comportamiento "de reserva", de modo que si `RecognizeTokenAsync` recibe un código, el bot lo puede enviar a Azure Bot Service y obtener un token. Si también se produce un error en esta llamada, puede decidir si notificar un error, o bien hacer otra cosa. Pero en la mayoría de los casos, el bot tendrá un token de usuario.

Si observa el código de bot de cada ejemplo, verá que las actividades `Event` y `Invoke` también se enrutan a la pila de diálogos.

# [<a name="c"></a>C#](#tab/csharp)

```csharp
// This can be called when the bot receives an Activity after sending an OAuthCard
private async Task<TokenResponse> RecognizeTokenAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (IsTokenResponseEvent(turnContext))
    {
        // The bot received the token directly
        var tokenResponseObject = turnContext.Activity.Value as JObject;
        var token = tokenResponseObject?.ToObject<TokenResponse>();
        return token;
    }
    else if (IsTeamsVerificationInvoke(turnContext))
    {
        var magicCodeObject = turnContext.Activity.Value as JObject;
        var magicCode = magicCodeObject.GetValue("state")?.ToString();

        var token = await adapter.GetUserTokenAsync(turnContext, _settings.ConnectionName, magicCode, cancellationToken).ConfigureAwait(false);
        return token;
    }
    else if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // make sure it's a 6-digit code
        var matched = _magicCodeRegex.Match(turnContext.Activity.Text);
        if (matched.Success)
        {
            var token = await adapter.GetUserTokenAsync(turnContext, _settings.ConnectionName, matched.Value, cancellationToken).ConfigureAwait(false);
            return token;
        }
    }

    return null;
}

private bool IsTokenResponseEvent(ITurnContext turnContext)
{
    var activity = turnContext.Activity;
    return activity.Type == ActivityTypes.Event && activity.Name == "tokens/response";
}

private bool IsTeamsVerificationInvoke(ITurnContext turnContext)
{
    var activity = turnContext.Activity;
    return activity.Type == ActivityTypes.Invoke && activity.Name == "signin/verifyState";
}
```

# [<a name="javascript"></a>JavaScript](#tab/javascript)

```javascript
private async recognizeToken(context: TurnContext): Promise<PromptRecognizerResult<TokenResponse>> {
    let token: TokenResponse|undefined;
    if (this.isTokenResponseEvent(context)) {
        token = context.activity.value as TokenResponse;
    } else if (this.isTeamsVerificationInvoke(context)) {
        const code: any = context.activity.value.state;
        await context.sendActivity({ type: 'invokeResponse', value: { status: 200 }});
        token = await this.getUserToken(context, code);
    } else if (context.activity.type === ActivityTypes.Message) {
        const matched: RegExpExecArray = /(\d{6})/.exec(context.activity.text);
        if (matched && matched.length > 1) {
            token = await this.getUserToken(context, matched[1]);
        }
    }

    return token !== undefined ? { succeeded: true, value: token } : { succeeded: false };
}

private isTokenResponseEvent(context: TurnContext): boolean {
    const activity: Activity = context.activity;
    return activity.type === ActivityTypes.Event && activity.name === 'tokens/response';
}

private isTeamsVerificationInvoke(context: TurnContext): boolean {
    const activity: Activity = context.activity;
    return activity.type === ActivityTypes.Invoke && activity.name === 'signin/verifyState';
}
```

---

### <a name="message-controller"></a>Controlador de mensajes

En las llamadas posteriores al bot, tenga en cuenta que este bot de ejemplo nunca almacena el token en caché. Se debe a que el bot siempre puede pedir el token a Azure Bot Service. Esto evita que el bot tenga que administrar el ciclo de vida del token, actualizar el token, etc., ya Azure Bot Service se encarga de todo esto de forma automática.

### <a name="further-reading"></a>Lecturas adicionales

- [Recursos adicionales de Bot Framework](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help) incluye vínculos para obtener más ayuda.
- El repositorio del [SDK Bot Framework](https://github.com/microsoft/botbuilder) tiene más información sobre repositorios, ejemplos, herramientas y especificaciones relativos al SDK Bot Builder.

<!-- Footnote-style links -->

[Azure portal]: https://ms.portal.azure.com
[azure-aad-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview
[aad-registration-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-dialog]: bot-builder-dialog-manage-conversation-flow.md
[dialog-prompts]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-auth-sample]: https://aka.ms/v4cs-bot-auth-sample
[js-auth-sample]: https://aka.ms/v4js-bot-auth-sample
[cs-msgraph-sample]: https://aka.ms/v4cs-auth-msgraph-sample
[js-msgraph-sample]: https://aka.ms/v4js-auth-msgraph-sample

---
title: Incorporación de autenticación al bot mediante Azure Bot Service | Microsoft Docs
description: Obtenga información sobre cómo usar las características de autenticación de Azure Bot Service para agregar el inicio de sesión único al bot.
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/04/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 165eac6ac134a5807119c7a067b77fb7bc6e3282
ms.sourcegitcommit: 312a4593177840433dfee405335100ce59aac347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/12/2019
ms.locfileid: "73933704"
---
<!-- 

Related TODO:
- Check code in [Web Chat channel](https://docs.microsoft.com/azure/bot-service/bot-service-channel-connect-webchat?view=azure-bot-service-4.0)
- Check guidance in [DirectLine authentication](https://docs.microsoft.com/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-authentication?view=azure-bot-service-4.0)
-->

<!-- General TODO: (Feedback from CSE (Nafis))
- Add note that: token management is based on user ID
- Explain why/how to share existing website authentication with a bot.
- Risk: Even people who use a DirectLine token can be vulnerable to user ID impersonation.
    Docs/samples that show exchange of secret for token don't specify a user ID, so an attacker can impersonate a different user by modifying the ID client side. There's a [blog post](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fblog.botframework.com%2F2018%2F09%2F01%2Fusing-webchat-with-azure-bot-services-authentication%2F&data=02%7C01%7Cv-jofing%40microsoft.com%7Cee005e1c9d2c4f4e7ea508d6b231b422%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C636892323874079713&sdata=b0DWMxHzmwQvg5EJtlqKFDzR7fYKmg10fXma%2B8zGqEI%3D&reserved=0) that shows how to do this properly.
"Major issues":
- This doc is a sample walkthrough, but there's no deeper documentation explaining how the Azure Bot Service is handling tokens. How does the OAuth flow work? Where is it storing my users' access tokens? What's the security and best practices around using it?

"Minor issues":
- AAD v2 steps tell you to add delegated permission scopes during registration, but this shouldn't be necessary in AAD v2 due to dynamic scopes. (Ming, "This is currently necessary because scopes are not exposed through our runtime API. We don’t currently have a way for the developer to specify which scope he wants at runtime.")

- "The scope of the connection setting needs to have both openid and a resource in the Azure AD graph, such as Mail.Read." Unclear if I need to take some action at this point to make happen. Kind of out of context. I'm registering an AAD application in the portal, there's no connection setting
- Does the bot need all of these scopes for the samples? (e.g. "Read all users' basic profiles")
-->

# <a name="add-authentication-to-your-bot-via-azure-bot-service"></a>Incorporación de autenticación al bot mediante Azure Bot Service

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

Azure Bot Service y el SDK v4 incluyen nuevas funciones de autenticación de bot, que proporcionan características para facilitar el desarrollo de un bot que autentica a los usuarios en varios proveedores de identidades como Azure AD (Azure Active Directory), GitHub, Uber y así sucesivamente. Estas actualizaciones también pueden mejorar la experiencia del usuario porque eliminan la _comprobación del código mágico_ para algunos clientes.

Antes de esto, era necesario que el bot incluyera controladores de OAuth y vínculos de inicio de sesión, que almacenara los identificadores de cliente de destino y los secretos, y que realizara la administración de los tokens de usuario. El bot pediría al usuario que inicie sesión en un sitio web que, a continuación, generaría un _código mágico_ que el usuario podría utilizar para comprobar su identidad.

Ahora, ya no es necesario que los desarrolladores de bots hospeden controladores de OAuth o administren el ciclo de vida de los tokens, ya que todo esto se puede realizar mediante Azure Bot Service.

Las características incluyen:

- Mejoras en los canales para admitir características de autenticación nuevas, como las bibliotecas WebChat y DirectLineJS nuevas para eliminar la necesidad de la comprobación de código mágico de seis dígitos.
- Mejoras en Azure Portal para agregar, eliminar y configurar opciones de conexión a distintos proveedores de identidades de OAuth.
- Compatibilidad con una variedad de proveedores de identidades estándar como Azure AD (los puntos de conexión v1 y v2), GitHub y otros.
- Actualizaciones en Bot Framework SDK para C# y Node.js con el fin de poder recuperar los tokens, crear OAuthCards y controlar eventos de TokenResponse.
- Ejemplos de cómo crear un bot que se autentique en Azure AD.

Para más información sobre cómo Azure Bot Service controla la autenticación, consulte [Autenticación de usuarios de una conversación](bot-builder-concept-authentication.md).

Puede extrapolar a partir de los pasos descritos en este artículo para agregar estas características a un bot existente. Estos bots de ejemplo muestran las nuevas características de autenticación.

> [!NOTE]
> Las características de autenticación también funcionan con BotBuilder v3. Sin embargo, en este artículo solo se describe código de ejemplo de v4.

### <a name="about-this-sample"></a>Acerca de este ejemplo

Debe crear un recurso de bot en Azure y, para ello, necesita:

1. Un registro de aplicación de Azure AD para permitir que el bot acceda a un recurso externo, como Office 365.
1. Un recurso de bot independiente. El recurso de bot registra las credenciales del bot. Estas credenciales son necesarias para probar las características de autenticación, aunque ejecute el código del bot localmente.

> [!IMPORTANT]
> Cuando se registra un bot en Azure, se le asigna una aplicación de Azure AD. Sin embargo, esta aplicación protege el acceso del canal al bot. Necesita una aplicación AAD adicional para cada aplicación que desee que el bot autentique en nombre del usuario.

En este artículo se crea un bot de ejemplo que se conecta a Microsoft Graph mediante un token de Azure AD v1 o v2. También se describe cómo crear y registrar la aplicación de Azure AD asociada. Como parte de este proceso, deberá usar código del repositorio [Microsoft-Samples/BotBuilder](https://github.com/Microsoft/BotBuilder-Samples) de GitHub. En este artículo se detallan estos procesos.

- **Creación de un recurso de bot**
- **Creación de una aplicación de Azure AD**
- **Registro de la aplicación de Azure AD con el bot**
- **Preparar el código de ejemplo del bot**

Cuando haya terminado, tendrá un bot ejecutándose localmente que puede responder a una serie de tareas simples en una aplicación de Azure AD, como las de comprobar y enviar un correo electrónico, o bien mostrar quién es usted y quién es el administrador. Para ello, el bot usará un token de una aplicación de Azure AD con la biblioteca Microsoft.Graph. No es necesario publicar su bot para probar las características de inicio de sesión de OAuth; sin embargo, su bot necesitará un identificador y una contraseña de aplicación de Azure válidos.

### <a name="web-chat-and-direct-line-considerations"></a>Consideraciones de Web Chat y Direct Line

<!-- Summarized from: https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/ -->

> [!IMPORTANT]
> Tenga en cuenta estas [consideraciones de seguridad](../rest-api/bot-framework-rest-direct-line-3-0-authentication.md#security-considerations) importantes.

## <a name="prerequisites"></a>Requisitos previos

- Conocimiento sobre [conceptos básicos de los bots][concept-basics], [administración de estado][concept-state], [biblioteca de diálogos][concept-dialogs], [implementación de flujos de conversación secuenciales][simple-dialog] y [reutilización de diálogos][component-dialogs].
- Conocimientos de desarrollo de Azure y OAuth 2.0.
- Visual Studio 2017 o versiones posteriores, Node.js, npm y git.
- Uno de estos ejemplos.

| Muestra | Versión de BotBuilder | Muestra |
|:---|:---:|:---|
| **Autenticación de bots** en [**CSharp**][cs-auth-sample] o [**JavaScript**][js-auth-sample] | v4 | Compatibilidad con OAuthCard |
| **Autenticación de bots de MSGraph** en [**CSharp**][cs-msgraph-sample] o [**JavaScript**][js-msgraph-sample] | v4 |  Compatibilidad de Microsoft Graph API con OAuth 2 |

## <a name="create-your-bot-resource-on-azure"></a>Creación de un recurso de bot en Azure

Cree un **recurso de bot** mediante [Azure Portal](https://portal.azure.com/).

Para más información, consulte [Creación de un bot con Azure Bot Service](./abs-quickstart.md).

## <a name="create-and-register-an-azure-ad-application"></a>Creación y registro de una aplicación de Azure AD

Necesita una aplicación de Azure AD que el bot pueda usar para conectarse a Microsoft Graph API.

Para este bot se pueden usar puntos de conexión v1 o v2 de Azure AD.
Para obtener información sobre las diferencias entre los puntos de conexión v1 y v2, vea la [comparación entre v1 y v2](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare) y la [introducción al punto de conexión v2.0 de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview).

### <a name="create-your-azure-ad-application"></a>Creación de una aplicación de Azure AD

Siga estos pasos para crear una aplicación de Azure AD. Puede usar los puntos de conexión v1 o v2 con la aplicación que cree.

> [!TIP]
> Deberá crear y registrar la aplicación de Azure AD en un inquilino en el que pueda dar su consentimiento para delegar los permisos solicitados por una aplicación.

1. Abra el panel [Azure Active Directory][azure-aad-blade] en Azure Portal.
    Si no está en el inquilino correcto, haga clic en **Cambiar directorio** para cambiar el inquilino. (Para obtener instrucciones sobre cómo crear un inquilino, consulte [Acceso al portal y creación de un inquilino](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant)).
1. Abra el panel **Registros de aplicaciones**.
1. En el panel **Registros de aplicaciones**, haga clic en **Nuevo registro**.
1. Rellene los campos obligatorios y cree el registro de aplicaciones.

   1. Asigne un nombre a la aplicación.
   1. Seleccione los **tipos de cuenta admitidos** para la aplicación. (Cualquiera de estas opciones funcionará con este ejemplo).
   1. Para el **URI de redirección**
       1. Seleccione **Web**.
       1. Establezca la dirección URL en `https://token.botframework.com/.auth/web/redirect`.
   1. Haga clic en **Registrar**.

      - Una vez creada, Azure muestra la página **Información general** de la aplicación.
      - Registre el valor del **identificador de aplicación (cliente)** . Usará este valor más adelante como _Id. de cliente_ al registrar la aplicación de Azure AD con el bot.
      - Registre también el valor del **identificador de directorio (inquilino)** . También lo usará para registrar esta aplicación con el bot.

1. En el panel de navegación, haga clic en **Certificados y secretos** para crear un secreto para la aplicación.

   1. En **Secretos de cliente**, haga clic en **Nuevo secreto de cliente**.
   1. Agregue una descripción para diferenciar a este secreto de otros que puede que tenga que crear para esta aplicación, como `bot login`.
   1. Establezca la opción **Expira** en **Nunca**.
   1. Haga clic en **Agregar**.
   1. Antes de salir de esta página, registre el secreto. Usará este valor más adelante como _Secreto de cliente_ al registrar la aplicación de Azure AD con el bot.

1. En el panel de navegación, haga clic en **Permisos de API** para que se abra el panel **Permisos de API**. Es un procedimiento recomendado establecer explícitamente los permisos de API de la aplicación.

   1. Haga clic en **Agregar un permiso** para que aparezca el panel **Solicitud de permisos de API**.
   1. Para este ejemplo, seleccione **Microsoft APIs** y **Microsoft Graph**.
   1. Elija **Permisos delegados** y asegúrese de que se seleccionan los permisos que necesita. Este ejemplo requiere estos permisos.

      > [!NOTE]
      > Los permisos marcados como **SE NECESITA EL CONSENTIMIENTO DEL ADMINISTRADOR** requerirán un usuario y un administrador de inquilinos para iniciar sesión, por lo que evite usarlos para el bot.

      - **openid**
      - **profile**
      - **Mail.Read**
      - **Mail.Send**
      - **User.Read**
      - **User.ReadBasic.All**

   1. Haga clic en **Agregar permisos**. (La primera vez que un usuario acceda a esta aplicación mediante el bot deberá conceder su consentimiento).

Ahora tiene una aplicación de Azure AD configurada.

### <a name="register-your-azure-ad-application-with-your-bot"></a>Registro de la aplicación de Azure AD con el bot

El paso siguiente consiste en registrar la aplicación de Azure AD que acaba de crear con el bot.

#### <a name="azure-ad-v1"></a>Azure AD v1

1. Vaya hasta la página de recursos del bot en [Azure Portal](https://portal.azure.com/).
1. Haga clic en **Configuración**.
1. En **Configuración de conexión de OAuth**, cerca de la parte inferior de la página, haga clic en **Agregar configuración**.
1. Rellene el formulario de la siguiente manera:

    1. Para **Nombre**, escriba un nombre para la conexión. Este nombre se usará en el código del bot.
    1. En **Proveedor de servicios**, seleccione **Azure Active Directory**. Después de seleccionar esta opción, se mostrarán los campos específicos de Azure AD.
    1. Para **Id. de cliente**, escriba el identificador de aplicación (cliente) que registró para la aplicación v1 de Azure AD.
    1. En **Secreto de cliente**, escriba el secreto que ha creado para conceder al bot acceso a la aplicación de Azure AD.
    1. En **Tipo de concesión**, escriba `authorization_code`.
    1. Para **Dirección URL de inicio de sesión**, escriba `https://login.microsoftonline.com`.
    1. Para el **identificador de inquilino**, escriba el **identificador de directorio (inquilino)** que registró anteriormente para la aplicación de AAD o **common** (común) según los tipos de cuenta admitidos seleccionados al crear la aplicación de AAD. Para decidir qué valor asignar, siga estos criterios:

        - Al crear la aplicación de AAD si seleccionó *Solo las cuentas de este directorio organizativo (solo Microsoft: un solo inquilino)* o *Cuentas en cualquier directorio organizativo (directorio de Microsoft AAD: multiinquilino)* escriba el **identificador de inquilino** que registró anteriormente para la aplicación de AAD.

        - Sin embargo, si seleccionó *Cuentas en cualquier directorio organizativo (cualquier directorio de AAD: multiinquilino y cuentas personales de Microsoft, como Skype, Xbox, Outlook.com)* , escriba la palabra **common** en lugar de un identificador de inquilino. De lo contrario, la aplicación de AAD realizará la verificación en el inquilino cuyo identificador se seleccionó y excluirá las cuentas personales de Microsoft.

       Este será el inquilino asociado a los usuarios que se pueden autenticar.

    1. En **URL de recursos**, escriba `https://graph.microsoft.com/`.
    1. Deje **Ámbitos** en blanco.

1. Haga clic en **Save**(Guardar).

> [!NOTE]
> Estos valores permiten que la aplicación acceda a datos de Office 365 a través de Microsoft Graph API.

#### <a name="azure-ad-v2"></a>Azure AD v2

1. Vaya a la página de registro de canales del bot en [Azure Portal](https://portal.azure.com/).
1. Haga clic en **Configuración**.
1. En **Configuración de conexión de OAuth**, cerca de la parte inferior de la página, haga clic en **Agregar configuración**.
1. Rellene el formulario de la siguiente manera:

    1. Para **Nombre**, escriba un nombre para la conexión. Lo usará en el código del bot.
    1. En **Proveedor de servicios**, seleccione **Azure Active Directory v2**. Después de seleccionar esta opción, se mostrarán los campos específicos de Azure AD.
    1. Para **Id. de cliente**, escriba el identificador de aplicación (cliente) que registró para la aplicación v1 de Azure AD.
    1. En **Secreto de cliente**, escriba el secreto que ha creado para conceder al bot acceso a la aplicación de Azure AD.
    1. Para el **identificador de inquilino**, escriba el **identificador de directorio (inquilino)** que registró anteriormente para la aplicación de AAD o **common** (común) según los tipos de cuenta admitidos seleccionados al crear la aplicación de AAD. Para decidir qué valor asignar, siga estos criterios:

        - Al crear la aplicación de AAD si seleccionó *Solo las cuentas de este directorio organizativo (solo Microsoft: un solo inquilino)* o *Cuentas en cualquier directorio organizativo (directorio de Microsoft AAD: multiinquilino)* escriba el **identificador de inquilino** que registró anteriormente para la aplicación de AAD.

        - Sin embargo, si seleccionó *Cuentas en cualquier directorio organizativo (cualquier directorio de AAD: multiinquilino y cuentas personales de Microsoft, como Skype, Xbox, Outlook.com)* , escriba la palabra **common** en lugar de un identificador de inquilino. De lo contrario, la aplicación de AAD realizará la verificación en el inquilino cuyo identificador se seleccionó y excluirá las cuentas personales de Microsoft.

       Este será el inquilino asociado a los usuarios que se pueden autenticar.

    1. En **Ámbitos**, escriba los nombres de los permisos que eligió en el registro de la aplicación: `Mail.Read Mail.Send openid profile User.Read User.ReadBasic.All`.

        > [!NOTE]
        > Para Azure AD v2, **Ámbitos** toma una lista de valores que distingue mayúsculas de minúsculas, separados por espacios.

1. Haga clic en **Save**(Guardar).

> [!NOTE]
> Estos valores permiten que la aplicación acceda a datos de Office 365 a través de Microsoft Graph API.

### <a name="test-your-connection"></a>Prueba de la conexión

1. Haga clic en la entrada de la conexión para abrir la conexión que acaba de crear.
1. Haga clic en **Probar conexión** en la parte superior del panel **Configuración de conexión del proveedor de servicios**.
1. La primera vez, se debería abrir una pestaña del explorador nueva con los permisos que solicita la aplicación y en la que se le pide que acepte.
1. Haga clic en **Aceptar**.
1. Esto debería redirigirle a la página **Test Connection to \<<your-connection-name> Succeeded** (Resultado satisfactorio de la prueba de conexión a "<nombreDeLaConexión>").

Ahora puede usar este nombre de conexión en el código del bot para recuperar los tokens de usuario.

## <a name="prepare-the-bot-code"></a>Preparación el código del bot

Para completar este proceso, necesitará el id. de la aplicación y la contraseña del bot.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

<!-- TODO: Add guidance (once we have it) on how not to hard-code IDs and ABS auth. -->

1. Clone el ejemplo con el que desee trabajar del repositorio de GitHub: [**Autenticación de bot**][cs-auth-sample] o [**Autenticación de bot MSGraph**][cs-msgraph-sample].
1. Actualice **appsettings.json**:

    - Establezca en `ConnectionName` el nombre del valor de conexión de OAuth que agregó al bot.
    - Establezca en `MicrosoftAppId` y `MicrosoftAppPassword` el id. de la aplicación y secreto de la aplicación del bot.

      En función de los caracteres del secreto del bot, es posible que sea necesario aplicar secuencias de escape XML a la contraseña. Por ejemplo, cualquier símbolo de Y comercial (&) tendrá que codificarse como `&amp;`.

    [!code-json[appsettings](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/appsettings.json)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. Clone el bot del repositorio de GitHub el con el que desee trabajar: [**Autenticación de bot**][js-auth-sample] o [**Autenticación de bot MSGraph**][js-msgraph-sample].
1. Actualice **.env**:

    - Establezca en `connectionName` el nombre del valor de conexión de OAuth que agregó al bot.
    - Establezca en los valores `MicrosoftAppId` y `MicrosoftAppPassword` el id. de la aplicación y secreto de la aplicación del bot.

      En función de los caracteres del secreto del bot, es posible que sea necesario aplicar secuencias de escape XML a la contraseña. Por ejemplo, cualquier símbolo de Y comercial (&) tendrá que codificarse como `&amp;`.

    [!code-txt[.env](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/.env)]

---

Si no sabe cómo obtener su **identificador de aplicación de Microsoft** y la **contraseña de aplicación de Microsoft**, puede crear una nueva contraseña [como se describe aquí](../bot-service-quickstart-registration.md#get-registration-password).

> [!NOTE]
> Este código de bot se puede publicar en una suscripción de Azure (para ello, debe hacer clic en el proyecto y seleccionar **Publicar**), pero para este artículo no es necesario. Tendría que establecer una configuración de publicación que usara la aplicación y el plan de hospedaje que usó durante la configuración del bot en Azure Portal.

## <a name="test-the-bot-using-the-emulator"></a>Prueba del bot en el emulador

Si aún no lo ha hecho, instale [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme). Consulte también [Depuración con el emulador](../bot-service-debug-emulator.md).

<!-- auth config steps -->
Para que el inicio de sesión del bot de ejemplo funcione, debe configurar el emulador tal y como se muestra en [Configuración del emulador para autenticación](../bot-service-debug-emulator.md#configure-the-emulator-for-authentication).

### <a name="testing"></a>Prueba

Después de haber configurado el mecanismo de autenticación, puede realizar las pruebas del bot de ejemplo.  

1. Ejecute el bot de ejemplo localmente en su máquina.
1. Inicie el emulador.
1. Cuando se conecte al bot, tendrá que especificar el identificador de aplicación y la contraseña del bot.
    - El identificador de aplicación y la contraseña se obtienen del registro de la aplicación en Azure. Son los mismos valores asignados a la aplicación de bot en el archivo `appsettings.json` o `.env`. En el emulador, esos valores se asignan en el archivo de configuración o la primera vez que se conecte al bot.
    - Si necesita aplicar secuencias de escape XML a la contraseña en el código de bot, también debe hacerlo aquí.
1. Escriba `help` para ver una lista de los comandos disponibles para el bot y probar las características de autenticación.
1. Una vez que haya iniciado sesión, no es necesario que vuelva a proporcionar las credenciales hasta que cierre la sesión.
1. Para cerrar la sesión y cancelar la autenticación, escriba `logout`.

> [!NOTE]
> La autenticación del bot requiere el uso de Bot Connector Service. El servicio accede a la información de registro de canales del bot.

## <a name="bot-authentication-example"></a>Ejemplo de autenticación de bot

En el ejemplo de **autenticación de un bot**, el diálogo está diseñado para recuperar el token del usuario cuando este haya iniciado sesión.

![Salida de ejemplo](media/how-to-auth/auth-bot-test.png)

## <a name="bot-authentication-msgraph-example"></a>Ejemplo de autenticación de bot MSGraph

En el ejemplo de **autenticación de un bot MSGraph**, el diálogo está diseñado para aceptar uyn conjunto limitado de comandos cuando el usuario haya iniciado sesión.

![Salida de ejemplo](media/how-to-auth/msgraph-bot-test.png)

---

## <a name="additional-information"></a>Información adicional

Cuando un usuario le pide al bot que haga algo para lo que es necesario que el usuario haya iniciado sesión, el bot puede usar `OAuthPrompt` para iniciar la recuperación de un token para una conexión determinada. `OAuthPrompt` crea un flujo de recuperación de tokens que consta de:

1. Comprobación de que Azure Bot Service ya tiene un token para el usuario y la conexión actuales. Si hay un token, este se devuelve.
1. Si Azure Bot Service no tiene un token almacenado en caché, se crea un elemento `OAuthCard` que es un botón de inicio de sesión en el que el usuario puede hacer clic.
1. Después de que el usuario hace clic en el botón de inicio de sesión `OAuthCard`, Azure Bot Service envía al bot el token del usuario directamente o presenta al usuario un código de autenticación de 6 dígitos para entrar en la ventana de chat.
1. Si se presenta al usuario un código de autenticación, el bot intercambia este código de autenticación por el token del usuario.

En las secciones siguientes se describe la forma en que el ejemplo implementa algunas tareas de autenticación comunes.

### <a name="use-an-oauth-prompt-to-sign-the-user-in-and-get-a-token"></a>Use un símbolo del sistema de OAuth para que el usuario inicie sesión y obtener un token

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

![Arquitectura del bot](media/how-to-auth/architecture.png)

<!-- The two authentication samples have nearly identical architecture. Using 18.bot-authentication for the sample code. -->

**Dialogs\MainDialog.cs**

Agregue un símbolo del sistema de OAuth a **MainDialog** en su constructor. En este caso, el valor del nombre de la conexión se recuperó del archivo **appsettings.json**.

[!code-csharp[Add OAuthPrompt](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=23-31)]

En un paso del diálogo, utilice `BeginDialogAsync` para iniciar el símbolo del sistema de OAuth, que pide al usuario que inicie sesión.

- Si el usuario ya ha iniciado sesión, se generará un evento de respuesta de token sin preguntar al usuario.
- En caso contrario, se le pedirá al usuario que inicie sesión. Azure Bot Service envía el evento de respuesta de token después de que el usuario intenta iniciar sesión.

[!code-csharp[Use the OAuthPrompt](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=49)]

En el paso siguiente del diálogo, compruebe la presencia de un token en el resultado del paso anterior. Si no es NULL, significa que el usuario iniciado sesión correctamente.

[!code-csharp[Get the OAuthPrompt result](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=54-56)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

![Arquitectura del bot](media/how-to-auth/architecture-js.png)

**dialogs/mainDialog.js**

Agregue un símbolo del sistema de OAuth a **MainDialog** en su constructor. En este caso, el valor del nombre de la conexión se recuperó del archivo **.env**.

[!code-javascript[Add OAuthPrompt](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=24-29)]

En un paso del diálogo, utilice `beginDialog` para iniciar el símbolo del sistema de OAuth, que pide al usuario que inicie sesión.

- Si el usuario ya ha iniciado sesión, se generará un evento de respuesta de token sin preguntar al usuario.
- En caso contrario, se le pedirá al usuario que inicie sesión. Azure Bot Service envía el evento de respuesta de token después de que el usuario intenta iniciar sesión.

[!code-javascript[Use OAuthPrompt](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=57)]

En el paso siguiente del diálogo, compruebe la presencia de un token en el resultado del paso anterior. Si no es NULL, significa que el usuario iniciado sesión correctamente.

[!code-javascript[Get OAuthPrompt result](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=62-63)]

---

### <a name="wait-for-a-tokenresponseevent"></a>Esperar a TokenResponseEvent

Cuando se inicia un símbolo del sistema de OAuth, espera un evento de respuesta del token del que recuperará el token del usuario.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\AuthBot.cs**

**AuthBot** deriva de `ActivityHandler` y controla explícitamente las actividades del evento de respuesta del token. En este caso, seguimos con el diálogo activo, lo que permite que el símbolo del sistema de OAuth procese el evento y recupere el token.

[!code-csharp[OnTokenResponseEventAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Bots/AuthBot.cs?range=32-38)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/authBot.js**

**AuthBot** deriva de `ActivityHandler` y controla explícitamente las actividades del evento de respuesta del token. En este caso, seguimos con el diálogo activo, lo que permite que el símbolo del sistema de OAuth procese el evento y recupere el token.

[!code-javascript[onTokenResponseEvent](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/bots/authBot.js?range=29-31)]

---

### <a name="log-the-user-out"></a>Cierre de la sesión del usuario

Es recomendable permitir a los usuarios cerrar sesión de forma explícita, en lugar de depender de que se agote el tiempo de espera de la conexión.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\LogoutDialog.cs**

[!code-csharp[Allow logout](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/LogoutDialog.cs?range=44-61&highlight=11)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/logoutDialog.js**

[!code-javascript[Allow logout](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/logoutDialog.js?range=31-42&highlight=7)]

---

### <a name="adding-teams-authentication"></a>Incorporación de la autenticación de Teams

Teams se comporta de forma ligeramente diferente a otros canales en lo que respecta a OAuth y requiere algunos cambios para implementar correctamente la autenticación. Agregaremos código desde el ejemplo de bot de autenticación de Teams ([C#][cs-teams-auth-sample]/[JavaScript][js-teams-auth-sample]).
 
Una diferencia entre otros canales y Teams es que Teams envía una actividad de *invocación* al bot, en lugar de una actividad de *evento*. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)  
**Bots/TeamsBot.cs**  
[!code-csharp[Invoke Activity](~/../botbuilder-samples/samples/csharp_dotnetcore/46.teams-auth/Bots/TeamsBot.cs?range=34-42&highlight=1)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)  
**bots/teamsBot.js**  
[!code-javascript[Invoke Activity](~/../botbuilder-samples/samples/javascript_nodejs/46.teams-auth/bots/teamsBot.js?range=27-32&highlight=3)]

---

Si usa un *símbolo del sistema de OAuth*, esta actividad de invocación se debe reenviar al diálogo. Lo haremos en `TeamsActivityHandler`. Agregue el siguiente código al archivo de diálogo principal. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)  
**Bots/DialogBot.cs**  
[!code-csharp[Dialogs Handler](~/../botbuilder-samples/samples/csharp_dotnetcore/46.teams-auth/Bots/DialogBot.cs?range=19)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)  
**Bots/dialogBot.js**  
[!code-javascript[Dialogs Handler](~/../botbuilder-samples/samples/javascript_nodejs/46.teams-auth/bots/dialogBot.js?range=4-6)]

---
Por último, asegúrese de agregar un archivo `TeamsActivityHandler` adecuado (`TeamsActivityHandler.cs` para los bots de C# y `teamsActivityHandler.js` para bots de JavaScript) en el nivel superior de la carpeta del bot.

También `TeamsActivityHandler` envía actividades de *reacción de mensajes*. Una actividad de reacción de mensaje hace referencia a la actividad original mediante el campo *reply to ID* (responder a id.). Esta actividad también debe ser visible mediante la [fuente de actividades][teams-activity-feed] de Microsoft Teams.

> [!NOTE]
> Debe crear un manifiesto e incluir `token.botframework.com` en la sección `validDomains`. En caso contrario, el botón **Iniciar sesión** de OAuthCard no abrirá la ventana de autenticación. Use [App Studio](https://docs.microsoft.com/microsoftteams/platform/get-started/get-started-app-studio) para generar el manifiesto.

### <a name="further-reading"></a>Lecturas adicionales

- [Recursos adicionales de Bot Framework](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help) incluye vínculos para obtener más ayuda.
- El repositorio del [SDK Bot Framework](https://github.com/microsoft/botbuilder) tiene más información sobre repositorios, ejemplos, herramientas y especificaciones relativos al SDK Bot Builder.

<!-- Footnote-style links -->

[Azure portal]: https://ms.portal.azure.com
[azure-aad-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview
[aad-registration-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredAppsPreview

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
[cs-teams-auth-sample]:https://aka.ms/cs-teams-auth-sample
[js-teams-auth-sample]:https://aka.ms/js-teams-auth-sample
[teams-activity-feed]:[https://aka.ms/teams-activity-feed

---
title: Solución de problemas de autenticación de Bot Framework | Microsoft Docs
description: Aprenda a solucionar errores de autenticación con un bot.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
ms.openlocfilehash: 9e86ea0fb677105be920e031979980baf479e42f
ms.sourcegitcommit: abde9e0468b722892f94caf2029fae165f96092f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/09/2018
ms.locfileid: "48875732"
---
# <a name="troubleshooting-bot-framework-authentication"></a>Solución de problemas de autenticación de Bot Framework

Esta guía puede ayudarle a solucionar problemas de autenticación con el bot evaluando una serie de escenarios para determinar dónde se produce el problema. 

> [!NOTE]
> Para completar todos los pasos descritos en esta guía, debe descargar y usar [Bot Framework Emulator][ Emulator] y debe tener acceso a la configuración de registro del bot en el <a href="https://dev.botframework.com" target="_blank">portal de Bot Framework</a>.

## <a id="PW"></a>Identificador de aplicación y contraseña

La seguridad del bot se configura con el **identificador de aplicación de Microsoft** y la **contraseña de aplicación de Microsoft** que se obtiene al registrar el bot con Bot Framework. Normalmente, estos valores son especificados en el archivo de configuración del bot y se usan para recuperar los tokens de acceso del servicio Cuenta Microsoft. 

Si aún no lo ha hecho, [implemente el bot en Azure](~/bot-builder-howto-deploy-azure.md) para obtener el **id. de la aplicación de Microsoft** y la **contraseña de la aplicación de Microsoft** que pueda usar para la autenticación. 

> [!NOTE]
> Para encontrar los valores de **AppID** y **AppPassword** de un bot que ya se ha implementado, consulte [MicrosoftAppID y MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

## <a name="step-1-disable-security-and-test-on-localhost"></a>Paso 1: Deshabilitar la seguridad y probar en el host local

En este paso, comprobará que su bot es accesible y funcional en el host local cuando la seguridad está deshabilitada. 

> [!WARNING]
> Deshabilitar la seguridad para el bot puede permitir que atacantes desconocidos suplanten a los usuarios. Solo debe implementar el procedimiento siguiente si está trabajando en un entorno de depuración protegido.

### <a id="disable-security-localhost"></a> Deshabilitar la seguridad

Para deshabilitar la seguridad para el bot, edite sus valores de configuración para quitar los valores del identificador de aplicación y la contraseña. 

::: moniker range="azure-bot-service-3.0"

Si usa el SDK de Bot Builder para. NET, edite estos valores en el archivo Web.config: 

```xml
<appSettings>
  <add key="MicrosoftAppId" value="" />
  <add key="MicrosoftAppPassword" value="" />
</appSettings>
```

Si usa Bot Builder SDK para Node.js, edite estos valores (o actualice las variables de entorno correspondientes):

```javascript
var connector = new builder.ChatConnector({
  appId: null,
  appPassword: null
});
```

::: moniker-end

::: moniker range="azure-bot-service-4.0"

Si usa el SDK de Bot Builder para. NET, edite los valores del archivo `appsettings.config`:

```xml
<appSettings>
  <add key="MicrosoftAppId" value="" />
  <add key="MicrosoftAppPassword" value="" />
</appSettings>
```

Si usa Bot Builder SDK para Node.js, edite estos valores (o actualice las variables de entorno correspondientes):

```javascript
const adapter = new BotFrameworkAdapter({
    appId: null,
    appPassword: null
});
```

Si usa el archivo `.bot` para la configuración, puede actualizar `appId` y `appPassword` a `""`.

::: moniker-end

### <a name="test-your-bot-on-localhost"></a>Prueba del bot en el host local 

A continuación, pruebe el bot en el host local mediante Bot Framework Emulator.

1. Inicie el bot en el host local.
2. Inicie Bot Framework Emulator.
3. Conecte el bot usando el emulador.
    - Escriba `http://localhost:port-number/api/messages` en la barra de direcciones del emulador, donde **port-number** coincide con el número de puerto mostrado en el explorador en el que se ejecuta la aplicación. 
    - Asegúrese de dejar vacíos los campos **Microsoft App ID** (Id. de aplicación de Microsoft) y **Microsoft App Password** (Contraseña de aplicación de Microsoft).
    - Haga clic en **Conectar**.
4. Para probar la conectividad al bot, escriba algún texto en el emulador y presione Entrar.

Si el bot responde a la entrada y no hay ningún error en la ventana de chat, ha comprobado que su bot es accesible y funcional en el host local cuando la seguridad está deshabilitada. Siga con el [Paso 2](#step-2).

Si se indica algún error en la ventana de chat, haga clic en él para obtener más información. Algunos problemas comunes son:

* La configuración del emulador especifica un punto de conexión incorrecto para el bot. Asegúrese de que ha incluido el número de puerto adecuado en la dirección URL y la ruta de acceso correcta al final de la dirección URL (p. ej., `/api/messages`).
* La configuración del emulador especifica un punto de conexión de bot que comienza con `https`. En el host local, el punto de conexión debe comenzar por `http`.
* La configuración del emulador especifica un valor para los campos **Microsoft App ID** (Id. de aplicación de Microsoft) o **Microsoft App Password** (Contraseña de la aplicación de Microsoft). Ambos campos deben estar vacíos.
* La seguridad no se ha deshabilitado para el bot. [Compruebe](#disable-security-localhost) que el bot no especifica un valor para el identificador de aplicación o la contraseña.

## <a id="step-2"></a> Paso 2: Comprobar el identificador de aplicación y la contraseña del bot

En este paso, comprobará que el identificador de aplicación y la contraseña que va a usar el bot para la autenticación son válidos. (Si no conoce estos valores, [obténgalos](#PW) ahora). 

> [!WARNING]
> Las siguientes instrucciones deshabilitan la comprobación de SSL para `login.microsoftonline.com`. Realice este procedimiento solo en una red segura y considere la posibilidad de cambiar la contraseña de su aplicación posteriormente.

### <a name="issue-an-http-request-to-the-microsoft-login-service"></a>Emisión de una solicitud HTTP para el servicio de inicio de sesión de Microsoft

Estas instrucciones describen cómo usar [cURL](https://curl.haxx.se/download.html) para emitir una solicitud HTTP para el servicio de inicio de sesión de Microsoft. Puede usar una herramienta alternativa como Postman, solo tiene que comprobar que la solicitud se ajusta al [protocolo de autenticación](~/rest-api/bot-framework-rest-connector-authentication.md) de Bot Framework.

Para comprobar que el identificador de aplicación y la contraseña del bot son válidos, emita la siguiente solicitud con **cURL**, reemplazando `APP_ID` y `APP_PASSWORD` con el identificador de aplicación y la contraseña de su bot.

```cmd
curl -k -X POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token -d "grant_type=client_credentials&client_id=APP_ID&client_secret=APP_PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default"
```

Esta solicitud intenta intercambiar el identificador de aplicación y la contraseña del su bot por un token de acceso. Si la solicitud se realiza correctamente, recibirá una carga JSON que contendrá una propiedad `access_token`, entre otros elementos. 

```json
{"token_type":"Bearer","expires_in":3599,"ext_expires_in":0,"access_token":"eyJ0eXAJKV1Q..."}
```

Si la solicitud se realiza correctamente, ha comprobado que el identificador de aplicación y la contraseña que especificó en la solicitud son válidos. Siga con el [Paso 3](#step-3).

Si recibe un error en respuesta a la solicitud, examine la respuesta para identificar la causa. Si la respuesta indica que el identificador de aplicación o la contraseña no son válidos, [obtenga los valores correctos](#PW) desde el portal de Bot Framework y vuelva a emitir la solicitud con los nuevos valores para confirmar que son válidos. 

## Paso 3: Deshabilitar la seguridad y probar en el host local<a id="step-3"></a>

En este momento, ha comprobado que su bot es accesible y funcional en el host local cuando la seguridad está deshabilitada y ha confirmado que el identificador de aplicación y la contraseña que utilizará el bot para la autenticación son válidos. En este paso, comprobará que su bot es accesible y funcional en el host local cuando la seguridad está habilitada.

### <a id="enable-security-localhost"></a> Habilitar la seguridad

La seguridad del bot se basa en los servicios de Microsoft, incluso cuando se ejecuta en el host local. Para habilitar la seguridad para el bot, edite sus valores de configuración para rellenar el identificador de aplicación y la contraseña con los valores que comprobó en el [Paso 2](#step-2).

Si usa el SDK de Bot Builder para. NET, rellene estos valores en el archivo `.bot` o `appsettings.config`:

```xml
<appSettings>
  <add key="MicrosoftAppId" value="APP_ID" />
  <add key="MicrosoftAppPassword" value="PASSWORD" />
</appSettings>
```

Si usa Bot Builder SDK para Node.js, rellene estos valores (o actualice las variables de entorno correspondientes):

```javascript
var connector = new builder.ChatConnector({
  appId: 'APP_ID',
  appPassword: 'PASSWORD'
});
```

> [!NOTE]
> Para buscar los valores **AppID** y **AppPassword** del bot, consulte [MicrosoftAppID y MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

### <a name="test-your-bot-on-localhost"></a>Prueba del bot en el host local 

A continuación, pruebe el bot en el host local mediante Bot Framework Emulator.

1. Inicie el bot en el host local.
2. Inicie Bot Framework Emulator.
3. Conecte el bot usando el emulador.
    - Escriba `http://localhost:port-number/api/messages` en la barra de direcciones del emulador, donde **port-number** coincide con el número de puerto mostrado en el explorador en el que se ejecuta la aplicación. 
    - Escriba el identificador de la aplicación del bot en el campo **Microsoft App ID** (Id. de aplicación de Microsoft).
    - Escriba la contraseña del bot en el campo **Microsoft App Password** (Contraseña de aplicación de Microsoft).
    - Haga clic en **Conectar**.
4. Para probar la conectividad al bot, escriba algún texto en el emulador y presione Entrar.

Si el bot responde a la entrada y no hay ningún error en la ventana de chat, ha comprobado que su bot es accesible y funcional en el host local cuando la seguridad está habilitada.  Siga con el [Paso 4](#step-4).

Si se indica algún error en la ventana de chat, haga clic en él para obtener más información. Algunos problemas comunes son:

* La configuración del emulador especifica un punto de conexión incorrecto para el bot. Asegúrese de que ha incluido el número de puerto adecuado en la dirección URL y la ruta de acceso correcta al final de la dirección URL (p. ej., `/api/messages`).
* La configuración del emulador especifica un punto de conexión de bot que comienza con `https`. En el host local, el punto de conexión debe comenzar por `http`.
* En la configuración del emulador, los campos **Microsoft App ID** (Id. de aplicación de Microsoft) y **Microsoft App Password** (Contraseña de la aplicación de Microsoft) no contienen valores válidos. Se deben rellenar ambos campos y cada uno debe contener el valor correspondiente que comprobó en el [Paso 2](#step-2).
* La seguridad no se ha habilitado para el bot. [Compruebe](#enable-security-localhost) que la configuración del bot especifica valores para el identificador de aplicación y la contraseña.

## Paso 4: Probar el bot en la nube <a id="step-4"></a>

En este momento, ha comprobado que su bot es accesible y funcional en el host local cuando la seguridad está deshabilitada, ha confirmado que el identificador de aplicación y la contraseña del bot son válidos, y ha comprobado que es accesible y funcional en el host local cuando la seguridad está habilitada. En este paso, implementará el bot en la nube y comprobará que es accesible y funcional con la seguridad habilitada. 

### <a name="deploy-your-bot-to-the-cloud"></a>Implementación de un bot en la nube

Bot Framework requiere que los bots sean accesibles desde Internet, por lo que debe implementar el bot en una plataforma con hospedaje en la nube como Azure. Asegúrese de habilitar la seguridad para el bot antes de la implementación, como se describe en el [Paso 3](#step-3).

> [!NOTE]
> Si aún no tiene un proveedor de hospedaje en la nube, puede registrarse para obtener una <a href="https://azure.microsoft.com/en-us/free/" target="_blank">cuenta gratuita</a>. 

Al implementar el bot en Azure, SSL se configurará automáticamente para la aplicación, lo que permite habilitar el punto de conexión de **HTTPS** que Bot Framework requiere. Si realiza la implementación en otro proveedor de hospedaje en la nube, asegúrese de comprobar que la aplicación está configurada para SSL, a fin de que el bot disponga de un punto de conexión **HTTPS**.

### <a name="test-your-bot"></a>Prueba del bot 

Para probar el bot en la nube con la seguridad habilitada, complete los pasos siguientes.

1. Asegúrese de que el bot se ha implementado correctamente y se está ejecutando. 
2. Inicie sesión en el <a href="https://dev.botframework.com" target="_blank">portal de Bot Framework</a>.
3. Haga clic en **My Bots** (Mis bots).
4. Seleccione el bot que quiera probar.
5. Haga clic en **Test** (Prueba) para abrir el bot en un control de charla web incrustado.
6. Para probar la conectividad al bot, escriba en el control de chat web y presione Entrar.

Si se indica un error en la ventana de chat, utilice el mensaje de error para determinar la causa del error. Algunos problemas comunes son: 

* La información de **Messaging endpoint** (Punto de conexión de mensajería) especificada en la página **Settings** (Configuración) para el bot en el portal de Bot Framework es incorrecta. Asegúrese de que ha incluido la ruta de acceso correcta al final de la dirección URL (p. ej., `/api/messages`).
* La información de **Messaging endpoint** (Punto de conexión de mensajería) especificada en la página **Settings** (Configuración) para el bot en el portal de Bot Framework no comienza con `https` o no es de confianza para Bot Framework. El bot debe tener un certificado de confianza en la cadena válido.
* Faltan valores de configuración para el bot o son incorrectos para el identificador de aplicación o la contraseña. [Compruebe](#enable-security-localhost) que los valores de configuración del bot son válidos para el identificador de aplicación y la contraseña.

Si el bot responde correctamente a la entrada, ha comprobado que su bot es accesible y funcional en la nube con la seguridad habilitada. En este momento, el bot está listo para [conectarse a un canal](~/bot-service-manage-channels.md) como Skype, Facebook Messenger, Direct Line y otros de forma segura.

## <a name="additional-resources"></a>Recursos adicionales

Si sigue experimentando problemas después de completar los pasos anteriores, puede:

* [Depurar el bot en la nube](~/bot-service-debug-emulator.md) mediante Bot Framework Emulator y <a href="https://ngrok.com/" target="_blank">ngrok</a>.
* Usar una herramienta de creación de conexiones proxy como [Fiddler](https://www.telerik.com/fiddler) para inspeccionar el tráfico HTTPS a y desde su bot. *Fiddler no es un producto de Microsoft.*
* Revisar la [Guía de autenticación de Bot Connector][ BotConnectorAuthGuide] para obtener información acerca de las tecnologías de autenticación que usa Bot Framework.
* Solicitar ayuda a otros usuarios con los recursos de [soporte técnico][Support] de Bot Framework. 

[BotConnectorAuthGuide]: ~/rest-api/bot-framework-rest-connector-authentication.md
[Support]: bot-service-resources-links-help.md
[Emulator]: bot-service-debug-emulator.md

---
title: Autenticación | Microsoft Docs
description: Aprenda a autenticar solicitudes de API en Direct Line API v3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/22/2019
ms.openlocfilehash: d79cea421e6743c504e3fa68056de71974194923
ms.sourcegitcommit: c200cc2db62dbb46c2a089fb76017cc55bdf26b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2019
ms.locfileid: "70037444"
---
# <a name="authentication"></a>Authentication

Un cliente puede autenticar solicitudes en Direct Line API 3.0 ya sea mediante un **secreto** que se [obtiene de la página de configuración del canal de Direct Line](../bot-service-channel-connect-directline.md) en el portal de Bot Framework, o mediante un **token** que se obtiene en el tiempo de ejecución. El secreto o token debe especificarse en el encabezado `Authorization` de cada solicitud, con este formato: 

```http
Authorization: Bearer SECRET_OR_TOKEN
```

## <a name="secrets-and-tokens"></a>Secretos y tokens

Un **secreto** de Direct Line es una clave maestra que se puede usar para acceder a cualquier conversación que pertenezca al bot asociado. Un **secreto** se puede usar también para obtener un **token**. Los secretos no expiran. 

Un **token** de Direct Line es una clave que se puede usar para acceder a una única conversación. Un token expira, pero se puede actualizar. 

Si va a crear una aplicación de servicio a servicio, puede que el enfoque más sencillo sea especificar el **secreto** en el encabezado `Authorization` de las solicitudes de Direct Line API. Si va a escribir una aplicación en la que el cliente se ejecuta en un explorador web o una aplicación móvil, puede que quiera intercambiar el secreto por un token (que solo sirve para una única conversación y expira a menos que se actualice) y especificar el **token** en el encabezado `Authorization` de las solicitudes de Direct Line API. Elija el modelo de seguridad que mejor funcione en su caso.

> [!NOTE]
> Las credenciales de cliente de Direct Line son diferentes de las credenciales del bot. Esto le permite revisar las claves de forma independiente y compartir los tokens de cliente sin revelar la contraseña del bot. 

## <a name="get-a-direct-line-secret"></a>Obtención de un secreto de Direct Line

También puede [obtener un secreto de Direct Line](../bot-service-channel-connect-directline.md) a través de la página de configuración del canal Direct Line de su bot en el <a href="https://dev.botframework.com/" target="_blank">portal de Bot Framework</a>:

![Configuración de Direct Line](../media/direct-line-configure.png)

## <a id="generate-token"></a> Generación de un token de Direct Line

Para generar un token de Direct Line que pueda usarse para acceder a una única conversación, primero obtenga el secreto de Direct Line de la página de configuración del canal Direct Line en el <a href="https://dev.botframework.com/" target="_blank">portal de Bot Framework</a>. A continuación, emita esta solicitud para intercambiar el secreto de Direct Line por un token de Direct Line:

```http
POST https://directline.botframework.com/v3/directline/tokens/generate
Authorization: Bearer SECRET
```

En el encabezado `Authorization` de esta solicitud, reemplace **SECRET** por el valor del secreto de Direct Line.

Los fragmentos de código siguientes proporcionan un ejemplo de la solicitud y respuesta de Generar un token.

### <a name="request"></a>Solicitud

```http
POST https://directline.botframework.com/v3/directline/tokens/generate
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

La carga de la solicitud, que contiene los parámetros del token, es opcional pero recomendable. Cuando se genera un token que se puede enviar al servicio Direct Line, proporcione la siguiente carga para hacer más segura la conexión. Al incluir estos valores, Direct Line puede realizar validaciones adicionales de la seguridad del identificador y el nombre de usuario, e impedir que clientes malintencionados manipulen estos valores. Con estos valores también mejora la capacidad de Direct Line de enviar la actividad _actualización de la conversación_, lo que le permite generar la actualización de la conversación inmediatamente cuando el usuario se une a la conversación. Cuando no se proporciona esta información, el usuario debe enviar contenido para que Direct Line pueda enviar la actualización de la conversación.

```json
{
  "user": {
    "id": "string",
    "name": "string"
  },
  "trustedOrigins": [
    "string"
  ]
}
```

| Parámetro | type | DESCRIPCIÓN |
| :--- | :--- | :--- |
| `user.id` | string | Opcional. Identificador de usuario específico del canal para su codificación dentro del token. Para un usuario de Direct Line, debe comenzar con `dl_`. Puede crear un identificador de usuario único para cada conversación y, para mejorar la seguridad, no se debería poder averiguar este identificador. |
| `user.name` | string | Opcional. Nombre para mostrar descriptivo del usuario para su codificación dentro del token. |
| `trustedOrigins` | matriz de cadenas | Opcional. Lista de dominios de confianza para insertar dentro del token. Estos son los dominios que pueden hospedar el cliente de chat web del bot. Debería coincidir con la lista de la página de configuración de Direct Line del bot. |

### <a name="response"></a>Response

Si la solicitud es correcta, la respuesta contiene un `token` que es válido para una conversación y un valor `expires_in` que indica el número de segundos hasta que el token expira. Para que el token siga siendo útil, debe [actualizarlo](#refresh-token) antes de que expire.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "expires_in": 1800
}
```

### <a name="generate-token-versus-start-conversation"></a>Generar token frente a Iniciar conversación

La operación Generar token (`POST /v3/directline/tokens/generate`) es parecida a la operación [Iniciar conversación](bot-framework-rest-direct-line-3-0-start-conversation.md) (`POST /v3/directline/conversations`) en el sentido de que ambas operaciones devuelven un `token` que se puede usar para acceder a una única conversación. Sin embargo, a diferencia de la operación Iniciar conversación, la operación Generar token no inicia la conversación o el contacto con el bot, y no crea una dirección URL de WebSocket de streaming. 

Si va a distribuir el token a los clientes y quiere que inicien la conversación, use la operación Generar token. Si tiene previsto iniciar inmediatamente la conversación, use en su lugar la operación [Iniciar conversación](bot-framework-rest-direct-line-3-0-start-conversation.md).

## <a id="refresh-token"></a> Actualización de un token de Direct Line

Un token de Direct Line se puede actualizar una cantidad ilimitada de veces, siempre y cuando no haya expirado. No se puede actualizar un token expirado. Para actualizar un token de Direct Line, emita esta solicitud: 

```http
POST https://directline.botframework.com/v3/directline/tokens/refresh
Authorization: Bearer TOKEN_TO_BE_REFRESHED
```

En el encabezado `Authorization` de esta solicitud, reemplace **TOKEN_TO_BE_REFRESHED** por el token de Direct Line que quiere actualizar.

Los fragmentos de código siguientes proporcionan un ejemplo de la solicitud y respuesta de Actualizar token.

### <a name="request"></a>Solicitud

```http
POST https://directline.botframework.com/v3/directline/tokens/refresh
Authorization: Bearer CurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Response

Si la solicitud tiene éxito, la respuesta contiene un nuevo `token` que es válido para la misma conversación que el token anterior, y un valor `expires_in` que indica el número de segundos hasta que expira el nuevo token. Para que el nuevo token siga siendo útil, debe [actualizarlo](#refresh-token) antes de que expire.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xniaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0",
  "expires_in": 1800
}
```

## <a name="azure-bot-service-authentication"></a>Autenticación de Azure Bot Service

La información que se presenta en esta sección se basa en el artículo [Incorporación de autenticación al bot mediante Azure Bot Service](../v4sdk/bot-builder-authentication.md).

La **autenticación de Azure Bot Service** le permite autenticar usuarios y obtener **tokens de acceso** de diversos proveedores de identidades como *Azure Active Directory*, *GitHub*, *Uber*, etc. También puede configurar la autenticación para un proveedor de identidades de **OAuth2** personalizado. Todo esto le permite escribir **un fragmento de código de autenticación** que funciona en todos los proveedores de identidades y canales admitidos. Para utilizar estas funcionalidades debe realizar los siguientes pasos:

1. Configure `settings` de forma estática en el bot que contiene los detalles del registro de aplicación con un proveedor de identidades.
2. Use un `OAuthCard`, con el respaldo de la información de la aplicación que proporcionó en el paso anterior, para iniciar sesión en un usuario.
3. Recupere los tokens de acceso mediante la **API de Azure Bot Service**.

### <a name="security-considerations"></a>Consideraciones sobre la seguridad

<!-- Summarized from: https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/ -->

Cuando se usa la *autenticación de Azure Bot Service* con [Web Chat](../bot-service-channel-connect-webchat.md) hay algunos aspectos de seguridad importantes que debe tener en cuenta.

1. **Suplantación**. La suplantación aquí significa que un atacante hace que el bot crea que es otra persona. En Web Chat, un atacante puede suplantar a otra persona **cambiando el identificador de usuario** de su instancia de Web Chat. Para evitar esto, se recomienda a los desarrolladores de bots que hagan que el **identificador de usuario no se pueda adivinar**. Si habilita las opciones de **autenticación mejoradas**, Azure Bot Service puede además detectar y rechazar cualquier cambio de identificador de usuario. Esto significa que en los mensajes de Direct Line al bot, el identificador de usuario (`Activity.From.Id`) siempre será igual al identificador con el que inicializó Web Chat. Tenga en cuenta que esta característica requiere que el identificador de usuario empiece por `dl_`.
1. **Identidades de usuario**. Debe tener en cuenta que está tratando con dos identidades de usuario:

    1. La identidad de un usuario en un canal.
    1. La identidad del usuario en un proveedor de identidades en el que el bot está interesado.
  
    Si un bot solicita al usuario A de un canal iniciar sesión en un proveedor de identidades P, el proceso de inicio de sesión debe garantizar que el usuario A es el que inicia sesión en P. Si se permitiera a otro usuario B iniciar sesión, el usuario A tendría acceso a los recursos del usuario B a través del bot. En Web Chat existen 2 mecanismos para asegurarse de que ha iniciado sesión el usuario correcto como se describe a continuación.

    1. En el pasado, al final del inicio de sesión, se presentaba al usuario un código de 6 dígitos generado aleatoriamente (también conocido como código mágico). El usuario debía escribir este código en la conversación que iniciaba el inicio de sesión para completar dicho proceso. Este mecanismo solía producir una mala experiencia del usuario. Además, aún así se corría el riesgo de sufrir ataques de suplantación de identidad (phishing). Un usuario malintencionado puede engañar a otro usuario para que inicie sesión y obtenga el código mágico mediante la suplantación de identidad (phishing).

    2. Debido a los problemas con el enfoque anterior, Azure Bot Service eliminó la necesidad del código mágico. Azure Bot Service garantiza que el proceso de inicio de sesión solo se puede completar en la **misma sesión del explorador** que el propio Web Chat. 
    Para habilitar esta protección, como desarrollador de bots, debe iniciar Web Chat con un **token de Direct Line** que contiene una **lista de dominios de confianza que pueden hospedar el cliente de Web Chat del bot**. Antes, solo podía obtener este token pasando un parámetro opcional no documentado a la API de token de Direct Line. Ahora, con las opciones de autenticación mejoradas, puede especificar estáticamente la lista de dominios de confianza (origen) en la página de configuración de Direct Line.

### <a name="code-examples"></a>Ejemplos de código

El siguiente controlador de .NET funciona con opciones de autenticación mejoradas habilitadas y devuelve un token y un identificador de usuario de Direct Line.

```csharp
public class HomeController : Controller
{
    public async Task<ActionResult> Index()
    {
        var secret = GetSecret();

        HttpClient client = new HttpClient();

        HttpRequestMessage request = new HttpRequestMessage(
            HttpMethod.Post,
            $"https://directline.botframework.com/v3/directline/tokens/generate");

        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", secret);

        var userId = $"dl_{Guid.NewGuid()}";

        request.Content = new StringContent(
            JsonConvert.SerializeObject(
                new { User = new { Id = userId } }),
                Encoding.UTF8,
                "application/json");

        var response = await client.SendAsync(request);
        string token = String.Empty;

        if (response.IsSuccessStatusCode)
        {
            var body = await response.Content.ReadAsStringAsync();
            token = JsonConvert.DeserializeObject<DirectLineToken>(body).token;
        }

        var config = new ChatConfig()
        {
            Token = token,
            UserId = userId  
        };

        return View(config);
    }
}

public class DirectLineToken
{
    public string conversationId { get; set; }
    public string token { get; set; }
    public int expires_in { get; set; }
}
public class ChatConfig
{
    public string Token { get; set; }
    public string UserId { get; set; }
}

```

El siguiente controlador de JavaScript funciona con opciones de autenticación mejoradas habilitadas y devuelve un token y un identificador de usuario de Direct Line.

```javascript
var router = express.Router(); // get an instance of the express Router

// Get a directline configuration (accessed at GET /api/config)
const userId = "dl_" + createUniqueId();

router.get('/config', function(req, res) {
    const options = {
        method: 'POST',
        uri: 'https://directline.botframework.com/v3/directline/tokens/generate',
        headers: {
            'Authorization': 'Bearer ' + secret
        },
        json: {
            User: { Id: userId }
        }
    };

    request.post(options, (error, response, body) => {
        if (!error && response.statusCode < 300) {
            res.json({ 
                    token: body.token,
                    userId: userId
                });
        }
        else {
            res.status(500).send('Call to retrieve token from Direct Line failed');
        } 
    });
});

```

## <a name="additional-resources"></a>Recursos adicionales

- [Conceptos clave](bot-framework-rest-direct-line-3-0-concepts.md)
- [Conectar un bot con Direct Line](../bot-service-channel-connect-directline.md)
- [Incorporación de autenticación al bot mediante Azure Bot Service](../bot-builder-tutorial-authentication.md)

---
title: Autenticación de solicitudes | Microsoft Docs
description: Obtenga información sobre cómo autenticar las solicitudes de API en Bot Connector API y en Bot State API.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: cbe2a6e449ecc2920e3a2d1ecb04a63dcb489b66
ms.sourcegitcommit: 8336a06941d09e1107b38f494d048dd785a13069
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2019
ms.locfileid: "68631566"
---
# <a name="authentication"></a>Authentication

El bot se comunica con el servicio Bot Connector con HTTP mediante un canal seguro (SSL/TLS). Cuando el bot envía una solicitud al servicio Connector, debe incluir información que el servicio Connector puede usar para comprobar su identidad. Del mismo modo, cuando el servicio Connector envía una solicitud al bot, debe incluir información que el bot puede usar para comprobar su identidad. En este artículo se describen las tecnologías y los requisitos para la autenticación en el nivel de servicio que tiene lugar entre un bot y el servicio Bot Connector. Si está escribiendo su propio código de autenticación, debe implementar los procedimientos de seguridad descritos en este artículo para habilitar al bot para el intercambio de mensajes con el servicio Bot Connector.

> [!IMPORTANT]
> Si está escribiendo su propio código de autenticación, es fundamental que aplique correctamente todos los procedimientos de seguridad. Mediante la implementación de todos los pasos de este artículo, puede mitigar el riesgo de que un atacante pueda leer los mensajes que se envían al bot, enviar mensajes que suplanten al bot y robar las claves secretas. 

Si usa [Bot Framework SDK para .NET](../dotnet/bot-builder-dotnet-overview.md) o [Bot Framework SDK para Node.js](../nodejs/index.md), no es necesario implementar los procedimientos de seguridad descritos en este artículo, que el SDK lo hace automáticamente. Basta con configurar el proyecto con el identificador de aplicación y la contraseña que obtuvo para el bot durante el [registro](../bot-service-quickstart-registration.md) y el SDK controlará el resto.

> [!WARNING]
> En diciembre de 2016, la versión v3.1 del protocolo de seguridad de Bot Framework presentó cambios en varios valores que se usan durante la generación y validación de los tokens. A finales de otoño de 2017, la versión v3.2 del protocolo de seguridad de Bot Framework presenta cambios en los valores que se usan durante la generación y validación de los tokens.
> Para más información, consulte [Cambios en el protocolo de seguridad](#security-protocol-changes).

## <a name="authentication-technologies"></a>Tecnologías de autenticación

Se utilizan cuatro tecnologías de autenticación para establecer la confianza entre un bot y Bot Connector:

| Technology | DESCRIPCIÓN |
|----|----|
| **SSL/TLS** | Se usa SSL/TLS para todas las conexiones de servicio a servicio. Se utilizan certificados`X.509v3` para establecer la identidad de todos los servicios HTTPS. **Los clientes siempre deben inspeccionar los certificados del servicio para asegurarse de que son válidos y de confianza.** (No se usan certificados de cliente como parte de este esquema). |
| **OAuth 2.0** | El inicio de sesión de OAuth 2.0 de la cuenta Microsoft (MSA) y el servicio de inicio de sesión de AAD v2 se usan para generar un token seguro que un bot puede usar para enviar mensajes. Este token es un token de servicio a servicio, no es necesario ningún inicio de sesión de usuario. |
| **JSON Web Token (JWT)** | Los JSON Web Token se usan para codificar los tokens que se envían hacia y desde el bot. **Los clientes deben comprobar completamente todos los tokens JWT que reciben**, según los requisitos descritos en este artículo. |
| **Metadatos de OpenID** | El servicio Bot Connector publica una lista de tokens válidos que usa para firmar sus propios tokens JWT en los metadatos de OpenID en un punto de conexión conocido y estático. |

En este artículo se describe cómo usar estas tecnologías mediante HTTPS y JSON estándar. No es necesario ningún SDK especial, aunque es posible que las aplicaciones auxiliares para OpenID y otras le sean útiles.

## <a id="bot-to-connector"></a> Autenticación de las solicitudes del bot al servicio Bot Connector

Para comunicarse con el servicio Bot Connector, debe especificar un token de acceso en el encabezado `Authorization` de cada solicitud de API, con este formato: 

```http
Authorization: Bearer ACCESS_TOKEN
```

Este diagrama muestra los pasos de la autenticación de bot a Connector:

![Autenticación con el servicio de inicio de sesión de MSA y, a continuación, con el bot](../media/connector/auth_bot_to_bot_connector.png)

> [!IMPORTANT]
> Si aún no lo ha hecho, deberá [registrar el bot](../bot-service-quickstart-registration.md) con Bot Framework para obtener el identificador de aplicación y la contraseña. Necesitará el identificador de la aplicación del bot y la contraseña para solicitar un token de acceso.

### <a name="step-1-request-an-access-token-from-the-msaaad-v2-login-service"></a>Paso 1: Solicitud de un token de acceso al servicio de inicio de sesión de MSA o AAD v2

Para solicitar un token de acceso al servicio de inicio de sesión de MSA o AAD v2, emita la siguiente solicitud y reemplace **MICROSOFT-APP-ID** y **MICROSOFT-APP-PASSWORD** por el identificador de aplicación y la contraseña que obtuvo cuando [registró](../bot-service-quickstart-registration.md) el bot con Bot Framework.

```http
POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=MICROSOFT-APP-ID&client_secret=MICROSOFT-APP-PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default
```

### <a name="step-2-obtain-the-jwt-token-from-the-msaaad-v2-login-service-response"></a>Paso 2: Obtención del token JWT de la respuesta del servicio de inicio de sesión de MSA o AAD v2

Si la aplicación es autorizada por el servicio de inicio de sesión de MSA o AAD v2, el cuerpo de la respuesta JSON especificará el token de acceso, su tipo y la expiración (en segundos). 

Al agregar el token al encabezado `Authorization` de una solicitud, debe usar el valor exacto especificado en esta respuesta (es decir, sin escape ni codificación del valor del token). El token de acceso es válido hasta su expiración. Para evitar que expire el token y afecte al rendimiento del bot, puede almacenarlo en caché y actualizar el token de forma proactiva.

En este ejemplo se muestra una respuesta del servicio de inicio de sesión de MSA o AAD v2:

```http
HTTP/1.1 200 OK
... (other headers) 
```

```json
{
    "token_type":"Bearer",
    "expires_in":3600,
    "ext_expires_in":3600,
    "access_token":"eyJhbGciOiJIUzI1Ni..."
}
```

### <a name="step-3-specify-the-jwt-token-in-the-authorization-header-of-requests"></a>Paso 3: Especificación del token JWT en el encabezado Authorization de las solicitudes

Cuando se envía una solicitud de API al servicio Bot Connector, debe especificar el token de acceso en el encabezado `Authorization` de la solicitud con este formato:

```http
Authorization: Bearer ACCESS_TOKEN
```

Todas las solicitudes que envíe al servicio Bot Connector deben incluir el token de acceso en el encabezado `Authorization`. Si el token tiene un formato correcto, no ha expirado y fue generado por el servicio de inicio de sesión de MSA o AAD v2, el servicio Bot Connector autorizará la solicitud. Se realizan comprobaciones adicionales para asegurarse de que el token pertenece al bot que envió la solicitud.

El ejemplo siguiente muestra cómo especificar el token de acceso en el encabezado `Authorization` de la solicitud. 

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/12345/activities 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
    
(JSON-serialized Activity message goes here)
```

> [!IMPORTANT]
> Especifique el token JWT solo en el encabezado `Authorization` de las solicitudes que envíe al servicio Bot Connector. No envíe el token por canales no seguros y no lo incluya en las solicitudes HTTP que se envían a otros servicios. El token JWT que obtiene del servicio de inicio de sesión de MSA o AAD v2 es como una contraseña y debe utilizarse con mucho cuidado. Cualquiera que tenga el token puede usarlo para realizar operaciones en nombre del bot. 

#### <a name="bot-to-connector-example-jwt-components"></a>Bot a Connector: componentes JWT de ejemplo

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "https://api.botframework.com",
  iss: "https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/",
  nbf: 1481049243,
  exp: 1481053143,
  appid: "<YOUR MICROSOFT APP ID>",
  ... other fields follow
}
```

> [!NOTE]
> Los campos reales pueden variar en la práctica. Debe crear y validar todos los tokens JWT según lo especificado anteriormente.

## <a id="connector-to-bot"></a> Autenticación de las solicitudes del servicio Bot Connector al bot

Cuando el servicio Bot Connector envía una solicitud al bot, especifica un token JWT firmado en el encabezado `Authorization` de la solicitud. El bot puede autenticar las llamadas desde el servicio Bot Connector al verificar la autenticidad del token JWT firmado. 

Este diagrama muestra los pasos de la autenticación de Connector a bot:

![Autenticación de las llamadas desde Bot Connector al bot](../media/connector/auth_bot_connector_to_bot.png)

### <a id="openid-metadata-document"></a> Paso 2: Obtención del documento de metadatos de OpenID

El documento de metadatos de OpenID especifica la ubicación de un segundo documento que enumera las claves de firma válidas del servicio Bot Connector. Para obtener el documento de metadatos de OpenID, emita esta solicitud mediante HTTPS:

```http
GET https://login.botframework.com/v1/.well-known/openidconfiguration
```

> [!TIP]
> Se trata de una dirección URL estática que se puede insertar directamente en la aplicación. 

El ejemplo siguiente muestra un documento de metadatos de OpenID devuelto en respuesta a la solicitud `GET`. La propiedad `jwks_uri` especifica la ubicación del documento que contiene las claves de firma válidas del servicio Bot Connector.

```json
{
    "issuer": "https://api.botframework.com",
    "authorization_endpoint": "https://invalid.botframework.com",
    "jwks_uri": "https://login.botframework.com/v1/.well-known/keys",
    "id_token_signing_alg_values_supported": [
      "RS256"
    ],
    "token_endpoint_auth_methods_supported": [
      "private_key_jwt"
    ]
}
```

### <a id="connector-to-bot-step-3"></a> Paso 3: Obtención de la lista de claves de firma válidas

Para obtener la lista de claves de firma válidas, emita una solicitud `GET` mediante HTTPS a la dirección URL especificada por la propiedad `jwks_uri` en el documento de metadatos de OpenID. Por ejemplo:

```http
GET https://login.botframework.com/v1/.well-known/keys
```

El cuerpo de la respuesta especifica el documento en [formato JWK](https://tools.ietf.org/html/rfc7517), pero también incluye una propiedad adicional para cada clave: `endorsements`. La lista de claves es relativamente estable y se puede almacenar en caché durante largos períodos de tiempo (de forma predeterminada, 5 días en Bot Framework SDK).

La propiedad `endorsements` dentro de cada clave contiene una o más cadenas de aprobación que puede usar para comprobar que el identificador de canal especificado en la propiedad `channelId` dentro del objeto [Activity][Activity] de la solicitud entrante es auténtico. La lista de identificadores de canal que requieren aprobaciones es configurable en cada bot. De forma predeterminada, será la lista de todos los identificadores de canal publicados, aunque los desarrolladores de bots pueden invalidar los valores de identificador de canal seleccionados en cualquier caso. 

### <a name="step-4-verify-the-jwt-token"></a>Paso 4: Verificación del token JWT

Para verificar la autenticidad del token enviado por el servicio Bot Connector, debe extraer el token del encabezado `Authorization` de la solicitud, analizar el token, verificar su contenido y verificar su firma. 

Hay disponibles bibliotecas de análisis de JWT para muchas plataformas y la mayoría implementa un análisis de los tokens JWT seguro y confiable, aunque normalmente deberá configurar estas bibliotecas para requerir que determinadas características del token (su emisor, audiencia, etc.) contengan valores correctos. Al analizar el token, debe configurar la biblioteca de análisis o escribir su propia validación para asegurarse de que el token cumple estos requisitos:

1. El token se ha enviado en el encabezado HTTP `Authorization` con un esquema "Bearer" (Portador).
2. El token tiene un formato JSON válido que se ajusta al [estándar JWT](http://openid.net/specs/draft-jones-json-web-token-07.html).
3. El token contiene una notificación "issuer" (emisor) con el valor `https://api.botframework.com`.
4. El token contiene una notificación "audience" (audiencia) con un valor igual al identificador de aplicación de Microsoft del bot.
5. El token está dentro de su período de validez. El tiempo estándar del sector es de 5 minutos.
6. El token tiene una firma criptográfica válida, con una clave enumerada en el documento de claves de OpenID que se recuperó en el [Paso 3](#connector-to-bot-step-3), utilizando el algoritmo de firma que se especifica en la propiedad `id_token_signing_alg_values_supported` del documento de metadatos de OpenID que se recuperó en [Paso 2](#openid-metadata-document).
7. El token contiene una notificación "serviceUrl" (dirección URL de servicio) cuyo valor coincide con la propiedad `servieUrl` en la raíz del objeto [Activity][Activity] de la solicitud entrante. 

Si se requiere aprobación para un identificador de canal:

- Debe requerir que cualquier objeto [Activity][Activity] enviado al bot con ese identificador de canal venga acompañado de un token JWT firmado con una aprobación para ese canal. 
- Si la aprobación no está presente, el bot debe rechazar la solicitud devolviendo un código de estado **HTTP 403 (prohibido)** .

> [!IMPORTANT]
> Todos estos requisitos son importantes, especialmente los requisitos 4 y 6. En caso de no implementarse todos estos requisitos de comprobación, se dejaría el bot abierto a ataques que podrían provocar que el bot divulgue su token JWT.

Los implementadores no deben exponer una manera de deshabilitar la validación del token JWT que se envía al bot.

#### <a name="connector-to-bot-example-jwt-components"></a>Connector a bot: componentes JWT de ejemplo

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "<YOU MICROSOFT APP ID>",
  iss: "https://api.botframework.com",
  nbf: 1481049243,
  exp: 1481053143,
  ... other fields follow
}
```

> [!NOTE]
> Los campos reales pueden variar en la práctica. Debe crear y validar todos los tokens JWT según lo especificado anteriormente.

## <a id="emulator-to-bot"></a> Autenticación de las solicitudes del emulador de Bot Framework al bot

> [!WARNING]
> A finales de otoño de 2017, se presenta la versión v3.2 del protocolo de seguridad de Bot Framework. Esta nueva versión incluye un nuevo valor "issuer" (emisor) dentro de los tokens que se intercambian entre el emulador de Bot Framework y el bot. Para prepararse para este cambio, los pasos siguientes describen cómo comprobar los valores de emisor de las versiones v3.1 y v3.2. 

El [Emulador de Bot Framework](../bot-service-debug-emulator.md) es una herramienta de escritorio que puede usar para probar la funcionalidad del bot. Aunque el emulador de Bot Framework usa las mismas [tecnologías de autenticación](#authentication-technologies) que se describieron anteriormente, no puede suplantar al servicio Bot Connector real. En su lugar, usa el identificador de aplicación de Microsoft y la contraseña de aplicación de Microsoft que se especifican al conectar el emulador al bot para crear tokens que son idénticos a los que crea el bot. Cuando el emulador envía una solicitud al bot, especifica el token JWT en el encabezado `Authorization` de la solicitud: en esencia, utiliza las propias credenciales del bot para autenticar la solicitud. 

Si está implementando una biblioteca de autenticación y desea aceptar solicitudes desde el emulador de Bot Framework, debe agregar esta ruta de acceso de comprobación adicional. La ruta de acceso es estructuralmente similar a la ruta de acceso de comprobación [Connector -> Bot](#connector-to-bot), pero usa el documento de OpenID de MSA en lugar del documento de OpenID de Bot Connector.

Este diagrama muestra los pasos de la autenticación de emulador a bot:

![Autenticación de las llamadas del emulador de Bot Framework al bot](../media/connector/auth_bot_framework_emulator_to_bot.png)

---
### <a name="step-2-get-the-msa-openid-metadata-document"></a>Paso 2: Obtención del documento de metadatos de OpenID de MSA

El documento de metadatos de OpenID especifica la ubicación de un segundo documento que enumera las claves de firma válidas. Para obtener el documento de metadatos de OpenID de MSA, emita esta solicitud mediante HTTPS:

```http
GET https://login.microsoftonline.com/botframework.com/v2.0/.well-known/openid-configuration
```

El ejemplo siguiente muestra un documento de metadatos de OpenID devuelto en respuesta a la solicitud `GET`. La propiedad `jwks_uri` especifica la ubicación del documento que contiene las claves de firma válidas.

```json
{
    "authorization_endpoint":"https://login.microsoftonline.com/common/oauth2/v2.0/authorize",
    "token_endpoint":"https://login.microsoftonline.com/common/oauth2/v2.0/token",
    "token_endpoint_auth_methods_supported":["client_secret_post","private_key_jwt"],
    "jwks_uri":"https://login.microsoftonline.com/common/discovery/v2.0/keys",
    ...
}
```

### <a id="emulator-to-bot-step-3"></a> Paso 3: Obtención de la lista de claves de firma válidas

Para obtener la lista de claves de firma válidas, emita una solicitud `GET` mediante HTTPS a la dirección URL especificada por la propiedad `jwks_uri` en el documento de metadatos de OpenID. Por ejemplo:

```http
GET https://login.microsoftonline.com/common/discovery/v2.0/keys 
Host: login.microsoftonline.com
```

El cuerpo de la respuesta especifica el documento en [formato JWK](https://tools.ietf.org/html/rfc7517). 

### <a name="step-4-verify-the-jwt-token"></a>Paso 4: Verificación del token JWT

Para verificar la autenticidad del token enviado por el emulador, debe extraer el token del encabezado `Authorization` de la solicitud, analizar el token, verificar su contenido y verificar su firma. 

Hay disponibles bibliotecas de análisis de JWT para muchas plataformas y la mayoría implementa un análisis de los tokens JWT seguro y confiable, aunque normalmente deberá configurar estas bibliotecas para requerir que determinadas características del token (su emisor, audiencia, etc.) contengan valores correctos. Al analizar el token, debe configurar la biblioteca de análisis o escribir su propia validación para asegurarse de que el token cumple estos requisitos:

1. El token se ha enviado en el encabezado HTTP `Authorization` con un esquema "Bearer" (Portador).
2. El token tiene un formato JSON válido que se ajusta al [estándar JWT](http://openid.net/specs/draft-jones-json-web-token-07.html).
3. El token contiene una notificación "issuer" (emisor) con el valor `https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/` o `https://sts.windows.net/f8cdef31-a31e-4b4a-93e4-5f571e91255a/`. (La comprobación de ambos valores de emisor garantizará que está comprobando tanto los valores de emisor del protocolo de seguridad v3.1 como los del v3.2)
4. El token contiene una notificación "audience" (audiencia) con un valor igual al identificador de aplicación de Microsoft del bot.
5. El token contiene una notificación "appid" (identificador de aplicación) con un valor igual al identificador de aplicación de Microsoft del bot.
6. El token está dentro de su período de validez. El tiempo estándar del sector es de 5 minutos.
7. El token tiene una firma criptográfica válida con una clave enumerada en el documento de claves de OpenID que se recuperó en el [paso 3](#emulator-to-bot-step-3).

> [!NOTE]
> El requisito 5 es específico para la ruta de acceso de comprobación del emulador. 

Si el token no cumple todos estos requisitos, el bot debe finalizar la solicitud devolviendo un código de estado **HTTP 403 (prohibido)** .

> [!IMPORTANT]
> Todos estos requisitos son importantes, especialmente los requisitos 4 y 7. En caso de no implementarse todos estos requisitos de comprobación, se dejaría el bot abierto a ataques que podrían provocar que el bot divulgue su token JWT.

#### <a name="emulator-to-bot-example-jwt-components"></a>Emulador a bot: componentes JWT de ejemplo

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "<YOUR MICROSOFT APP ID>",
  iss: "https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/",
  nbf: 1481049243,
  exp: 1481053143,
  ... other fields follow
}
```

> [!NOTE]
> Los campos reales pueden variar en la práctica. Debe crear y validar todos los tokens JWT según lo especificado anteriormente.

## <a name="security-protocol-changes"></a>Cambios del protocolo de seguridad

> [!WARNING]
> La compatibilidad con la versión v3.0 del protocolo de seguridad se interrumpió el **31 de julio de 2017**. Si ha escrito su propio código de autenticación (es decir, no ha usado Bot Framework SDK para crear el bot), debe actualizar a la versión v3.1 del protocolo de seguridad mediante la actualización de la aplicación para que utilice los valores de la versión v3.1 que se enumeran a continuación. 

### <a name="bot-to-connector-authenticationbot-to-connector"></a>[Autenticación bot a Connector](#bot-to-connector)

#### <a name="oauth-login-url"></a>Dirección URL de inicio de sesión de OAuth

| Versión del protocolo | Valor válido |
|----|----|
| v3.1 y v3.2 | `https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token` |

#### <a name="oauth-scope"></a>Ámbito de OAuth

| Versión del protocolo | Valor válido |
|----|----|
| v3.1 y v3.2 |  `https://api.botframework.com/.default` |

### <a name="connector-to-bot-authenticationconnector-to-bot"></a>[Autenticación de Connector a bot](#connector-to-bot)

#### <a name="openid-metadata-document"></a>Documento de metadatos de OpenID

| Versión del protocolo | Valor válido |
|----|----|
| v3.1 y v3.2 | `https://login.botframework.com/v1/.well-known/openidconfiguration` |

#### <a name="jwt-issuer"></a>Emisor de JWT

| Versión del protocolo | Valor válido |
|----|----|
| v3.1 y v3.2 | `https://api.botframework.com` |

### <a name="emulator-to-bot-authenticationemulator-to-bot"></a>[Autenticación de emulador a bot](#emulator-to-bot)

#### <a name="oauth-login-url"></a>Dirección URL de inicio de sesión de OAuth

| Versión del protocolo | Valor válido |
|----|----|
| v3.1 y v3.2 | `https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token` |

#### <a name="oauth-scope"></a>Ámbito de OAuth

| Versión del protocolo | Valor válido |
|----|----|
| v3.1 y v3.2 |  Identificador de aplicación de Microsoft del bot + `/.default` |

#### <a name="jwt-audience"></a>Audiencia de JWT

| Versión del protocolo | Valor válido |
|----|----|
| v3.1 y v3.2 | Identificador de aplicación de Microsoft del bot |

#### <a name="jwt-issuer"></a>Emisor de JWT

| Versión del protocolo | Valor válido |
|----|----|
| v3.1 | `https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/` |
| v3.2 | `https://sts.windows.net/f8cdef31-a31e-4b4a-93e4-5f571e91255a/` |

#### <a name="openid-metadata-document"></a>Documento de metadatos de OpenID

| Versión del protocolo | Valor válido |
|----|----|
| v3.1 y v3.2 | `https://login.microsoftonline.com/botframework.com/v2.0/.well-known/openid-configuration` |

## <a name="additional-resources"></a>Recursos adicionales

- [Solución de problemas de autenticación de Bot Framework](../bot-service-troubleshoot-authentication-problems.md)
- [JSON Web Token (JWT) draft-jones-json-web-token-07](http://openid.net/specs/draft-jones-json-web-token-07.html)
- [Firma web JSON (JWS) draft-jones-json-web-signature-04](https://tools.ietf.org/html/draft-jones-json-web-signature-04)
- [Clave web JSON (JWK) RFC 7517](https://tools.ietf.org/html/rfc7517)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object

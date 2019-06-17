---
title: Autenticación de usuarios en Azure Bot Service | Microsoft Docs
description: Más información sobre las características de autenticación de usuarios en Azure Bot Service.
keywords: azure bot service, autenticación, servicio de token de bot framework
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 05/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6eea954f58096d89cd3278058146b93fa04f435f
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693765"
---
# <a name="user-authentication-within-a-conversation"></a>Autenticación de usuarios de una conversación

Para realizar determinadas operaciones en nombre de un usuario, como comprobar el correo electrónico, hacer referencia a un calendario, comprobar el estado de un vuelo o realizar un pedido, el bot deberá llamar a un servicio externo, como Microsoft Graph, GitHub o un servicio REST de la empresa.
Cada uno de los servicios externos tiene una manera de proteger las llamadas y una forma habitual de hacerlo es emitir esas solicitudes mediante un _token de usuario_ que identifica de manera exclusiva al usuario en ese servicio externo (lo cual se conoce también como un JWT).

Para proteger la llamada a un servicio externo, el bot debe pedir al usuario que inicie sesión para poder adquirir el token del usuario para ese servicio.
Muchos servicios admiten la recuperación de token mediante el protocolo OAuth o OAuth2.
Azure Bot Service ofrece tarjetas y servicios especializados de inicio de sesión que funcionan con el protocolo OAuth y administran el ciclo de vida del token, y un bot puede usar estas características para adquirir un token de usuario.

- Como parte de la configuración del bot, se registra una _configuración de conexión de OAuth_ en el recurso de Azure Bot Service en Azure.

    Cada configuración de conexión contiene información sobre el servicio externo o el proveedor de identidades que se van a utilizar, junto con un identificador de cliente de OAuth válido y secreto, los ámbitos de OAuth que se van a habilitar y cualquier otro metadato de conexión que requiera ese servicio externo o proveedor de identidades.

- En el código del bot, se usa una configuración de conexión de OAuth para ayudar a un usuario a que inicie sesión y obtenga un token de usuario.

Estos servicios intervienen en el flujo de trabajo de inicio de sesión.

![Información general sobre la autenticación](./media/bot-builder-concept-authentication.png)

## <a name="about-the-bot-framework-token-service"></a>Acerca del servicio de token de Bot Framework

El servicio de token de Framework Bot es responsable de:

- Facilitar el uso del protocolo OAuth con una amplia variedad de servicios externos.
- Almacenar de forma segura los tokens para un bot, canal, conversación y usuario determinados.
- Administrar el ciclo de vida del token, incluido el intentar actualizar los tokens.

Por ejemplo, un bot que puede comprobar los correos electrónicos recientes de un usuario mediante Microsoft Graph API, necesitará un token de usuario de Azure Active Directory. Durante el diseño, el desarrollador del bot registrará una aplicación de Azure Active Directory con el servicio de token de Bot Framework (a través de Azure Portal) y, a continuación, configurará un valor de conexión de OAuth (llamado `GraphConnection`) para el bot. Si un usuario interactúa con el bot, el flujo de trabajo sería:

1. El usuario pide al bot: "comprueba mi correo electrónico".
1. Una actividad con este mensaje se envía desde el usuario al servicio de canal de Bot Framework. El servicio de canal garantiza que se ha configurado el campo `userid` de la actividad y que el mensaje se envía al bot.

    Los identificadores de usuario son específicos del canal como, por ejemplo, el identificador de Facebook del usuario o su número de teléfono para SMS.

1. El bot recibe la actividad de mensaje y determina que la intención es comprobar el correo electrónico del usuario. El bot realiza una solicitud al servicio de token de Framework Bot que le pregunta si ya tiene un token para el identificador de usuario de la configuración de conexión de OAuth `GraphConnection`.
1. Puesto que es la primera vez que este usuario ha interactuado con el bot, el servicio de token de Bot Framework no tiene todavía ningún token para este usuario y devuelve un resultado _NotFound_ al bot.
1. El bot crea una tarjeta OAuthCard con un nombre de conexión `GraphConnection` y responde al usuario pidiéndole que inicie sesión con esta tarjeta.
1. La actividad pasa a través del servicio de canal de Bot Framework, que llama al servicio de token de Bot Framework para crear una dirección URL de inicio de sesión de OAuth válida para esta solicitud. Esta dirección URL de inicio de sesión se agrega a OAuthCard y la tarjeta se devuelve al usuario.
1. Al usuario se le presenta un mensaje para que inicie sesión haciendo clic en el botón Iniciar sesión de OAuthCard.
1. Cuando el usuario hace clic en el botón de inicio de sesión, el servicio de canal abre un explorador web y llama al servicio externo para que cargue su página de inicio de sesión.
1. El usuario inicia sesión en esta página para el servicio externo. Una vez finalizado, el servicio externo completa el intercambio del protocolo OAuth con el servicio de token de Bot Framework, lo que hace que el servicio externo envíe al servicio de token de Bot Framework el token del usuario. El servicio de token de Bot Framework almacena de forma segura este token y envía una actividad al bot con él.
1. El bot recibe la actividad con el token y puede usarlo para realizar llamadas en Graph API.

## <a name="securing-the-sign-in-url"></a>Protección de la dirección URL de inicio de sesión

Una consideración importante a tener en cuenta cuando Bot Framework facilita el inicio de sesión de un usuario es la protección de la dirección URL de inicio de sesión. Cuando se presenta a un usuario una dirección URL de inicio de sesión, esta se asocia a un identificador de conversación y de usuario específicos para ese bot. Esta dirección URL no debe compartirse ya que, de hacerlo, podría provocar un inicio de sesión incorrecto para una determinada conversación del bot. Para mitigar los ataques de seguridad relacionados con el uso compartido de la dirección URL de inicio de sesión, es necesario asegurarse de que la máquina y la persona que hace clic en la dirección URL de inicio de sesión es la persona que _posee_ la ventana de la conversación.

Algunos canales como Cortana, Teams, Direct Line y WebChat pueden hacer esto sin que el usuario se dé cuenta. Por ejemplo, WebChat utiliza cookies de sesión para asegurarse de que el flujo de inicio de sesión tuvo lugar en el mismo explorador que la conversación de WebChat. Sin embargo, en el caso de otros canales, al usuario se le presenta a menudo un _código mágico_ de 6 dígitos. Esto se parece a la autenticación multifactor integrada ya que el servicio de token de Bot Framework no publicará el token para el bot a menos que el usuario termine la autenticación final y demuestre que la persona que ha iniciado sesión tiene acceso a la experiencia de chat escribiendo el código de 6 dígitos.

## <a name="azure-activity-directory-application-registration"></a>Registro de aplicación de Azure Activity Directory

Cada bot que se registra como una instancia de Azure Bot Service usa un identificador de aplicación de Azure Active Directory (AD). Es importante **no** reutilizar este identificador de aplicación y la contraseña para el inicio de sesión de usuarios. El identificador de aplicación de Azure AD para Azure Bot Service se usa para proteger la comunicación de servicio a servicio entre el bot y el servicio de canal de Bot Framework. Si desea que los usuarios inicien sesión en Azure AD, debe crear un registro de aplicación de Azure AD independiente con los permisos y ámbitos adecuados.

## <a name="configure-an-oauth-connection-setting"></a>Configuración de un valor de conexión de OAuth

Para más información sobre cómo registrar y usar un valor de conexión de OAuth, consulte [Adición de autenticación al bot](bot-builder-authentication.md).

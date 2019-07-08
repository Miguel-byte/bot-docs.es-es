---
title: Conectar un bot a Cortana | Microsoft Docs
description: Obtenga información sobre cómo configurar un bot para acceder a través de la interfaz de Cortana.
keywords: cortana, canal de bot, configurar cortana
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/30/2018
ms.openlocfilehash: 3df9d22b486e56547452cc5bce4add3946f670f5
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405937"
---
# <a name="connect-a-bot-to-cortana"></a>Conectar un bot a Cortana

Cortana es un canal habilitado para voz que puede enviar y recibir mensajes de voz, además de tener conversaciones por escrito. Un bot que se vaya a conectar a Cortana debe diseñarse para que admita tanto la voz y el texto. Una *habilidad* de Cortana es un bot que se puede invocar con un cliente de Cortana. Si publica un bot, se agregará a la lista de habilidades disponibles.

Para agregar el canal de Cortana, abra el bot en [Azure Portal](https://portal.azure.com/), haga clic en la hoja **Canales** y luego en **Cortana**.

![Agregar el canal de Cortana](~/media/channels/cortana-addchannel.png)

## <a name="configure-cortana"></a>Configurar Cortana

Al conectar su bot con el canal de Cortana, parte de la información básica sobre el mismo se completará previamente en el formulario de registro. Revise esta información detenidamente. Este formulario consta de los siguientes campos:

| Campo | DESCRIPCIÓN |
|------|------|
| **Icono de habilidades** | Un icono que se muestra en el lienzo de Cortana cuando se invoca la habilidad. También se usa cuando las habilidades son detectables (como Microsoft Store). (32 KB máximo; solo PNG).|
| **Nombre para mostrar** | El nombre de la habilidad de Cortana se muestra al usuario en la parte superior de la interfaz de usuario visual. (Límite de 30 caracteres). |
| **Nombre de invocación** | Este es el nombre que los usuarios deben decir al invocar una habilidad. No debe tener más de tres palabras y debe ser fácil pronunciar. Consulte la [Invocation Name Guidelines][invocation] (Información del nombre de invocación) para obtener más información sobre cómo elegir este nombre.|

![Configuración predeterminada](~/media/channels/cortana-defaultsettings.png)

>!NOTA: Cortana no admite actualmente el uso de autenticación de la cuenta de Azure Active Directory (AAD). Deberá usar una cuenta de Microsoft (MSA) para publicar correctamente su bot en Cortana.

## <a name="general-bot-information"></a>Información general sobre el bot

En la sección **Manage user identity through connected services** (Administrar la identidad del usuario a través de los servicios conectados), seleccione esta opción para habilitarla. Rellene el formulario.

Todos los campos marcados con un asterisco (*) son obligatorios. Para que un bot se pueda publicar en Cornada, debe publicarse primero en Azure.

![Administración de identidades de usuario, parte 1](~/media/channels/cortana-manageidentity-1.png)
![Administracion de identidades de usuario, parte 2](~/media/channels/cortana-manageidentity-2.png)

### <a name="when-should-cortana-prompt-for-a-user-to-sign-in"></a>Cuándo debe solicitar Cortana a un usuario que inicie sesión

Seleccione **Sign in at invocation** (Iniciar sesión durante la invocación) si quiere que Cortana inicie la sesión del usuario cuando invoca su habilidad.

Seleccione **Sign in when required** (Iniciar sesión cuando sea necesario) si usa una tarjeta de inicio de sesión de Bot Service para iniciar la sesión del usuario. Normalmente, usará esta opción si quiere iniciar la sesión del usuario solo si este utiliza una característica que requiera autenticación. Cuando su habilidad envía un mensaje que incluye la tarjeta de inicio de sesión como un archivo adjunto, Cortana la ignora y realiza el flujo de autorización mediante la configuración para conectar la cuenta.

### <a name="account-name"></a>Nombre de cuenta

El nombre de habilidad que quiere mostrar cuando el usuario inicie sesión en la misma.

### <a name="client-id-for-third-party-services"></a>Id. de cliente de servicios de terceros

Identificador de aplicación del bot. Recibió el identificador al registrar el bot.

### <a name="space-separated-list-of-scopes"></a>Lista de ámbitos separada por espacios

Especifique los ámbitos que necesita el servicio (consulte la documentación del servicio).

### <a name="authorization-url"></a>Dirección URL de autorización

Establézcalo en `https://login.microsoftonline.com/common/oauth2/v2.0/authorize`.

### <a name="token-options"></a>Opciones de token

Seleccione **POST**.

### <a name="grant-type"></a>Tipo de concesión

Seleccione **Authorization code** (Código de autorización) para usar el flujo de concesión de código, o seleccione **Implicit** (Implícito) para usar el flujo implícito.

### <a name="token-url"></a>Dirección URL de token

En el tipo de concesión **Authorization code** (Código de autorización), establezca `https://login.microsoftonline.com/common/oauth2/v2.0/token`.

### <a name="client-secretpassword-for-third-party-services"></a>Contraseña o secreto de cliente para servicios de terceros

Contraseña del bot. Recibió la contraseña al registrar el bot.

### <a name="client-authentication-scheme"></a>Esquema de autenticación de clientes

Seleccione **HTTP Basic** (HTTP básica).

### <a name="internet-access-required-to-authenticate-users"></a>Se necesita acceso a Internet para autenticar a los usuarios

Deje esta opción desactivada.

### <a name="request-user-profile-data-optional"></a>Solicitar datos de perfil de usuario (opcional)

Cortana proporciona acceso a diferentes tipos de información de perfil de usuario, que puede usar para personalizar el bot para el usuario. Por ejemplo, si una habilidad tiene acceso al nombre y la ubicación del usuario, entonces la habilidad puede proporcionar una respuesta personalizada como "Hola Kamran, espero que estés teniendo un día agradable en Bellevue, Washington".

Haga clic en **Add a user profile request** (Agregar una solicitud de perfil de usuario) y seleccione en la lista desplegable la información del perfil de usuario que quiera usar. Agregue un nombre descriptivo que se pueda usar para obtener acceso a esta información desde el código del bot.

### <a name="deploy-on-cortana"></a>Implementar en Cortana

Cuando haya completado el formulario de registro de su habilidad de Cortana, haga clic en **Deploy on Cortana** (Implementar en Cortana) para finalizar la conexión. Esto le llevará de vuelta a la hoja Canales del bot, donde verá que ya está conectado a Cortana.

En este momento su bot está implementado como una habilidad de Cortana en su cuenta.

## <a name="next-steps"></a>Pasos siguientes

* [Cortana Skills Kit](https://aka.ms/CortanaSkillsKitOverview) (Kit de habilidades de Cortana)
* [Habilitar depuración](bot-service-debug-cortana-skill.md)
* [Publicar una habilidad de Cortana][publish]

[invocation]: https://docs.microsoft.com/cortana/skills/cortana-invocation-guidelines
[publish]: https://docs.microsoft.com/cortana/skills/publish-skill
[CortanaEntity]: https://aka.ms/lgvcto

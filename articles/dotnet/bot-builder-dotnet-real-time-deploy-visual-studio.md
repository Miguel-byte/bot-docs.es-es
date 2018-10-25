---
title: Implementación de un bot multimedia en tiempo real de Skype en Azure | Microsoft Docs
description: Obtenga información sobre cómo implementar un bot de audio y vídeo en tiempo real de Skype en Azure mediante la característica de publicación integrada de Visual Studio.
author: MalarGit
ms.author: malarch
manager: ssulzer
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 22cce8ad5bef3c1c6f08a8efc28118e0209dd3af
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999442"
---
# <a name="deploy-a-real-time-media-bot-from-visual-studio-to-azure"></a>Implementación de un bot multimedia en tiempo real de Visual Studio en Azure

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Los bots multimedia en tiempo real pueden hospedarse en una instancia de Azure Virtual Machine "IaaS" o en un servicio de nube de Azure "clásico". En este artículo se muestra cómo implementar un bot, hospedado en un rol de trabajo del servicio de nube de Azure, desde Visual Studio con la funcionalidad de publicación integrada.

## <a name="prerequisites"></a>Requisitos previos

Debe tener una suscripción a Microsoft Azure antes de poder implementar un bot en Azure. Si aún no tiene una suscripción, puede registrarse para obtener una <a href="https://azure.microsoft.com/en-us/free/" target="_blank">cuenta gratuita</a>. Además, el proceso descrito en este artículo requiere Visual Studio. Si aún no tiene Visual Studio, puede descargar gratis <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017 Community</a>.

### <a name="certificate-from-a-valid-certificate-authority"></a>Certificado de una entidad de certificación válida
El bot debe configurarse con un certificado válido de una autoridad de certificado de confianza (CA). El nombre del firmante (SN) o la última entrada del nombre alternativo del firmante (SAN) del certificado debe ser el nombre del servicio de nube. Actualmente no se admiten certificados comodín. Si se usa un registro CNAME para apuntar al servicio en la nube, el CNAME SN debe ser el SN o la última entrada del SAN del certificado.

## <a name="configure-application-settings"></a>Configuración de la aplicación
Para que el bot funcione correctamente en la nube, debe asegurarse de que la configuración de la aplicación sea correcta. Más concretamente, establezca los siguientes valores de clave en el archivo app.config de su rol de trabajo:
> <ul><li>MicrosoftAppId</li><li>MicrosoftAppPassword</li></ul>

> [!NOTE]
> Para buscar **AppID** y **AppPassword** del bot, consulte [MicrosoftAppID y MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

## <a name="create-worker-role-in-the-azure-portal"></a>Creación del rol de trabajo en Azure Portal
### <a name="step-1-create-cloud-serviceclassic"></a>Paso 1: Crear el servicio en la nube (clásico)
Inicie sesión en <a href="https://portal.azure.com">Azure Portal</a>. Seleccione **+** en el lado izquierdo de la pantalla y elija **Servicio en la nube (clásico)**. Proporcione la información necesaria en el formulario y haga clic en **Crear**.

![Creación del servicio en la nube](../media/real-time-media-bot-portal-service-creation.png)

> [!NOTE]
> Debe proporcionarse el nombre DNS del bot en la dirección URL para el registro del bot.

### <a name="step-2-upload-the-certificate-for-the-bot"></a>Paso 2: Cargar el certificado para el bot
Una vez creado el bot, cargue el certificado para el bot.

![Carga del certificado](../media/real-time-media-bot-portal-certificates.png)

## <a name="modify-service-configuration-with-worker-role-details"></a>Modificación de la configuración del servicio con los detalles del rol de trabajo
El nombre de dominio completo (FQDN) del bot no está disponible a través de las API de Azure RoleEnvironment. Por lo tanto, se debe proporcionar el FQDN al bot. También es necesario que sepa sobre el certificado para HTTPS. Esto se puede configurar en el archivo de configuración del servicio (.cscfg) del rol de trabajo.

> [!TIP]
> Si va a implementar un ejemplo del repositorio de Git BotBuilder RealTimeMediaCalling:
> - Sustituya el $DnsName$ por el nombre del servicio en la nube o el registro CNAME si se usa uno en la configuración del servicio.
>   ```xml
>      <Setting name="ServiceDnsName" value="$DnsName$" />
>   ```
> 
> - Sustituya $CertThumbprint$ por la huella digital del certificado cargado en el bot en las siguientes líneas de la configuración.
>   ```xml
>      <Setting name="DefaultCertificate" value="$CertThumbprint$" />
>      <Certificate name="Default" thumbprint="$CertThumbprint$" thumbprintAlgorithm="sha1" />
>   ```

## <a name="publish-the-bot-from-visual-studio"></a>Publicación del bot desde Visual Studio
### <a name="step-1-launch-the-microsoft-azure-publishing-wizard-in-visual-studio"></a>Paso 1: Iniciar el Asistente para la publicación de Microsoft Azure en Visual Studio

Abra el proyecto en Visual Studio. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto del servicio en la nube y seleccione **Publicar**. Esto inicia el Asistente para la publicación de Microsoft Azure. Use sus credenciales para iniciar sesión en la suscripción adecuada.

![Haga clic con el botón derecho en el proyecto y elija Publicar para iniciar al Asistente para la publicación de Microsoft Azure.](../media/real-time-media-bot-publish-signin.png)

### <a name="step-2-publish-the-bot"></a>Paso 2: Publicar el bot

Haga clic en **Next**. Se abrirá la pestaña **Configuración**. Especifique Servicio en la nube, Entorno, Configuración de compilación y Configuración de servicio para implementar el bot.

![Haga clic en Siguiente y vaya a la pestaña Configuración](../media/real-time-media-bot-publish-settings.png)

Como alternativa, puede elegir **Configuración avanzada** y especificar la cuenta de almacenamiento para los registros de implementación (que puede usar para depurar problemas).

![Haga clic en la pestaña Configuración avanzada](../media/real-time-media-bot-publish-advanced-settings.png)

Compruebe la configuración en la pestaña **Resumen** y haga clic en **Publicar** para implementar el bot en Microsoft Azure.

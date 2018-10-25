---
title: Administración de un bot | Microsoft Docs
description: Aprenda a administrar un bot mediante el portal de Bot Service.
keywords: azure portal, administración de bots, probar en chat web, MicrosoftAppID, MicrosoftAppPassword, configuración de la aplicación
author: v-ducvo
ms.author: rstand
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: c1d8adef58c688b3678af7a80964021292d0b1d0
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997612"
---
# <a name="manage-a-bot"></a>Administración de un bot

En este tema se explica cómo administrar un bot mediante Azure Portal.

## <a name="bot-settings-overview"></a>Introducción a la configuración de bots

![Introducción a la configuración de bots](~/media/azure-manage-a-bot/overview.png)

En la hoja **Información general**, puede encontrar información general sobre el bot. Por ejemplo, puede ver el **identificador de suscripción**, el **plan de tarifa** y el **punto de conexión de mensajería** del bot.

## <a name="bot-management"></a>Administración de bots

 La mayoría de las opciones de administración del bot se pueden encontrar en la sección **ADMINISTRACIÓN DE BOTS**. Esta es una lista de opciones para ayudarle a administrar su bot.

![Administración de bots](~/media/azure-manage-a-bot/bot-management.png)

| Opción |  DESCRIPCIÓN |
| ---- | ---- |
| **Compilar** | La pestaña Compilar proporciona opciones para realizar cambios en el bot. Esta opción no está disponible para **Registration Only Bot** (Solo bot de registro). |
| **Probar en Chat en web** | Use el control de chat web integrado para ayudarle a probar rápidamente su bot. |
| **Analytics** | Si el análisis está activado para el bot, puede ver los datos de análisis que Application Insights ha recopilado para el bot. |
| **Channels** | Configure los canales que usa su bot para comunicarse con los usuarios. |
| **Configuración** | Administre varias configuraciones de perfiles de bots, como nombre para mostrar, análisis y punto de conexión de mensajería. |
| **Preparación para la voz** | Administre las conexiones entre la aplicación de LUIS y el servicio Bing Speech... |
| **Precios de Bot Service** | Administre el plan de tarifa del servicio de bots. |

## <a name="app-service-settings"></a>Configuración de App Service

![Configuración de App Service](~/media/azure-manage-a-bot/app-service-settings.png)

La hoja **Configuración de la aplicación** contiene información detallada sobre su bot, como el entorno del bot, el identificador, la clave de Application Insights, el identificador de aplicación de Microsoft y la contraseña de aplicación de Microsoft.

### <a name="microsoftappid-and-microsoftapppassword"></a>MicrosoftAppID y MicrosoftAppPassword

Puede encontrar los valores **MicrosoftAppID** y **MicrosoftAppPassword** de su bot en la hoja **Configuración de la aplicación**.

![Microsoft AppID y contraseña](~/media/azure-manage-a-bot/app-settings.png)

> [!NOTE]
> El servicio de bots **Registro de canales de bots** incluye un valor *MicrosoftAppID*, pero como no hay un servicio de aplicaciones asociado con este tipo de servicio, no hay una hoja **Configuración de la aplicación** para que consulte el valor *MicrosoftAppPassword*. Para obtener la contraseña, debe generar una. Para generar la contraseña para un **registro de canales de Bot**, consulte [Bot Channels Registration password](bot-service-quickstart-registration.md#bot-channels-registration-password) (Contraseña de registro de canales de bots).

## <a name="next-steps"></a>Pasos siguientes
Ahora que ha explorado la hoja de Bot Service en Azure Portal, aprenda a usar el Editor de código en línea para personalizar su bot.
> [!div class="nextstepaction"]
> [Uso del editor de código en línea](bot-service-build-online-code-editor.md)

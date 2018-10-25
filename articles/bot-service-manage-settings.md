---
title: Configuración de las opciones del bot | Microsoft Docs
description: Obtenga información sobre cómo configurar las distintas opciones para el bot mediante Azure Portal.
keywords: configure opciones de bot, nombre para mostrar, icono, Application Insights, hoja Configuración
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: 640e5675adce627ed14d732e529e9a86ec8b195f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996692"
---
# <a name="configure-bot-settings"></a>Configuración de las opciones del bot

La configuración del bot, como el nombre para mostrar, el icono y Application Insights, puede verse y modificarse en su hoja **Configuración**.

![Hoja de configuración del bot](~/media/bot-service-portal-configure-settings/bot-settings-blade.png)

A continuación, se encuentra la lista de campos de la hoja **Configuración**:

| Campo | DESCRIPCIÓN |
| :---  | :---        |
| Icono | Icono personalizado para identificar visualmente el bot en canales y como el icono de Skype, Cortana y otros servicios. Este icono debe estar en formato PNG y no debe tener un tamaño de más de 30 KB. Este valor puede cambiarse en cualquier momento. |
| Nombre para mostrar | El nombre de su bot en los canales y directorios. Este valor puede cambiarse en cualquier momento. Límite de 35 caracteres. |
| Bot handle (Identificador de bot) | Identificador único del bot. No se puede cambiar este valor después de crear su bot con Bot Service. |
| Messaging endpoint (Punto de conexión de mensajería) | El punto de conexión para comunicarse con el bot. |
| Microsoft App ID (Id. de aplicación de Microsoft) | Identificador único del bot. No se puede cambiar este valor. Para generar una nueva contraseña, haga clic en el vínculo **Administrar**. |
| Application Insights Instrumentation key (Clave de instrumentación de Application Insights) | Clave única para la telemetría del bot. Copie la clave de Azure Application Insights en este campo si desea recibir telemetría del bot para este bot. Este valor es opcional. Esta clave se generará automáticamente para los bots creados en Azure Portal. Para obtener más detalles sobre este campo, consulte [Claves de App Insights](~/bot-service-resources-app-insights-keys.md). |
| Clave de API de Application Insights | Clave única para el análisis del bot. Copie la clave de API de Azure Application Insights en este campo si desea ver el análisis sobre su bot en el panel. Este valor es opcional. Para obtener más detalles sobre este campo, consulte [Claves de App Insights](~/bot-service-resources-app-insights-keys.md). |
| Application Insights Application ID (Id. de aplicación de Application Insights) | Clave única para el análisis del bot. Copie la clave de id. de Azure Application Insights en este campo si desea ver el análisis sobre su bot en el panel. Este valor es opcional. Esta clave se generará automáticamente para los bots creados en Azure Portal. Para obtener más detalles sobre este campo, consulte [Claves de App Insights](~/bot-service-resources-app-insights-keys.md). |

> [!NOTE]
> Después de haber cambiado la configuración del bot, haga clic en el botón **Guardar** situado en la parte superior de la hoja para guardar la nueva configuración del bot.

## <a name="next-steps"></a>Pasos siguientes
Ahora que ha aprendido cómo configurar las opciones para el servicio del bot, obtenga información sobre cómo configurar la preparación para la voz.
> [!div class="nextstepaction"]
> [Preparación para la voz](bot-service-manage-speech-priming.md)
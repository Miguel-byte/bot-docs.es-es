---
title: Análisis del bot | Microsoft Docs
description: Obtenga información sobre cómo usar la recopilación y el análisis de datos para mejorar su bot con análisis de Bot Framework.
keywords: análisis de bot, application insights, tráfico, latencia, integraciones, AppInsights
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/04/2018
ms.openlocfilehash: 324050c625f5d9666811f63191d783643816104c
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298688"
---
# <a name="bot-analytics"></a>Análisis del bot

Analytics es una extensión de [Application Insights](/azure/application-insights/app-insights-analytics). Application Insights proporciona datos del **nivel de servicio** y de instrumentación, como tráfico, latencia e integraciones. Analytics proporciona informes de **nivel de conversación** relativos a los datos del usuario, los mensajes y los canales.

## <a name="view-analytics-for-a-bot"></a>Visualización de análisis para un bot

Para acceder a Analytics, abra el bot en Azure Portal y haga clic en **Analytics**.

¿Demasiados datos? También puede [habilitar y configurar el muestreo](/azure/application-insights/app-insights-sampling) eb la instancia de Application Insights vinculada al bot. Así se reduce el tráfico y el almacenamiento de datos de telemetría, a la vez que se mantiene un análisis estadísticamente correcto.

### <a name="specify-channel"></a>Especificación del canal

Elija los canales que aparecen en los gráficos a continuación. Tenga en cuenta que si un bot no está habilitado en un canal, no habrá ningún dato de ese canal.

![Selección del canal](~/media/analytics-channels.png)

* Active la casilla para incluir un canal en el gráfico.
* Desmarque la casilla para quitar un canal del gráfico.

### <a name="specify-time-period"></a>Indicación del período de tiempo

El análisis está disponible para los últimos 90 días únicamente. La recopilación de datos comenzó cuando se habilitó Application Insights.

![Selección del período de tiempo](~/media/analytics-timepick.png)

Haga clic en el menú desplegable y, a continuación, haga clic en la cantidad de tiempo que se deben mostrar los gráficos.
Tenga en cuenta que, al cambiar el período de tiempo general, hará que el incremento de tiempo (eje X) en los gráficos cambié en consecuencia.

### <a name="grand-totals"></a>Totales generales

El número total de usuarios activos y de actividades enviadas y recibidas durante el período de tiempo especificado.
Los guiones `--` indican que no hay ninguna actividad.

### <a name="retention"></a>Retención

Retención realiza un seguimiento de cuántos usuarios que enviaron un mensaje volvieron más adelante y enviaron otro.
El gráfico abarca un plazo móvil de 10 días; los resultados no se ven afectados al cambiar el período de tiempo.

![Gráfico de retención](~/media/analytics-retention.png)

Tenga en cuenta que la fecha más reciente posible es dos días atrás: un usuario envió mensajes anteayer y luego *volvió* ayer.

### <a name="user"></a>Usuario

El gráfico Usuarios realiza un seguimiento de cuántos usuarios han accedido al bot con cada canal durante el período de tiempo especificado.

![Gráfico de usuarios](~/media/analytics-users.png)

* El gráfico de porcentaje muestra qué porcentaje de los usuarios utiliza cada canal.
* El gráfico de líneas indica cuántos usuarios accedieron al bot en un momento determinado.
* La leyenda del gráfico de líneas indica qué color representa a qué canal e incluye el número total de usuarios durante el período de tiempo especificado.

### <a name="activities"></a>Actividades

El gráfico Actividades realiza el seguimiento de cuántas actividades se han enviado y recibido con cada canal durante el período de tiempo especificado.

![gráfico de actividades](~/media/analytics-activities.png)

* El gráfico de porcentaje muestra qué porcentaje de los actividades se comunicó a través de cada canal.
* El gráfico de líneas indica cuántas actividades se enviaron y recibieron durante el período de tiempo especificado.
* La leyenda del gráfico de líneas indica qué color de línea representa a cada canal y el número total de actividades enviadas y recibidas en ese canal durante el período de tiempo especificado.

## <a name="enable-analytics"></a>Habilitar análisis

Analytics no está disponibles hasta que se haya habilitado y configurado Application Insights. Application Insights comenzará a recopilar los datos tan pronto como esté habilitado. Por ejemplo, si Application Insights se habilitó hace una semana para un bot de seis meses de antigüedad, solo habrá recopilado datos de una semana.

> [!NOTE]
> Analytics requiere tanto una suscripción a Azure como un [recurso](/azure/application-insights/app-insights-create-new-resource) de Application Insights.
Para acceder a Application Insights, abra el bot en [Azure Portal](https://portal.azure.com/) y haga clic en **Configuración**.

Application Insights se puede agregar al crear el recurso del bot.

También se puede crear un recurso de Application Insights más adelante y conectarlo al bot.

1. Cree un [recurso](/azure/application-insights/app-insights-create-new-resource) de Application Insights.
2. Abra el bot en el panel. Haga clic en **Configuración** y desplácese hacia abajo hasta la sección **Analytics**.
3. Escriba la información para conectar el bot a Application Insights. Todos los campos son obligatorios.

![Conexión de Insights](~/media/analytics-enable.png)

<!--Snip: As of 12/04/2018, parts of this appear to be out of date. However, ~/bot-service-resources-app-insights-keys.md appears to be up to date.

### AppInsights Instrumentation Key

To find this value, open the Application Insights resource for your bot and navigate to **Configure** > **Properties**.

### AppInsights API key

Provide an Azure App Insights API key. Learn how to [generate a new API key](https://dev.applicationinsights.io/documentation/Authorization/API-key-and-App-ID). Only **Read** permission is required.

### AppInsights Application ID

To find this value, open Application Insights and navigate to **Configure** > **API Access**.

/Snip-->

Para obtener más información sobre cómo ubicar estos valores, consulte [Claves de Application Insights](~/bot-service-resources-app-insights-keys.md).

## <a name="additional-resources"></a>Recursos adicionales
* [Claves de Application Insights](~/bot-service-resources-app-insights-keys.md)
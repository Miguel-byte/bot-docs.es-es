---
title: Solución de errores de HTTP 500 de bot | Microsoft Docs
description: Cómo solucionar errores de HTTP 500 en un bot implementado.
keywords: solución de problemas, HTTP 500, problemas
author: jonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/30/2019
ms.openlocfilehash: 93689b7cee1c89bd9a7079c15ddf6aa16fcacc26
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033081"
---
# <a name="troubleshoot-http-500-errors"></a>Solución de errores de HTTP 500

El primer paso para solucionar errores de HTTP 500 es habilitar Application Insights.

<!-- TODO: Add links back in once there's a fresh AppInsights sample.
The luis-with-appinsights ([C# sample](https://aka.ms/cs-luis-with-appinsights-sample) / [JS sample](https://aka.ms/js-luis-with-appinsights-sample)) and qna-with-appinsights ([C# sample](https://aka.ms/qna-with-appinsights) / [JS sample](https://aka.ms/js-qna-with-appinsights-sample)) samples demonstrate bots that support Azure Application Insights.
-->
Consulte los [datos de telemetría de análisis de conversaciones](https://aka.ms/botframeworkanalytics) para más información acerca de cómo agregar Application Insights a un bot existente.

## <a name="enable-application-insights-on-aspnet"></a>Habilitación de Application Insights en ASP.Net

Para una compatibilidad básica con Application Insights, consulte [Configuración de Application Insights para un sitio web de ASP.NET](https://docs.microsoft.com/azure/application-insights/app-insights-asp-net). Bot Framework (a partir de la versión v4.2) proporciona un nivel adicional de telemetría de Application Insights, pero no esta no es necesaria para diagnosticar errores de HTTP 500.

## <a name="enable-application-insights-on-nodejs"></a>Habilitación de Application Insights en Node.js

Para una compatibilidad básica de Application Insights, consulte [Supervisión de servicios y aplicaciones de Node.js con Application Insights](https://docs.microsoft.com/azure/azure-monitor/learn/nodejs-quick-start). Bot Framework (a partir de la versión v4.2) proporciona un nivel adicional de telemetría de Application Insights, pero no esta no es necesaria para diagnosticar errores de HTTP 500.

## <a name="query-for-exceptions"></a>Consulta de excepciones

El método más sencillo de analizar errores de HTTP con código de estado 500 es empezar por las excepciones.

Las consultas siguientes le indicarán las excepciones más recientes:

```sql
exceptions
| order by timestamp desc
| project timestamp, operation_Id, appName
```

En la primera consulta, seleccione algunos identificadores de operación y busque más información:

```sql
let my_operation_id = "d298f1385197fd438b520e617d58f4fb";
let union_all = () {
    union
    (traces | where operation_Id == my_operation_id),
    (customEvents | where operation_Id == my_operation_id),
    (requests | where operation_Id == my_operation_id),
    (dependencies | where operation_Id  == my_operation_id),
    (exceptions | where operation_Id == my_operation_id)
};

union_all
    | order by timestamp desc
```

Si solo tiene `exceptions`, analice los detalles y vea si se corresponden a las líneas del código. Si solo ve excepciones procedentes del conector de canal (`Microsoft.Bot.ChannelConnector`), consulte [No hay eventos de Application Insights](#no-application-insights-events) para asegurarse de que Application Insights se ha configurado correctamente y que el código está registrando eventos.

## <a name="no-application-insights-events"></a>No hay eventos de Application Insights

Si recibe errores 500 y no existen más eventos del bot en Application Insights compruebe lo siguiente:

### <a name="ensure-bot-runs-locally"></a>Asegúrese de que el bot se ejecuta localmente

Primero, asegúrese de que el bot se ejecuta localmente con el emulador.

### <a name="ensure-configuration-files-are-being-copied-net-only"></a>Asegúrese de que se están copiando los archivos de configuración (solo para .NET)

Asegúrese de que `appsettings.json` y cualquier otro archivo de configuración se empaquetan correctamente durante el proceso de implementación.

#### <a name="application-assemblies"></a>Ensamblados de aplicación

Asegúrese de que los ensamblados de Application Insights se han empaquetado correctamente durante el proceso de implementación.

- Microsoft.ApplicationInsights
- Microsoft.ApplicationInsights.TraceListener
- Microsoft.AI.Web
- Microsoft.AI.WebServer
- Microsoft.AI.ServeTelemetryChannel
- Microsoft.AI.PerfCounterCollector
- Microsoft.AI.DependencyCollector
- Microsoft.AI.Agent.Intercept

Asegúrese de que `appsettings.json` y cualquier otro archivo de configuración se empaquetan correctamente durante el proceso de implementación.

#### <a name="appsettingsjson"></a>appsettings.json

En el archivo `appsettings.json` asegúrese de que se ha establecido la clave de instrumentación.

## <a name="aspnet-web-apitabdotnetwebapi"></a>[ASP.NET Web API](#tab/dotnetwebapi)

```json
{
    "Logging": {
        "IncludeScopes": false,
        "LogLevel": {
            "Default": "Debug",
            "System": "Information",
            "Microsoft": "Information"
        },
        "Console": {
            "IncludeScopes": "true"
        }
    }
}
```

## <a name="aspnet-coretabdotnetcore"></a>[ASP.NET Core](#tab/dotnetcore)

```json
{
    "ApplicationInsights": {
        "InstrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
}
```

---

### <a name="verify-config-file"></a>Comprobación del archivo de configuración

Asegúrese de que hay una clave de Application Insights incluida en el archivo de configuración.

```json
{
    "ApplicationInsights": {
        "type": "appInsights",
        "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subscriptionId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "resourceGroup": "my resource group",
        "name": "my appinsights name",
        "serviceName": "my service name",
        "instrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "applicationId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "apiKeys": {},
        "id": ""
    }
},
```

### <a name="check-logs"></a>Compruebe los registros

ASP.Net y Node emiten registros en el nivel de servidor que se pueden inspeccionar.

#### <a name="set-up-a-browser-to-watch-your-logs"></a>Configuración de un explorador para inspeccionar los registros

1. Abra el bot en [Azure Portal](http://portal.azure.com/).
1. Abra la página **Configuración de App Service/Toda la configuración de App Service** para ver todos los valores de configuración del servicio.
1. Abra la página **Supervisión/Registros de diagnóstico** del servicio de aplicación.
   - Asegúrese de que la opción **Registro de la aplicación (sistema de archivos)** está habilitada. No olvide hacer clic en **Guardar** si cambia este valor.
1. Vaya a la página **Supervisión/Secuencia de registro**.
   - Seleccione **Registros del servidor web** y asegúrese de que ve un mensaje que le indica que está conectado. Debe tener un aspecto similar al siguiente:

     ```bash
     Connecting...
     2018-11-14T17:24:51  Welcome, you are now connected to log-streaming service.
     ```

     Mantenga esta ventana abierta.

#### <a name="set-up-browser-to-restart-your-bot-service"></a>Configuración del explorador para reiniciar el servicio de bot

1. En un explorador diferente, abra el bot en Azure Portal.
1. Abra la página **Configuración de App Service/Toda la configuración de App Service** para ver todos los valores de configuración del servicio.
1. Vaya a la página **Información general** del servicio de aplicación y haga clic en **Reiniciar**.
   - Se le pedirá confirmación. Seleccione **Sí**.
1. Vuelva a la primera ventana del explorador y mire los registros.
1. Compruebe que ha recibido los nuevos registros.
   - Si no hay ninguna actividad, vuelva a implementar el bot.
   - A continuación, vaya a la página **Registros de aplicación** y busque errores.

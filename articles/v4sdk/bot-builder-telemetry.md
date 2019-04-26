---
title: Adición de telemetría al bot | Microsoft Docs
description: Vea cómo integrar su bot con las nuevas características de telemetría.
keywords: telemetry, appinsights, monitor bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 75e12ab44915783c33c3b2ee10775cc6f00487bb
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2019
ms.locfileid: "59905038"
---
# <a name="add-telemetry-to-your-bot"></a>Adición de telemetría al bot

[!INCLUDE[applies-to](../includes/applies-to.md)]

En la versión 4.2 de Bot Framework SDK, se agregó el registro de datos de telemetría al producto.  Esto permite que las aplicaciones de bot envíen datos de eventos a servicios tales como Application Insights.

En este documento se indica cómo integrar su bot con las nuevas características de telemetría.  

## <a name="using-bot-configuration-option-1-of-2"></a>Mediante la configuración del bot (opción 1 de 2)
Hay dos métodos para configurar un bot.  En el primero, se da por hecho que está integrando con Application Insights.

El archivo de configuración del bot contiene metadatos sobre los servicios externos que el bot usa mientras se ejecuta.  Por ejemplo, aquí se almacenan los metadatos y la conexión de servicio de CosmosDB, Application Insights y Language Understanding (LUIS).   

Si desea almacenar Application Insights, para que no sea necesaria ninguna otra configuración específica de Application Insights adicional (por ejemplo, los inicializadores de datos de telemetría), pase el objeto de configuración del bot durante la inicialización.   Este es el método de inicialización más sencillo y configurará Application Insights para que comience el seguimiento de las solicitudes, las llamadas externas a otros servicios y los eventos relacionados entre los servicios.

```csharp
// ASP.Net Core - Startup.cs

public void ConfigureServices(IServiceCollection services)
{
     ...
     // Add Application Insights - pass in the bot configuration
     services.AddBotApplicationInsights(botConfig);
     ...
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
     app.UseBotApplicationInsights()
                 ...
                .UseDefaultFiles()
                .UseStaticFiles()
                .UseBotFramework();
                ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

**Sin Application Insights en la configuración del bot** ¿Qué ocurre si la configuración del bot no incluye Application Insights?  No hay problema; asume un cliente NULL como valor predeterminado que no procesa las llamadas de método.

**Varias instancias de Application Insights** ¿Tiene varias secciones de Application Insights en la configuración del bot?  Puede designar qué instancia del servicio Application Insights desea utilizar dentro de la configuración del bot.

```csharp
// ASP.Net Core - Startup.cs

public void ConfigureServices(IServiceCollection services)
{
     // Add Application Insights
     services.AddBotApplicationInsights(botConfig, "myAppInsights");
     ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

## <a name="overriding-the-telemetry-client-option-2-of-2"></a>Invalidación del cliente de telemetría (opción 2 de 2)

Si desea personalizar el cliente de Application Insights o quiere iniciar sesión en un servicio completamente independiente, tendrá que configurar el sistema de manera diferente.

**Modificación de la configuración de Application Insights**

```csharp

public void ConfigureServices(IServiceCollection services)
{
     ...
     // Create Application Insight Telemetry Client
     // with custom configuration.
     var telemetryClient = TelemetryClient(myCustomConfiguration)
     
     // Add Application Insights
     services.AddBotApplicationInsights(new BotTelemetryClient(telemetryClient), "InstrumentationKey");
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

**Uso de datos de telemetría personalizados** Si desea registrar los eventos de telemetría generados por Bot Framework en un sistema completamente independiente, cree una nueva clase derivada de la interfaz base y configúrela.  

```csharp
public void ConfigureServices(IServiceCollection services)
{
     ...
     // Create my IBotTelemetryClient-based logger
     var myTelemetryClient = MyTelemetryLogger();
     
     // Add Application Insights
     services.AddBotApplicationInsights(myTelemetryClient);
     ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

## <a name="add-custom-logging-to-your-bot"></a>Incorporación de un registro personalizado a su bot

Cuando el bot tenga configurada la compatibilidad con el nuevo registro de datos de telemetría, puede empezar a agregar datos de telemetría al bot.  `BotTelemetryClient` (en C#, `IBotTelemetryClient`) tiene varios métodos para registrar distintos tipos de eventos.  Elija el tipo de evento adecuado para aprovechar las ventajas de los informes existentes en Application Insights (si usa Application Insights).  Para escenarios generales, normalmente se usa `TraceEvent`.  Los datos registrados con `TraceEvent` se envían a la tabla `CustomEvent` en Kusto.

Si usa un diálogo en el bot, todos los objetos basados en el diálogo (incluidos los avisos) contendrán una nueva propiedad `TelemetryClient`.  Se trata de `BotTelemetryClient`, que permite realizar el registro.  Esto no se hace simplemente por comodidad; como veremos más adelante en este artículo, si se establece esta propiedad, `WaterfallDialogs` generará eventos.

### <a name="identifiers-and-custom-events"></a>Identificadores y eventos personalizados

Cuando se registran eventos en Application Insights, los eventos generados contienen propiedades predeterminadas que no tendrá que rellenar.  Por ejemplo, las propiedades `user_id` y `session_id` se encuentran en cada evento personalizado (generado con la API `TraceEvent`).  También se agregan `activitiId`, `activityType` y `channelId`.

>Nota: Los clientes de telemetría personalizados no proporcionan estos valores.

Propiedad |Type | Detalles
--- | --- | ---
`user_id`| `string` | [ChannelID](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id) + [From.Id](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#from)
`session_id`| `string`|  [ConversationID](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#conversation)
`customDimensions.activityId`| `string` | [Identificador de la actividad del bot](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#id)
`customDimensions.activityType` | `string` | [Tipo de actividad del bot](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id)
`customDimensions.channelId` | `string` |  [Identificador de canal de la actividad del bot ](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id)

## <a name="waterfalldialog-events"></a>Eventos WaterfallDialog

Además de generar sus propios eventos, el objeto `WaterfallDialog` del SDK ahora genera eventos. En la siguiente sección se describen los eventos generados desde Bot Framework. Al establecer la propiedad `TelemetryClient` en `WaterfallDialog`, estos eventos se almacenarán.

Aquí podemos ver cómo modificar un ejemplo (BasicBot) que usa un diálogo `WaterfallDialog` para registrar los eventos de telemetría.  BasicBot usa un patrón común en el lugar donde se coloca un diálogo `WaterfallDialog` dentro de un diálogo `ComponentDialog` (`GreetingDialog`).

```csharp
// IBotTelemetryClient is direct injected into our Bot
public BasicBot(BotServices services, UserState userState, ConversationState conversationState, IBotTelemetryClient telemetryClient)
...

// The IBotTelemetryClient passed to the GreetingDialog
...
Dialogs = new DialogSet(_dialogStateAccessor);
Dialogs.Add(new GreetingDialog(_greetingStateAccessor, telemetryClient));
...

// The IBotTelemetryClient to the WaterfallDialog
...
AddDialog(new WaterfallDialog(ProfileDialog, waterfallSteps) { TelemetryClient = telemetryClient });
...

```

Cuando el diálogo `WaterfallDialog` tiene un cliente `IBotTelemetryClient` configurado, comenzará a registrar los eventos.

### <a name="customevent-waterfallstart"></a>CustomEvent: "WaterfallStart" 

Cuando se inicia un diálogo WaterfallDialog, se registra un evento `WaterfallStart`.

- `user_id` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `session_id` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Este es el identificador de diálogo (cadena) que se pasa a la cascada.  Se puede considerar el "tipo de cascada")
- `customDimensions.InstanceID` (único para cada instancia del diálogo)

### <a name="customevent-waterfallstep"></a>CustomEvent: "WaterfallStep" 

Registra los pasos individuales de un diálogo en cascada.

- `user_id` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `session_id` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Este es el identificador de diálogo (cadena) que se pasa a la cascada.  Se puede considerar el "tipo de cascada")
- `customDimensions.StepName` (nombre del método o `StepXofY` si es una expresión lambda)
- `customDimensions.InstanceID` (único para cada instancia del diálogo)

### <a name="customevent-waterfalldialogcomplete"></a>CustomEvent: "WaterfallDialogComplete"

Registra cuando se completa un diálogo en cascada.

- `user_id` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `session_id` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Este es el identificador de diálogo (cadena) que se pasa a la cascada.  Se puede considerar el "tipo de cascada")
- `customDimensions.InstanceID` (único para cada instancia del diálogo)

### <a name="customevent-waterfalldialogcancel"></a>CustomEvent: "WaterfallDialogCancel" 

Registra cuando se cancela un diálogo en cascada.

- `user_id` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `session_id` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([Desde el inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Este es el identificador de diálogo (cadena) que se pasa a la cascada.  Se puede considerar el "tipo de cascada")
- `customDimensions.StepName` (nombre del método o `StepXofY` si es una expresión lambda)
- `customDimensions.InstanceID` (único para cada instancia del diálogo)


## <a name="events-generated-by-the-bot-framework-service"></a>Eventos generados por el servicio Bot Framework

Además de `WaterfallDialog`, que genera eventos desde el código del bot, el servicio Bot Framework Channel también registra eventos.  Esto ayuda a diagnosticar problemas con los canales o errores de bot generales.

### <a name="customevent-activity"></a>CustomEvent: "Activity"
**Registrado de:** servicio Channel Registrado por el servicio Channel cuando se recibe un mensaje.

### <a name="exception-bot-errors"></a>Excepción: "Bot Errors"
**Registrado de:** servicio Channel Registrado por el servicio Channel cuando una llamada al bot devuelve una respuesta HTTP distinta de 2XX.

## <a name="additional-events"></a>Eventos adicionales

[Enterprise Template](https://github.com/Microsoft/AI/tree/master/templates/Enterprise-Template) es un código fuente abierto que se puede copiar libremente.  Contiene diversos componentes que se pueden volver a usar y modificar para adaptarlos a sus necesidades de informes.

### <a name="customevent-botmessagereceived"></a>CustomEvent: BotMessageReceived 
**Registrado de:** TelemetryLoggerMiddleware (**ejemplo de Enterprise**)

Registra cuando el bot recibe un mensaje nuevo.

- UserID  ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ConversationID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Channel ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityType ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Text (opcional para información de identificación personal)
- FromId
- FromName
- RecipientId
- RecipientName
- ConversationId
- ConversationName
- Configuración regional

### <a name="customevent-botmessagesend"></a>CustomEvent: BotMessageSend 
**Registrado de:** TelemetryLoggerMiddleware (**ejemplo de Enterprise**)

Registra cuando el bot envía un mensaje.

- UserID  ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ConversationID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Channel ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityType ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ReplyToID
- Channel (canal de origen; por ejemplo, Skype, Cortana, Teams)
- RecipientId
- ConversationName
- Configuración regional
- Text (opcional para información de identificación personal)
- RecipientName (opcional para información de identificación personal)

### <a name="customevent-botmessageupdate"></a>CustomEvent: BotMessageUpdate
**Registrado de:** TelemetryLoggerMiddleware Registra cuando el bot actualiza un mensaje (poco frecuente)

### <a name="customevent-botmessagedelete"></a>CustomEvent: BotMessageDelete
**Registrado de:** TelemetryLoggerMiddleware Registra cuando el bot elimina un mensaje (poco frecuente)

### <a name="customevent-luisintentinentname"></a>CustomEvent: LuisIntent.INENTName 
**Registrado de:** TelemetryLuisRecognizer (**ejemplo de Enterprise**)

Registra los resultados del servicio LUIS.

- UserID  ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ConversationID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Channel ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityType ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Intención
- IntentScore
- Pregunta
- ConversationId
- SentimentLabel
- SentimentScore
- *Entidades LUIS*
- **NUEVO** DialogId

### <a name="customevent-qnamessage"></a>CustomEvent: QnAMessage
**Registrado de:** TelemetryQnaMaker (**ejemplo de Enterprise**)

Registra los resultados del servicio QnA Maker.

- UserID  ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ConversationID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Channel ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityType ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Nombre de usuario
- ConversationId
- OriginalQuestion
- Pregunta
- Respuesta
- Score (*opcional*: si encontramos conocimientos)

## <a name="querying-the-data"></a>Consultas de datos
Cuando se usa Application Insights, todos los datos relacionan entre sí (incluso de un servicio a otro).  Lo podemos comprobar consultando una solicitud correcta y viendo todos los eventos asociados para esa solicitud.  
Las consultas siguientes le indicarán las solicitudes más recientes:
```sql
requests 
| where timestamp > ago(3d) 
| where resultCode == 200
| order by timestamp desc
| project timestamp, operation_Id, appName
| limit 10
```

En la primera consulta, seleccione algunos identificadores `operation_Id` y, después, busque más información:

```sql
let my_operation_id = "<OPERATION_ID>";
let union_all = () {
    union
    (traces | where operation_Id == my_operation_id),
    (customEvents | where operation_Id == my_operation_id),
    (requests | where operation_Id == my_operation_id),
    (dependencies | where operation_Id  == my_operation_id),
    (exceptions | where operation_Id == my_operation_id)
};
union_all
    | order by timestamp asc
    | project itemType, name, performanceBucket
```

Esto le da el desglose cronológico de una sola solicitud, con el depósito de duración de cada llamada.
![Llamada de ejemplo](media/performance_query.png)

> Nota: La marca de tiempo del evento `customEvent` "Activity" no funciona porque estos eventos se registran de forma asincrónica.

## <a name="create-a-dashboard"></a>Creación de un panel

La manera más fácil de probar es creando un panel con la [página de implementación de plantilla de Azure Portal](https://portal.azure.com/#create/Microsoft.Template).  
- Haga clic en ["Cree su propia plantilla en el editor"].
- Copie y pegue uno de estos archivos .json que se proporcionan como ayuda para crear el panel:
  - [Panel de estado del sistema](https://aka.ms/system-health-appinsights)
  - [Panel de estado de la conversación](https://aka.ms/conversation-health-appinsights)
- Haga clic en "Guardar"
- Rellene `Basics`: 
   - Suscripción: <your test subscription>
   - Grupos de recursos: <a test resource group>
   - Ubicación: <such as West US>
- Rellene `Settings`:
   - Nombre del componente de Insights: <como `core672so2hw`>
   - Grupo de recursos del componente de Insights: <como `core67`>
   - Nombre de panel: <como `'ConversationHealth'` o `SystemHealth`>
- Haga clic en `I agree to the terms and conditions stated above`
- Haga clic en `Purchase`
- Validación
   - Haga clic en [`Resource Groups`](https://ms.portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.Resources%2Fsubscriptions%2FresourceGroups).
   - Seleccione el grupo de recursos arriba (como `core67`).
   - Si no ve un nuevo recurso, busque en "Implementaciones" y compruebe si alguna tiene errores.
   - Normalmente, los errores se ven aquí:
     
```json
{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"BadRequest","message":"{\r\n \"error\": {\r\n \"code\": \"InvalidTemplate\",\r\n \"message\": \"Unable to process template language expressions for resource '/subscriptions/45d8a30e-3363-4e0e-849a-4bb0bbf71a7b/resourceGroups/core67/providers/Microsoft.Portal/dashboards/Bot Analytics Dashboard' at line '34' and column '9'. 'The template parameter 'virtualMachineName' is not found. Please see https://aka.ms/arm-template/#parameters for usage details.'\"\r\n }\r\n}"}]}
```

Para ver los datos, vaya a Azure Portal. Haga clic en **Panel** a la izquierda y seleccione el panel que creó en la lista desplegable.

## <a name="additional-resources"></a>Recursos adicionales
Puede hacer referencia a estos ejemplos que implementan datos de telemetría:
- C#
  - [LUIS con AppInsights](https://aka.ms/luis-with-appinsights-cs)
  - [QnA con AppInsights](https://aka.ms/qna-with-appinsights-cs)
- JS
  - [LUIS con AppInsights](https://aka.ms/luis-with-appinsights-js)
  - [QnA con AppInsights](https://aka.ms/qna-with-appinsights-js)


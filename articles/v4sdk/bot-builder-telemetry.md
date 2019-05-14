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
ms.openlocfilehash: 414417e3722e2d9063e1d177b534b6caa814c0db
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032441"
---
# <a name="add-telemetry-to-your-bot"></a>Adición de telemetría al bot

[!INCLUDE[applies-to](../includes/applies-to.md)]

En la versión 4.2 de Bot Framework SDK, se agregó el registro de datos de telemetría al producto.  Esto permite que las aplicaciones de bot envíen datos de eventos a servicios tales como Application Insights. La primera sección trata estos dos métodos, con características más amplias de telemetría después de eso.

En este documento se indica cómo integrar su bot con las nuevas características de telemetría.

## <a name="basic-telemetry-options"></a>Opciones de telemetría básicas

### <a name="basic-application-insights"></a>Application Insights básico
Hay dos métodos para configurar un bot.  En el primero, se da por hecho que está integrando con Application Insights.

El archivo de configuración contiene metadatos sobre los servicios externos que el bot usa mientras se ejecuta.  Por ejemplo, aquí se almacenan los metadatos y la conexión de servicio de CosmosDB, Application Insights y Language Understanding (LUIS).   

Si desea almacenar Application Insights, para que no sea necesaria ninguna otra configuración específica de Application Insights adicional (por ejemplo, los inicializadores de datos de telemetría), pase el objeto de configuración (normalmente `IConfiguration`) durante la inicialización.   Este es el método de inicialización más sencillo y configurará Application Insights para que comience el seguimiento de las solicitudes, las llamadas externas a otros servicios y los eventos relacionados entre los servicios.

Deberá agregar el paquete de NuGet **Microsoft.Bot.Builder.Integration.ApplicationInsights.Core**.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs**
```csharp
public void ConfigureServices(IServiceCollection services)
{
     ...
     // Add Application Insights - pass in the bot configuration
     services.AddBotApplicationInsights(<your IConfiguration variable name - likely "config">);
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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(<your configuration variable name - likely "config">);
```

---

### <a name="overriding-the-telemetry-client"></a>Invalidación del cliente de telemetría

Si desea personalizar el cliente de Application Insights o quiere iniciar sesión en un servicio completamente independiente, tendrá que configurar el sistema de manera diferente. Descargue el paquete `Microsoft.Bot.Builder.ApplicationInsights` desde nuget o use npm para instalar `botbuilder-applicationinsights`. La clave de instrumentación se puede encontrar en Azure Portal.

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

**Uso de datos de telemetría personalizados** Si desea registrar los eventos de telemetría generados por Bot Framework en un sistema completamente independiente, cree una nueva clase derivada de la interfaz base y configúrela.  

```csharp
public void ConfigureServices(IServiceCollection services)
{
     ...
     // Create my IBotTelemetryClient-based logger
     var myTelemetryClient = MyTelemetryLogger();
     
     // Add Application Insights
     services.AddSingleton(myTelemetryClient);
     ...
}
```

### <a name="add-custom-logging-to-your-bot"></a>Incorporación de un registro personalizado a su bot

Cuando el bot tenga configurada la compatibilidad con el nuevo registro de datos de telemetría, puede empezar a agregar datos de telemetría al bot.  `BotTelemetryClient` (en C#, `IBotTelemetryClient`) tiene varios métodos para registrar distintos tipos de eventos.  Elija el tipo de evento adecuado para aprovechar las ventajas de los informes existentes en Application Insights (si usa Application Insights).  Para escenarios generales, normalmente se usa `TraceEvent`.  Los datos registrados con `TraceEvent` se envían a la tabla `CustomEvent` en Kusto.

Si usa un diálogo en el bot, todos los objetos basados en el diálogo (incluidos los avisos) contendrán una nueva propiedad `TelemetryClient`.  Se trata de `BotTelemetryClient`, que permite realizar el registro.  Esto no se hace simplemente por comodidad; como veremos más adelante en este artículo, si se establece esta propiedad, `WaterfallDialogs` generará eventos.

#### <a name="identifiers-and-custom-events"></a>Identificadores y eventos personalizados

Cuando se registran eventos en Application Insights, los eventos generados contienen propiedades predeterminadas que no tendrá que rellenar.  Por ejemplo, las propiedades `user_id` y `session_id` se encuentran en cada evento personalizado (generado con la API `TraceEvent`).  También se agregan `activitiId`, `activityType` y `channelId`.

>Nota: Los clientes de telemetría personalizados no proporcionan estos valores.

Propiedad |Type | Detalles
--- | --- | ---
`user_id`| `string` | [ChannelID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id) + [From.Id](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)
`session_id`| `string`|  [ConversationID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation)
`customDimensions.activityId`| `string` | [Identificador de la actividad del bot](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id)
`customDimensions.activityType` | `string` | [Tipo de actividad del bot](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)
`customDimensions.channelId` | `string` |  [Identificador de canal de la actividad del bot ](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)

## <a name="in-depth-telemetry"></a>Telemetría detallada

Hay tres nuevos componentes que se han agregado al SDK en la versión 4.4.  Todos los componentes registran mediante la interfaz `IBotTelemetryClient` (o `BotTelemetryClient` en node.js), que se puede reemplazar por una implementación personalizada.

- Un componente de middleware de Bot Framework (*TelemetryLoggerMiddleware*) que registrará cuándo se reciben, envían, actualizan o eliminan los mensajes. Se puede invalidar para un registro personalizado.
- Clase *LuisRecognizer*.  Se puede invalidar para un registro personalizado de dos maneras: por invocación (agregar o reemplazar propiedades) o por clases derivadas.
- Clase *QnAMaker*.  Se puede invalidar para un registro personalizado de dos maneras: por invocación (agregar o reemplazar propiedades) o por clases derivadas.

### <a name="telemetry-middleware"></a>Middleware de telemetría

|C#  | JavaScript |
|:-----|:------------|
|**Microsoft.Bot.Builder.TelemetryLoggerMiddleware** | **botbuilder-core** |

#### <a name="out-of-box-usage"></a>Uso directo

TelemetryLoggerMiddleware es un componente de Bot Framework que se puede agregar sin modificaciones y realizará un registro que permite informes listos para usarse que se suministran con Bot Framework SDK. 

```csharp
var telemetryClient = sp.GetService<IBotTelemetryClient>();
var telemetryLogger = new TelemetryLoggerMiddleware(telemetryClient, logPersonalInformation: true);
options.Middleware.Add(telemetryLogger);  // Add to the middleware collection
```

#### <a name="adding-properties"></a>Adición de propiedades
Si decide agregar propiedades adicionales, se puede derivar la clase TelemetryLoggerMiddleware.  Por ejemplo, si desea agregar la propiedad "MyImportantProperty" al evento `BotMessageReceived`.  `BotMessageReceived` se registra cuando el usuario envía un mensaje al bot.  La adición de la propiedad adicional se puede realizar de la siguiente manera:

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    public Task OnReceiveActivityAsync(
                  Activity activity,
                  CancellationToken cancellation)
    {
        // Fill in the "standard" properties for BotMessageReceived
        // and add our own property.
        var properties = FillReceiveEventProperties(activity, 
                    new Dictionary<string, string>
                    { {"MyImportantProperty", "myImportantValue" } } );
                    
        // Use TelemetryClient to log event
        TelemetryClient.TrackEvent(
                        TelemetryLoggerConstants.BotMsgReceiveEvent,
                        properties);
    }
    ...
}
```

Y en Startup, se agregaría la nueva clase:

```csharp
var telemetryLogger = new TelemetryLuisRecognizer(telemetryClient, logPersonalInformation: true);
options.Middleware.Add(telemetryLogger);  // Add to the middleware collection
```

#### <a name="completely-replacing-properties--additional-events"></a>Reemplazo completo de propiedades / Eventos adicionales

Si decide reemplazar completamente las propiedades que se registran, la clase `TelemetryLoggerMiddleware` se puede derivar (como anteriormente al extender propiedades).   De forma similar, el registro de nuevos eventos se realiza de la misma manera.

Por ejemplo, si quisiera reemplazar completamente las propiedades de `BotMessageSend` y enviar varios eventos, se muestra seguidamente cómo se podría realizar esto:

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    public Task<RecognizerResult> OnLuisRecognizeAsync(
                  Activity activity,
                  string dialogId = null,
                  CancellationToken cancellation)
    {
        // Override properties for BotMsgSendEvent
        var botMsgSendProperties = new Dictionary<string, string>();
        properties.Add("MyImportantProperty", "myImportantValue");
        // Log event
        TelemetryClient.TrackEvent(
                        TelemetryLoggerConstants.BotMsgSendEvent,
                        botMsgSendProperties);
                        
        // Create second event.
        var secondEventProperties = new Dictionary<string, string>();
        secondEventProperties.Add("activityId",
                                   activity.Id);
        secondEventProperties.Add("MyImportantProperty",
                                   "myImportantValue");
        TelemetryClient.TrackEvent(
                        "MySecondEvent",
                        secondEventProperties);
    }
    ...
}
```
Nota: Cuando no se registran las propiedades estándar, provocará que los informes listos para usarse incluidos con el producto dejen de funcionar.

#### <a name="events-logged-from-telemetry-middleware"></a>Eventos registrados desde el middleware de telemetría
[BotMessageSend](#customevent-botmessagesend)
[BotMessageReceived](#customevent-botmessagereceived)
[BotMessageUpdate](#customevent-botmessageupdate)
[BotMessageDelete](#customevent-botmessagedelete)

### <a name="telemetry-support-luis"></a>Compatibilidad de telemetría en LUIS 

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.Luis** | **botbuilder-ai** |

#### <a name="out-of-box-usage"></a>Uso directo
LuisRecognizer es un componente de Bot Framework existente y se puede habilitar la telemetría pasando una interfaz IBotTelemetryClient con `luisOptions`.  Puede invalidar las propiedades predeterminadas que se registran y registrar nuevos eventos según sea necesario.

Durante la construcción de `luisOptions`, se debe proporcionar un objeto `IBotTelemetryClient` para que esto funcione.

```csharp
var luisOptions = new LuisPredictionOptions(
      ...
      telemetryClient,
      false); // Log personal information flag. Defaults to false.

var client = new LuisRecognizer(luisApp, luisOptions);
```

#### <a name="adding-properties"></a>Adición de propiedades
Si decide agregar propiedades adicionales, se puede derivar la clase `LuisRecognizer`.  Por ejemplo, si desea agregar la propiedad "MyImportantProperty" al evento `LuisResult`.  `LuisResult` se registra cuando se realiza una llamada de predicción de LUIS.  La adición de la propiedad adicional se puede realizar de la siguiente manera:

```csharp
class MyLuisRecognizer : LuisRecognizer 
{
   ...
   override protected Task OnRecognizerResultAsync(
           RecognizerResult recognizerResult,
           ITurnContext turnContext,
           Dictionary<string, string> properties = null,
           CancellationToken cancellationToken = default(CancellationToken))
   {
       var luisEventProperties = FillLuisEventProperties(result, 
               new Dictionary<string, string>
               { {"MyImportantProperty", "myImportantValue" } } );
        
        TelemetryClient.TrackEvent(
                        LuisTelemetryConstants.LuisResultEvent,
                        luisEventProperties);
        ..
   }    
   ...
}
```

#### <a name="add-properties-per-invocation"></a>Adición de propiedades por invocación
A veces es necesario agregar propiedades adicionales durante la invocación:
```csharp
var additionalProperties = new Dictionary<string, string>
{
   { "dialogId", "myDialogId" },
   { "conversationInfo", "myConversationInfo" },
};

var result = await recognizer.RecognizeAsync(turnContext,
     additionalProperties,
     CancellationToken.None).ConfigureAwait(false);
```

#### <a name="completely-replacing-properties--additional-events"></a>Reemplazo completo de propiedades / Eventos adicionales
Si decide reemplazar completamente las propiedades que se registran, se puede derivar la clase `LuisRecognizer` (como anteriormente al extender propiedades).   De forma similar, el registro de nuevos eventos se realiza de la misma manera.

Por ejemplo, si quisiera reemplazar completamente las propiedades de `LuisResult` y enviar varios eventos, se muestra seguidamente cómo se podría realizar esto:

```csharp
class MyLuisRecognizer : LuisRecognizer
{
    ...
    override protected Task OnRecognizerResultAsync(
             RecognizerResult recognizerResult,
             ITurnContext turnContext,
             Dictionary<string, string> properties = null,
             CancellationToken cancellationToken = default(CancellationToken))
    {
        // Override properties for LuisResult event
        var luisProperties = new Dictionary<string, string>();
        properties.Add("MyImportantProperty", "myImportantValue");
        
        // Log event
        TelemetryClient.TrackEvent(
                        LuisTelemetryConstants.LuisResult,
                        luisProperties);
                        
        // Create second event.
        var secondEventProperties = new Dictionary<string, string>();
        secondEventProperties.Add("MyImportantProperty2",
                                   "myImportantValue2");
        TelemetryClient.TrackEvent(
                        "MySecondEvent",
                        secondEventProperties);
        ...
    }
    ...
}
```
Nota: Cuando no se registran las propiedades estándar, provocará que los informes listos para usarse de Application Insights incluidos con el producto dejen de funcionar.

#### <a name="events-logged-from-telemetryluisrecognizer"></a>Eventos registrados desde TelemetryLuisRecognizer
[LuisResult](#customevent-luisevent)

### <a name="telemetry-qna-recognizer"></a>Telemetría de QnA Recognizer

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.QnA** | **botbuilder-ai** |


#### <a name="out-of-box-usage"></a>Uso directo
La clase QnAMaker es un componente de Bot Framework existente que agrega dos parámetros de constructor adicionales que habilitan el registro que posibilita los informes listos para su uso que se suministran con Bot Framework SDK. El nuevo elemento `telemetryClient` hace referencia a una interfaz `IBotTelemetryClient` que realiza el registro.  

```csharp
var qna = new QnAMaker(endpoint, options, client, 
                       telemetryClient: telemetryClient,
                       logPersonalInformation: true);
```
#### <a name="adding-properties"></a>Adición de propiedades 
Si decide agregar propiedades adicionales, hay dos métodos para hacerlo: cuando las propiedades se deben agregar durante la llamada de QnA para recuperar las respuestas o bien la derivación desde la clase `QnAMaker`.  

El siguiente ejemplo muestra la derivación desde la clase `QnAMaker`.  En el ejemplo se muestra cómo agregar la propiedad "MyImportantProperty" al evento `QnAMessage`.  El evento `QnAMessage` se registra cuando se realiza una llamada `GetAnswers` de QnA.  Además, se registra un segundo evento "MySecondEvent".

```csharp
class MyQnAMaker : QnAMaker 
{
   ...
   protected override Task OnQnaResultsAsync(
                 QueryResult[] queryResults, 
                 ITurnContext turnContext, 
                 Dictionary<string, string> telemetryProperties = null, 
                 Dictionary<string, double> telemetryMetrics = null, 
                 CancellationToken cancellationToken = default(CancellationToken))
   {
            var eventData = await FillQnAEventAsync(queryResults, turnContext, telemetryProperties, telemetryMetrics, cancellationToken).ConfigureAwait(false);

            // Add my property
            eventData.Properties.Add("MyImportantProperty", "myImportantValue");

            // Log QnaMessage event
            TelemetryClient.TrackEvent(
                            QnATelemetryConstants.QnaMsgEvent,
                            eventData.Properties,
                            eventData.Metrics
                            );

            // Create second event.
            var secondEventProperties = new Dictionary<string, string>();
            secondEventProperties.Add("MyImportantProperty2",
                                       "myImportantValue2");
            TelemetryClient.TrackEvent(
                            "MySecondEvent",
                            secondEventProperties);       }    
    ...
}
```

#### <a name="adding-properties-during-getanswersasync"></a>Adición de propiedades durante GetAnswersAsync
Si tiene propiedades que se deben agregar en tiempo de ejecución, el método `GetAnswersAsync` puede proporcionar propiedades y métricas para agregar al evento.

Por ejemplo, si desea agregar `dialogId` al evento, lo puede hacer de modo similar al siguiente:
```csharp
var telemetryProperties = new Dictionary<string, string>
{
   { "dialogId", myDialogId },
};

var results = await qna.GetAnswersAsync(context, opts, telemetryProperties);
```
La clase `QnaMaker` proporciona la capacidad de reemplazar las propiedades, incluidas las propiedades de PersonalInfomation.

#### <a name="completely-replacing-properties--additional-events"></a>Reemplazo completo de propiedades / Eventos adicionales
Si decide reemplazar completamente las propiedades que se registran, se puede derivar la clase `TelemetryQnAMaker` (como anteriormente al extender propiedades).   De forma similar, el registro de nuevos eventos se realiza de la misma manera.

Por ejemplo, si quisiera reemplazar completamente las propiedades de `QnAMessage`, se muestra seguidamente cómo se podría realizar esto:

```csharp
class MyLuisRecognizer : TelemetryQnAMaker
{
    ...
    protected override Task OnQnaResultsAsync(
         QueryResult[] queryResults, 
         ITurnContext turnContext, 
         Dictionary<string, string> telemetryProperties = null, 
         Dictionary<string, double> telemetryMetrics = null, 
         CancellationToken cancellationToken = default(CancellationToken))
    {
        // Add properties from GetAnswersAsync
        var properties = telemetryProperties ?? new Dictionary<string, string>();
        // GetAnswerAsync properties overrides - don't add if already present.
        properties.TryAdd("MyImportantProperty", "myImportantValue");

        // Log event
        TelemetryClient.TrackEvent(
                           QnATelemetryConstants.QnaMsgEvent,
                            properties);
    }
    ...
}
```
Nota: Cuando no se registran las propiedades estándar, provocará que los informes listos para usarse incluidos con el producto dejen de funcionar.

#### <a name="events-logged-from-telemetryluisrecognizer"></a>Eventos registrados desde TelemetryLuisRecognizer
[QnAMessage](#customevent-qnamessage)


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

## <a name="events-generated-by-the-bot-framework-service"></a>Eventos generados por el servicio Bot Framework

Además de `WaterfallDialog`, que genera eventos desde el código del bot, el servicio Bot Framework Channel también registra eventos.  Esto ayuda a diagnosticar problemas con los canales o errores de bot generales.

### <a name="customevent-activity"></a>CustomEvent: "Activity"
**Registrado de:** servicio Channel Registrado por el servicio Channel cuando se recibe un mensaje.

### <a name="exception-bot-errors"></a>Excepción: "Bot Errors"
**Registrado de:** servicio Channel Registrado por el servicio Channel cuando una llamada al bot devuelve una respuesta HTTP distinta de 2XX.

## <a name="all-events-generated"></a>Todos los eventos generados

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

### <a name="customevent-botmessagereceived"></a>CustomEvent: BotMessageReceived 
Registra cuando el bot recibe un mensaje nuevo de un usuario.

Si no se reemplaza, este evento se registra desde `Microsoft.Bot.Builder.TelemetryLoggerMiddleware` con el método `Microsoft.Bot.Builder.IBotTelemetry.TrackEvent()`.

- Identificador de sesión  
  - Cuando se usa Application Insights, se registra desde `TelemetryBotIdInitializer` como el identificador de **sesión** (*Temeletry.Context.Session.Id*) utilizado en Application Insights.  
  - Corresponde al [Identificador de conversación](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation) tal como se define en el protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `session_id`.

- Identificador de usuario
  - Cuando se usa Application Insights, se registra desde `TelemetryBotIdInitializer` como el identificador de **usuario** (*Temeletry.Context.User.Id*) utilizado en Application Insights.  
  - El valor de esta propiedad es una combinación de las propiedades [Identificador de canal](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id) e [ de usuario](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) (concatenadas) tal como se define mediante el protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `user_id`.

- Identificador de actividad 
  - Cuando se usa Application Insights, se registra desde `TelemetryBotIdInitializer` como una propiedad en el evento.
  - Corresponde al [Identificador de actividad](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#Id) tal como se define en el protocolo de Bot Framework.
  - El nombre de la propiedad es `activityId`.

- Identificador de canal
  - Cuando se usa Application Insights, se registra desde `TelemetryBotIdInitializer` como una propiedad en el evento.  
  - Corresponde al [Identificador de canal](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `channelId`.

- ActivityType 
  - Cuando se usa Application Insights, se registra desde `TelemetryBotIdInitializer` como una propiedad en el evento.  
  - Corresponde al [Tipo de actividad](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#type) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `activityType`.

- Texto
  - Se registra **opcionalmente** cuando la propiedad `logPersonalInformation` se establece en `true`.
  - Corresponde al campo [Texto de actividad](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#text) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `text`.

- Speak

  - Se registra **opcionalmente** cuando la propiedad `logPersonalInformation` se establece en `true`.
  - Corresponde al campo [Texto hablado de actividad](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#speak) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `speak`.

  - 

- FromId
  - Corresponde al campo [Desde el identificador](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `fromId`.

- FromName
  - Se registra **opcionalmente** cuando la propiedad `logPersonalInformation` se establece en `true`.
  - Corresponde al campo [Desde el nombre](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `fromName`.

- RecipientId
  - Corresponde al campo [Desde el nombre](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `fromName`.

- RecipientName
  - Corresponde al campo [Desde el nombre](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `fromName`.

- ConversationId
  - Corresponde al campo [Desde el nombre](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `fromName`.

- ConversationName
  - Corresponde al campo [Desde el nombre](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `fromName`.

- Configuración regional
  - Corresponde al campo [Desde el nombre](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) del protocolo de Bot Framework.
  - El nombre de propiedad que se registra es `fromName`.

### <a name="customevent-botmessagesend"></a>CustomEvent: BotMessageSend 
**Registrado de:** TelemetryLoggerMiddleware 

Registra cuando el bot envía un mensaje.

- UserID  ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- SessionID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Channel ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityType ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ReplyToID
- RecipientId
- ConversationName
- Configuración regional
- RecipientName (opcional para información de identificación personal)
- Text (opcional para información de identificación personal)
- Speak (opcional para información de identificación personal)


### <a name="customevent-botmessageupdate"></a>CustomEvent: BotMessageUpdate
**Registrado de:** TelemetryLoggerMiddleware Registra cuando el bot actualiza un mensaje (poco frecuente)
- UserID  ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- SessionID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Channel ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityType ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- RecipientId
- ConversationId
- ConversationName
- Configuración regional
- Text (opcional para información de identificación personal)


### <a name="customevent-botmessagedelete"></a>CustomEvent: BotMessageDelete
**Registrado de:** TelemetryLoggerMiddleware Registra cuando el bot elimina un mensaje (poco frecuente)
- UserID  ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- SessionID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Channel ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityType ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- RecipientId
- ConversationId
- ConversationName

### <a name="customevent-luisevent"></a>CustomEvent: LuisEvent
**Registrado de:** LuisRecognizer

Registra los resultados del servicio LUIS.

- UserID  ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- SessionID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Channel ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityType ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ApplicationId
- Intención
- IntentScore
- Intent2 
- IntentScore2 
- FromId
- SentimentLabel
- SentimentScore
- Entities (como JSON)
- Question (opcional para información de identificación personal)

## <a name="customevent-qnamessage"></a>CustomEvent: QnAMessage
**Registrado de:** QnAMaker

Registra los resultados del servicio QnA Maker.

- UserID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- SessionID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityID ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Channel ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- ActivityType ([del inicializador de datos de telemetría](https://aka.ms/telemetry-initializer))
- Username (opcional para información de identificación personal)
- Question (opcional para información de identificación personal)
- MatchedQuestion
- QuestionId
- Respuesta
- Score
- ArticleFound

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


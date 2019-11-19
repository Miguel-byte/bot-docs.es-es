---
title: Adición de telemetría al bot | Microsoft Docs
description: Vea cómo integrar su bot con las nuevas características de telemetría.
keywords: telemetry, appinsights, monitor bot
author: WashingtonKayaker
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 07/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 524ffa37d1d089bfec01fa7b89a456ecdda719f9
ms.sourcegitcommit: 312a4593177840433dfee405335100ce59aac347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/12/2019
ms.locfileid: "73933711"
---
# <a name="add-telemetry-to-your-bot"></a>Adición de telemetría al bot

[!INCLUDE[applies-to](../includes/applies-to.md)]


En la versión 4.2 de SDK Bot Framework, se agregó el registro de datos de telemetría.  Esto permite que las aplicaciones de bot envíen datos de eventos a servicios de telemetría como [Application Insights](https://aka.ms/appinsights-overview). La telemetría ofrece información al bot mostrando las características que se usan con mayor frecuencia, permite detectar comportamientos no deseados y ofrece visibilidad sobre la disponibilidad, el rendimiento y el uso.

***Nota: En la versión 4.6, el método estándar para implementar la telemetría en un bot se ha actualizado con el fin de garantizar que la telemetría se registra correctamente cuando se usa un adaptador personalizado. Este artículo se ha actualizado para mostrar el método actualizado. Los cambios son compatibles con versiones anteriores y los bots que usan el método anterior seguirán registrando correctamente la telemetría.***


En este artículo, aprenderá a implementar la telemetría en el bot mediante Application Insights:

* El código necesario para conectar la telemetría con el bot y conectarse a Application Insights

* Habilitar la telemetría en los [diálogos](bot-builder-concept-dialog.md) del bot

* Habilitar la telemetría para capturar datos de uso de otros servicios como [Luis](bot-builder-howto-v4-luis.md) y [QnA Maker](bot-builder-howto-qna.md).

* Visualización de los datos de telemetría en Application Insights

## <a name="prerequisites"></a>Requisitos previos

* [Código de ejemplo de CoreBot](https://aka.ms/cs-core-sample)

* [Código de ejemplo de Application Insights](https://aka.ms/csharp-corebot-app-insights-sample)

* Una suscripción a [Microsoft Azure](https://portal.azure.com/)

* Una [clave de Application Insights](../bot-service-resources-app-insights-keys.md)

* Experiencia con [Application Insights](https://aka.ms/appinsights-overview)

> [!NOTE]
> El [código de ejemplo de Application Insights](https://aka.ms/csharp-corebot-app-insights-sample) se compiló a partir del [código de ejemplo de CoreBot](https://aka.ms/cs-core-sample). Este artículo le guía a través de la modificación del código de ejemplo de CoreBot para incorporar datos de telemetría. Si está trabajando con Visual Studio, tendrá el código de ejemplo de Application Insights cuando hayamos terminado.

## <a name="wiring-up-telemetry-in-your-bot"></a>Conexión de los datos de telemetría con el bot

Vamos a empezar con la [aplicación de ejemplo de CoreBot](https://aka.ms/cs-core-sample) y agregaremos el código necesario para integrar los datos de telemetría en cualquier bot. Esto permitirá a Application Insights empezar a realizar el seguimiento de solicitudes.

1. Abra la [aplicación de ejemplo de CoreBot](https://aka.ms/cs-core-sample) en Visual Studio.

2. Agregue el paquete NuGet `Microsoft.Bot.Builder.Integration.ApplicationInsights.Core `. Para más información sobre el uso de NuGet, consulte [Instalación y administración de paquetes en Visual Studio](https://aka.ms/install-manage-packages-vs):


3. Incluya las siguientes instrucciones en `Startup.cs`:
    ```csharp
    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.Bot.Builder.ApplicationInsights;
    using Microsoft.Bot.Builder.Integration.ApplicationInsights.Core;
    using Microsoft.Bot.Builder.Integration.AspNet.Core;
    ```

    Nota: Si continúa con la actualización del código de ejemplo de CoreBot, observará que la instrucción using de `Microsoft.Bot.Builder.Integration.AspNet.Core` ya existe en el ejemplo.

4. Incluya el siguiente código en el método `ConfigureServices()` en `Startup.cs`. Esto hará que los servicios de telemetría estén disponibles para su bot a través de la [inserción de dependencias (DI)](https://aka.ms/asp.net-core-dependency-interjection):
    ```csharp
    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        ...
        // Create the Bot Framework Adapter with error handling enabled.
        services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();

        // Add Application Insights services into service collection
        services.AddApplicationInsightsTelemetry();

        // Add the standard telemetry client
        services.AddSingleton<IBotTelemetryClient, BotTelemetryClient>();

        // Create the telemetry middleware to track conversation events
        services.AddSingleton<TelemetryLoggerMiddleware>();

        // Add the telemetry initializer middleware
        services.AddSingleton<IMiddleware, TelemetryInitializerMiddleware>();

        // Add telemetry initializer that will set the correlation context for all telemetry items
        services.AddSingleton<ITelemetryInitializer, OperationCorrelationTelemetryInitializer>();

        // Add telemetry initializer that sets the user ID and session ID (in addition to other bot-specific properties, such as activity ID)
        services.AddSingleton<ITelemetryInitializer, TelemetryBotIdInitializer>();
        ...
    }
    ```
    
    Nota: Si continúa con la actualización del código de ejemplo de CoreBot, observará que `services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();` ya existe. 

5. Indique al adaptador que use el código de middleware que se agregó al método `ConfigureServices()`. Puede hacerlo en `AdapterWithErrorHandler.cs` con el parámetro IMiddleware middleware de la lista de parámetros de los constructores y en la instrucción `Use(middleware);` del constructor como se indica aquí:
    ```csharp
    public AdapterWithErrorHandler(ICredentialProvider credentialProvider, ILogger<BotFrameworkHttpAdapter> logger, IMiddleware middleware, ConversationState conversationState = null)
            : base(credentialProvider)
    {
        ...

        Use(middleware);
    }
    ```

7. Agregue la clave de instrumentación de Application Insights en el archivo `appsettings.json`. El archivo `appsettings.json` contiene metadatos acerca de los servicios externos que usa el bot durante la ejecución. Por ejemplo, aquí se almacenan los metadatos y la conexión de servicio de CosmosDB, Application Insights y Language Understanding (LUIS). La adición al archivo `appsettings.json` debe estar en este formato:

    ```json
    {
        "MicrosoftAppId": "",
        "MicrosoftAppPassword": "",
        "ApplicationInsights": {
            "InstrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        }
    }
    ```
    Nota: Puede encontrar información detallada sobre cómo obtener la _clave de instrumentación de Application Insights_ en el artículo [Claves de Application Insights](../bot-service-resources-app-insights-keys.md).

En este punto, ya se ha realizado el trabajo preliminar para habilitar la telemetría mediante Application Insights.  Puede ejecutar el bot localmente mediante el emulador de bots y, a continuación, entrar en Application Insights para ver qué se está registrando como, por ejemplo, el tiempo de respuesta, el estado general de la aplicación y la información de ejecución general. 

## <a name="enabling--disabling-activity-event-and-personal-information-logging"></a>Habilitación o deshabilitación del registro de información personal y de eventos de actividades

### <a name="enabling-or-disabling-activity-logging"></a>Habilitación o deshabilitación del registro de actividades

De forma predeterminada, `TelemetryInitializerMiddleware` usará `TelemetryLoggerMiddleware` para registrar la telemetría cuando el bot envía o recibe actividades. El registro de actividades crea registros de eventos personalizados en el recurso de Application Insights.  Si lo desea, puede deshabilitar el registro de eventos de actividades si establece `logActivityTelemetry` en False en `TelemetryInitializerMiddleware` antes de registrarlo en **Startup.cs**.

```cs
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add the telemetry initializer middleware
    services.AddSingleton<IMiddleware, TelemetryInitializerMiddleware>(sp =>
            {
                var httpContextAccessor = sp.GetService<IHttpContextAccessor>();
                var loggerMiddleware = sp.GetService<TelemetryLoggerMiddleware>();
                return new TelemetryInitializerMiddleware(httpContextAccessor, loggerMiddleware, logActivityTelemetry: false);
            });
    ...
}
```

### <a name="enable-or-disable-logging-personal-information"></a>Habilitación o deshabilitación del registro de información personal

De forma predeterminada, si está habilitado el registro de actividades, algunas propiedades de las actividades entrantes y salientes se excluyen del registro, ya que es probable que contengan información personal, como el nombre de usuario y el texto de la actividad. Puede optar por incluir estas propiedades en el registro realizando el siguiente cambio en **Startup.cs** al registrar `TelemetryLoggerMiddleware`.

```cs
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add the telemetry initializer middleware
    services.AddSingleton<TelemetryLoggerMiddleware>(sp =>
            {
                var telemetryClient = sp.GetService<IBotTelemetryClient>();
                return new TelemetryLoggerMiddleware(telemetryClient, logPersonalInformation: true);
            });
    ...
}
```

A continuación, vamos a ver qué se debe incluir para agregar la funcionalidad de telemetría a los diálogos. Esto le permitirá obtener información adicional como, por ejemplo, qué diálogos ejecutar y las estadísticas de cada uno.

## <a name="enabling-telemetry-in-your-bots-dialogs"></a>Habilitar la telemetría en los diálogos del bot

 Siga los pasos que se indican a continuación para actualizar el ejemplo de CoreBot:

1.  En `MainDialog.cs`, agregue un nuevo campo TelemetryClient a la clase `MainDialog` y, a continuación, actualice la lista de parámetros de constructores para que incluya un parámetro `IBotTelemetryClient` y, finalmente, páselo a cada llamada al método `AddDialog()`.


    * Agregue el parámetro `IBotTelemetryClient telemetryClient` al constructor de clases MainDialog y asígnelo al campo `TelemetryClient`:

        ```csharp
        public MainDialog(IConfiguration configuration, ILogger<MainDialog> logger, IBotTelemetryClient telemetryClient)
            : base(nameof(MainDialog))
        {
            TelemetryClient = telemetryClient;

            ...

        }
        ```

    * Cuando agregue cada diálogo mediante el método `AddDialog`, establezca el valor `TelemetryClient`:

        ```cs
        public MainDialog(IConfiguration configuration, ILogger<MainDialog> logger, IBotTelemetryClient telemetryClient)
            : base(nameof(MainDialog))
        {
            ...

            AddDialog(new TextPrompt(nameof(TextPrompt))
            {
                TelemetryClient = telemetryClient,
            });

            ...
        }
        ```

    * Los diálogos en cascada generan eventos independientemente de otros diálogos. Establezca la propiedad `TelemetryClient` después de la lista de los pasos de la cascada:

        ```csharp
        // The IBotTelemetryClient to the WaterfallDialog
        AddDialog(new WaterfallDialog(
            nameof(WaterfallDialog),
            new WaterfallStep[]
        {
            IntroStepAsync,
            ActStepAsync,
            FinalStepAsync,
        })
        {
            TelemetryClient = telemetryClient,
        });

        ```

2. En `DialogExtensions.cs`, establezca la propiedad `TelemetryClient` del objeto `dialogSet` del método `Run()`:


    ```csharp
    public static async Task Run(this Dialog dialog, ITurnContext turnContext, IStatePropertyAccessor<DialogState> accessor, CancellationToken cancellationToken = default(CancellationToken))
    {
        ...

        dialogSet.TelemetryClient = dialog.TelemetryClient;

        ...
        
    }

    ```

3. En `BookingDialog.cs` use el mismo proceso que se usa al actualizar `MainDialog.cs` para habilitar la telemetría en los cuatro diálogos agregados a este archivo. No olvide agregar el campo `TelemetryClient` a la clase BookingDialog y el parámetro `IBotTelemetryClient telemetryClient` al constructor BookingDialog.


> [!TIP] 
> Si continúa y actualiza el código de ejemplo de CoreBot, puede hacer referencia al [código de ejemplo de Application Insights](https://aka.ms/csharp-corebot-app-insights-sample) si le surge cualquier problema.

Eso es todo lo que hay que hacer para agregar la telemetría a los diálogos del bot. En este punto, si ha ejecutado el bot, debería ver que se están registrando cosas en Application Insights. Sin embargo, si tiene alguna tecnología integrada como Luis y QnA Maker necesitará agregar `TelemetryClient` también a ese código.


## <a name="enabling-telemetry-to-capture-usage-data-from-other-services-like-luis-and-qna-maker"></a>Habilitar la telemetría para capturar datos de uso de otros servicios como Luis y QnA Maker

A continuación, implementaremos la funcionalidad de telemetría en el servicio LUIS. El servicio LUIS incluye un registro de telemetría integrado disponible, por lo que hay muy poco que deba hacer para empezar a obtener datos de telemetría de LUIS.  

En este ejemplo, solo es necesario proporcionar el cliente de telemetría de una manera similar a como se hizo con los diálogos. 

1. El parámetro _`IBotTelemetryClient telemetryClient`_ es necesario en el método `ExecuteLuisQuery()` de `LuisHelper.cs`:

    ```cs
    public static async Task<BookingDetails> ExecuteLuisQuery(IBotTelemetryClient telemetryClient, IConfiguration configuration, ILogger logger, ITurnContext turnContext, CancellationToken cancellationToken)
    ```

2. La clase `LuisPredictionOptions` le permite proporcionar parámetros opcionales para una solicitud de predicción de LUIS.  Para habilitar la telemetría, tendrá que establecer el parámetro `TelemetryClient` cuando cree el objeto `luisPredictionOptions` en `LuisHelper.cs`:

    ```cs
    var luisPredictionOptions = new LuisPredictionOptions()
    {
        TelemetryClient = telemetryClient,
    };

    var recognizer = new LuisRecognizer(luisApplication, luisPredictionOptions);
    ```

3. El último paso es pasar el cliente de telemetría a la llamada a `ExecuteLuisQuery` en el método `ActStepAsync()` de `MainDialog.cs`:

    ```cs
    await LuisHelper.ExecuteLuisQuery(TelemetryClient, Configuration, Logger, stepContext.Context, cancellationToken)
    ```

En este caso, debe tener un bot funcional que registre los datos de telemetría en Application Insights. Puede usar [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) para ejecutar el bot localmente. No debería observar ningún cambio en el comportamiento del bot, pero este estará registrando información en Application Insights. Interactúe con el bot mediante el envío de varios mensajes y, en la sección siguiente, revisaremos los resultados de la telemetría en Application Insights.

Para más información sobre cómo probar y depurar el bot, puede consultar los siguientes artículos:

 * [Depuración de un bot](../bot-service-debug-bot.md)
 * [Directrices de prueba y depuración](bot-builder-testing-debugging.md)
 * [Depuración con el emulador](../bot-service-debug-emulator.md)


## <a name="visualizing-your-telemetry-data-in-application-insights"></a>Visualización de los datos de telemetría en Application Insights
Application Insights supervisa la disponibilidad, el rendimiento y el uso de la aplicación del bot, tanto si está hospedada en la nube como en un entorno local. Esta solución utiliza la eficaz plataforma de análisis de datos de Azure Monitor para proporcionarle información exhaustiva sobre las operaciones de la aplicación y diagnosticar errores sin esperar a que un usuario lo notifique. Hay varias maneras de ver los datos de telemetría recopilados por Application Insights. Dos de los métodos principales son mediante consultas y a través del panel. 

### <a name="querying-your-telemetry-data-in-application-insights-using-kusto-queries"></a>Consulta de los datos de telemetría en Application Insights mediante consultas de Kusto
Utilice esta sección como punto de partida para aprender a usar las consultas de registro en Application Insights. Muestra dos consultas útiles y proporciona vínculos a otra documentación con información adicional.

Para consultar los datos

1. Vaya a [Azure Portal](https://portal.azure.com).
2. Vaya a Application Insights. Para ello, lo más sencillo es hacer clic en **Supervisar > Aplicaciones** y buscarla allí. 
3. Una vez en Application Insights, puede hacer clic en _Registros (Analytics)_ en la barra de navegación.

    ![Registros (Analytics)](media/AppInsights-LogView.png)

4. Se abrirá la ventana de consulta.  Escriba la siguiente consulta y seleccione _Ejecutar_:

    ```sql
    customEvents
    | where name=="WaterfallStart"
    | extend DialogId = customDimensions['DialogId']
    | extend InstanceId = tostring(customDimensions['InstanceId'])
    | join kind=leftouter (customEvents | where name=="WaterfallComplete" | extend InstanceId = tostring(customDimensions['InstanceId'])) on InstanceId    
    | summarize starts=countif(name=='WaterfallStart'), completes=countif(name1=='WaterfallComplete') by bin(timestamp, 1d), tostring(DialogId)
    | project Percentage=max_of(0.0, completes * 1.0 / starts), timestamp, tostring(DialogId) 
    | render timechart
    ```
5. Esto devolverá el porcentaje de diálogos en cascada que se ejecutan hasta completarse.

    ![Registros (Analytics)](media/AppInsights-Query-PercentCompleteDialog.png)


> [!TIP]
> Puede anclar cualquier consulta al panel de Application Insights seleccionando el botón situado en la parte superior derecha de la hoja **Registros (Analytics)** . Simplemente seleccione el panel al que desea anclarlo y estará disponible la próxima vez que visite ese panel.


## <a name="the-application-insights-dashboard"></a>Panel de Application Insights

Cada vez que cree un recurso de Application Insights en Azure, se creará un nuevo panel y se le asociará automáticamente.  Puede ver ese panel seleccionando el botón situado en la parte superior de la hoja de Application Insights, etiquetado como **Panel de la aplicación**. 

![Vínculo Panel de la aplicación](media/Application-Dashboard-Link.png)


Como alternativa, para ver los datos, vaya a Azure Portal. Haga clic en **Panel** a la izquierda y seleccione el panel que desee en la lista desplegable.

Allí puede ver información predeterminada sobre el rendimiento del bot y sobre cualquier consulta adicional que haya anclado al panel.



## <a name="additional-information"></a>Información adicional

* [¿Qué es Application Insights?](https://aka.ms/appinsights-overview)

* [Uso de Búsqueda en Application Insights](https://aka.ms/search-in-application-insights)

* [Creación de paneles de indicadores clave de rendimiento (KPI) personalizados con Azure Application Insights](https://aka.ms/custom-kpi-dashboards-application-insights)


<!--
The easiest way to test is by creating a dashboard using [Azure portal's template deployment page](https://portal.azure.com/#create/Microsoft.Template).
- Click ["Build your own template in the editor"]
- Copy and paste either one of these .json file that is provided to help you create the dashboard:
  - [System Health Dashboard](https://aka.ms/system-health-appinsights)
  - [Conversation Health Dashboard](https://aka.ms/conversation-health-appinsights)
- Click "Save"
- Populate `Basics`: 
   - Subscription: <your test subscription>
   - Resource group: <a test resource group>
   - Location: <such as West US>
- Populate `Settings`:
   - Insights Component Name: <like `core672so2hw`>
   - Insights Component Resource Group: <like `core67`>
   - Dashboard Name:  <like `'ConversationHealth'` or `SystemHealth`>
- Click `I agree to the terms and conditions stated above`
- Click `Purchase`
- Validate
   - Click on [`Resource Groups`](https://ms.portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.Resources%2Fsubscriptions%2FresourceGroups)
   - Select your Resource Group from above (like `core67`).
   - If you don't see a new Resource, then look at "Deployments" and see if any have failed.
   - Here's what you typically see for failures:
     
```json
{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug-deployment for usage details.","details":[{"code":"BadRequest","message":"{\r\n \"error\": {\r\n \"code\": \"InvalidTemplate\",\r\n \"message\": \"Unable to process template language expressions for resource '/subscriptions/45d8a30e-3363-4e0e-849a-4bb0bbf71a7b/resourceGroups/core67/providers/Microsoft.Portal/dashboards/Bot Analytics Dashboard' at line '34' and column '9'. 'The template parameter 'virtualMachineName' is not found. Please see https://aka.ms/arm-template-structure for usage details.'\"\r\n }\r\n}"}]}
```
-->









<!--
## Additional information

### Customize your telemetry client

If you want to customize your telemetry to log into a separate service, you have to configure the system differently. If using Application Insights to do so, download the package `Microsoft.Bot.Builder.ApplicationInsights` via NuGet, or use npm to install `botbuilder-applicationinsights`. Details on getting the Application Insights keys can be found [here](../bot-service-resources-app-insights-keys.md). Otherwise, include what is necessary for logging to that service, then follow the section below.

**Use Custom Telemetry**
If you want to log telemetry events generated by the Bot Framework into a completely separate system, create a new class derived from the base interface `IBotTelemetryClient` and configure. Then, when adding your telemetry client as above, just inject your custom client. 

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add the telemetry client.
    services.AddSingleton<IBotTelemetryClient, CustomTelemetryClient>();
    ...
}
```

### Add custom logging to your bot

Once the Bot has the new telemetry logging support configured, you can begin adding telemetry to your bot.  The `BotTelemetryClient`(in C#, `IBotTelemetryClient`) has several methods to log distinct types of events.  Choosing the appropriate type of event enables you to take advantage of Application Insights existing reports (if you are using Application Insights).  For general scenarios `TraceEvent` is typically used.  The data logged using `TraceEvent` lands in the `CustomEvent` table in Kusto.

If using a Dialog within your Bot, every Dialog-based object (including Prompts) will contain a new `TelemetryClient` property.  This is the `BotTelemetryClient` that enables you to perform logging.  This is not just a convenience, we'll see later in this article if this property is set, `WaterfallDialogs` will generate events.

### Details of telemetry options

There are three main components available for your bot to log telemetry, and each component has customization available for logging your own events, which are discussed in this section. 

- A  [Bot Framework Middleware component](#telemetry-middleware) (*TelemetryLoggerMiddleware*) that will log when messages are received, sent, updated or deleted. You can override for custom logging.
- [*LuisRecognizer* class.](#telemetry-support-luis)  You can override for custom logging in two ways - per invocation (add/replace properties) or derived classes.
- [*QnAMaker*  class.](#telemetry-qnamaker)  You can override for custom logging in two ways - per invocation (add/replace properties) or derived classes.


All components log using the `IBotTelemetryClient`  (or `BotTelemetryClient` in node.js) interface which can be overridden with a custom implementation.


#### Telemetry Middleware

|C#  | JavaScript |
|:-----|:------------|
|**Microsoft.Bot.Builder.TelemetryLoggerMiddleware** | **botbuilder-core** |

##### Out of box usage

The TelemetryLoggerMiddleware is a Bot Framework component that can be added without modification, and it will perform logging that enables out of the box reports that ship with the Bot Framework SDK. 

```csharp
// Create the telemetry middleware to track conversation events
services.AddSingleton<IMiddleware, TelemetryLoggerMiddleware>();

// Create the Bot Framework Adapter with error handling enabled.
services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();
```

And in our adapter, we would specify the use of middleware:

```csharp
public AdapterWithErrorHandler(ICredentialProvider credentialProvider, ILogger<BotFrameworkHttpAdapter> logger, IMiddleware middleware, ConversationState conversationState = null)
           : base(credentialProvider)
{
    ...
    Use(middleware);
    ...
}
```

##### Adding properties
If you decide to add additional properties, the TelemetryLoggerMiddleware class can be derived.  For example, if you would like to add the property "MyImportantProperty" to the `BotMessageReceived` event.  `BotMessageReceived` is logged when the user sends a message to the bot.  Adding the additional property can be accomplished in the following way:

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    protected override Task OnReceiveActivityAsync(
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

In Startup, we would add the new class:

```csharp
// Create the telemetry middleware to track conversation events
services.AddSingleton<IMiddleware, MyTelemetryMiddleware>();
```

##### Completely replacing properties / Additional event(s)

If you decide to completely replace properties being logged, the `TelemetryLoggerMiddleware` class can be derived (like above when extending properties).   Similarly, logging new events is performed in the same way.

For example, if you would like to completely replace the`BotMessageSend` properties and send multiple events, the following demonstrates how this could be performed:

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    protected override Task OnReceiveActivityAsync(
                  Activity activity,
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
Note: When the standard properties are not logged, it will cause the out of box reports shipped with the product to stop working.

##### Events Logged from Telemetry Middleware
[BotMessageSend](bot-builder-telemetry-reference.md#customevent-botmessagesend)
[BotMessageReceived](bot-builder-telemetry-reference.md#customevent-botmessagereceived)
[BotMessageUpdate](bot-builder-telemetry-reference.md#customevent-botmessageupdate)
[BotMessageDelete](bot-builder-telemetry-reference.md#customevent-botmessagedelete)

#### Telemetry support LUIS 

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.Luis** | **botbuilder-ai** |

##### Out of box usage
The LuisRecognizer is an existing Bot Framework component, and telemetry can be enabled by passing a IBotTelemetryClient interface through `luisOptions`.  You can override the default properties being logged and log new events as required.

During construction of `luisOptions`, an `IBotTelemetryClient` object must be provided for this to work.

```csharp
var luisPredictionOptions = new LuisPredictionOptions()
{
    TelemetryClient = telemetryClient
};
var recognizer = new LuisRecognizer(luisApplication, luisPredictionOptions);
```

##### Adding properties
If you decide to add additional properties, the `LuisRecognizer` class can be derived.  For example, if you would like to add the property "MyImportantProperty" to the `LuisResult` event.  `LuisResult` is logged when a LUIS prediction call is performed.  Adding the additional property can be accomplished in the following way:

```csharp
class MyLuisRecognizer : LuisRecognizer 
{
   ...
   protected override Task OnRecognizerResultAsync(
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

##### Add properties per invocation
Sometimes it's necessary to add additional properties during the invocation:
```csharp
var additionalProperties = new Dictionary<string, string>
{
   { "dialogId", "myDialogId" },
   { "conversationInfo", "myConversationInfo" },
};

var result = await recognizer.RecognizeAsync(turnContext,
     additionalProperties).ConfigureAwait(false);
```

##### Completely replacing properties / Additional event(s)
If you decide to completely replace properties being logged, the `LuisRecognizer` class can be derived (like above when extending properties).   Similarly, logging new events is performed in the same way.

For example, if you would like to completely replace the`LuisResult` properties and send multiple events, the following demonstrates how this could be performed:

```csharp
class MyLuisRecognizer : LuisRecognizer
{
    ...
    protected override Task OnRecognizerResultAsync(
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
Note: When the standard properties are not logged, it will cause the Application Insights out of box reports shipped with the product to stop working.

##### Events Logged from TelemetryLuisRecognizer
[LuisResult](bot-builder-telemetry-reference.md#customevent-luisevent)

### Telemetry QnAMaker

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.QnA** | **botbuilder-ai** |


##### Out of box usage
The QnAMaker class is an existing Bot Framework component that adds two additional constructor parameters which enable logging that enable out of the box reports that ship with the Bot Framework SDK. The new `telemetryClient` references a `IBotTelemetryClient` interface which performs the logging.  

```csharp
var qna = new QnAMaker(endpoint, options, client, 
                       telemetryClient: telemetryClient,
                       logPersonalInformation: true);
```
##### Adding properties 
If you decide to add additional properties, there are two methods of doing this - when properties need to be added during the QnA call to retrieve answers or deriving from the `QnAMaker` class.  

The following demonstrates deriving from the `QnAMaker` class.  The example shows adding the property "MyImportantProperty" to the `QnAMessage` event.  The`QnAMessage` event is logged when a QnA `GetAnswers`call is performed.  In addition, we log a second event "MySecondEvent".

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

##### Adding properties during GetAnswersAsync
If you have properties that need to be added during runtime, the `GetAnswersAsync` method can provide properties and/or metrics to add to the event.

For example, if you want to add a `dialogId` to the event, it can be done like the following:
```csharp
var telemetryProperties = new Dictionary<string, string>
{
   { "dialogId", myDialogId },
};

var results = await qna.GetAnswersAsync(context, opts, telemetryProperties);
```
The `QnaMaker` class provides the capability of overriding properties, including PersonalInfomation properties.

##### Completely replacing properties / Additional event(s)
If you decide to completely replace properties being logged, the `TelemetryQnAMaker` class can be derived (like above when extending properties).   Similarly, logging new events is performed in the same way.

For example, if you would like to completely replace the`QnAMessage` properties, the following demonstrates how this could be performed:

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
Note: When the standard properties are not logged, it will cause the out of box reports shipped with the product to stop working.

##### Events Logged from TelemetryLuisRecognizer
[QnAMessage](bot-builder-telemetry-reference.md#customevent-qnamessage)

### All other events

A full list of events logged for your bot's telemetry can be found on the [telemetry reference page](bot-builder-telemetry-reference.md).

#### Identifiers and Custom Events

When logging events into Application Insights, the events generated contain default properties that you won't have to fill.  For example, `user_id` and `session_id`properties are contained in each Custom Event (generated with the `TraceEvent` API).  In addition, `activitiId`, `activityType` and `channelId` are also added.

> [!NOTE]
> Custom telemetry clients will not be provided these values.

Property |Type | Details
--- | --- | ---
`user_id`| `string` | [ChannelID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id) + [From.Id](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)
`session_id`| `string`|  [ConversationID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation)
`customDimensions.activityId`| `string` | [The bot activity ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id)
`customDimensions.activityType` | `string` | [The bot activity type](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)
`customDimensions.channelId` | `string` |  [The bot activity channel ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)
-->

---
title: Creación de middleware propio | Microsoft Docs
description: Aprenda a escribir su propio software intermedio.
keywords: software intermedio, software intermedio personalizado, cortocircuito, reserva, controladores de eventos
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 78c0afd6095cf9a10e04e1b0131d66ce6964f369
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000202"
---
# <a name="create-your-own-middleware"></a>Creación de middleware propio

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

El software intermedio le permite escribir complementos enriquecidos para los bots, que otros también pueden utilizar. Aquí se muestra cómo agregar e implementar software intermedio básico y, además, se explica cómo funciona. El SDK v4 le proporciona algún software intermedio, para tareas como la administración de estados, LUIS, QnAMaker y traducción. Eche un vistazo a Bot Builder SDK para [.NET](https://github.com/Microsoft/botbuilder-dotnet) o [JavaScript](https://github.com/Microsoft/botbuilder-js) para obtener más información.

## <a name="adding-middleware"></a>Adición de software intermedio

En el ejemplo siguiente, basado en el ejemplo de bot básico creado en la experiencia [Creación de un bot con Bot Service](~/bot-service-quickstart.md), se agregan dos fragmentos distintos de middleware a nuestros servicios con una instancia nueva de cada una de estas clases.

> [!IMPORTANT]
> Recuerde que el orden en que se agregan a las opciones determina el orden en que se ejecutan. Debe tener en cuenta cómo funcionará si usa más de un fragmento del software intermedio.

**Startup.cs**

# <a name="ctabcsaddmiddleware"></a>[C#](#tab/csaddmiddleware)

Agregue llamadas del método `options.Middleware.Add(new MyMiddleware());` a las opciones del servicio de bot para cada fragmento del software intermedio que desee agregar.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<HelloBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        options.Middleware.Add(new MyMiddleware());
        options.Middleware.Add(new MyOtherMiddleware());
    });
}
```
# <a name="javascripttabjsaddmiddleware"></a>[JavaScript](#tab/jsaddmiddleware)

Agregue `adapter.use(MyMiddleware());` a su adaptador para cada pieza de software intermedio que desee agregar.

```javascript
// Create adapter
const adapter = new botbuilder.BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

adapter.use(MyMiddleware());
adapter.use(MyOtherMiddleware());
```

---


## <a name="implementing-your-middleware"></a>Implementación del software intermedio

Cada fragmento del software intermedio se hereda de una interfaz de software intermedio y siempre implementa su controlador de procesamiento, que se ejecuta en cada actividad que se envía al bot. Para cada fragmento de software intermedio agregado, el controlador de procesamiento tiene la oportunidad de modificar el objeto de contexto o de realizar una tarea, como los registros, antes de permitir que otro software intermedio o la lógica del bot interactúen con el objeto de contexto a medida que continúa en la canalización.

# <a name="ctabcsetagoverwrite"></a>[C#](#tab/csetagoverwrite)

Cada fragmento de software intermedio se hereda de `IMiddleware` y siempre implementa `OnTurn()`.

**ExampleMiddleware.cs**
```csharp
public class MyMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {            
        // This simple middleware reports the request type and if we responded
        await context.SendActivity($"Request type: {context.Activity.Type}");
        
        await next();            

        // Report if any responses were recorded
        string response = context.Responded ? "yes" : "no";
        await context.SendActivity($"Responded?  {response}");
    }
}

public class MyOtherMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        // simple middleware to add an additional send activity
        await context.SendActivity($"My other middleware just saying Hi before the bot logic");

        await next();
    }
}

```

# <a name="javascripttabjsimplementmiddleware"></a>[JavaScript](#tab/jsimplementmiddleware)

Cada fragmento de software intermedio se hereda de `MiddlewareSet` y siempre implementa `onTurn()`.

**ExampleMiddleware.js**
```js
adapter.use({onTurn: async (context, next) =>{

    // This simple middleware reports the activity type and if we responded
    await context.sendActivity(`Activity type: ${context.activity.type}`); 
    await next();            

    // Report if any responses were recorded
    const response = context.activity.text ? "yes" : "no";
    await context.sendActivity(`Responded?  ${response}`);

}}, {onTurn: async (context, next) => {

     // simple middleware to add an additional send activity
     await context.sendActivity("My other middleware just saying Hi before the bot logic");

     await next();
}})
```

---

Una llamada a `next()` hace que la ejecución continúe hasta el siguiente fragmento de software intermedio. La posibilidad de elegir cuándo transferir la ejecución permite escribir código que se ejecuta **después** de que se haya ejecutado el resto de la pila del software intermedio. Se puede realizar una acción en el "borde final" del controlador de procesamiento, después de que se hayan ejecutado la lógica del bot y otro software intermedio.  En el ejemplo anterior, el primer software intermedio implementado hizo eso exactamente, elaborar informes si se respondía a este objeto de contexto, antes de volver a pasar la ejecución a la canalización.

## <a name="short-circuit-routing"></a>Enrutamiento de cortocircuito

En algunos casos, puede que desee detener cualquier procesamiento adicional de la actividad recibida, lo que podemos denominar como cortocircuito. Esto resulta útil para los casos en que el software intermedio se hace cargo totalmente de una solicitud, ofrece una respuesta sencilla a comandos específicos o puede controlar de otro modo una solicitud entrante sin que la lógica del bot necesite verla.

Se va a crear un fragmento de código del software intermedio que enviará una respuesta y evitará cualquier enrutamiento adicional de la solicitud cada vez que el usuario diga "ping":

# <a name="ctabcsmiddlewareshortcircuit"></a>[C#](#tab/csmiddlewareshortcircuit)
```cs
public class ExampleMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        var utterance = context.Activity?.AsMessageActivity()?.Text.Trim().ToLower();

        if (utterance == "ping") 
        {
            context.SendActivity("pong");
            return;
        } 
        else 
        {
            await next();
        }
    }
}
```
# <a name="javascripttabjsmiddlewareshortcircuit"></a>[JavaScript](#tab/jsmiddlewareshortcircuit)
```JavaScript
adapter.use({onTurn: async (context, next) =>{
    const utterance = (context.activity.text || '').trim().toLowerCase();
        if (utterance == "ping") 
        {
            await context.sendActivity("pong");
            return;
        } 
        else 
        {
            await next();
        }

}})

```

---

## <a name="fallback-processing"></a>Procesamiento de reserva

Otra cosa que puede que tenga que hacer es responder a una solicitud a la que aún no se haya respondido. Esto se realiza fácilmente con el borde final del controlador de procesamiento comprobando la propiedad `context.Responded`. Se va a crear un fragmento sencillo de software intermedio que dice automáticamente "No lo he entendido", en caso de que el bot no pueda controlar la solicitud:

# <a name="ctabcsfallback"></a>[C#](#tab/csfallback)
```cs
public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
{
    await next();

    if (!context.Responded) 
    {
        context.SendActivity("I didn't understand.");
    }
}
```
# <a name="javascripttabjsfallback"></a>[JavaScript](#tab/jsfallback)
```JavaScript
adapter.use({onTurn: async (context, next) =>{
    await next();

    if (!context.responded) 
    {
       await context.sendActivity("I didn't understand.");
    }

}})
```

---

> [!NOTE] 
> Esto podría no funcionar en todos los casos, como cundo otro software intermedio puede responder al usuario o cuando el bot recibe un mensaje correctamente pero no responde. Responder con "No lo entiendo" resultaría confuso para el usuario.



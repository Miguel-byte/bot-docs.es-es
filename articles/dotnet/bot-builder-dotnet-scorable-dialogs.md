---
title: Controladores de mensajes globales mediante diálogos puntuables
description: Cree diálogos más flexibles mediante elementos puntuables en Bot Framework SDK para .NET.
author: matthewshim-ms
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 101de0274c0daf4cbeb003703e299e0598f12a8a
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297296"
---
# <a name="global-message-handlers-using-scorables"></a>Controladores de mensajes globales mediante diálogos puntuables

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Los usuarios intentan acceder a ciertas funcionalidades dentro del bot con palabras como "ayuda", "cancelar" o "empezar de nuevo" en medio de una conversación cuando el bot está esperando una respuesta distinta. Puede diseñar su bot para controlar correctamente estas solicitudes mediante los diálogos puntuables.

Los diálogos puntuables supervisan todos los mensajes entrantes y determinan si un mensaje es accionable de alguna manera. A los mensajes puntuables se les asigna una puntuación entre [0 y 1] en cada diálogo puntuable. El diálogo puntuable que determina la puntuación más alta se agrega a la parte superior de la pila del diálogo y, a continuación, entrega la respuesta al usuario. Después de que el diálogo puntuable finalice su ejecución, la conversación continúa desde donde se dejó.

Los diálogos puntuables le permiten crear conversaciones más flexibles, ya que así los usuarios pueden "interrumpir" el flujo normal de conversación que se ve en los diálogos normales.

## <a name="create-a-scorable-dialog"></a>Creación de un diálogo puntuable

En primer lugar, defina un nuevo [diálogo](bot-builder-dotnet-dialogs.md). El código siguiente usa un diálogo que se deriva de la interfaz `IDialog`.

```cs
public class SampleDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        await context.PostAsync("This is a Sample Dialog which is Scorable. Reply with anything to return to the prior prior dialog.");

        context.Wait(this.MessageReceived);
    }

    private async Task MessageReceived(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        var message = await result;

        if ((message.Text != null) && (message.Text.Trim().Length > 0))
        {
            context.Done<object>(null);
        }
        else
        {
            context.Fail(new Exception("Message was not a string or was an empty string."));
        }
    }
}
```
Para hacer un diálogo puntuable, cree una clase que herede de la clase abstracta `ScorableBase`. En el siguiente código se muestra una clase `SampleScorable`.

```cs
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
using Microsoft.Bot.Builder.Internals.Fibers;
using Microsoft.Bot.Builder.Scorables.Internals;

public class SampleScorable : ScorableBase<IActivity, string, double>
{
    private readonly IDialogTask task;

    public SampleScorable(IDialogTask task)
    {
        SetField.NotNull(out this.task, nameof(task), task);
    }
}
```
La clase abstracta `ScorableBase` hereda desde la interfaz `IScorable`. Deberá implementar los siguientes métodos `IScorable` en la clase:

- `PrepareAsync` es el primer método que se llama en la instancia puntuable. Acepta la actividad de mensaje entrante y analiza y establece el estado del diálogo, el cual se pasa a todos los demás métodos de la interfaz `IScorable`.

```cs
protected override async Task<string> PrepareAsync(IActivity item, CancellationToken token)
{
        // TODO: insert your code here
}
```

- El método `HasScore` comprueba la propiedad de estado para determinar si el diálogo puntuable debe darle una puntuación al mensaje. Si devuelve false, se omitirá el mensaje en el diálogo puntuable.

```cs
protected override bool HasScore(IActivity item, string state)
{
        // TODO: insert your code here
}
```

- `GetScore` solo se desencadenará si `HasScore` devuelve true. Deberá aprovisionar la lógica en este método para determinar la puntuación para un mensaje entre 0 y 1.

```cs
protected override double GetScore(IActivity item, string state)
{
        // TODO: insert your code here
}
```
- En el método `PostAsync`, defina acciones básicas para que la clase puntuable las realice. Todos los diálogos puntuables supervisarán a los mensajes entrantes y asignarán puntuaciones a los mensajes válidos en función del método GetScore de puntuables. La clase puntuable que determina la puntuación más alta (entre 0 y 1,0) desencadenará el método `PostAsync` del puntuable.

```cs
protected override Task PostAsync(IActivity item, string state, CancellationToken token)
{
        //TODO: insert your code here
}
```

- Se realiza una llamada a `DoneAsync` una vez completado el proceso de puntuación. Use este método para eliminar los recursos con ámbito.

```cs
protected override Task DoneAsync(IActivity item, string state, CancellationToken token)
{
        //TODO: insert your code here
}
```

## <a name="create-a-module-to-register-the-iscorable-service"></a>Creación de un módulo para registrar el servicio IScorable

A continuación, defina `Module` que registrará a la clase `SampleScorable` como un componente. Esto aprovisionará al servicio `IScorable`.

```cs
public class GlobalMessageHandlersBotModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        base.Load(builder);

        builder
            .Register(c => new SampleScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();
    }
}
```
## <a name="register-the-module"></a>Registro del módulo  

El último paso del proceso es aplicar `SampleScorable` al contenedor de conversación del bot. Esto registrará el servicio puntuable dentro de la canalización de control de mensajes de Bot Framework. En el siguiente código se muestra cómo actualizar a `Conversation.Container` dentro de la inicialización de la aplicación del bot en **Global.asax.cs**:

```cs
public class WebApiApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
        this.RegisterBotModules();
        GlobalConfiguration.Configure(WebApiConfig.Register);
    }

    private void RegisterBotModules()
    {
        var builder = new ContainerBuilder();
        builder.RegisterModule(new ReflectionSurrogateModule());

        //Register the module within the Conversation container
        builder.RegisterModule<GlobalMessageHandlersBotModule>();

        builder.Update(Conversation.Container);
    }
}
```

## <a name="additional-resources"></a>Recursos adicionales
* [Ejemplo de controladores de mensajes globales](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers)
* [Ejemplo de Bot puntuable simple](https://github.com/Microsoft/BotFramework-Samples/tree/master/blog-samples/CSharp/ScorableBotSample)
* [Introducción a los diálogos](bot-builder-dotnet-dialogs.md)
* [AutoFac](https://autofac.org/)

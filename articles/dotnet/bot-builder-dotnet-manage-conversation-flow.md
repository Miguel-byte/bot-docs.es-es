---
title: Administración del flujo de conversación con diálogos | Microsoft Docs
description: Aprenda a modelar las conversaciones y a administrar el flujo de conversación mediante diálogos y Bot Framework SDK para Node.js.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 9e183c375e16a951ed77819ab982790944ac0c13
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298822"
---
# <a name="manage-conversation-flow-with-dialogs"></a>Administración del flujo de conversación con diálogos

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-manage-conversation-flow.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-manage-conversation-flow.md)

[!INCLUDE [Dialog flow example](../includes/snippet-dotnet-manage-conversation-flow-intro.md)]

En este artículo se describe cómo modelar este flujo de conversación con [diálogos](bot-builder-dotnet-dialogs.md) y Bot Framework SDK para. NET. 

## <a name="invoke-the-root-dialog"></a>Invocación del diálogo raíz

En primer lugar, el controlador de bot invoca el "diálogo raíz". En el ejemplo siguiente se muestra cómo conectar la llamada HTTP POST básica a un controlador y luego invocar el diálogo raíz. 

```cs
public class MessagesController : ApiController
{
    public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
    {
            // Redirect to the root dialog.
            await Conversation.SendAsync(activity, () => new RootDialog()); 
            ...
    }
}
```

## <a name="invoke-the-new-order-dialog"></a>Invocación del diálogo "Nuevo pedido"

A continuación, el diálogo raíz invoca el diálogo "Nuevo pedido". 

```cs
[Serializable]
public class RootDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        // Root dialog initiates and waits for the next message from the user. 
        // When a message arrives, call MessageReceivedAsync.
        context.Wait(this.MessageReceivedAsync); 
    }

    public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        var message = await result; // We've got a message!
        if (message.Text.ToLower().Contains("order"))
        {
            // User said 'order', so invoke the New Order Dialog and wait for it to finish.
            // Then, call ResumeAfterNewOrderDialog.
            await context.Forward(new NewOrderDialog(), this.ResumeAfterNewOrderDialog, message, CancellationToken.None);
        }
        // User typed something else; for simplicity, ignore this input and wait for the next message.
        context.Wait(this.MessageReceivedAsync);
    }

    private async Task ResumeAfterNewOrderDialog(IDialogContext context, IAwaitable<string> result)
    {
        // Store the value that NewOrderDialog returned. 
        // (At this point, new order dialog has finished and returned some value to use within the root dialog.)
        var resultFromNewOrder = await result;

        await context.PostAsync($"New order dialog just told me this: {resultFromNewOrder}");

        // Again, wait for the next message from the user.
        context.Wait(this.MessageReceivedAsync);
    }
}
```

## <a id="dialog-lifecycle"></a> Ciclo de vida de los diálogos

Cuando se invoca un diálogo, toma el control del flujo de la conversación. Todos los mensajes nuevos estarán sujetos a procesamiento por parte de ese diálogo hasta que se cierra o redirija a otro diálogo. 

En C#, puede usar `context.Wait()` para especificar la devolución de llamada para invocar la próxima vez que el usuario envíe un mensaje. Para cerrar un diálogo y quitarlo de la pila (de forma que se envíe de nuevo al usuario al diálogo anterior de la pila), use `context.Done()`. Todos los métodos de diálogo se deben terminar con `context.Wait()`, `context.Fail()`, `context.Done()`, o alguna directiva de redireccionamiento como `context.Forward()` o `context.Call()`. Un método de diálogo que no termina de alguna de estas maneras generará un error (porque el marco no sabe qué acción realizar la próxima vez que el usuario envíe un mensaje).

## <a name="passing-state-between-dialogs"></a>Paso del estado entre diálogos

Aunque puede almacenar el estado en Bot State, también puede pasar datos entre distintos diálogos mediante la sobrecarga del constructor de clases de diálogos.

```cs
[Serializable]
public class AgeDialog : IDialog<int>
{
    private string name;

    public AgeDialog(string name)
    {
        this.name = name;
    }
}
 ```

Código de diálogo de llamada, paso del valor de nombre del usuario.

```cs
private async Task NameDialogResumeAfter(IDialogContext context, IAwaitable<string> result)
{
    try
    {
        this.name = await result;
        context.Call(new AgeDialog(this.name), this.AgeDialogResumeAfter);
    }
    catch (TooManyAttemptsException)
    {
        await context.PostAsync("I'm sorry, I'm having issues understanding you. Let's try again.");
        await this.SendWelcomeMessageAsync(context);
    }
}
```

## <a name="sample-code"></a>Código de ejemplo 

Para obtener un ejemplo completo que muestra cómo administrar una conversación mediante diálogos en Bot Framework SDK para. NET, consulte [Ejemplo básico de varios diálogos](https://aka.ms/v3cs-MultiDialog-Sample) en GitHub.

## <a name="additional-resources"></a>Recursos adicionales

- [Diálogos](bot-builder-dotnet-dialogs.md)
- [Diseño y control del flujo de conversación](../bot-service-design-conversation-flow.md)
- [Ejemplo básico de varios diálogos (GitHub)](https://aka.ms/v3cs-MultiDialog-Sample)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Referencia de Bot Framework SDK para .NET</a>

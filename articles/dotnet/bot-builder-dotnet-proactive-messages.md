---
title: Envío de mensajes automáticos | Microsoft Docs
description: Aprenda a enviar mensajes automáticos con Bot Framework SDK para .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d86ee290ebf33dbfd13017c3fe882ecfdd9102e4
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225730"
---
# <a name="send-proactive-messages"></a>Envío de mensajes automáticos

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-proactive-messages.md)
> - [Node.js](../nodejs/bot-builder-nodejs-proactive-messages.md)

[!INCLUDE [Introduction to proactive messages - part 1](../includes/snippet-proactive-messages-intro-1.md)]

## <a name="types-of-proactive-messages"></a>Tipos de mensajes automáticos 

[!INCLUDE [Introduction to proactive messages - part 2](../includes/snippet-proactive-messages-intro-2.md)]

## <a name="send-an-ad-hoc-proactive-message"></a>Envío de un mensaje automático ad hoc

Los ejemplos de código siguientes muestran cómo enviar un mensaje automático ad hoc con Bot Framework SDK para .NET.

Para poder enviar un mensaje ad hoc a un usuario, el bot en primer lugar debe recopilar y almacenar información sobre el usuario de la conversación actual. 

```cs
public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
{
    var message = await result;
    
    // Extract data from the user's message that the bot will need later to send an ad hoc message to the user. 
    // Store the extracted data in a custom class "ConversationStarter" (not shown here).

    ConversationStarter.toId = message.From.Id;
    ConversationStarter.toName = message.From.Name;
    ConversationStarter.fromId = message.Recipient.Id;
    ConversationStarter.fromName = message.Recipient.Name;
    ConversationStarter.serviceUrl = message.ServiceUrl;
    ConversationStarter.channelId = message.ChannelId;
    ConversationStarter.conversationId = message.Conversation.Id;

    // (Save this information somewhere that it can be accessed later, such as in a database.)

    await context.PostAsync("Hello user, good to meet you! I now know your address and can send you notifications in the future.");
    context.Wait(MessageReceivedAsync);
}
```
> [!NOTE]
> Por motivos de simplicidad, este ejemplo no especifica cómo almacenar los datos de usuario. No importa cómo se almacenen los datos, siempre y cuando el bot pueda recuperarlos posteriormente.

Ahora que se han almacenado los datos, el bot puede recuperar los datos simplemente, crear el mensaje automático ad hoc y enviarlo. 

```cs
// Use the data stored previously to create the required objects.
var userAccount = new ChannelAccount(toId,toName);
var botAccount = new ChannelAccount(fromId, fromName);
var connector = new ConnectorClient(new Uri(serviceUrl));

// Create a new message.
IMessageActivity message = Activity.CreateMessageActivity();
if (!string.IsNullOrEmpty(conversationId) && !string.IsNullOrEmpty(channelId))  
{
    // If conversation ID and channel ID was stored previously, use it.
    message.ChannelId = channelId;
}
else
{
    // Conversation ID was not stored previously, so create a conversation. 
    // Note: If the user has an existing conversation in a channel, this will likely create a new conversation window.
    conversationId = (await connector.Conversations.CreateDirectConversationAsync( botAccount, userAccount)).Id;
}

// Set the address-related properties in the message and send the message.
message.From = botAccount;
message.Recipient = userAccount;
message.Conversation = new ConversationAccount(id: conversationId);
message.Text = "Hello, this is a notification";
message.Locale = "en-us";
await connector.Conversations.SendToConversationAsync((Activity)message);
```

> [!NOTE]
> Si el bot especifica un identificador de conversación que se almacenó previamente, es probable que el mensaje se entregue al usuario en la ventana de conversación existente en el cliente. Si el bot genera un nuevo identificador de conversación, el mensaje se entregará al usuario en una nueva ventana de conversación en el cliente, siempre que el cliente admita varias ventanas de conversación. 

## <a name="send-a-dialog-based-proactive-message"></a>Envío de un mensaje automático basado en diálogos

Los ejemplos de código siguientes muestran cómo enviar un mensaje automático basado en diálogos con Bot Framework SDK para .NET.

Para poder enviar un mensaje automático basado en diálogos a un usuario, el bot en primer lugar debe recopilar y guardar información de la conversación actual. 

```cs
public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
{
    var message = await result;
    
    // Store information about this specific point the conversation, so that the bot can resume this conversation later.
    var conversationReference = message.ToConversationReference();
    ConversationStarter.conversationReference = JsonConvert.SerializeObject(conversationReference);

    await context.PostAsync("Greetings, user! I now know how to start a proactive message to you."); 
    context.Wait(MessageReceivedAsync);
}
```

Cuando llega el momento de enviar el mensaje, el bot crea un diálogo y lo agrega a la parte superior de la pila del diálogo. El nuevo diálogo controla la conversación, entrega el mensaje automático, lo cierra y luego devuelve el control al diálogo anterior de la pila. 

```cs
// This will interrupt the conversation and send the user to SurveyDialog, then wait until that's done 
public static async Task Resume() 
{
    // Recreate the message from the conversation reference that was saved previously.
    var message = JsonConvert.DeserializeObject<ConversationReference>(conversationReference).GetPostToBotMessage(); 
    var client = new ConnectorClient(new Uri(message.ServiceUrl));

    // Create a scope that can be used to work with state from bot framework.
    using (var scope = DialogModule.BeginLifetimeScope(Conversation.Container, message))
    {
        var botData = scope.Resolve<IBotData>();
        await botData.LoadAsync(CancellationToken.None);

        // This is our dialog stack.
        var task = scope.Resolve<IDialogTask>();

        // Create the new dialog and add it to the stack.
        var dialog = new SurveyDialog();
        // interrupt the stack. This means that we're stopping whatever conversation that is currently happening with the user
        // Then adding this stack to run and once it's finished, we will be back to the original conversation
        task.Call(dialog.Void<object, IMessageActivity>(), null);
        
        await task.PollAsync(CancellationToken.None);

        // Flush the dialog stack back to its state store.
        await botData.FlushAsync(CancellationToken.None);        
    }
}
```
`SurveyDialog` controla la conversación hasta que termina. Cuando finaliza su tarea, llama a `context.Done` y se cierra, devolviendo el control al diálogo anterior. 

```cs
[Serializable]
public class SurveyDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        await context.PostAsync("Hello, I'm the survey dialog. I'm interrupting your conversation to ask you a question. Type \"done\" to resume");

        context.Wait(this.MessageReceivedAsync);
    }
    public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        if ((await result).Text == "done")
        {
            await context.PostAsync("Great, back to the original conversation!");
            context.Done(String.Empty); // Finish this dialog.
        }
        else
        {
            await context.PostAsync("I'm still on the survey until you type \"done\"");
            context.Wait(MessageReceivedAsync); // Not done yet.
        }
    }
}
```

## <a name="sample-code"></a>Código de ejemplo

Para obtener un ejemplo completo en el que se muestre cómo enviar mensajes automáticos con Bot Framework SDK para .NET, consulte <a href="https://aka.ms/proactive-messaging-cs-v3 " target="_blank">Ejemplo de mensajes automáticos</a> en GitHub. En el ejemplo de mensajes automáticos, <a href="https://aka.ms/proactive-sendmessage-cs-v3 " target="_blank">simpleSendMessage</a> muestra cómo enviar un mensaje automático ad hoc y <a href="https://aka.ms/proactive-newdialog-cs-v3 " target="_blank">startNewDialog</a> muestra cómo enviar un mensaje automático basado en diálogos. 

## <a name="additional-resources"></a>Recursos adicionales

- [Diseño y control del flujo de conversación](../bot-service-design-conversation-flow.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Referencia de Bot Framework SDK para .NET</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-proactiveMessages" target="_blank">Ejemplo de mensajes automáticos (GitHub)</a>


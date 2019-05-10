---
title: Realización de llamadas de audio con Skype | Microsoft Docs
description: Obtenga información sobre cómo realizar llamadas de audio con Skype con Bot Framework SDK para .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 72b2c42acb4743c67c63f158fa37c2bdd0e09ab9
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563926"
---
# <a name="conduct-audio-calls-with-skype"></a>Realización de llamadas de audio con Skype

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Introduction to conducting audio calls](../includes/snippet-audio-call-intro.md)]

La arquitectura de un bot que admita llamadas de audio es muy similar a la de un bot típico. Los ejemplos de código siguientes muestran cómo habilitar la compatibilidad con llamadas de audio mediante Skype con Bot Framework SDK para .NET. 

## <a name="enable-support-for-audio-calls"></a>Habilitación de la compatibilidad con llamadas de audio

Para habilitar un bot para que admita llamadas de audio, defina el elemento `CallingController`.

```cs
[BotAuthentication]
[RoutePrefix("api/calling")]
public class CallingController : ApiController
{
    public CallingController() : base()
    {
        CallingConversation.RegisterCallingBot(callingBotService => new IVRBot(callingBotService));
    }

    [Route("callback")]
    public async Task<HttpResponseMessage> ProcessCallingEventAsync()
    {
        return await CallingConversation.SendAsync(this.Request, CallRequestType.CallingEvent);
    }

    [Route("call")]
    public async Task<HttpResponseMessage> ProcessIncomingCallAsync()
    {
        return await CallingConversation.SendAsync(this.Request, CallRequestType.IncomingCall);
    }
}
```

> [!NOTE]
> Además del elemento `CallingController`, que admite llamadas de audio, un bot también puede contener un elemento `MessagesController` para admitir mensajes. Proporcionar ambas opciones permite a los usuarios interactuar con el bot de la manera que prefieran. <!-- docs on MessagesController are where? -->

##  <a name="answer-the-call"></a>Responder la llamada

La tarea `ProcessIncomingCallAsync` se ejecutará cada vez que un usuario inicia una llamada a este bot desde Skype.
El constructor registra la clase `IVRBot`, que tiene un controlador predefinido para el elemento `incomingCallEvent`.

La primera acción del flujo de trabajo debe determinar si el bot responde o rechaza la llamada entrante. Este flujo de trabajo le indica al bot que responda a la llamada entrante y, a continuación, reproduzca un mensaje de bienvenida. 

```cs
private Task OnIncomingCallReceived(IncomingCallEvent incomingCallEvent)
{
    this.callStateMap[incomingCallEvent.IncomingCall.Id] = new CallState(incomingCallEvent.IncomingCall.Participants);

    incomingCallEvent.ResultingWorkflow.Actions = new List<ActionBase>
    {
        new Answer { OperationId = Guid.NewGuid().ToString() },
        GetPromptForText(WelcomeMessage)
    };

    return Task.FromResult(true);
}
```

## <a name="after-the-bot-answers"></a>Después de que el bot responda

Si el bot responde a la llamada, las acciones posteriores especificadas en el flujo de trabajo indicarán a la **plataforma de llamadas para bots de Skype** que reproduzca un aviso, grabe audio, realice un reconocimiento de voz o recopile dígitos desde el panel de marcado. La acción final del flujo de trabajo podría ser finalizar la llamada. 

Este ejemplo de código define un controlador que establecerá un menú al finalizar el mensaje de bienvenida.

```cs
private Task OnPlayPromptCompleted(PlayPromptOutcomeEvent playPromptOutcomeEvent)
{
    var callState = this.callStateMap[playPromptOutcomeEvent.ConversationResult.Id];
    SetupInitialMenu(playPromptOutcomeEvent.ResultingWorkflow);
    return Task.FromResult(true);
}
```

El método `CreateIvrOptions` define el menú que se presentará al usuario.

```cs
private static Recognize CreateIvrOptions(string textToBeRead, int numberOfOptions, bool includeBack)
{
    if (numberOfOptions > 9)
    {
        throw new Exception("too many options specified");
    }

    var choices = new List<RecognitionOption>();

    for (int i = 1; i <= numberOfOptions; i++)
    {
        choices.Add(new RecognitionOption { Name = Convert.ToString(i), DtmfVariation = (char)('0' + i) });
    }

    if (includeBack)
    {
        choices.Add(new RecognitionOption { Name = "#", DtmfVariation = '#' });
    }

    var recognize = new Recognize
    {
        OperationId = Guid.NewGuid().ToString(),
        PlayPrompt = GetPromptForText(textToBeRead),
        BargeInAllowed = true,
        Choices = choices
    };

    return recognize;
}
```

La clase `RecognitionOption` define la respuesta de voz y la variación del doble tono multifrecuencia (DTMF) correspondiente. DTMF permite al usuario responder escribiendo los dígitos correspondientes en el teclado en lugar de hablar.

El método `OnRecognizeCompleted` procesa la selección del usuario y el parámetro de entrada `recognizeOutcomeEvent` contiene el valor de la selección del usuario.

```cs
private Task OnRecognizeCompleted(RecognizeOutcomeEvent recognizeOutcomeEvent)
{
    var callState = this.callStateMap[recognizeOutcomeEvent.ConversationResult.Id];
    ProcessMainMenuSelection(recognizeOutcomeEvent, callState);
    return Task.FromResult(true);
}
```

## <a name="support-natural-language"></a>Compatibilidad con el lenguaje natural
También se puede diseñar el bot para que admita respuestas de lenguaje natural. **Bing Speech API** permite al bot reconocer palabras de la respuesta hablada del usuario.

```cs
private async Task OnRecordCompleted(RecordOutcomeEvent recordOutcomeEvent)
{
    recordOutcomeEvent.ResultingWorkflow.Actions = new List<ActionBase>
    {
        GetPromptForText(EndingMessage),
        new Hangup { OperationId = Guid.NewGuid().ToString() }
    };

    // Convert the audio to text
    if (recordOutcomeEvent.RecordOutcome.Outcome == Outcome.Success)
    {
        var record = await recordOutcomeEvent.RecordedContent;
        string text = await this.GetTextFromAudioAsync(record);

        var callState = this.callStateMap[recordOutcomeEvent.ConversationResult.Id];

        await this.SendSTTResultToUser("We detected the following audio: " + text, callState.Participants);
    }

    recordOutcomeEvent.ResultingWorkflow.Links = null;
    this.callStateMap.Remove(recordOutcomeEvent.ConversationResult.Id);
}
```

## <a name="sample-code"></a>Código de ejemplo

Para obtener un ejemplo completo que muestra cómo se admiten llamadas de audio con Skype mediante Bot Framework SDK para .NET, consulte <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/skype-CallingBot" target="_blank">Ejemplo Skype Calling Bot</a> en GitHub.

## <a name="additional-resources"></a>Recursos adicionales

- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Referencia de Bot Framework SDK para .NET</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/skype-CallingBot" target="_blank">Ejemplo Skype Calling Bot (GitHub)</a>

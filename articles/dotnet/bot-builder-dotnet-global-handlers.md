---
title: Implementación de controladores de mensajes globales | Microsoft Docs
description: Obtenga información sobre cómo permitir que el bot escuche y controle la entrada de usuario que contiene ciertas palabras clave mediante Bot Framework SDK para .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8e997b33e48964b5723d6cd1fef0b1e6542b4ba3
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297237"
---
# <a name="implement-global-message-handlers"></a>Implementación de controladores de mensajes globales

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Introduction to global message handlers](../includes/snippet-global-handlers-intro.md)]

## <a name="listen-for-keywords-in-user-input"></a>Escucha de las palabras clave en la entrada de usuario

En el tutorial siguiente se muestra cómo implementar controladores de mensajes globales mediante Bot Framework SDK para .NET.

En primer lugar, `Global.asax.cs` registra `GlobalMessageHandlersBotModule`, que se implementa como se muestra aquí. En este ejemplo, el módulo registra dos diálogos puntuables: uno para administrar una solicitud para cambiar la configuración (`SettingsScorable`) y otro para administrar una solicitud para cancelar (`CancelScoreable`).

```cs
public class GlobalMessageHandlersBotModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        base.Load(builder);

        builder
            .Register(c => new SettingsScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();

        builder
            .Register(c => new CancelScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();
    }
}
```

`CancelScorable` contiene un método `PrepareAsync` que define el desencadenador: si el texto del mensaje es "Cancelar", se desencadenará este diálogo puntuable.

```cs
protected override async Task<string> PrepareAsync(IActivity activity, CancellationToken token)
{
    var message = activity as IMessageActivity;
    if (message != null && !string.IsNullOrWhiteSpace(message.Text))
    {
        if (message.Text.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
        {
            return message.Text;
        }
    }
    return null;
}
```

Cuando se recibe una solicitud de "Cancelar", el método `PostAsync` dentro de `CancelScoreable` restablece la pila de diálogos. 

```cs
protected override async Task PostAsync(IActivity item, string state, CancellationToken token)
{
    this.task.Reset();
}
```

Cuando se recibe una solicitud de "Cambiar configuración", el método `PostAsync` dentro de `SettingsScorable` invoca a `SettingsDialog` (pasando la solicitud a ese diálogo), con lo que se agrega `SettingsDialog` a la parte superior de la pila de diálogos y se pone en control de la conversación.

```cs
protected override async Task PostAsync(IActivity item, string state, CancellationToken token)
{
    var message = item as IMessageActivity;
    if (message != null)
    {
        var settingsDialog = new SettingsDialog();
        var interruption = settingsDialog.Void<object, IMessageActivity>();
        this.task.Call(interruption, null);
        await this.task.PollAsync(token);
    }
}
```

## <a name="sample-code"></a>Código de ejemplo

Para ver un ejemplo completo que muestra cómo implementar controladores de mensajes globales mediante Bot Framework SDK para .NET, consulte <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers" target="_blank">Ejemplo de controladores de mensajes globales</a> en GitHub.

## <a name="additional-resources"></a>Recursos adicionales

- [Diseño y control del flujo de conversación](../bot-service-design-conversation-flow.md)
- <a href="/dotnet/api/?view=botbuilder-3.12.2.4" target="_blank">Referencia de Bot Framework SDK para .NET</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers" target="_blank">Ejemplo de controladores de mensajes globales (GitHub)</a>

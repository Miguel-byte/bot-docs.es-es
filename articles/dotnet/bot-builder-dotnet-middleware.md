---
title: Interceptación de mensajes | Microsoft Docs
description: Obtenga información sobre cómo interceptar mensajes entre el usuario y un bot con Bot Builder SDK para .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 21607dae5c2a8d08ed4b7bf1b6e6983cd9bf1196
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999732"
---
# <a name="intercept-messages"></a>Interceptación de mensajes

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-middleware.md)
> - [Node.js](../nodejs/bot-builder-nodejs-intercept-messages.md)

[!INCLUDE [Introducton to message logging](../includes/snippet-message-logging-intro.md)]

## <a name="intercept-and-log-messages"></a>Interceptación y registro de mensajes

En el ejemplo de código siguiente se muestra cómo interceptar los mensajes que se intercambian entre el usuario y el bot utilizando el concepto de **software intermedio** de Bot Builder SDK para .NET. 

Primero, cree una clase `DebugActivityLogger` y defina un método `LogAsync` para especificar qué operación se realiza para cada mensaje interceptado. Este ejemplo imprime solo alguna información sobre cada mensaje.

```cs
public class DebugActivityLogger : IActivityLogger
{
    public async Task LogAsync(IActivity activity)
    {
        Debug.WriteLine($"From:{activity.From.Id} - To:{activity.Recipient.Id} - Message:{activity.AsMessageActivity()?.Text}");
    }
}
```

Después agregue el siguiente código a `Global.asax.cs`.  Todos los mensajes intercambiados entre el usuario y el bot (en cualquier dirección) ahora desencadenarán el método `LogAsync` en la clase `DebugActivityLogger`. 

```cs
    public class WebApiApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            var builder = new ContainerBuilder();
            builder.RegisterType<DebugActivityLogger>().AsImplementedInterfaces().InstancePerDependency();
            builder.Update(Conversation.Container);

            GlobalConfiguration.Configure(WebApiConfig.Register);
        }
    }
```

Aunque este ejemplo solo imprime alguna información sobre cada mensaje, puede actualizar el método `LogAsync` para especificar las acciones que desea realizar para cada mensaje. 

## <a name="sample-code"></a>Código de ejemplo 

Para obtener un ejemplo completo en el que se muestre cómo interceptar y registrar mensajes con Bot Builder SDK para .NET, consulte el <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-Middleware" target="_blank">ejemplo de software intermedio</a> en GitHub. 

## <a name="additional-resources"></a>Recursos adicionales

- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Referencia de Bot Builder SDK para .NET</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-Middleware" target="_blank">Ejemplo de software intermedio (GitHub)</a>

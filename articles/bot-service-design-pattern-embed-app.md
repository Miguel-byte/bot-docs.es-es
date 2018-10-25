---
title: Inserción de un bot en una aplicación | Microsoft Docs
description: Obtenga información sobre cómo diseñar un bot que se incrustará en otra aplicación.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 08/15/2018
ms.openlocfilehash: e68358ed65b9300ed8595496b88c5d0736819671
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998222"
---
# <a name="embed-a-bot-in-an-app"></a>Inserción de un bot en una aplicación

Aunque los bots normalmente existen fuera de las aplicaciones, también puede integrarse con las aplicaciones. Por ejemplo, puede insertar un [bot de conocimiento](~/bot-service-design-pattern-knowledge-base.md) dentro de una aplicación para ayudar a los usuarios a encontrar información que, de lo contrario, podría ser difícil de ubicar dentro de las estructuras complejas de la aplicación. Podría insertar un bot dentro de una aplicación de departamento de soporte técnico para que actúe como primer respondedor a solicitudes entrantes de los usuarios. El bot podría resolver problemas sencillos de manera independiente y [derivar](~/bot-service-design-pattern-handoff-human.md) problemas más complejos a un agente humano. 

## <a name="integrating-bot-with-app"></a>Integración del bot con la aplicación

La forma de integrar un bot con una aplicación varía según el tipo de aplicación. 

### <a name="native-mobile-app"></a>Aplicación móvil nativa

Una aplicación que se crea en código nativo puede comunicarse con Bot Framework mediante [Direct Line API][directLineAPI], ya sea a través de REST o websockets.

### <a name="web-based-mobile-app"></a>Aplicación móvil basada en web

Una aplicación móvil que se compila con lenguaje y marcos web, como <a href="https://cordova.apache.org/" target="_blank">Cordova</a>, puede comunicarse con Bot Framework mediante los mismos componentes que usaría un [bot insertado en un sitio web](~/bot-service-design-pattern-embed-web-site.md), simplemente encapsulado dentro del shell de una aplicación nativa.

### <a name="iot-app"></a>Aplicación de IoT

Una aplicación de IoT puede comunicarse con Bot Framework mediante [Direct Line API][directLineAPI]. En algunos escenarios, también puede usar <a href="https://www.microsoft.com/cognitive-services/" target="_blank">Microsoft Cognitive Services</a> para habilitar funcionalidades como reconocimiento de imágenes y voz.

### <a name="other-types-of-apps-and-games"></a>Otros tipos de aplicaciones y juegos

Otros tipos de aplicaciones y juegos pueden comunicarse con Bot Framework mediante [Direct Line API][directLineAPI]. 

## <a name="creating-a-cross-platform-mobile-app-that-runs-a-bot"></a>Creación de una aplicación móvil multiplataforma que ejecuta un bot

Este ejemplo de creación de una aplicación móvil que ejecuta un bot usa <a href="https://www.xamarin.com/" target="_blank">Xamarin</a>, una herramienta popular para crear aplicaciones móviles multiplataforma. 

En primer lugar, cree un componente de vista web simple y úsela para hospedar un <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">control de chat en web</a>. A continuación, mediante Azure Portal, agregue el canal del chat en web. 

A continuación, especifique la dirección URL de chat en web registrada como origen para el control de vista web en la aplicación de Xamarin:

```cs
public class WebPage : ContentPage
{
    public WebPage()
    {
        var browser = new WebView();
        browser.Source = "https://webchat.botframework.com/embed/<YOUR SECRET KEY HERE>";
        this.Content = browser;
    }
}
```

Con este proceso, puede crear una aplicación móvil multiplataforma que represente la vista web insertada con el control de chat en web.

![Canal secundario](~/media/bot-service-design-pattern-embed-app/xamarin-apps.png)

<!-- TODO: No sample bot available
## Sample code

For a complete sample that shows how to create a cross-platform mobile app that runs a bot (as described in this article), see the <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/capability-BotInApps" target="_blank">Bot in Apps sample</a> in GitHub.
-->

## <a name="additional-resources"></a>Recursos adicionales

- [Direct Line API][directLineAPI] (API de Direct Line)
- <a href="https://www.microsoft.com/cognitive-services/" target="_blank">Microsoft Cognitive Services</a>

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle

---
title: Insertar un bot en un sitio web | Microsoft Docs
description: Obtenga información sobre cómo diseñar un bot que se insertará en un sitio web.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 96ccbe453574f15ceda8ae8b4ec1eb44492de738
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298170"
---
# <a name="embed-a-bot-in-a-website"></a>Insertar un bot en un sitio web

Aunque los bots comúnmente existen fuera de los sitios web, también pueden estar integrados en estos. Por ejemplo, puede insertar un [bot de conocimiento](~/bot-service-design-pattern-knowledge-base.md) dentro de un sitio web para que los usuarios puedan encontrar información que, de lo contrario, podría ser difícil de ubicar dentro de las estructuras complejas del sitio web. Asimismo, también podría insertar un bot dentro de un sitio web del departamento de soporte técnico para que actúe como primer respondedor a solicitudes entrantes de los usuarios. El bot puede resolver problemas sencillos de manera independiente y [derivar](~/bot-service-design-pattern-handoff-human.md) problemas más complejos a un agente humano. 

En este artículo se explora la integración de bots con sitios web y el proceso de utilizar el mecanismo *backchannel* para facilitar la comunicación privada entre una página web y un bot. 

Microsoft proporciona dos formas diferentes de integrar un bot en un sitio web: el control web de Skype y un control web de código abierto.

## <a name="skype-web-control"></a>Control web de Skype

El [control web de Skype](https://aka.ms/bot-skype-web-control) es esencialmente un cliente de Skype en un control habilitado para la web. La autenticación integrada de Skype permite que el robot autentique y reconozca a los usuarios, sin que el desarrollador tenga que escribir ningún código personalizado. Skype reconocerá automáticamente las cuentas de Microsoft utilizadas en su cliente web. 

Debido a que el control web de Skype simplemente actúa como interfaz para Skype, el cliente de Skype del usuario tiene acceso  automático al contexto completo de cualquier conversación que facilite el control web. Incluso después de que se cierre el navegador web, el usuario puede continuar interactuando con el bot mediante el cliente de Skype. 

## <a name="open-source-web-control"></a>Control web de código abierto

El <a href="https://aka.ms/BotFramework-WebChat" target="_blank">control Chat en web de código abierto</a> se basa en ReactJS y usa [Direct Line API][directLineAPI] para comunicarse con Bot Framework. El control de chat en web proporciona un lienzo en blanco para implementar el chat en web, lo que le proporciona un control total sobre el comportamiento que tendrá y la experiencia del usuario que ofrecerá. 

El mecanismo *backchannel* permite que la página web que aloja el control se comunique directamente con el bot de una manera totalmente invisible para el usuario. Esta capacidad proporciona cierta cantidad de escenarios útiles: 

- La página web puede enviar datos relevantes al bot (por ejemplo, la ubicación GPS).
- La página web puede asesorar al bot sobre las acciones del usuario (por ejemplo, "el usuario acaba de seleccionar la opción A del menú desplegable").
- La página web puede enviar el bot al token de autenticación de un usuario conectado.
- El bot puede enviar datos relevantes a la página web (por ejemplo, el valor actual de la cartera del usuario).
- El bot puede enviar "comandos" a la página web (por ejemplo, cambiar el color de fondo).

## <a name="using-the-backchannel-mechanism"></a>Usar el mecanismo backchannel

[!INCLUDE [Introduction to backchannel mechanism](~/includes/snippet-backchannel.md)]

## <a name="sample-code"></a>Código de ejemplo

El <a href="https://aka.ms/BotFramework-WebChat" target="_blank">control de chat en web de código abierto </a> está disponible en GitHub. Para más detalles sobre cómo puede implementar el mecanismo backchannel mediante el control de chat en web de código abierto y Bot Framework SDK para Node.js, consulte [Uso del mecanismo backchannel](~/nodejs/bot-builder-nodejs-backchannel.md).

## <a name="additional-resources"></a>Recursos adicionales

- [Direct Line API][directLineAPI]
- [Control de chat en web de código abierto](https://github.com/Microsoft/BotFramework-WebChat)
- [Use the backchannel mechanism](~/nodejs/bot-builder-nodejs-backchannel.md) (Usar el mecanismo backchannel)

[directLineAPI]: https://docs.botframework.com/restapi/directline3/#navtitle

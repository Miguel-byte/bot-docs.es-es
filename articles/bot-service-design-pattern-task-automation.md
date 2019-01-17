---
title: Creación de bots para automatizar tareas | Microsoft Docs
description: Obtenga información sobre cómo diseñar bots que realizan tareas sin intervención humana.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 2/13/2018
ms.openlocfilehash: c14814dc7e2c83f740202db90b7e41efcdfb66a5
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224410"
---
# <a name="create-task-automation-bots"></a>Creación de bots para automatizar tareas

[!INCLUDE [pre-release-label](./includes/pre-release-label-v3.md)]

Un bot para automatizar tareas permite que el usuario complete una tarea específica o un conjunto de tareas sin la ayuda de ninguna persona. Este tipo de bot a menudo se asemeja a una aplicación o un sitio web típicos, ya que se comunica con el usuario principalmente a través de texto y controles de usuario avanzados. Puede incluir funcionalidades de comprensión del lenguaje natural para enriquecer las conversaciones con los usuarios. 

## <a name="example-use-case-password-reset"></a>Caso de uso de ejemplo: restablecimiento de contraseña

Para comprender mejor la naturaleza de un bot de tareas, considere el siguiente caso de uso de ejemplo: restablecimiento de contraseña. 

La empresa Contoso recibe varias llamadas al departamento de soporte técnico cada día de empleados que necesitan restablecer sus contraseñas. Contoso quiere automatizar la tarea simple y repetitiva del restablecimiento de contraseña de un empleado para que los empleados del departamento de soporte técnico puedan dedicar su tiempo a abordar los problemas más complejos. 

José, un desarrollador experimentado de Contoso, decide crear un bot para automatizar la tarea de restablecimiento de contraseña. Comienza escribiendo una especificación de diseño para el bot, de la misma forma que lo haría si estuviera creando una nueva aplicación o sitio web. 

::: moniker range="azure-bot-service-3.0"

### <a name="navigation-model"></a>Modelo de navegación

La especificación define el modelo de navegación:

![Estructura del diálogo](~/media/bot-service-design-pattern-task-automation/simple-task1.png)

El usuario comienza en `RootDialog`. Cuando solicite un restablecimiento de contraseña,  
se le llevará a `ResetPasswordDialog`. En `ResetPasswordDialog`, el bot solicitará que el usuario escriba dos datos: el número de teléfono y la fecha de nacimiento. 

> [!IMPORTANT]
> El diseño del bot descrito en este artículo está pensado únicamente con fines de ejemplo. En escenarios del mundo real, un bot de restablecimiento de contraseña probablemente implementará un proceso de verificación de identidad más sólido.

### <a name="dialogs"></a>Diálogos

A continuación, la especificación describe la apariencia y funcionalidad de cada cuadro de diálogo. 

#### <a name="root-dialog"></a>Cuadro de diálogo raíz

El cuadro de diálogo raíz ofrece dos opciones al usuario: 

1. **Cambiar contraseña**, que se utiliza para escenarios en los cuales el usuario conoce la contraseña actual y simplemente quiere cambiarla.
2. **Restablecer contraseña**, que se utiliza para escenarios en los cuales el usuario ha olvidado o ha perdido su contraseña y debe generar una nueva.

> [!NOTE]
> Por motivos de simplicidad, este artículo solo describe el flujo para **restablecer la contraseña**.

La especificación describe el cuadro de diálogo raíz, como se muestra en la siguiente captura de pantalla.

![Estructura del diálogo](~/media/bot-service-design-pattern-task-automation/simple-task2.png)

#### <a name="resetpassword-dialog"></a>Cuadro de diálogo ResetPassword

Cuando el usuario elige **Restablecer contraseña** desde el cuadro de diálogo raíz, se invoca el cuadro de diálogo `ResetPassword`. 
A continuación, el cuadro de diálogo `ResetPassword` invoca otros dos cuadros de diálogo. 
Primero, invoca el cuadro de diálogo `PromptStringRegex` para recopilar el número de teléfono del usuario. 
Luego, invoca el cuadro de diálogo `PromptDate` para recopilar la fecha de nacimiento del usuario. 

> [!NOTE]
> En este ejemplo, José decide implementar la lógica para recopilar el número de teléfono y la fecha de nacimiento del usuario mediante dos cuadros de diálogo independientes. El enfoque no solo simplifica el código necesario para cada cuadro de diálogo, sino que también aumenta las probabilidades de que estos cuadros de diálogo se puedan utilizar en otros escenarios en el futuro. 

La especificación describe el cuadro de diálogo `ResetPassword`.

![Estructura del diálogo](~/media/bot-service-design-pattern-task-automation/simple-task3.png)

#### <a name="promptstringregex-dialog"></a>Cuadro de diálogo PromptStringRegex

El cuadro de diálogo `PromptStringRegex` pide al usuario que escriba su número de teléfono y verifique que coincida con el formato esperado. 
También tiene en cuenta el escenario en el cual el usuario proporciona varias veces una entrada no válida. 
La especificación describe el cuadro de diálogo `PromptStringRegex`.

![Estructura del diálogo](~/media/bot-service-design-pattern-task-automation/simple-task4.png)

### <a name="prototype"></a>Prototipo

Por último, la especificación proporciona un ejemplo de un usuario que se comunica con el bot para completar correctamente la tarea de restablecimiento de contraseña.

![Estructura del diálogo](~/media/bot-service-design-pattern-task-automation/simple-task5.png)

::: moniker-end 

## <a name="bot-app-or-website"></a>¿Bot, aplicación o sitio web?

Quizás se pregunte, si un bot de automatización de tareas se parece a una aplicación o sitio web, ¿por qué no simplemente crear una aplicación o un sitio web en su lugar? En función del escenario concreto, la creación de una aplicación o un sitio web en lugar de un bot puede ser una opción completamente razonable. Incluso puede insertar el bot en una aplicación, mediante el uso de la [API Direct Line de Bot Framework][directLineAPI] o el <a href="https://aka.ms/BotFramework-WebChat" target="_blank">control Chat en web</a>. La implementación del bot en el contexto de una aplicación ofrece lo mejor de ambos mundos: una experiencia de aplicación avanzada y una experiencia de conversación, todo en un solo lugar. 

Sin embargo, en muchos casos la creación de una aplicación o sitio web puede ser mucho más compleja y más cara que la creación de un bot. Una aplicación o sitio web a menudo deben admitir varias plataformas y clientes, el empaquetado y la implementación pueden ser procesos tediosos y lentos y la experiencia del usuario de tener que descargar e instalar una aplicación no es necesariamente ideal. Por estas razones, un bot suele ofrecer una forma mucho más sencilla de resolver el problema en cuestión. 

Además, los bots ofrecen la libertad de ampliarse y extenderse fácilmente. Por ejemplo, un desarrollador puede optar por agregar funcionalidades de lenguaje natural y de voz al bot para el restablecimiento de contraseña, de modo que se pueda acceder a él a través de llamadas de audio, o bien puede agregar compatibilidad con mensajes de texto. La empresa puede configurar quioscos multimedia en toda la compilación y puede insertar el bot de restablecimiento de contraseña en esa experiencia.

::: moniker range="azure-bot-service-3.0"
<!-- TODO: SimpleTaskAutomation no longer exists
## Sample code

For a complete sample that shows how to implement simple task automation using the Bot Framework SDK for .NET, see the <a href="https://aka.ms/capability-SimpleTaskAutomation" target="_blank">Simple Task Automation sample</a> in GitHub.

For a complete sample that shows how to implement simple task automation using the Bot Framework SDK for Node.js, see the <a href="https://aka.ms/capability-SimpleTaskAutomation" target="_blank">Simple Task Automation sample</a> in GitHub.
-->

## <a name="additional-resources"></a>Recursos adicionales

- [Diálogos](~/dotnet/bot-builder-dotnet-dialogs.md)
- [Administración del flujo de conversación con diálogos (.NET)](~/dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [Manage conversation flow with dialogs (Node.js)](~/nodejs/bot-builder-nodejs-manage-conversation-flow.md) (Administración del flujo de conversación con cuadros de diálogo [Node.js])

::: moniker-end

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle

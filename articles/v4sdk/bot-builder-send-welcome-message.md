---
title: Envío de mensajes de bienvenida a los usuarios | Microsoft Docs
description: Aprenda a desarrollar su bot para proporcionar una experiencia de bienvenida al usuario.
keywords: overview, develop, user experience, welcome, personalized experience, C#, JS, welcome message, bot, greet, greeting
author: DanDev33
ms.author: v-dashel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f7709d6273739be6d2b3e9e3174f24ea2734f22d
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215311"
---
# <a name="send-welcome-message-to-users"></a>Envío de mensajes de bienvenida a los usuarios

[!INCLUDE[applies-to](../includes/applies-to.md)]

El objetivo principal al crear cualquier bot es que el usuario participe en una conversación que tenga sentido. Una de las mejores formas de lograr este objetivo es asegurarse de que desde el momento en que un usuario se conecta por primera vez, comprende la finalidad principal del bot y sus funcionalidades, es decir, el motivo de que se haya creado el bot. Este artículo proporciona ejemplos de código que le ayudarán a dar la bienvenida a los usuarios del bot.

## <a name="prerequisites"></a>Requisitos previos
- Comprender los [conceptos básicos de bot](bot-builder-basics.md). 
- Una copia del **ejemplo de bienvenida al usuario** en [C#](https://aka.ms/welcome-user-mvc) o en [JS](https://aka.ms/bot-welcome-sample-js). El código del ejemplo se utiliza para explicar cómo enviar mensajes de bienvenida.

## <a name="about-this-sample-code"></a>Acerca de este código de ejemplo
Este código de ejemplo muestra cómo detectar y dar la bienvenida a los nuevos usuarios cuando se conectan al bot inicialmente. El siguiente diagrama muestra el flujo de lógica de este bot. 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
Los dos principales eventos detectados por el bot son
- `OnMembersAddedAsync`, que se llama cada vez que un nuevo usuario se conecta al bot
- `OnMessageActivityAsync`, que se llama cada vez que se recibe la entrada de un nuevo usuario.

![flujo de lógica de bienvenida del usuario](media/welcome-user-flow.png)

Cada vez que un nuevo usuario se conecta, el bot le proporciona los elementos `WelcomeMessage`, `InfoMessage` y `PatternMessage`. Cuando se recibe una nueva entrada de usuario, WelcomeUserState se comprueba para ver si `DidBotWelcomeUser` está establecido en _true_. Si no es así, se devuelve un mensaje de bienvenida inicial al usuario.

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
Los dos principales eventos detectados por el bot son
- `onMembersAdded`, que se llama cada vez que un nuevo usuario se conecta al bot
- `onMessage`, que se llama cada vez que se recibe la entrada de un nuevo usuario.

![flujo de lógica de bienvenida del usuario](media/welcome-user-flow-js.png)

Cada vez que un nuevo usuario se conecta, el bot le proporciona los elementos `welcomeMessage`, `infoMessage` y `patternMessage`. Cuando se recibe una nueva entrada de usuario, se comprueba `welcomedUserProperty` para ver si `didBotWelcomeUser` está establecido en _true_. Si no es así, se devuelve un mensaje de bienvenida inicial al usuario.

---

 Si DidBotWelcomeUser es _true_, se evalúa la entrada del usuario. Según el contenido de la entrada del usuario, este bot realizará una de las siguientes acciones:
- Devolver un saludo recibido desde el usuario.
- Mostrar una tarjeta de imagen prominente que proporciona información adicional sobre los bots.
- Volver a enviar el mensaje `WelcomeMessage` explicando las entradas esperadas por el bot.

## <a name="create-user-object"></a>Creación del objeto de usuario
### <a name="ctabcsharp"></a>[C#](#tab/csharp)
El objeto de estado del usuario se crea en el inicio y se inserta la dependencia en el constructor del bot.

**Startup.cs** [!code-csharp[ConfigureServices](~/../botBuilder-samples/samples/csharp_dotnetcore/03.welcome-user/Startup.cs?range=29-33)]

**WelcomeUserBot.cs** [!code-csharp[Class](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=41-47)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
En el inicio, tanto el estado del usuario como el almacenamiento de memoria se definen en index.js.

**Index.js** [!code-javascript[DefineUserState](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/Index.js?range=8-10,33-41)]

---

## <a name="create-property-accessors"></a>Creación de los descriptores de acceso de la propiedad
### <a name="ctabcsharp"></a>[C#](#tab/csharp)
Ahora creamos un descriptor de acceso de la propiedad que nos proporciona un identificador de WelcomeUserState dentro del método OnMessageActivityAsync.
A continuación, llamamos al método GetAsync para obtener la clave con el ámbito correcto. A continuación, guardamos los datos de estado del usuario después de cada iteración de entrada del usuario con el método `SaveChangesAsync`.

**WelcomeUserBot.cs** [!code-csharp[OnMessageActivityAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=68-71, 102-105)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
Ahora creamos un descriptor de acceso de la propiedad que nos proporciona un identificador de WelcomedUserProperty que se conserva en UserState.

**WelcomeBot.js** [!code-javascript[DefineUserState](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=7-10,16-22)]

---

## <a name="detect-and-greet-newly-connected-users"></a>Detección y saludo a los usuarios recién conectados

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
En **WelcomeUserBot**, se comprueba si hay una actualización de actividad mediante `OnMembersAddedAsync()` para ver si se ha agregado un nuevo usuario a la conversación y, a continuación, se envía un conjunto de tres mensajes iniciales de bienvenida: `WelcomeMessage`, `InfoMessage` y `PatternMessage`. A continuación se muestra el código completo de esta interacción.

**WelcomeUserBot.cs** [!code-csharp[WelcomeMessages](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=20-40, 55-66)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
Este código JavaScript envía mensajes de bienvenida iniciales cuando se agrega un usuario. Esto se realiza mediante la comprobación de la actividad de la conversación y la verificación de que se ha agregado un nuevo miembro a la conversación.

**WelcomeBot.js** [!code-javascript[DefineUserState](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=65-87)]

---

## <a name="welcome-new-user-and-discard-initial-input"></a>Bienvenida al nuevo usuario y descarte de la entrada inicial

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
También es importante tener en cuenta cuándo la entrada del usuario puede contener realmente información útil y esto puede variar según el canal. Para asegurarse de que el usuario tenga una buena experiencia en todos los canales posibles, comprobamos la marca de estado _didBotWelcomeUser_ y, si es "false", no se procesa la entrada de usuario inicial. En su lugar, proporcionamos al usuario un mensaje de bienvenida inicial. El valor booleano _welcomedUserProperty_, a continuación, se establece en "true", se almacena en UserState y el código procesará la entrada del usuario de todas las actividades de mensaje adicionales.

**WelcomeUserBot.cs** [!code-csharp[DidBotWelcomeUser](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=68-82)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
También es importante tener en cuenta cuándo la entrada del usuario puede contener realmente información útil y esto puede variar según el canal. Para asegurarse de que el usuario tenga una buena experiencia en todos los canales posibles, comprobamos la propiedad didBotWelcomedUser y, si no existe, se establece en "false" y no se procesa la entrada de usuario inicial. En su lugar, proporcionamos al usuario un mensaje de bienvenida inicial. El valor booleano _didBotWelcomeUser_ se establece en "true" y el código procesa los datos de entrada del usuario en todas las actividades de mensajes adicionales.

**WelcomeBot.js** [!code-javascript[DidBotWelcomeUser](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=24-38,57-59,63)]

---

## <a name="process-additional-input"></a>Proceso de entradas adicionales

Una vez se ha dado la bienvenida a un nuevo usuario, la información de entrada del usuario se evalúa en cada turno de mensaje y el bot proporciona una respuesta en función del contexto de esa entrada del usuario. El código siguiente muestra la lógica de decisiones que se usa para generar esa respuesta. 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
Una entrada de "intro" o "help" llama a la función `SendIntroCardAsync` para presentar al usuario una tarjeta de imagen prominente informativa. El código se examina en la sección siguiente de este artículo.

**WelcomeUserBot.cs** [!code-csharp[SwitchOnUtterance](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=85-100)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
Una entrada de "intro" o "help" utiliza CardFactory para presentar al usuario una tarjeta adaptable de introducción. El código se examina en la sección siguiente de este artículo.

**WelcomeBot.js** [!code-javascript[SwitchOnUtterance](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=40-56)]

---

## <a name="using-hero-card-greeting"></a>Uso del saludo de la tarjeta de imagen prominente

Como se mencionó anteriormente, algunas entradas de usuario generan una _tarjeta de imagen prominente_ en respuesta a su solicitud. Puede aprender más sobre las tarjetas de imagen prominente aquí: [Envío de una tarjeta de introducción](./bot-builder-howto-add-media-attachments.md). A continuación está el código necesario para crear la respuesta de tarjeta de imagen prominente de este bot.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**WelcomeUserBot.cs** [!code-csharp[SendHeroCardGreeting](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=107-127)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**WelcomeBot.js** [!code-javascript[SendIntroCard](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=91-116)]

---

## <a name="test-the-bot"></a>Probar el bot

Descargue e instale la versión más reciente de [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).

1. Ejecute el ejemplo localmente en la máquina. Si necesita instrucciones, consulte el archivo Léame en el ejemplo de [C#](https://aka.ms/welcome-user-mvc) o [JS](https://aka.ms/bot-welcome-sample-js).
1. Use el emulador para probar el bot tal y como se muestra a continuación.

![ejemplo de bot de bienvenida de prueba](media/welcome-user-emulator-1.png)

Pruebe el saludo de la tarjeta de imagen prominente.

![tarjeta del bot de bienvenida de prueba](media/welcome-user-emulator-2.png)

## <a name="additional-resources"></a>Recursos adicionales

Más información sobre diversas respuestas de elementos multimedia en [Adición de elementos multimedia a los mensajes](./bot-builder-howto-add-media-attachments.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Recopilación de entradas del usuario](bot-builder-prompts.md)

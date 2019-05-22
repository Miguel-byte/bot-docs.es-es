---
title: Creación de mensajes propios para recopilar datos de entrada del usuario | Microsoft Docs
description: Aprenda a administrar un flujo de conversación con avisos primitivos en Bot Framework SDK.
keywords: flujo de la conversación, avisos, estado de conversación, estado de usuario, avisos personalizados
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/08/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3470b1c8f3fbcfb7fecbb060a54b1a356ad41b61
ms.sourcegitcommit: 4086189a9c856fbdc832eb1a1d205e5f1b4e3acd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65733320"
---
# <a name="create-your-own-prompts-to-gather-user-input"></a>Creación de mensajes propios para recopilar datos de entrada del usuario

[!INCLUDE[applies-to](../includes/applies-to.md)]

A menudo, una conversación entre un usuario y un bot implica mostrar preguntas (avisos) al usuario para obtener información, analizar la respuesta del usuario y, a continuación, actuar en dicha información. El bot debe hacer un seguimiento del contexto de una conversación, para que pueda controlar su comportamiento y recordar las respuestas a las preguntas anteriores. El *estado* de un bot es la información que sigue con el fin de responder de forma adecuada a los mensajes entrantes. 

> [!TIP]
> La biblioteca de diálogos proporciona avisos incorporados que proporcionan más funcionalidad que pueden usar los usuarios. Se pueden encontrar ejemplos de estos avisos en el artículo [Implementación de un flujo de conversación secuencial](bot-builder-dialog-manage-conversation-flow.md).

## <a name="prerequisites"></a>Requisitos previos

- El código de este artículo se basa en el ejemplo para solicitar una entrada a los usuarios. Necesitará una copia del **[ejemplo de C#](https://aka.ms/cs-primitive-prompt-sample) o del [ejemplo de JavaScript](https://aka.ms/js-primitive-prompt-sample)**.
- Conocimientos sobre la [administración del estado](bot-builder-concept-state.md) y cómo [guardar los datos de usuario y conversación](bot-builder-howto-v4-state.md).

## <a name="about-the-sample-code"></a>Acerca del código de ejemplo

El bot de ejemplo le formula al usuario una serie de preguntas, valida algunas de sus respuestas y guarda sus comentarios. El siguiente diagrama muestra la relación entre el bot, el perfil de usuario y las clases de flujos de conversación. 

## <a name="ctabcsharp"></a>[C#](#tab/csharp)
![custom-prompts](media/CustomPromptBotSample-Overview.png)

- Una clase `UserProfile` para la información de usuario que recopilará el bot.
- Una clase `ConversationFlow` para controlar el estado de la conversación mientras se recopila información del usuario.
- Una enumeración `ConversationFlow.Question` interna para el seguimiento de dónde nos encontramos en la conversación.

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
![custom-prompts](media/CustomPromptBotSample-JS-Overview.png)

- Una clase `userProfile` para la información de usuario que recopilará el bot.
- Una clase `conversationFlow` para controlar el estado de la conversación mientras se recopila información del usuario.
- Una enumeración `conversationFlow.question` interna para el seguimiento de dónde nos encontramos en la conversación.

---

El estado del usuario realiza un seguimiento del nombre, edad y fecha elegida del usuario, y el estado de conversación, un seguimiento de lo que se acaba de preguntar al usuario.
Como no tenemos previsto implementar este bot, vamos a configurar tanto el estado de usuario como el de conversación para que utilicen el _almacenamiento en memoria_. 

Se usa el controlador de turnos de mensaje del bot y las propiedades de estado de usuario y de conversación para administrar el flujo de la conversación y la colección de entradas. En el bot, se registrará la información de propiedad de estado que se reciba durante cada iteración del controlador de turnos de mensajes.

## <a name="create-conversation-and-user-objects"></a>Creación de objetos de conversación y usuario

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Los objetos de estado de usuario y conversación se crean en el inicio y se inserta la dependencia en el constructor del bot. 

**Startup.cs** [!code-csharp[Startup](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Startup.cs?range=27-34)]

**Bots/CustomPromptBot.cs** [!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=21-28)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En **index.js**, cree las propiedades de estado y el bot y, a continuación, llame al método del bot `run` desde dentro de `processActivity`.

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/index.js?range=32-35)]

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/index.js?range=55-58)]

---

## <a name="create-property-accessors"></a>Creación de los descriptores de acceso de la propiedad

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

En primer lugar, se crearán los descriptores de acceso de la propiedad que nos proporcionarán un control sobre `BotState` en el método `OnMessageActivityAsync`. A continuación, se llamará al método `GetAsync` para obtener la clave con el ámbito correcto:

**Bots/CustomPromptBot.cs** [!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=30-37)]

Y por último, se guardarán los datos mediante el método `SaveChangesAsync`.

[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=42-43)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En el constructor, se crearán los descriptores de acceso de la propiedad de estado y se configurarán los objetos de administración de estados (que se crearon anteriormente) para nuestra conversación.

**bots/customPromptBot.js** [!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=23-29)]

A continuación, se definirá un segundo controlador, `onDialog`, para que se ejecute después del controlador de mensajes principal (esto se explica en la sección siguiente). Este segundo controlador garantiza que se guardará nuestro estado en cada turno.

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=41-48)]

---

## <a name="the-bots-message-turn-handler"></a>Controlador de turnos de mensaje del bot

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Para controlar las actividades de los mensajes, se usa el método auxiliar _FillOutUserProfileAsync()_ antes de guardar el estado mediante _SaveChangesAsync()_. Aquí está todo el código.

**Bots/CustomPromptBot.cs** [!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=30-44)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Para controlar las actividades de los mensajes, se configurarán nuestros datos de usuario y de conversación y, a continuación, se usará el método auxiliar `fillOutUserProfile()`. Este es el código completo para el controlador de turnos.

**bots/customPromptBot.js** [!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=31-39)]
---

## <a name="filling-out-the-user-profile"></a>Rellenado del perfil de usuario

Comenzaremos por recopilar la información. Cada uno proporcionará una interfaz similar.

- El valor devuelto indica si la entrada es una respuesta válida para esta pregunta.
- Si la validación es correcta, se genera un valor normalizado y analizado para guardarlo.
- Si se produce un error de validación, se genera un mensaje con el que el bot puede volver a pedir la información.

 En la sección siguiente, se van a definir los métodos auxiliares para analizar y validar la entrada del usuario.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots/CustomPromptBot.cs** [!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=46-103)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/customPromptBot.js** [!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=52-116)]

---

## <a name="parse-and-validate-input"></a>Análisis y validación de la entrada

Vamos a usar los siguientes criterios para validar la entrada.

- El **nombre** debe ser una cadena que no esté vacía. Lo normalizaremos al recortar los espacios en blanco.
- La **edad** debe estar entre 18 y 120. Lo normalizaremos al devolver un entero.
- La **fecha** debe ser cualquier fecha u hora al menos una hora en el futuro.
  La normalizaremos al devolver solo la parte de fecha de la entrada analizada.

> [!NOTE]
> Para la entrada de edad y fecha, utilizamos las bibliotecas [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text/) para realizar el análisis inicial.
> Aunque proporcionamos código de ejemplo, no explicamos cómo funcionan las bibliotecas de los reconocedores de texto, y esta es solo una forma de analizar la entrada.
> Para más información acerca de estas bibliotecas, consulte el archivo **Léame** del repositorio.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Agregue los siguientes métodos de validación al bot.

**Bots/CustomPromptBot.cs** [!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=105-203)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/customPromptBot.cs** [!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=118-189)]

---

## <a name="test-the-bot-locally"></a>Prueba local del bot
Descargue e instale [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) para probar el bot localmente.

1. Ejecute el ejemplo localmente en la máquina. Si necesita instrucciones, consulte el archivo Léame en el ejemplo de [C#](https://aka.ms/cs-primitive-prompt-sample) o [JS](https://aka.ms/js-primitive-prompt-sample).
1. Pruébelo con el emulador, tal como se muestra a continuación.

![primitive-prompts](media/primitive-prompts.png)

## <a name="additional-resources"></a>Recursos adicionales

La [biblioteca de diálogos](bot-builder-concept-dialog.md) proporciona clases que automatizan muchos aspectos de la administración de las conversaciones. 

## <a name="next-step"></a>Paso siguiente

> [!div class="nextstepaction"]
> [Implementación de flujo de conversación secuencial](bot-builder-dialog-manage-conversation-flow.md)

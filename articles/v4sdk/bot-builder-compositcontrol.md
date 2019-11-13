---
title: Reutilización de diálogos | Microsoft Docs
description: Aprenda a modularizar la lógica del bot mediante diálogos de componentes en Bot Framework SDK.
keywords: control compuesto, lógica de bot modular
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/05/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6ef79b62aecbc79ed277f3962606d5ed5d9ceeb3
ms.sourcegitcommit: 312a4593177840433dfee405335100ce59aac347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/12/2019
ms.locfileid: "73933574"
---
# <a name="reuse-dialogs"></a>Reutilización de diálogos

[!INCLUDE[applies-to](../includes/applies-to.md)]

Con los diálogos de componentes se pueden crear diálogos independientes para controlar escenarios concretos, y dividir un conjunto de diálogos grande en elementos más manejables. Cada uno de estos elementos tiene su propio conjunto de diálogos y evita los conflictos de nombres con los conjuntos de diálogos exteriores.

## <a name="prerequisites"></a>Requisitos previos

- Conocimiento de los [conceptos básicos de bots][concept-basics], la [biblioteca de diálogos][concept-dialogs] y la [administración de conversaciones][simple-flow].
- Una copia del ejemplo de aviso con varios turnos en [**C#** ][cs-sample] o [**JavaScript**][js-sample].

## <a name="about-the-sample"></a>Sobre el ejemplo

En el ejemplo de aviso de varios turnos, usamos un diálogo en cascada, algunos avisos y un diálogo de componente para crear una interacción simple que formula al usuario una serie de preguntas. El código usa un diálogo para desplazarse por estos pasos:

| Pasos        | Tipo de aviso  |
|:-------------|:-------------|
| Pedir al usuario su modo de transporte | Aviso de elección |
| Pedir al usuario su nombre | Aviso de texto |
| Pedir al usuario si desea proporcionar su edad | Aviso de confirmación |
| Si responde Sí, solicitar su edad  | Aviso numérico con validación para que solo acepte edades mayores que 0 y menores de 150. |
| Preguntar si la información recopilada es correcta | Reutilización del aviso de confirmación |

Por último, si responde Sí, mostrar la información recopilada; de lo contrario, indicar al usuario que no se conservará su información.

## <a name="implement-the-component-dialog"></a>Implementación del diálogo de componente

En el ejemplo de aviso de varios turnos, usamos un _diálogo en cascada_, algunos _avisos_ y un _diálogo de componente_ para crear una interacción simple que formula al usuario una serie de preguntas.

Un diálogo de componente encapsula uno o varios diálogos. El diálogo de componente tiene un conjunto de diálogos interno y tanto los diálogos como los avisos que agregue a dicho conjunto tienen sus propios identificadores, que solo se pueden ver desde el diálogo de componente.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Para usar diálogos, instale el paquete de NuGet **Microsoft.Bot.Builder.Dialogs**.

**Dialogs\UserProfileDialog.cs**

Aquí la `UserProfileDialog` clase deriva de ka `ComponentDialog` clase.

[!code-csharp[Class](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=13)]

Dentro del constructor, el método `AddDialog` agrega diálogos y avisos al diálogo de componente. El primer elemento que se agregue con este método se establece como diálogo inicial, pero se puede cambiar. Para ello, es preciso establecer explícitamente la propiedad `InitialDialogId`. Al iniciar un diálogo de componente, se iniciará su _initial dialog_.

[!code-csharp[Constructor](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=17-42)]

Se trata de la implementación del primer paso del diálogo en cascada.

[!code-csharp[First step](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=44-54)]

Para más información acerca de la implementación de los diálogo en cascada, consulte cómo [implementar un flujo de conversaciones secuencial](bot-builder-dialog-manage-complex-conversation-flow.md).

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Para usar diálogos, el proyecto debe instalar el paquete de npm **botbuilder-dialogs**.

**dialogs/userProfileDialog.js**

Aquí la clase `UserProfileDialog` extiende `ComponentDialog`.

[!code-javascript[Class](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=24)]

Dentro del constructor, el método `AddDialog` agrega diálogos y avisos al diálogo de componente. El primer elemento que se agregue con este método se establece como diálogo inicial, pero se puede cambiar. Para ello, es preciso establecer explícitamente la propiedad `InitialDialogId`. Al iniciar un diálogo de componente, se iniciará su _initial dialog_.

[!code-javascript[Constructor](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=25-45)]

Se trata de la implementación del primer paso del diálogo en cascada.

[!code-javascript[First step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=64-71)]

Para más información acerca de la implementación de los diálogo en cascada, consulte cómo [implementar un flujo de conversaciones secuencial](bot-builder-dialog-manage-complex-conversation-flow.md).

---

En tiempo de ejecución, el diálogo de componente mantiene su propia pila de diálogos. Cuando el diálogo de componente se inicia:

- Se crea una instancia y se agrega a la pila de diálogos externa
- Crea una pila de diálogos interna que agrega a su estado
- Comienza su diálogo inicial y lo agrega a la pila de diálogos interna.

Desde el contexto primario, parecerá que el componente es el diálogo activo. Desde dentro del componente, parecerá que el diálogo inicial es el activo.

### <a name="implement-the-rest-of-the-dialog-and-add-it-to-the-bot"></a>Implemente el resto del diálogo y agréguelo al bot

En el conjunto de diálogos externo, al que agrega el diálogo de componente, el diálogo de componente tiene el identificador con el que lo creó. En el conjunto externo, el componente parece un diálogo individual, así como los avisos.

Para usar un diálogo de componente, agregue una instancia de él al conjunto de diálogos del bot (este es el conjunto de diálogos externo).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\DialoBot.cs**

En el ejemplo, esto se hace con el método `RunAsync` al que se llama desde el método `OnMessageActivityAsync` del bot.

[!code-csharp[OnMessageActivityAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Bots/DialogBot.cs?range=42-48)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/userProfileDialog.js**

En el ejemplo, se ha agregado un método `run` al diálogo del perfil de usuario.

[!code-javascript[run method](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=53-62)]

**bots/dialogBot.js**

Se llama al método `run` desde el método `onMessage` del bot.

[!code-javascript[onMessage](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/bots/dialogBot.js?range=24-31&highlight=5)]

---

## <a name="to-test-the-bot"></a>Prueba del bot

1. Si aún no lo ha hecho, instale [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).
1. Ejecute el ejemplo localmente en la máquina.
1. Inicie el emulador, conéctese al bot y envíe mensajes como se muestra a continuación.

![Ejecución de ejemplo del diálogo de aviso de varios turnos](../media/emulator-v4/multi-turn-prompt.png)

## <a name="additional-information"></a>Información adicional

### <a name="how-cancellation-works-for-component-dialogs"></a>Funcionamiento de la cancelación en los diálogos de componentes

Si se llama a _cancelar todos los diálogos_ desde el contexto del diálogo del componente, dicho diálogo cancelará todos los diálogos de su pila interna y, después, finalizará, con lo que devolverá el control al siguiente diálogo de la pila externa.

Si se llama a _cancelar todos los diálogos_ desde el contexto externo, se cancelan tanto componente, como el resto de los diálogos del contexto externo.

Téngalo en cuenta al administrar diálogos de componentes anidados en el bot.

## <a name="next-steps"></a>Pasos siguientes

Puede mejorar los bots para que reaccionen a entradas adicionales, como "ayuda" o "cancelar", que pueden interrumpir el flujo normal de la conversación.

> [!div class="nextstepaction"]
> [Control de las interrupciones del usuario](bot-builder-howto-handle-user-interrupt.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-flow]: bot-builder-dialog-manage-conversation-flow.md
[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-multi-prompts-sample
[js-sample]: https://aka.ms/js-multi-prompts-sample

---
title: Control de las interrupciones del usuario | Microsoft Docs
description: Obtenga información sobre cómo controlar el flujo de conversación directa y la interrupción del usuario.
keywords: interrumpir, interrupciones, cambio de tema, interrupción
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/18/2019
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bd8682966dbb2e33a536a72a4016ef23e9c1fc75
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032621"
---
# <a name="handle-user-interruptions"></a>Control de las interrupciones del usuario

[!INCLUDE[applies-to](../includes/applies-to.md)]

El control de interrupciones es un aspecto importante de un bot sólido. Los usuarios no siempre seguirán el flujo de conversación que ha definido paso a paso. Es posible que intenten formular una pregunta en mitad del proceso o simplemente deseen cancelarlo en lugar de completarlo. En este tema se exploran algunas formas habituales de controlar las interrupciones de usuario en el bot.

## <a name="prerequisites"></a>Requisitos previos

- Conocimiento de los [conceptos básicos de bots][concept-basics], [la administración del estado][concept-state], la [biblioteca de diálogos][concept-dialogs] y la [reutilización de diálogos][component-dialogs].
- Una copia del ejemplo de bot básico en [**CSharp**][cs-sample] o [**JavaScript**][js-sample].

## <a name="about-this-sample"></a>Acerca de este ejemplo

El ejemplo utilizado en este artículo modela un bot de reservas de vuelos que usa diálogos para obtener la información del vuelo del usuario. En cualquier momento durante la conversación con el bot, el usuario puede emitir los comandos _help_ o _cancel_ para provocar una interrupción. Hay dos tipos de interrupciones que se controlan aquí:

- **Nivel de turno**: omitir el procesamiento en el nivel de turno pero dejar el diálogo en la pila con la información que se proporcionó. En el siguiente turno, continuar desde donde se terminó. 
- **Nivel de diálogo**: cancelar el procesamiento por completo para que el bot pueda volver a empezar.

## <a name="define-and-implement-the-interruption-logic"></a>Definición e implementación de la lógica de interrupción

En primer lugar, tenemos que definir e implementar las interrupciones _help_ y _cancel_.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Para usar diálogos, instale el paquete de NuGet **Microsoft.Bot.Builder.Dialogs**.

**Dialogs\CancelAndHelpDialog.cs**

Comenzamos por la implementación de la clase `CancelAndHelpDialog` para controlar las interrupciones de usuario.

[!code-csharp[Class signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/CancelAndHelpDialog.cs?range=11)]

En la clase `CancelAndHelpDialog`, los métodos `OnBeginDialogAsync` y `OnContinueDialogAsync` llaman al método `InerruptAsync` para comprobar si el usuario ha interrumpido el flujo normal o no. Si se interrumpe el flujo, se llama a los métodos de la clase base; en caso contrario, se devuelve el valor devuelto desde `InterruptAsync`.

[!code-csharp[Overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/CancelAndHelpDialog.cs?range=18-27)]

Si el usuario escribe "help", el método `InterrupAsync` envía un mensaje y, a continuación, llama a `DialogTurnResult (DialogTurnStatus.Waiting)` para indicar que el diálogo en la parte superior espera una respuesta del usuario. De este modo, se interrumpe el flujo de conversación solo para un turno y en el siguiente turno seguimos donde terminamos.

Si el usuario escribe "cancel", llama a `CancelAllDialogsAsync` en el contexto de diálogo interno, que borra la pila de diálogos y hace que se cierre con un estado cancelado y sin valor de resultado. Para `MainDialog` (que se muestra más adelante), parece que el diálogo de reservas finalizó y devolvió NULL, de modo similar a cuando el usuario decide no confirmar su reserva.

[!code-csharp[Interrupt](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/CancelAndHelpDialog.cs?range=40-61&highlight=11-12,16-17)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Para usar diálogos, instale el paquete de npm **botbuilder-dialogs**.

**dialogs/cancelAndHelpDialog.js**

Comenzamos por la implementación de la clase `CancelAndHelpDialog` para controlar las interrupciones de usuario.

[!code-javascript[Class signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/cancelAndHelpDialog.js?range=10)]

En la clase `CancelAndHelpDialog`, los métodos `onBeginDialog` y `onContinueDialog` llaman al método `interrupt` para comprobar si el usuario ha interrumpido el flujo normal o no. Si se interrumpe el flujo, se llama a los métodos de la clase base; en caso contrario, se devuelve el valor devuelto desde `interrupt`.

[!code-javascript[Overrides](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/cancelAndHelpDialog.js?range=11-25)]

Si el usuario escribe "help", el método `interrupt` envía un mensaje y, a continuación, devuelve un objeto `{ status: DialogTurnStatus.waiting }` para indicar que el diálogo en la parte superior espera una respuesta del usuario. De este modo, se interrumpe el flujo de conversación solo para un turno y en el siguiente turno seguimos donde terminamos.

Si el usuario escribe "cancel", llama a `cancelAllDialogs` en el contexto de diálogo interno, que borra la pila de diálogos y hace que se cierre con un estado cancelado y sin valor de resultado. Para `MainDialog` (que se muestra más adelante), parece que el diálogo de reservas finalizó y devolvió NULL, de modo similar a cuando el usuario decide no confirmar su reserva.

[!code-javascript[Interrupt](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/cancelAndHelpDialog.js?range=27-40&highlight=7-8,11-12)]

---

## <a name="check-for-interruptions-each-turn"></a>Comprobación de las interrupciones en cada turno

Ahora que hemos analizado cómo funciona la clase de control de interrupciones, vamos a retroceder y ver lo que ocurre cuando el bot recibe un mensaje nuevo del usuario.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\MainDialog.cs**

Cuando llega la nueva actividad de mensaje, el bot ejecuta `MainDialog`. `MainDialog` pide al usuario en qué puede ayudarle. Y, a continuación, inicia `BookingDialog` en el método `MainDialog.ActStepAsync`, con una llamada a `BeginDialogAsync` tal como se muestra a continuación.

[!code-csharp[ActStepAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/MainDialog.cs?range=54-68&highlight=13-14)]

Seguidamente, en el método `FinalStepAsync` de la clase `MainDialog`, el diálogo de reservas finaliza y la reserva se considera completada o cancelada.

[!code-csharp[FinalStepAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/MainDialog.cs?range=70-91)]

El código de `BookingDialog` no se muestra aquí, ya que no está directamente relacionado con el control de interrupciones. Se utiliza para pedir a los usuarios los detalles de la reserva. Puede encontrar dicho código en **Dialogs\BookingDialogs.cs**.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

Cuando llega la nueva actividad de mensaje, el bot ejecuta `MainDialog`. `MainDialog` pide al usuario en qué puede ayudarle. Y, a continuación, inicia `bookingDialog` en el método `MainDialog.actStep`, con una llamada a `beginDialog` tal como se muestra a continuación.

[!code-javascript[Act step](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/mainDialog.js?range=71-88&highlight=16-17)]

Seguidamente, en el método `finalStep` de la clase `MainDialog`, el diálogo de reservas finaliza y la reserva se considera completada o cancelada.

[!code-javascript[Final step](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/mainDialog.js?range=93-110)]

El código de `BookingDialog` no se muestra aquí, ya que no está directamente relacionado con el control de interrupciones. Se utiliza para pedir a los usuarios los detalles de la reserva. Puede encontrar dicho código en **dialogs/bookingDialogs.js**.

---

## <a name="handle-unexpected-errors"></a>Control de errores inesperados

A continuación, se tratan las excepciones no controladas que se produzcan.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**AdapterWithErrorHandler.cs**

En nuestro ejemplo, el controlador `OnTurnError` del adaptador recibe las excepciones producidas por la lógica de turnos del bot. Si se produce una excepción, el controlador elimina el estado de la conversación actual para impedir que el bot se bloquee en un bucle de error causado por estar en mal estado.

[!code-csharp[AdapterWithErrorHandler](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/AdapterWithErrorHandler.cs?range=12-41)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

En nuestro ejemplo, el controlador `onTurnError` del adaptador recibe las excepciones producidas por la lógica de turnos del bot. Si se produce una excepción, el controlador elimina el estado de la conversación actual para impedir que el bot se bloquee en un bucle de error causado por estar en mal estado.

[!code-javascript[AdapterWithErrorHandler](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/index.js?range=28-38)]

---

## <a name="register-services"></a>Registro de los servicios

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs**

Por último, en `Startup.cs`, se crea el bot como transitorio y, en cada turno, se crea una nueva instancia del bot.

[!code-csharp[Add transient bot](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Startup.cs?range=46-47)]

Como referencia, estas son las definiciones de clase que se usan en la llamada para crear el bot anterior.

[!code-csharp[MainDialog signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/MainDialog.cs?range=15)]
[!code-csharp[DialogAndWelcomeBot signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Bots/DialogAndWelcomeBot.cs?range=16)]
[!code-csharp[DialogBot signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Bots/DialogBot.cs?range=18)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

Por último, en `index.js`, se crea el bot.

[!code-javascript[Create bot](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/index.js?range=55-56)]

Como referencia, estas son las definiciones de clase que se usan en la llamada para crear el bot anterior.

[!code-javascript[MainDialog signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/mainDialog.js?range=12)]
[!code-javascript[DialogAndWelcomeBot signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/bots/dialogAndWelcomeBot.js?range=8)]
[!code-javascript[DialogBot signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/bots/dialogBot.js?range=6)]

---

## <a name="to-test-the-bot"></a>Prueba del bot

1. Si aún no lo hizo, instale [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).
1. Ejecute el ejemplo localmente en la máquina.
1. Inicie el emulador, conéctese al bot y envíe mensajes como se muestra a continuación.

<!--![test dialog prompt sample](~/media/emulator-v4/test-dialog-prompt.png)-->

## <a name="additional-information"></a>Información adicional

- El [ejemplo de autenticación](https://aka.ms/logout) muestra cómo controlar el cierre de sesión, que usa un patrón similar al mostrado aquí para controlar las interrupciones.

- Se debería enviar una respuesta predeterminada, en lugar de no hacer nada y dejar al usuario preguntándose qué es lo que pasa. La respuesta predeterminada debe indicar al usuario cuáles son los comandos que entiende el bot, de modo que el usuario pueda retomar el proceso.

- En cualquier punto del turno, la propiedad _responded_ del contexto del turno indica si el bot ha enviado un mensaje al usuario en el turno actual. Antes de que finalice el turno, el bot debería enviar algún mensaje al usuario, aunque sea un simple reconocimiento de su entrada.

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[using-luis]: bot-builder-howto-v4-luis.md
[using-qna]: bot-builder-howto-qna.md

[simple-flow]: bot-builder-dialog-manage-conversation-flow.md
[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-core-sample
[js-sample]: https://aka.ms/js-core-sample

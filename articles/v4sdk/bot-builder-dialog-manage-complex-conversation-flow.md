---
title: Creación de un flujo de conversación avanzado con ramas y bucles | Microsoft Docs
description: Aprenda a administrar un flujo de conversación complejo con diálogos en Bot Framework SDK.
keywords: flujo de conversación complejo, repetir, bucle, menú, diálogos, avisos, cascadas, conjunto de diálogos
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bc11e5a4a5dec1a9588254b3a9d28d56ad163fb4
ms.sourcegitcommit: 409e8f89a1e9bcd0e69a29a313add424f66a81e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2019
ms.locfileid: "67153062"
---
# <a name="create-advanced-conversation-flow-using-branches-and-loops"></a>Creación de un flujo de conversación avanzado con ramas y bucles

[!INCLUDE[applies-to](../includes/applies-to.md)]

Mediante la biblioteca Dialogs es posible administrar flujos de conversación simples y complejos.
En este artículo, le mostraremos cómo administrar conversaciones complejas que se ramifican y enlazan en bucle.
También le mostraremos cómo pasar argumentos entre las distintas partes del diálogo.

## <a name="prerequisites"></a>Requisitos previos

- Conocimiento de [conceptos básicos de los bots][concept-basics], [administración de estado][concept-state], la [biblioteca de diálogos][concept-dialogs] y la [implementación de flujos de conversación secuenciales][simple-dialog].
- Una copia del ejemplo de diálogo complejo en [**CSharp**][cs-sample] o [**JavaScript**][js-sample].

## <a name="about-this-sample"></a>Acerca de este ejemplo

En este ejemplo representa un bot que puede registrar usuarios para revisar hasta dos compañías desde una lista.

`DialogAndWelcomeBot` extiende `DialogBot`, que define los controladores de las diferentes actividades y el controlador de turnos del bot. `DialogBot` ejecuta los diálogos:

- El método _run_ lo usa `DialogBot` para iniciar el diálogo.
- `MainDialog` es el elemento primario de los otros dos diálogos, a los que se llama en determinadas momentos en los diálogos. En este artículo se proporcionan detalles acerca de dichos diálogos.

Los diálogos se dividen en `MainDialog`, `TopLevelDialog` y `ReviewSelectionDialog` diálogos de componentes, que juntos hacen lo siguiente:

- Preguntan el nombre y la edad del usuario, y luego se _ramifican_ en función de la edad del usuario.
  - Si el usuario es demasiado joven, no se le pide que revise ninguna empresa.
  - Si el usuario es lo suficientemente mayor, empiezan a recopilar las preferencias de revisión del usuario.
    - Permiten al usuario seleccionar la empresa que desea revisar.
    - Si el usuario elige una empresa, crean un _bucle_ que permite seleccionar una segunda empresa.
- Por último, agradecen su participación al usuario.

Utilizan diálogos en cascada y algunas indicaciones para administrar una conversación compleja.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

![Flujo de bot complejo](./media/complex-conversation-flow.png)

Para usar diálogos, es preciso que el proyecto instale el paquete de NuGet **Microsoft.Bot.Builder.Dialogs**.

**Startup.cs**

Registramos los servicios del bot en `Startup`. Estos servicios están disponibles para otras partes del código mediante la inserción de dependencias.

- Servicios básicos de un bot: un proveedor de credenciales, un adaptador y la implementación del bot.
- Servicios para administrar el estado: almacenamiento, estado del usuario y estado de la conversación.
- El diálogo que va a usar el bot.

[!code-csharp[ConfigureServices](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Startup.cs?range=22-39)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

![Flujo de bot complejo](./media/complex-conversation-flow-js.png)

Para usar diálogos, el proyecto debe instalar el paquete de npm **botbuilder-dialogs**.

**index.js**

Creamos servicios para el bot que otras partes del código requieren.

- Servicios básicos de un bot: un adaptador y la implementación del bot.
- Servicios para administrar el estado: almacenamiento, estado del usuario y estado de la conversación.
- El diálogo que va a usar el bot.

[!code-javascript[ConfigureServices](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/index.js?range=25-38)]
[!code-javascript[ConfigureServices](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/index.js?range=43-45)]

---

> [!NOTE]
> El almacenamiento en memoria se usa solo con fines de prueba y no está pensado para su uso en producción.
> Asegúrese de usar un tipo de almacenamiento persistente para un bot de producción.

## <a name="define-a-class-in-which-to-store-the-collected-information"></a>Definición de una clase en el que almacenar la información recopilada

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**UserProfile.cs**

[!code-csharp[UserProfile class](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/UserProfile.cs?range=8-16)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**userProfile.js**

[!code-javascript[UserProfile class](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/userProfile.js?range=4-12)]

---

## <a name="create-the-dialogs-to-use"></a>Creación de los diálogos que se usan

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\MainDialog.cs**

Hemos definido un diálogo de componente, `MainDialog`, que contiene un par de pasos principales y dirige los diálogos y advertencias. El paso inicial llama a `TopLevelDialog`, que se explica a continuación.

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/MainDialog.cs?range=31-50&highlight=3)]

**Dialogs\TopLevelDialog.cs**

El diálogo inicial, de nivel superior, tiene cuatro pasos:

1. Preguntar por el nombre del usuario.
1. Preguntar por la edad del usuario.
1. Ramificar en función de la edad del usuario.
1. Por último, agradecer al usuario su participación y devolver la información recopilada.

En el primer paso vamos a borrar el perfil del usuario, por lo que el diálogo se iniciará con un perfil vacío cada vez. Dado que el último paso devolverá información al finalizar, `AcknowledgementStepAsync` concluye guardándola en el estado de usuario y, después, devolviendo dicha información al diálogo principal para su uso en el paso final.

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/TopLevelDialog.cs?range=39-96&highlight=3-4,47-49,56-57)]

**Dialogs\ReviewSelectionDialog.cs**

El diálogo de selección de revisión se inicia desde el diálogo nivel superior `StartSelectionStepAsync` y consta de dos pasos:

1. Pida al usuario que elija una empresa para revisar o elija `done` para terminar.
1. Repita este diálogo o salga, según corresponda.

En este diseño, el diálogo de nivel superior siempre precederá al diálogo de selección de revisión en la pila, y este último se puede considerar un elemento secundario del primero.

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/ReviewSelectionDialog.cs?range=42-106)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

Hemos definido un diálogo de componente, `MainDialog`, que contiene un par de pasos principales y dirige los diálogos y advertencias. El paso inicial llama a `TopLevelDialog`, que se explica a continuación.

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/mainDialog.js?range=43-55&highlight=2)]

**dialogs/topLevelDialog.js**

El diálogo inicial, de nivel superior, tiene cuatro pasos:

1. Preguntar por el nombre del usuario.
1. Preguntar por la edad del usuario.
1. Ramificar en función de la edad del usuario.
1. Por último, agradecer al usuario su participación y devolver la información recopilada.

En el primer paso vamos a borrar el perfil del usuario, por lo que el diálogo se iniciará con un perfil vacío cada vez. Dado que el último paso devolverá información al finalizar, `acknowledgementStep` concluye guardándola en el estado de usuario y, después, devolviendo dicha información al diálogo principal para su uso en el paso final.

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/topLevelDialog.js?range=32-76&highlight=2-3,37-39,43-44)]

**dialogs/reviewSelectionDialog.js**

El diálogo de selección de revisión se inicia desde el diálogo nivel superior `startSelectionStep` y consta de dos pasos:

1. Pida al usuario que elija una empresa para revisar o elija `done` para terminar.
1. Repita este diálogo o salga, según corresponda.

En este diseño, el diálogo de nivel superior siempre precederá al diálogo de selección de revisión en la pila, y este último se puede considerar un elemento secundario del primero.

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/reviewSelectionDialog.js?range=33-78)]

---

## <a name="implement-the-code-to-manage-the-dialog"></a>Implementación del código para administrar el diálogo

El controlador de turno del bot repite el flujo de conversación definido por estos diálogos.
Cuando se recibe un mensaje del usuario:

1. Continúe el diálogo activo, si lo hay.
   - Si no hay ningún diálogo activo, borramos el perfil del usuario e iniciamos el diálogo de nivel superior.
   - Si se completa el diálogo activo, recopilamos la información devuelta y mostramos un mensaje de estado.
   - De lo contrario, el diálogo activo está todavía en medio del proceso, y no necesitamos hacer nada más en este momento.
1. Guarde el estado de la conversación, de modo que se mantengan todas las actualizaciones del estado del diálogo.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**DialogExtensions.cs**

En este ejemplo, se ha definido un método asistente `Run` que se usará para crear el contexto del diálogo y acceder a él.
Puesto que los diálogos de componentes definen un conjunto de diálogos interno, debemos crear un conjunto de diálogos externo que sea visible para el código del controlador de mensajes y usarlo para crear un contexto de diálogo.

- `dialog` es el diálogo del componente principal del bot.
- `turnContext` es el contexto del turno actual del bot.

[!code-csharp[Run method](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/DialogExtensions.cs?range=13-24)]

**Bots\DialogBot.cs**

El controlador de mensajes llama al método asistente `Run` para administrar el diálogo y hemos invalidado el controlador de turnos para guardar los cambios en la conversación y en el estado del usuario que se puedan haber ocurrido durante el turno. La base `OnTurnAsync` llamará al método `OnMessageActivityAsync`, lo que garantiza que se producen llamadas de guardado al final del turno.

[!code-csharp[Overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Bots/DialogBot.cs?range=33-48&highlight=5-7)]

**Bots\DialogAndWelcome.cs**

`DialogAndWelcomeBot` extiende `DialogBot` para proporcionar un mensaje de bienvenida cuando el usuario se une a la conversación y es a lo que llama `Startup.cs`.

[!code-csharp[On members added](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Bots/DialogAndWelcome.cs?range=21-38)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

En este ejemplo, se ha definido un método `run` que se usará para crear el contexto del diálogo y acceder a él.
Puesto que los diálogos de componentes definen un conjunto de diálogos interno, debemos crear un conjunto de diálogos externo que sea visible para el código del controlador de mensajes y usarlo para crear un contexto de diálogo.

- `turnContext` es el contexto del turno actual del bot.
- `accessor` es un descriptor de acceso que hemos creado para administrar el estado del diálogo.

[!code-javascript[run method](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/mainDialog.js?range=32-41)]

**bots/dialogBot.js**

El controlador de mensajes llama al método asistente `run` para administrar el diálogo e implementamos un controlador de turnos para guardar los cambios en la conversación y en el estado del usuario que se puedan haber ocurrido durante el turno. La llamada a `next` permitirá que la implementación base llame al método `onDialog`, lo que garantiza que se producen llamadas de guardado al final del turno.

[!code-javascript[Overrides](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/bots/dialogBot.js?range=30-47)]

**bots/dialogAndWelcomeBot.js**

`DialogAndWelcomeBot` extiende `DialogBot` para proporcionar un mensaje de bienvenida cuando el usuario se une a la conversación y es a lo que llama `Startup.cs`.

[!code-javascript[On members added](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/bots/dialogAndWelcomeBot.js?range=10-21)]

---

## <a name="branch-and-loop"></a>Rama y bucle

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\TopLevelDialog.cs**

Esta es la lógica de la rama de ejemplo desde un paso del diálogo de _nivel superior_:

[!code-csharp[branching logic](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/TopLevelDialog.cs?range=68-80)]

**Dialogs\ReviewSelectionDialog.cs**

Esta es una lógica de bucle de ejemplo desde un paso del diálogo de _selección de revisión_:

[!code-csharp[looping logic](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/ReviewSelectionDialog.cs?range=96-105)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/topLevelDialog.js**

Esta es la lógica de la rama de ejemplo desde un paso del diálogo de _nivel superior_:

[!code-javascript[branching logic](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/topLevelDialog.js?range=56-64)]

**dialogs/reviewSelectionDialog.js**

Esta es una lógica de bucle de ejemplo desde un paso del diálogo de _selección de revisión_:

[!code-javascript[looping logic](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/reviewSelectionDialog.js?range=71-77)]

---

## <a name="to-test-the-bot"></a>Prueba del bot

1. Si aún no lo ha hecho, instale [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).
1. Ejecute el ejemplo localmente en la máquina.
1. Inicie el emulador, conéctese al bot y envíe mensajes como se muestra a continuación.

![ejemplo de diálogo complejo de prueba](~/media/emulator-v4/test-complex-dialog.png)

## <a name="additional-resources"></a>Recursos adicionales

Para ver una introducción acerca de cómo implementar un diálogo, consulte el artículo sobre cómo [implementar el flujo de conversación secuencial][simple-dialog], que utiliza un único diálogo en cascada y unas pocas indicaciones para crear una interacción simple que formule al usuario una serie de preguntas.

La biblioteca de diálogos incluye una validación básica de los mensajes. También puede agregar una validación personalizada. Para más información, consulte el artículo acerca de la [recopilación de datos de entrada del usuario mediante un aviso de diálogo][dialog-prompts].

Para simplificar el código de diálogo y reutilizarlo en varios bots, puede definir partes de un conjunto de diálogo como una clase independiente.
Para más información, consulte [Reutilización de diálogos][component-dialogs].

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Volver a usar diálogos](bot-builder-compositcontrol.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-dialog]: bot-builder-dialog-manage-conversation-flow.md
[dialog-prompts]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-complex-dialog-sample
[js-sample]: https://aka.ms/js-complex-dialog-sample

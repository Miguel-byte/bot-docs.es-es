---
title: Diseño y control del flujo de conversación | Microsoft Docs
description: Obtenga información sobre cómo diseñar y controlar el flujo de conversación en el bot para proporcionar una buena experiencia de usuario.
keywords: diseño, control, flujo de conversación, controlar interrupciones, información general
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/19/2018
ms.openlocfilehash: ae019f7da32969821db0c59a4710e32b57623b14
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563679"
---
::: moniker range="azure-bot-service-3.0"

# <a name="design-and-control-conversation-flow"></a>Diseño y control del flujo de conversación

[!INCLUDE [pre-release-label](./includes/pre-release-label-v3.md)]

En una aplicación tradicional, la interfaz de usuario (UI) es una serie de pantallas. 
Una única aplicación o sitio web puede usar una o varias pantallas según sea necesario para intercambiar información con el usuario. 
La mayoría de las aplicaciones se inician con una pantalla principal a la que los usuarios llegan inicialmente y proporcionan características de navegación que conducen a otras pantallas para realizar distintas funciones, como el inicio de un nuevo pedido, la exploración de productos o la búsqueda de ayuda.

Al igual que las aplicaciones y sitios web, los bots tienen una interfaz de usuario, pero consta de **diálogos**, en lugar de pantallas. Los diálogos ayudan a mantener su posición dentro de una conversación, piden datos de entrada a los usuarios cuando es necesario y ejecutan la validación de la entrada. Son útiles para administrar conversaciones de varios turnos y recopilar información "basada en formularios" simple para realizar actividades como la reserva de un vuelo.

Los diálogos permiten al desarrollador del bot separar de forma lógica las diversas áreas de funcionalidad del bot y guiar el flujo de conversación. Por ejemplo, puede diseñar un diálogo que contenga la lógica que ayuda al usuario a examinar productos y un diálogo independiente que contenga la lógica que ayuda al usuario a crear un nuevo pedido. 

Los diálogos pueden tener o no interfaces gráficas. Pueden contener botones, texto y otros elementos o estar completamente basados en voz. Los diálogos también contienen acciones para realizar tareas, como la invocación de otros diálogos o el procesamiento de entrada del usuario.

## <a name="using-dialogs-to-manage-conversation-flow"></a>Uso de los diálogos para administrar el flujo de conversación

[!INCLUDE [Dialog flow example](./includes/snippet-dotnet-manage-conversation-flow-intro.md)]

Para obtener un tutorial detallado de la administración del flujo de conversación mediante diálogos y Bot Framework SDK, consulte:

- [Administración del flujo de conversación con diálogos (.NET)](./dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [Manage conversation flow with dialogs (Node.js)](./nodejs/bot-builder-nodejs-manage-conversation-flow.md) [Administración del flujo de conversación con diálogos (Node.js)]

## <a name="dialog-stack"></a>Pila de diálogos

Cuando un diálogo invoca a otro, Bot Builder agrega el diálogo nuevo a la parte superior de la pila de diálogos. 
El diálogo que está en la parte superior de la pila es el que controla la conversación. 
Todos los mensajes nuevos enviados por el usuario estarán sujetos a procesamiento por parte de ese diálogo hasta que se cierra o se redirija a otro diálogo. 
Cuando se cierra un diálogo, se quita de la pila y el diálogo anterior de la pila asume el control de la conversación. 

> [!IMPORTANT]
> Para poder diseñar de forma eficaz el flujo de conversación de un bot es fundamental comprender el concepto de cómo Bot Builder construye y deconstruye la pila de diálogos mientras los diálogos se invocan entre sí, se cierran, etc. 

## <a name="dialogs-stacks-and-humans"></a>Diálogos, pilas y humanos

Puede resultar tentador suponer que los usuarios navegarán a través de los diálogos creando una pila de diálogos y que, en algún momento, volverán a la dirección de la que vinieron, desapilando los diálogos uno por uno de forma clara y ordenada. 
Por ejemplo, el usuario comienza en el diálogo raíz, invoca el diálogo de nuevo pedido desde allí y, a continuación, invoca el diálogo de búsqueda de productos. 
A continuación, el usuario selecciona un producto y lo confirma, saliendo del diálogo de búsqueda de productos, completa el pedido, saliendo del diálogo de nuevo pedido, y vuelve al diálogo raíz. 

Aunque sería estupendo si los usuarios siempre recorriesen dicha ruta lógica y lineal, esto se produce raras veces. 
Los humanos no se comunican en "pilas". Tienden a cambiar de opinión con frecuencia. 
Considere el siguiente ejemplo: 

![bot](./media/bot-service-design-conversation-flow/stack-issue.png)

Mientras que el bot podría haber construido lógicamente una pila de diálogos, el usuario podría decidir hacer algo completamente diferente o formular una pregunta que no tenga ninguna relación con el tema actual. 
En el ejemplo, el usuario hace una pregunta en lugar de facilitar la respuesta afirmativa o negativa que el diálogo espera. 
¿Cómo debe responder el diálogo?

- Insistir en que el usuario responda primero a la pregunta. 
- Pasar por alto todo lo que el usuario ha hecho hasta el momento, restablecer la pila de diálogos al completo y empezar desde cero intentando responder a la pregunta del usuario. 
- Intentar responder a la pregunta del usuario y, después, regresar a la pregunta con respuesta afirmativa o negativa para intentar reanudar la conversación desde ahí. 

No hay una respuesta *correcta* a esta pregunta, ya que la mejor solución dependerá de los aspectos específicos del escenario y de cómo espera el usuario que responda el bot. Sin embargo, a medida que la complejidad de la conversación aumenta, los **diálogos** resultan más difíciles de administrar. Para situaciones de bifurcaciones complejas, puede ser más fácil crear su propio flujo de lógica de control para realizar un seguimiento de la conversación de los usuarios.

## <a name="next-steps"></a>Pasos siguientes

La administración de la navegación del usuario en los diálogos y el diseño del flujo de conversación de forma que permita a los usuarios lograr sus objetivos (incluso de forma no lineal) son un desafío fundamental del diseño del bot. 
En el [siguiente artículo](./bot-service-design-navigation.md) se revisan algunas dificultades habituales de una navegación mal diseñada y se describen las estrategias para evitar estos problemas. 

::: moniker-end

::: moniker range="azure-bot-service-4.0"

# <a name="design-and-control-conversation-flow"></a>Diseño y control del flujo de conversación

En una aplicación tradicional, la interfaz de usuario (UI) consta de una serie de pantallas y una sola aplicación o sitio web puede usar una o varias pantallas según sea necesario para intercambiar información con el usuario.
La mayoría de las aplicaciones se inician con una pantalla principal a la que los usuarios llegan inicialmente y esa pantalla proporciona características de navegación que conducen a otras pantallas para realizar distintas funciones, como el inicio de un nuevo pedido, la exploración de productos o la búsqueda de ayuda.

Al igual que las aplicaciones y sitios web, los bots tienen una interfaz de usuario, pero esta consta de **mensajes**, en lugar de pantallas. Los mensajes pueden contener botones, texto y otros elementos o estar completamente basados en voz.

Mientras que una aplicación tradicional o un sitio web puede solicitar varios fragmentos de información en una pantalla a la vez, un bot recopilará la misma cantidad de información mediante varios mensajes. De este modo, el proceso de recopilación de información del usuario es una experiencia activa, donde el usuario tiene una conversación activa con el bot.

Un bot bien diseñado tendrá un flujo de conversación que parece natural. El bot debería ser capaz de controlar la conversación principal sin problemas, así como las interrupciones o cambios de temas de las conversaciones correctamente.

## <a name="procedural-conversation-flow"></a>Flujo de conversación de procedimientos

Por lo general, la conversación con un bot se centra en torno a la tarea que el bot intenta conseguir, lo cual se denomina flujo de procedimientos. Aquí es donde el bot hace una serie de preguntas al usuario para recopilar toda la información que necesita para poder procesar la tarea.

En un flujo de conversación de procedimientos, usted define el orden de las preguntas y el bot las formulará en el orden definido. Puede organizar las preguntas en *módulos* lógicos para mantener el código centralizado mientras se centra en guiar la conversación. Por ejemplo, puede diseñar un módulo que contenga la lógica que ayuda al usuario a examinar productos y un módulo independiente que contenga la lógica que ayuda al usuario a crear un nuevo pedido.

Puede estructurar estos módulos para que fluyan como quiera, desde una forma libre a una secuencial. Bot Framework SDK ofrece varias bibliotecas que le permiten generar cualquier flujo de conversación que el bot necesite. Por ejemplo, la biblioteca `prompts` le permite solicitar entradas a los usuarios, la biblioteca `waterfall` le permite definir una secuencia de pares de preguntas y respuestas, la biblioteca `dialog control` le permite modular la lógica del flujo de conversación, etc. Todas estas bibliotecas están vinculadas entre sí a través de un objeto `dialogs`. Echemos un vistazo a cómo se implementan los módulos como `dialogs` para diseñar y administrar flujos de conversación y ver cómo ese flujo es similar al flujo de aplicación tradicional.

![bot](./media/designing-bots/core/dialogs-screens.png)

En una aplicación tradicional, todo comienza con la **pantalla principal**.
La **pantalla principal** invoca la **pantalla de nuevo pedido**.
La **pantalla de nuevo pedido** permanece en control hasta que se cierra o invoca otras pantallas, como la **pantalla de búsqueda de productos**.
Si la **pantalla de nuevo pedido** se cierra, se devuelve al usuario a la **pantalla principal**.

En un bot, todo comienza con el **diálogo raíz**.
El **diálogo raíz** invoca el **diálogo de nuevo pedido**.
En ese momento, el **diálogo de nuevo pedido** toma el control de la conversación y permanece en control hasta que se cierra o invoca otros diálogos, como el **diálogo de búsqueda de productos**.
Si el **diálogo de nuevo pedido** se cierra, se devuelve el control de la conversación al **diálogo raíz**.

Para obtener ejemplos de cómo implementar el flujo de conversación, consulte [Creación de mensajes propios para recopilar datos de entrada del usuario](./v4sdk/bot-builder-primitive-prompts.md) y use los cuadros de diálogo para [implementar un flujo de conversación secuencial](./v4sdk/bot-builder-dialog-manage-conversation-flow.md).

## <a name="handle-interruptions"></a>Control de las interrupciones

Puede resultar tentador suponer que los usuarios realizarán tareas de procedimientos una por una de forma clara y ordenada.
Por ejemplo, en un flujo de conversación de procedimientos mediante `dialogs`, el usuario inicia en el diálogo raíz, invoca el diálogo de nuevo pedido desde allí y, a continuación, invoca el diálogo de búsqueda de productos. A continuación, el usuario selecciona un producto y lo confirma, saliendo del diálogo de búsqueda de productos, completa el pedido, saliendo del diálogo de nuevo pedido, y vuelve al diálogo raíz.

Aunque sería estupendo si los usuarios siempre recorriesen dicha ruta lógica y lineal, esto se produce raras veces.
Los humanos no se comunican en `dialogs` secuenciales. Tienden a cambiar de opinión con frecuencia.
Considere el siguiente ejemplo:

![bot](./media/bot-service-design-conversation-flow/stack-issue.png)

Aunque que el bot puede estar centrado en procedimientos, el usuario puede decidir hacer algo completamente diferente o formular una pregunta que no tenga ninguna relación con el tema actual.
En el ejemplo anterior, el usuario hace una pregunta en lugar de facilitar la respuesta afirmativa o negativa que el bot espera.
¿Cómo debe responder el bot?

- Insistir en que el usuario responda primero a la pregunta.
- Pasar por alto todo lo que el usuario ha hecho hasta el momento, restablecer la pila de diálogos al completo y empezar desde cero intentando responder a la pregunta del usuario.
- Intentar responder a la pregunta del usuario y, después, regresar a la pregunta con respuesta afirmativa o negativa para intentar reanudar la conversación desde ahí.

No hay ninguna respuesta *correcta* a esta pregunta, ya que la mejor solución dependerá de los aspectos concretos del escenario y de cómo espera el usuario que responda el bot. Para más información, consulte [Handle user interrupt](v4sdk/bot-builder-howto-handle-user-interrupt.md) (Control de las interrupciones del usuario).

## <a name="next-steps"></a>Pasos siguientes

La administración de la navegación del usuario en los diálogos y el diseño del flujo de conversación de forma que permita a los usuarios lograr sus objetivos (incluso de forma no lineal) son un desafío fundamental del diseño del bot.
En el [siguiente artículo](~/bot-service-design-navigation.md) se revisan algunas dificultades habituales de una navegación mal diseñada y se describen las estrategias para evitar estos problemas.

::: moniker-end

---
title: Diseño de navegación de bots | Microsoft Docs
description: Obtenga información sobre cómo diseñar una estructura de navegación adecuada para el bot y cómo evitar los errores más comunes de diseño de navegación.
keywords: navegación, información general, bot terco, bot despistado, bot misterioso, bot de capitán obviedad, bot que no puede olvidar
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: f2a97b35f7e83a825e533be528951e8c04c521a1
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998090"
---
# <a name="design-bot-navigation"></a>Diseño de navegación de bots

Los usuarios pueden navegar por sitios web con las rutas de navegación, por aplicaciones con los menús y por navegadores web con botones, como **adelante** y **atrás**. Sin embargo, ninguna de estas técnicas de navegación estandarizados aborda por completo los requisitos de navegación dentro de un bot. Como se explicó [anteriormente](~/bot-service-design-conversation-flow.md#handle-interruptions), los usuarios interactúan a menudo con bots de forma que no es lineal, lo que dificulta el diseño de una navegación de bots que siempre ofrezca una experiencia de usuario excelente. 

Considere los dilemas siguientes:

- ¿Cómo se asegura de que un usuario no se pierda en una conversación con un bot? 
- ¿Puede un usuario navegar "atrás" en una conversación con un bot? 
- ¿Cómo hace un usuario para ir al "menú principal" durante una conversación con un bot? 
- ¿Cómo hace un usuario para "cancelar" una operación durante una conversación con un bot? 

Los detalles de diseño de navegación de su bot dependerán en gran medida las características y funcionalidad que admita el bot. Independientemente del tipo de bot que esté desarrollando, querrá evitar los escollos comunes de las interfaces conversacionales mal diseñadas. En este artículo se describen estos riesgos en cuanto a cinco personalidades: el "bot terco", el "bot despistado", el "bot misterioso", el "bot capitán obviedad" y el"bot que no puede olvidar". 

> [!TIP]
> Mitigar cada una de estas personalidades en su bot suele ser posible si se [controlan interrupciones de los usuarios](v4sdk/bot-builder-howto-handle-user-interrupt.md) correctamente.

## <a name="the-stubborn-bot"></a>El "bot terco"

El bot terco insiste en mantener el curso actual de la conversación, incluso cuando el usuario intenta dirigir las cosas en otra dirección. 

Considere el siguiente escenario: 

![bot](~/media/bot-service-design-navigation/stubborn-bot-new.png)

A menudo, los usuarios cambian de parecer, deciden cancelar o a veces desean volver empezar por completo. 

> [!TIP]
> <b>Sí</b>: diseñe el bot para que tenga en cuenta que un usuario puede intentar cambiar el curso de la conversación en cualquier momento. 
>
> <b>No</b>: diseñe el bot para ignorar la entrada del usuario y seguir repitiendo la misma pregunta en un bucle interminable. 

Existen muchos métodos para evitar este problema, pero quizás la manera más fácil de impedir que un bot haga la misma pregunta constantemente sea simplemente especificar un número máximo de reintentos para cada pregunta. Si se diseña de este modo, el bot no hace nada "inteligente" para comprender la entrada del usuario y responder de manera apropiada, pero al menos evitará hacer la misma pregunta en un bucle infinito. 

## <a name="the-clueless-bot"></a>El "bot despistado"

El bot despistado responde de manera absurda cuando no entiende el intento de un usuario por acceder a cierta funcionalidad. Un usuario puede intentar comandos comunes de palabra clave, como "ayuda" o "cancelar" con la expectativa razonable de que el bot responderá correctamente.

Considere el siguiente escenario: 

![bot](~/media/bot-service-design-navigation/clueless-bot.png)

Aunque es posible que se vea tentado a diseñar cada cuadro de diálogo en el bot para que escuche ciertas palabras clave y responda de manera apropiada a ellas, no se recomienda este enfoque. 

> [!TIP]
> <b>Sí</b>: implemente [software intermedio](v4sdk/bot-builder-create-middleware.md) que examine la entrada del usuario para las palabras clave que especifique (por ejemplo, "ayuda", "cancelar", "empezar de nuevo", etc.) y responder según corresponda. 
> 
> <b>No</b>: diseñe cada cuadro de diálogo para examinar la entrada del usuario en busca de una lista de palabras clave. 

Al definir la lógica en el **software intermedio**, la hará accesible a cada intercambio con el usuario. Con este enfoque, los avisos y cuadros de diálogo individuales pueden crearse para pasar por alto las palabras clave con seguridad, si es necesario.

## <a name="the-mysterious-bot"></a>El "bot misterioso"

El bot misterioso no reconoce de inmediato la entrada del usuario de ninguna manera. 

Considere el siguiente escenario: 

![bot](~/media/bot-service-design-navigation/mysterious-bot.png)

En algunos casos, esta situación podría ser un indicador de que el bot tiene una interrupción del servicio. Sin embargo, podría ser simplemente que el bot está ocupado procesando la entrada del usuario y aún no ha terminado de compilar la respuesta. 

> [!TIP]
> <b>Sí</b>: diseñe el bot para que reconozca de inmediato la entrada del usuario, incluso en casos donde el bot pueda tardar algún tiempo en compilar la respuesta. 
> 
> <b>No</b>: diseñe el bot para posponer la confirmación de la entrada del usuario hasta que el bot termine de compilar la respuesta.

Al reconocer de inmediato la entrada del usuario, elimina cualquier posibilidad de confusión en cuanto al estado del bot. Si la respuesta tarda mucho tiempo en compilarse, considere la posibilidad de enviar un mensaje de "escritura" para indicar que el bot está trabajando y, a continuación, envíe un [mensaje proactivo](v4sdk/bot-builder-howto-proactive-message.md).

## <a name="the-captain-obvious-bot"></a>El "bot capitán obviedad"

El bot capitán obviedad proporciona información que no se pidió y que es totalmente obvia y, por tanto, inútil para el usuario. 

Considere el siguiente escenario:

![bot](~/media/bot-service-design-navigation/captainobvious-bot.png)

> [!TIP]
> <b>Sí</b>: diseñe el bot para proporcionar información que sea útil para el usuario. 
> 
> <b>No</b>: diseñe el bot para proporcionar información que el usuario no pidió y probablemente le resulte inútil.

Al diseñar el bot para proporcionar información útil, aumentará la probabilidad de que el usuario interactúe con el bot.

## <a name="the-bot-that-cant-forget"></a>El "bot que no puede olvidar"

El bot que no puede olvidar integra de manera incorrecta información de conversaciones anteriores a la conversación actual. 

Considere el siguiente escenario:

![bot](~/media/bot-service-design-navigation/rememberall-bot.png)

> [!TIP]
> <b>Sí</b>: diseñe el bot para que mantenga el tema actual de la conversación, a menos/hasta que el usuario exprese su deseo de volver a un tema anterior. 
> 
> <b>No</b>: diseñe el bot para interponer información de las conversaciones anteriores cuando no sea pertinente para la conversación actual.

Al mantener el tema actual de la conversación, reduce la posibilidad de confusión y frustración, así como aumenta la probabilidad de que el usuario continúe interactuando con el bot.

## <a name="next-steps"></a>Pasos siguientes

Al diseñar el bot para que evite estos problemas comunes de interfaces conversacionales mal diseñadas, dará un paso importante para garantizar una experiencia de usuario excelente. 

A continuación, obtenga más información sobre los [elementos de la experiencia de usuario](~/bot-service-design-user-experience.md) en que los bots se basan normalmente para intercambiar información con los usuarios. 

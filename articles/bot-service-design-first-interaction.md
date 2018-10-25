---
title: Diseño de la primera interacción del usuario de un bot | Microsoft Docs
description: Obtenga información sobre lo que es una excelente primera experiencia del usuario y cómo diseñar los bots para una operación correcta.
keywords: primera impresión, inicio, lenguaje frente a menú
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: f59a7acbdb7d580aebeef6ffe81d8b15505aed90
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999812"
---
# <a name="design-a-bots-first-user-interaction"></a>Diseño de la primera interacción del usuario de un bot

## <a name="first-impressions-matter"></a>Las primeras impresiones son importantes

La primera interacción entre el usuario y el bot es fundamental para la experiencia del usuario. Al diseñar su bot, tenga en cuenta que ese primer mensaje es más que solo decir "Hola". Al compilar una aplicación, la primera pantalla se diseña de manera que brinde importantes indicaciones de [navegación](bot-service-design-navigation.md). Los usuarios deben entender de manera intuitiva aspectos como dónde se ubica el menú y cómo funciona, dónde obtener ayuda, cuál es la directiva de privacidad, etc. Cuando se diseña un bot, la primera interacción del usuario con el bot debe proporcionar el mismo tipo de información. 

## <a name="language-versus-menus"></a>Lenguaje frente a menús 

Considere estos dos diseños:

### <a name="design-1"></a>Diseño 1

![bot](~/media/bot-service-design-first-interaction/hello1.png)


### <a name="design-2"></a>Diseño 2

![bot](~/media/bot-service-design-first-interaction/hello2.png)

Por lo general, no se recomienda iniciar el bot con una pregunta abierta como "¿En qué puedo ayudarlo?" Si el bot puede hacer cien cosas distintas, lo más probable es que los usuarios no puedan adivinar la mayoría de ellas. El bot no les indica lo que puede hacer, así es que es difícil que ellos puedan saberlo.

Los menús proporcionan una solución sencilla para ese problema. En primer lugar, al enumerar las opciones disponibles, el bot transmite sus funcionalidades al usuario. En segundo lugar, los menús evitan que el usuario tenga que escribir mucho; en lugar de eso, simplemente pueden hacer clic. Por último, el uso de los menús puede simplificar considerablemente los modelos de lenguaje natural al limitar el ámbito de la entrada que el bot podría recibir por parte del usuario. 

> [!TIP]
> Los menús son una herramienta valiosa en el momento de diseñar los bots para lograr una excelente experiencia del usuario; no los descarte por no ser "lo suficientemente inteligentes". Puede diseñar el bot para que use los menús a la vez que mantiene la compatibilidad con la entrada de forma libre. Si un usuario escribe para responder al menú inicial en lugar de seleccionar una opción del menú, el bot podría intentar analizar la entrada de texto del usuario. 

También puede formular preguntas más precisas para guiar al usuario si el bot tiene una función específica. Por ejemplo, si el bot es responsable de tomar pedidos de sándwiches, la primera interacción podría ser "Hola, voy a tomar su pedido. ¿Qué tipo de pan le gustaría? Tenemos pan blanco, de trigo o centeno". De este modo, el usuario sabe cómo responder y recibe indicaciones de navegación a través de la conversación.

## <a name="other-considerations"></a>Otras consideraciones

Además de proporcionar una primera interacción intuitiva y fácil de navegar, un bot bien diseñado proporciona al usuario acceso a información sobre su directiva de privacidad y sus términos de uso. 

> [!TIP]
> Si el bot recopila información personal del usuario, es importante que se transmita y se describa lo que se hará con los datos.

## <a name="next-steps"></a>Pasos siguientes

Ahora que está familiarizado con algunos principios básicos del diseño de la primera interacción entre el usuario y el bot, obtenga más información sobre cómo [diseñar el flujo de la conversación](~/bot-service-design-conversation-flow.md).

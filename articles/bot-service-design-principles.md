---
title: Principios de diseño de bots | Microsoft Docs
description: Obtenga información sobre qué es lo que hace que un bot de conversación sea bueno y cómo planear y diseñar bots para que se ajusten a sus necesidades y satisfagan a sus usuarios.
keywords: procedimientos recomendados, diseño de bots
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 27553a88ee4373eacb089eb6358520820509e195
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297594"
---
# <a name="principles-of-bot-design"></a>Principios de diseño de bots

Bot Framework permite que los desarrolladores creen experiencias de bot atractivas que solucionen una gran variedad de problemas empresariales. Una vez haya aprendido los conceptos descritos en esta sección, estará preparado para diseñar un bot que se alinee con los procedimientos recomendados y aproveche las lecciones aprendidas hasta ahora en esta área relativamente nueva. 

## <a name="designing-a-bot"></a>Diseñar un bot

Si está creando un bot, se puede suponer que espera que los usuarios lo usen. También se puede suponer que espera que los usuarios preferirán la experiencia del bot a través de experiencias alternativas, como aplicaciones, sitios web, llamadas telefónicas y otros medios, para abordar sus necesidades concretas. En otras palabras, el bot compite por el tiempo de los usuarios contra cosas como aplicaciones y sitios web. Por lo tanto, ¿cómo puede maximizar las probabilidades de que su bot consiga el objetivo final de captar y retener a los usuarios? Es simplemente cuestión de dar prioridad a los factores correctos a la hora de diseñar el bot.

## <a name="factors-that-do-not-guarantee-a-bots-success"></a>Factores que no garantizan el éxito de un bot

Al diseñar su bot, tenga en cuenta que ninguno de los siguientes factores necesariamente garantiza el éxito de un bot: 

- **La "inteligencia" del bot**: en la mayoría de los casos, es poco probable que hacer que su bot sea más inteligente garantice la satisfacción de los usuarios o la adopción de su plataforma. En realidad, muchos bots tienen funcionalidades de lenguaje natural o aprendizaje automático poco avanzadas. Un bot puede incluir esas funcionalidades si son necesarias para resolver los problemas para los cuales ha sido diseñado, pero no debe suponer que hay ninguna correlación entre la inteligencia de un bot y la adopción del bot por el usuario.

- **Cantidad de lenguaje natural que admite el bot**: el bot puede ser ideal para las conversaciones. Puede tener un gran vocabulario e incluso puede hacer bromas excelentes. Pero, a menos que resuelva los problemas que los usuarios necesitan resolver, estas funcionalidades pueden contribuir muy poco para lograr que su bot tenga éxito. De hecho, algunos bots no tienen ninguna funcionalidad de conversación. Y, en muchos casos, esto no presenta ningún problema.

- **Voz**: la habilitación de la voz en los bots no siempre implica que proporcionen buenas experiencias de usuario. A menudo, obligar a los usuarios a usar la voz puede provocar una experiencia de usuario frustrante. Al diseñar su bot, considere siempre si la voz es el canal apropiado para el problema especificado. ¿Habrá un entorno ruidoso? ¿La voz transmitirá la información que necesita compartirse con el usuario? 

## <a name="factors-that-do-influence-a-bots-success"></a>Factores que influyen en el éxito de un bot

La mayoría de las aplicaciones y sitios web con mayor éxito tienen al menos una cosa en común: una experiencia de usuario excelente. Los bots no son distintos en ese aspecto. Por lo tanto, garantizar una experiencia de usuario excelente debe ser su primera prioridad a la hora de diseñar un bot. Entre algunas consideraciones clave se incluyen:

- ¿El bot soluciona fácilmente el problema del usuario con el número mínimo de pasos?

- ¿El bot soluciona el problema del usuario mejor, más fácilmente o más rápidamente que cualquiera de los servicios alternativos?

- ¿El bot se ejecuta en dispositivos y plataformas que interesan al usuario?

- ¿El bot se puede detectar? ¿Los usuarios saben de forma natural qué hacer cuando lo usan?

Ninguna de estas preguntas se relaciona directamente con factores tales como: la inteligencia del bot, la cantidad de lenguaje natural que tiene, si usa aprendizaje automático o qué lenguaje de programación se usó para crearlo. Los usuarios no suelen interesarse por ninguno de estos aspectos si el bot resuelve el problema que necesitan abordar y ofrece una experiencia de usuario excelente. Una experiencia de usuario excelente del bot no requiere que los usuarios escriban demasiado, hablen demasiado, se repitan a sí mismos varias veces o expliquen los aspectos que el bot debe saber automáticamente.

> [!TIP]
> Independientemente del tipo de aplicación que cree (bot, sitio web o aplicación), priorice la experiencia del usuario.

El proceso para diseñar un bot es como el proceso para diseñar una aplicación o un sitio web, por lo que las lecciones aprendidas a lo largo de las décadas con la creación de interfaces de usuario y la entrega de experiencias de usuario para aplicaciones y sitios web aún se aplican para el diseño de bots. 

Siempre que no esté seguro sobre si el enfoque de diseño es adecuado para su bot, retroceda y hágase la siguiente pregunta: ¿cómo resolvería ese problema en una aplicación o un sitio web? Lo más probable es que la misma respuesta se aplique para el diseño del bot. 

## <a name="next-steps"></a>Pasos siguientes

Ahora que está familiarizado con algunos principios básicos del diseño del bot, obtenga más información sobre el diseño de la experiencia del usuario y patrones comunes en lo que sigue de esta sección.
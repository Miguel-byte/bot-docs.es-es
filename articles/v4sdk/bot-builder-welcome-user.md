---
redirect_url: /bot-framework/bot-builder-send-welcome-message
ms.openlocfilehash: ef281fd1b6539484c06f68caffbd4e87ec8acc2b
ms.sourcegitcommit: cb0b70d7cf1081b08eaf1fddb69f7db3b95b1b09
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/09/2018
ms.locfileid: "51332909"
---
<a name="--"></a><!--
---
title: Bienvenida al usuario | Microsoft Docs description: Obtenga información para diseñar un bot que proporcione una experiencia de bienvenida al usuario.
keywords: introducción, diseño, experiencia de usuario, bienvenida, experiencia personalizada author: dashel ms.author: dashel manager: kamrani ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date: 8/30/2018 monikerRange: 'azure-bot-service-4.0'
---

# <a name="welcoming-the-user"></a>Dar la bienvenida al usuario

El objetivo principal al crear cualquier bot es que el usuario participe en una conversación que tenga sentido. Una de las mejores formas de lograr este objetivo es asegurarse de que desde el momento en que un usuario se conecta por primera vez, comprende la finalidad principal del bot y sus funcionalidades, es decir, el motivo de que se haya creado el bot.

## <a name="show-your-purpose"></a>Mostrar su finalidad

Imagine por un momento la experiencia de un usuario que se conecta al bot llamado "Información turística de Chicago" con el fin de informarse sobre los hoteles de Chicago. Poco se imagina que el bot es en realidad para los entusiastas de las pizzas estilo Chicago y solo ofrece sugerencias de restaurantes. Una vez que ha quedado claro que las preguntas no se están respondiendo correctamente, este usuario se desconectaría rápidamente y consideraría que el sitio proporciona una mala experiencia en línea. Una mala experiencia como esta para el usuario puede evitarse dando la bienvenida a los usuarios con información suficiente para que comprendan la finalidad principal y las funcionalidades del bot. 

![Mensaje de bienvenida](./media/welcome_message.png)

Tras leer el mensaje de bienvenida, si el bot no proporciona el tipo de información que un usuario desea, este puede avanzar rápidamente antes de experimentar una interacción frustrante.
Cada vez que los usuarios interactúan por primera vez con el bot, se debe generar un mensaje de bienvenida. Para lograr esto, puede supervisar los tipos de **actividad** del bot y esperar nuevas conexiones. Según el canal, cada nueva conexión puede generar hasta dos actividades de actualización de la conversación.

- Una cuando el bot del usuario está conectado a la conversación.
- Y otra cuando el usuario se une a la conversación.

Resulta tentador generar simplemente un mensaje de bienvenida cada vez que se detecta una nueva actualización de la conversación, pero eso puede provocar resultados inesperados cuando se accede al bot mediante diversos canales.

## <a name="design-for-different-channels"></a>Diseñar para diferentes canales

Aunque tiene control total sobre la información presentada por el bot, puede que no tenga control sobre cómo los diferentes canales implementan la presentación de esa información. Algunos canales crean una actualización de la conversación cuando un usuario se conecta inicialmente a ese canal y otra solo después de recibir un mensaje de entrada inicial del usuario. Otros canales generan ambas actividades cuando el usuario se conecta inicialmente al canal. Si simplemente espera un evento de actualización de la conversación y muestra un mensaje de bienvenida en un canal con dos actividades de actualización de la conversación, el usuario podría recibir lo siguiente:

![Doble mensaje de bienvenida](./media/double_welcome_message.png)

Este mensaje duplicado se puede evitar mediante la generación de un mensaje de bienvenida inicial solo para el segundo evento de actualización de la conversación. El segundo evento se puede detectar en estos dos casos:
- Se ha producido un evento de actualización de la conversación.
- Se ha agregado un nuevo miembro (usuario) a la conversación.

También es importante tener en cuenta cuándo la intervención del usuario puede contener realmente información útil, y esto también puede variar según el canal. Si un canal genera ambas actividades de actualización de la conversación tras la conexión inicial de un bot, la primera entrada del usuario se puede evaluar correctamente como una respuesta al aviso del mensaje de bienvenida, como la conversación mostrada a continuación.

![Respuesta de entrada única](./media/single_input_response.png)

Sin embargo, si un canal espera la intervención inicial de un usuario antes de generar un segundo evento de actualización de la conversación, el mismo código exacto usado anteriormente produciría en su lugar la siguiente experiencia de usuario.

![Respuesta incorrecta de entrada única](./media/single_input_wrong_response.png)

Para asegurarse de que los usuarios tengan una buena experiencia en todos los canales posibles, un procedimiento recomendado es no esperar información útil dentro de la entrada inicial de conversación de un usuario. En su lugar, considere la entrada inicial como datos "desperdiciados" y, tras la recepción, pida al usuario que le proporcione la información necesaria para continuar la conversación. Esta técnica generará una experiencia de usuario coherente con independencia del canal que se use para acceder inicialmente a su bot.

![Deshacerse de la primera entrada](./media/no_first_input_response.png)

## <a name="personalize-the-user-experience"></a>Personalizar la experiencia del usuario

No hay nada que haga sentir a un usuario menos bienvenido y más impersonal que pedirle continuamente información que ya ha proporcionado. Si su bot conserva información de los usuarios que han visitado anteriormente, es una buena idea comprobar esta información la primera vez y, si está disponible, darle la bienvenida de nuevo al usuario por su nombre almacenado de una visita anterior. 

![Mensaje de bienvenido de nuevo](./media/welcome_back.png)

En este caso, la lógica de la conversación omite la solicitud de un nombre y continúa a la siguiente actividad de conversación.

Su bot también puede personalizar la experiencia del usuario mediante la respuesta oportuna a intervenciones del usuario inesperadas, como solicitudes para finalizar una conversación.

![Responder a la solicitud de salida](./media/respond_to_exit.png)

Mantener las interacciones oportunas y conversacionales ayuda a los usuarios a experimentar una interacción más acogedora y divertida con el bot.

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Envío de mensajes de bienvenida a los usuarios](bot-builder-send-welcome-message.md)

-->

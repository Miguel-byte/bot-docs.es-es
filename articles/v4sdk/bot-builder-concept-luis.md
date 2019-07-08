---
title: Language Understanding | Microsoft Docs
description: Aprenda a agregar inteligencia artificial a los bots con Microsoft Cognitive Services para que sean más útiles y atractivos.
keywords: LUIS, intención, reconocedor, herramienta de distribución, qna, qna maker
author: ivorb
ms.author: v-ivorb
manager: rstand
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 09/19/2018
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0c0918b0ac0a10927bd8d7c52283e74b4fd480bf
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404644"
---
# <a name="language-understanding"></a>Language Understanding

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Los bots pueden usar una variedad de estilos de conversación, desde estructurados y guiados a los de forma libre y abiertos. Un bot debe decidir qué hacer a continuación en su flujo de conversación según lo que haya dicho el usuario, y en una conversación abierta, hay una gama más amplia de respuestas de usuario.

| Guiado | Abrir |
|------|------|
| Soy el bot de viajes. Seleccione una de las opciones siguientes: buscar vuelos, buscar hoteles, buscar coches de alquiler. | Puedo ayudarle a reservar un viaje. ¿Qué le gustaría hacer? |
| ¿Necesita algo más? Haga clic en Sí o No. | ¿Necesita algo más? |

La interacción entre los usuarios y los bots suele ser de forma libre, y los bots deben comprender el lenguaje natural y contextual. En este artículo se explica cómo Language Understanding (LUIS) ayuda a determinar lo que los usuarios quieren, a identificar conceptos y entidades en las frases y, finalmente, permitir que los bots respondan con la acción apropiada.

## <a name="recognize-intent"></a>Reconocimiento de intenciones

[LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/home) ayuda a determinar la **intención** de los usuarios (lo que quieren hacer) en función de lo que dicen, para que el bot pueda responder de forma correcta. LUIS es especialmente útil cuando lo que dicen al bot no sigue una estructura predecible o un patrón específico. Si un bot tiene una interfaz de usuario de conversación, en la que el usuario habla o escribe una respuesta, puede haber infinitas variantes de *expresiones*, que son la entrada de texto o voz del usuario.

Por ejemplo, considere las numerosas formas en las que un usuario de un bot de viajes puede pedir la reserva de un vuelo.

![Expresiones de varias formas diferentes para reservar un vuelo](media/cognitive-services-add-bot-language/cognitive-services-luis-utterances.png)

Estas expresiones pueden tener otras estructuras y contener varios sinónimos de "vuelo" en los que no haya pensado. En el bot, puede ser complicado escribir la lógica que coincida con todas las expresiones y distinguirlas de otras intenciones que contengan las mismas palabras. Además, el bot tiene que extraer las *entidades*, que son otros palabras importantes, como las ubicaciones y las horas. LUIS facilita este proceso mediante la identificación contextual de las intenciones y las entidades de forma automática.

Al diseñar el bot para la entrada de lenguaje natural, se determinan las intenciones y entidades que el bot debe reconocer, y se piensa en cómo se conectarán a las acciones que realiza el bot. En [luis.ai](https://www.luis.ai), se definen intenciones y entidades personalizadas y para especificar su comportamiento se proporcionan ejemplos para cada intención y se etiquetan las entidades que contienen.

El bot usa la intención reconocida por LUIS para determinar el tema de conversación, o bien iniciar un flujo de conversación. Por ejemplo, cuando un usuario dice "Me gustaría reservar un vuelo", el bot detecta la intención ReservarVuelo e invoca el flujo de conversación para iniciar una búsqueda de vuelos. LUIS detecta entidades como la ciudad de destino y la fecha de salida, en la expresión original que desencadena la intención y más adelante en el flujo de conversación. Una vez que el bot tiene toda la información que necesita, puede satisfacer la intención del usuario.

![Una conversación con un bot desencadenada con la intención ReservarVuelo](media/cognitive-services-add-bot-language/cognitive-services-luis-conversation-high-level.png)

### <a name="recognize-intent-in-common-scenarios"></a>Reconocimiento de intenciones en escenarios comunes

Para ahorrar tiempo de desarrollo, LUIS proporciona modelos de lenguaje previamente entrenados que reconocen expresiones comunes para categorías comunes de bots. 

Los **dominios creados previamente** son colecciones de intenciones y entidades previamente entrenadas y listas para usar que funcionan bien para escenarios comunes como citas, recordatorios, administración, fitness, entretenimiento, comunicación, reservas y muchos otros. El dominio creado previamente **Utilidades** ayuda a que el bot administre tareas comunes como Cancelar, Confirmar, Ayuda, Repetir y Detener. Eche un vistazo a los [dominios creados previamente](https://docs.microsoft.com/azure/cognitive-services/LUIS/luis-how-to-use-prebuilt-domains) que ofrece LUIS.

Las **entidades precompiladas** ayudan a que el bot reconozca tipos comunes de información como fechas, horas, números, temperatura, moneda, geografía y edad. Consulte [Uso de entidades creadas previamente](https://docs.microsoft.com/azure/cognitive-services/LUIS/pre-builtentities) para obtener información sobre los tipos que LUIS puede reconocer.

## <a name="how-your-bot-gets-messages-from-luis"></a>Cómo obtiene el bot los mensajes de LUIS

Una vez que haya configurado y conectado LUIS, el bot puede enviar el mensaje a la aplicación de LUIS, que devuelve una respuesta JSON que contiene las intenciones y entidades. A continuación, puede usar el [contexto de turno](~/v4sdk/bot-builder-basics.md#defining-a-turn) del _controlador de turnos_ del bot para enrutar el flujo de la conversación según la intención de la respuesta de LUIS. 

![Cómo se pasan las intenciones y entidades al bot](./media/cognitive-services-add-bot-language/cognitive-services-luis-message-flow-bot-code.png)

Para comenzar a usar una aplicación de LUIS con el bot, consulte [Uso de LUIS para reconocimiento del lenguaje](https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-luis?view=azure-bot-service-4.0).

## <a name="best-practices-for-language-understanding"></a>Procedimientos recomendados para Language Understanding

Tenga en cuenta los procedimientos siguientes al diseñar un modelo de lenguaje para el bot.

### <a name="consider-the-number-of-intents"></a>Tenga en cuenta el número de intenciones

Las aplicaciones de LUIS reconocen la intención al clasificar una expresión en varias categorías. Un resultado natural es que determinar la categoría correcta entre un gran número de intenciones puede reducir la capacidad de una aplicación de LUIS para diferenciarlas.

Una manera de reducir el número de intenciones consiste en usar un diseño jerárquico. Considere el caso de un bot de asistente personal que tiene tres intenciones relacionadas con el tiempo, tres intenciones relacionadas con la automatización del hogar y otras tres de utilidad (Ayuda, Cancelar y Saludo). Si coloca todas los intenciones en la misma aplicación de LUIS, tendrá nueve y, a medida que agregue características al bot, puede acabar con decenas de ellas. En su lugar, puede usar una aplicación de LUIS de distribuidor para determinar si la solicitud del usuario es para el tiempo, la automatización del hogar o utilidad, y después llamar a la aplicación de LUIS para la categoría que determine el distribuidor. En este caso, cada una de las aplicaciones de LUIS solo comienza con tres intenciones.

### <a name="use-a-none-intent"></a>Usar una intención None

Es habitual que los usuarios del bot digan algo inesperado o que no esté relacionado con el flujo de la conversación actual. La intención _None_ se proporciona para administrar esos mensajes.

Si no se entrena una intención para controlar los casos de reserva, predeterminados o "ninguno de los anteriores", la aplicación de LUIS solo puede clasificar los mensajes en las intenciones que haya definido. Por ejemplo, supongamos que tiene una aplicación de LUIS con dos intenciones: `HomeAutomation.TurnOn` y `HomeAutomation.TurnOff`. Si son las únicas intenciones y la entrada es algo no relacionado, como "Programar una cita el viernes", la aplicación de LUIS no tiene otra opción que clasificar ese mensaje como HomeAutomation.TurnOn o HomeAutomation.TurnOff. Si la aplicación de LUIS tiene una intención `None` con varios ejemplos, puede proporcionar alguna lógica de reserva en el bot para controlar las expresiones inesperadas.

La intención `None` es muy útil para mejorar los resultados del reconocimiento. En este escenario de automatización del hogar, "programar una cita el viernes" puede producir la intención `HomeAutomation.TurnOn` con un nivel de confianza bajo y el bot debería rechazarlo. Puede agregar estas frases al modelo en la intención `None` para que se resuelvan correctamente en `None`.

### <a name="review-the-utterances-that-luis-app-receives"></a>Revisar las expresiones que recibe la aplicación de LUIS

Las aplicaciones de LUIS proporcionan una característica para mejorar el rendimiento de la aplicación mediante la revisión de los mensajes que los usuarios le envían. Consulte [Expresiones sugeridas](https://docs.microsoft.com/azure/cognitive-services/LUIS/label-suggested-utterances) para obtener un tutorial paso a paso.


## <a name="integrate-multiple-luis-apps-and-qna-services-with-the-dispatch-tool"></a>Integración de varias aplicaciones de LUIS y servicios QnA con la herramienta de distribución

Al crear un bot multiuso que comprenda varios temas de conversación, puede empezar a desarrollar servicios independientes para cada función y después integrarlos todos. Estos servicios pueden incluir aplicaciones de Language Understanding (LUIS) y servicios de QnAMaker. Estos son algunos escenarios de ejemplo en los que un bot podría combinar varias aplicaciones de LUIS, varios servicios de QnAMaker o una combinación de ambos:

* Un bot de asistente personal permite al usuario invocar una serie de comandos. Cada categoría de comandos forma una "aptitud" que se puede desarrollar por separado y cada aptitud tiene una aplicación de LUIS.
* Un bot busca en muchas bases de conocimiento para encontrar respuestas a las más preguntas más frecuentes (P+F).
* Un bot para una empresa tiene aplicaciones de LUIS para crear cuentas de cliente y realizar pedidos, y también tiene un servicio de QnAMaker para sus Preguntas más frecuentes.  

### <a name="the-dispatch-tool"></a>La herramienta de distribución

La herramienta de distribución ayuda a integrar varias aplicaciones de LUIS y servicios de QnA Maker con el bot, mediante la creación de un *aplicación de distribución*, que es una aplicación de LUIS nueva que enruta los mensajes a los servicios de LUIS y QnAMaker apropiados. Consulte el [tutorial de distribución](./bot-builder-tutorial-dispatch.md) para obtener un tutorial paso a paso en el que se combinan varias aplicaciones de LUIS y QnA Maker en un bot.

## <a name="use-luis-to-improve-speech-recognition"></a>Uso de LUIS para mejorar el reconocimiento de voz

Para un bot con el que los usuarios van a hablar, la integración con LUIS puede ayudar al bot a identificar palabras que podrían malinterpretarse al convertir la voz en texto.  Por ejemplo, en un escenario de ajedrez, un usuario podría decir: "Move knight to A 7" (Mover Caballo a A7). Si no hay contexto para la intención del usuario, la expresión se podría reconocer como: "Move night 287" (Mover noche 287). Mediante la creación de entidades que representan coordenadas y piezas del ajedrez, y al etiquetarlas en las expresiones, se proporciona contexto para que el reconocimiento de voz las identifique. Puede [habilitar el reconocimiento de voz](https://docs.microsoft.com/azure/bot-service/bot-service-manage-speech-priming?view=azure-bot-service-4.0) con canales de Bot Framework que se integran con Bing Speech, como Web Chat, Bot Framework Emulator y Cortana.  

## <a name="additional-resources"></a>Recursos adicionales
Consulte la documentación de [Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/) para más información.

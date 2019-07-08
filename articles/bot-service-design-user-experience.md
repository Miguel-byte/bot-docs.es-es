---
title: Diseñar la experiencia del usuario | Microsoft Docs
description: Aprenda a diseñar su bot para que ofrezca una experiencia de usuario atractiva, mediante controles de usuario enriquecidos, comprensión del lenguaje natural y voz.
keywords: descripción general, diseño, experiencia de usuario, UX, control de usuario enriquecido
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/20/2018
ms.openlocfilehash: ecccbcadab93417dd52f72512a0046e70a83e85e
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405823"
---
# <a name="design-the-user-experience"></a>Diseñar la experiencia de usuario

Puede crear bots con una variedad de características como texto, botones, imágenes, tarjetas enriquecidas que se muestran en carrusel o en forma de lista, y mucho más. Sin embargo, cada canal, como Facebook, Slack o Skype, entre otros, controla en última instancia cómo representan las características los clientes de mensajería. Aunque varios canales admitan una característica, cada canal puede representar dicha característica de una manera ligeramente diferente. En los casos en que un mensaje contenga características que un canal no admita de forma nativa, el canal puede intentar representar los contenidos del mensaje como texto o como una imagen estática, lo que puede afectar significativamente la apariencia del mensaje en el cliente. En algunos casos, un canal puede no ser compatible con una característica en particular. Por ejemplo, los clientes de GroupMe no pueden mostrar un indicador de escritura.

## <a name="rich-user-controls"></a>Controles de usuario enriquecidos

Los **controles de usuario enriquecidos** son controles de interfaz de usuario comunes, como botones, imágenes, carruseles y menús que el bot presenta al usuario y con los que el usuario interactúa para comunicar su elección y la intención. Un bot puede usar una colección de controles de interfaz de usuario para imitar una aplicación, o incluso puede ejecutarse integrado dentro de una aplicación. Cuando se inserta un bot en una aplicación o sitio web, este puede representar virtualmente cualquier control de la interfaz de usuario; para ello, aprovecha las capacidades de la aplicación que lo hospeda. 

Durante décadas, los desarrolladores de aplicaciones y sitios web han confiado en los controles de la interfaz de usuario para que los usuarios pudieran interactuar con sus aplicaciones; igualmente, estos mismos controles de interfaz de usuario también pueden ser muy efectivos en los bots. Por ejemplo, los botones son una excelente manera de presentarle al usuario una opción simple. Permitir que el usuario comunique "Hoteles" haciendo clic en un botón etiquetado **Hoteles** es más fácil y rápido que forzar al usuario a escribir "Hoteles". Esto es especialmente cierto en los dispositivos móviles, donde es preferible hacer clic en vez de escribir algo.

## <a name="cards"></a>Tarjetas

Las tarjetas le permiten presentar a los usuarios una variedad de mensajes visuales, de audio o seleccionables, y le ayudan a facilitar el flujo de la conversación. Si un usuario necesita seleccionar un elemento en un conjunto fijo de elementos, puede mostrar un carrusel de tarjetas, cada una de ellas con una imagen, una descripción de texto y un solo botón de selección. Si un usuario tiene un conjunto de opciones para un solo elemento, puede presentar una imagen individual más pequeña, y una colección de botones con varias opciones a elegir. ¿Pidieron más información sobre un tema? Las tarjetas pueden proporcionar información detallada mediante la salida de audio o vídeo, o un recibo que detalle la experiencia de compra. Hay un amplio rango de usos para las tarjetas que le permitirán guiar la conversación entre el usuario y el bot. El tipo de tarjeta que use se determinará en función de las necesidades de la aplicación. Echemos un vistazo más de cerca a las tarjetas, sus acciones y algunos usos recomendados. 

Las tarjetas de Microsoft Bot Service son objetos programables que contienen colecciones estandarizadas de controles de usuario enriquecidos que se reconocen en una amplia gama de canales. En la siguiente tabla se describe la lista de tarjetas disponibles y las sugerencias de procedimientos recomendados para usar cada tipo de tarjeta.

| Tipo de tarjeta | Ejemplo | DESCRIPCIÓN |
| ---- | ---- | ---- |
| AdaptiveCard | ![Imagen de una tarjeta adaptable](./media/adaptive-card.png) | Un formato abierto de intercambio de tarjetas representado como un objeto JSON. Se utiliza normalmente para la implementación de canales de tarjetas. Las tarjetas se adaptan a la apariencia de cada canal de host. |
| AnimationCard | ![Imagen de una tarjeta de animación](./media/animation-card1.png) | Una tarjeta que puede reproducir archivos GIF animados o vídeos cortos. |
| AudioCard | ![Imagen de una tarjeta de audio](./media/audio-card.png) | Una tarjeta que puede reproducir un archivo de audio. |
| HeroCard | ![Imagen de una tarjeta de imagen prominente](./media/hero-card1.png) | Una tarjeta que normalmente contiene una sola imagen grande, uno o varios botones y texto. Se utiliza normalmente para resaltar visualmente una posible selección del usuario. |
| ThumbnailCard | ![Imagen de una tarjeta de miniatura](./media/thumbnail-card.png) | Una tarjeta que normalmente contiene una sola imagen de miniatura, uno o varios botones y texto. Se utiliza normalmente para resaltar visualmente los botones de una posible selección del usuario. |
| ReceiptCard | ![Imagen de una tarjeta de recepción](./media/receipt-card1.png) | Una tarjeta que permite que un bot proporcione un recibo al usuario. Normalmente, contiene la lista de elementos que se incluyen en el recibo, la información de impuestos y del total y otro tipo de texto. |
| SignInCard | ![Imagen de una tarjeta de inicio de sesión](./media/sign-in-card.png) | Una tarjeta que permite al bot solicitar el inicio de sesión del usuario. Normalmente contiene texto y uno o más botones en los cuales el usuario puede hacer clic para comenzar el proceso de inicio de sesión. |
| SuggestedAction | ![Imagen de una tarjeta de acciones sugeridas](./media/suggested-actions.png) | Presenta al usuario un conjunto de CardActions que representan la opción del usuario. Esta tarjeta desaparece una vez que se ha seleccionado alguna de las acciones sugeridas. |
| VideoCard | ![Imagen de una tarjeta de vídeo](./media/video-card.png) | Una tarjeta que puede reproducir vídeos. Se utiliza normalmente para abrir una dirección URL y transmitir un vídeo disponible. |
| CardCarousel | ![Imagen de una tarjeta de carrusel](./media/card-carousel.png) | Una colección de tarjetas desplazables horizontalmente que permiten al usuario ver fácilmente una serie de posibles opciones.|

Las tarjetas le permiten diseñar su bot una vez, y hacer que funcione en una variedad de canales. Sin embargo, no todos los tipos de tarjeta son totalmente compatibles en todos los canales disponibles. 

Puede encontrar instrucciones detalladas para agregar tarjetas al bot en estas secciones: [Agregar archivos adjuntos multimedia en tarjetas enriquecidas](v4sdk/bot-builder-howto-add-media-attachments.md) y [Agregar acciones sugeridas a los mensajes](v4sdk/bot-builder-howto-add-suggested-actions.md). El ejemplo de código también se puede encontrar aquí para las tarjetas: [C#](https://aka.ms/bot-cards-sample-code-cs)/[JS](https://aka.ms/bot-cards-sample-code-js), tarjetas adaptables: [C#](https://aka.ms/bot-adaptive-cards-sample-code)/[JS](https://aka.ms/bot-adaptive-cards-js-sample-code), datos adjuntos: [C#](https://aka.ms/bot-attachments-sample-code)/[JS](https://aka.ms/bot-attachments-js-sample-code), y acciones sugeridas: [C#](https://aka.ms/bot-suggested-actions-code)/[JS](https://aka.ms/bot-suggested-actions-js-code).



Cuando diseñe su bot, no descarte automáticamente los elementos comunes de la interfaz de usuario, ya que no son "lo suficientemente inteligentes". Tal como se indicó [anteriormente](~/bot-service-design-principles.md#designing-a-bot), el bot debe estar diseñado para resolver el problema del usuario de la mejor manera posible y de forma rápida y sencilla. Evite la tentación de comenzar incorporando la comprensión del lenguaje natural, ya que a menudo es innecesaria y añade una complejidad injustificada.

> [!TIP]
> Comience usando los controles mínimos de la interfaz de usuario que permiten al bot resolver el problema del usuario, y agregue otros elementos más adelante si esos controles dejan de ser suficientes.


## <a name="text-and-natural-language-understanding"></a>Comprensión del lenguaje natural y del texto

Un bot puede aceptar las entradas de **texto** de los usuarios e intentar analizar esas entradas usando la coincidencia de expresiones regulares o las API de **comprensión del lenguaje natural**, como <a href="https://www.luis.ai" target="_blank">LUIS</a>. Según el tipo de entrada que proporcione el usuario, la comprensión del lenguaje natural puede ser o no una buena solución.

En algunos casos, un usuario puede estar **respondiendo una pregunta muy específica**. Por ejemplo, si el bot pregunta "¿Cuál es su nombre?", el usuario puede responder con texto que especifique solo el nombre ("Juan") o con una frase ("Mi nombre es Juan").

Hacer preguntas específicas reduce el ámbito de las posibles respuestas que el bot podría recibir, lo que disminuye la complejidad de la lógica necesaria para analizar y comprender la respuesta. Por ejemplo, considere la siguiente pregunta amplia y abierta: "¿Cómo se siente?". Comprender las gran cantidad de permutaciones posibles para responder a ese tipo de pregunta es una tarea muy compleja.

Por el contrario, preguntas específicas como "¿Sientes dolor? Sí/No" y "¿Dónde sientes dolor? Pecho/cabeza/brazo/pierna" probablemente generarían respuestas más específicas que un bot puede analizar y comprender sin necesidad de implementar la comprensión del lenguaje natural. 

> [!TIP]
> Siempre que sea posible, haga preguntas específicas que no requieran capacidades de comprensión del lenguaje natural para analizar la respuesta. Esto simplificará el bot e incrementará el éxito a la hora de entender al usuario.

  
En otros casos, un usuario puede estar **escribiendo un comando específico**. Por ejemplo, un bot de DevOps que permite a los desarrolladores administrar máquinas virtuales puede diseñarse para aceptar comandos específicos como "/ STOP VM XYZ" o "/ START VM XYZ". Diseñar un bot para que acepte comandos específicos como este, hace que la experiencia del usuario sea buena, ya que la sintaxis es fácil de aprender y el resultado esperado de cada comando es claro. Además, el bot no requerirá capacidades de comprensión del lenguaje natural, ya que la entrada del usuario se puede analizar fácilmente utilizando expresiones regulares. 

> [!TIP]
> Diseñar un bot para que requiera comandos específicos del usuario a menudo puede proporcionar una buena experiencia de usuario y, al mismo tiempo, eliminar la necesidad de una capacidad de comprensión del lenguaje natural.

  
En el caso de que sea un bot de una *base de conocimiento*  o un bot de *preguntas y respuestas*, el usuario puede **hacer preguntas generales**. Por ejemplo, imagine un bot que pueda responder preguntas basadas en el contenido de miles de documentos. <a href="https://qnamaker.ai" target="_blank">QnA Maker</a> y <a href="https://azure.microsoft.com/services/search/" target="_blank">Azure Search</a> son tecnologías diseñadas específicamente para este tipo de escenario. Para obtener más información, consulte [Design knowledge bots](bot-service-design-pattern-knowledge-base.md) (Diseñar bots de conocimiento).

> [!TIP]
> Si está diseñando un bot que responderá preguntas basadas en datos estructurados o no estructurados de bases de datos, páginas web o documentos, considere usar tecnologías diseñadas específicamente para abordar este escenario en lugar de intentar resolver el problema con la comprensión del lenguaje natural.

  
En otros escenarios, un usuario puede **escribir solicitudes simples basadas en el lenguaje natural**. Por ejemplo, un usuario puede escribir "Quiero una pizza de pepperoni" o "¿Hay algún restaurante vegetariano a menos de 3 millas de mi casa y que esté abierto ahora?". Las API de comprensión del lenguaje natural como [LUIS.ai](https://www.luis.ai) son ideales para escenarios como este. 

Mediante las API, el bot puede extraer los componentes clave del texto del usuario para identificar la intención de este. Cuando implemente las capacidades de comprensión del lenguaje natural en el bot, establezca expectativas realistas para el nivel de detalle que es probable que los usuarios proporcionen en sus comentarios. 

![cómo hablan los usuarios](./media/bot-service-design-user-experience/buy-house.png)

> [!TIP]
> Al construir modelos de lenguaje natural, no suponga que los usuarios proporcionarán toda la información requerida en su consulta inicial. Diseñe el bot para que solicite específicamente la información que necesita, guiando al usuario para que proporcione esa información; para ello, debe hacer una serie de preguntas si fuera necesario. 

  
## <a name="speech"></a>Voz

Un bot puede usar entradas o salidas de **voz** para comunicarse con los usuarios. En los casos en que un bot está diseñado para admitir dispositivos que no tienen teclado o monitor, la característica de voz es el único medio de comunicación con el usuario. 

## <a name="choosing-between-rich-user-controls-text-and-natural-language-and-speech"></a>Elegir entre controles de usuario enriquecidos, texto y lenguaje natural y voz

Igual que las personas se comunican entre sí mediante una combinación de gestos, voz y símbolos, los bots pueden comunicarse con los usuarios mediante una combinación de controles de usuario enriquecidos, texto (a veces incluido el lenguaje natural) y voz. Estos métodos de comunicación se pueden usar juntos; no es necesario que elija uno u otro. 

Por ejemplo, imagine "bot de cocina" que ayude a los usuarios con las recetas, y donde el bot puede proporcionar instrucciones reproduciendo un vídeo o mostrando una serie de imágenes para explicar lo que se debe hacer. Algunos usuarios pueden preferir pasar las páginas de la receta o hacer preguntas al bot usando la voz mientras siguen la receta. Otros pueden preferir tocar la pantalla de un dispositivo en lugar de interactuar con el bot a través de la voz. Cuando diseñe su bot, incorpore elementos de la experiencia de usuario que respalden la forma en que los usuarios probablemente prefieran interactuar con el bot, dados los casos de uso específicos para los que es compatible. 


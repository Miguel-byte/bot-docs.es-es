---
title: Diseñar bots de conocimientos | Microsoft Docs
description: Obtenga información sobre las diferentes maneras de diseñar un bot de conocimientos que busca y devuelve información en respuesta a la entrada o la consulta del usuario.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 12/13/2017
ms.openlocfilehash: e228209b4d239a05f9c76203e9fd2fb342c14d36
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999292"
---
# <a name="design-knowledge-bots"></a>Diseñar bots de conocimientos

Puede diseñar un bot de conocimientos para proporcionar información sobre prácticamente cualquier tema. Por ejemplo, un bot de conocimientos podría responder a preguntas sobre eventos, como "¿Qué actos sobre bots se celebran en esta conferencia?", "¿Cuándo es el próximo concierto de reggae?" o "¿Quiénes son Tame Impala?". Otro podría responder a preguntas sobre TI, por ejemplo, "¿Cómo actualizo mi sistema operativo?" o "¿Adónde tengo que ir para restablecer la contraseña?". Otro incluso podría responder a preguntas sobre los contactos, como "¿Quién es Fulano de Tal?" o "¿Cuál es la dirección de correo electrónico de Mengano de Cual?". 

Independientemente del uso para el que se haya diseñado un bot de conocimientos, su objetivo fundamental es siempre el mismo: buscar y devolver la información que el usuario ha solicitado mediante el aprovechamiento de un corpus de datos, como datos relacionales de una base de datos SQL, datos JSON de un almacén no relacional o documentos PDF de un almacén de documentos. 

## <a name="search"></a>Search

La funcionalidad de búsqueda puede ser una herramienta valiosa en un bot. 

En primer lugar, la "búsqueda aproximada" permite que un bot devuelva información que probablemente sea pertinente para la pregunta del usuario, sin necesidad de que el usuario proporcione una entrada precisa. Por ejemplo, si el usuario le pide a un bot de conocimientos de música información sobre "impala" (en lugar de "Tame Impala"), el bot puede responder con información que probablemente sea pertinente para esa entrada.

![Estructura de cuadro de diálogo](~/media/bot-service-design-pattern-knowledge-base/fuzzySearch2.png)

Las puntuaciones de búsqueda indican el nivel de confianza de los resultados de una búsqueda específica, lo que permite a un bot ordenar los resultados en consecuencia, o incluso adaptar su comunicación en función del nivel de confianza. Por ejemplo, si el nivel de confianza es alto, el bot puede responder: "Este es el evento que coincide mejor con la búsqueda:".

![Estructura de cuadro de diálogo](~/media/bot-service-design-pattern-knowledge-base/searchScore2.png)

Si el nivel de confianza es bajo, el bot podría: "Mmm… ¿Buscabas alguno de estos eventos?".

![Estructura de cuadro de diálogo](~/media/bot-service-design-pattern-knowledge-base/searchScore1.png)

### <a name="using-search-to-guide-a-conversation"></a>Usar la búsqueda para guiar una conversación

Si su motivación para crear un bot es habilitar la funcionalidad básica de motor de búsqueda, puede que ni siquiera necesite un bot. ¿Qué ofrece a los usuarios una interfaz conversacional que no les puede proporcionar un motor de búsqueda típico de un explorador web? 

Los bots de conocimientos suelen ser más eficaces cuando están diseñados para guiar la conversación. Una conversación se compone de un intercambio entre el usuario y el bot, que ofrece al bot la oportunidad de formular preguntas aclaradoras, presentar opciones y validar los resultados de una manera que le es ajena a la búsqueda básica. Por ejemplo, el bot siguiente guía al usuario en una conversación que clasifica y filtra un conjunto de datos hasta que encuentra la información que busca el usuario.

![Estructura de cuadro de diálogo](~/media/bot-service-design-pattern-knowledge-base/guidedConvo1.png)

![Estructura de cuadro de diálogo](~/media/bot-service-design-pattern-knowledge-base/guidedConvo2.png)

![Estructura de cuadro de diálogo](~/media/bot-service-design-pattern-knowledge-base/guidedConvo3.png)

![Estructura de cuadro de diálogo](~/media/bot-service-design-pattern-knowledge-base/guidedConvo4.png)

Al procesar la entrada de usuario en cada paso y presentar las opciones pertinentes, el bot guía al usuario hasta la información que está buscando. Una vez que el bot ha proporcionado dicha información, puede incluso orientar al usuario sobre las formas más eficaces de encontrar información similar en el futuro. 

![Estructura del diálogo](~/media/bot-service-design-pattern-knowledge-base/Training.png)

### <a name="azure-search"></a>Azure Search

Mediante el uso de <a href="https://azure.microsoft.com/en-us/services/search/" target="_blank">Azure Search</a>, puede crear un índice de búsqueda eficaz en el que el bot puede realizar búsquedas fácilmente, así como clasificarlo y filtrarlo. Imagine un índice de búsqueda creado mediante Azure Portal.

![Estructura de cuadro de diálogo](~/media/bot-service-design-pattern-knowledge-base/search3.PNG)

Como quiere poder acceder a todas las propiedades del almacén de datos, establece cada propiedad como "recuperable". También quiere poder encontrar músicos por nombre, por lo que establece la propiedad **Name** como "buscable". Por último, quiere poder clasificar y filtrar los períodos en los que se enmarcan los músicos, de modo que marca la propiedad **Eras** como "clasificable" y "filtrable". 

Las categorías determinan los valores que existen en el almacén de datos para una propiedad determinada, junto con la magnitud de cada valor. Por ejemplo, en esta captura de pantalla se muestra que hay cinco períodos musicales distintos en el almacén de datos:

![Estructura de cuadro de diálogo](~/media/bot-service-design-pattern-knowledge-base/facet.png)

Los filtros, a su vez, seleccionan solo las instancias especificadas de una propiedad determinada. Por ejemplo, puede filtrar el conjunto de resultados anterior para que solo contenga los elementos en los que el período (**Era**) es igual a "Romántico". 

> [!NOTE]
> Vea <a href="https://github.com/ryanvolum/AzureSearchBot" target="_blank">un bot de ejemplo</a> para obtener un ejemplo completo de un bot de conocimientos creado con Azure Document DB, Azure Search y Microsoft Bot Framework.
> 
> Para simplificar, en el ejemplo anterior se muestra un índice de búsqueda creado mediante Azure Portal. 
> Los índices también pueden crearse mediante programación.

## <a name="qna-maker"></a>QnA Maker

Algunos bots de conocimientos podrían simplemente aspirar a responder preguntas más frecuentes (P+F). 
<a href="https://www.microsoft.com/cognitive-services/en-us/qnamaker" target="_blank">QnA Maker</a> es una herramienta eficaz diseñada para usarse específicamente en este caso. QnA Maker cuenta con la capacidad integrada de extraer las preguntas y las respuestas de un sitio existente de preguntas más frecuentes y, además, permite configurar manualmente una lista personalizada de preguntas y respuestas. QnA Maker dispone de capacidades de procesamiento del lenguaje natural, lo que le permite proporcionar respuestas incluso a preguntas escritas de una manera ligeramente diferente a la esperada. Aun así, no tiene la capacidad de entender el lenguaje semántico. Por ejemplo, no puede saber que un cachorro es un tipo de perro. 

Mediante la interfaz web de QnA Maker, puede configurar una base de conocimientos con tres pares de preguntas y respuestas: 

![Estructura de cuadro de diálogo](~/media/bot-service-design-pattern-knowledge-base/KnowledgeBaseConfig.png)

Después, para probarla, formule una serie de preguntas: 

![Estructura de cuadro de diálogo](~/media/bot-service-design-pattern-knowledge-base/exampleQnAConvo.png)

El bot responde correctamente a las preguntas que se asignan directamente a las que se han configurado en la base de conocimientos. Aun así, responde incorrectamente a la pregunta "Can I bring my tea?" (¿Puedo llevar mi té?), porque esta pregunta tiene una estructura más parecida a la de la pregunta "Can I bring my vodka? (¿Puedo llevar mi vodka?). La razón de que QnA Maker proporcione una respuesta incorrecta es que no entiende intrínsecamente el significado de las palabras. No sabe que el té es un tipo de bebida no alcohólica, por lo que responde que "Alcohol is not allowed" (No está permitido el alcohol).

> [!TIP]
> Cree los pares de QnA y, después, pruebe y vuelva a entrenar el bot mediante el botón "Inspeccionar" situado bajo la conversación, a fin de seleccionar una respuesta alternativa para cada respuesta incorrecta que se ha proporcionado. 

## <a name="luis"></a>LUIS

Algunos bots de conocimientos requieren capacidades de procesamiento del lenguaje natural (NLP) para poder analizar los mensajes de un usuario con el objetivo de determinar su intención. 
[Language Understanding (LUIS)](https://www.luis.ai) proporciona un medio rápido y eficaz de agregar capacidades de NLP a los bots. LUIS no solo le ofrece la posibilidad de usar modelos existentes precompilados de Bing y Cortana siempre que cumplan sus necesidades, sino que le permite crear sus propios modelos especializados. 

Al trabajar con grandes conjuntos de datos, no es necesariamente factible entrenar un modelo de NLP con cada variación de una entidad. Por ejemplo, en un bot de reproducción de música, un usuario podría enviar el mensaje "Reproducir reggae", "Reproducir Bob Marley" o "Reproducir 'One love'". Aunque un bot podría asignar cada uno de estos mensajes a la intención "playMusic" sin que se le haya entrenado con todos los nombres de artistas, géneros y nombres de canciones, un modelo de NLP no sería capaz de identificar si la entidad es un género, un artista o una canción. Al usar un modelo de NLP para identificar la entidad genérica de tipo "música", el bot podría buscar esa entidad en su almacén de datos y continuar a partir de ahí. 

## <a name="combining-search-qna-maker-andor-luis"></a>Combinar Search, QnA Maker y LUIS

Search, QnA Maker y LUIS son herramientas eficaces por sí mismas, pero también se pueden combinar para crear bots de conocimientos que posean más de una de estas capacidades.

### <a name="luis-and-search"></a>LUIS y Search

En el ejemplo de bot de festivales de música [tratado anteriormente](#search), el bot guía la conversación mostrando botones que representan las bandas del cartel. Aun así, este bot también podría incorporar comprensión del lenguaje natural si usara LUIS para determinar la intención y las entidades de preguntas como "What kind of music does Romit Girdhar play?" (¿Qué tipo de música toca Romit Girdhar?). Después, el bot podría buscar el nombre del músico en un índice de Azure Search. 
 
No sería factible entrenar el modelo con todos los nombres de músicos posibles, ya que hay muchísimos valores potenciales, pero se podrían proporcionar los ejemplos representativos suficientes para que LUIS identificase correctamente la entidad en cuestión.  Por ejemplo, imagine que entrena el modelo con ejemplos de músicos: 

![Estructura de cuadro de diálogo](~/media/bot-service-design-pattern-knowledge-base/answerGenre.png)
![Estructura de cuadro de diálogo](~/media/bot-service-design-pattern-knowledge-base/answerGenreOneWord.png)

Al probar este modelo con expresiones nuevas como "¿Qué tipo de música tocan los Beatles?", LUIS determina correctamente la intención "answerGenre" e identifica la entidad "los Beatles". A pesar de ello, si envía una pregunta más larga, como "¿Qué tipo de música tocan The Devil Makes Three?", LUIS identifica "The Devil" como la entidad.

![Estructura de cuadro de diálogo](~/media/bot-service-design-pattern-knowledge-base/devilMakesThreeScore.png)

Si entrena el modelo con entidades de ejemplo representativas del conjunto de datos subyacente, puede aumentar la precisión de la comprensión del lenguaje del bot. 

> [!TIP]
> En general, es mejor que el modelo se equivoque por identificar demasiadas palabras durante el reconocimiento de entidades (por ejemplo, identificar "John Smith, por favor" en la expresión "Llamar a John Smith, por favor"), en vez de por identificar demasiadas pocas palabras (por ejemplo, identificar "John" en la expresión "Llamar a John Smith, por favor"). El índice de búsqueda pasará por alto las palabras irrelevantes, como "por favor", en la frase "John Smith, por favor". 

### <a name="luis-and-qna-maker"></a>LUIS y QnA Maker

Algunos bots de conocimientos podría usar QnA Maker para responder a preguntas básicas en combinación con LUIS para determinar las intenciones, extraer entidades e invocar cuadros de diálogo más elaborados. Por ejemplo, imagine un bot simple del Servicio de asistencia del departamento de TI. Este bot podría usar QnA Maker para responder a preguntas básicas sobre Windows o Outlook, pero también podría tener que facilitar escenarios como el restablecimiento de contraseñas, que requiere el reconocimiento de intenciones y la comunicación entre el usuario y bot. Un bot puede implementar un híbrido de LUIS y QnA Maker de varias maneras:

1. Puede llamar a QnA Maker y LUIS al mismo tiempo y responder al usuario con la información del primero que devuelva una puntuación con un umbral específico. 
2. Puede llamar primero a LUIS y, si ninguna intención cumple una puntuación de umbral específica, se desencadena la intención "None" y llama a QnA Maker. También puede crear una intención de LUIS para QnA Maker y proporcionarle al modelo de LUIS las preguntas de QnA de ejemplo asignadas a "QnAIntent." 
3. Puede llamar primero a QnA Maker y, si ninguna respuesta cumple una puntuación de umbral específica, llamar a LUIS. 

Bot Builder SDK ofrece compatibilidad integrada con LUIS y QnA Maker. Esto permite desencadenar cuadros de diálogo o responder a preguntas automáticamente con LUIS o QnA Maker sin necesidad de implementar llamadas personalizadas a ninguna de estas herramientas. Para más información, consulte el [Tutorial de la herramienta de distribución de Bot Builder](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0).

> [!TIP]
> Al implementar una combinación de LUIS, QnA Maker y Azure Search, pruebe las entradas con cada una de las herramientas para determinar la puntuación de umbral de cada uno de los modelos. LUIS, QnA Maker y Azure Search generan puntuaciones mediante criterios de puntuación diferentes, por lo que las puntuaciones generadas en estas herramientas no son comparables directamente. Además, LUIS y QnA Maker normalizan las puntuaciones. Una puntuación determinada podría considerarse "buena" en un modelo de LUIS, pero no en otro modelo. 

## <a name="sample-code"></a>Código de ejemplo

- Para consultar un ejemplo en el que se muestra cómo crear un bot de conocimientos básico mediante Bot Builder SDK para. NET, vea el <a href="https://aka.ms/qna-with-appinsights" target="_blank">ejemplo de bot de conocimientos</a> en GitHub. 
<!-- TODO: Do not have a current bot sample to work with this
- For a sample that shows how to create more complex knowledge bots using the Bot Builder SDK for .NET, see the <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search" target="_blank">Search-powered Bots sample</a> in GitHub.
-->

[qnamakerTemplate]: https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle

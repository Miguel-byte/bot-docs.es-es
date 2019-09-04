---
title: Cognitive Services | Microsoft Docs
description: Aprenda a agregar inteligencia artificial a los bots con Microsoft Cognitive Services para que sean más útiles y atractivos.
keywords: reconocimiento del lenguaje, extracción de conocimientos, reconocimiento de voz, búsqueda web
author: RobStand
ms.author: rstand
manager: rstand
ms.topic: article
ms.service: bot-service
ms.date: 12/17/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 7dbbe8492761360941eca5e724a7339e28968769
ms.sourcegitcommit: eacf1522d648338eebefe2cc5686c1f7866ec6a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2019
ms.locfileid: "70167008"
---
# <a name="cognitive-services"></a>Cognitive Services

Microsoft Cognitive Services le permite sacar partido a una creciente colección de potentes algoritmos de inteligencia artificial desarrollados por expertos en los campos de visión artificial, voz, procesamiento de lenguaje natural, extracción de conocimientos y búsqueda web. Estos servicios facilitan varias tareas basadas en inteligencia artificial, procurándole una forma rápida de incorporar las tecnologías inteligentes de última generación a sus bots con solo unas cuantas líneas de código. Las API se integran en las plataformas y lenguajes más modernos. Las API están en un proceso de mejora, aprendizaje e inteligencia continuo, por lo que las experiencias están siempre actualizadas. 

Los bots inteligentes responden como si pudieran ver la realidad como las personas. Descubren información y extraen conocimientos de diferentes orígenes para proporcionar respuestas útiles y, lo mejor de todo, pueden aprender a medida que adquieren más experiencia y mejorar continuamente sus capacidades. 

## <a name="language-understanding"></a>Comprensión del lenguaje

La interacción entre los usuarios y los bots suele ser de forma libre, por lo que los bots deben comprender el lenguaje de forma natural y contextual. Las API de lenguaje de Cognitive Services proporcionan eficaces modelos de lenguaje para determinar lo que los usuarios quieren, identificar conceptos y entidades en una determinada frase y, en última instancia, permitir que los bots respondan con la acción apropiada. Las cinco API admiten varias funcionalidades de análisis de texto como la corrección ortográfica, la detección de opiniones, el modelado de lenguaje y la extracción de información detallada y precisa del texto. 

Cognitive Services proporciona estas API de reconocimiento del lenguaje:

- <a href="https://www.microsoft.com/cognitive-services/language-understanding-intelligent-service-luis" target="_blank">Language Understanding Intelligent Service (LUIS)</a> es capaz de procesar lenguaje natural mediante modelos de lenguaje predefinidos o con entrenamiento personalizado. Puede encontrar más información en [Language Understanding para bots](v4sdk/bot-builder-concept-luis.md)

- <a href="https://www.microsoft.com/cognitive-services/text-analytics-api" target="_blank">Text Analytics API</a> detecta opiniones, frases clave, temas e idioma a partir del texto.

- <a href="https://www.microsoft.com/cognitive-services/bing-spell-check-api" target="_blank">Bing Spell Check API</a> ofrece eficaces funcionalidades de corrección ortográfica y es capaz de reconocer la diferencia entre nombres, nombres de marcas y jerga.

- Los <a href="https://docs.microsoft.com/azure/machine-learning/studio/text-analytics-module-tutorial" target ="_blank">modelos de Text Analytics en Azure Machine Learning Studio</a> le permiten crear y usar modelos de análisis de texto como, por ejemplo, la lematización y el preprocesamiento de texto. Estos modelos pueden ayudarle a resolver problemas de clasificación de documentos o de análisis de opinión.

Obtenga más información sobre [Language Understanding][language] con Microsoft Cognitive Services.

## <a name="knowledge-extraction"></a>Extracción de conocimientos

Cognitive Services proporciona cuatro API de conocimientos que le permiten identificar entidades con nombre o frases en texto no estructurado, agregar recomendaciones personalizadas, proporcionar sugerencias de Autocompletar basadas en la interpretación natural de las consultas del usuario y búsqueda en documentos y otras investigaciones académicas como, por ejemplo, un servicio personalizado de preguntas frecuentes.

- <a href="https://www.microsoft.com/cognitive-services/entity-linking-intelligence-service" target="_blank">Entity Linking Intelligence Service</a> anota el texto no estructurado con las entidades adecuadas que se mencionan en el texto. Según el contexto, la misma palabra o expresión puede referirse a cosas diferentes. Este servicio reconoce el contexto del texto suministrado e identificará cada entidad del texto.    

- <a href="https://www.microsoft.com/cognitive-services/knowledge-exploration-service" target="_blank">Knowledge Exploration Service</a> ofrece una interpretación del lenguaje natural de las consultas del usuario y devuelve interpretaciones anotadas que permiten unas experiencias de búsqueda y finalización automática enriquecidas que se anticipan a lo que va a escribir el usuario. Las sugerencias instantáneas de finalización de consultas y el refinamiento predictivo de estas se basan en sus propios datos y en gramáticas específicas de la aplicación que permiten a los usuarios realizar consultas rápidas.    

- <a href="https://www.microsoft.com/cognitive-services/academic-knowledge-api" target="_blank">Academic Knowledge API</a> devuelve trabajos de investigación académicos, autores, revistas especializadas, conferencias, temas y universidades sacados de <a href="https://www.microsoft.com/research/project/microsoft-academic-graph/" target="_blank">Microsoft Academic Graph</a>. Compilada como un ejemplo específico del dominio de Knowledge Exploration Service, Academic Knowledge API proporciona una base de conocimientos que usa un diálogo de tipo gráfico con funcionalidades de búsqueda en cientos de millones de entidades relacionadas con la investigación. Busque un tema, un profesor, una universidad o una conferencia y la API le proporcionará las publicaciones adecuadas y las entidades relacionadas. La gramática también admite consultas naturales como "Artículos de Michael Jordan sobre aprendizaje automático posteriores a 2010".

- <a href="https://qnamaker.ai" target="_blank">QnA Maker</a> es una API de REST fácil de usar y un servicio basado en web que permite entrenar a la inteligencia artificial para responder a preguntas del usuario de una manera natural y conversacional. Gracias a una lógica de aprendizaje automático optimizada y a la capacidad para integrar un procesamiento del lenguaje líder del sector, QnA Maker sintetiza datos semiestructurados como, por ejemplo, pares de pregunta y respuesta, en respuestas útiles diferenciadas.

Más información sobre la [extracción de conocimientos][knowledge] con Microsoft Cognitive Services.

## <a name="speech-recognition-and-conversion"></a>Reconocimiento y conversión de voz

Use Speech API para agregar habilidades avanzadas de voz al bot que aprovechen los algoritmos líderes del sector para la conversión de voz a texto y de texto a voz, así como el reconocimiento del hablante. Speech API usa modelos acústicos y de lenguaje integrados que abarcan una amplia gama de escenarios con alta precisión. 

Para las aplicaciones que requieran una mayor personalización, puede usar Custom Recognition Intelligent Service (CRIS). Este le permite calibrar los modelos acústicos y de lenguaje del reconocedor de voz para adaptarlo al vocabulario de la aplicación e incluso al estilo de habla de los usuarios.

Hay tres tipos de Speech API disponibles en Cognitive Services para procesar o sintetizar voz:

- <a href="https://www.microsoft.com/cognitive-services/speech-api" target="_blank">Bing Speech API</a> proporciona funciones de conversión de voz a texto y de texto a voz.
- <a href="https://www.microsoft.com/cognitive-services/custom-recognition-intelligent-service-cris" target="_blank">Custom Recognition Intelligent Service (CRIS)</a> le permite crear modelos de reconocimiento de voz personalizados para adaptar la conversión de voz a texto a un vocabulario de una aplicación o al estilo de habla del usuario.
- <a href="https://www.microsoft.com/cognitive-services/speaker-recognition-api" target="_blank">Speaker Recognition API</a> permite la identificación del hablante y la comprobación a través de voz.

Los siguientes recursos proporcionan información adicional acerca de cómo agregar el reconocimiento de voz al bot.

* [Vídeo introductorio sobre conversaciones de bots para aplicaciones](https://channel9.msdn.com/events/Build/2017/P4114)
* [Biblioteca de Microsoft.Bot.Client para aplicaciones UWP o Xamarin](https://aka.ms/bot-client)
* [Ejemplo de biblioteca cliente de bots](https://aka.ms/trivia-bot-speech-sample)
* [Cliente de chat en web compatible con voz](https://aka.ms/BotFramework-WebChat)

Más información sobre [reconocimiento de voz y conversión][speech] con Microsoft Cognitive Services.

## <a name="web-search"></a>Búsqueda web

Bing Search API le permite agregar funciones de búsqueda web inteligentes a los bots. Con tan solo unas pocas líneas de código, puede acceder a miles de millones de páginas web, imágenes, vídeos, noticias y otros tipos de resultados. Puede configurar las API para que devuelvan los resultados por ubicación geográfica, mercado o idioma para que resulten más adecuados. Puede personalizar aún más la búsqueda mediante los parámetros de búsqueda admitidos como SafeSearch para filtrar contenido para adultos y Freshness para devolver resultados en función de una fecha específica.

Hay cinco tipos de Bing Search API en Cognitive Services.

- <a href="https://www.microsoft.com/cognitive-services/bing-web-search-api" target="_blank">Web Search API</a> proporciona resultados de búsqueda de webs, imágenes, vídeos, noticias y otros contenidos relacionados mediante una única llamada API.

- <a href="https://www.microsoft.com/cognitive-services/bing-image-search-api" target="_blank">Image Search API</a> devuelve resultados de imágenes con metadatos mejorados (color dominante, tipo de imagen, etc) y admite varios filtros de imágenes para personalizar los resultados.

- <a href="https://www.microsoft.com/cognitive-services/bing-video-search-api" target="_blank">Video Search API</a> recupera resultados de vídeo con metadatos enriquecidos (tamaño del vídeo, calidad, precio, etc), versiones preliminares del vídeo, y admite varios filtros de vídeo para personalizar los resultados.

- <a href="https://www.microsoft.com/cognitive-services/bing-news-search-api" target="_blank">News Search API</a> busca artículos de noticias de todo el mundo que coincidan con su consulta de búsqueda o que sean actualmente tendencia en Internet.

- <a href="https://www.microsoft.com/cognitive-services/bing-autosuggest-api" target="_blank">Autosuggest API</a> ofrece sugerencias automáticas instantáneas de finalización de consultas para completar la búsqueda más rápidamente y con un menor uso del teclado. 

Obtenga más información sobre la [búsqueda web][search] con Microsoft Cognitive Services.

## <a name="image-and-video-understanding"></a>Reconocimiento de imágenes y vídeo

Vision API proporciona habilidades de reconocimiento de imágenes y vídeo avanzadas a los bots. Algoritmos de última generación le permiten procesar las imágenes o los vídeos y recibir información que puede transformar en acciones. Por ejemplo, puede usarlas para reconocer objetos, caras de personas, edad, sexo o incluso sentimientos. 

Vision API admite una gran variedad de características de reconocimiento de imágenes. Pueden identificar contenido para adultos o explícito, detectar colores de énfasis, clasificar el contenido de imágenes, realizar un reconocimiento óptico de caracteres y describir una imagen con frases completas en inglés. Vision API también admite varias funcionalidades de procesamiento de imágenes y de vídeo como, por ejemplo, la generación inteligente de miniaturas de imágenes o vídeos o la estabilización de la salida de un vídeo.

Cognitive Services proporciona cuatro tipos de API que puede usar para procesar imágenes o vídeos:

- <a href="https://www.microsoft.com/cognitive-services/computer-vision-api" target="_blank">Computer Vision API</a> extrae información valiosa sobre las imágenes (como objetos o personas), determina si la imagen contiene contenido para adultos o explícito y procesa texto (mediante OCR) de las imágenes.

- <a href="https://www.microsoft.com/cognitive-services/emotion-api" target="_blank">Emotion API</a> analiza caras humanas y reconoce sus emociones y las clasifica en ocho categorías de emociones humanas posibles.

- <a href="https://www.microsoft.com/cognitive-services/face-api" target="_blank">Face API</a> detecta rostros humanos, los compara con caras similares e incluso puede organizar personas en grupos por parecidos visuales.

- <a href="https://www.microsoft.com/cognitive-services/video-api" target="_blank">Video API</a> analiza y procesa vídeo para estabilizar salida de vídeo, detecta movimiento, rastrea caras y puede generar un resumen de miniaturas de movimiento del vídeo.

Obtenga más información sobre el [reconocimiento de imágenes y de vídeo][vision] con Microsoft Cognitive Services.

## <a name="additional-resources"></a>Recursos adicionales

Puede encontrar una completa documentación de cada producto y las correspondientes referencias de API en la <a href="https://docs.microsoft.com/azure/cognitive-services" target="_blank">documentación sobre Cognitive Services</a>.

[language]: https://docs.microsoft.com/azure/cognitive-services/luis/home
[search]: https://docs.microsoft.com/azure/cognitive-services/bing-web-search/search-the-web
[vision]: https://docs.microsoft.com/azure/cognitive-services/computer-vision/home
[knowledge]: https://docs.microsoft.com/azure/cognitive-services/kes/overview
[speech]: https://docs.microsoft.com/azure/cognitive-services/speech/home
[location]: https://docs.microsoft.com/azure/cognitive-services/

---
title: Introducción a Custom Assistant | Microsoft Docs
description: Aprenda a crear su propio asistente personalizado
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 13/12/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: dcef01d8ba07d44aebaeeeecb5637af691caccb9
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997082"
---
## <a name="custom-assistant-overview"></a>Introducción a Custom Assistant

## <a name="overview"></a>Información general

Hemos observado que nuestros clientes y asociados tienen una gran necesidad de ofrecer un asistente de conversación adaptado a su marca, personalizado para sus clientes y disponible en una amplia variedad de dispositivos y lienzos de conversación. Siguiendo el enfoque de código abierto que Microsoft ha aplicado al SDK de Bot Framework, la solución Custom Personal Assistant de código abierto proporciona un control total sobre la experiencia del usuario final basándose en un conjunto de funcionalidades fundamentales. Además, la experiencia puede dotarse con inteligencia sobre la información de cualquier ecosistema o dispositivo y usuario final para una experiencia inteligente y realmente integrada.

Creemos firmemente que nuestros clientes deberían tener y enriquecer sus propios datos y relaciones con los clientes. Por eso, cualquier asistente personalizado proporciona un control total sobre la experiencia del usuario para nuestros clientes y asociados. El nombre, la voz y la personalidad se pueden cambiar para satisfacer las necesidades de la organización. Nuestra solución Custom Assistant simplifica la creación de su propio asistente y le permite empezar en cuestión de minutos. 

El ámbito de la funcionalidad Custom Personal Assistant es amplio y normalmente ofrece a los usuarios finales una amplia gama de funcionalidades. Para aumentar la productividad de los desarrolladores y habilitar un ecosistema dinámico de experiencias de conversación reutilizables, les proporcionamos ejemplos iniciales de aptitudes de conversación reutilizables. Estas aptitudes se pueden agregar a la aplicación de conversación para aclarar una experiencia de conversación específica, como la búsqueda de un punto de interés, la interacción con el calendario, las tareas, el correo electrónico y muchos otros escenarios. Las aptitudes son totalmente personalizables y consisten en modelos para varios idiomas, diálogos y código.

En este momento estamos ejecutando una versión preliminar inicial y colaboramos estrechamente con los asociados y clientes iniciales en un repositorio de código abierto para lanzar las primeras experiencias y ponerla a disposición de los usuarios de forma más amplia en los próximos meses. 

![Diagrama de Custom Assistant](media/enterprise-template/CustomAssistantDiagram.jpg)

## <a name="complete-control-of-the-user-experience"></a>Control total de la experiencia de usuario

Podrá controlar todos los aspectos de la experiencia del usuario final, que le pertenece. Esto incluye la personalización de marca, el nombre, la voz, la personalidad, las respuestas y el avatar. El código fuente de Custom Assistant y de las aptitudes auxiliares está disponible para que lo ajuste según sea necesario.

## <a name="complete-ownership-and-control-of-data"></a>Propiedad total y control de los datos

Custom Assistant se implementará en la suscripción de Azure. Por lo tanto, todos los datos generados por el asistente (preguntas, comportamiento de los usuarios, etc.) están incluidos por completo en la suscripción de Azure. Consulte [Cognitive Services Azure Trusted Cloud](https://www.microsoft.com/en-us/trustcenter/cloudservices/cognitiveservices) y la [sección de Azure del Centro de confianza](https://www.microsoft.com/en-us/TrustCenter/CloudServices/Azure) en concreto para más información.

## <a name="your-assistant-anywhere"></a>El asistente, donde sea...

Custom Assistant aprovecha la plataforma de inteligencia artificial conversacional de Microsoft y, por lo tanto, se puede utilizar con cualquier [canal](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0) de Bot Framework, por ejemplo, WebChat, FaceBook Messenger, Skype, etc. Además, mediante la [línea directa](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-concepts?view=azure-bot-service-4.0) se pueden insertar experiencias en aplicaciones de escritorio y móviles, incluidos dispositivos como los automóviles, los altavoces, los relojes con alarma, etc.

## <a name="built-on-enterprise-grade-technology"></a>Compilación con tecnología empresarial

La solución Custom Assistant se basa en Azure Bot Service, Language Understanding Cognitive Service y Unified Speech, y cuenta con un amplio conjunto de componentes de Azure auxiliares, para que pueda disfrutar de todas las ventajas de la [infraestructura global de Azure](https://azure.microsoft.com/en-gb/global-infrastructure/).

Además, LUIS Cognitive Service ofrece compatibilidad con Language Understanding para admitir un amplio conjunto de idiomas, que se [enumeran aquí](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-supported-languages). Para ampliar el alcance de Custom Assistant, [Translator Cognitive Service](https://azure.microsoft.com/en-us/services/cognitive-services/translator-text-api/) ofrece funcionalidades de traducción automática adicionales.

## <a name="integrated-and-context-aware"></a>Integración y reconocimiento contextual

Custom Assistant se puede integrar en su dispositivo y ecosistema para disfrutar de una experiencia realmente integrada e inteligente. Con este reconocimiento contextual se pueden desarrollar experiencias más inteligentes y se consigue mayor personalización que de cualquier otra forma.

## <a name="3rd-party-assistant-integration"></a>Integración de asistentes de terceros

Custom Assistant permite ofrecer una experiencia propia única, pero también el asistente digital preferido para determinados tipos de preguntas de los usuarios finales.

## <a name="flexible-integration"></a>Integración flexible

La arquitectura de Custom Assistant es flexible y se integra con las inversiones existentes que haya realizado en las funcionalidades de voz basadas en dispositivos o de procesamiento de idioma natural y, por supuesto, también con los sistemas de back-end y las API existentes.

## <a name="adaptive-cards"></a>Tarjetas adaptables

Las [tarjetas adaptables](https://adaptivecards.io/) permiten a Custom Assistant devolver elementos de la experiencia del usuario (como tarjetas, imágenes, botones) junto a las respuestas de texto base. Si el lienzo de conversación o el dispositivo tienen pantalla, las tarjetas adaptables se pueden representar en una amplia gama de dispositivos y plataformas y proporcionar compatibilidad con la experiencia del usuario cuando proceda. [Aquí](https://adaptivecards.io/samples/) encontrará ejemplos de tarjetas adaptables e información sobre las opciones de representación en [esta](https://docs.microsoft.com/en-us/adaptive-cards/rendering-cards/getting-started) documentación.


## <a name="skills"></a>Aptitudes

Además del asistente base, existe un amplio conjunto de funcionalidades comunes que requieren la compilación del desarrollador. La productividad es un buen ejemplo donde cada organización necesitaría crear modelos de idioma (LUIS), diálogos (código), integración (código) y generación de idioma (respuestas) para habilitar escenarios comunes, como puntos de interés, correo electrónico, calendarios o tareas.

Esto se complica por la necesidad de compatibilidad con varios y idiomas, y deriva en una gran cantidad de trabajo para cualquier organización que compile su propio asistente.

Nuestra solución Custom Assistant incluye una nueva funcionalidad de aptitud que aporta nuevas funcionalidades que conectar a un asistente personalizado mediante configuración. 

Los desarrolladores pueden personalizar todos los aspectos de cada aptitud (modelo de idioma, diálogos, código de integración y generación de idioma), ya que todo el código fuente se encuentra en GitHub con Custom Assistant.

## <a name="example-scenarios"></a>Escenarios de ejemplo

Custom Assistant abarca una amplia gama de escenarios de la industria. A continuación se muestran algunos de ejemplo como referencia:

- Sector de automoción: asistente personal con voz integrado en el automóvil para que el usuario final pueda utilizar funcionalidades tradicionales (como la navegación o la radio) y escenarios centrados en la productividad, como posponer reuniones al llegar tarde, incorporar elementos a la lista de tareas y experiencias proactivas en las que el automóvil sugiere completar tareas en función de eventos como el arranque del motor, ir a casa o activar el control de crucero. Las tarjetas adaptables se representan en la unidad principal y la integración de voz se realiza con interacciones de tipo presionar para hablar o con palabras de activación.

- Hostelería: asistente personal con voz integrado en un dispositivo en la habitación del hotel que ofrece una amplia gama de funcionalidades para escenarios centrados en la hostelería (como la ampliación de la estancia, la solicitud de retrasar el registro de salida, el servicio de habitaciones) incluido el conserje y la posibilidad de buscar restaurantes y atracciones locales. La vinculación opcional a las cuentas de productividad abren la puerta a experiencias más personalizadas, como sugerencias de llamadas de alarma, advertencias sobre el tiempo y el aprendizaje de patrones con las estancias. Una evolución de la personalización de la televisión actual de las habitaciones de hoy en día.

- Empresa: experiencias de asistentes de voz y texto con marca para empleados, integradas en dispositivos empresariales y lienzos conversacionales existentes (como Teams, WebChat, Slack) para que los empleados administren sus calendarios, encuentren las salas de reuniones disponibles, busquen personas con aptitudes específicas o realicen operaciones de recursos humanos. 

## <a name="getting-started"></a>Introducción

En este momento estamos ejecutando una versión preliminar inicial y colaboramos estrechamente con los asociados y clientes iniciales en un repositorio de código abierto para lanzar las primeras experiencias y ponerla a disposición de los usuarios de forma más amplia en los próximos meses. Para registrar su interés y empezar a usarlo, rellene [este formulario](https://aka.ms/customassistantpreviewform) y nos pondremos en contacto con usted.


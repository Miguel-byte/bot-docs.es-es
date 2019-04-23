---
title: Introducción a Virtual Assistant | Microsoft Docs
description: Aprenda a crear su propio asistente virtual.
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 13/12/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1d231672284e20e3f215cbba16d881ae0ecab7ec
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/12/2019
ms.locfileid: "59540991"
---
# <a name="virtual-assistant-solution-overview"></a>Introducción a la solución Virtual Assistant

## <a name="overview"></a>Información general
Hemos observado que nuestros clientes y asociados tienen una gran necesidad de ofrecer un asistente de conversación adaptado a su marca, personalizado para sus clientes y disponible en una amplia variedad de dispositivos y lienzos de conversación. Siguiendo el enfoque de código abierto que Microsoft ha aplicado al SDK de Bot Framework, la solución Custom Personal Assistant de código abierto proporciona un control total sobre la experiencia del usuario final basándose en un conjunto de funcionalidades fundamentales. Además, la experiencia puede dotarse con inteligencia sobre la información de cualquier ecosistema o dispositivo y usuario final para una experiencia inteligente y realmente integrada.

Creemos firmemente que nuestros clientes deberían tener y enriquecer sus propios datos y relaciones con los clientes. Por eso, cualquier asistente virtual proporciona un control total sobre la experiencia del usuario para nuestros clientes y asociados mediante el código abierto de GitHub. El nombre, la voz y la personalidad se pueden cambiar para satisfacer las necesidades de la organización. Nuestra solución Virtual Assistant simplifica la creación de su propio asistente, y le permite empezar en cuestión de minutos para, posteriormente, ampliar conocimientos con nuestras herramientas de desarrollo globales.

Normalmente, Virtual Assistant ofrece a los usuarios finales una amplia gama de funcionalidades. Para aumentar la productividad de los desarrolladores y habilitar un ecosistema dinámico de experiencias de conversación reutilizables, les proporcionamos ejemplos iniciales de aptitudes de conversación reutilizables. Estas aptitudes se pueden agregar a una aplicación de conversación para aclarar una experiencia de conversación específica, como la búsqueda de un punto de interés, la interacción con el calendario, las tareas, el correo electrónico y muchos otros escenarios. Las aptitudes son totalmente personalizables y consisten en modelos de lenguaje para varios idiomas, diálogos y código.

En este momento estamos trabajando estrechamente con los clientes y asociados iniciales en un repositorio de código abierto para dar vida a las primeras experiencias.

![Diagrama de Virtual Assistant](./media/enterprise-template/customassistantdiagram.jpg)

## <a name="whats-in-the-box"></a>Contenido 

Virtual Assistant se basa en las funcionalidades de la [plantilla de empresa](./bot-builder-enterprise-template-overview.md), que proporciona funcionalidades fundamentales para experiencias de conversación que incluyen intenciones conversacionales en varios idiomas, distribución, preguntas y respuestas, y conclusiones conversacionales. Las siguientes funcionalidades relacionadas con el asistente se proporcionan en este momento. Hay otras funcionalidades planeadas y colaboraremos estrechamente con clientes y asociados para informar sobre la hoja de ruta.

Característica | DESCRIPCIÓN |
------------ | -------------
Incorporación | Un ejemplo de flujo de incorporación permite al asistente saludar al usuario y recopilar la información inicial.
Arquitectura de eventos | Los eventos en el contexto de Virtual Assistant permiten a la aplicación cliente que hospeda al asistente (en un explorador web o en un dispositivo como un coche o un altavoz) intercambiar información sobre el usuario o sobre los eventos del dispositivo mientras se siguen recibiendo eventos para realizar otras operaciones del dispositivo.
Cuentas vinculadas | En un escenario guiado por voz, no resulta práctico para un usuario escribir su nombre de usuario y contraseña para sistemas auxiliares mediante comandos de voz. Por tanto, una experiencia complementaria independiente proporciona una oportunidad para que el usuario inicie sesión y proporcione permisos para que un asistente virtual recupere tokens para su uso posterior.
Habilitación de aptitudes | Hoy en día, existe un amplio conjunto de funcionalidades comunes que requieren que los desarrolladores las compilen por sí mismos. Nuestra solución Virtual Assistant incluye una nueva funcionalidad de aptitud que permite la conexión de nuevas funcionalidades en un asistente virtual solo mediante configuración y proporciona un mecanismo de autenticación para que las aptitudes soliciten tokens para las actividades de niveles inferiores.
Aptitud de punto de interés | La versión preliminar de la aptitud de punto de interés (PoI) proporciona un modelo de lenguaje completo para buscar puntos de interés y solicitar instrucciones. Actualmente, esta aptitud proporciona integración en Azure Maps.
Aptitud de calendario | La versión preliminar de la aptitud de calendario proporciona un modelo de lenguaje completo para las actividades habituales relacionadas con el calendario. Esta aptitud está integrada actualmente en Microsoft Graph (Office 365/Outlook.com) con compatibilidad para Google API próximamente.
Aptitud de correo electrónico | La versión preliminar de la aptitud de correo electrónico proporciona un modelo de lenguaje completo para las actividades habituales relacionadas con el correo electrónico. Esta aptitud está integrada actualmente en Microsoft Graph (Office 365/Outlook.com) con compatibilidad para Google API próximamente.
Aptitud de tareas pendientes | La versión preliminar de la aptitud de tareas pendientes proporciona un modelo de lenguaje completo para las actividades habituales relacionadas. La aptitud está integrada actualmente en OneNote con compatibilidad para Microsoft Graph (OutlookTask) próximamente.
Integración de dispositivos | Nuestros SDK de Azure Bot Service (DirectLine) junto con las tarjetas adaptables y los SDK de Voz permiten una fácil integración entre plataformas para los dispositivos. Están previstos ejemplos de integración de dispositivos y plataformas, entre las que se incluye, Edge.
Herramientas de ejecución de pruebas | Además del emulador de Bot Framework, se proporciona una herramienta de ejecución de pruebas basada en WebChat que permite la comprobación de escenarios de autenticación más complejos. Una sencilla herramienta de ejecución de pruebas basada en una consola muestra el enfoque para intercambiar mensajes para ayudar a la plataforma a facilitar la integración del dispositivo.
Implementación automatizada | Todos los recursos de Azure necesarios para el asistente se implementan automáticamente: registro de bots, Azure App Service, LUIS, QnA Maker, Content Moderator, CosmosDB, Azure Storage y Application Insights. Además, se crean modelos de LUIS para todas las aptitudes, y modelos de QnAMaker y de Dispatch, y se entrenan y publican para permitir la realización inmediata de pruebas.
Modelo de lenguaje de automoción | Un modelo de lenguaje de automoción que abarca dominios fundamentales como telefonía, navegación y control de características del vehículo estará disponible próximamente.

## <a name="example-scenarios"></a>Escenarios de ejemplo

Virtual Assistant abarca una amplia gama de escenarios de la industria. A continuación se muestran algunos ejemplos como referencia.

- Sector de automoción: asistente personal con voz integrado en el automóvil para que el usuario final pueda utilizar funcionalidades tradicionales (como la navegación o la radio) y escenarios centrados en la productividad, como posponer reuniones al llegar tarde, incorporar elementos a la lista de tareas y experiencias proactivas en las que el automóvil sugiere completar tareas en función de eventos como el arranque del motor, ir a casa o activar el control de crucero. Las tarjetas adaptables se representan en la unidad principal y la integración de voz se realiza con interacciones de tipo presionar para hablar o con palabras de activación.

- Hostelería: asistente personal con voz integrado en un dispositivo de habitación de hotel que ofrece una amplia gama de funcionalidades para escenarios centrados en la hostelería (como la ampliación de la estancia, la solicitud de retrasar el registro de salida, el servicio de habitaciones) incluido el conserje y la posibilidad de buscar restaurantes y atracciones locales. La vinculación opcional a las cuentas de productividad abren la puerta a experiencias más personalizadas, como sugerencias de llamadas de alarma, advertencias sobre el tiempo y el aprendizaje de patrones con las estancias. Una evolución de la personalización de la televisión actual de las habitaciones de hoy en día.

- Empresa: experiencias de asistentes de voz y texto con marca para empleados, integradas en dispositivos empresariales y lienzos conversacionales existentes (como Teams, WebChat, Slack) para que los empleados administren sus calendarios, encuentren las salas de reuniones disponibles, busquen personas con aptitudes específicas o realicen operaciones de recursos humanos.

## <a name="virtual-assistant-principles"></a>Principios de Virtual Assistant

### <a name="your-data-your-brand-and-your-experience"></a>Sus datos, su marca y su experiencia
Podrá controlar todos los aspectos de la experiencia del usuario final, que le pertenece. Esto incluye la personalización de marca, el nombre, la voz, la personalidad, las respuestas y el avatar. El código fuente de Virtual Assistant y de las aptitudes auxiliares está disponible para que lo ajuste según sea necesario.

Virtual Assistant se implementará en la suscripción de Azure. Por lo tanto, todos los datos generados por el asistente (preguntas, comportamiento de los usuarios, etc.) están incluidos por completo en la suscripción de Azure. Consulte [Cognitive Services Azure Trusted Cloud](https://www.microsoft.com/en-us/trustcenter/cloudservices/cognitiveservices) y la [sección de Azure del Centro de confianza](https://www.microsoft.com/en-us/TrustCenter/CloudServices/Azure) en concreto para más información.

### <a name="write-it-once-embed-it-anywhere"></a>Escríbalo solo una vez, insértelo en cualquier lugar
Virtual Assistant aprovecha la plataforma de inteligencia artificial conversacional de Microsoft y, por lo tanto, se puede utilizar con cualquier [canal](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0) de Bot Framework, por ejemplo, WebChat, FaceBook Messenger, Skype,etc. 

Además, mediante el canal [DirectLine](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-concepts?view=azure-bot-service-4.0) se pueden insertar experiencias en aplicaciones de escritorio y móviles, incluidos dispositivos como automóviles, altavoces, relojes con alarma, etc.

### <a name="enterprise-grade-solutions"></a>Soluciones de clase empresarial
La solución Virtual Assistant se basa en Azure Bot Service, Language Understanding Cognitive Service y Unified Speech, y cuenta con un amplio conjunto de componentes de Azure auxiliares, para que pueda disfrutar de todas las ventajas de la [infraestructura global de Azure](https://azure.microsoft.com/en-gb/global-infrastructure/), incluida la certificación ISO 27018, HIPPA, PCI DSS, SOC 1, 2 y 3.

Además, LUIS Cognitive Service ofrece compatibilidad con Language Understanding para admitir un amplio conjunto de idiomas, que se [enumeran aquí](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-supported-languages). Para ampliar el alcance de Virtual Assistant, [Translator Cognitive Service](https://azure.microsoft.com/en-us/services/cognitive-services/translator-text-api/) ofrece funcionalidades de traducción automática adicionales.

### <a name="integrated-and-context-aware"></a>Integración y reconocimiento contextual
Virtual Assistant se puede integrar en el dispositivo y el ecosistema para una experiencia realmente integrada e inteligente. Con este reconocimiento contextual se pueden desarrollar experiencias más inteligentes y se consigue mayor personalización que de cualquier otra forma.

### <a name="3rd-party-assistant-integration"></a>Integración de asistentes de terceros
Virtual Assistant permite ofrecer una experiencia propia única, pero también el asistente digital preferido para determinados tipos de preguntas de los usuarios finales.

### <a name="flexible-integration"></a>Integración flexible
La arquitectura de Virtual Assistant es flexible y se integra con las inversiones existentes que haya realizado en las funcionalidades de voz basadas en dispositivos o de procesamiento de lenguaje natural y, por supuesto, también con los sistemas de back-end y las API existentes.

### <a name="adaptive-cards"></a>Tarjetas adaptables
Las [tarjetas adaptables](https://adaptivecards.io/) permiten a Virtual Assistant devolver elementos de la experiencia del usuario (como tarjetas, imágenes, botones) junto a las respuestas de texto base. Si el lienzo de conversación o el dispositivo tienen pantalla, las tarjetas adaptables se pueden representar en una amplia gama de dispositivos y plataformas y proporcionar compatibilidad con la experiencia del usuario cuando proceda. [Aquí](https://adaptivecards.io/samples/) encontrará ejemplos de tarjetas adaptables e información sobre las opciones de representación en [esta](https://docs.microsoft.com/en-us/adaptive-cards/rendering-cards/getting-started) documentación.

### <a name="skills"></a>Aptitudes
Además del asistente base, existe un amplio conjunto de funcionalidades comunes que cada desarrollador debe compilar. La productividad es un buen ejemplo donde cada organización necesitaría crear modelos de lenguaje (LUIS), diálogos (código), integración (código) y generación de lenguaje (respuestas) para habilitar experiencias comunes, como calendarios, tareas o correos electrónicos.

Esto se complica por la necesidad de admitir varios idiomas, y supone una gran cantidad de trabajo para cualquier organización que compile su propio asistente.

Nuestra solución Virtual Assistant incluye una nueva funcionalidad de aptitud que permiten la conexión de nuevas funcionalidades a un asistente personalizado solo mediante configuración. 

Todos los aspectos de cada aptitud (modelo de lenguaje, diálogos, código de integración y generación de lenguaje) son completamente personalizables por los desarrolladores, ya que todo el código fuente se encuentra en GitHub junto con Virtual Assistant.

## <a name="getting-started"></a>Introducción

La solución de asistente virtual está disponible en [este repositorio GitHub](https://github.com/Microsoft/AI/tree/master/solutions/Virtual-Assistant) que se actualiza regularmente por el equipo del asistente virtual. Está disponible una documentación más detallada en el mismo repositorio y los problemas y se pueden proporcionar comentarios directamente mediante los mecanismos de comentarios de GitHub. Si tiene un escenario clave que le gustaría discutir con nosotros, rellene [este formulario](https://aka.ms/customassistantpreviewform) y nos pondremos en contacto con usted.


---
title: Introducción a Virtual Assistant | Microsoft Docs
description: Aprenda a crear su propio asistente virtual.
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2b238f4455fc031e1d8ac66f9b408e6d5d936ec9
ms.sourcegitcommit: 7b3d2b5b9b8ce77887a9e6124a347ad798a139ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2019
ms.locfileid: "68991929"
---
# <a name="virtual-assistant-overview"></a>Introducción a Virtual Assistant

## <a name="overview"></a>Información general

Los clientes y asociados tienen una gran necesidad de ofrecer un asistente de conversación adaptado a su marca, personalizado para sus clientes y disponible en una amplia variedad de dispositivos y lienzos.

Siguiendo el enfoque de código abierto de Microsoft en relación a Bot Framework SDK, la solución Virtual Assistant de código abierto proporciona un conjunto de funcionalidades básicas centrales y control total sobre la experiencia del usuario final.

Esta plantilla incorpora la plantilla empresarial anterior y reúne todos los procedimientos recomendados y los componentes auxiliares identificados a través de la creación de experiencias de conversación, y simplifica considerablemente la creación de un proyecto de bot que incluya: intenciones de conversación básicas, integración de Dispatch, QnA Maker, Application Insights y una implementación automatizada.

Creemos firmemente que nuestros clientes deberían tener y enriquecer sus propios datos y relaciones con los clientes. Por eso, cualquier asistente virtual proporciona un control total sobre la experiencia del usuario para nuestros clientes y asociados mediante el código abierto de GitHub. El nombre, la voz y la personalidad se pueden cambiar para satisfacer las necesidades de la organización. Nuestra solución Virtual Assistant simplifica la creación de su propio asistente, y le permite empezar en cuestión de minutos para, posteriormente, ampliar conocimientos con nuestras herramientas de desarrollo globales.

Normalmente, Virtual Assistant ofrece a los usuarios finales una amplia gama de funcionalidades. Para aumentar la productividad de los desarrolladores y habilitar un ecosistema dinámico de experiencias de conversación reutilizables, les proporcionamos ejemplos iniciales de aptitudes de conversación reutilizables. Estas aptitudes se pueden agregar a una aplicación de conversación para aclarar una experiencia de conversación específica, como la búsqueda de un punto de interés, la interacción con el calendario, las tareas, el correo electrónico y muchos otros escenarios. Las aptitudes son totalmente personalizables y consisten en modelos de lenguaje para varios idiomas, diálogos y código.

![Diagrama de Virtual Assistant](./media/enterprise-template/customassistantdiagram.jpg)

## <a name="getting-started"></a>Introducción

Para más información, explore la documentación de [Virtual Assistant y sus aptitudes](https://aka.ms/bfsolutionsdocs).

## <a name="whats-in-the-box"></a>Contenido 

La plantilla de Virtual Assistant reúne una serie de procedimientos recomendados que hemos identificado mediante la creación de experiencias de conversación y automatiza la integración de los componentes que creemos muy ventajosos para los desarrolladores de Bot Framework. En esta sección se cubre información de fondo de las decisiones clave para ayudar a explicar el funcionamiento de la plantilla.

La plantilla de Virtual Assistant ahora incorpora las funcionalidades de la plantilla de empresa anterior, lo que incluye intenciones conversacionales en varios idiomas, distribución, preguntas y respuestas, e información detallada en las conversaciones. Las siguientes funcionalidades relacionadas con el asistente se proporcionan en este momento. Hay otras funcionalidades planeadas y colaboraremos estrechamente con clientes y asociados para informar sobre la hoja de ruta.

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

- **Sector de automoción**: asistente personal con voz integrado en el automóvil para que el usuario final pueda utilizar funcionalidades tradicionales (como la navegación o la radio) y escenarios centrados en la productividad, como posponer reuniones al llegar tarde, incorporar elementos a la lista de tareas y experiencias proactivas en las que el automóvil sugiere completar tareas en función de eventos como el arranque del motor, ir a casa o activar el control de crucero. Las tarjetas adaptables se representan en la unidad principal y la integración de voz se realiza con interacciones de tipo presionar para hablar o con palabras de activación.

- **Hostelería**: asistente personal con voz integrado en un dispositivo de habitación de hotel que ofrece una amplia gama de funcionalidades para escenarios centrados en la hostelería (como la ampliación de la estancia, la solicitud de retrasar el registro de salida, el servicio de habitaciones) incluido el conserje y la posibilidad de buscar restaurantes y atracciones locales. La vinculación opcional a las cuentas de productividad abren la puerta a experiencias más personalizadas, como sugerencias de llamadas de alarma, advertencias sobre el tiempo y el aprendizaje de patrones con las estancias. Una evolución de la personalización de la televisión actual de las habitaciones de hoy en día.

- **Enterprise**: experiencias de asistentes de voz y texto con marca para empleados, integradas en dispositivos empresariales y lienzos conversacionales existentes (como Teams, WebChat, Slack) para que los empleados administren sus calendarios, encuentren las salas de reuniones disponibles, busquen personas con aptitudes específicas o realicen operaciones de recursos humanos.

## <a name="virtual-assistant-principles"></a>Principios de Virtual Assistant

### <a name="your-data-your-brand-and-your-experience"></a>Sus datos, su marca y su experiencia
Podrá controlar todos los aspectos de la experiencia del usuario final, que le pertenece. Esto incluye la personalización de marca, el nombre, la voz, la personalidad, las respuestas y el avatar. El código fuente de Virtual Assistant y de las aptitudes auxiliares está disponible para que lo ajuste según sea necesario.

Virtual Assistant se implementará en la suscripción de Azure. Por lo tanto, todos los datos generados por el asistente (preguntas, comportamiento de los usuarios, etc.) están incluidos por completo en la suscripción de Azure. Consulte [Cognitive Services Azure Trusted Cloud](https://www.microsoft.com/trustcenter/cloudservices/cognitiveservices) y, más concretamente, la [sección de Azure del Centro de confianza](https://www.microsoft.com/TrustCenter/CloudServices/Azure) para más información.

### <a name="write-it-once-embed-it-anywhere"></a>Escríbalo solo una vez, insértelo en cualquier lugar
Virtual Assistant aprovecha la plataforma de inteligencia artificial conversacional de Microsoft y, por lo tanto, se puede utilizar con cualquier [canal](https://docs.microsoft.com/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0) de Bot Framework.

Además, se pueden insertar experiencias en aplicaciones de escritorio y móviles, incluidos dispositivos como automóviles, altavoces y despertadores mediante el canal [Direct Line](https://docs.microsoft.com/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-concepts?view=azure-bot-service-4.0).

### <a name="enterprise-grade-solutions"></a>Soluciones de clase empresarial
La solución Virtual Assistant se basa en Azure Bot Service, Language Understanding Cognitive Service y Unified Speech, y cuenta con un amplio conjunto de componentes de Azure auxiliares. Esto significa que puede beneficiarse de la [infraestructura global de Azure](https://azure.microsoft.com/global-infrastructure/), incluidas las certificaciones ISO 27018, HIPPA, PCI DSS y SOC 1, 2 y 3.

Además, LUIS Cognitive Service ofrece compatibilidad con Language Understanding para admitir un amplio conjunto de idiomas, que se [enumeran aquí](https://docs.microsoft.com/azure/cognitive-services/luis/luis-supported-languages). Para ampliar el alcance de Virtual Assistant, [Translator Cognitive Service](https://azure.microsoft.com/services/cognitive-services/translator-text-api/) ofrece funcionalidades de traducción automática adicionales.

### <a name="integrated-and-context-aware"></a>Integración y reconocimiento contextual
Virtual Assistant se puede incorporar en el dispositivo y el ecosistema para una experiencia realmente integrada e inteligente. Con este reconocimiento contextual se pueden desarrollar experiencias más inteligentes y se consigue mayor personalización que de cualquier otra forma.

### <a name="3rd-party-assistant-integration"></a>Integración de asistentes de terceros
Virtual Assistant permite ofrecer una experiencia propia única, pero también el asistente digital preferido para determinados tipos de preguntas de los usuarios finales.

### <a name="flexible-integration"></a>Integración flexible
La arquitectura de Virtual Assistant es flexible y se integra con las inversiones existentes que haya realizado en las funcionalidades de voz basadas en dispositivos o de procesamiento de lenguaje natural y también con los sistemas de back-end y las API existentes.

### <a name="adaptive-cards"></a>Tarjetas adaptables
Las [tarjetas adaptables](https://adaptivecards.io/) permiten a Virtual Assistant devolver elementos de la experiencia del usuario (como tarjetas, imágenes, botones) junto a las respuestas basadas en texto. Si el lienzo de conversación o el dispositivo tienen pantalla, las tarjetas adaptables se pueden representar en una amplia gama de dispositivos y plataformas y proporcionar compatibilidad con la experiencia del usuario cuando proceda. [Aquí](https://adaptivecards.io/samples/) encontrará ejemplos de tarjetas adaptables e información sobre las opciones de representación en [esta](https://docs.microsoft.com/adaptive-cards/rendering-cards/getting-started) documentación.

### <a name="skills"></a>Aptitudes
Además del asistente base, existe un amplio conjunto de funcionalidades comunes que cada desarrollador debe compilar. La productividad es un buen ejemplo donde cada organización necesitaría crear modelos de lenguaje (LUIS), diálogos (código), integración (código) y generación de lenguaje (respuestas) para habilitar experiencias comunes, como calendarios, tareas o correos electrónicos.

Esto se complica por la necesidad de admitir varios idiomas, y supone una gran cantidad de trabajo para cualquier organización que compile su propio asistente.

Nuestra solución Virtual Assistant incluye una nueva funcionalidad de aptitud que permite la conexión de nuevas funcionalidades a un asistente personalizado solo mediante configuración. 

Todos los aspectos de cada aptitud (modelo de lenguaje, diálogos, código de integración y generación de lenguaje) son completamente personalizables por los desarrolladores, ya que todo el código fuente se encuentra en GitHub junto con Virtual Assistant.

## <a name="getting-started"></a>Introducción

Consulte los [tutoriales](https://aka.ms/bfstutorials) para aprender a crear e implementar Virtual Assistant.

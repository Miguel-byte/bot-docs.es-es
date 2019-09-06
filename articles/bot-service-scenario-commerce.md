---
title: Escenario de bot comercial | Microsoft Docs
description: Explore el escenario de bot de comercial con Bot Framework.
author: BrianRandell
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 9e2eac132922ad90437c88404cbfb97f6f495945
ms.sourcegitcommit: eacf1522d648338eebefe2cc5686c1f7866ec6a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2019
ms.locfileid: "70167082"
---
# <a name="commerce-bot-scenario"></a>Escenario de bot comercial

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

El escenario de [bot comercial](bot-service-scenario-commerce.md) describe un bot que reemplaza las interacciones tradicionales de correo electrónico y llamada telefónica que normalmente tienen los usuarios con el servicio de conserjería de un hotel. El bot aprovecha Cognitive Services para procesar mejor las solicitudes de los clientes por medio de texto y voz con contexto recopilado de la integración con servicios back-end.

![El diagrama de bot de aplicación](~/media/scenarios/bot-service-scenario-commerce-bot.png)

Este es el flujo de la lógica de un bot comercial que funciona como el conserje de un hotel:

1. El cliente usa la aplicación móvil del hotel.
2. El usuario se autentica en Azure AD B2C.
3. El usuario solicita información con un bot de aplicación personalizado. 
4. Cognitive Services ayuda a procesar las solicitudes de lenguaje natural.
5. El cliente revisa la respuesta, quien además puede matizar la pregunta mediante una conversación natural.
6. Una vez que el usuario está satisfecho con el resultado, el bot de aplicación actualiza la reserva del cliente.
7. Application Insights recopila telemetría de tiempo de ejecución para ayudar al desarrollo con el uso y el rendimiento del bot.

## <a name="sample-bot"></a>Bot de ejemplo
El bot comercial de ejemplo está diseñado en torno a un servicio de conserjería de hotel ficticio. Escrito en C#, los clientes acceden al bot una vez que han autenticado Azure AD B2C en un hotel mediante la aplicación móvil de los servicios de miembros de la cadena. La cadena almacena las reservas en una base de datos SQL. Un cliente puede usar preguntas de frases naturales como "¿Cuánto cuesta alquilar una cabaña con piscina durante mi estancia?". El bot a su vez tiene contexto sobre el hotel y la duración de la estancia del huésped. Además, Language Understanding Service (LUIS) permite que los bots obtengan contexto de forma fácil a partir de una frase tan simple como "cabaña con piscina". El bot proporciona la respuesta y, a continuación, puede ofrecerse a reservar una cabaña para el huésped, con opciones en torno al número de días y el tipo de cabaña. Una vez que el bot tiene todos los datos necesarios, reserva la petición. El huésped también puede usar su voz para realizar la misma petición.

Puede descargar o clonar el código fuente de este bot de ejemplo desde los [ejemplos para escenarios comunes de Bot Framework](https://aka.ms/abs-scenarios).

## <a name="components-youll-use"></a>Componentes que usará
El bot comercial usa los siguientes componentes:
-   Azure AD para la autenticación
-   Cognitive Services: LUIS
-   Application Insights

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)
Azure Active Directory (Azure AD) es el directorio basado en la nube multiempresa y el servicio de administración de identidades de Microsoft. Como desarrollador de bots, Azure AD le permite centrarse en la creación del bot al facilitar y acelerar la integración con una solución de administración de identidades de clase mundial usada por millones de organizaciones de todo el mundo. Azure AD admite un conector B2C, lo que permite identificar los individuos mediante identificadores externos, como Google, Facebook o cuenta Microsoft. Azure AD elimina la responsabilidad de tener que administrar las credenciales y se centra en la solución del bot, al saber que usted puede correlacionar el usuario del bot con los datos correctos que expone la aplicación.

### <a name="cognitive-services-luis"></a>Cognitive Services: LUIS
Como miembro de la familia de tecnologías Cognitive Services, Language Understanding (LUIS) incorpora la eficacia del aprendizaje automático en las aplicaciones. Actualmente, LUIS admite varios lenguajes que permiten al bot entender lo que quiere una persona. Cuando se integra con LUIS, usted expresará la intención y definirá las entidades que el bot entiende. Luego, enseñará al bot a entender esas intenciones y entidades mediante su entrenamiento con expresiones de ejemplo. También tiene la posibilidad de ajustar la integración mediante listas de frases y características de expresiones regulares para que el bot funcione lo más fluidamente posible de acuerdo con sus necesidades de conversación específicas.

### <a name="application-insights"></a>Application Insights
Application Insights le ayuda a obtener un conocimiento práctico y detallado mediante la administración del rendimiento de las aplicaciones (APM) y los análisis instantáneos. Obtiene de manera inmediata supervisión del rendimiento, alertas muy eficaces y paneles muy fáciles de usar que le ayudarán a garantizar que el bot esté disponible y con el rendimiento que espera de él. Puede ver rápidamente si hay algún problema y luego llevar a cabo un análisis de la causa principal para detectar el error y corregirlo.

## <a name="next-steps"></a>Pasos siguientes
A continuación, conocerá el escenario de bot de habilidades de Cortana.

> [!div class="nextstepaction"]
> [Escenario de bot de habilidades de Cortana](bot-service-scenario-cortana-skill.md)

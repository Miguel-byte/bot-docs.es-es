---
title: Escenario de bot de información | Microsoft Docs
description: Explore el escenario del bot de información con Bot Framework.
author: BrianRandell
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f62f58de5e15092e34c1e612ef0b54bc2e4b5886
ms.sourcegitcommit: eacf1522d648338eebefe2cc5686c1f7866ec6a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2019
ms.locfileid: "70167374"
---
# <a name="information-bot-scenario"></a>Escenario de bot de información

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Este bot de información podría responder a preguntas definidas en un conjunto de conocimientos o en preguntas frecuentes mediante QnA Maker de Cognitive Services y responder a más preguntas abiertas con Azure Search.

A menudo, la información se encuentra enterrada en almacenes de datos estructurados, como SQL Server, que se pueden desenterrar fácilmente mediante una búsqueda. Imagine buscar el estado del pedido de un cliente mediante sencillos comandos conversacionales. Con QnA Maker de Cognitive Services, al usuario se le presentan un conjunto de opciones de búsqueda válidas, como buscar un cliente, revisar el pedido más reciente del cliente, etc. Con el formato de QnA definido, el usuario puede hacer preguntas fácilmente que estén respaldadas por Azure Search, que puede buscar los datos almacenados en una instancia de SQL Database.

![El diagrama de bot de información](~/media/scenarios/bot-service-scenario-informational-bot.png)

Este es el flujo de la lógica de un bot de información:

1. El empleado inicia el bot de información.
2. Azure Active Directory valida la identidad del empleado.
3. El empleado puede preguntar al bot qué tipo de consultas se admiten.
4. Cognitive Services devuelve un bot de preguntas frecuentes creado con QnA Maker.
5. El empleado define una consulta válida.
6. El bot envía la consulta a Azure Search, que devuelve información sobre los datos de la aplicación.
7. Application Insights recopila telemetría de tiempo de ejecución para ayudar al desarrollo con el uso y el rendimiento del bot.

## <a name="sample-bot"></a>Bot de ejemplo
El bot de ejemplo, escrito en C#, se ejecuta en Microsoft Azure, y trabaja con datos indexados por Azure Search desde una instancia de SQL Database. El bot expone una lista de preguntas que se pueden hacer con información sobre cómo formular la pregunta (la respuesta) mediante Cognitive Services: QnA Maker. El usuario del bot puede escribir entonces una consulta que busca los datos mediante Azure Search en un área amplia o específica de la base de datos que se indexa. En el ejemplo se proporciona una base de datos sencilla con información de clientes y pedidos. Application Insights realiza un seguimiento de uso del bot y le ayuda a supervisar el bot en busca de excepciones. El bot se publica como una aplicación de Azure AD para que pueda restringir quién tiene acceso a la información.

Puede descargar o clonar el código fuente de este bot de ejemplo desde los [ejemplos para escenarios comunes de Bot Framework](https://aka.ms/abs-scenarios).

## <a name="components-youll-use"></a>Componentes que usará
El bot de información usa los siguientes componentes:
-   Azure AD para la autenticación
-   Cognitive Services: QnA Maker
-   Azure Search
-   Application Insights

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)
Azure Active Directory (Azure AD) es el directorio basado en la nube multiempresa y el servicio de administración de identidades de Microsoft. Como desarrollador de bots, Azure AD le permite centrarse en la creación del bot al facilitar y acelerar la integración con una solución de administración de identidades de clase mundial usada por millones de organizaciones de todo el mundo. Mediante la definición de una aplicación de Azure AD, puede controlar quién tiene acceso a su bot y a los datos que expone, sin necesidad de implementar su propio sistema complejo de autenticación y autorización.

### <a name="cognitive-services-qna-maker"></a>Cognitive Services: QnA Maker
QnA Maker de Cognitive Services le ayuda a proporcionar un origen de datos de preguntas frecuentes que los usuarios pueden consultar desde su bot. Cuando deba hacer frente a enormes cantidades de información almacenada en distintos sistemas, puede ser útil ayudar a los usuarios a filtrar el origen y el conjunto de información. Una única base de datos SQL puede tener enormes cantidades de información que cuando se aplica una búsqueda de forma libre devuelve demasiada información. Si usa primero QnA Maker, puede definir un mapa de carreteras para los usuarios del bot de forma que sepan cómo hacer preguntas inteligentes que luego se puedan recuperar mediante Azure Search.

### <a name="azure-search"></a>Azure Search
Azure Search es un servicio de búsqueda en la nube de aplicaciones que le permite tener sus índices de búsqueda en funcionamiento rápidamente. Como se ejecuta sobre Microsoft Azure, puede escalar y reducir verticalmente de manera fácil según la demanda de uso. Los resultados de búsqueda se pueden conectar con los objetivos empresariales y tener un mayor control sobre la clasificación de búsqueda y los datos de superficie ocultos en las bases de datos.

### <a name="application-insights"></a>Application Insights
Application Insights le ayuda a obtener un conocimiento práctico y detallado mediante la administración del rendimiento de las aplicaciones (APM) y los análisis instantáneos. Obtiene de manera inmediata supervisión del rendimiento, alertas muy eficaces y paneles muy fáciles de usar que le ayudarán a garantizar que el bot esté disponible y con el rendimiento que espera de él. Puede ver rápidamente si hay algún problema y luego llevar a cabo un análisis de la causa principal para detectar el error y corregirlo.

## <a name="next-steps"></a>Pasos siguientes
A continuación, conozca el escenario de bot de Internet de las cosas.

> [!div class="nextstepaction"]
> [Escenario de bot de Internet de las cosas](bot-service-scenario-internet-things.md)

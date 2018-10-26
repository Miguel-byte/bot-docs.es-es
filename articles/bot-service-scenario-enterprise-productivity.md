---
title: Escenario de bot de productividad empresarial | Microsoft Docs
description: Explore el escenario de bot de productividad empresarial con Bot Framework.
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: feefa25d0c85c3aa62705b075253372f12e9146e
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326382"
---
# <a name="enterprise-productivity-bot-scenario"></a>Escenario de bot de productividad empresarial

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

El bot empresarial muestra cómo puede aumentar su productividad mediante la integración de un bot con el calendario de Office 365 y otros servicios.

El bot de productividad empresarial se trata del acceso rápido a información del cliente sin la necesidad de tener abiertas muchas ventanas. Mediante comandos de chat sencillos, un representante de ventas puede buscar un cliente y revisar su próxima cita a través de Graph API y Office 365. Desde ahí, puede acceder a información específica del cliente almacenada en Dynamics CRM, como la obtención de un caso o la creación de uno nuevo.

![El diagrama del bot empresarial](~/media/scenarios/bot-service-scenario-enterprise-bot.png)

Este es el flujo lógico de un bot de productividad empresarial:

1. El empleado accede al bot de productividad empresarial.
2. Azure Active Directory valida la identidad del empleado.
3. El bot de productividad empresarial puede consultar el calendario de Office 365 del empleado a través de Azure Graph.
4. Con los datos recopilados del calendario, el bot accede a información del caso en Dynamics CRM.
5. La información se devuelve al empleado, que puede filtrar los datos sin dejar el bot.
6. Application Insights recopila telemetría de tiempo de ejecución para ayudar al desarrollo con el uso y el rendimiento del bot.

Puede descargar o clonar el código fuente de este bot de ejemplo desde los [ejemplos para escenarios comunes de Bot Framework](https://aka.ms/bot/scenarios).

## <a name="sample-bot"></a>Bot de ejemplo
Como los bots son accesibles desde una variedad de canales, puede usarlos en el escritorio desde un portal corporativo o desde Skype en cualquier lugar: solo se debe autenticar. Con la integración de Azure AD, el bot de productividad empresarial sabe que si puede acceder a él, es porque Azure AD lo autenticó. Desde allí, puede pedirle al bot que compruebe cuándo es su próxima cita con un cliente determinado. El bot obtiene esta información mediante una consulta a Office 365 a través de Graph API. A continuación, si hay una cita en los próximos siete días, el bot consulta a CMR que busca cualquier caso reciente para el cliente. El bot responde ya sea sin casos encontrados o con el número de casos abiertos y cerrados. Desde allí puede pedirle al bot para enumerar los casos por tipo y profundizar en los casos individuales.

## <a name="components-youll-use"></a>Componentes que usará
El bot de productividad empresarial usa estos componentes:
-   Azure AD para la autenticación
-   Graph API a Office 365
-   Dynamics CRM
-   Application Insights

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)
Azure Active Directory (Azure AD) es el directorio basado en la nube multiempresa y el servicio de administración de identidades de Microsoft. Como desarrollador de bots, Azure AD le permite centrarse en la creación del bot al facilitar y acelerar la integración con una solución de administración de identidades de clase mundial usada por millones de organizaciones de todo el mundo. Mediante la definición de una aplicación de Azure AD, puede controlar quién tiene acceso a su bot y a los datos que expone, sin necesidad de implementar su propio sistema complejo de autenticación y autorización.

### <a name="graph-api-to-office-365"></a>Graph API a Office 365
Microsoft Graph expone varias API de Office 365 y otros Servicios en la nube de Microsoft a través de un punto de conexión único en https://graph.microsoft.com. Microsoft Graph facilita que usted y el bot ejecuten consultas. La API expone los datos de varios Servicios en la nube de Microsoft, lo que incluye Exchange Online como parte de Office 365, Azure Active Directory, SharePoint, etc. Puede usar la API para navegar entre las entidades y las relaciones. Puede usar la API de los bots con los puntos de conexión de SDK o REST, así como de las otras aplicaciones con compatibilidad nativa de Android, iOS, Ruby, UWP, Xamarin, etc.

### <a name="dynamics-crm"></a>Dynamics CRM
Dynamics CRM es una plataforma de interacción con los clientes. Mediante el uso de los bots y las API de CRM, puede compilar bots interactivos enriquecidos que pueden acceder a los datos enriquecidos almacenados en CRM. La potencia de Dynamics CRM está disponible para que el bot cree casos, compruebe estados, realice búsquedas de administración de conocimiento, etc.

### <a name="application-insights"></a>Application Insights
Application Insights le ayuda a obtener un conocimiento práctico y detallado mediante la administración del rendimiento de las aplicaciones (APM) y los análisis instantáneos. Obtiene de manera inmediata supervisión del rendimiento, alertas muy eficaces y paneles muy fáciles de usar que le ayudarán a garantizar que el bot esté disponible y con el rendimiento que espera de él. Puede ver rápidamente si hay algún problema y luego llevar a cabo un análisis de la causa principal para detectar el error y corregirlo.

## <a name="next-steps"></a>Pasos siguientes
A continuación, obtenga información sobre el escenario de bot de información.

> [!div class="nextstepaction"]
> [Escenario de bot de información](bot-service-scenario-informational.md)

---
title: Escenario del bot de habilidades de Cortana | Microsoft Docs
description: Explore el escenario del bot de habilidades de Cortana con Bot Framework.
author: BrianRandell
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d6ba5b414ff7e600fac1e5d4ebce27363340f42f
ms.sourcegitcommit: eacf1522d648338eebefe2cc5686c1f7866ec6a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2019
ms.locfileid: "70167053"
---
# <a name="cortana-skills-bot-scenario"></a>Escenario del bot de habilidades de Cortana

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

El bot de habilidades de Cortana amplía Cortana para que, por ejemplo, sea más fácil reservar una cita para realizar el mantenimiento del automóvil mediante la voz y con el contexto del calendario.

Cortana es su asistente personal. Mediante la interfaz natural de su voz y un bot de habilidad de Cortana personalizado, puede pedir a Cortana que hable con una organización, por ejemplo, una empresa de automóviles, para ayudarle a fijar una cita. El servicio puede proporcionar una lista de servicios, las horas disponibles y la duración. Cortana puede consultar su calendario para ver si tiene algo que hacer a esa misma hora y, si no es así, creará la cita y la agregará al calendario.

![Diagrama del bot de habilidades de Cortana](~/media/scenarios/bot-service-scenario-cortana-skill.png)

Este es el flujo lógico de un bot habilidades de Cortana para una tienda de automóviles:

1. El usuario accede a Cortana desde su PC o dispositivo móvil.
2. Usando los comandos de texto o de voz, el usuario solicita una cita para hacer el mantenimiento del automóvil.
3. Como el bot está integrado en Cortana, tiene acceso al calendario del usuario y aplica lógica a la solicitud.
4. Con esa información, el bot puede consultar al servicio de automóviles para ver si hay horas disponibles.
5. A continuación, se muestran varias opciones contextuales al usuario para que pueda reservar una cita.
6. Application Insights recopila telemetría en tiempo de ejecución para facilitar el desarrollo en función del uso y el rendimiento del bot.

## <a name="sample-bot"></a>Bot de ejemplo
Con un bot de habilidades de Cortana, todo gira en torno a un contexto personal. Gracias a Cortana, puede usar su voz para solicitar "Mantenimiento de automóviles de Bob" para que venga a hacer el mantenimiento del automóvil en función de su ubicación. Al usar información personal expuesta a través de Cortana, el bot puede confirmar la ubicación en función de dónde se encuentra el usuario cuando está hablando con el bot.

Puede descargar o clonar el código fuente de este bot de ejemplo de [Samples for Common Bot Framework Scenarios](https://aka.ms/abs-scenarios) (Muestras para escenarios comunes de Bot Framework).

## <a name="components-youll-use"></a>Componentes que va a utilizar
El bot Cortana utiliza los siguientes componentes:
-   Cortana
-   Application Insights

### <a name="cortana"></a>Cortana
Ahora puede agregar soporte técnico al bot mediante la creación de una habilidad de Cortana. Utilice el kit de habilidades de Cortana para crear nuevas características (llamadas habilidades) para Cortana. Una habilidad es una construcción que permite a Cortana hacer más cosas. Desarrolle habilidades e intégrelas con los bots, para que Cortana pueda realizar y completar tareas. Como parte del proceso de invocación, Cortana puede (con el consentimiento del usuario) pasar información sobre el usuario a una habilidad en tiempo de ejecución, de modo que la habilidad pueda personalizar la experiencia en consecuencia. El conocimiento contextual de Cortana permite al bot ser más útil e inteligente. Una vez invocado, ciertos tipos de habilidades pueden manipular la interfaz de Cortana para mantener una conversación entre la habilidad y el usuario final. Una vez publicadas, los usuarios pueden ver y utilizar las habilidades de Cortana para la Actualización de aniversario de Windows 10 (escritorio y móvil), iOS y Android.

### <a name="application-insights"></a>Application Insights
Application Insights le ayuda a obtener un conocimiento práctico y detallado mediante la administración del rendimiento de las aplicaciones (APM) y los análisis instantáneos. Obtiene de manera inmediata supervisión del rendimiento, alertas muy eficaces y paneles muy fáciles de usar que le ayudarán a garantizar que el bot esté disponible y con el rendimiento que espera de él. Puede ver rápidamente si hay algún problema y luego llevar a cabo un análisis de la causa principal para detectar el error y corregirlo.

## <a name="next-steps"></a>Pasos siguientes
A continuación, obtenga información acerca del escenario de bot de productividad empresarial.

> [!div class="nextstepaction"]
> [Escenario de bot de productividad empresarial](bot-service-scenario-enterprise-productivity.md)

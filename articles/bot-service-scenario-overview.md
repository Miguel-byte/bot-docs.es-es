---
title: Introducción a los escenarios de Servicio de bots | Microsoft Docs
description: Explore los escenarios clave para bots potentes y eficaces creados con Servicio de bots.
author: BrianRandell
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: c4a0af8bfd6496b82ddcc9219e948395d87e4ff3
ms.sourcegitcommit: eacf1522d648338eebefe2cc5686c1f7866ec6a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2019
ms.locfileid: "70167357"
---
# <a name="bot-scenarios"></a>Escenarios de bot

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

En este tema se exploran los escenarios clave para bots potentes y eficaces creados con Servicio de bots.

Puede descargar o clonar el código fuente de todos los ejemplos de escenarios de bots de [Samples for Common Bot Framework Scenarios](https://aka.ms/abs-scenarios).

## <a name="commerce-bot-scenario"></a>Escenario de bot comercial
El escenario de [bot comercial](bot-service-scenario-commerce.md) describe un bot que reemplaza las interacciones tradicionales de correo electrónico y llamada telefónica que normalmente tienen los usuarios con el servicio de conserjería de un hotel. El bot aprovecha Cognitive Services para procesar mejor las solicitudes de los clientes a través de texto y voz con contexto recopilado de la integración con servicios back-end.

En el escenario de bot comercial, una clienta puede realizar una solicitud de servicios de conserjería a un hotel. Se ha autenticado a través de un punto de conexión de autenticación de Azure Active Directory v2. El bot puede buscar las reservas de la clienta y proporcionar distintas opciones de servicio. Por ejemplo, la cliente podría reservar una cabaña junto a la piscina. El bot usa Language Understanding Intelligent Services (LUIS) para analizar la solicitud y, a continuación, el bot guía a la usuaria por el proceso de registro de una cabaña para una reserva existente.

## <a name="cortana-skill-bot-scenario"></a>Escenario de bot de habilidad de Cortana
El escenario de [bot de habilidad de Cortana](bot-service-scenario-cortana-skill.md) aprovecha las ventajas de Cortana. Mediante la interfaz natural de su voz y un bot de habilidad de Cortana personalizado, puede pedir a Cortana que hable con una organización, por ejemplo, una empresa de estética automotriz móvil, para ayudarle a fijar una cita en función de dónde se encuentre cuando hace la llamada. El bot puede proporcionar una lista de servicios, las horas disponibles y la duración.

## <a name="enterprise-productivity-bot-scenario"></a>Escenario de bot de productividad empresarial
El escenario de [bot de productividad empresarial](bot-service-scenario-enterprise-productivity.md) muestra cómo integrar un bot con el calendario de Office 365 y otros servicios para aumentar su productividad.

El bot se integra con Office 365 para que sea más rápido y fácil crear una solicitud de reunión con otra persona. En el proceso, podría tener acceso a servicios adicionales, como Dynamics CRM. Este ejemplo proporciona el código necesario para la integración con Office 365 mediante la autenticación a través de Azure Active Directory. Proporciona puntos de entrada ficticias para servicios externos como ejercicio para el lector.

## <a name="information-bot-scenario"></a>Escenario de bot de información
Este [bot de información](bot-service-scenario-informational.md) puede responder preguntas definidas en una serie de conocimientos o en las preguntas frecuentes mediante QnA Maker de Cognitive Services y responder más preguntas ilimitadas con Azure Search.

A menudo, la información se encuentra enterrada en almacenes de datos estructurados, como SQL Server, que se pueden desenterrar fácilmente mediante una búsqueda. Imagine buscar el estado del pedido de un cliente mediante sencillos comandos conversacionales. Con QnA Maker de Cognitive Services, se presenta al usuario un conjunto de opciones de búsqueda válidas, como buscar un cliente, revisar el pedido más reciente del cliente, etc. Con el formato de QnA definido, el usuario puede hacer preguntas fácilmente que estén respaldadas por Azure Search, que puede buscar los datos almacenados en una instancia de SQL Database.

## <a name="iot-bot-scenario"></a>Escenario de bot de IoT
Este [bot de Internet de las cosas (IoT)](bot-service-scenario-internet-things.md) le hace más fácil la tarea de controlar los dispositivos de su casa, por ejemplo, una luz Philips Hue mediante comandos de chat interactivo.

Mediante este sencillo bot, puede controlar las luces Philips Hue junto con el servicio gratuito IFTTT (si esto, entonces aquello). Como dispositivo de IoT, Philips Hue se puede controlar localmente a través de su API expuesta. Sin embargo, esta API no se expone para el acceso general desde fuera de la red local. Sin embargo, IFTTT es miembro de "[Friends of Hue](http://www2.meethue.com/friends-of-hue/ifttt/)", por tanto, tiene expuesta una serie de comandos de control que se pueden emitir, como encender y apagar luces, cambiar el color o la intensidad de la luz.

## <a name="next-steps"></a>Pasos siguientes
Ahora que tiene una visión general de los escenarios, profundice en cada uno.

> [!div class="nextstepaction"]
> [Bot comercial](bot-service-scenario-commerce.md)

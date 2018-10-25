---
title: Escenario de bot de Internet de las cosas | Microsoft Docs
description: Explore el escenario de bot de Internet de las cosas con Bot Framework.
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0cdade289bc7474a3d2597ae54fd2c355a7969f9
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997002"
---
# <a name="internet-of-things-iot-bot-scenario"></a>Escenarios de bot de Internet de las cosas (IoT)

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Este bot de Internet de las cosas (IoT) le hace más fácil la tarea de controlar los dispositivos de su casa, por ejemplo, una luz Philips Hue mediante comandos de voz o de chat interactivo.

A la gente le encanta hablar con sus cosas. Desde los días del primer mando para TV, a las personas les ha encantado no tener que moverse para interactuar con su entorno. Este bot de IoT permite a una persona administrar una luz Philips Hue mediante comandos simples de chat o voz. Además, al utilizar el chat, una persona puede recibir opciones visuales relacionadas con los colores para elegir.

![Diagrama del bot de Internet de las cosas](~/media/scenarios/bot-service-scenario-iot-bot.png)

Este es el flujo lógico de un bot de IoT:

1. El usuario se registra en Skype y accede al bot de IoT.
2. El usuario pide por voz al bot que encienda las luces a través del dispositivo IoT.
3. La solicitud se retransmite a un servicio de terceros que tiene acceso a la red del dispositivo IoT.
4. Los resultados del comando se devuelven al usuario.
5. Application Insights recopila telemetría de tiempo de ejecución para facilitar el desarrollo con el uso y el rendimiento del bot.

## <a name="sample-bot"></a>Bot de ejemplo
El bot de IoT le permitirá usar rápidamente comandos de chat de canales como Skype o Slack para controlar la luz Hue. Para facilitar el acceso remoto, llamará a applets IFTTT predefinidos para que funcionen con Hue.

Puede descargar o clonar el código fuente de este bot de ejemplo de [Samples for Common Bot Framework Scenarios](https://aka.ms/bot/scenarios).

## <a name="components-youll-use"></a>Componentes que va a utilizar
El bot de Internet de las cosas (IoT) usa los siguientes componentes:
-   Philips Hue
-   IFTTT (si esto, entonces aquello)
-   Application Insights

### <a name="philips-hue"></a>Philips Hue
Las bombillas y el puente conectados Philips Hue le permite asumir el control completo de la iluminación. No importa lo que quiera hacer con la iluminación, Hue puede encargarse. Hue tiene una API que usted puede usar desde la red local. Sin embargo, desea poder acceder a los dispositivos y luces Hue controlados desde cualquier lugar mediante una interfaz sencilla de bot. Por lo tanto, accederá a Hue mediante IFTTT.

### <a name="ifttt"></a>IFTTT
IFTTT es un servicio gratuito basado en web que las personas usan para crear cadenas de instrucciones condicionales sencillas, llamadas applets. Puede desencadenar un applet desde el bot para que haga algo en su nombre. Hay una serie de applets de Hue predefinidos disponibles para encender y apagar las luces, cambiar la escena y mucho más.

### <a name="application-insights"></a>Application Insights
Application Insights le ayuda a obtener un conocimiento práctico y detallado mediante la administración del rendimiento de las aplicaciones (APM) y los análisis instantáneos. Obtiene de manera inmediata supervisión del rendimiento, alertas muy eficaces y paneles muy fáciles de usar que le ayudarán a garantizar que el bot esté disponible y con el rendimiento que espera de él. Puede ver rápidamente si hay algún problema y luego llevar a cabo un análisis de la causa principal para detectar el error y corregirlo.

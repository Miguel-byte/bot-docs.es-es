---
title: Conversaciones de transición de bot a humano | Microsoft Docs
description: Aprenda a diseñar situaciones donde un usuario inicia una conversación con un bot y, a continuación, se le pasa a un humano.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: f18b375a1e4ebcf06d00d045e383db8b05fb5111
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225740"
---
# <a name="transition-conversations-from-bot-to-human"></a>Conversaciones de transición de bot a humano

Con independencia de cuánta inteligencia artificial posea un bot, todavía puede haber ocasiones en que sea necesario pasar la conversación a un ser humano. El bot debe reconocer esta necesidad y proporcionar al usuario una transición suave y clara.

## <a name="scenarios-that-require-human-involvement"></a>Escenarios que requieren la intervención humana

Existe una amplia variedad de escenarios que pueden requerir que un bot transfiera el control de la conversación a un humano. Algunos de estos escenarios son *evaluación de prioridades*, *escalamiento* y *supervisión*. 

### <a name="triage"></a>Evaluación de errores

Una llamada habitual al departamento de soporte técnico con algunas preguntas muy básicas que puede responder fácilmente un bot. Como el primero en responder a las solicitudes entrantes de los usuarios, un bot podría recopilar el nombre del usuario, la dirección, la descripción del problema o cualquier otra información pertinente, y luego transferir el control de la conversación a un agente. Usar un bot para evaluar las prioridades de las solicitudes entrantes permite a los agentes dedicar su tiempo a solucionar el problema en lugar de recopilar información.

### <a name="escalation"></a>Escalado

En el escenario del departamento de soporte técnico, un bot puede responder a preguntas básicas y resolver problemas sencillos, además de recopilar información, como el restablecimiento de la contraseña de un usuario. Sin embargo, si una conversación indica que el problema de un usuario es lo bastante complejo como para requerir la intervención humana, el bot debe escalar el problema a un agente humano. Para implementar este tipo de escenario, un bot debe ser capaz de diferenciar entre los problemas que puede resolver de forma independiente y los problemas que deben escalarse a ser humano. Hay muchas maneras en que un bot puede determinar que necesita transferir el control de la conversación a un humano. 

#### <a name="user-driven-menus"></a>Menús controlados por el usuario

Quizás la forma más sencilla para un bot de hacer frente a este dilema es presentar al usuario un menú de opciones. Las tareas que el bot puede controlar de forma independiente aparecen en el menú situado encima de un vínculo llamado "Chat con un agente". Este tipo de implementación no requiere conocimientos avanzados de aprendizaje automático ni comprender el lenguaje natural. El bot simplemente transfiere el control de la conversación a un agente humano cuando el usuario selecciona la opción "Chat con un agente". 

#### <a name="scenario-driven"></a>Según el escenario

El bot puede decidir si transferir o no el control en función de si determina o no que es capaz de controlar el escenario en cuestión. El bot recopila alguna información sobre la solicitud del usuario y, a continuación, consulta su lista interna de funcionalidades para determinar si es capaz de abordar esa solicitud. Si el bot determina que es capaz de abordar la solicitud, lo hace, pero si determina que la solicitud está fuera del ámbito de problemas que puede resolver, transfiere el control de la conversación a un agente humano.

#### <a name="natural-language"></a>Lenguaje natural

La comprensión del lenguaje natural y el análisis de opiniones ayudan al bot a decidir cuándo se debe transferir el control de la conversación a un agente humano. Esto resulta especialmente útil al intentar determinar cuándo el usuario se siente frustrado o quiere hablar con un agente humano. 
 
El bot analiza el contenido de los mensajes del usuario mediante <a href="https://www.microsoft.com/cognitive-services/en-us/text-analytics-api" target="blank">Text Analytics API</a> para deducir los sentimientos o por medio de <a href="https://www.luis.ai" target="_blank">LUIS API</a>. 


> [!TIP]
> Puede que la comprensión del lenguaje natural no sea siempre el mejor método para determinar cuándo un bot debe transferir el control de la conversación a un ser humano. Los bots, al igual que los humanos, no siempre aciertan y las respuestas no válidas dejan al usuario frustrado. Sin embargo, si el usuario selecciona de un menú de opciones válidas, el bot siempre responderá correctamente a esa intervención. 

### <a name="supervision"></a>Supervisión

En algunos casos, el agente humano querrá supervisar la conversación en lugar de tomar el control.

Por ejemplo, considere un escenario del departamento de soporte técnico donde un bot se está comunicando con un usuario para diagnosticar problemas del equipo. Un modelo de aprendizaje automático ayuda al bot a determinar la causa más probable del problema; sin embargo, antes de aconsejar al usuario tomar un curso de acción específico, el bot puede confirmar de forma privada el diagnóstico y el remedio con el agente humano y solicitar autorización para continuar. El agente entonces hace clic en un botón, el bot presenta la solución al usuario y el problema se resuelve. El bot sigue realizando la mayoría del trabajo, pero el agente mantiene el control de la decisión final. 

## <a name="transitioning-control-of-the-conversation"></a>Transición del control de la conversación 

Cuando un bot decide transferir el control de una conversación a un humano, puede informar al usuario de que se le está transfiriendo y poner la conversación en estado de espera hasta que confirme que hay un agente disponible. 

Cuando el bot está esperando a un humano, puede responder automáticamente a todos los mensajes de usuario entrantes con una respuesta predeterminada como "esperando en cola". Además, podría hacer que el bot quitara la conversación de estado de espera si el usuario envía ciertos mensajes como "no se preocupe" o "cancelar".

Usted especifica cómo se asignarán los agentes a los usuarios en espera al diseñar su bot. Por ejemplo, el bot puede implementar un sistema de cola sencillo: primero en entrar, primero en salir. Una lógica más compleja asignaría a los usuarios a los agentes según la ubicación geográfica, el idioma u otros factores. El bot también podría presentar algún tipo de interfaz de usuario al agente que puede usar para seleccionar un usuario. Cuando un agente está disponible, se conecta con el bot y se une a la conversación.

> [!IMPORTANT]
> Incluso después de que un agente está interactuando, el bot permanece en segundo plano como facilitador de la conversación. El usuario y el agente nunca se comunican directamente entre sí; simplemente enrutan los mensajes a través del bot. 

## <a name="routing-messages-between-user-and-agent"></a>Enrutamiento de mensajes entre el usuario y el agente

Después de que el agente se conecta al bot, el bot empieza a enrutar los mensajes entre usuario y el agente. Aunque puede parecer que el usuario y el agente están conversando directamente entre ellos, en realidad están intercambiando mensajes a través del bot. El bot recibe mensajes del usuario y envía esos mensajes al agente y, a su vez, recibe mensajes del agente y envía esos mensajes al usuario. 

> [!NOTE]
> En escenarios más avanzados, el bot puede asumir una responsabilidad que va más allá de simplemente enrutar los mensajes entre el usuario y el agente. Por ejemplo, el bot puede decidir qué respuesta es adecuada y simplemente pedir al agente confirmación para continuar.

## <a name="sample-code"></a>Código de ejemplo

Para ver un ejemplo completo que muestra cómo pasar las conversaciones de bot a humano mediante Bot Framework SDK para Node.js, consulte <a href="https://github.com/palindromed/Bot-HandOff" target="_blank">Ejemplo Bot-HandOff</a> en GitHub.

## <a name="additional-resources"></a>Recursos adicionales

::: moniker range="azure-bot-service-4.0"

- [Diálogos](v4sdk/bot-builder-dialog-manage-conversation-flow.md)
- <a href="https://www.microsoft.com/cognitive-services/en-us/text-analytics-api" target="blank">Text Analytics API</a>

::: moniker-end

::: moniker range="azure-bot-service-3.0"

- [Administración del flujo de conversación con diálogos (.NET)](~/dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [Manage conversation flow with dialogs (Node.js)](~/nodejs/bot-builder-nodejs-manage-conversation-flow.md) (Administración del flujo de conversación con cuadros de diálogo [Node.js])
- <a href="https://www.microsoft.com/cognitive-services/en-us/text-analytics-api" target="blank">Text Analytics API</a>


::: moniker-end


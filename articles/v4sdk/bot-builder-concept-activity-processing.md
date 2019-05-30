---
title: Procesamiento de actividades | Microsoft Docs
description: Descripción del procesamiento de actividades en el SDK del bot.
keywords: adaptador de bot, software intermedio personalizado, cortocircuito, reserva, controladores de eventos
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 69ce362f35054c5ee42035d8bffefb17c6fa7f71
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215503"
---
# <a name="activity-processing"></a>Procesamiento de actividades

[!INCLUDE[applies-to](../includes/applies-to.md)]

El bot y el usuario interactúan e intercambian información mediante actividades. Cada una de las actividades que recibe la aplicación de bot se pasa a un adaptador de bot, que a su vez pasa la información de la actividad a la lógica del bot y, por último, envía las respuestas al usuario. La recepción de una actividad y su posterior procesamiento por el bot se denomina turno; representa un ciclo completo del bot. Un turno finaliza cuando se termina toda la ejecución, la actividad se ha procesado por completo y se han completado todas las capas del bot.

Las actividades, especialmente las [enviadas desde un bot](#generating-responses) durante un turno del bot, se controlan de forma asincrónica. Se trata de una parte necesaria de la creación de un bot; si necesita repasar cómo funciona todo esto, vea [Programación asincrónica para .NET](https://docs.microsoft.com/en-us/dotnet/csharp/async) o [async para JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function), según el lenguaje que haya elegido.

## <a name="the-bot-adapter"></a>El adaptador de bot

El adaptador de bot encapsula los procesos de autenticación, envía actividades a Bot Connector Service y recibe actividades de este. Cuando el bot recibe una actividad, el adaptador encapsula todo lo relativo a esa actividad, crea un [objeto de contexto](#turn-context) para el turno, lo pasa a la lógica de aplicación del bot y envía las respuestas que genera el bot al canal del usuario.

## <a name="authentication"></a>Authentication

El adaptador autentica todas las actividades entrantes que recibe la aplicación mediante la información de la actividad y el encabezado `Authentication` de la solicitud de REST. El adaptador usa un objeto de conector y las credenciales de la aplicación para autenticar las actividades de salida con destino al usuario.

La autenticación de Bot Connector Service usa tokens `Bearer` de JWT (JSON Web Token), así como el **identificador de aplicación de Microsoft** y la **contraseña de aplicación de Microsoft** que Azure genera automáticamente cuando se crea un servicio de bot o se registra un bot. La aplicación necesitará estas credenciales en el tiempo de inicialización para permitir que el adaptador autentique el tráfico.

> [!NOTE]
> Si está ejecutando o probando el bot localmente (por ejemplo, mediante Bot Framework Emulator), puede hacerlo sin configurar el adaptador para autenticar el tráfico hacia y desde el bot.

## <a name="turn-context"></a>Contexto de turno

Cuando un adaptador recibe una actividad, genera un objeto de **contexto de turno**, que proporciona información sobre la actividad entrante, el remitente, el receptor, el canal, la conversación y otros datos necesarios para procesar la actividad. Después, el adaptador pasa este objeto de contexto al bot. El objeto de contexto se conserva mientras dure el turno y proporciona información sobre lo siguiente:

* Conversación: identifica la conversación e incluye información sobre el bot y el usuario que participan en la conversación.
* Actividad: las solicitudes y las respuestas de una conversación son los tipos de actividades. Este contexto proporciona información sobre la actividad entrante, incluida la información de enrutamiento, la información sobre el canal, la conversación, el remitente y el receptor.
* Información personalizada: si extiende el bot mediante la implementación de software intermedio o dentro de la lógica del bot, puede hacer que haya información adicional disponible en cada turno.

El objeto de contexto también se puede usar para enviar una respuesta al usuario y obtener una referencia al adaptador.<!-- to create a new conversation or continue an existing one-->.

> [!NOTE]
> La aplicación y el adaptador controlarán las solicitudes de forma asincrónica, pero no es necesario que la lógica de negocios esté controlada por la solicitud-respuesta.

## <a name="middleware"></a>Software intermedio

Puede agregar software intermedio al adaptador. El software intermedio y la lógica del bot usan el objeto de contexto para recuperar información sobre la actividad y actuar en consecuencia. El software intermedio y el bot también pueden actualizar o agregar información al objeto de contexto, por ejemplo, para llevar un seguimiento del estado de un turno, una conversación u otro ámbito. Para información más detallada sobre el middleware, consulte el [artículo sobre el middleware](~/v4sdk/bot-builder-concept-middleware.md).

## <a name="generating-responses"></a>Generación de respuestas

El objeto de contexto proporciona métodos de respuesta de actividad para permitir que el código responda a una actividad:

* Los métodos _send activity_ y _send activities_ envían una o más actividades a la conversación.
* Si el canal lo admite, el método _update activity_ actualiza una actividad de la conversación.
* Si el canal lo admite, el método _delete activity_ elimina una actividad de la conversación.

Cada método de respuesta se ejecuta en un proceso asincrónico. Cuando se llama al método de respuesta de actividad, este clona la lista de [controladores de eventos](#response-event-handlers) asociados antes de empezar a llamar a los controladores, lo que significa que contendrá todos los controladores agregados hasta ese momento, pero no contendrá nada de lo que se agregue una vez iniciado el proceso.

Esto también significa que no se garantiza el orden de las respuestas, sobre todo cuando una tarea es más compleja que otra. Si el bot puede generar varias respuestas para una actividad entrante, asegúrese de que tienen sentido en cualquier orden en el que las reciba el usuario.

> [!IMPORTANT]
> El subproceso que administra el turno de bot principal se ocupa de desechar el objeto de contexto cuando termina. Si una respuesta (incluidos sus controladores) tarda mucho e intenta actuar sobre el objeto de contexto, es posible que reciba un error `Context was disposed`. **Asegúrese de usar `await` para las llamadas de actividad**, a fin de que el subproceso principal espere la actividad generada antes de finalizar su procesamiento y desechar el contexto de turno.

## <a name="response-event-handlers"></a>Controladores de eventos de respuesta

Además de la lógica del bot y del software intermedio, se pueden agregar controladores de respuesta (a veces denominados controladores de eventos o controladores de eventos de actividad) al objeto de contexto. Se llama a estos controladores cuando la [respuesta](#generating-responses) asociada se produce en el objeto de contexto actual, antes de ejecutar la respuesta real. Estos controladores son útiles si sabe que querrá hacer algo, ya sea antes o después del evento real, para todas las actividades de ese tipo durante el resto de la respuesta actual.

> [!WARNING]
> Tenga cuidado de no llamar a un método de respuesta de actividad desde su controlador de eventos de respuesta correspondiente, por ejemplo, mediante una llamada al método de actividad de envío desde un controlador _on send activity_. Si lo hace, puede generar un bucle infinito.

Cada actividad nueva obtiene un nuevo subproceso en el que se ejecuta. Cuando se crea el subproceso para procesar la actividad, la lista de controladores de esa actividad se copia en ese subproceso nuevo. No se ejecutará para ese evento de actividad específico ningún controlador agregado después de ese punto.

El adaptador administra los controladores registrados en un objeto de contexto de forma muy similar al modo en que administra la [canalización de middleware](~/v4sdk/bot-builder-concept-middleware.md#the-bot-middleware-pipeline). Es decir, se llama a los controladores en el orden en que se agregan, y al llamar al delegado _next_ se pasa el control al siguiente controlador de eventos registrado. Si un controlador no llama al delegado next, no se llama a ninguno de los controladores de eventos posteriores; se produce un [cortocircuito](~/v4sdk/bot-builder-concept-middleware.md#short-circuiting) en el evento y el adaptador no envía la respuesta al canal.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Software intermedio](~/v4sdk/bot-builder-concept-middleware.md)

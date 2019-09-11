---
ms.openlocfilehash: c6abff25dcccce4c18ad0083ef24b69495222c9f
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/16/2019
ms.locfileid: "70297352"
---
Normalmente, cada mensaje que envía un bot al usuario se relaciona directamente con la anterior entrada del usuario.
En algunos casos, puede que un bot tenga que enviar al usuario un mensaje que no está relacionado directamente con el tema actual de la conversación o con el último mensaje que envió el usuario. Este tipo de mensajes se llaman **mensajes proactivos**.

Los mensajes proactivos pueden ser útiles en diversos escenarios.
Si un bot establece un temporizador o un recordatorio, deberá notificar al usuario cuando llegue la hora.
O bien, si un bot recibe una notificación desde un evento externo, es posible que deba comunicar esa información al usuario inmediatamente.
Por ejemplo, si el usuario ha solicitado anteriormente al bot que supervise el precio de un producto, el bot puede alertar al usuario si el precio del producto ha descendido un 20 %. O bien, si un bot necesita algo de tiempo para compilar una respuesta a la pregunta del usuario, puede informar al usuario del retraso y permitir que la conversación continúe mientras tanto. Cuando el bot termine de compilar la respuesta a la pregunta, compartirá esta información con el usuario.

Al implementar mensajes proactivos en el bot:

- *No* envíe varios mensajes proactivos dentro de un intervalo corto de tiempo. Algunos canales imponen restricciones sobre la frecuencia con que un bot puede enviar mensajes al usuario, y deshabilitarán el bot si se infringen tales restricciones.
- *No* envíe mensajes proactivos a usuarios que no hayan interactuado anteriormente con el bot o que no hayan solicitado contacto con el bot por otros medios como correo electrónico o SMS.

Los mensajes proactivos pueden producir un comportamiento inesperado. Considere el siguiente escenario:

![cómo hablan los usuarios](~/media/designing-bots/capabilities/proactive1.png)

En este ejemplo, el usuario ha solicitado anteriormente al bot que supervise los precios de un hotel en Las Vegas.
El bot inicia una tarea de supervisión en segundo plano, que se ha ejecutado de manera continuada durante los últimos días.
En la conversación, el usuario está reservando actualmente un viaje a Londres cuando la tarea en segundo plano desencadena un mensaje de notificación sobre un descuento para un hotel de Las Vegas. El bot introduce esta información en la conversación, lo que crea una experiencia de usuario confusa.

¿Cómo debería tratar el bot esta situación?

- Esperar a que finalice la reserva de viaje actual y luego entregar la notificación. Este enfoque sería muy poco perjudicial, pero el retraso en comunicar la información podría hacer que el usuario perdiera la oportunidad de un precio bajo por el hotel de Las Vegas.
- Cancelar el flujo de la reserva de viaje actual y entregar la notificación inmediatamente. Este enfoque entrega la información de manera puntual, pero podría ser frustrante para el usuario al obligarle a empezar con su reserva de viaje.
- Interrumpir la reserva actual, cambiar claramente el tema de conversación al hotel de Las Vegas hasta que el usuario responda y luego volver a la reserva de viaje en curso y continuar desde donde se produjo la interrupción. Este enfoque puede parecer la mejor opción, pero introduce complejidad tanto para el desarrollador del bot como para el usuario.

Más comúnmente, el bot usará alguna combinación de **mensajes proactivos ad hoc** y otras técnicas para tratar situaciones como esta.

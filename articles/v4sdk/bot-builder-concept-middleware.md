---
title: Software intermedio | Microsoft Docs
description: Descripción del software intermedio y de sus usos en el SDK del bot.
keywords: software intermedio, canalización de software intermedio, cortocircuito, usos del software intermedio
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/8/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 713a53947a8ea6681f1793f9796a86c6d8014e29
ms.sourcegitcommit: cb0b70d7cf1081b08eaf1fddb69f7db3b95b1b09
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/09/2018
ms.locfileid: "51332929"
---
# <a name="middleware"></a>Software intermedio

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

El software intermedio es simplemente una clase que se encuentra entre el adaptador y la lógica del bot, que se agrega a la colección de software intermedio del adaptador durante la inicialización. El SDK le permite escribir su propio middleware o agregar middleware creado por terceros. Todas las actividades que entran y salen de su bot pasan por el middleware.

El adaptador procesa y dirige las actividades entrantes a través de la canalización de software intermedio del bot a la lógica del bot y, luego, otra vez de vuelta. Cuando las actividades entran y salen de los bots, cada fragmento de software intermedio puede inspeccionar o actuar sobre la actividad, tanto antes como después de que se ejecute la lógica del bot.

Antes de centrarse en el software intermedio, es importante entender los [bots en general](~/v4sdk/bot-builder-basics.md) y [cómo procesan las actividades](~/v4sdk/bot-builder-basics.md#the-activity-processing-stack).

## <a name="uses-for-middleware"></a>Usos del software intermedio
A menudo surge la siguiente pregunta: "¿Cuándo debo implementar acciones como middleware, en lugar de usar la lógica normal del bot?" El middleware le brinda oportunidades adicionales para interactuar con el flujo de conversación de los usuarios tanto antes como después de que se procese cada _turno_ de la conversación. También permite almacenar y recuperar información relativa a la conversación y llamar a una lógica de procesamiento adicional cuando sea necesario. A continuación encontrará algunos escenarios comunes que muestran situaciones en las que el middleware puede resultar útil.

### <a name="looking-at-or-acting-on-every-activity"></a>Registrar o actuar sobre todas las actividades
Existen muchas situaciones que requieren que un bot haga algo en todas las actividades, o bien en las actividades de un tipo determinado. Por ejemplo, tal vez desee registrar todas las actividades de mensajes que recibe el bot, o proporcionar una respuesta de reserva si el bot no ha generado ninguna respuesta en este turno. El software intermedio es idóneo para hacerlo, ya que puede actuar tanto antes como después de que se haya ejecutado el resto de la lógica del bot.

### <a name="modifying-or-enhancing-the-turn-context"></a>Modificar o mejorar el contexto de turno
Algunas conversaciones pueden ser mucho más provechosas si el bot tiene más información de la que se proporciona en la actividad. En este caso, el middleware podría comprobar la información de estado de la conversación que se tiene hasta el momento, consultar un origen de datos externo y anexarlo al objeto del [contexto de turno](~/v4sdk/bot-builder-basics.md#defining-a-turn) antes de pasar la ejecución a la lógica del bot. 

El SDK define el middleware de registro que puede grabar las actividades de entrada y salida, pero el usuario también puede definir su propio middleware.

## <a name="the-bot-middleware-pipeline"></a>La canalización de software intermedio del bot
Para cada actividad, el adaptador llama al middleware en el orden en que se ha agregado. El adaptador pasa el objeto de contexto para el turno y un delegado _next_, y el software intermedio llama al delegado para pasar el control al siguiente software intermedio de la canalización. El software intermedio también puede llevar a cabo acciones después de que se devuelva el delegado _next_ antes de completar el método. Es como si cada objeto de software intermedio tuviese la oportunidad de actuar directamente sobre los objetos de software intermedio que lo siguen en la canalización.

Por ejemplo: 

- El primer controlador de turno del objeto de software intermedio ejecuta código antes de llamar a _next_.
  - El segundo controlador de turno del objeto de software intermedio ejecuta código antes de llamar a _next_.
    - El controlador de turno del bot se ejecuta y se devuelve.
  - El segundo controlador de turno del objeto de software intermedio ejecuta el código restante antes de devolver.
- El primer controlador de turno del objeto de software intermedio ejecuta el código restante antes de devolver.

Si el software intermedio no llama al delegado next, el adaptador no llama a ninguno de los controladores de turno posteriores de software intermedio o bot y se produce un cortocircuito en la canalización.

Una vez que se ha completado la canalización de software intermedio del bot, el turno acaba y el contexto de turno sale del ámbito.

El software intermedio o el bot pueden generar respuestas y registrar los controladores de eventos de respuesta, pero debe tener en cuenta que las respuestas se controlan en procesos independientes.

## <a name="order-of-middleware"></a>Orden del software intermedio
Puesto que el orden en el que se agrega el software intermedio determina el orden en el que este procesa una actividad, es importante decidir la secuencia que se seguirá para agregar dicho software intermedio.

> [!NOTE]
> Esto está pensado para proporcionarle un patrón común que funcione para la mayoría de los bots, pero debe tener en cuenta cómo interactuará cada fragmento de software intermedio con los demás según su situación.

Los primeros elementos de la canalización de software intermedio probablemente serán los que se encarguen de las tareas de nivel más bajo que se usan de cada vez, como el registro, el control de excepciones y la traducción. El orden de estos elementos puede variar en función de sus necesidades, por ejemplo, si quiere que el mensaje entrante se traduzca primero, antes de que se guarden los mensajes, o si primero se deben almacenar, lo que podría significar que los mensajes almacenados no se traducirían.

Los últimos elementos de la canalización de software intermedio deberían ser el software intermedio específico del bot, que es el que se implementa para realizar el procesamiento de cada mensaje que se envía al bot. Si el software intermedio usa información de estado u otra información definida en el contexto del bot, agréguela a la canalización de software intermedio después del software intermedio que modifica el estado o el contexto.

## <a name="short-circuiting"></a>Cortocircuitos
Una idea importante relacionada con el middleware y los controladores de respuesta es el _cortocircuito_. Si la ejecución va a continuar por las capas siguientes, el middleware (o un controlador de respuestas) debe pasar la ejecución mediante una llamada al delegado _next_.  Si no se llama al delegado next en dicho middleware (o controlador de respuestas), se producirá un cortocircuito en la canalización asociada y las capas posteriores no se ejecutarán, lo que significa que se omiten no solo toda la lógica del bot, sino también el middleware posterior de la canalización. Hay una sutil diferencia entre que el middleware cortocircuite un turno y que lo haga el controlador de respuestas.

Si el middleware cortocircuita un turno, no se llama al controlador de turnos del bot, pero todo el código del middleware que se haya ejecutado antes de ese momento en la canalización se seguirá ejecutando hasta su finalización. 

En el caso de los controladores de eventos, si no se llama a _next_ el evento se cancela, lo que supone un resultado muy diferente del que se obtiene cuando el software intermedio omite la lógica. Al no procesar el resto del evento, el adaptador no lo envía nunca.

> [!TIP]
> Si provoca un cortocircuito en un evento de respuestas, como `SendActivities`, asegúrese de que es el comportamiento que tenía previsto. De lo contrario, puede resultar muy difícil solucionar los errores.

## <a name="response-event-handlers"></a>Controladores de eventos de respuesta
Además de la lógica de la aplicación y del middleware, se pueden agregar controladores de respuesta (a veces llamados controladores de eventos o controladores de eventos de actividad) al objeto de contexto. Se llama a estos controladores cuando la respuesta asociada se produce en el objeto de contexto actual, antes de ejecutar la respuesta real. Estos controladores son útiles si sabe que querrá hacer algo, ya sea antes o después del evento real, para todas las actividades de ese tipo durante el resto de la respuesta actual.

> [!WARNING] 
> Tenga cuidado de no llamar a un método de respuesta de actividad desde su controlador de eventos de respuesta correspondiente, por ejemplo, mediante una llamada al método de actividad de envío desde un controlador de actividad de envío. Si lo hace, puede generar un bucle infinito.

Recuerde que cada actividad nueva obtiene un nuevo subproceso en el que se ejecuta. Cuando se crea el subproceso para procesar la actividad, la lista de controladores de esa actividad se copia en ese subproceso nuevo. No se ejecutará para ese evento de actividad específico ningún controlador agregado después de ese punto.
El adaptador administra los controladores registrados en un objeto de contexto de forma muy similar al modo en que administra la canalización de middleware. Es decir, se llama a los controladores en el orden en que se agregan y al llamar al delegado next se pasa el control al siguiente controlador de eventos registrado. Si un controlador no llama al delegado next, no se llama a ninguno de los controladores de eventos posteriores; se produce un cortocircuito en el evento y el adaptador no envía la respuesta al canal.

## <a name="additional-resources"></a>Recursos adicionales
Puede echar un vistazo al middleware del registrador de transcripciones, tal como está implementado en el SDK Bot Builder [[C#](https://github.com/Microsoft/botbuilder-dotnet/blob/master/libraries/Microsoft.Bot.Builder/TranscriptLoggerMiddleware.cs) | [JS](https://github.com/Microsoft/botbuilder-js/blob/master/libraries/botbuilder-core/src/transcriptLogger.ts)].

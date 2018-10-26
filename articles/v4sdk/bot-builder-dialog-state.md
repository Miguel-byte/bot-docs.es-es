---
title: Estado del diálogo | Microsoft Docs
description: Describe cómo funciona el estado en el SDK de Bot Builder.
keywords: estado, estado del bot, estado de la conversación, estado de usuario
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 9/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0fb42643b195f55c8fae73f41b622377f8da2a5c
ms.sourcegitcommit: d4afc924b0e1907c4d6f7a6fc5ac1fe521aeef7e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/28/2018
ms.locfileid: "47447330"
---
# <a name="dialog-state"></a>Estado del diálogo

Los diálogos son un método de implementación de una lógica de conversación con varios turnos y, como tal, son un ejemplo de una característica del SDK que se basa en un estado persistente. 

Un bot basado en diálogos contiene normalmente una colección DialogSet como una variable de miembro en su implementación. DialogSet se crea con un identificador a un objeto que se llama Descriptor de acceso. 

El descriptor de acceso es el concepto central en el modelo de estados del SDK. Un descriptor de acceso implementa la interfaz IStatePropertyAccessor, lo cual significa que debe proporcionar implementaciones para los métodos Get, Set y Delete. Para crear un descriptor de acceso, proporciónele un nombre de propiedad. 

El código que utiliza el descriptor de acceso puede llamar a los métodos Get, Set y Delete sabiendo que estos harán referencia a una propiedad de ese nombre. Si un componente de nivel superior requiere alguna persistencia de estado, deberá utilizar el descriptor de acceso. De esta forma, diferentes escenarios pueden aportar sus implementaciones del almacenamiento de estados pero, no obstante, seguirán usando el mismo código de nivel superior. Por ejemplo, los diálogos usan el descriptor de acceso y esa es la única forma que tienen de acceder al estado persistente.

![estado del diálogo](media/bot-builder-dialog-state.png)

Cuando se llama a OnTurn del bot, este inicializará el subsistema de diálogos llamando a CreateContext en DialogSet. La creación de un DialogContext requiere un estado, por lo que DialogSet usará su descriptor de acceso para obtener el JSON con el estado de diálogo adecuado. El SDK proporciona una implementación del descriptor de acceso con la forma de la clase BotState. Las aplicaciones que quieran aprovechar la implementación de estados pueden subclasificar BotState y heredar así una implementación del descriptor de acceso. Hay dos subclases de BotState incluidas en el SDK:

- UserState
- ConversationState

UserState y ConversationState usan claves para el almacenamiento subyacente. La clave se pasa al almacenamiento físico. Lógicamente, la clave actúa como un espacio de nombres para la propiedad que el descriptor de acceso nombre. La implementación de BotState usa internamente la actividad de entrada de TurnContext para generar dinámicamente la clave de almacenamiento que usa.

- UserState crea una clave con el identificador de canal y el identificador de procedencia. Por ejemplo, _{Activity.ChannelId}/conversations/{Activity.From.Id}#DialogState_
- ConversationState crea una clave con el identificador de canal y el identificador de conversación. Por ejemplo, _{Activity.ChannelId}/users/{Activity.Conversation.Id}#YourPropertyName_

Una aplicación debe proporcionar un descriptor de acceso y el enlace al nombre de la clave y la propiedad de almacenamiento adecuadas creadas dinámicamente se producirá en segundo plano. La implementación de BotState del descriptor de acceso incluye algunas optimizaciones: 

- La primera optimización es una carga almacenada en caché y diferida. Una carga de la implementación de almacenamiento real se aplazará hasta que se llame al primer método Get del descriptor de acceso, y el resultado de esa carga se almacenará en una memoria caché. Una referencia a esta caché se agrega a TurnContext, mediante una clave que proporciona el BotState correspondiente. Por tanto, la memoria caché correspondiente a UserState se guardará en un campo denominado "UserState" y la memoria caché correspondiente a ConversationState se guardará en un campo denominado "ConversationState". Las llamadas a los distintos objetos del descriptor de acceso se pasan a TurnContext y, de esta forma, pueden ir y recuperar la memoria caché adecuada.

- La segunda optimización consiste en que la clase BotState contiene lógica para determinar si se ha realizado algún cambio en el estado. Solo en caso de que se hayan realizado cambios se ejecutará una operación real para guardar en el almacenamiento subyacente.

- La tercera optimización dentro de la implementación de BotState está en una clase denominada BotStateSet. BotStateSet es una colección de objetos de BotState que delega una llamada en su método SaveChanges para cada miembro de la colección en paralelo.

Es significativo que la llamada de SaveChanges en BotState no forma parte de la interfaz IStatePropertyAccessor. El motivo es que SaveChanges es una optimización particular de la implementación de BotState en lugar de un aspecto fundamental del modelo. En concreto, el código como el de Diálogos no conoce a BotState y mucho menos a SaveChanges. De hecho, el código de Diálogos solo se acopla al descriptor de acceso. La intención es que se debe llamar al método SaveChanges desde fuera del sistema de diálogos una vez que se haya completado su ejecución. Por ejemplo, como se muestra en el diagrama, se le puede llamar desde dentro del método OnTurn del bot.

Es importante tener en cuenta que la implementación de BotState trae consigo una semántica específica. En concreto, admite un comportamiento del tipo "la última escritura prevalece" por el cual la última escritura sobrescribirá al estado escrito anteriormente. Esto puede funcionar bien para muchas aplicaciones, pero tiene otras implicaciones, especialmente en escenarios de escalado horizontal en los que puede haber un cierto nivel de simultaneidad en juego. Si esto supone un problema, la solución es implementar su propio descriptor de acceso y pasarlo al diálogo. Hay muchos enfoques alternativos. Por ejemplo, una solución puede aprovechar la condición eTag que resulta conocida en servicios de almacenamiento en nube como Azure Storage. En este caso, la solución implementaría probablemente otros elementos importantes. Por ejemplo, podría almacenar en búfer actividades salientes y enviarlas únicamente tras una operación Save correcta. Un punto importante a tener en cuenta es que este comportamiento no es el comportamiento de la implementación de BotState sino algo que una aplicación podría proporcionar e incorporar en el nivel del descriptor de acceso.

La propia implementación de BotState tiene un modelo de almacenamiento conectable. Este sigue un patrón sencillo Cargar o Guardar y el SDK proporciona dos implementaciones alternativas para producción. Una para Azure Storage y otra para CosmosDB, y una implementación en memoria con fines de prueba. Lo importante a tener en cuenta aquí, sin embargo, es que la semántica del tipo "la última escritura prevalece" viene dictada por la implementación de BotState.

## <a name="handling-state-in-middleware"></a>Control del estado en el middleware
El diagrama anterior muestra una llamada a SaveAllChanges que ocurre al final del método OnTurn del bot. Aquí está ese mismo diagrama con especial atención a la llamada.

![problemas de estado de middleware](media/bot-builder-dialog-state-problem.png)

El problema con este enfoque es que las actualizaciones de estado realizadas por parte de algún middleware personalizado que tienen lugar una vez que vuelve el método OnTurn del bot, no se guardarán en un almacenamiento duradero. La solución consiste en mover la llamada a SaveAllChanges una vez que el middleware personalizado haya finalizado con la incorporación de AutoSaveChangesMiddleware al final de la pila de middleware. A continuación se muestra la ejecución.

![solución de estados de middleware](media/bot-builder-dialog-state-solution.png)

## <a name="additional-resources"></a>Recursos adicionales
Para más información, consulte el SDK de Bot Builder en GitHub [[C#](https://github.com/Microsoft/BotBuilder-dotnet) | [JavaScript](https://github.com/Microsoft/BotBuilder-js)].

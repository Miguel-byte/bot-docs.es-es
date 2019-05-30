---
title: Administración de estados | Microsoft Docs
description: Describe cómo funciona el estado en Bot Framework SDK.
keywords: estado, estado del bot, estado de la conversación, estado de usuario
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 037331b10719a47fba42485b19d5ecc9dab08cbe
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215482"
---
# <a name="managing-state"></a>Administración de estados

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

El estado dentro de un bot sigue los mismos paradigmas que las aplicaciones web modernas, y Bot Framework SDK proporciona algunas abstracciones para facilitar la administración de estados.

Al igual que con las aplicaciones web, un bot intrínsecamente carece de estado; una instancia diferente del puede controlar cualquier turno determinado de la conversación. Para algunos bots, se prefiere esta simplicidad, pues el bot puede funcionar sin información adicional, o la información necesaria se garantiza dentro del mensaje entrante. Para otros, el estado (por ejemplo, en qué parte de la conversación estamos o hemos recibido datos sobre el usuario) es necesario para que el bot mantenga una conversación útil.

**¿Por qué necesito el estado?**

Mantener el estado permite que el bot mantenga conversaciones más significativas al recordar ciertas cosas sobre un usuario o una conversación. Por ejemplo, si ha hablado con un usuario anteriormente, puede guardar información previa sobre él, para no tener que volver a pedirla. El estado también mantiene los datos durante más tiempo que el turno actual, de modo que el bot conserva la información en el transcurso de una conversación de varios turnos.

En lo que respecta a los bots, hay algunas capas de uso del estado que se verán aquí: la capa de almacenamiento, la administración de estados (se encuentra en el estado del bot del diagrama siguiente) y los descriptores de acceso de la propiedad de estado. Este diagrama muestra las partes de la secuencia de interacción entre estas capas. Las flechas de línea continua representan una llamada al método, mientras que las flechas de línea discontinua representan la respuesta (con o sin un valor devuelto).

![estado del bot](media/bot-builder-state.png)

El flujo de este diagrama se explica en las siguientes secciones y se incluyen detalles de cada capa.

## <a name="storage-layer"></a>Capa de almacenamiento

Comenzando por el back-end, donde se almacena la información de estado, se encuentra la *capa de almacenamiento*. Se puede considerar como el almacenamiento físico, como el almacenamiento en memoria, en Azure, o en un servidor de terceros.

Bot Framework SDK incluye algunas implementaciones de la capa de almacenamiento:

- **Almacenamiento en memoria** implementa el almacenamiento en memoria con fines de prueba. El almacenamiento de datos en memoria está destinado únicamente a pruebas locales, ya que este almacenamiento es volátil y temporal. Los datos se borran cada vez que se reinicia el bot.
- **Azure Blob Storage** se conecta a una base de datos de objetos Azure Blob Storage.
- **El almacenamiento de Azure Cosmos DB**  se conecta a una base de datos Cosmos DB NoSQL.

Para obtener instrucciones sobre cómo conectarse a otras opciones de almacenamiento, consulte [Escritura directa en el almacenamiento](bot-builder-howto-v4-storage.md).

## <a name="state-management"></a>Administración de estados

La *administración de estados* automatiza la lectura y escritura de estado del bot a la capa de almacenamiento subyacente. El estado se almacena como *propiedades de estado*, que son pares clave-valor que el bot puede leer y escribir mediante el objeto de administración de estados sin preocuparse por la implementación subyacente específica. Las propiedades de estado definen cómo se almacena esa información. Por ejemplo, cuando se recupera una propiedad que se ha definido como una clase u objeto específico, se sabe cómo se estructurarán los datos.

Estas propiedades de estado se agrupan en el ámbito de "cubos", que son solo colecciones para ayudar a organizar esas propiedades. El SDK incluye tres de estos "cubos":

- Estado de usuario
- Estado de conversación
- Estado de conversación privada

Todos estos cubos son subclases de la clase *estado de bot*, que se puede derivar para definir otros tipos de cubos con ámbitos diferentes.

Estos cubos predefinidos tienen una cierta visibilidad, dependiendo del cubo:

- El estado de usuario está disponible en cualquier momento que el bot esté conversando con ese usuario en ese canal, independientemente de la conversación.
- El estado de conversación está disponible en cualquier momento de una conversación específica, independientemente del usuario (es decir, conversaciones de grupo).
- El estado de conversación privada se extiende tanto a la conversación específica como a ese usuario específico.

> [!TIP]
> Tanto el estado del usuario como el de la conversación están delimitados por canal.
> La misma persona que utiliza diferentes canales para acceder al bot aparece como usuarios diferentes, uno para cada canal y cada uno con un estado de usuario distinto.

Las claves utilizadas para cada uno de estos cubos predefinidos son específicas para el usuario y la conversación, o para ambos. Al establecer el valor de su propiedad de estado, la clave se define internamente con información contenida en el contexto del turno para asegurar que cada usuario o conversación se coloca en el cubo y la propiedad correctos. En concreto, las claves se definen de la manera siguiente:

- El estado de usuario crea una clave con el *identificador de canal* y el *identificador de procedencia*. Por ejemplo, _{Activity.ChannelId}/users/{Activity.From.Id}#YourPropertyName_
- El estado de conversación crea una clave con el *identificador de canal* y el *identificador de conversación*. Por ejemplo, _{Activity.ChannelId}/conversations/{Activity.Conversation.Id}#YourPropertyName_
- El estado de conversación privada crea una clave con el *identificador de canal*, el *identificador de procedencia* y el *identificador de conversación*. Por ejemplo, _{Activity.ChannelId}/conversations/{Activity.Conversation.Id}/users/{Activity.From.Id}#YourPropertyName_

### <a name="when-to-use-each-type-of-state"></a>Cuándo usar cada tipo de estado

El estado de conversación es bueno para realizar el seguimiento del contexto de la conversación, como por ejemplo:

- Si el bot le formula una pregunta al usuario, y cuál fue la pregunta
- Cuál es el tema actual de conversación o cuál fue el último

El estado de usuario es bueno para realizar el seguimiento de la información sobre el usuario, como por ejemplo:

- Información de usuario no crítica, como el nombre y las preferencias, una configuración de alarma o una preferencia de alerta
- Información sobre la última conversación que tuvieron con el bot
  - Por ejemplo, un bot de soporte técnico de productos puede realizar el seguimiento de los productos sobre los que el usuario ha preguntado.

El estado de conversación privada es bueno para los canales que admiten conversaciones de grupo, pero en los que se desea realizar un seguimiento de la información específica del usuario y de la conversación. Por ejemplo, si tiene un bot de sistema de respuesta en el aula:

- El bot puede agregar y mostrar las respuestas de los alumnos para una pregunta dada.
- El bot puede agregar el rendimiento de cada alumno y transmitirlo en privado al final de la sesión.

Para más información sobre el uso de estos cubos predefinidos, consulte el [artículo de procedimientos de estado](bot-builder-howto-v4-state.md).

### <a name="connecting-to-multiple-databases"></a>Conexión a varias bases de datos

Si el bot necesita conectarse a varias bases de datos, cree una capa de almacenamiento para cada base de datos.
Para cada capa de almacenamiento, cree los objetos de administración de estado que necesita para admitir las propiedades de estado.

## <a name="state-property-accessors"></a>Descriptores de acceso de propiedad de estado

*Los descriptores de acceso de la propiedad de estado* se usan para leer o escribir una de las propiedades de estado y proporcionan los métodos para *obtener*, *establecer* y *eliminar*  con el fin de acceder a las propiedades de estado desde dentro de un turno. Para crear un descriptor de acceso, debe proporcionar el nombre de la propiedad, que normalmente tiene lugar cuando se inicializa el bot. Entonces, puede usar ese descriptor de acceso para obtener y manipular esa propiedad de estado del bot.

Los descriptores de acceso permiten que el SDK obtenga el estado del almacenamiento subyacente y actualice la *caché de estado* del bot para el usuario. La caché de estado es una caché local mantenida por el bot que almacena el objeto de estado por el usuario, lo que permite operaciones de lectura y escritura sin acceder al almacenamiento subyacente. Si aún no está en la memoria caché, una llamada al método para *obtener* el descriptor de acceso recupera el estado y también lo coloca en la memoria caché. Una vez recuperada, la propiedad de estado puede manipularse como una variable local.

El método para *eliminar*  el descriptor de acceso quita la propiedad de la memoria caché y también la elimina del almacenamiento subyacente.

> [!IMPORTANT]
> Para la primera llamada a un método para *obtener* el descriptor de acceso, debe proporcionar un método de generador para crear el objeto si aún no existe en el estado. Si no se especifica ningún método de generador, obtendrá una excepción. Encontrará más información sobre cómo usar un método de generador en el [artículo de procedimientos de estado](bot-builder-howto-v4-state.md).

Para conservar los cambios que realice en la propiedad de estado que obtiene del descriptor de acceso, la propiedad de la caché de estado debe actualizarse. Puede hacerlo mediante una llamada al método para *establecer* los descriptores de acceso, que establece el valor de la propiedad en la caché y está disponible si necesita leerlo o actualizarlo más tarde en ese turno. Para que esos datos permanezcan realmente en el almacenamiento subyacente (y por lo tanto estén disponibles después del turno actual), debe entonces [guardar su estado](#saving-state).

### <a name="how-the-state-property-accessor-methods-work"></a>Funcionamiento de los métodos de descriptor de acceso de propiedades de estado

Los métodos de descriptor de acceso son la manera principal para el bot interactúe con el estado. La forma como trabajan y cómo interactúan las capas subyacentes son las siguientes:

- Método para *obtener* el descriptor de acceso:
  - El descriptor de acceso solicita la propiedad desde la caché de estado.
  - Si la propiedad está en la caché, la devuelve. De lo contrario, obténgalo del objeto de administración de estados.
    (Si todavía no está en el estado, utilice patrón de diseño Factory Method proporcionado en la llamada para *obtener* los descriptores de acceso).
- Método para *establecer* el descriptor de acceso:
  - Actualice la caché de estado con el nuevo valor de propiedad.
- Método para *guardar cambios* del objeto de administración de estados:
  - Compruebe los cambios de la propiedad en la caché de estado.
  - Escriba esa propiedad en el almacenamiento.

## <a name="saving-state"></a>Almacenamiento del estado

Cuando llama al método para establecer el descriptor de acceso para registrar el estado actualizado, esa propiedad de estado todavía no se ha guardado en el almacenamiento persistente y, en su lugar, solo se guarda en la caché de estado del bot. Para guardar cualquier cambio en la caché de estado en el estado persistente, debe llamar al método para *guardar cambios* del objeto de administración de estados, que está disponible en la implementación de la clase de estado de bot mencionada anteriormente (como el estado de usuario o el estado de conversación).

La llamada al método para guardar cambios para un objeto de administración de estados (es decir, los cubos mencionados anteriormente) guarda todas las propiedades en la caché de estado que ha configurado hasta ese momento para ese cubo, pero no para ninguno de los otros cubos que pueda tener en el estado de bot.

> [!TIP]
> El estado del bot implementa un comportamiento del tipo "la última escritura prevalece", por el cual la última escritura sobrescribirá al estado escrito anteriormente. Esto puede funcionar para muchas aplicaciones, pero tiene otras implicaciones, especialmente en escenarios de escalado horizontal en los que puede haber un cierto nivel de simultaneidad o latencia en juego.

Si tiene algún middleware personalizado que pueda actualizar el estado después de que su controlador de turno se complete, considere [controlar el estado en el middleware](bot-builder-concept-middleware.md#handling-state-in-middleware).

## <a name="additional-resources"></a>Recursos adicionales

- [Estado del diálogo](bot-builder-concept-dialog.md#dialog-state)
- [Escritura directa en el almacenamiento](bot-builder-howto-v4-storage.md)
- [Guardado de los datos de usuario y la conversación](bot-builder-howto-v4-state.md)

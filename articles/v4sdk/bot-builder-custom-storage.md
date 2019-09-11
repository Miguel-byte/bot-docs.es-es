---
title: Implementación de almacenamiento personalizado en un bot | Microsoft Docs
description: Creación de almacenamiento personalizado en Bot Framework SDK v4.0
keywords: custom, storage, state, dialog
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7197cf4716369b00a8ccdff0f0e289bd3a0fdd16
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299382"
---
# <a name="implement-custom-storage-for-your-bot"></a>Implementación de almacenamiento personalizado en un bot

[!INCLUDE[applies-to](../includes/applies-to.md)]

Las interacciones de un bot se dividen en tres áreas: en primer lugar, el intercambio de actividades con Azure Bot Service, en segundo lugar, la carga y almacenamiento de estados de diálogos con Store y, por último, cualquier otro servicio back-end que el bot necesite para realizar su trabajo.

![diagrama de escalado horizontal](../media/scale-out/scale-out-interaction.png)


## <a name="prerequisites"></a>Requisitos previos
- El código de ejemplo completo que se usa en este artículo puede encontrarse aquí: [Ejemplo en C#](http://aka.ms/scale-out).

En este artículo, vamos a analizar la semántica en torno a las interacciones del bot con Azure Bot Service y con Store.

Bot Framework incluye una implementación predeterminada. Muy probablemente, esta implementación se adaptará a las necesidades de muchas aplicaciones y todo lo que tendrá que hacer para usarla es conectar todas las piezas con unas pocas líneas de código de inicialización. Muchos de los ejemplos muestran exactamente eso.

El objetivo en este caso, sin embargo, es describir lo que puede hacer cuando la semántica de la implementación predeterminada no funciona como le gustaría en la aplicación. Lo principal es que esta es una plataforma y no una aplicación rígida con un comportamiento fijo. En otras palabras, la implementación de muchos de los mecanismos de la plataforma es solo la implementación predeterminada, pero no la única implementación posible.

Más concretamente, la plataforma no dicta la relación entre el intercambio de actividades con Azure Bot Service y la carga y almacenamiento de cualquier estado del bot, solo proporciona una opción predeterminada. Para ilustrar este punto aún más, vamos a desarrollar una implementación alternativa con una semántica diferente. Esta solución alternativa se adapta igualmente bien a la plataforma y puede incluso ser más adecuada para la aplicación que se está desarrollando. Todo depende del escenario.

## <a name="behavior-of-the-default-botframeworkadapter-and-storage-providers"></a>Comportamiento de los proveedores predeterminados de BotFrameworkAdapter y Storage

En primer lugar, vamos a revisar la implementación predeterminada que se incluye como parte de los paquetes de la plataforma tal y como se muestra en el siguiente diagrama de secuencia:

![diagrama de escalado horizontal](../media/scale-out/scale-out-default.png)

Al recibir una actividad, el bot carga el estado correspondiente a esta conversación. A continuación, ejecuta la lógica del diálogo con ese estado y la actividad que acaba de llegar. Durante la ejecución del diálogo, se crearán una o varias actividades salientes y se enviarán inmediatamente. Una vez que el procesamiento del diálogo está completo, el bot guarda el estado actualizado y sobrescribe el anterior estado con el nuevo.

Merece la pena tener en cuenta un par de cosas que pueden salir mal con este comportamiento.

En primer lugar, si se ha producido un error en la operación de guardar por algún motivo, el estado pierde la sincronización con lo que se ve en el canal. El usuario que vea las respuestas tendrá la impresión de que el estado ha cambiado, pero realmente no es así. Esto resulta, por lo general, peor que si el estado fue correcto al igual que los mensajes de respuesta. Esto puede tener implicaciones en el diseño de la conversación: por ejemplo, el diálogo podría incluir intercambios de confirmación adicionales y redundantes con el usuario. 

En segundo lugar, si la implementación se escala horizontalmente entre varios nodos, puede que el estado se sobrescriba accidentalmente, lo cual puede resultar especialmente confuso ya que el diálogo habrá enviado probablemente actividades al canal que incluyen mensajes de confirmación. Considere el ejemplo de un bot para pedir pizza. Si el usuario, cuando se le pregunta por un ingrediente agrega champiñones e inmediatamente después agrega queso, en un escenario de escalabilidad horizontal con varias instancias ejecutando actividades subyacentes, este se puede enviar simultáneamente a las distintas máquinas que ejecutan el bot. Cuando esto sucede, se produce lo que se conoce como "condición de carrera" por la cual una máquina podría sobrescribir el estado que ha escrito otra. Sin embargo, en nuestro escenario, dado que ya se han enviado las respuestas, el usuario ha recibido confirmación de que se agregaron los champiñones y el queso. Lamentablemente, cuando llega la pizza, solo llevará champiñones o queso, pero no ambos.

## <a name="optimistic-locking"></a>Bloqueo optimista

La solución consiste en introducir algún bloqueo para proteger el estado. El estilo concreto de bloqueo que se va a emplear aquí se llama bloqueo optimista ya que va a dejar que todo se ejecute como si fuera el único proceso en ejecución y, posteriormente, se detectarán todas las infracciones de simultaneidad una vez que el proceso se haya completado. Esto puede parecer complicado pero es muy fácil de compilar mediante tecnologías de almacenamiento en la nube y los puntos de extensión adecuados en la plataforma del bot.

Vamos a usar un mecanismo estándar de HTTP basado en el encabezado de etiqueta de entidad (ETag). Entender este mecanismo resulta crucial para comprender el código que sigue. En el siguiente diagrama se ilustra la secuencia.

![diagrama de escalado horizontal](../media/scale-out/scale-out-precondition-failed.png)

El diagrama ilustra el caso de dos clientes que están llevando a cabo una actualización en algún recurso. Cuando un cliente emite una solicitud GET y se devuelve un recurso desde el servidor, este viene acompañado de un encabezado ETag. El encabezado ETag es un valor opaco que representa el estado del recurso. Si se cambia un recurso, se actualizará el encabezado ETag. Una vez que el cliente haya hecho la actualización del estado, se vuelve a publicar en el servidor. Al hacer esta solicitud, el cliente asocia el valor ETag que ha recibido anteriormente a un encabezado If-Match de condición previa. Si este valor de ETag no coincide con el último valor que devolvió el servidor (en cualquier respuesta, para cualquier cliente), se producirá un error 412 de condición previa en la comprobación. Este error es un indicador para el cliente que realiza la solicitud POST de que el recurso se ha actualizado. Al ver este error, el comportamiento normal de un cliente será volver a obtener el recurso, aplicar la actualización que deseaba y publicar el recurso de nuevo. Esta segunda solicitud POST será correcta suponiendo, naturalmente, que ningún otro cliente haya llegado y actualizado el recurso y, en ese caso, el cliente solo tendría que intentarlo de nuevo.

Este proceso se denomina "optimista" porque el cliente, habiendo obtenido un recurso pasa a realizar su procesamiento ya que el recurso en sí no está "bloqueado" en el sentido de que otros usuarios pueden acceder a él sin ninguna restricción. Cualquier contención entre los clientes sobre cuál debe ser el estado del recurso no se determina hasta que se ha realizado el procesamiento. Como norma, en un sistema distribuido, esta estrategia es mejor que el enfoque opuesto, el "pesimista".

El mecanismo de bloqueo optimista que hemos explicado aquí supone que la lógica del programa se puede volver a intentar con seguridad. No es necesario decir que lo importante aquí es considerar qué sucede a las llamadas de servicio externas. En este caso, la solución ideal es que se pueda hacer que estos servicios sean idempotentes. En informática, una operación idempotente es aquella que no tiene ningún efecto adicional si se la llama varias veces con los mismos parámetros de entrada. Los servicios HTTP REST puros que implementan las operaciones GET, PUT y DELETE se ajustan a esta descripción. El razonamiento aquí es intuitivo: podríamos volver a intentar el procesamiento y el hacer todas las llamadas necesarias no tendría ningún efecto adicional ya que ellas se vuelven a ejecutar como parte de ese reintento. Por lo que respecta a este análisis, vamos a suponer que se trata de un escenario ideal y que los servicios de back-end que aparecen a la derecha de la imagen del sistema al principio de este artículo son todos servicios HTTP REST idempotentes. A partir de aquí solo nos centraremos en el intercambio de actividades.

## <a name="buffering-outbound-activities"></a>Almacenamiento en búfer de actividades salientes

El envío de una actividad no es una operación idempotente, ni está claro que tenga mucho sentido en un escenario de un extremo a otro. Después de todo, la actividad a menudo solo transmite un mensaje que se anexa a una vista o que quizás ha pronunciado un agente de texto a voz.

Lo más importante que queremos evitar en el envío de actividades es enviarlas varias veces. El problema es que el mecanismo de bloqueo optimista requerirá posiblemente que se vuelva a ejecutar nuestra lógica varias veces. La solución es sencilla: debemos almacenar en búfer las actividades salientes del diálogo hasta que estemos seguros de que no vamos a volver a ejecutar la lógica. Y eso solo se producirá cuando hayamos completado una operación de guardar correctamente. Buscamos un flujo parecido al siguiente:

![diagrama de escalado horizontal](../media/scale-out/scale-out-buffer.png)

Suponiendo que podamos crear un bucle de reintento en torno a la ejecución del diálogo, obtendremos el siguiente comportamiento si se produce un error de condición previa en la operación de guardar:

![diagrama de escalado horizontal](../media/scale-out/scale-out-save.png)

Si aplicamos este mecanismo y repasamos el ejemplo desde el principio no veremos nunca una confirmación positiva errónea de un ingrediente de pizza agregado a un pedido. De hecho, aunque podríamos haber escalado horizontalmente nuestra implementación en varias máquinas, hemos serializado eficazmente las actualizaciones de estado con el esquema de bloqueo optimista. En el ejemplo de pedido de pizza, la confirmación después de agregar un elemento se puede escribir ahora para reflejar el estado completo correctamente. Por ejemplo, si el usuario escribe inmediatamente "queso" y, a continuación, antes de que el bot haya tenido ocasión de responder "champiñones", las dos respuestas ahora pueden ser "pizza con queso" y, después, "pizza con queso y champiñones".

Si observamos el diagrama de secuencia, podemos ver que las respuestas se podrían perder después de una operación de guardar correcta. No obstante, también se podrían perder en cualquier punto de todo el proceso de comunicación. La cuestión en este caso es que no se trata de un problema que la infraestructura de administración de estados pueda solucionar. Requerirá un protocolo de nivel superior y, posiblemente, uno que incluya al usuario del canal. Por ejemplo, si el usuario considera que el bot no ha respondido, es razonable esperar que el usuario lo volverá intentar de nuevo o esperar un comportamiento similar. Por tanto, aunque podría ser razonable que un escenario como este tenga interrupciones transitorias, es mucho menos razonable esperar que un usuario pueda filtrar confirmaciones positivas erróneas o cualquier otro tipo de mensajes no intencionados. 

En resumen, en nuestra nueva solución de almacenamiento personalizada, vamos a hacer tres cosas que la implementación personalizada de la plataforma no puede hacer. En primer lugar, vamos a usar ETags para detectar la contención. En segundo lugar, vamos a volver a intentar el procesamiento si se detecta un error de ETag y, por último, vamos a almacenar en búfer todas las actividades salientes hasta que se produzca una operación de guardar correcta. El resto de este artículo describe la implementación de estas tres partes.

## <a name="implementing-etag-support"></a>Implementación de la compatibilidad con ETag

Para admitir las pruebas unitarias vamos a empezar por definir una interfaz para nuestra nueva tienda con compatibilidad para ETag. Tener la interfaz significa que podemos escribir dos versiones: una para las pruebas unitarias, que se ejecuta en la memoria sin necesidad de que afecte a la red, y otra para producción. La interfaz hará que sea mucho más fácil aprovechar los mecanismos de inyección de dependencia de que disponemos en ASP.NET.

La interfaz consta de los métodos Load y Save. Ambos emplean la clave que usaremos para el estado. El método Load devolverá los datos y la ETag asociada. Y el método Save los guardará. Además, el método Save devolverá un valor booleano. Este valor booleano indicará si la ETag coincide y si el método Save se realizó correctamente. No se pretende usar esto como un indicador general de error si no, más bien, como un indicador específico de error de condición previa. Vamos a modelar esto como un código de devolución en lugar de como una excepción ya que escribiremos la lógica del flujo de control con la forma de nuestro bucle de reintentos.

Como nos gustaría que el nivel inferior de este elemento de almacenamiento fuera conectable, nos aseguraremos de evitar incluir requisitos de serialización en él. No obstante, nos gustaría especificar que el contenido guardado debe ser JSON ya que, de esa forma, el tipo de contenido puede establecerse durante la implementación del almacén. La manera más fácil y natural de hacer esto en .NET es a través de tipos de argumento y, más concretamente, vamos a escribir el argumento del contenido como JObject. En JavaScript o TypeScript esto será un objeto nativo normal.  

Esta es la interfaz resultante:

**IStore.cs**  
[!code-csharp[IStore](~/../botbuilder-samples/samples/csharp_dotnetcore/42.scaleout/IStore.cs?range=14-19)]

La implementación de esto en Azure Blob Storage es sencilla.

**BlobStore.cs**  
[!code-csharp[BlobStore](~/../botbuilder-samples/samples/csharp_dotnetcore/42.scaleout/BlobStore.cs?range=18-101)]

Como puede verse aquí, Azure Blob Storage es el que realiza el trabajo aquí. Tenga en cuenta la captura de excepciones específicas y cómo que se traduce esto a la hora de cumplir con las expectativas del código que llama. Es decir, queremos que, en la carga, una excepción No encontrado devuelva un valor NULL y que la excepción de guardado Error de condición previa devuelva un valor booleano.

Todo este código fuente estará disponible en el [ejemplo](https://aka.ms/scale-out) correspondiente y ese ejemplo incluirá una implementación de almacén de memoria.

## <a name="implementing-the-retry-loop"></a>Implementación del bucle de reintentos
La forma básica del bucle se deriva directamente del comportamiento que se muestra en los diagramas de secuencia.

Al recibir una actividad se crea una clave para el estado correspondiente de esa conversación. No se va a cambiar la relación entre la actividad y el estado de la conversación, por lo que se va a crear la clave exactamente de la misma forma que se hizo en la implementación de estado predeterminada.

Después de haber creado la clave adecuada se intentará cargar el estado correspondiente. Posteriormente, se ejecutarán los diálogos del bot y, finalmente, se intentará realizar la operación de guardar. Si la operación de guardar es correcta, se enviarán las actividades salientes que resultaron de la ejecución del diálogo y terminaremos. En caso contrario, volveremos a repetir todo el proceso desde antes de la carga. Rehacer la carga nos ofrecerá un nuevo valor de ETag de modo que la próxima vez, la operación de guardar se realizará correctamente.

La implementación OnTurn resultante tiene el siguiente aspecto:

**ScaleoutBot.cs**  
[!code-csharp[OnMessageActivity](~/../botbuilder-samples/samples/csharp_dotnetcore/42.scaleout/Bots/ScaleOutBot.cs?range=43-72)]

Tenga en cuenta que hemos modelado la ejecución del diálogo como una llamada de función. Quizás, en una implementación más sofisticada, se habría definido una interfaz y hecho que esta dependencia fuera inyectable, pero para nuestros propósitos, hacer que el diálogo esté detrás de una función estática enfatiza la naturaleza funcional de nuestro enfoque. Como norma general, organizar la implementación de tal forma que los elementos cruciales sean funcionales nos sitúa en muy buena posición en lo que respecta al uso correcto en las redes.


## <a name="implementing-outbound-activity-buffering"></a>Implementación del almacenamiento en búfer para actividades salientes 

El siguiente requisito es que las actividades salientes se almacenen en búfer hasta que se haya realizado una operación de guardar correcta. Esto requerirá una implementación personalizada de BotAdapter. En este código, se implementará la función abstracta SendActivity para agregar la actividad a una lista en lugar de enviarla. El diálogo que vamos a hospedar no será el más correcto.
En este escenario en particular, no se admiten las operaciones UpdateActivity y DeleteActivity, por lo que estos métodos solo generarán respuestas de No implementado. Tampoco nos vamos a preocupar por el valor devuelto por la operación SendActivity. Esto lo usan algunos canales en casos en los que se deben enviar las actualizaciones de las actividades, por ejemplo, para deshabilitar los botones o tarjetas que aparecen en el canal. Estos intercambios de mensajes pueden resultar complicados, especialmente cuando se requiere el estado, lo cual está fuera del objetivo de este artículo. La implementación completa del BotAdapter personalizado tiene este aspecto:

**DialogHostAdapter.cs**  
[!code-csharp[DialogHostAdapter](~/../botbuilder-samples/samples/csharp_dotnetcore/42.scaleout/DialogHostAdapter.cs?range=19-46)]

## <a name="integration"></a>Integración

Ya solo queda unir todas estas diversas piezas y conectarlas a las piezas existentes en el marco. El bucle de reintentos principal se encuentra en la función IBot OnTurn. Contiene nuestra implementación IStore personalizada que, con fines de prueba, hemos hecho que sea con dependencia inyectable. Hemos puesto todo el código de hospedaje del diálogo en una clase denominada DialogHost que expone una única función estática pública. Esta función está definida para tomar la actividad entrante y el estado anterior y, a continuación, devolver las actividades resultantes y el nuevo estado.

Lo primero que hay que hacer en esta función es crear el BotAdapter personalizado que se mencionó anteriormente. Posteriormente, vamos a ejecutar el diálogo exactamente de la misma forma en que lo solemos hacer creando un DialogSet y un DialogContext, y realizando los flujos Continue o Begin habituales. La única parte que no hemos tratado es la necesidad de un descriptor de acceso personalizado. Esto resulta ser una clase shim muy sencilla que facilita el traslado del estado del diálogo al sistema de diálogos. El descriptor de acceso utiliza semántica de referencia cuando trabaja con el sistema de diálogos y, por ello, lo único necesario es pasar el control. Para facilitar aún más las cosas, hemos restringido la plantilla de clase que vamos a usar para la semántica de referencia.

Estamos siendo muy cuidadosos con la disposición en capas, y vamos a poner el código de JsonSerialization insertado en el código de hospedaje porque no queremos que esté dentro de la capa de almacenamiento conectable, ya que las diferentes implementaciones se podrían serializar de distinta manera.

Este es el código del controlador:

**DialogHost.cs**  
[!code-csharp[DialogHost](~/../botbuilder-samples/samples/csharp_dotnetcore/42.scaleout/DialogHost.cs?range=22-72)]

Y por último, el descriptor de acceso personalizado, en el que solo debemos implementar Get, porque el estado es por referencia:

**RefAccessor.cs**  
[!code-csharp[RefAccessor](~/../botbuilder-samples/samples/csharp_dotnetcore/42.scaleout/RefAccessor.cs?range=22-60)]

## <a name="additional-information"></a>Información adicional
El código de [ejemplo de C#](http://aka.ms/scale-out) que se usa en este artículo está disponible en GitHub.


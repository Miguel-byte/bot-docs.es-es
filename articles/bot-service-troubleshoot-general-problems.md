---
title: Solución de problemas de bots | Microsoft Docs
description: Solución de problemas generales de desarrollo de bots con las preguntas técnicas más frecuentes.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/26/2018
ms.openlocfilehash: 34a23910c76a22fe39d1ce5457bb74dd285ca939
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225526"
---
# <a name="troubleshooting-general-problems"></a>Solución de problemas generales
Estas preguntas más frecuentes pueden ayudarle a solucionar problemas comunes de desarrollo de bots o problemas de funcionamiento.

## <a name="how-can-i-troubleshoot-issues-with-my-bot"></a>¿Cómo puedo solucionar problemas con mi bot?

1. Depure el código de fuente del bot con [Visual Studio Code](debug-bots-locally-vscode.md) o [Visual Studio](https://docs.microsoft.com/en-us/visualstudio/debugger/navigating-through-code-with-the-debugger?view=vs-2017).
2. Pruebe el bot con el [emulador](bot-service-debug-emulator.md) antes de implementarlo en la nube.
3. Implemente el bot en una plataforma de hospedaje en la nube como Azure y, a continuación, pruebe la conectividad con el bot mediante el control de chat web integrado en el panel del bot en el <a href="https://dev.botframework.com" target="_blank">portal de Bot Framework</a>. Si tiene problemas con el bot después de implementarlo en Azure, puede consultar este artículo de blog: [Understanding Azure troubleshooting and support](https://azure.microsoft.com/en-us/blog/understanding-azure-troubleshooting-and-support/) (Descripción de la solución de problemas y el soporte técnico de Azure).
4. Descarte la [autenticación][TroubleshootingAuth] como un posible problema.
5. Pruebe el bot en Skype. Esto le ayudará a validar la experiencia del usuario de un extremo a otro.
6. Considere la posibilidad de probar el bot en canales que tienen requisitos de autenticación adicionales, como Direct Line o Web Chat.

## <a name="how-can-i-troubleshoot-authentication-issues"></a>¿Cómo puedo solucionar los problemas de autenticación?

Para más información acerca de cómo solucionar problemas de autenticación con el bot, consulte [Solución de problemas de autenticación de Bot Framework][TroubleshootingAuth].

## <a name="im-using-the-bot-framework-sdk-for-net-how-can-i-troubleshoot-issues-with-my-bot"></a>Estoy usando Bot Framework SDK para .NET. ¿Cómo puedo solucionar problemas con mi bot?

**Busque las excepciones.**  
En Visual Studio 2017, vaya a **Depurar** > **Windows** > **Configuración de excepciones**. En la ventana **Configuración de excepciones**, seleccione la casilla de verificación **Interrumpir cuando se produzcan** junto a **Excepciones de Common Language Runtime**. También puede ver la salida del diagnóstico en la ventana de salida cuando hay excepciones iniciadas o no controladas.

**Examine la pila de llamadas.**  
En Visual Studio, puede elegir si depura [Solo mi código](https://msdn.microsoft.com/en-us/library/dn457346.aspx) o no. Examinar la pila de llamadas completa puede proporcionar información adicional sobre los problemas.

**Asegúrese de que todos los métodos de diálogo finalizan con un plan para tratar el mensaje siguiente.**  
Todos los pasos del diálogo deben ir al paso siguiente de la cascada o finalizar el diálogo actual para que se extraigan de la pila. Si un paso no se controla correctamente, la conversación no continuará como se espera. Eche un vistazo al artículo de conceptos de [diálogos](v4sdk/bot-builder-concept-dialog.md) para más información sobre los diálogos.

## <a name="why-doesnt-the-typing-activity-do-anything"></a>¿Por qué la actividad de escritura no hace nada?
Algunos canales no admiten actualizaciones de escritura transitorias en su cliente.

## <a name="what-is-the-difference-between-the-connector-library-and-builder-library-in-the-sdk"></a>¿Cuál es la diferencia entre la biblioteca Connector y la biblioteca Builder en el SDK?

La biblioteca Connector es la exposición de la API REST. La biblioteca Builder agrega el modelo de programación de diálogos conversacionales y otras características como avisos, cascadas, cadenas y relleno guiado de formularios. La biblioteca Builder también proporciona acceso a servicios de Cognitive Services como LUIS.

## <a name="what-causes-an-error-with-http-status-code-429-too-many-requests"></a>¿Qué provoca un error con código de estado HTTP 429 "Demasiadas solicitudes"?

Una respuesta de error con código de estado HTTP 429 indica que se han emitido demasiadas solicitudes en un período de tiempo determinado. El cuerpo de la respuesta debe incluir una explicación del problema y también puede especificar el intervalo mínimo necesario entre solicitudes. Un posible origen de este error es [ngrok](https://ngrok.com/). Si está en un plan gratuito y alcanza los límites de ngrok, vaya a la página de precios y límites en su sitio web para más [opciones](https://ngrok.com/product#pricing). 

## <a name="why-arent-my-bot-messages-getting-received-by-the-user"></a>¿Por qué el usuario no recibe los mensajes del bot?

La actividad de mensaje generada en la respuesta se debe dirigir correctamente, de lo contrario no llega a su destino previsto. En la mayoría de los casos no necesitará controlar esto de forma explícita; el SDK se encarga del direccionamiento de la actividad de mensaje. 

El direccionamiento correcto de una actividad significa incluir los detalles de *referencia de la conversación* junto con detalles sobre el remitente y el destinatario. En la mayoría de los casos, la actividad de mensaje se envía como respuesta a una que ha llegado. Por lo tanto, los detalles de direccionamiento se pueden obtener de la actividad de entrada. 

Si examina los seguimientos o los registros de auditoría, puede comprobar para asegurarse de que los mensajes se direccionan correctamente. Si no es así, establezca un punto de interrupción en el bot y compruebe dónde se establecen los identificadores para el mensaje.

## <a name="how-can-i-run-background-tasks-in-aspnet"></a>¿Cómo puedo ejecutar tareas en segundo plano en ASP.NET? 

En algunos casos, es posible que desee iniciar una tarea asincrónica que espera unos segundos y, a continuación, ejecuta algún código para borrar el perfil de usuario o restablecer el estado de la conversación o el diálogo. Para más información sobre cómo lograr esto, consulte [Cómo ejecutar tareas en segundo plano en ASP.NET](https://www.hanselman.com/blog/HowToRunBackgroundTasksInASPNET.aspx). En particular, considere el uso de [HostingEnvironment.QueueBackgroundWorkItem](https://msdn.microsoft.com/en-us/library/dn636893(v=vs.110).aspx). 


## <a name="how-do-user-messages-relate-to-https-method-calls"></a>¿Cómo se relacionan los mensajes de usuario con las llamadas a métodos HTTPS?

Cuando el usuario envía un mensaje por un canal, el servicio web de Bot Framework emitirá una solicitud HTTPS POST al punto de conexión de servicio web del bot. El bot puede enviar cero, uno o varios mensajes al usuario en ese canal emitiendo una solicitud HTTPS POST independiente a Bot Framework por cada mensaje que envía.

## <a name="my-bot-is-slow-to-respond-to-the-first-message-it-receives-how-can-i-make-it-faster"></a>Mi bot es lento para responder al primer mensaje que recibe. ¿Cómo puedo hacerlo más rápido?

Los bots son servicios web y algunas plataformas de hospedaje, incluida Azure, ponen automáticamente el servicio en modo de suspensión si no recibe tráfico en un período de tiempo determinado. Si le ocurre esto al bot, este debe comenzar desde cero la próxima vez que recibe un mensaje, lo que hace que su respuesta sea mucho más lenta que si ya se estaba ejecutando.

Algunas plataformas de hospedaje le permiten configurar el servicio para que no entre en suspensión. Para hacer esto en Azure, vaya al servicio del bot en [Azure Portal](https://portal.azure.com), seleccione **Configuración de la aplicación** y, a continuación, seleccione **Siempre disponible**. Esta opción está disponible en la mayoría, pero no para todos los planes de servicio.

## <a name="how-can-i-guarantee-message-delivery-order"></a>¿Cómo puedo garantizar el orden de entrega de los mensajes?

Bot Framework conservará el orden de los mensajes tanto como sea posible. Por ejemplo, si envía un mensaje A y espera la finalización de esa operación HTTP antes de iniciar otra operación HTTP para enviar el mensaje B, Bot Framework comprenderá automáticamente que el mensaje A debe preceder al mensaje B. Sin embargo, en general, el orden de entrega de los mensajes no se puede garantizar puesto que el canal es el responsable último de la entrega de los mensajes y puede reordenar los mensajes. Para mitigar el riesgo de que los mensajes se entreguen en orden incorrecto, puede implementar un retraso de tiempo entre los mensajes.

## <a name="how-can-i-intercept-all-messages-between-the-user-and-my-bot"></a>¿Cómo puedo interceptar todos los mensajes entre el usuario y el bot?

Mediante Bot Framework SDK para .NET, puede proporcionar implementaciones de las interfaces `IPostToBot` e `IBotToUser` al contenedor de inserción de dependencias `Autofac`. Con Bot Framework SDK para cualquier lenguaje, puede usar middleware para el mismo propósito. El repositorio [BotBuilder-Azure](https://github.com/Microsoft/BotBuilder-Azure) contiene bibliotecas de C# y Node.js que registrarán estos datos en una tabla de Azure.

## <a name="why-are-parts-of-my-message-text-being-dropped"></a>¿Por qué se eliminan partes del texto del mensaje?

Bot Framework y muchos canales interpretan el texto como si tuviese formato [Markdown](https://en.wikipedia.org/wiki/Markdown). Compruebe si el texto contiene caracteres que se puedan interpretar como sintaxis de Markdown.

## <a name="how-can-i-support-multiple-bots-at-the-same-bot-service-endpoint"></a>¿Cómo puedo admitir varios bots en el mismo punto de conexión de servicio de bots? 

Este [ejemplo](https://github.com/Microsoft/BotBuilder/issues/2258#issuecomment-280506334) muestra cómo configurar el elemento `Conversation.Container` con el elemento `MicrosoftAppCredentials` correcto y usar un simple elemento `MultiCredentialProvider` para autenticar varios identificadores de aplicación y contraseñas.

## <a name="identifiers"></a>Identifiers (Identificadores)

## <a name="how-do-identifiers-work-in-the-bot-framework"></a>¿Cómo funcionan los identificadores en Bot Framework?

Para más información acerca de los identificadores en Bot Framework, consulte la [Guía de identificadores de Bot Framework][BotFrameworkIDGuide].

## <a name="how-can-i-get-access-to-the-user-id"></a>¿Cómo se puede obtener acceso al identificador de usuario?

Los SMS y los mensajes de correo electrónico proporcionarán el identificador de usuario sin procesar en la propiedad `from.Id`. En los mensajes de Skype, la propiedad `from.Id` contendrá un identificador único para el usuario que difiere del identificador de Skype del usuario. Si necesita conectarse a una cuenta existente, puede usar una tarjeta de inicio de sesión e implementar su propio flujo de OAuth para conectar el identificador de usuario con el identificador de usuario de su propio servicio.

## <a name="why-are-my-facebook-user-names-not-showing-anymore"></a>¿Por qué ya no se muestran los nombres de usuario de Facebook?

¿Ha cambiado la contraseña de Facebook? Si lo ha hecho, invalidará el token de acceso y deberá actualizar la configuración del bot para el canal de Facebook Messenger en el <a href="https://dev.botframework.com" target="_blank">portal de Bot Framework</a>.

## <a name="why-is-my-kik-bot-replying-im-sorry-i-cant-talk-right-now"></a>¿Por qué mi bot de Kik responde "Lo siento, no puedo hablar en este momento"?

Los bots en desarrollo en Kik permiten 50 suscriptores. Después de que 50 usuarios únicos hayan interactuado con el bot, cualquier nuevo usuario que intente charlar con el bot recibirá el mensaje "Lo siento, no puedo hablar en este momento." Para más información, consulte la [Documentación de Kik](https://botsupport.kik.com/hc/en-us/articles/225764648-How-can-I-share-my-bot-with-Kik-users-while-in-development-).

## <a name="how-can-i-use-authenticated-services-from-my-bot"></a>¿Cómo puedo usar servicios autenticados desde mi bot?

Para la autenticación de Azure Active Directory, consulte la adición de autenticación [V3](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-authentication?view=azure-bot-service-3.0&tabs=csharp) | [V4](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-authentication?view=azure-bot-service-4.0&tabs=csharp). 

> [!NOTE] 
> Si agrega funcionalidades de seguridad y autenticación al bot, debe asegurarse de que los patrones que implementa en el código cumplen con los estándares de seguridad adecuados para la aplicación.

## <a name="how-can-i-limit-access-to-my-bot-to-a-pre-determined-list-of-users"></a>¿Cómo puedo limitar el acceso al bot a una lista predefinida de usuarios?

Algunos canales, como SMS y el correo electrónico, proporcionan direcciones sin ámbito. En estos casos, los mensajes del usuario contendrán el identificador de usuario sin procesar en la propiedad `from.Id`.

Otros canales, como Skype, Facebook y Slack, proporcionan direcciones de ámbito o con inquilinos de forma que se impide que un bot pueda predecir el identificador de un usuario antes de tiempo. En estos casos, deberá autenticar al usuario mediante un vínculo de inicio de sesión o un secreto compartido para determinar si están o no autorizados para usar el bot.

## <a name="why-does-my-direct-line-11-conversation-start-over-after-every-message"></a>¿Por qué mi conversación de Direct Line 1.1 se inicia después de cada mensaje?

Si la conversación de Direct Line se inicia después de cada mensaje, probablemente falta la propiedad `from` o tiene valor `null` en los mensajes que el cliente de Direct Line envió al bot. Cuando un cliente de Direct Line envía un mensaje sin la propiedad `from` o con valor `null`, el servicio de Direct Line asigna automáticamente un identificador, por lo que todos los mensajes que envía el cliente parecerán provenir de un usuario nuevo y diferente.

Para solucionar este problema, establezca la propiedad `from` en cada mensaje que envía el cliente de Direct Line en un valor estable que identifique de forma única al usuario que envía el mensaje. Por ejemplo, si un usuario ya ha iniciado sesión en una página web o aplicación, podría usar ese identificador de usuario existente como el valor de la propiedad `from` en los mensajes que envía el usuario. Como alternativa, puede generar un identificador de usuario aleatorio en la carga de la página o en la carga de la aplicación, almacenar ese identificador en una cookie o en un estado de dispositivo y usar este identificador como el valor de la propiedad `from` en los mensajes que envía el usuario.

## <a name="what-causes-the-direct-line-30-service-to-respond-with-http-status-code-502-bad-gateway"></a>¿Qué hace que el servicio de Direct Line 3.0 responda con código de estado HTTP 502 "Puerta de enlace incorrecta"?
Direct Line 3.0 devuelve el código de estado HTTP 502 cuando intenta ponerse en contacto con el bot pero la solicitud no finaliza correctamente. Este error indica que el bot devolvió un error o que la solicitud ha agotado el tiempo de espera. Para más información sobre los errores que genera el bot, vaya al panel del bot en el <a href="https://dev.botframework.com" target="_blank">portal de Bot Framework</a> y haga clic en el vínculo "Problemas" del canal afectado. Si ha configurado Application Insights para el bot, también puede encontrar allí información detallada del error. 

::: moniker range="azure-bot-service-3.0"

## <a name="what-is-the-idialogstackforward-method-in-the-bot-framework-sdk-for-net"></a>¿Qué es el método IDialogStack.Forward en Bot Framework SDK para. NET?

El propósito principal de `IDialogStack.Forward` consiste en reutilizar un diálogo secundario existente que suele ser "reactivo", donde el diálogo secundario (en `IDialog.StartAsync`) espera un elemento `T` de un objeto con un controlador `ResumeAfter`. En particular, si tiene un diálogo secundario que espera un elemento `IMessageActivity` `T`, puede reenviar el elemento `IMessageActivity` entrante (que se recibió en algún diálogo primario) mediante el uso del método `IDialogStack.Forward`. Por ejemplo, para reenviar un elemento `IMessageActivity` entrante a un elemento `LuisDialog`, llame a `IDialogStack.Forward` para insertar el elemento `LuisDialog` en la pila de diálogos, ejecute el código de `LuisDialog.StartAsync` hasta que se programe una espera para el mensaje siguiente y, a continuación, alimente de inmediato la espera con el elemento `IMessageActivity` reenviado.

`T` suele ser un elemento `IMessageActivity`, puesto que `IDialog.StartAsync` normalmente se construye para esperar a ese tipo de actividad. Puede usar `IDialogStack.Forward` a un elemento `LuisDialog` como un mecanismo para interceptar los mensajes del usuario para su procesamiento antes de reenviar el mensaje a un elemento `LuisDialog` existente. Como alternativa, también puede usar `DispatchDialog` con `ContinueToNextGroup` para ese propósito.

Se espera encontrar el elemento reenviado en el primer controlador `ResumeAfter` (por ejemplo, `LuisDialog.MessageReceived`) que se ha programado por `StartAsync`.

::: moniker-end

## <a name="what-is-the-difference-between-proactive-and-reactive"></a>¿Cuál es la diferencia entre "proactivo" y "reactivo"?

Desde la perspectiva del bot, "reactivo" significa que el usuario inicia la conversación enviando un mensaje al bot y el bot reacciona respondiendo a ese mensaje. En cambio, "proactivo" significa que el bot inicia la conversación enviando el primer mensaje al usuario. Por ejemplo, un bot puede enviar un mensaje proactivo que notifique al usuario cuando un temporizador expira o se produce un evento.

## <a name="how-can-i-send-proactive-messages-to-the-user"></a>¿Cómo puedo enviar mensajes proactivos al usuario?

Para obtener ejemplos que muestran cómo enviar mensajes proactivos, consulte los [ejemplos de C# V4](https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/16.proactive-messages) y los [ejemplos de Node.js V4](https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/16.proactive-messages) del repositorio BotBuilder-Samples en GitHub.

## <a name="how-can-i-reference-non-serializable-services-from-my-c-dialogs"></a>¿Cómo puedo hacer referencia a servicios no serializables desde los diálogos de C#?

Existen varias opciones:

* Resolver la dependencia mediante `Autofac` y `FiberModule.Key_DoNotSerialize`. Esta es la solución más limpia.
* Usar los atributos [NonSerialized](https://msdn.microsoft.com/en-us/library/system.nonserializedattribute(v=vs.110).aspx) y [OnDeserialized](https://msdn.microsoft.com/en-us/library/system.runtime.serialization.ondeserializedattribute(v=vs.110).aspx) para restaurar la dependencia en la deserialización. Esta es la solución más sencilla.
* No almacene esa dependencia para que no se serialice. No se recomienda esta solución, aunque es técnicamente posible.
* Utilizar el suplente de serialización de reflexión. Esta solución puede no ser posible en algunos casos y tiene el riesgo de serializar demasiado.

::: moniker range="azure-bot-service-3.0"

## <a name="where-is-conversation-state-stored"></a>¿Donde se almacena el estado de la conversación?

Los datos de las bolsas de propiedades de usuario, de conversación y de conversación privada se almacenan mediante la interfaz `IBotState` de Connector. El identificador del bot determina el ámbito de cada bolsa de propiedades. La bolsa de propiedades de usuario tiene la clave por identificador de usuario, la bolsa de propiedades de conversación tiene la clave por identificador de conversación y la bolsa de propiedades de conversación privada tiene la clave por identificador de usuario e identificador de conversación. 

Si usa Bot Framework SDK para .NET o Bot Framework SDK para Node.js para crear el bot, la pila de diálogos y los datos de los diálogos se almacenarán automáticamente como entradas en la bolsa de propiedades de conversación privada. La implementación de C# utiliza la serialización binaria y la implementación de Node.js usa la serialización de JSON.

Dos servicios implementan la interfaz REST `IBotState`.

* Bot Framework Connector proporciona un servicio en la nube que implementa esta interfaz y almacena los datos en Azure.  Estos datos se cifran en reposo e intencionadamente no expiran.
* El emulador de Bot Framework proporciona una implementación en memoria de esta interfaz para depurar el bot. Estos datos expiran cuando termina el proceso del emulador.

Si desea almacenar estos datos dentro de los centros de datos, puede proporcionar una implementación personalizada del servicio de estado. Esto puede hacerse al menos de dos formas:

* Utilizar la capa REST para proporcionar un servicio `IBotState` personalizado.
* Utilizar las interfaces de Builder en la capa de lenguaje (Node.js o C#).

> [!IMPORTANT]
> No se recomienda usar Bot Framework State Service API en entornos de producción y es posible que deje de usarse en una versión futura. Se recomienda actualizar el código del bot para que use el almacenamiento en memoria para realizar pruebas o usar una de las **extensiones de Azure** para bots de producción. Para más información, consulte el tema **Administración de datos de estado** para la implementación de [.NET](~/dotnet/bot-builder-dotnet-state.md) o [Node](~/nodejs/bot-builder-nodejs-state.md).

::: moniker-end

## <a name="what-is-an-etag--how-does-it-relate-to-bot-data-bag-storage"></a>¿Qué es una ETag?  ¿Cómo se relaciona con el almacenamiento de las bolsas de datos del bot?

Una [ETag](https://en.wikipedia.org/wiki/HTTP_ETag) es un mecanismo para el [control de simultaneidad optimista](https://en.wikipedia.org/wiki/Optimistic_concurrency_control). El almacenamiento de las bolsas de datos del bot utiliza ETags para evitar actualizaciones conflictivas de los datos. Un error de ETag con código de estado HTTP 412 "Error en la condición previa" indica que hubo varias secuencias de "lectura-modificación-escritura" en ejecución simultáneamente para esa bolsa de datos del bot.

La pila de diálogos y el estado se almacenan en bolsas de datos del bot. Por ejemplo, es posible que vea el error de ETag "Error en la condición previa" si el bot todavía está procesando un mensaje anterior cuando recibe un nuevo mensaje para esa conversación.

## <a name="what-causes-an-error-with-http-status-code-412-precondition-failed-or-http-status-code-409-conflict"></a>¿Qué provoca un error con código de estado HTTP 412 "Error en la condición previa" o código de estado HTTP 409 "Conflicto"?

El servicio `IBotState` de Connector se usa para almacenar las bolsas de datos del bot (es decir, las bolsas de datos de usuario, de conversación y de conversación privada, donde la bolsa de datos privada incluye el estado del "flujo de control" de la pila de diálogos). El control de la simultaneidad en el servicio `IBotState` se administra por simultaneidad optimista mediante ETags. Si hay un conflicto de actualización (debido a una actualización simultánea de una única bolsa de datos del bot) durante una secuencia de "lectura-modificación-escritura":

* Si se conservan las ETags, se produce un error con código de estado HTTP 412 "Error en la condición previa" en el servicio `IBotState`. Este es el comportamiento predeterminado en Bot Framework SDK para .NET.
* Si no se conservan las ETags (es decir, ETag se establece en `\*`), la directiva "la última escritura gana" entrará en vigor, lo que evita el error "Error en la condición previa", pero existe riesgo de pérdida de datos. Este es el comportamiento predeterminado en Bot Framework SDK para Node.js.

## <a name="how-can-i-fix-precondition-failed-412-or-conflict-409-errors"></a>¿Cómo se pueden corregir los errores "Error en la condición previa" (412) o "Conflicto" (409)?

Estos errores indican que el bot ha procesado varios mensajes para la misma conversación a la vez. Si el bot se conecta con servicios que requieren mensajes ordenados con precisión, considere la posibilidad de bloquear el estado de la conversación para asegurarse de que los mensajes no se procesan en paralelo. 

::: moniker range="azure-bot-service-3.0"

Bot Framework SDK para .NET proporciona un mecanismo (la clase `LocalMutualExclusion`, que implementa `IScope`) para serializar de forma pesimista el tratamiento de una sola conversación con un semáforo en memoria. Puede extender esta implementación para que use una concesión de Redis, con el ámbito de la dirección de la conversación.

Si el bot no está conectado a servicios externos o si el procesamiento de mensajes de la misma conversación en paralelo es aceptable, puede agregar este código para ignorar los conflictos que se produzcan en Bot State API. Esto permitirá que la última respuesta establezca el estado de la conversación.

```cs
var builder = new ContainerBuilder();
builder
    .Register(c => new CachingBotDataStore(c.Resolve<ConnectorStore>(), CachingBotDataStoreConsistencyPolicy.LastWriteWins))
    .As<IBotDataStore<BotData>>()
    .AsSelf()
    .InstancePerLifetimeScope();
builder.Update(Conversation.Container);
```
::: moniker-end

## <a name="is-there-a-limit-on-the-amount-of-data-i-can-store-using-the-state-api"></a>¿Hay un límite en la cantidad de datos que puedo almacenar mediante State API?

Sí, cada almacén de estado (es decir, las bolsas de datos de usuario, de conversación y de conversación privada del bot) puede contener hasta 64 KB de datos. Para más información, consulte [Administración de datos de estado][StateAPI].

::: moniker range="azure-bot-service-3.0"

## <a name="how-do-i-version-the-bot-data-stored-through-the-state-api"></a>¿Cómo se determina la versión de los datos del bot almacenados mediante State API?

> [!IMPORTANT]
> No se recomienda usar Bot Framework State Service API en entornos de producción o bots v4, y es posible que deje de usarse en una versión futura. Se recomienda actualizar el código del bot para que use el almacenamiento en memoria para realizar pruebas o usar una de las **extensiones de Azure** para bots de producción. Para más información, consulte el tema [Administración de datos de estado](v4sdk/bot-builder-howto-v4-state.md).

El servicio de estado permite conservar el progreso en los diálogos de una conversación para que un usuario pueda volver a una conversación con un bot más adelante sin perder su posición. Para conservar esto, las bolsas de propiedades de los datos del bot que se almacenan mediante State API no se borran automáticamente cuando se modifica el código del bot. Debe decidir si se deben borrar o no los datos del bot, en función de si el código modificado es compatible con versiones anteriores de los datos. 

* Si desea restablecer manualmente la pila de diálogos y el estado de la conversación durante el desarrollo del bot, puede usar el comando ` /deleteprofile` para eliminar los datos de estado. Asegúrese de incluir el espacio inicial en este comando para impedir que el canal lo interprete.
* Después de que el bot se haya implementado en producción, puede controlar la versión de los datos del bot para que si se cambia la versión, se borren los datos de estado asociados. Con Bot Framework SDK para Node.js, esto se puede realizar mediante middleware y con Bot Framework SDK para .NET, se puede realizar con una implementación de `IPostToBot`.

> [!NOTE]
> Si la pila de diálogos no se puede deserializar correctamente, debido a cambios de formato de la serialización o porque el código ha cambiado demasiado, se restablecerá el estado de la conversación.

::: moniker-end

## <a name="what-are-the-possible-machine-readable-resolutions-of-the-luis-built-in-date-time-duration-and-set-entities"></a>¿Cuáles son las posibles resoluciones legibles por máquina de las entidades integradas de fecha, hora, duración y conjunto de LUIS?

Para obtener una lista de ejemplos, consulte la [sección de entidades integradas][LUISPreBuiltEntities] de la documentación de LUIS.

## <a name="how-can-i-use-more-than-the-maximum-number-of-luis-intents"></a>¿Cómo puedo usar más del número máximo de intenciones de LUIS?

Puede dividir el modelo y llamar al servicio de LUIS en serie o en paralelo.

## <a name="how-can-i-use-more-than-one-luis-model"></a>¿Cómo se puede usar más de un modelo de LUIS?

Tanto Bot Framework SDK para Node.js como Bot Framework SDK para .NET admiten llamar a varios modelos de LUIS desde un único diálogo de intenciones de LUIS. Tenga en cuenta las siguientes observaciones:

* Si usa varios modelos de LUIS, se da por supuesto que los modelos de LUIS tienen conjuntos de intenciones no superpuestos.
* Si usa varios modelos de LUIS, se da por supuesto que las puntuaciones de los distintos modelos son comparables, para seleccionar la "intención más coincidente" entre varios modelos.
* Si usa varios modelos de LUIS significa que si una intención coincide con un modelo, también tendrá una alta coincidencia con la intención "none" de los otros modelos. Puede evitar la selección de la intención "none" en esta situación; Bot Framework SDK para Node.js rebajará automáticamente la puntuación de las intenciones "none" para evitar este problema.

## <a name="where-can-i-get-more-help-on-luis"></a>¿Dónde puedo obtener más ayuda sobre LUIS?

* [Introducción a Language Understanding (LUIS): Microsoft Cognitive Services](https://www.youtube.com/watch?v=jWeLajon9M8) (vídeo)
* [Sesión de aprendizaje avanzado para Language Understanding (LUIS) ](https://www.youtube.com/watch?v=39L0Gv2EcSk) (vídeo)
* [Documentación de LUIS](/azure/cognitive-services/LUIS/Home)
* [Foro de Language Understanding](https://social.msdn.microsoft.com/forums/azure/en-US/home?forum=LUIS) 


## <a name="what-are-some-community-authored-dialogs"></a>¿Cuáles son algunos de los diálogos creados por la comunidad?

* [BotAuth](https://www.nuget.org/packages/BotAuth): autenticación de Azure Active Directory
* [BestMatchDialog](http://www.garypretty.co.uk/2016/08/01/bestmatchdialog-for-microsoft-bot-framework-now-available-via-nuget/): envío de texto de usuario a métodos de diálogo basado en expresiones regulares

## <a name="what-are-some-community-authored-templates"></a>¿Cuáles son algunas de las plantillas creadas por la comunidad?

* [ES6 BotBuilder](https://github.com/brene/botbuilder-es6-template): plantilla de ES6 de Bot Builder

## <a name="why-do-i-get-an-authorizationrequestdenied-exception-when-creating-a-bot"></a>¿Por qué obtengo una excepción Authorization_RequestDenied al crear un bot?

Los permisos para crear bots de Azure Bot Service se administran mediante el portal de Azure Active Directory (AAD). Si los permisos no están configurados correctamente en el [portal de AAD](http://aad.portal.azure.com), los usuarios obtendrán la excepción **Authorization_RequestDenied** al intentar crear un servicio de bots.

En primer lugar, compruebe si es un "Invitado" del directorio:

1. Inicie sesión en [Azure Portal](http://portal.azure.com).
2. Haga clic en **Todos los servicios** y busque *active*.
3. Seleccione **Azure Active Directory**.
4. Haga clic en **Usuarios**.
5. Busque el usuario en la lista y asegúrese de que el **Tipo de usuario** no es un **Invitado**.

![Tipo de usuario de Azure Active Directory](~/media/azure-active-directory/user_type.png)

Una vez que haya comprobado que no es un **Invitado**, para asegurarse de que los usuarios de un directorio activo pueden crear el servicio de bots, el administrador de directorio debe configurar las opciones siguientes:

1. Inicie sesión en el [portal de AAD](http://aad.portal.azure.com). Vaya a **Usuarios y grupos** y seleccione **Configuración de usuario**.
2. En la sección **Registro de aplicación**, establezca **Los usuarios pueden registrar aplicaciones** en **Sí**. Esto permite a los usuarios del directorio crear el servicio de bots.
3. En la sección **Usuarios externos**, establezca **Los permisos de los usuarios invitados están limitados** en **No**. Esto permite a los usuarios invitados del directorio crear el servicio de bots.

![Centro de administración de Azure Active Directory](~/media/azure-active-directory/admin_center.png)

## <a name="why-cant-i-migrate-my-bot"></a>¿Por qué no puedo migrar mi bot?

Si tiene problemas para migrar el bot, es posible que el bot pertenezca a un directorio que no sea el directorio predeterminado. Siga estos pasos:

1. Desde el directorio de destino, agregue un nuevo usuario (mediante la dirección de correo electrónico) que no sea miembro del directorio predeterminado y conceda al usuario el rol de colaborador en las suscripciones que son el destino de la migración.

2. Desde el [Portal para desarrolladores](https://dev.botframework.com), agregue la dirección de correo electrónico del usuario como copropietario del bot que se va a migrar. A continuación, cierre la sesión.

3. Inicie sesión en el [Portal para desarrolladores](https://dev.botframework.com) como el nuevo usuario y proceda a migrar el bot.

## <a name="where-can-i-get-more-help"></a>¿Dónde puedo obtener más ayuda?

* Aproveche la información de las preguntas respondidas previamente en [Stack Overflow](https://stackoverflow.com/questions/tagged/botframework) o publique sus propias preguntas utilizando la etiqueta `botframework`. Tenga en cuenta que en Stack Overflow deberá escribir un título descriptivo, un resumen conciso del problema y detalles suficientes para reproducir el problema. Las solicitudes de características o las preguntas demasiado amplias no se tendrán en cuenta; los nuevos usuarios deben visitar el [Centro de ayuda de Stack Overflow](https://stackoverflow.com/help/how-to-ask) para más detalles.
* Consulte [Problemas de BotBuilder](https://github.com/Microsoft/BotBuilder/issues) en GitHub para obtener información sobre los problemas conocidos con Bot Framework SDK o para notificar un problema nuevo.
* Aproveche la información del debate de la Comunidad BotBuilder en [Gitter](https://gitter.im/Microsoft/BotBuilder).




[LUISPreBuiltEntities]: /azure/cognitive-services/luis/pre-builtentities
[BotFrameworkIDGuide]: bot-service-resources-identifiers-guide.md
[StateAPI]: ~/rest-api/bot-framework-rest-state.md
[TroubleshootingAuth]: bot-service-troubleshoot-authentication-problems.md


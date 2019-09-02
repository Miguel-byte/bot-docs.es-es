---
title: Introducción a la migración del SDK Bot Framework | Microsoft Docs
description: Proporciona una introducción a los cambios en el SDK y cómo migrar de v3 a v4.
keywords: migración de bots
author: mmiele
ms.author: v-mimiel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 06/11/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 576947edf99705e5d0d8850837b3469f13381d06
ms.sourcegitcommit: 008aa6223aef800c3abccda9a7f72684959ce5e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2019
ms.locfileid: "70026407"
---
# <a name="migration-overview"></a>Información general sobre la migración

El SDK Bot Framework v4 se basa en los comentarios de los clientes y la experiencia obtenida de los SDK anteriores. La nueva versión presenta los niveles adecuados de abstracción al tiempo que habilita una arquitectura flexible en los componentes de los bots. Esto, por ejemplo, permite crear un bot simple y después aumentar su sofisticación gracias a la modularidad y la extensibilidad del SDK Bot Framework v4.

> [!NOTE]
> El SDK Bot Framework v4 se esfuerzan por mantener las cosas sencillas y hacer posible lo complejo.

Se ha adoptado un enfoque abierto y, por lo tanto, el SDK Bot Framework v4 se ha creado con la cooperación de la comunidad. Cuando se envía una solicitud de incorporación de cambios, un [contrato de licencia de colaborador](https://cla.opensource.microsoft.com/) (CLA) determina automáticamente si necesita una licencia. Solo tendrá que hacerlo una vez en todos los repositorios. Normalmente, hay un intervalo de tiempo para establecer el conjunto de objetivos que se debe alcanzar.

## <a name="what-happens-to-bots-built-using-sdk-v3"></a>¿Qué ocurre con los bots compilados con el SDK v3?

El SDK Bot Framework v3 se va a retirar pero las cargas de trabajo de los bots v3 existentes continuarán ejecutándose sin interrupción. Para más información, consulte: [Compatibilidad mientras dure la vigencia del SDK versión 3 de Bot Framework](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-resources-bot-framework-faq?view=azure-bot-service-4.0#bot-framework-sdk-version-3-lifetime-support).

Es muy recomendable que inicie la migración de sus bots v3 a v4. Para poder dar soporte a esta migración, hemos redactado la documentación relacionada y proporcionaremos soporte técnico ampliado para las iniciativas de migración a través de los canales estándar.

## <a name="advantages"></a>Ventajas

- Arquitectura enriquecida, flexible y abierta: permite un diseño más flexible de las conversaciones.
- Disponibilidad: presenta escenarios adicionales con nuevas funcionalidades de canal.
- Más personal de expertos en la materia (SME) en el ciclo de desarrollo: el nuevo diseñador de interfaz gráfica de usuario permite que usuarios que no son desarrolladores colaboren en el diseño de la conversación.
- Velocidad de desarrollo: nuevas herramientas de depuración y pruebas.
- Información sobre el rendimiento: nuevas características de telemetría para evaluar y mejorar la calidad de la conversación.
- Inteligencia: funcionalidades de Cognitive Services mejoradas.

## <a name="why-migrate"></a>Por qué migrar
<!-- [!] The declarative model introduced with Adaptive Dialogs would go great here (when ready).  -->
- Administración mejorada y flexible de las conversaciones.
  - Adaptador de bot para el procesamiento de la actividad.
  - Administración de estados refactorizada.
  - Nueva biblioteca de diálogos.
  - Middleware de diseños que se pueden componer y ampliar: enlaces limpios y coherentes para personalizar el comportamiento.
- Creado para .NET Core
  - rendimiento mejorado.
  - Compatibilidad multiplataforma (Windows/Mac/Linux).
- Modelo de programación coherente en varios lenguajes de programación.
- Documentación mejorada.
- El inspector de bots proporciona funcionalidades de depuración ampliadas.
- Asistente virtual
  - Completa solución que simplifica la creación de bots con intenciones básicas de conversación, integración con Dispatch, QnA Maker, Application Insights y una implementación automatizada.
  - Aptitudes extensibles. Composición de experiencias de conversación mediante la combinación de funcionalidades de conversación reutilizables, conocidas como aptitudes.
- Plataforma de pruebas: Funcionalidades de prueba listas para usar con la nueva arquitectura de adaptador independiente de transporte.
- Telemetría: Obtenga información clave sobre el estado y el comportamiento del bot con el análisis de inteligencia artificial conversacional.
- Próximamente (versión preliminar)
  - Diálogos adaptables: cree conversaciones que se pueden cambiar dinámicamente a medida que la conversación progresa.
  - Generación de lenguaje: defina múltiples variantes de una frase.
- Futuro
  - El diseño declarativo permite el nivel de abstracción para los diseñadores.
  - Diseñador GUI de diálogos
- Azure Bot Service 
  - Canal Direct Line Speech. Reúne el SDK Bot Framework y los servicios de voz de Microsoft. Esto proporciona un canal que permiten la transmisión bidireccional de voz y texto entre el cliente y la aplicación del bot.

## <a name="whats-changed"></a>Lo que ha cambiado

El SDK Bot Framework v4 admite el mismo Bot Framework Service subyacente que la versión v3. Sin embargo, la versión v4 es una refactorización de la versión de los SDK anteriores que ofrecen más flexibilidad y control a la hora de crear los bots. Aquí se incluye lo siguiente:

- Se ha introducido un adaptador de bot
  - El adaptador es parte de la pila de procesamiento de la actividad.
  - Controla la autenticación de Bot Framework e inicializa el contexto de cada turno.
  - Administra el tráfico entrante y saliente entre un canal y el controlador de turnos del bot, que encapsula las llamadas a Bot Framework Service.
  - Para más información, consulte [cómo funcionan los bots](../bot-builder-basics.md).
- Administración de estados refactorizada.
  - Los datos de estado ya no están disponibles automáticamente dentro de un bot.
  - Ahora, el estado se administra mediante objetos de administración de estado y descriptores de acceso de propiedad.
  - Para más información, consulte [Administración del estado](../bot-builder-concept-state.md).
- Se ha introducido una nueva biblioteca de diálogos
  - Los diálogos de la versión v3 tienen que volver a escribirse para la nueva biblioteca de diálogos.
  - Para más información, consulte [Biblioteca de diálogos](../bot-builder-concept-dialog.md).

## <a name="whats-involved-in-migration-work"></a>Qué implica el trabajo de migración

- Actualizar la lógica de configuración.
- Migrar el estado de los usuarios críticos.
  - Nota: el estado de los usuarios confidenciales no se puede mantener en el estado de un bot; en su lugar, se debe almacenar en un almacenamiento independiente bajo su control.
- Migrar la lógica del bot y el diálogo (consulte los temas específicos del lenguaje para más información).

### <a name="migration-estimation-worksheet"></a>Hoja de cálculo de estimación de la migración

La siguiente hoja de cálculo puede ayudarle a calcular la carga de trabajo de la migración. En la columna **Repeticiones**, reemplace *count* por el valor numérico real. En la columna **Camisa**, escriba valores como: *Pequeña*, *Mediana*, *Grande* según su estimación.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

| Paso | V3 | V4 | Repeticiones | Complejidad | Camisa |
| -- | -- | -- | -- | -- | -- |
Para obtener la actividad entrante | IDialogContext.Activity | ITurnContext.Activity | count | Pequeña  
Para crear y enviar una actividad al usuario | activity.CreateReply(“texto”) IDialogContext.PostAsync | MessageFactory.Text(“texto”) ITurnContext.SendActivityAsync | count | Pequeña |
Administración de estados | UserData, ConversationData y PrivateConversationData context.UserData.SetValue context.UserData.TryGetValue botDataStore.LoadAsyn | UserState, ConversationState y PrivateConversationState Con descriptores de acceso | context.UserData.SetValue - count context.UserData.TryGetValue - count botDataStore.LoadAsyn - count | Mediana a Grande (consulte la [administración de estados de usuario](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-state?view=azure-bot-service-4.0#state-management) disponible) |
Controlar el inicio del diálogo | Implemente IDialog.StartAsync | Haga que este sea el primer paso de un diálogo en cascada. | count | Pequeña |  
Envío de una actividad | IDialogContext.PostAsync. | Llame a ITurnContext.SendActivityAsync. | count | Pequeña |  
Esperar la respuesta de un usuario | Use un parámetro IAwaitable<IMessageActivity> y llame a IDialogContext.Wait | Espere a la devolución de ITurnContext.PromptAsync para comenzar un diálogo de confirmación. Después, recupere el resultado en el paso siguiente de la cascada. | count | Mediana (depende del flujo) |  
Controlar la continuación del diálogo | IDialogContext.Wait | Agregue pasos adicionales a un diálogo en cascada o implemente Dialog.ContinueDialogAsync | count | grande |  
Indicar el final del procesamiento hasta el siguiente mensaje del usuario | IDialogContext.Wait | Devuelva Dialog.EndOfTurn. | count | Mediano |  
Iniciar un diálogo secundario | IDialogContext.Call | Espere a la devolución de BeginDialogAsyncmethod del contexto del paso. Si el diálogo secundario devuelve un valor, el valor está disponible en el paso siguiente de la cascada mediante la propiedad Result del contexto del paso. | count | Mediano |  
Reemplazar el diálogo actual por un nuevo diálogo | IDialogContext.Forward | Espere a la devolución de ITurnContext.ReplaceDialogAsync. | count | grande |  
Señalizar que el diálogo actual se ha completado | IDialogContext.Done | Espere a la devolución del método EndDialogAsync del contexto del paso. | count | Mediano |  
Terminar un diálogo debido a un error | IDialogContext.Fail | Inicie una excepción que se detecte en otro nivel del bot, finalice el paso con el estado Cancelled o llame al método CancelAllDialogsAsync del paso o el contexto del paso. | count | Pequeña |  

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

| Paso | V3 | V4 | Repeticiones | Complejidad | Camisa |
| -- | -- | -- | -- | -- | -- |
Para obtener la actividad entrante | IMessage | TurnContext.activity | count | Pequeña  
Para crear y enviar una actividad al usuario | Llame a Session.send('message') | Llame a TurnContext.sendActivity | count | Pequeña |
Administración de estados | UserState & ConversationState UserState.get(), UserState.saveChanges(), ConversationState.get(), ConversationState.saveChanges() | UserState y ConversationState con descriptores de acceso de propiedades | count | Mediana a Grande (consulte la [administración de estados de usuario](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-state?view=azure-bot-service-4.0#state-management) disponible) |
Controlar el inicio del diálogo | llame a session.beginDialog, pasando el identificador del diálogo | llame a DialogContext.beginDialog | count | Pequeña |  
Envío de una actividad | Llame a Session.send | Llame a TurnContext.sendActivity | count | Pequeña |  
Esperar la respuesta de un usuario | Llame a un símbolo del sistema desde el paso de la cascada, por ejemplo: builder.Prompts.text(session, 'Please enter your destination'). Recupere la respuesta en el paso siguiente. | Espere a la devolución de TurnContext.prompt para comenzar un diálogo de confirmación. Después, recupere el resultado en el paso siguiente de la cascada. | count | Mediana (depende del flujo) |  
Controlar la continuación del diálogo | Automático | Agregue pasos adicionales a un diálogo en cascada o implemente Dialog.continueDialog | count | grande |  
Indicar el final del procesamiento hasta el siguiente mensaje del usuario | Session.endDialog | Devuelva Dialog.EndOfTurn. | count | Mediano |  
Iniciar un diálogo secundario | Session.beginDialog | Espere a la devolución del método beginDialog del contexto del paso. Si el diálogo secundario devuelve un valor, el valor está disponible en el paso siguiente de la cascada mediante la propiedad Result del contexto del paso. | count | Mediano |  
Reemplazar el diálogo actual por un nuevo diálogo | Session.replaceDialog | ITurnContext.replaceDialog | count | grande |  
Señalizar que el diálogo actual se ha completado | Session.endDialog | Espere a la devolución del método endDialog del contexto del paso. | count | Mediano |  
Terminar un diálogo debido a un error | Session.pruneDialogStack | Inicie una excepción que se detecte en otro nivel del bot, finalice el paso con el estado Cancelled o llame al método cancelAllDialogs del contexto del paso o del diálogo. | count | Pequeña |  

---

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

El SDK Bot Framework v4 se basa en las mismas API REST subyacentes que el SDK v3. Sin embargo, la versión v4 es una refactorización de la versión anterior del SDK que ofrece más flexibilidad y control sobre sus bots.

Se recomienda migrar a .NET Core, ya que mejora mucho el rendimiento. Sin embargo, algunos bots v3 existentes usan bibliotecas externas que no tienen un equivalente en .NET Core. En este caso, el SDK Bot Framework v4 se puede utilizar con .NET Framework 4.6.1 o versiones posteriores. Encontrará un ejemplo en la ubicación [corebot](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi).

Al migrar un proyecto de v3 a v4, puede elegir una de estas opciones: convertir en el momento a **.NET Framework** o migrar a un nuevo proyecto para **.NET Core**.

#### <a name="net-framework"></a>.NET Framework

- Actualización e instalación de los paquetes NuGet
- Actualización del archivo Global.asax.cs
- Actualización de la clase MessagesController
- Conversión de los cuadros de diálogo

Para más información, consulte [Migración de un bot v3 de .NET a un bot v4 de .NET Framework](conversion-framework.md).

#### <a name="net-core"></a>.NET Core

- Crear el nuevo proyecto mediante una plantilla
- Instale paquetes NuGet adicionales según sea necesario.
- Personalice su bot, actualice el archivo Startup.cs y actualice la clase de controlador.
- Actualizar la clase del bot
- Copie y actualice los diálogos y los modelos.

Para más información, consulte [Migración de un bot v3 de .NET v3 a un bot v4 de .NET Core](conversion-core.md).

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El **SDK Bot Framework v4 para JavaScript** introduce varios cambios fundamentales que afectan a cómo se crean los bots. Estos cambios afectan a la sintaxis para el desarrollo de bots en JavaScript, especialmente en la creación de objetos de bot, la definición de diálogos y la codificación de la lógica del control de eventos. El SDK Bot Framework v4 se basa en las mismas API REST subyacentes que el SDK v3. Sin embargo, la versión v4 es una refactorización de la versión anterior del SDK, y ofrece más flexibilidad y control sobre sus bots, en particular:

- Los diálogos y las instancias de los bot se han desacoplado aún más. En v3, se registraron los diálogos directamente en el constructor del bot. En v4, ahora puede pasar diálogos a instancias de los bot como argumentos, lo que proporciona una mayor flexibilidad de composición.
- La versión v4 proporciona una clase `ActivityHandler`, que ayuda a automatizar el control de diferentes tipos de actividades, tales como mensajes, actualización de conversación y actividades de eventos.
- Para la migración de bots de NodeJS, deberá crear un bot v4 para NodeJS nuevo en JavaScript o TypeScript. Volver a crear la lógica de bot con las nuevas construcciones de v4 descritas en la documentación sobre migración relacionada

#### <a name="migrate-from-v3-to-v4"></a>Migrar desde v3 a v4

- Crear el nuevo proyecto y agregar las dependencias.
- Actualizar el punto de entrada y definir las constantes.
- Crear los diálogos y volver a implementarlos con el SDK v4.
- Actualizar el código del bot para ejecutar los diálogos.
- Migrar el archivo de utilidad `store.js`.

Para más información, consulte [Migración de un bot del SDK v3 para Javascript a v4](conversion-javascript.md).

---

## <a name="additional-resources"></a>Recursos adicionales

Los siguientes recursos adicionales proporcionan más información que puede servir de ayuda durante la migración.  

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

<!-- _Mini-TOC with explainer for .NET topics_ -->
En los temas siguientes describen las diferencias entre las versiones v3 y v4 del SDK Bot Framework para .NET, las diferencias más importantes entre las dos versiones y los pasos para migrar un bot de v3 a v4.

| Tema | DESCRIPCIÓN |
| :--- | :--- |
| [Diferencias entre las versiones v3 y v4 del SDK para .NET](migration-about.md) |Diferencias comunes entre las versiones v3 y v4 del SDK |
| [Referencia rápida de migración de .NET](net-migration-quickreference.md) |Principales cambios en las versiones v3 y v4 del SDK |
| [Migración de un bot de .NET v3 a un bot de Framework v4](conversion-framework.md) |Migrar un bot de v3 a v4 con el mismo tipo de proyecto |
| [Migración de un bot v3 para .NET a un bot v4 para .NET Core](conversion-core.md) | Migrar un bot v3 a v4 en un nuevo proyecto de .NET Core|

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

<!-- _Mini-TOC with explainer for JavaScript topics_ -->
En los temas siguientes se describen las diferencias entre las versiones v3 y v4 del SDK Bot Framework para JavaScript, las diferencias más importantes entre las dos versiones y los pasos para migrar un bot de v3 a v4.

| Tema | DESCRIPCIÓN |
| :--- | :--- |
| [Diferencias entre las versiones v3 y v4 del SDK de JavaScript](migration-about-javascript.md) | Diferencias comunes entre las versiones v3 y v4 del SDK |
| [Referencia rápida de migración de JavaScript](javascript-migration-quickreference.md)| Principales cambios en las versiones v3 y v4 del SDK|
| [Migración de un bot de JavaScript v3 a la versión v4](conversion-javascript.md) |Migrar un bot v3 a v4 |

---

### <a name="code-samples"></a>Ejemplos de código

Los siguientes son ejemplos de código que puede usar para aprender SDK Bot Framework v4 o para comenzar proyecto directamente.

| Ejemplos | DESCRIPCIÓN |
| :--- | :--- |
| [Ejemplos de migración de Bot Framework v3 a v4](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4) <img width="200">| Ejemplos de migración del SDK Bot Framework v3 a v4. |
| [Ejemplos de Bot Builder para .NET](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore) | Ejemplos de Bot Builder para .NET en C#. |
| [Ejemplos de Bot Builder para JavaScript](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs) | Ejemplos de Bot Builder para JavaScript (node.js). |
| [Todos los ejemplos de Bot Builder](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples) | Todos los ejemplos de Bot Builder. |

### <a name="getting-help"></a>Ayuda

Estos recursos ofrecen información adicional y soporte técnico para el desarrollo de bots.

[Recursos adicionales para Bot Framework](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-resources-links-help?view=azure-bot-service-4.0)

### <a name="references"></a>Referencias

Consulte los siguientes recursos para más información.

| Tema | DESCRIPCIÓN |
| :--- | :--- |
| [Novedades de Bot Framework](https://docs.microsoft.com/en-us/azure/bot-service/what-is-new?view=azure-bot-service-4.0) | Principales mejoras y características de Bot Framework y Azure Bot Service|
|[Funcionamiento de los bots](../bot-builder-basics.md)|El mecanismo interno de un bot|
|[Administración de estados](../bot-builder-concept-state.md)|Abstracciones para facilitar la administración de estados|
|[Biblioteca de cuadros de diálogo](../bot-builder-concept-dialog.md)| Conceptos centrales para administrar una conversación|
|[Envío y recepción de mensajes de texto](../bot-builder-howto-send-messages.md)|Principal forma de comunicación de un bot con los usuarios|
|[Enviar elemento multimedia](../bot-builder-howto-add-media-attachments.md)|Archivos multimedia adjuntos, como imágenes, vídeo, audio y archivos| 
|[Flujo de conversación secuencial](../bot-builder-dialog-manage-conversation-flow.md)| Hacer preguntas es la principal forma de interacción de un bot con los usuarios.|
|[Guardar usuario y datos de conversación](../bot-builder-howto-v4-state.md)|Seguimiento de una conversación mientras está sin estado|
|[Flujo complejo](../bot-builder-dialog-manage-complex-conversation-flow.md)|Administración de flujos de conversación complejos |
|[Reutilización de diálogos](../bot-builder-compositcontrol.md)|Crear cuadros de diálogo independientes para controlar escenarios específicos|
|[Interrupciones](../bot-builder-howto-handle-user-interrupt.md)| Controlar las interrupciones para crear un bot sólido|
|[Esquema de la actividad](https://aka.ms/botSpecs-activitySchema)|Esquema para seres humanos y software automatizados|

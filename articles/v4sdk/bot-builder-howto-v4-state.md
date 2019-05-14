---
title: Guardado de los datos del usuario y la conversación | Microsoft Docs
description: Obtenga información sobre cómo guardar y recuperar datos de estado con Bot Framework SDK.
keywords: estado de la conversación, estado de usuario, conversación, guardar el estado, administrar estado del bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/16/19
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 38a8034687ef1a0b8b3bcf3e01d3b33b91bdfd18
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033236"
---
# <a name="save-user-and-conversation-data"></a>Guardado de los datos del usuario y la conversación

[!INCLUDE[applies-to](../includes/applies-to.md)]

Inherentemente, un bot no tiene estado. Una vez implementado el bot, no se puede ejecutar en el mismo proceso o en el mismo equipo de turno a otro. Sin embargo, el bot debe poder hacer un seguimiento del contexto de una conversación, para que pueda controlar su comportamiento y recordar las respuestas a las preguntas anteriores. Las características de almacenamiento y estado de Bot Framework SDK permiten agregar un estado al bot. Los bots usan objetos de almacenamiento y administración de estado para administrar y conservar el estado. El administrador de estados proporciona una capa de abstracción que permite acceder a las propiedades de estado mediante descriptores de acceso de las propiedades, independientemente del tipo de almacenamiento subyacente.

## <a name="prerequisites"></a>Requisitos previos
- Es necesario conocer los [conceptos básicos de los bots](bot-builder-basics.md) y cómo los bots [administran el estado](bot-builder-concept-state.md).
- El código de este artículo se basa en el **ejemplo de bot de administración de estados**. Necesitará una copia del ejemplo en [CSharp](https://aka.ms/statebot-sample-cs) o [JavaScript](https://aka.ms/statebot-sample-js).

## <a name="about-this-sample"></a>Acerca de este ejemplo
Al recibir la entrada del usuario, este ejemplo comprueba el estado de la conversación almacenada para ver si se le ha pedido a este usuario que proporcione su nombre anteriormente. Si no es así, se solicita el nombre del usuario y se almacena esa entrada en el estado del usuario. Si ya se le ha pedido, se usará el nombre almacenado en el estado del usuario para conversar con este y se devolverán sus datos de entrada junto con la hora de recepción y el identificador del canal de entrada, al usuario. Los valores de hora y de identificador del canal se recuperan de los datos de conversación del usuario y se guardan en el estado de conversación. El siguiente diagrama muestra la relación entre el bot, el perfil de usuario y las clases de datos de conversación.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)
![bot de estado de ejemplo](media/StateBotSample-Overview.png)

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
![bot de estado de ejemplo](media/StateBotSample-JS-Overview.png)

---

## <a name="define-classes"></a>Definir las clases

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

El primer paso para configurar la administración de estados es definir las clases que contendrán toda la información que queremos administrar sobre los estados del usuario y de la conversación. En este ejemplo se han definido los siguientes:

- En **UserProfile.cs**, se define una clase `UserProfile` para la información de usuario que recopilará el bot. 
- En **ConversationData.cs**, se define una clase `ConversationData` para controlar el estado de la conversación mientras se recopila información del usuario.

En el ejemplo de código siguiente se muestra la creación de la definición de la clase UserProfile.

**UserProfile.cs** [!code-csharp[UserProfile](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/UserProfile.cs?range=7-11)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El primer paso es solicitar el servicio botbuilder que incluye definiciones de `UserState` y `ConversationState`.

**index.js** [!code-javascript[BotService](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/index.js?range=7-9)]

---

## <a name="create-conversation-and-user-state-objects"></a>Creación de objetos de estado de conversación y usuario

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

A continuación, se registra `MemoryStorage` que se usa para crear objetos `UserState` y `ConversationState`. Los objetos de estado de usuario y conversación se crean en `Startup` y se inserta la dependencia en el constructor del bot. Otros servicios que se registran para un bot son: un proveedor de credenciales, un adaptador y la implementación del bot.

**Startup.cs** [!code-csharp[ConfigureServices](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/Startup.cs?range=16-38)]

**StateManagementBot.cs** [!code-csharp[StateManagement](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=15-22)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

A continuación, se registra `MemoryStorage` que se usa para crear objetos `UserState` y `ConversationState`.

**index.js** [!code-javascript[DefineMemoryStore](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/index.js?range=32-38)]

---

## <a name="add-state-property-accessors"></a>Incorporación de descriptores de acceso de propiedad de estado

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Ahora se crearán descriptores de acceso de propiedad mediante el método `CreateProperty` que proporciona un control sobre el objeto `BotState`. Cada descriptor de acceso a una propiedad de estado permite obtener o establecer el valor de la propiedad de estado correspondiente. Antes de usar nuestras propiedades de estado, usamos cada descriptor de acceso para cargar la propiedad desde el almacenamiento y obtenerla de la caché de estado. Para obtener la clave de ámbito correcto asociada con la propiedad de estado, el método se llamará `GetAsync`.

**StateManagementBot.cs** [!code-csharp[StateAccessors](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=38-46)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

A continuación, se crearán los descriptores de acceso de `UserState` y `ConversationState`. Cada descriptor de acceso a una propiedad de estado permite obtener o establecer el valor de la propiedad de estado correspondiente. El descriptor de acceso se usa para cargar la propiedad asociada del almacenamiento y para recuperar su estado actual de la caché.

**StateManagementBot.js**.
[!code-javascript[BotService](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/bots/stateManagementBot.js?range=6-19)]

---

## <a name="access-state-from-your-bot"></a>Estado de acceso desde el bot
En las secciones anteriores se describen los pasos en tiempo de inicialización para agregar a nuestro bot los descriptores de acceso de las propiedades de estado. Ahora, podemos usar los descriptores de acceso a las propiedades de estado para leer y escribir la información sobre el estado en tiempo de ejecución. El siguiente ejemplo de código usa este flujo de lógica:
- Si userProfile.Name está vacío y conversationData.PromptedUserForName es _true_, se recupera el nombre de usuario proporcionado y se almacena esta información en el estado de usuario.
- Si userProfile.Name está vacío y conversationData.PromptedUserForName es _false_, se solicitará el nombre del usuario.
- Si ya se almacenó userProfile.Name anteriormente, se recuperará la hora del mensaje y el identificador del canal a partir de la entrada del usuario, y se enviarán de nuevo todos los datos al usuario y se almacenarán los datos recuperados en el estado de la conversación.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

**StateManagementBot.cs** [!code-csharp[OnMessageActivityAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=38-85)]

Antes de salir del controlador de turnos, se usará el método _SaveChangesAsync()_ de los objetos de administración de estados para escribir todos los cambios de estados de nuevo en el almacenamiento.

**StateManagementBot.cs** [!code-csharp[OnTurnAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=24-31)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**StateManagementBot.js** [!code-javascript[OnMessage](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/bots/stateManagementBot.js?range=21-54)]

Antes de salir de cada turno de diálogo, se usará el método _saveChanges()_ de los objetos de administración de estados para conservar todos los cambios escribiendo el estado en el almacenamiento.

**StateManagementBot.js** [!code-javascript[OnDialog](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/bots/stateManagementBot.js?range=60-67)]

---

## <a name="test-the-bot"></a>Probar el bot

Descargue e instale la versión más reciente de [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).

1. Ejecute el ejemplo localmente en la máquina. Si necesita instrucciones, consulte el archivo Léame en el ejemplo de [C#](https://aka.ms/statebot-sample-cs) o [JS](https://aka.ms/statebot-sample-js).
1. Use el emulador para probar el bot tal y como se muestra a continuación.

![prueba del bot de estado de ejemplo](media/state-bot-testing-emulator.png)

## <a name="additional-resources"></a>Recursos adicionales

**Privacidad:** Si tiene intención de almacenar datos personales del usuario, debe garantizar el cumplimiento del [Reglamento general de protección de datos](https://blog.botframework.com/2018/04/23/general-data-protection-regulation-gdpr).

**Administración de estados:** Todas las llamadas de administración de estado son asincrónicas y prevalece el último en escribir de forma predeterminada. En la práctica, debe obtener, establecer y guardar el estado lo más próximos en el bot como sea posible.

**Datos empresariales críticos:** Utilice el estado del bot para almacenar las preferencias, el nombre de usuario o lo último que haya solicitado, pero no lo utilice para almacenar datos empresariales críticos. Para los datos críticos, [cree sus propios componentes de almacenamiento](bot-builder-custom-storage.md) o escríbalos directamente en el [almacenamiento](bot-builder-howto-v4-storage.md).

**Recognizer-Text:** El ejemplo usa las bibliotecas Microsoft/Recognizers-Text para analizar y validar la entrada del usuario. Para más información, consulte la página [Información general](https://github.com/Microsoft/Recognizers-Text#microsoft-recognizers-text-overview).

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe cómo configurar el estado para ayudarle a leer y escribir los datos del bot en el almacenamiento, vamos a aprender cómo realizar al usuario una serie de preguntas, validar sus respuestas y guardar su entrada.

> [!div class="nextstepaction"]
> [Creación de mensajes propios para recopilar datos de entrada del usuario](bot-builder-primitive-prompts.md)

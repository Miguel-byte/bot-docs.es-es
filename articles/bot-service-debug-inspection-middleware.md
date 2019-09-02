---
title: Depuración de un bot con middleware de inspección | Microsoft Docs
description: Aprenda a depurar un bot con middleware de inspección.
author: zxyanliu
ms.author: v-liyan
keywords: Bot Framework SDK, debug bot, inspection middleware, bot emulator, Azure Bot Channels Registration
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 7/9/2019
ms.openlocfilehash: fe96131a7087f3f2c4980fe4f2eacb94a4ae9e4a
ms.sourcegitcommit: c200cc2db62dbb46c2a089fb76017cc55bdf26b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2019
ms.locfileid: "70037510"
---
# <a name="debug-a-bot-with-inspection-middleware"></a>Depuración de un bot con middleware de inspección
En este artículo se describe cómo depurar el bot mediante middleware de inspección. Esta característica permite a Bot Framework Emulator depurar el tráfico dentro y fuera del bot, además de examinar el estado actual de este. Puede usar un mensaje de seguimiento para enviar datos al emulador y, a continuación, inspeccionar el estado del bot en cualquier turno determinado de la conversación. 

Se usa un EchoBot compilado localmente mediante Bot Framework v4 ([C#](https://docs.microsoft.com/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0) | [JavaScript](https://docs.microsoft.com/azure/bot-service/javascript/bot-builder-javascript-quickstart?view=azure-bot-service-4.0)) para mostrar cómo depurar e inspeccionar el estado del mensaje de bot. También puede [depurar mediante IDE](./bot-service-debug-bot.md) o con [Bot Framework Emulator](./bot-service-debug-emulator.md) pero para depurar el estado necesitará agregar un middleware de inspección al bot. Los ejemplos de bot de inspección están disponibles aquí: [C#](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/47.inspection) y [JavaScript](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/47.inspection). 

## <a name="prerequisites"></a>Requisitos previos
- Descargar e instalar [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started)
- Conocimientos del [middleware](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-middleware?view=azure-bot-service-4.0) del bot
- Conocimientos sobre el [estado de administración](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-state?view=azure-bot-service-4.0) del bot
- Descargar e instalar [ngrok](https://ngrok.com/) (si quiere depurar un bot configurado en Azure para usar canales adicionales)

## <a name="update-your-emulator-to-the-latest-version"></a>Actualización del emulador a la versión más reciente 
Antes de usar el middleware de inspección para depurar el bot, debe actualizar el emulador a la versión 4.5 o posterior. Compruebe la [versión más reciente](https://github.com/Microsoft/BotFramework-Emulator/releases) para buscar actualizaciones. 

Para comprobar la versión del emulador, seleccione **Ayuda** -> **Acerca de** en el menú. Aparecerá la versión actual del emulador. 

![current-version](./media/bot-debug-inspection-middleware/bot-debug-check-emulator-version.png) 

## <a name="update-your-bots-code"></a>Actualización del código del bot

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
Configure el estado de inspección en el archivo de **inicio**. Agregue el middleware de inspección al adaptador. El estado de inspección se proporciona mediante la inserción de dependencias. Vea la siguiente actualización del código o consulte este ejemplo de inspección aquí: [C#](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/47.inspection). 

**Startup.cs** [!code-csharp [inspection bot sample](~/../botbuilder-samples/samples/csharp_dotnetcore/47.inspection/Startup.cs?range=17-37)]

**AdapterWithInspection.cs**  
[!code-csharp [inspection bot sample](~/../botbuilder-samples/samples/csharp_dotnetcore/47.inspection/AdapterwithInspection.cs?range=11-21)]

**EchoBot.cs** [!code-csharp [inspection bot sample](~/../botbuilder-samples/samples/csharp_dotnetcore/47.inspection/Bots/EchoBot.cs?range=14-43)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
Antes de actualizar el código del bot, debe actualizar sus paquetes a las versiones más recientes ejecutando el siguiente comando en el terminal: 
```cmd
npm install --save botbuilder@latest 
```  
Después, debe actualizar el código de su bot de JavaScript como se indica a continuación. Puede leer más aquí: [Actualización del código del bot](https://github.com/Microsoft/BotFramework-Emulator/blob/master/content/CHANNELS.md#1-update-your-bots-code) o consultar el ejemplo de inspección aquí: [JavaScript](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/47.inspection). 

**index.js**

Configure el estado de inspección y agregue el middleware de inspección al adaptador en el archivo **index.js**. 

[!code-javascript [inspection bot sample](~/../botbuilder-samples/samples/javascript_nodejs/47.inspection/index.js?range=10-43)]

**bot.js**

Actualice la clase del bot en el archivo **bot.js**. 

[!code-javascript [inspection bot sample](~/../botbuilder-samples/samples/javascript_nodejs/47.inspection/bot.js?range=6-50)]

---

## <a name="test-your-bot-locally"></a>Prueba local del bot 
Después de actualizar el código, puede ejecutar el bot localmente y probar la característica de depuración con dos emuladores: uno para enviar y recibir mensajes, y el otro para inspeccionar el estado de los mensajes en el modo de depuración. Para probar el bot localmente, realice los pasos siguientes: 

1. Vaya al directorio del bot en un terminal y ejecute el siguiente comando para ejecutar el bot localmente: 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cmd
dotnet run
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```cmd
npm start 
```

---

2. Abra el emulador. Haga clic en **Open Bot** (Abrir bot). Rellene la dirección URL del bot con http://localhost:3978/api/messages y los valores **MicrosoftAppId** y **MicrosoftAppPassword**. Si tiene un bot de JavaScript, puede encontrar estos valores en el archivo **.env** del bot. Si tiene un bot de C# , puede encontrar estos valores en el archivo **appsettings.json**. Haga clic en **Conectar**. 

3. Ahora abra otro emulador. Este segundo emulador funcionará como un depurador. Siga las instrucciones que se describen en el paso anterior. Seleccione **Open in debug mode** (Abrir en modo de depuración) y haga clic en **Conectar**. 

4. En este momento, verá un UUID (`/INSPECT attach <identifier>`) en el emulador de depuración. Copie el UUID y péguelo en el cuadro de chat del primer emulador. 

> [!NOTE]
> Un identificador único universal (UUID) es un identificador único que identifica la información. Se genera un UUID cada vez que el emulador se inicia en modo de depuración después de agregar el middleware de inspección al código del bot. 

5. Ahora puede enviar mensajes en el cuadro de chat del primer emulador e inspeccionar los mensajes en el emulador de depuración. Para inspeccionar el estado de los mensajes haga clic en **Bot State** en el emulador de depuración y despliegue los **valores** en la ventana **JSON** situada a la derecha. Podrá ver el estado del bot de la siguiente manera: ![estado del bot](./media/bot-debug-inspection-middleware/bot-debug-bot-state.png)

## <a name="inspect-the-state-of-a-bot-configured-in-azure"></a>Inspección del estado de un bot configurado en Azure 
Si desea inspeccionar el estado de un bot configurado en Azure y conectado a canales (como Teams), deberá instalar y ejecutar [ngrok](https://ngrok.com/).

### <a name="run-ngrok"></a>Ejecución de ngrok
En este punto, ha actualizado el emulador a la versión más reciente y ha agregado el middleware de inspección al código del bot. El siguiente paso consiste en ejecutar ngrok y configurar el bot local en el registro de canales del bot de Azure. Antes de ejecutar ngrok, debe ejecutar el bot localmente. 

Para ejecutar el bot localmente, haga lo siguiente: 
1. Vaya a la carpeta del bot en un terminal y configure el registro npm para que use las [versiones más recientes](https://botbuilder.myget.org/feed/botbuilder-v4-js-daily/package/npm/botbuilder-azure).

2. Ejecute el bot localmente. Verá que el bot expone un número de puerto como, por ejemplo, el 3978. 

3. Abra otro símbolo del sistema y vaya a la carpeta de proyecto del bot. Ejecute el siguiente comando:
```
ngrok http 3978
```
4. ngrok ahora está conectado al bot que se ejecuta localmente. Copie la dirección IP pública. 
![ngrok-success](./media/bot-debug-inspection-middleware/bot-debug-ngrok.png)

### <a name="update-channel-registrations-for-your-bot"></a>Actualización de los registros de canales del bot
Ahora que el bot local está conectado a ngrok puede configurarlo en el registro de canales del bot en Azure.

1. Vaya al registro de canales del bot en Azure. Haga clic en **Configuración** en el menú de la izquierda y establezca **Messaging endpoint** (Punto de conexión de la mensajería) en la dirección IP de ngrok. Si es necesario, agregue **/api/messages** después de la dirección IP. (Por ejemplo: https://e58549b6.ngrok.io/api/messages). Seleccione **Enable Streaming Endpoint** (Habilitar punto de conexión de streaming) y **Guardar**.
![endpoint](./media/bot-debug-inspection-middleware/bot-debug-channels-setting-ngrok.png)
> [!TIP]
> Si **Guardar** no está habilitado, puede anular la selección de **Enable Streaming Endpoint** (Habilitar punto de conexión de streaming) y hacer clic en **Guardar** y, a continuación, seleccionar **Enable Streaming Endpoint** y hacer clic en **Guardar** de nuevo. Asegúrese de que **Enable Streaming Endpoint** (Habilitar punto de conexión de streaming) está seleccionado y se guardará la configuración del punto de conexión. 

2. Vaya al grupo de recursos del bot, haga clic en **Implementación** y seleccione el registro de canales de bot que implementó previamente de manera correcta. Haga clic en **Entradas** situado a la izquierda y obtenga los valores de **appId** y **appSecret**. Actualice el archivo **.env** del bot (o el archivo **appsettings.json** si tiene un bot de C#) con los valores de **appId** y **appSecret**. 
![get-inputs](./media/bot-debug-inspection-middleware/bot-debug-get-inputs-id-secret.png)

3. Inicie el emulador, haga clic en **Open Bot** (Abrir bot), y ponga http://localhost:3978/api/messages en la **dirección URL del bot**. Rellene **Microsoft App ID** (Id. de aplicación de Microsoft) y **Microsoft App password** (Contraseña de aplicación de Microsoft) con los mismos valores de **appId** y **appSecret** que agregó al archivo **.env** (o **appsettings.json**) del bot. Haga clic en **Conectar**. 

4. El bot en ejecución está conectado ahora al registro de canales de bot en Azure. Puede probar el chat en web haciendo clic en **Test in Web Chat** (Probar en chat en web) y enviando mensajes en el cuadro de chat. 
![test-web-chat](./media/bot-debug-inspection-middleware/bot-debug-test-webchat.png)

5. Ahora vamos a habilitar el modo de depuración en el emulador. En el emulador, seleccione **Depurar** -> **Iniciar depuración**. Rellene la dirección IP de ngrok (no olvide agregar **/API/messages**) en la **dirección URL de bot** (por ejemplo, https://e58549b6.ngrok.io/api/messages) ). Rellene **Microsoft App ID** (Id. de aplicación de Microsoft) con **appId** y **Microsoft App password** (Contraseña de aplicación de Microsoft) con **appSecret**. Asegúrese de que **Open in debug mode** (Abrir en modo de depuración) también está seleccionado. Haga clic en **Conectar**. 

6. Cuando el modo de depuración está habilitado, se generará un UUID en el emulador. Un UUID es un identificador único que se genera cada vez que se inicia el modo de depuración en el emulador. Copie y pegue el UUID en el cuadro de chat de **Test in Web Chat** (Probar en chat en web) del cuadro de chat del canal. Aparecerá el mensaje "Attached to session, all traffic is being replicated for inspection" (Conectado a la sesión, todo el tráfico se replicará para su inspección) en el cuadro de chat. 

 Puede empezar a depurar el bot enviando mensajes en el cuadro de chat del canal configurado. El emulador local actualizará automáticamente los mensajes con todos los detalles de la depuración. Para inspeccionar el estado de los mensajes del bot, haga clic en **Bot State** y despliegue los **valores** en la ventana JSON correcta. 

 ![debug-inspection-middleware](./media/bot-debug-inspection-middleware/debug-state-inspection-channel-chat.gif)

## <a name="additional-resources"></a>Recursos adicionales
- Pruebe los nuevos ejemplos del bot de inspección aquí: [C#](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/47.inspection) y [JavaScript](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/47.inspection). 
- Lea [Solución de problemas generales](bot-service-troubleshoot-bot-configuration.md) y otros artículos de solución de problemas en esa sección.
- Lea el artículo [Depuración con el emulador](bot-service-debug-emulator.md).

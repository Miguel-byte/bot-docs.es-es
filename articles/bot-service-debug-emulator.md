---
title: Prueba y depuración de bots con Bot Framework Emulator | Microsoft Docs
description: Obtenga información sobre cómo inspeccionar, probar y depurar bots con la aplicación de escritorio Bot Framework Emulator.
keywords: transcripción, herramienta msbot, servicios de lenguaje, reconocimiento de voz
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/26/2019
ms.openlocfilehash: 847ae51791ae66ef190ebefee765f2806ec91c5e
ms.sourcegitcommit: 23a1808e18176f1704f2f6f2763ace872b1388ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/25/2019
ms.locfileid: "68484038"
---
# <a name="debug-with-the-emulator"></a>Depuración con el emulador

Bot Framework Emulator es una aplicación de escritorio que permite que los desarrolladores de bots prueben y depuren sus bots de manera local o remota. Con el emulador, puede chatear con el bot e inspeccionar los mensajes que este envía y recibe. El emulador muestra los mensajes tal como aparecerían en la interfaz de usuario de un chat web y registra las respuestas y las solicitudes de JSON a medida que intercambia mensajes con su bot. Antes de implementar el bot en la nube, ejecútelo localmente y pruébelo con el emulador. Puede probar el bot con el emulador incluso si todavía no lo ha [creado](./bot-service-quickstart.md) con Azure Bot Service ni lo ha configurado para que se ejecute en ningún canal.

## <a name="prerequisites"></a>Requisitos previos
- Instale [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started)

## <a name="run-a-bot-locally"></a>Ejecución local de un bot
Antes de conectar el bot a Bot Framework Emulator, debe ejecutar el bot localmente. Puede usar Visual Studio o Visual Studio Code para ejecutar el bot, o bien usar la línea de comandos. Para ejecutar un bot mediante la línea de comandos, haga lo siguiente:


# <a name="ctabcsharp"></a>[C#](#tab/csharp)

* En el símbolo del sistema, cambie el directorio al del proyecto del bot.
* Para iniciar el bot, ejecute el siguiente comando: 
    ```
    dotnet run
    ```
* Copie el número de puerto que se indica la línea situada antes de *Application started. Press CTRL+C to shut down.* (Aplicación iniciada. Presione Ctrl+C para apagar).

    ![Número de puerto (C#)](media/bot-service-debug-emulator/csharp_port_number.png)


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

* En el símbolo del sistema, cambie el directorio al del proyecto del bot.
* Para iniciar el bot, ejecute el siguiente comando:
    ```
    node index.js
    ```
* Copie el número de puerto en el que restify escucha.

    ![Número de puerto (JS)](media/bot-service-debug-emulator/js_port_number.png)
---

En este momento, el bot se ejecuta de forma local. 


## <a name="connect-to-a-bot-running-on-localhost"></a>Una conexión a un bot que se ejecuta en un host local.

![UI del emulador](media/emulator-v4/emulator-welcome.png)

Para conectarse a un bot que se ejecuta localmente, haga clic en **Abrir bot**. Agregue el número de puerto que copió anteriormente en la siguiente dirección URL y pegue la dirección URL actualizada en la barra de direcciones URL del bot:

*http://localhost:**número de puerto**/api/messages*

![UI del emulador](media/bot-service-debug-emulator/open_bot_emulator.png)

Si el bot se está ejecutando con [credenciales de una cuenta Microsoft (MSA)](#use-bot-credentials), escriba también estas credenciales.


### <a name="use-bot-credentials"></a>Usar credenciales de bot

Cuando abra el bot, establezca el **id. de aplicación de Microsoft** y la **contraseña de aplicación de Microsoft** si su bot se ejecuta con credenciales. Si ha creado su bot con Azure Bot Service, las credenciales están disponibles en la instancia de App Service del bot, en la sección **Configuración -> Configuración**. Si no conoce los valores, puede quitarlos del archivo de configuración del bot que se ejecuta localmente y luego ejecutar el bot en el emulador. Si el bot no se ejecuta con esta configuración, tampoco es necesario ejecutar el emulador con la configuración. 

## <a name="view-detailed-message-activity-with-the-inspector"></a>Vista detallada de la actividad de mensaje con el inspector

Envíe un mensaje al bot y este debería responder. Puede hacer clic en una burbuja de mensaje dentro de la ventana de la conversación e inspeccionar la actividad JSON sin formato con la característica **INSPECTOR** que se encuentra a la derecha de la ventana. Cuando se selecciona, la burbuja de mensaje cambia a color amarillo y el objeto JSON de la actividad se mostrará a la izquierda de la ventana del chat. Esta información de JSON incluye metadatos de clave que incluyen el identificador de canal, el tipo de actividad, el identificador de la conversación, el mensaje de texto, la dirección URL del punto de conexión, etc. Puede inspeccionar las actividades de inspección enviadas desde el usuario, así como también las actividades con las que responde el bot.

![Actividad de mensaje del emulador](media/emulator-v4/emulator-view-message-activity-03.png)

> [!TIP]
> Para depurar los cambios de estado en un bot conectado a un canal, puede agregar [middleware de inspección](bot-service-debug-inspection-middleware.md) al bot.

<!--
## Save and load conversations with bot transcripts

Activities in the emulator can be saved as transcripts. From an open live chat window, select **Save Transcript As** to the transcript file. The **Start Over** button can be used any time to clear a conversation and restart a connection to the bot.  

![Emulator save transcripts](media/emulator-v4/emulator-save-transcript.png)

To load transcripts, simply select **File > Open Transcript File** and select the transcript. A new Transcript window will open and render the message activity to the output window. 

![Emulator load transcripts](media/emulator-v4/emulator-load-transcript.png)
--->
<!---
## Add services 

You can easily add a LUIS app, QnA knowledge base, or dispatch model to your bot directly from the emulator. When the bot is loaded, select the services button on the far left of the emulator window. You will see options under the **Services** menu to add LUIS, QnA Maker, and Dispatch. 

To add a service app, simply click on the **+** button and select the service you want to add. You will be prompted to sign in to the Azure portal to add the service to the bot file, and connect the service to your bot application. 

> [!IMPORTANT]
> Adding services only works if you're using a `.bot` configuration file. Services will need to be added independently. For details on that, see [Manage bot resources](v4sdk/bot-file-basics.md) or the individual how to articles for the service you're trying to add.
>
> If you are not using a `.bot` file, the left pane won't have your services listed (even if your bot uses services) and will display *Services not available*.

![LUIS connect](media/emulator-v4/emulator-connect-luis-btn.png)

When either service is connected, you can go back to a live chat window and verify that your services are connected and working. 

![QnA connected](media/emulator-v4/emulator-view-message-activity.png)

--->

## <a name="inspect-services"></a>Inspección de servicios

Con el nuevo emulador v4 también puede inspeccionar las respuestas de JSON provenientes de LUIS y QnA. Mediante un bot con un servicio de lenguaje conectado, puede seleccionar **trace** en la ventana LOG (Registro) en la esquina inferior derecha. Esta herramienta nueva también proporciona características para actualizar los servicios de lenguaje directamente desde el emulador. 

![Inspector de LUIS](media/emulator-v4/emulator-luis-inspector.png)

Con un servicio de LUIS conectado, verá que el vínculo de seguimiento especifica el **seguimiento de LUIS**. Cuando se seleccione, verá la respuesta sin formato proveniente del servicio de LUIS, que incluye intenciones y entidades junto con sus puntuaciones especificadas. También tiene la opción de volver a asignar intenciones a las expresiones del usuario. 

![Inspector de QnA](media/emulator-v4/emulator-qna-inspector.png)

Con un servicio de QnA conectado, el registro mostrará el **seguimiento de QnA** y cuando esté seleccionado, podrá obtener una vista previa del par de pregunta y respuesta asociado con esa actividad, junto con una puntuación de confianza. Desde aquí, puede agregar frases de preguntas alternativas para una respuesta.

<!--## Configure ngrok

If you are using Windows and you are running the Bot Framework Emulator behind a firewall or other network boundary and want to connect to a bot that is hosted remotely, you must install and configure **ngrok** tunneling software. The Bot Framework Emulator integrates tightly with ngrok tunnelling software (developed by [inconshreveable][inconshreveable]), and can launch it automatically when it is needed.

Open the **Emulator Settings**, enter the path to ngrok, select whether or not to bypass ngrok for local addresses, and click **Save**.

![ngrok path](media/emulator-v4/emulator-ngrok-path.png)
-->

<!---## Login to Azure

You can use Emulator to login in to your Azure account. This is particularly helpful for you to add and manage services your bot depends on. 
See [above](#add-services) to learn more about services you can manage using the Emulator.
-->

### <a name="login-to-azure"></a>Inicio de sesión en Azure
Puede usar Emulator para iniciar sesión en la cuenta de Azure. Esto es especialmente útil para agregar y administrar los servicios de los que depende el bot. Siga estos pasos para iniciar sesión en Azure:
- Haga clic en Archivo -> Iniciar sesión con Azure ![Inicio de sesión de Azure](media/emulator-v4/emulator-azure-login.png)
- En la pantalla de inicio de sesión haga clic en Iniciar sesión con su cuenta de Azure. Opcionalmente, puede hacer que Emulator conserve su sesión iniciada entre varios reinicios de esta aplicación.
![Inicio de sesión de Azure](media/emulator-v4/emulator-azure-login-success.png)

## <a name="disabling-data-collection"></a>Deshabilitación de la recopilación de datos

Si decide que ya no desea permitir que Emulator recopile datos de uso, puede deshabilitar fácilmente la recopilación de datos siguiendo estos pasos:

1. Vaya a la página de configuración de Emulator. Para ello, haga clic en el botón de configuración (icono de engranaje) en la barra de navegación del lado izquierdo.

    ![deshabilitar la recopilación de datos](media/emulator-v4/emulator-disable-data-1.png)

2. Desactive la casilla denominada *Help improve the Emulator by allowing us to collect usage data* (Ayúdenos a mejorar Emulator permitiéndonos recopilar datos de uso) en la sección **Recopilación de datos**.

    ![deshabilitar la recopilación de datos](media/emulator-v4/emulator-disable-data-2.png)

3. Haga clic en el botón “Save” (Guardar).

    ![deshabilitar la recopilación de datos](media/emulator-v4/emulator-disable-data-3.png)
    
Si cambia de opinión, puede habilitar esta opción siempre que quiera volviendo a activar la casilla.

## <a name="additional-resources"></a>Recursos adicionales

Bot Framework Emulator es de código abierto. También puede [contribuir][EmulatorGithubContribute] to the development and [submit bugs and suggestions][EmulatorGithubBugs].

Para solucionar el problema, consulte cómo [solucionar problemas generales](bot-service-troubleshoot-bot-configuration.md) y otros artículos de solución de problemas en esa sección.

## <a name="next-steps"></a>Pasos siguientes

Use el middleware de inspección para depurar un bot conectado a un canal.

> [!div class="nextstepaction"]
> [Depuración del bot mediante archivos de transcripción](bot-service-debug-inspection-middleware.md)

<!--
Saving a conversation to a transcript file allows you to quickly draft and replay a certain set of interactions for debugging.

> [!div class="nextstepaction"]
> [Debug your bot using transcript files](~/v4sdk/bot-builder-debug-transcript.md)
-->

<!-- Footnote-style URLs -->

[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/

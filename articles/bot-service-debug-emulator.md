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
ms.openlocfilehash: 0e548700e81fff5029031fd1e349cc75d9d0bc7a
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464643"
---
# <a name="debug-with-the-emulator"></a>Depuración con el emulador

Bot Framework Emulator es una aplicación de escritorio que permite que los desarrolladores de bots prueben y depuren sus bots de manera local o remota. Con el emulador, puede chatear con el bot e inspeccionar los mensajes que este envía y recibe. El emulador muestra los mensajes tal como aparecerían en la interfaz de usuario de un chat web y registra las respuestas y las solicitudes de JSON a medida que intercambia mensajes con su bot. Antes de implementar el bot en la nube, ejecútelo localmente y pruébelo con el emulador. Puede probar el bot con el emulador incluso si todavía no lo ha [creado](./bot-service-quickstart.md) con Azure Bot Service ni lo ha configurado para que se ejecute en ningún canal.

## <a name="prerequisites"></a>Requisitos previos
- Instale el [emulador](https://aka.ms/Emulator-wiki-getting-started).

## <a name="connect-to-a-bot-running-on-localhost"></a>Una conexión a un bot que se ejecuta en un host local.

![UI del emulador](media/emulator-v4/emulator-welcome.png)

Para conectarse a un bot que se ejecuta localmente, haga clic en **Open bot** (Abrir bot) o seleccione el archivo de configuración preconfigurado (un archivo .bot). No es necesario un archivo de configuración para conectarse al bot, pero el emulador seguirá funcionando con uno si el bot lo tiene. Si el bot se está ejecutando con [credenciales de una cuenta Microsoft (MSA)](#use-bot-credentials), escriba también estas credenciales.

![UI del emulador](media/emulator-v4/emulator-open-bot.png)

### <a name="use-bot-credentials"></a>Usar credenciales de bot

Cuando abra el bot, establezca el **id. de aplicación de Microsoft** y la **contraseña de aplicación de Microsoft** si su bot se ejecuta con credenciales. Si ha creado su bot con Azure Bot Service, las credenciales están disponibles en la instancia de App Service del bot, en la sección **Configuración -> Configuración**. Si no conoce los valores, puede quitarlos del archivo de configuración del bot que se ejecuta localmente y luego ejecutar el bot en el emulador. Si el bot no se ejecuta con esta configuración, tampoco es necesario ejecutar el emulador con la configuración. 

## <a name="view-detailed-message-activity-with-the-inspector"></a>Vista detallada de la actividad de mensaje con el inspector

Envíe un mensaje al bot y este debería responder. Puede hacer clic en una burbuja de mensaje dentro de la ventana de la conversación e inspeccionar la actividad JSON sin formato con la característica **INSPECTOR** que se encuentra a la derecha de la ventana. Cuando se selecciona, la burbuja de mensaje cambia a color amarillo y el objeto JSON de la actividad se mostrará a la izquierda de la ventana del chat. Esta información de JSON incluye metadatos de clave que incluyen el identificador de canal, el tipo de actividad, el identificador de la conversación, el mensaje de texto, la dirección URL del punto de conexión, etc. Puede inspeccionar las actividades de inspección enviadas desde el usuario, así como también las actividades con las que responde el bot. 

![Actividad de mensaje del emulador](media/emulator-v4/emulator-view-message-activity-03.png)

## <a name="save-and-load-conversations-with-bot-transcripts"></a>Guardado y carga de conversaciones con transcripciones de bots

Las actividades del emulador se pueden guardar como transcripciones. En una ventana de chat en directo abierta, seleccione **Save Transcript As** (Guardar transcripción como) en el archivo de transcripción. El botón **Start Over** (Volver a empezar) se puede usar en cualquier momento para borrar una conversación y reiniciar una conexión con el bot.  

![Guardar las transcripciones en el emulador](media/emulator-v4/emulator-save-transcript.png)

Para cargar las transcripciones, simplemente seleccione **Archivo > Open Transcript File** (Abrir archivo de transcripción) y seleccione la transcripción. Se abrirá una ventana de transcripción nueva que presentará la actividad de mensaje en la ventana de salida. 

![Cargar transcripciones en el emulador](media/emulator-v4/emulator-load-transcript.png)

## <a name="add-services"></a>Agregar servicios 

Puede agregar fácilmente una aplicación de LUIS o una base de conocimientos de QnA o un modelo de envío en el bot directamente desde el emulador. Cuando el bot se cargue, seleccione el botón de servicios en el extremo izquierdo de la ventana del emulador. En el menú **Servicios** verá opciones para agregar LUIS, QnA Maker y Dispatch. 

Para agregar una aplicación de servicio, solo tiene que hacer clic en el botón **+** y seleccionar el servicio que desea agregar. Se le pedirá que inicie sesión en Azure Portal para agregar el servicio al archivo de bot y conectar el servicio a la aplicación bot. 

> [!IMPORTANT]
> La adición de servicios solo funciona si usa un archivo de configuración `.bot`. Se deben agregar los servicios de manera independiente. Para más información sobre esto, consulte [Administración de recursos de bot](v4sdk/bot-file-basics.md) o los artículos de procedimientos específicos del servicio que está intentando agregar.
>
> Si no usa un archivo `.bot`, el panel izquierdo no mostrará los servicios (incluso aunque el bot los use) y aparecerá un mensaje que indica que los *servicios no están disponibles*.

![Conexión de LUIS](media/emulator-v4/emulator-connect-luis-btn.png)

Cuando se conecta cualquiera de esos servicios, puede volver a una ventana de chat activo y comprobar que los servicios están conectados y en funcionamiento. 

![QnA conectado](media/emulator-v4/emulator-view-message-activity.png)

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

## <a name="login-to-azure"></a>Inicio de sesión en Azure

Puede usar Emulator para iniciar sesión en la cuenta de Azure. Esto es especialmente útil para agregar y administrar los servicios de los que depende el bot. Consulte la información [anterior](#add-services) para más información acerca de los servicios que puede administrar mediante Emulator.

### <a name="to-login"></a>Para iniciar sesión

![Inicio de sesión de Azure](media/emulator-v4/emulator-azure-login.png)

Para iniciar sesión
- Puede hacer clic en Archivo -> Iniciar sesión con Azure
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

Guardar una conversación en un archivo de transcripción le permite crear rápidamente un borrador y reproducir un determinado conjunto de interacciones para la depuración.

> [!div class="nextstepaction"]
> [Depuración del bot mediante archivos de transcripción](~/v4sdk/bot-builder-debug-transcript.md)

<!-- Footnote-style URLs -->

[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/

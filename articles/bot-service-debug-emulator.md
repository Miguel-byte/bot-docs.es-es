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
ms.openlocfilehash: 2266dcb936205a20e1d19d97983a3b802fbe2736
ms.sourcegitcommit: cf3786c6e092adec5409d852849927dc1428e8a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/01/2019
ms.locfileid: "57224873"
---
# <a name="debug-with-the-emulator"></a>Depuración con el emulador

Bot Framework Emulator es una aplicación de escritorio que permite que los desarrolladores de bots prueben y depuren sus bots de manera local o remota. Con el emulador, puede chatear con el bot e inspeccionar los mensajes que este envía y recibe. El emulador muestra los mensajes tal como aparecerían en la interfaz de usuario de un chat web y registra las respuestas y las solicitudes de JSON a medida que intercambia mensajes con su bot. Antes de implementar el bot en la nube, ejecútelo localmente y pruébelo con el emulador. Puede probar el bot con el emulador incluso si todavía no lo ha [creado](./bot-service-quickstart.md) con Azure Bot Service ni lo ha configurado para que se ejecute en ningún canal.

## <a name="prerequisites"></a>Requisitos previos
- Instale el [emulador](https://aka.ms/Emulator-wiki-getting-started).
- Instale el software de tunelización [ngrok][ngrokDownload].

## <a name="connect-to-a-bot-running-on-localhost"></a>Una conexión a un bot que se ejecuta en un host local.

Para conectarse a un bot que se ejecuta localmente, haga clic en **Abrir bot** y seleccione el archivo .bot. 

![UI del emulador](media/emulator-v4/emulator-welcome.png)

## <a name="view-detailed-message-activity-with-the-inspector"></a>Vista detallada de la actividad de mensaje con el inspector

Envíe un mensaje al bot y este debería responder. Puede hacer clic en una burbuja de mensaje dentro de la ventana de la conversación e inspeccionar la actividad JSON sin formato con la característica **INSPECTOR** que se encuentra a la derecha de la ventana. Cuando se selecciona, la burbuja de mensaje cambia a color amarillo y el objeto JSON de la actividad se mostrará a la izquierda de la ventana del chat. Esta información de JSON incluye metadatos de clave que incluyen el identificador de canal, el tipo de actividad, el identificador de la conversación, el mensaje de texto, la dirección URL del punto de conexión, etc. Puede inspeccionar las actividades de inspección enviadas desde el usuario, así como también las actividades con las que responde el bot. 

![Actividad de mensaje del emulador](media/emulator-v4/emulator-view-message-activity-02.png)

## <a name="save-and-load-conversations-with-bot-transcripts"></a>Guardado y carga de conversaciones con transcripciones de bots

Las actividades del emulador se pueden guardar como transcripciones. En una ventana de chat en directo abierta, seleccione **Save Transcript As** (Guardar transcripción como) en el archivo de transcripción. El botón **Start Over** (Volver a empezar) se puede usar en cualquier momento para borrar una conversación y reiniciar una conexión con el bot.  

![Guardar las transcripciones en el emulador](media/emulator-v4/emulator-live-chat.png)

Para cargar las transcripciones, simplemente seleccione **Archivo > Open Transcript File** (Abrir archivo de transcripción) y seleccione la transcripción. Se abrirá una ventana de transcripción nueva que presentará la actividad de mensaje en la ventana de salida. 

![Cargar transcripciones en el emulador](media/emulator-v4/emulator-load-transcript.png)

## <a name="add-services"></a>Agregar servicios 

Puede agregar fácilmente una aplicación de LUIS o una base de conocimientos de QnA o un modelo de envío en el archivo .bot directamente desde el emulador. Cuando el archivo .bot se cargue, seleccione el botón de servicios en el extremo izquierdo de la ventana del emulador. En el menú **Servicios** verá opciones para agregar LUIS, QnA Maker y Dispatch. 

Para agregar una aplicación de servicio, solo tiene que hacer clic en el botón **+** y seleccionar el servicio que desea agregar. Se le pedirá que inicie sesión en Azure Portal para agregar el servicio al archivo de bot y conectar el servicio a la aplicación bot. 

![Conexión de LUIS](media/emulator-v4/emulator-connect-luis-btn.png)

Cuando se conecta cualquiera de esos servicios, puede volver a una ventana de chat activo y comprobar que los servicios están conectados y en funcionamiento. 

![QnA conectado](media/emulator-v4/emulator-view-message-activity.png)

## <a name="inspect-services"></a>Inspección de servicios

Con el nuevo emulador v4 también puede inspeccionar las respuestas de JSON provenientes de LUIS y QnA. Mediante un bot con un servicio de lenguaje conectado, puede seleccionar **trace** en la ventana LOG (Registro) en la esquina inferior derecha. Esta herramienta nueva también proporciona características para actualizar los servicios de lenguaje directamente desde el emulador. 

![Inspector de LUIS](media/emulator-v4/emulator-luis-inspector.png)

Con un servicio de LUIS conectado, verá que el vínculo de seguimiento especifica el **seguimiento de LUIS**. Cuando se seleccione, verá la respuesta sin formato proveniente del servicio de LUIS, que incluye intenciones y entidades junto con sus puntuaciones especificadas. También tiene la opción de volver a asignar intenciones a las expresiones del usuario. 

![Inspector de QnA](media/emulator-v4/emulator-qna-inspector.png)

Con un servicio de QnA conectado, el registro mostrará el **seguimiento de QnA** y cuando esté seleccionado, podrá obtener una vista previa del par de pregunta y respuesta asociado con esa actividad, junto con una puntuación de confianza. Desde aquí, puede agregar frases de preguntas alternativas para una respuesta.

## <a name="configure-ngrok"></a>Configuración de ngrok

Si usa Windows y ejecuta Bot Framework Emulator detrás de un firewall o de otro límite de red y quiere conectarse a un bot hospedado remotamente, debe instalar y configurar el software de tunelización **ngrok**. Bot Framework Emulator se integra estrechamente con el software de tunelización ngrok (desarrollado por [inconshreveable][inconshreveable]) y puede iniciarlo automáticamente cuando sea necesario.

Abra **Configuración del emulador**, escriba la ruta de acceso a ngrok, seleccione si omitir o no ngrok para las direcciones locales y haga clic en **Guardar**.

![ruta de acceso a ngrok](media/emulator-v4/emulator-ngrok-path.png)

## <a name="additional-resources"></a>Recursos adicionales

Bot Framework Emulator es de código abierto. También puede [colaborar][EmulatorGithubContribute] en el desarrollo y [enviar errores y sugerencias][EmulatorGithubBugs].

Para solucionar el problema, consulte cómo [solucionar problemas generales](bot-service-troubleshoot-bot-configuration.md) y otros artículos de solución de problemas en esa sección.

<!-- Footnote-style URLs -->

[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/

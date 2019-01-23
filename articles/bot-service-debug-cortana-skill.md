---
title: Probar una habilidad de Cortana | Microsoft Docs
description: Aprenda a probar un bot de Cortana invocando una habilidad de Cortana.
keywords: Bot Framework SDK, registrar el bot, cortana
author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/01/18
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: acf4f73ea83646878e08d14e036d0d2d66686756
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224960"
---
# <a name="test-a-cortana-skill"></a>Probar una habilidad de Cortana

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]
 
Si ha creado una habilidad de Cortana con Bot Framework SDK, puede probarla invocándola desde Cortana. Las siguientes instrucciones le guiarán a través de los pasos necesarios para probar una habilidad de Cortana.

## <a name="register-your-bot"></a>Registrar el bot
Si [creó el bot](~/bot-service-quickstart.md) usando Bot Service en Azure, entonces ya estará registrado y puede omitir este paso.

Si implementó el bot en otro lugar o si quiere probar el bot de forma local, debe [registrarlo](bot-service-quickstart-registration.md) para que pueda conectarse a Cortana. Durante el proceso de registro, deberá proporcionar el **punto de conexión de mensajería** del bot. Si decide probar el bot de forma local, debe ejecutar un software de tunelización como [ngrok](http://ngrok.com) para obtener un punto de conexión para el bot local.

## <a name="get-messaging-endpoint-using-ngrok"></a>Obtener el punto de conexión de mensajería con ngrok

Si está ejecutando el bot de forma local, puede obtener un punto de conexión y usarlo para realizar pruebas; para ello, ejecute un software de tunelización como [ngrok](https://ngrok.com). Para usar ngrok para obtener un punto de conexión, escriba lo siguiente desde la ventana de una consola: 

```cmd
 ngrok.exe http 3979 -host-header="localhost:3979"
``` 

Esto configura y muestra un enlace de reenvío ngrok que reenvía las solicitudes al bot, que se ejecuta en el puerto 3978. La dirección URL del enlace de reenvío debe tener el siguiente aspecto: `https://0d6c4024.ngrok.io`.  Anexe `/api/messages` al enlace para crear una dirección URL del punto de conexión de mensajería que tenga este formato: `https://0d6c4024.ngrok.io/api/messages`. 

Escriba esta dirección URL del punto de conexión en la sección **Configuración** de la hoja [Configuración](~/bot-service-manage-settings.md) del bot.

## <a name="enable-speech-recognition-priming"></a>Habilitar el desbloqueo del reconocimiento de voz
Si el bot utiliza una aplicación de tipo Language Understanding (LUIS), asegúrese de asociar el id. de la aplicación LUIS al servicio de bot registrado. Esto permitirá al robot a reconocer expresiones verbales que se hayan definido en el modelo LUIS. Para obtener más información, consulte [Speech priming](~/bot-service-manage-speech-priming.md) (Desbloqueo de voz).

## <a name="add-the-cortana-channel"></a>Agregar el canal de Cortana
Para agregar Cortana como un canal, siga los pasos indicados en [Connect a bot to Cortana](bot-service-channel-connect-cortana.md) (Conectar un bot a Cortana).

## <a name="test-using-web-chat-control"></a>Probar el control Chat en web

Para probar el bot con el control Chat en web integrado en Bot Service, haga clic en **Probar en Chat en web**, y escriba un mensaje para comprobar que el bot funciona.

## <a name="test-using-emulator"></a>Hacer pruebas con el emulador

Para probar el bot con el [emulador](~/bot-service-debug-emulator.md), haga lo siguiente:

1. Ejecute el bot.
2. Abra el emulador y rellene la información necesaria. Para buscar los valores de **AppID** y **AppPassword** del bot, consulte [MicrosoftAppID and MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword) (MicrosoftAppID y MicrosoftAppPassword). 
3. Haga clic en **Conectar** para conectar el emulador al bot.
4. Escriba un mensaje para comprobar que el bot funciona.

## <a name="test-using-cortana"></a>Hacer pruebas usando Cortana
Puede invocar las habilidades de Cortana con la voz; solo diga una frase de invocación a Cortana. 
1. Abra Cortana.
2. Abra el Cuaderno de Cortana y haga clic en **Acerca de mí** para ver qué cuenta está usando en Cortana. Asegúrese de que tiene la misma cuenta de Microsoft que utilizó para registrar el bot. 
   ![Iniciar sesión en el Cuaderno de Cortana](~/media/cortana/cortana-notebook.png)
2. Haga clic en el botón del micrófono en la aplicación Cortana o en el cuadro de búsqueda "Ask me anything" (Pregúntame cualquier cosa) en Windows, y diga la [frase de invocación][InvocationNameGuidelines] del bot. La frase de invocación incluye un *nombre de invocación*, que identifica de manera única la habilidad de invocar. Por ejemplo, si el nombre de invocación de una habilidad es "Northwind Photo", una frase de invocación adecuada puede incluir "Pídele a Northwind Photo que..." o "Dile a Northwind Photo que...".

   Debe especificar el *Nombre de invocación* del bot cuando lo configure en Cortana.
   ![Escriba el nombre de la invocación cuando configure el canal de Cortana](~/media/cortana/cortana-invocation-name-callout.png)

3. Si Cortana reconoce su frase de invocación, el bot se inicia en el lienzo de Cortana. 

## <a name="troubleshoot"></a>Solución de problemas

Si su habilidad de Cortana no se inicia, compruebe lo siguiente:
* Asegúrese de haber iniciado sesión en Cortana con la misma cuenta de Microsoft que usó para registrar el bot en el portal de Bot Framework.
* Compruebe si el bot funciona; para ello, haga clic en **Probar en Chat en web** para abrir la ventana **Chat**, y escriba un mensaje.
* Compruebe si el nombre de invocación cumple con las [instrucciones][InvocationNameGuidelines] indicadas. Si su nombre de invocación tiene más de tres palabras, es difícil de pronunciar o suena igual que otras palabras, Cortana puede tener dificultades para reconocerlo.
* Si su habilidad usa un modelo LUIS, asegúrese de [habilitar el desbloqueo del reconocimiento de voz ](~/bot-service-manage-speech-priming.md).

Consulte [Enable Debugging of Cortana skills][Cortana-TestBestPractice] (Habilitar la depuración de las habilidades de Cortana) para obtener sugerencias adicionales para la solución de problemas e información sobre cómo habilitar la depuración de la habilidad en el panel de Cortana. 


## <a name="next-steps"></a>Pasos siguientes

Una vez que haya probado la habilidad de Cortana y comprobado que funciona adecuadamente, puede implementarla en un grupo de evaluadores beta o publicarla para el público en general. Consulte [Publishing Cortana Skills][Cortana-Publish] (Publicar habilidades de Cortana) para obtener más información.

## <a name="additional-resources"></a>Recursos adicionales
* [Kit de habilidades de Cortana][CortanaGetStarted]

[CortanaGetStarted]: /cortana/getstarted

[BFPortal]: https://dev.botframework.com/
[CortanaDevCenter]: https://developer.microsoft.com/en-us/cortana

[CortanaSpecificEntities]: https://aka.ms/lgvcto
[CortanaAuth]: https://aka.ms/vsdqcj

[InvocationNameGuidelines]: https://aka.ms/cortana-invocation-guidelines 


[Cortana-Debug]: https://aka.ms/cortana-enable-debug
[Cortana-TestBestPractice]: https://aka.ms/cortana-test-best-practice
[Cortana-Publish]: /cortana/skills/publish-skill

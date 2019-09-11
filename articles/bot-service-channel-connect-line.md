---
title: Conexión de un bot a LINE | Microsoft Docs
description: Aprenda a configurar la conexión de un bot a LINE.
keywords: connect a bot, bot channel, LINE bot, credentials, configure, phone
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 1/7/2019
ms.openlocfilehash: 8be0c7f89595e3222e5170fc7f11d052f9cb6851
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298472"
---
# <a name="connect-a-bot-to-line"></a>Conexión de un bot a LINE

Puede configurar el bot para que se comunique con personas mediante la aplicación LINE.

## <a name="log-into-the-line-console"></a>Inicio de sesión en la consola de LINE

Inicie sesión en la [consola del desarrollador de LINE](https://developers.line.biz/console/register/messaging-api/provider/) con su cuenta de LINE, con la opción *Iniciar sesión con LINE*. 

> [!NOTE]
> Si no lo ha hecho ya, [descargue LINE](https://line.me/) y vaya a la configuración para registrar su dirección de correo electrónico.

### <a name="register-as-a-developer"></a>Registro como desarrollador

Si es su primera vez en la consola del desarrollador de LINE, escriba su dirección de correo electrónico y su nombre para crear una cuenta de desarrollador.

![Captura de pantalla de LINE: registro de desarrollador](./media/channels/LINE-screenshot-1.png)

## <a name="create-a-new-provider"></a>Creación de un nuevo proveedor

En primer lugar, cree un proveedor para el bot si aún no tiene uno. El proveedor es la entidad (individuo o empresa) que ofrece la aplicación.

![Captura de pantalla de LINE: crear proveedor](./media/channels/LINE-screenshot-2.png)

## <a name="create-a-messaging-api-channel"></a>Creación de un canal de la API de mensajería

A continuación, cree un canal de la API de mensajería. 

![Captura de pantalla de LINE: tipo de canal](./media/channels/LINE-channel-type-selection.png)

Para crear un nuevo canal de la API de mensajería, haga clic en el cuadrado verde.

![Captura de pantalla de LINE: crear canal](./media/channels/LINE-create-channel.png)

El nombre no puede incluir "LINE" ni ninguna cadena similar. Rellene los campos obligatorios y confirme la configuración del canal.

![Captura de pantalla de LINE: configuración del canal](./media/channels/LINE-screenshot-4.png)

## <a name="get-necessary-values-from-your-channel-settings"></a>Obtención de los valores necesarios de la configuración del canal

Una vez que haya confirmado la configuración del canal, se le dirigirá a una página similar a la siguiente.

![Captura de pantalla de LINE: página del canal](./media/channels/LINE-screenshot-5.png)

Haga clic en el canal que ha creado para acceder a su configuración y desplácese hacia abajo hasta encontrar **Basic information > Channel secret** (Información básica > Secreto de canal). Guárdelo durante unos momentos. Compruebe que entre las características disponibles se encuentra `PUSH_MESSAGE`.

![Captura de pantalla de LINE: secreto del canal](./media/channels/LINE-screenshot-6.png)

Después, vaya hasta **Messaging settings** (Configuración de mensajería). Allí verá un campo **Channel access token** (Token de acceso de canal) con un botón *issue* (emitir). Haga clic en ese botón para obtener el token de acceso y guárdelo también por el momento.

![Captura de pantalla de LINE: token de canal](./media/channels/LINE-screenshot-8.png)

## <a name="connect-your-line-channel-to-your-azure-bot"></a>Conexión del canal de LINE a su bot de Azure

Inicie sesión en [Azure Portal](https://portal.azure.com/), busque su bot y haga clic en **Channels** (Canales). 

![Captura de pantalla de LINE: configuración de Azure](./media/channels/LINE-channel-setting-2.png)

Allí, seleccione el canal de LINE y pegue el secreto de canal y el token de acceso anteriores en los campos correspondientes. Asegúrese de guardar los cambios.

Copie la dirección URL del webhook personalizado que le ofrece Azure.

![Captura de pantalla de LINE: configuración de Azure](./media/channels/LINE-channel-setting-1.png)

## <a name="configure-line-webhook-settings"></a>Configuración de las opciones del webhook de LINE

A continuación, vuelva a la consola del desarrollador de LINE y pegue la dirección URL del webhook de Azure en **Message settings > Webhook URL** (Configuración de mensajes > URL de webhook), y haga clic en **Verify** (Verificar) para comprobar la conexión. Si acaba de crear el canal en Azure, puede tardar unos minutos en surtir efecto.

A continuación, habilite **Message settings > Use webhooks** (Configuración de mensajes > Usar webhooks).

> [!IMPORTANT]
> En la consola del desarrollador de LINE, debe establecer primero la dirección URL del webhook y, después, habilite **Use webhooks** (Usar webhooks). Si se habilitan primero los webhooks con una dirección URL vacía, el estado no se establece en Habilitado aunque la interfaz de usuario diga otra cosa.

Después de agregar una dirección URL de webhook y de habilitar los webhooks, asegúrese de volver a cargar esta página y compruebe que los cambios se han establecido correctamente.

![Captura de pantalla de LINE: webhooks](./media/channels/LINE-screenshot-9.png)

## <a name="test-your-bot"></a>Prueba del bot

Cuando haya completado estos pasos, el bot estará configurado correctamente para comunicarse con los usuarios en LINE y estará listo para las pruebas.

### <a name="add-your-bot-to-your-line-mobile-app"></a>Adición del bot a su aplicación móvil LINE

En la consola del desarrollador de LINE, vaya a la página de configuración y verá el código QR de su bot. 

En la aplicación móvil LINE, vaya a la pestaña de navegación situada más a la derecha, con tres puntos [ **...** ], y mantenga pulsado el icono del código QR. 

![Captura de pantalla de LINE: aplicación móvil](./media/channels/LINE-screenshot-12.jpg)

En la consola del desarrollador, apunte al código QR con el lector de códigos QR. Ahora podrá interactuar con su bot y probarlo en la aplicación móvil LINE.

### <a name="automatic-messages"></a>Mensajes automáticos

Al iniciar las pruebas de un bot, quizás observe que el bot envía mensajes inesperados que no son los que ha especificado en la actividad `conversationUpdate`.  El diálogo tendrá un aspecto similar al siguiente:

![Captura de pantalla de LINE: conversación](./media/channels/LINE-screenshot-conversation.jpg)

Para evitar el envío de estos mensajes, deberá desactivar los mensajes de respuesta automática.

![Captura de pantalla de LINE: respuesta automática](./media/channels/LINE-screenshot-10.png)

También puede mantener estos mensajes. En este caso, sería buena idea hacer clic en "Establecer el mensaje" y editarlo.

![Captura de pantalla de LINE: establecer la respuesta automática](./media/channels/LINE-screenshot-11.png)

## <a name="troubleshooting"></a>solución de problemas

* Si su bot no responde a ninguno de los mensajes, vaya al bot en Azure Portal y elija Probar en Chat en web.  
    * Si el bot funciona allí pero no responde en LINE, vuelva a cargar la página de la consola del desarrollador de LINE y repita las instrucciones anteriores para el webhook. Asegúrese de establecer la **dirección URL del webhook** antes de habilitar los webhooks.
    * Si el bot no funciona en Chat en web, depure el problema del bot luego finalice la configuración de su canal de LINE.


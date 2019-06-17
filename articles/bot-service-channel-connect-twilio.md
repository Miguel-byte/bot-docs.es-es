---
title: Conexión de un bot a Twilio | Microsoft Docs
description: Obtenga información sobre cómo configurar la conexión de un bot a Twilio.
keywords: Twilio, canales de bot, SMS, aplicación, teléfono, configurar Twilio, comunicación en la nube, texto
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/9/2018
ms.openlocfilehash: 817623dd04612cd07d8877c8e9a199c05a2fd9e8
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693622"
---
# <a name="connect-a-bot-to-twilio"></a>Conexión de un bot a Twilio

Puede configurar su bot para que se comunique con las personas mediante la plataforma de comunicaciones en la nube de Twilio.

## <a name="log-in-to-or-create-a-twilio-account-for-sending-and-receiving-sms-messages"></a>Iniciar sesión en Twilio o crear una cuenta para enviar y recibir mensajes SMS

Si no tiene cuenta de Twilio, <a href="https://www.twilio.com/try-twilio" target="_blank">cree una</a>.

## <a name="create-a-twiml-application"></a>Crear una aplicación de TwiML

<a href="https://support.twilio.com/hc/en-us/articles/223180928-How-Do-I-Create-a-TwiML-App-" target="_blank">Cree una aplicación de TwiML</a> siguiendo las instrucciones.

![Creación de una aplicación](~/media/channels/twi-StepTwiml.png)

En **Properties** (Propiedades), escriba un **NOMBRE DESCRIPTIVO**. En este tutorial se usa "My TwiML app" como ejemplo. La **DIRECCIÓN URL DE SOLICITUD** de Voz puede dejarse en blanco. En **Messaging** (Mensajería), la **dirección URL de solicitud** debe ser `https://sms.botframework.com/api/sms`.

## <a name="select-or-add-a-phone-number"></a>Seleccionar o agregar un número de teléfono

Siga <a href = "https://support.twilio.com/hc/en-us/articles/223180048-Adding-a-Verified-Phone-Number-or-Caller-ID-with-Twilio" target="_blank">estas</a> instrucciones para agregar un identificador de llamada verificado mediante el sitio de la consola. Cuando termine, verá el número verificado en **Active Numbers** (Números activos) en **Manage Numbers** (Administrar números).

![Establecer el número de teléfono](~/media/channels/twi-StepPhone.png)

## <a name="specify-application-to-use-for-voice-and-messaging"></a>Especificación de la aplicación que se usará para voz y mensajería

Haga clic en el número y vaya a **Configure** (Configurar). En voz y mensajería, establezca **CONFIGURE WITH** (CONFIGURAR CON) con TwiML App y **TWIML APP** (APLICACIÓN DE TWIML), con My TwiML app. Cuando termine, haga clic en **Save** (Guardar).

![Especificación de la aplicación](~/media/channels/twi-StepPhone2.png)

Vuelva a **Manage Numbers** (Administrar números), verá como la configuración de voz y mensajería han cambiado a TwiML App.

![Número especificado](~/media/channels/twi-StepPhone3.png)


## <a name="gather-credentials"></a>Obtener las credenciales

Vuelva a la [página principal de la consola](https://www.twilio.com/console/), verá el identificador de seguridad de la cuenta y el token de autenticación en el panel del proyecto, como se muestra a continuación.

![Obtener las credenciales de la aplicación](~/media/channels/twi-StepAuth.png)

## <a name="submit-credentials"></a>Enviar las credenciales

En otra ventana, vuelva al sitio de Bot Framework en https://dev.botframework.com/. 

- Seleccione **My bots** (Mis bots) y elija el bot que desea que se conecte con Twilio. Esto le dirigirá a Azure Portal.
- Seleccione **Channels** (Canales) en **Bot Management** (Administración del bot). Haga clic en el icono de Twilio (SMS).
- Escriba el número de teléfono, el identificador de seguridad de cuenta y el token de autenticación que registró anteriormente. Cuando termine, haga clic en **Save** (Guardar).

![Enviar las credenciales](~/media/channels/twi-StepSubmit.png)

Cuando haya completado estos pasos, el bot se configurará correctamente para comunicarse con los usuarios mediante Twilio.

## <a name="also-available-as-an-adapter"></a>También está disponible como un adaptador

Este canal también está [disponible como un adaptador](https://botkit.ai/docs/v4/platforms/twilio-sms.html). Para ayudarle a elegir entre un adaptador y un canal, consulte [Adaptadores disponibles actualmente](bot-service-channel-additional-channels.md#currently-available-adapters).
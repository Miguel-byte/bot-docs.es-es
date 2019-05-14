---
title: Solución de problemas de configuración del bot | Microsoft Docs
description: Solución de problemas de configuración en un bot implementado.
keywords: solución de problemas, configuración, chat en web, problemas.
author: jonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/30/2019
ms.openlocfilehash: c208cef52d1850a00b62828ae0ea622a2606ec5b
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033423"
---
# <a name="troubleshoot-bot-configuration-issues"></a>Solución de problemas de configuración del bot

El primer paso para solucionar problemas de un bot es probarlo en Chat en web. Esto le permitirá determinar si el problema es específico del bot (el bot no funciona en ningún canal) o de un canal determinado (el bot funciona en algunos canales, pero no en otros).

## <a name="test-in-web-chat"></a>Probar en Chat en web

1. Abra el recurso del bot en [Azure Portal](http://portal.azure.com/).
1. Abra el panel **Test in Web Chat** (Probar en Chat en web).
1. Envíe un mensaje al bot.

![Probar en Chat en web](./media/test-in-webchat.png)

Si el bot no responde con la salida esperada, vaya a la sección [El bot no funciona en Chat en web](#bot-does-not-work-in-web-chat). De lo contrario, vaya al apartado [El bot funciona en Chat en web pero no en otros canales](#bot-works-in-web-chat-but-not-in-other-channels).

## <a name="bot-does-not-work-in-web-chat"></a>El bot no funciona en Chat en web

Puede haber varias razones por las que un bot no funciona. Probablemente, la aplicación de bot está fuera de servicio y no puede recibir mensajes, o los recibe, pero no responde.

Para ver si se está ejecutando el bot:

1. Abra el panel de **información general**.
1. Copie el **punto de conexión de mensajería** y péguelo en el explorador.

Si el punto de conexión devuelve un error HTTP 405, significa que se puede acceder al bot y que este puede responder a los mensajes. Debe investigar si su bot [agota el tiempo de espera](https://github.com/daveta/analytics/blob/master/troubleshooting_timeout.md) o [produce un error HTTP 5xx](bot-service-troubleshoot-500-errors.md).

Si el punto de conexión devuelve un error "This site can't be reached" (No se puede acceder a este sitio) o "can't reach this page" (No se puede acceder a esta página), significa que su bot está fuera de servicio y tiene que volver a implementarlo.

## <a name="bot-works-in-web-chat-but-not-in-other-channels"></a>El bot funciona en Chat en web pero no en otros canales

Si el bot funciona según lo previsto en Chat en web pero se produce un error en algún otro canal, hay varios motivos posibles:

- [Problemas de configuración del canal](#channel-configuration-issues)
- [Comportamiento específico del canal](#channel-specific-behavior)
- [Interrupción del canal](#channel-outage)

### <a name="channel-configuration-issues"></a>Problemas de configuración del canal

Es posible que los parámetros de configuración del canal se hayan establecido incorrectamente o se hayan cambiado externamente. Por ejemplo, un bot ha configurado el canal de Facebook para una página determinada y, posteriormente, se ha eliminado esa página. La solución más sencilla es eliminar el canal y rehacer la configuración del canal de nuevo.

Los vínculos siguientes proporcionan instrucciones para configurar los canales compatibles con Bot Framework:

- [Cortana](bot-service-channel-connect-cortana.md)
- [DirectLine](bot-service-channel-connect-directline.md)
- [Correo electrónico](bot-service-channel-connect-email.md)
- [Facebook](bot-service-channel-connect-facebook.md)
- [GroupMe](bot-service-channel-connect-groupme.md)
- [Kik](bot-service-channel-connect-kik.md)
- [Microsoft Teams](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-overview)
- [Skype](bot-service-channel-connect-skype.md)
- [Skype Empresarial](bot-service-channel-connect-skypeforbusiness.md)
- [Slack](bot-service-channel-connect-slack.md)
- [Telegram](bot-service-channel-connect-telegram.md)
- [Twilio](bot-service-channel-connect-twilio.md)

### <a name="channel-specific-behavior"></a>Comportamiento específico del canal

La implementación de algunas características puede ser diferente según el canal. Por ejemplo, no todos los canales admiten las tarjetas adaptables. La mayoría de los canales admiten botones, pero se representan de una forma específica en cada canal. Si ve diferencias en como funcionan algunos tipos de mensaje en diferentes canales, consulte la [referencia del canal](bot-service-channels-reference.md).

A continuación se muestran algunos vínculos adicionales que pueden ayudarle con determinados canales individuales:

- [Add bots to Microsoft Teams apps](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-overview) (Incorporación de bots a las aplicaciones de Microsoft Teams)
- [Facebook: Introducción a la plataforma de Messenger](https://developers.facebook.com/docs/messenger-platform/introduction)
- [Principles of Cortana Skills design](https://docs.microsoft.com/cortana/skills/design-principles) (Principios de diseño de aptitudes de Cortana)
- [Skype for Developers](https://dev.skype.com/bots) (Skype para desarrolladores)
- [Slack: Enabling interactions with bots](https://api.slack.com/bot-users) (Slack: Habilitar interacciones con bots)

### <a name="channel-outage"></a>Interrupción del canal

En ocasiones, algunos canales pueden tener una interrupción del servicio. Normalmente, esas interrupciones no duran mucho tiempo. Sin embargo, si sospecha que hay una interrupción del servicio, consulte el sitio web del canal o las redes sociales.

Otra manera de determinar si un canal tiene una interrupción es crear un bot de prueba (como, por ejemplo, un sencillo bot de eco) y agregar un canal. Si el bot de prueba funciona con algunos canales, pero no con otros, eso indica que el problema no está en el bot de producción.

## <a name="additional-resources"></a>Recursos adicionales

Consulte el tema de procedimientos para [depurar un bot](bot-service-debug-bot.md) y los otros artículos de depuración de esa sección.
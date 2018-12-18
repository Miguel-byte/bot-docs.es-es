---
title: Conexión de un bot a Skype | Microsoft Docs
description: Aprenda a configurar un bot para acceder mediante la interfaz de Skype.
keywords: skype, canales de bot, configurar skype, publicar, conectarse a canales
author: v-ducvo
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/11/2018
ms.openlocfilehash: f52c8fd668299486eccda3920181df971f475a90
ms.sourcegitcommit: 75f32b3325dd0fc4d8128dee6c22ebf91e5785b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/09/2018
ms.locfileid: "53120662"
---
# <a name="connect-a-bot-to-skype"></a>Conexión de un bot a Skype

Skype le mantiene conectado con los usuarios a través de mensajería instantánea, teléfono y videollamadas. Para ampliar esta funcionalidad, cree bots que los usuarios pueden detectar y con los que puedan interactuar a través de la interfaz de Skype.

Para agregar el canal de Skype, abra el bot en [Azure Portal](https://portal.azure.com/), haga clic en la hoja **Canales** y luego en **Skype**.

![Agregar el canal de Skype](~/media/channels/skype-addchannel.png)

Esto le llevará a la página **Configurar Skype**.

![Configuración del canal de Skype](~/media/channels/skype_configure.png)

Debe establecer la configuración en **Control web**, **Mensajería**, **Llamadas**, **Grupos** y **Publicación**. Vamos a verlos de uno en uno.

## <a name="web-control"></a>Control web

Para insertar el bot en su sitio web, haga clic en el botón **Obtener código para insertar** de la sección **Control web**. Esto le dirigirá a la página de Skype para desarrolladores. Siga las instrucciones para obtener el código para insertar.

## <a name="messaging"></a>Mensajería

En esta sección configura la manera en que el bot envía y recibe mensajes de Skype.

## <a name="calling"></a>Llamada

En esta sección se configura la característica de llamadas de Skype en el bot. Puede especificar si **Llamada** está habilitada para el bot, y si está habilitada, si se usará la funcionalidad IVR o de multimedia en tiempo real.

## <a name="groups"></a>Grupos

En esta sección se configura si el bot puede agregarse a un grupo y cómo se comporta en un grupo para la mensajería; también se utiliza para habilitar las videollamadas grupales para los bots de llamada.

## <a name="publish"></a>Publicar

En esta sección se establece la configuración de publicación del bot. Todos los campos con * son campos obligatorios.

Los bots en **versión preliminar** están limitados a 100 contactos. Si necesita más de 100 contactos, envíe el bot para revisión. Al hacer clic en **Enviar para revisión**, hará automáticamente que el bot se pueda buscar en Skype si se lo acepta. Si no se aprueba la solicitud, se le notificará qué debe cambiar para que pueda aprobarse.

> [!TIP]
> Si desea enviar el bot para que lo examinen, tenga en cuenta debe cumplir la [lista de comprobación de certificación de Skype](https://github.com/Microsoft/skype-dev-bots/blob/master/certification/CHECKLIST.md) para ser aceptado.

Al terminar la configuración, haga clic en **Guardar** y acepte las **Condiciones del servicio**. El canal de Skype se habrá agregado al bot.

## <a name="next-steps"></a>Pasos siguientes

* [Skype Empresarial](bot-service-channel-connect-skypeforbusiness.md)

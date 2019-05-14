---
title: Guía de identificadores de Framework Bot | Microsoft Docs
description: Esta guía describe las características de los campos de identificador presentes en el protocolo de Bot Framework v3.
keywords: identificador, bots, protocolo
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/30/2019
ms.openlocfilehash: 28932ca45c6faaad2f17ecc03f026ba04352a5a1
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033073"
---
# <a name="id-fields-in-the-bot-framework"></a>Campos de identificador en Bot Framework

Esta guía describe las características de los campos de identificador en Bot Framework.

## <a name="channel-id"></a>Identificador de canal

Todos y cada uno de los canales de Bot Framework se identifican mediante un identificador único.

Ejemplo: `"channelId": "skype"`

Los identificadores de canal sirven como espacios de nombres para otros identificadores. Las llamadas en tiempo de ejecución en el protocolo de Bot Framework tienen que producirse dentro del contexto de un canal; el canal proporciona significado a la conversación y a los identificadores de cuenta que se usan al comunicarse.

Por convención, todos los identificadores de canal están en minúsculas. Los canales garantizan que los identificadores de canal que emiten mantienen coherencia en el uso de mayúsculas y minúsculas y, por lo tanto, los bots pueden usar comparaciones ordinales para establecer la equivalencia.

### <a name="rules-for-channel-ids"></a>Reglas para los identificadores de canal

- Los identificadores distinguen entre mayúsculas y minúsculas.

## <a name="bot-handle"></a>Identificador de bot

Cada bot que se ha registrado con Azure Bot Service tiene un identificador de bot.

Ejemplo: `FooBot`

El identificador de bot representa el registro de un bot con Azure Bot Service en línea. Este registro se asocia con un punto de conexión de webhook HTTP y los registros de canales.

Azure Bot Service garantiza la exclusividad de los identificadores de bot. Azure Portal realiza una comprobación de la exclusividad de mayúsculas y minúsculas (lo que significa que las variaciones de mayúsculas y minúsculas del identificador de bot se tratan como un único identificador), aunque esto sea una característica de Azure Portal, y no necesariamente del propio identificador de bot.

### <a name="rules-for-bot-handles"></a>Reglas para los identificadores de bot

* Los identificadores de bot son únicos (mayúsculas y minúsculas) dentro de Bot Framework.

## <a name="app-id"></a>Id. de aplicación

Cada bot que se ha registrado con Azure Bot Service tiene un identificador de aplicación.

> [!NOTE]
> Anteriormente, las aplicaciones se conocían normalmente como "Aplicaciones de MSA" o "Aplicaciones de AAD/MSA". Ahora las aplicaciones se conocen normalmente solo como "aplicaciones", pero algunos elementos de protocolo pueden seguir haciendo referencia a las aplicaciones como "Aplicaciones de MSA".

Ejemplo: `"msaAppId": "353826a6-4557-45f8-8d88-6aa0526b8f77"`

Una aplicación representa un registro con el portal de aplicaciones del equipo de identidad y sirve como mecanismo de identidad de servicio a servicio dentro del protocolo de tiempo de ejecución de Bot Framework. Las aplicaciones pueden tener otras asociaciones que no sean de bot, como sitios web y aplicaciones de escritorio o móviles.

Cada bot registrado tiene exactamente una aplicación. Aunque no es posible para el propietario de un bot cambiar de manera independiente la aplicación que está asociada con su bot, el equipo de Bot Framework sí que puede hacerlo en un pequeño número de casos excepcionales.

Los bots y los canales pueden usar los identificadores de aplicación para identificar de forma única a un bot registrado.

Los identificadores de aplicación son siempre GUID. Los identificadores de aplicación se deben comparar sin diferenciar entre mayúsculas y minúsculas.

### <a name="rules-for-app-ids"></a>Reglas para los identificadores de aplicación

* Los identificadores de aplicación son únicos (comparación GUID) dentro de la plataforma de Microsoft App.
* Cada bot tiene exactamente una aplicación correspondiente.
* El cambio de la aplicación que está asociada a un bot requiere la ayuda del equipo de Bot Framework.

## <a name="channel-account"></a>Cuenta de canal

Cada bot y usuario tienen una cuenta dentro de cada canal. La cuenta contiene un identificador (`id`) y otros datos informativos no estructurales del bot, como un nombre opcional.

Ejemplo: `"from": { "id": "john.doe@contoso.com", "name": "John Doe" }`

Esta cuenta describe la dirección dentro del canal donde se pueden enviar y recibir mensajes. En algunos casos, estos registros existen dentro de un único servicio (p. ej. Skype, Facebook). En otros casos, se registran en varios sistemas (direcciones de correo electrónico, números de teléfono). En los canales más anónimos (p. ej. Chat en web), el registro puede ser efímero.

Las cuentas de canal están anidadas dentro de los canales. Una cuenta de Facebook, por ejemplo, es simplemente un número. Este número puede tener un significado diferente en otros canales, y no tiene significado fuera de todos los canales.

La relación entre las cuentas de canal y los usuarios (personas reales) depende de las convenciones asociadas a cada canal. Por ejemplo, un número SMS suele hacer referencia a una persona durante un período de tiempo, pasado este tiempo el número se puede transferir a otra persona. Por el contrario, una cuenta de Facebook suele hacer referencia a una persona de forma perpetua, aunque no es raro que dos personas compartan una cuenta de Facebook.

En la mayoría de los canales, se puede pensar en una cuenta de canal como una especie de buzón donde se pueden entregar mensajes. Es habitual que la mayoría de los canales permitan que varias direcciones se asignen a un solo buzón; Por ejemplo, "jdoe@contoso.com"y"john.doe@service.contoso.com" se pueden resolver con la misma bandeja de entrada. Algunos canales van más allá y modifican la dirección de la cuenta en función de qué bot tiene acceso a ella; por ejemplo, Skype y Facebook alteran los identificadores de usuario para que cada bot tenga una dirección diferente para enviar y recibir mensajes.

Aunque en algunos casos se puede establecer la equivalencia entre las direcciones, el establecimiento de equivalencias entre los buzones y equivalencias entre personas requiere el conocimiento de las convenciones dentro del canal, y en muchos casos no es posible.

El bot recibe información de su dirección de cuenta de canal en el campo `recipient` en las actividades que se envían al bot.

### <a name="rules-for-channel-accounts"></a>Reglas para las cuentas de canal

* Las cuentas de canal tienen significado solo dentro de su canal asociado.
* Se puede resolver más de un identificador a la misma cuenta.
* La comparación ordinal puede utilizarse para establecer que dos identificadores son el mismo.
* Normalmente, no hay ninguna comparación que puede usarse para identificar si dos identificadores diferentes se resuelven a la misma cuenta, bot o persona.
* La estabilidad de las asociaciones entre identificadores, cuentas, buzones y personas depende del canal.

## <a name="conversation-id"></a>Identificador de conversación

Los mensajes se envían y reciben en el contexto de una conversación, que se puede reconocer por su identificador.

Ejemplo: `"conversation": { "id": "1234" }`

Una conversación contiene un intercambio de mensajes y otras actividades. Cada conversación tiene cero o más actividades, y cada actividad aparece exactamente en una conversación. Las conversaciones pueden ser perpetuas, o pueden tener principios y finales claros. El proceso de creación, modificación o finalización de una conversación se produce dentro del canal (es decir, una conversación existe cuando el canal es consciente de ello), y es el canal el que establece las características de estos procesos.

Las actividades dentro de una conversación las envían los usuarios y los bots. La definición para la que los usuarios "participan" en una conversación varía según el canal, y puede incluir, en teoría, a los usuarios actuales, los usuarios que nunca han recibido un mensaje y los usuarios que hayan enviado un mensaje.

Varios canales (por ejemplo, SMS, Skype y posiblemente otros) tienen la peculiaridad de que el identificador de conversación asignado a una conversación de 1:1 es el identificador de cuenta de canal remoto. Esta peculiaridad tiene dos efectos secundarios:
1. El identificador de conversación es subjetivo en base a quién lo esté viendo. Si los participantes A y B están hablando, el participante A ve el identificador de conversación como "B" mientras el participante B ve el identificador de conversación como "A".
2. Si el bot tiene varias cuentas de canal dentro de este canal (por ejemplo, si el bot tiene dos números SMS), el identificador de conversación no es suficiente para identificar de forma exclusiva la conversación dentro campo de vista del bot.

Por lo tanto, un identificador de conversación no identifica necesariamente una sola conversación dentro de un canal, incluso para un solo bot.

### <a name="rules-for-conversation-ids"></a>Reglas para los identificadores de conversación

* Las conversaciones tienen significado solo dentro de su canal asociado.
* Se puede resolver más de un identificador a la misma conversación.
* La igualdad ordinal no establece necesariamente que dos identificadores de conversación son la misma conversación, aunque en la mayoría de los casos, sí lo hace.

## <a name="activity-id"></a>Identificador de actividad

Las actividades se envían y reciben dentro del protocolo de Bot Framework y a veces son identificables.

Ejemplo: `"id": "5678"`

Los identificadores de actividad son opcionales y los emplean los canales para dar al bot una manera de hacer referencia al identificador en llamadas API posteriores, si están disponibles:
* Respuesta a una actividad específica
* Consulta de la lista de participantes en el nivel de actividad

Dado que no se han establecido más casos de uso, no hay reglas adicionales para el tratamiento de los identificadores de actividad.

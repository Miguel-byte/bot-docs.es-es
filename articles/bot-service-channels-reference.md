---
title: Referencia de canales
description: Referencia de canales de Bot Framework
keywords: channels reference, bot builder channels, bot framework channels
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: tools
ms.date: 03/01/2019
ms.openlocfilehash: 28f284e4d69cbef7a1741d298b3ae9e6e127e9dd
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/21/2019
ms.locfileid: "59541091"
---
# <a name="categorized-activities-by-channel"></a>Actividades clasificadas por canal

Las siguientes tablas muestran qué eventos (actividades de la conexión) pueden proceder de los canales.

Esta es la clave para las tablas:

Símbolo              | Significado
:------------------:|:------------------------------------------------
:white_check_mark:  |El bot debe esperar recibir esta actividad.
:x:                 |El bot no debe esperar **nunca** recibir esta actividad.
:white_large_square:|No se ha determinado actualmente si el bot puede recibirlo.

Las actividades se pueden dividir de forma significativa en categorías distintas. Para cada categoría, tenemos una tabla de las actividades posibles.

<a name="conversational"></a>Conversacional
--------------

 \                      | Cortana            | Direct Line        | Direct Line (Chat en web) | Email              | Facebook           | GroupMe            | Kik                | Teams              | Slack              | Skype   | Skype Empresarial | Telegram | Twilio  
:---------------------- | :-----:            | :----------------: | :--------------------: |:----:              | :------:           | :-----:            | :-----:            | :---:              | :---:              | :---:   | :------------: | :------: | :----:  
Message                 | :white_check_mark: | :white_check_mark: | :white_check_mark:     | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark:        | :white_check_mark:  | :white_check_mark: 
MessageReaction         | :x:                | :x:                | :x:                    | :x:                | :x:                | :x:                | :x:                | :white_check_mark: | :x:                | :x:      | :x:             | :x:       | :x:      

- Todos los canales envían actividades de mensajes.
- Cuando se usa un diálogo, las actividades de mensajes por lo general siempre se deben pasar al diálogo.
- Esto probablemente no es así para MessageReaction, aunque forman parte de la conversación.
- Desde el punto de vista lógico, son dos tipos de MessageReaction: agregados y quitados.


> [!TIP]
> Las "reacciones a los mensajes" son como un "me gusta" en un comentario anterior. Pueden producirse sin orden, por lo que se pueden considerar como similares a los botones. Actualmente, esta actividad se envía por el canal de Teams.  


<a name="welcome"></a>Bienvenido
-------

 \                         | Cortana            | Direct Line        | Direct Line (Chat en web) | Email   | Facebook             | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Empresarial | Telegram | Twilio  
:----------------------    | :-----:            | :---------:        | :--------------------: |:----:   | :------:             | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
ConversationUpdate         | :white_check_mark: | :white_check_mark: | :white_check_mark:     | :x:     | :white_large_square: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:      | :x:             | :white_check_mark:  | :x:     
ContactRelationUpdate      | :x:                | :x:                | :x:                    | :x:     | :x:                  | :x:      | :x:      | :x:      | :x:      | :white_check_mark: | :white_check_mark:        | :x:       | :x:     

- Es habitual que los canales envíen actividades ConversationUpdate.
- Desde el punto de vista lógico, son dos tipos de MessageReaction: agregados y quitados.
- Es muy tentador suponer que el comportamiento del bot "Bienvenida" se puede implementar simplemente con ConversationUpdate.Added, y a veces funciona.
- Sin embargo, esto es una simplificación; para producir un comportamiento de "Bienvenida" confiable, la implementación del bot también puede necesitar usar el estado.


<a name="application-extensibility"></a>Extensibilidad de la aplicación
-------------------------

 \                      | Cortana            | Direct Line        | Direct Line (Chat en web) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Empresarial | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Event.*                    | :white_large_square: | :white_check_mark: | :white_check_mark:    | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.CreateConversation   | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.ContinueConversation | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  

- Las actividades de evento son un mecanismo de extensibilidad de Direct Line (_también conocido como Chat en web_).
- Una aplicación que posee tanto el cliente como el servidor puede elegir tunelizar sus propios eventos a través del servicio con esta actividad de evento.


<a name="microsoft-teams"></a>Equipos de Microsoft
------------------

 \                      | Cortana            | Direct Line        | Direct Line (Chat en web) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Empresarial | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Invoke.TeamsVerification   | :x:      | :x:          | :x: | :x:   | :x:       | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
Invoke.ComposeResponse     | :x:      | :x:          | :x: | :x:   | :x:       | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     

- Además de algunas otras actividades con tipo, Microsoft Teams define algunas actividades de invocación específicas de Teams.
- Las actividades de invocación son específicas de una aplicación y no es algo que un cliente definiría.
- No hay ninguna noción general de subtipos específicos de la actividad de invocación.
- Actualmente, la invocación es la única actividad que desencadena un comportamiento de solicitud-respuesta en el bot.

Esto es muy importante: si se usan diálogos, la actividad Invoke.TeamsVerification debe reenviarse al diálogo para que el aviso de OAuth funcione.


<a name="message-update"></a>Actualización de mensajes
--------------

 \                      | Cortana            | Direct Line        | Direct Line (Chat en web) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Empresarial | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
MessageUpdate | :x:      | :x:          | :x:    | :x: | :x:      | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
MessageDelete | :x:      | :x:          | :x:    | :x: | :x:      | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     

- Actualmente, Teams admite la actualización de mensajes.


<a name="oauth"></a>OAuth
-------

 \                      | Cortana            | Direct Line        | Direct Line (Chat en web) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Empresarial | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Event.TokenResponse| :white_large_square:  | :white_check_mark:   | :white_check_mark:    | :x:    | :white_large_square: | :white_large_square: | :white_large_square: | :x:    | :white_large_square: | :white_large_square: | :white_large_square:       | :white_large_square: | :white_large_square: 

Esto es muy importante: si se usan diálogos, la actividad Event.TokenResponse debe reenviarse al diálogo para que el aviso de OAuth funcione.


<a name="uncategorized"></a>Sin categoría 
-------------

 \                      | Cortana  | Direct Line        | Direct Line (Chat en web) | Email | Facebook | GroupMe | Kik     | Teams | Slack | Skype | Skype Empresarial | Telegram | Twilio  
:---------------------- | :-----:  | :---------:        | :--------------------: |:----: | :------: | :-----: | :-----: | :---: | :---: | :---: | :------------: | :------: | :----:  
EndOfConversation       | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
InstallationUpdate      | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Escritura                  | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Handoff                 | :x:      | :x:                | :x:                    | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     


<a name="out-of-use-includes-payment-specific-invoke"></a>En desuso (incluye la invocación específica del pago)
---------------------------------------------
- DeleteUserData 
- Invoke.PaymentRequest  
- Invoke.Address
- Ping

---

## <a name="summary-of-activities-supported-per-channel"></a>Resumen de las actividades admitidas por cada canal

<a name="cortana"></a>Cortana
-------
- Message
- ConversationUpdate
- _Event.TokenResponse_
- _EndOfConversation (¿cuando la ventana se cierra?)_

<a name="direct-line"></a>Direct Line
--------
- Message
- ConversationUpdate
- Event.TokenResponse
- Event.*
- _Event.CreateConversation_
- _Event.ContinueConversation_

<a name="email"></a>Email
-----
- Message

<a name="facebook"></a>Facebook
--------
- Message
- _Event.TokenResponse_

<a name="groupme"></a>GroupMe
-------
- Message
- ConversationUpdate
- _Event.TokenResponse_

<a name="kik"></a>Kik
---
- Message
- ConversationUpdate
- _Event.TokenResponse_

<a name="teams"></a>Teams
-----
- Message
- ConversationUpdate
- MessageReaction
- MessageUpdate
- MessageDelete
- Invoke.TeamsVerification
- Invoke.ComposeResponse


<a name="slack"></a>Slack
-----
- Message
- ConversationUpdate
- _Event.TokenResponse_


<a name="skype"></a>Skype
-----
- Message
- ContactRelationUpdate
- _Event.TokenResponse_


<a name="skype-business"></a>Skype Empresarial
--------------
- Message
- ContactRelationUpdate 
- _Event.TokenResponse_


<a name="telegram"></a>Telegram
--------
- Message
- ConversationUpdate
- _Event.TokenResponse_


<a name="twilio"></a>Twilio
------
- Message

## <a name="summary-table-all-activities-to-all-channels"></a>Tabla de resumen de todas las actividades a todos los canales

 \                         | Cortana              | Direct Line          | Direct Line (Chat en web) | Email                | Facebook             | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Empresarial | Telegram | Twilio  
:----------------------    | :-----:              | :---------:          | :--------------------: |:----:                | :------:             | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Message                    | :white_check_mark:   | :white_check_mark:   | :white_check_mark:     | :white_check_mark:   | :white_check_mark:   | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark:        | :white_check_mark:  | :white_check_mark: 
MessageReaction            | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:      | :x:      | :x:             | :x:       | :x:      
ConversationUpdate         | :white_check_mark:   | :white_check_mark:   | :white_check_mark:     | :x:                  | :white_large_square: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:      | :x:             | :white_check_mark:  | :x:     
ContactRelationUpdate      | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :x:      | :x:      | :white_check_mark: | :white_check_mark:        | :x:       | :x:     
Event.*                    | :white_large_square: | :white_check_mark:   | :white_check_mark:     | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.CreateConversation   | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.ContinueConversation | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Invoke.TeamsVerification   | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
Invoke.ComposeResponse     | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
MessageUpdate              | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
MessageDelete              | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
Event.TokenResponse        | :white_large_square: | :white_check_mark:   | :white_check_mark:     | :x:                  | :white_large_square: | :white_large_square: | :white_large_square: | :x:    | :white_large_square: | :white_large_square: | :white_large_square:       | :white_large_square: | :white_large_square: 
EndOfConversation          | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
InstallationUpdate         | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Escritura                     | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Handoff                    | :x:                  | :x:                  | :x:                    | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     

## <a name="web-chat"></a>Chat en web 
Chat en web enviará:
- "message": con "text" y "attachments".
- "event": "name" y "value" (JSON o cadena).
- "typing": si el usuario establece una opción, por ejemplo, "sendTypingIndicator", Chat en web no enviará "contactRelationUpdate". Chat en web no admite "messageReaction"; nadie nos ha pedido compatibilidad con esta característica.

De forma predeterminada, Chat en web representará:
- "message": se representará como un carrusel o apilado, según la opción de la actividad.
- "typing": se representará durante 5 s y se ocultará, o hasta que llegue la siguiente actividad.
- "conversationUpdate": se ocultará.
- "event": se ocultará.
- Otros: mostrará un cuadro de advertencia (nunca lo vemos en producción). Puede modificar esta canalización de representación para agregar, quitar o reemplazar una representación personalizada.

Puede usar Chat en web para enviar cualquier tipo de actividad y carga. No documentamos ni recomendamos esta característica. Debe usar la actividad "event" en su lugar.

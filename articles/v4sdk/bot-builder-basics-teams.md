---
title: Cómo funcionan los bots de Microsoft Teams
description: Una continuación del artículo sobre cómo funcionan los bots específico de los bots de Microsoft Teams
author: WashingtonKayaker
ms.author: kamrani
manager: kamrani
ms.topic: overview
ms.service: bot-service
ms.date: 10/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3a7e60e1fa72402b5f783676f5281579a2be5371
ms.sourcegitcommit: 490810d278d1c8207330b132f28a5eaf2b37bd07
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/05/2019
ms.locfileid: "73592203"
---
# <a name="how-microsoft-teams-bots-work"></a>Cómo funcionan los bots de Microsoft Teams

Se trata de una introducción que se basa en lo que aprendió en el artículo [Cómo funcionan los bots](https://docs.microsoft.com/azure/bot-service/bot-builder-basics), debe estar familiarizado con ese artículo antes de leer este.

Las principales diferencias en los bots desarrollados para Microsoft Teams están en cómo se controlan las actividades. El controlador de actividades de Microsoft Teams se deriva del controlador de actividades de Bot Framework para enrutar todas las actividades de Teams antes de permitir que se controlen las actividades específicas que no sean de Teams.

## <a name="teams-activity-handlers"></a>Controladores de actividades de Teams

Al igual que cualquier otro bot, cuando un bot diseñado para Microsoft Teams recibe una actividad, se la pasa a sus *controladores de actividades*. En segundo plano, hay un controlador base llamado *controlador de turnos* por el que se enrutan todas las actividades. El *controlador de turnos* llama al controlador de actividades necesario para controlar cualquier tipo de actividad que se haya recibido. La diferencia en un bot diseñado para Microsoft Teams es que se deriva de una clase `Activity Handler` de _Teams_, que se deriva de la clase `Activity Handler` de Bot Framework.  La clase _Teams_ `Activity Handler` incluye varios controladores de actividades específicos de Microsoft Teams que se tratarán en este artículo.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Como sucede con cualquier bot creado con Microsoft Bot Framework, si el bot recibe una actividad de mensaje, el controlador de turnos verá esa actividad entrante y la enviará al controlador de actividades `OnMessageActivityAsync`. Aunque esta funcionalidad sigue siendo la misma, si el bot recibe una actividad de actualización de conversación, el controlador de turnos verá esa actividad entrante y la enviará al controlador de actividades `OnConversationUpdateActivityAsync` de _Teams_, que comprobará primero los eventos específicos de Teams y la pasará al controlador de actividades de Bot Framework si no se encuentra ninguno.

En la clase del controlador de actividades de Teams hay dos controladores de actividades de Teams principales, `OnConversationUpdateActivityAsync`, que enruta todas las actividades de actualización de conversación y `OnInvokeActivityAsync` que enruta todas las actividades de invocación de Teams.  Todos los controladores de actividades que se describen en la sección [Lógica del bot](https://aka.ms/how-bots-work#bot-logic) del artículo `How bots work` seguirán funcionando como lo hacen con un bot que no es de Teams, con la excepción de controlar los miembros agregados y las actividades eliminadas, estas serán diferentes en el contexto de un equipo, donde el nuevo miembro se agrega al equipo en lugar de a un subproceso de mensaje.  Consulte la tabla _Actividades de actualización de conversación de Teams_ de la sección [Lógica del bot](#bot-logic) para más información.

Para implementar la lógica de estos controladores de actividades específicos de Teams, deberá reemplazar estos métodos en el bot como se muestra en la sección [Lógica del bot](#bot-logic), que encontrará más adelante. En estos controladores no hay implementación base, por lo que se puede agregar la lógica que se desee en la invalidación.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Como sucede con cualquier bot creado con Microsoft Bot Framework, si el bot recibe una actividad de mensaje, el controlador de turnos verá esa actividad entrante y la enviará al controlador de actividades `onMessage`. Aunque esta funcionalidad sigue siendo la misma, si el bot recibe una actividad de actualización de conversación, el controlador de turnos verá esa actividad entrante y la enviará al controlador de actividades `dispatchConversationUpdateActivity` de _Teams_, que comprobará primero los eventos específicos de Teams y la pasará al controlador de actividades de Bot Framework si no se encuentra ninguno.

En la clase del controlador de actividades de Teams hay dos controladores de actividades de Teams principales, `dispatchConversationUpdateActivity`, que enruta todas las actividades de actualización de conversación y `onInvokeActivity`, que enruta todas las actividades de invocación de Teams.  Todos los controladores de actividades que se describen en la sección [Lógica del bot](https://aka.ms/how-bots-work#bot-logic) del artículo `How bots work` seguirán funcionando como lo hacen con un bot que no es de Teams, con la excepción de controlar los miembros agregados y las actividades eliminadas, estas serán diferentes en el contexto de un equipo, donde el nuevo miembro se agrega al equipo en lugar de a un subproceso de mensaje.  Consulte la tabla _Actividades de actualización de conversación de Teams_ de la sección [Lógica del bot](#bot-logic) para más información.

Al igual que con cualquier controlador del bot, para implementar la lógica de los controladores de Teams, deberá reemplazar los métodos en el bot como se muestra en la sección [Lógica del bot](#bot-logic), que encontrará más adelante. Defina la lógica del bot de cada uno de estos controladores y, después, **no olvide llamar a `next()` al final**. Al llamar a `next()`, asegúrese de que se ejecuta el siguiente controlador.

---

### <a name="bot-logic"></a>Lógica del bot

La lógica del bot procesa las actividades entrantes de uno o varios canales de los bots y genera actividades salientes como respuesta.  Esto sigue siendo verdad en un bot derivado de la clase del controlador de actividades de Teams, que primero comprueba las actividades de Teams y, a continuación, pasa todas las demás actividades al controlador de actividades de Bot Framework.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

#### <a name="teams-conversation-update-activities"></a>Actividades de actualización de conversación de Teams

A continuación se muestra una lista de todos los controladores de actividades de Teams a los que se llama desde el controlador de actividades `OnConversationUpdateActivityAsync` de _Teams_. En el artículo [Eventos de actualización de conversación](https://aka.ms/azure-bot-subscribe-to-conversation-events) se describe cómo usar cada uno de estos eventos en un bot.

| Evento | Controlador | DESCRIPCIÓN |
| :-- | :-- | :-- |
| channelCreated | `OnTeamsChannelCreatedAsync` | Reemplazar para controlar un canal de Teams que se va a crear. Para más información, consulte [Channel created (Canal creado)](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-created). |
| channelDeleted | `OnTeamsChannelDeletedAsync` | Reemplazar para controlar un canal de Teams que se va a eliminar. Para más información, consulte [Channel deleted (Canal eliminado)](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-deleted). |
| channelRenamed | `OnTeamsChannelRenamedAsync` | Reemplazar para controlar un canal de Teams al que se va a cambiar el nombre. Para más información, consulte [Channel renamed (Nombre de canal cambiado)](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-renamed). |
| teamRenamed | `OnTeamsTeamRenamedAsync` | `return Task.CompletedTask;` Reemplazar para controlar un equipo de Teams al que se va a cambiar el nombre. Para más información, consulte [Team renamed (Nombre del equipo cambiado)](https://aka.ms/azure-bot-subscribe-to-conversation-events#team-renamed). |
| MembersAdded | `OnTeamsMembersAddedAsync` | Llama al método `OnMembersAddedAsync` de `ActivityHandler`. Reemplazar para controlar a los miembros que se unen a un equipo. Para más información, consulte [Team member added (Miembro del equipo agregado)](https://aka.ms/azure-bot-subscribe-to-conversation-events#Team-Member-Added).|
| MembersRemoved | `OnTeamsMembersRemovedAsync` | Llama al método `OnMembersRemovedAsync` de `ActivityHandler`. Reemplazar para controlar a los miembros que abandonan un equipo. Para más información, consulte [Team member removed (Miembro del equipo eliminado)](https://aka.ms/azure-bot-subscribe-to-conversation-events#Team-Member-Removed).|


<!--
| Event | Handler | Description |
| :-- | :-- | :-- |
| channelCreated | `OnTeamsChannelCreatedAsync` | Override this to handle a Teams channel being created. |
| channelDeleted | `OnTeamsChannelDeletedAsync` | Override this to handle a Teams channel being deleted. |
| channelRenamed | `OnTeamsChannelRenamedAsync` | Override this to handle a Teams channel being renamed. |
| teamRenamed | `OnTeamsTeamRenamedAsync` | `return Task.CompletedTask;` Override this to handle a Teams Team being Renamed. |
| MembersAdded | `OnTeamsMembersAddedAsync` | Calls the `OnMembersAddedAsync` method in `ActivityHandler`. Override this to handle members joining a team. |
| MembersRemoved | `OnTeamsMembersRemovedAsync` | Calls the `OnMembersRemovedAsync` method in `ActivityHandler`. Override this to handle members leaving a team. |
-->

#### <a name="teams-invoke--activities"></a>Actividades de invocación de Teams

A continuación se muestra una lista de todos los controladores de actividades de Teams a los que se llama desde el controlador de actividades `OnInvokeActivityAsync` de _Teams_:

| Tipos de invocación                    | Controlador                              | DESCRIPCIÓN                                                  |
| :-----------------------------  | :----------------------------------- | :----------------------------------------------------------- |
| CardAction.Invoke               | `OnTeamsCardActionInvokeAsync`       | Invocación de acción de tarjeta de Teams. |
| fileConsent/invoke              | `OnTeamsFileConsentAcceptAsync`      | Aceptación de consentimiento de archivo de Teams. |
| fileConsent/invoke              | `OnTeamsFileConsentAsync`            | Consentimiento de archivo de Teams. |
| fileConsent/invoke              | `OnTeamsFileConsentDeclineAsync`     | Consentimiento de archivo de Teams. |
| actionableMessage/executeAction | `OnTeamsO365ConnectorCardActionAsync` | Acción de tarjeta del conector de O365 de Teams. |
| signin/verifyState              | `OnTeamsSigninVerifyStateAsync`      | Comprobación de estado de inicio de sesión de Teams. |
| task/fetch                      | `OnTeamsTaskModuleFetchAsync`        | Captura del módulo de tareas de Teams. |
| task/submit                     | `OnTeamsTaskModuleSubmitAsync`       | Envío del módulo de tareas de Teams. |

Las actividades de invocación enumeradas anteriormente son para los bots de conversación de Teams. Bot Framework SDK también admite invocaciones específicas de las extensiones de mensajería. Para más información, consulte [¿Qué son las extensiones de mensajería?](https://aka.ms/azure-bot-what-are-messaging-extensions)

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

#### <a name="teams-conversation-update-activities"></a>Actividades de actualización de conversación de Teams

A continuación se muestra una lista de todos los controladores de actividades de Teams a los que se llama desde el controlador de actividades `dispatchConversationUpdateActivity` de _Teams_. En el artículo [Eventos de actualización de conversación](https://aka.ms/azure-bot-subscribe-to-conversation-events) se describe cómo usar cada uno de estos eventos en un bot.

| Evento | Controlador | DESCRIPCIÓN |
| :-- | :-- | :-- |
| channelCreated | `OnTeamsChannelCreatedEvent` | Reemplazar para controlar un canal de Teams que se va a crear. Para más información, consulte [Channel created (Canal creado)](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-created). |
| channelDeleted | `OnTeamsChannelDeletedEvent` | Reemplazar para controlar un canal de Teams que se va a eliminar. Para más información, consulte [Channel deleted (Canal eliminado)](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-deleted).|
| channelRenamed | `OnTeamsChannelRenamedEvent` | Reemplazar para controlar un canal de Teams al que se va a cambiar el nombre. Para más información, consulte [Channel renamed (Nombre de canal cambiado)](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-renamed). |
| teamRenamed | `OnTeamsTeamRenamedEvent` | `return Task.CompletedTask;` Reemplazar para controlar un equipo de Teams al que se va a cambiar el nombre. Para más información, consulte [Team renamed (Nombre del equipo cambiado)](https://aka.ms/azure-bot-subscribe-to-conversation-events#team-renamed). |
| MembersAdded | `OnTeamsMembersAddedEvent` | Llama al método `OnMembersAddedEvent` de `ActivityHandler`. Reemplazar para controlar a los miembros que se unen a un equipo. Para más información, consulte [Team member added (Miembro del equipo agregado)](https://aka.ms/azure-bot-subscribe-to-conversation-events#Team-Member-Added). |
| MembersRemoved | `OnTeamsMembersRemovedEvent` | Llama al método `OnMembersRemovedEvent` de `ActivityHandler`. Reemplazar para controlar a los miembros que abandonan un equipo. Para más información, consulte [Team member removed (Miembro del equipo eliminado)](https://aka.ms/azure-bot-subscribe-to-conversation-events#Team-Member-Removed). |

<!--
| Event | Handler | Description |
| :-- | :-- | :-- |
| channelCreated | `OnTeamsChannelCreatedEvent` | Override this to handle a Teams channel being created. |
| channelDeleted | `OnTeamsChannelDeletedEvent` | Override this to handle a Teams channel being deleted. |
| channelRenamed | `OnTeamsChannelRenamedEvent` | Override this to handle a Teams channel being renamed. |
| teamRenamed | `OnTeamsTeamRenamedEvent` | `return Task.CompletedTask;` Override this to handle a Teams Team being Renamed. |
| MembersAdded | `OnTeamsMembersAddedEvent` | Calls the `OnMembersAddedEvent` method in `ActivityHandler`. Override this to handle members joining a team. |
| MembersRemoved | `OnTeamsMembersRemovedEvent` | Calls the `OnMembersRemovedEvent` method in `ActivityHandler`. Override this to handle members leaving a team. |
-->

#### <a name="teams-invoke--activities"></a>Actividades de invocación de Teams

A continuación se muestra una lista de todos los controladores de actividades de Teams a los que se llama desde el controlador de actividades `onInvokeActivity` de _Teams_:

| Tipos de invocación                    | Controlador                              | DESCRIPCIÓN                                                  |
| :-----------------------------  | :----------------------------------- | :----------------------------------------------------------- |
| CardAction.Invoke               | `handleTeamsCardActionInvoke`       | Invocación de acción de tarjeta de Teams. |
| fileConsent/invoke              | `handleTeamsFileConsentAccept`      | Aceptación de consentimiento de archivo de Teams. |
| fileConsent/invoke              | `handleTeamsFileConsent`            | Consentimiento de archivo de Teams. |
| fileConsent/invoke              | `handleTeamsFileConsentDecline`     | Consentimiento de archivo de Teams. |
| actionableMessage/executeAction | `handleTeamsO365ConnectorCardAction` | Acción de tarjeta del conector de O365 de Teams. |
| signin/verifyState              | `handleTeamsSigninVerifyState`      | Comprobación de estado de inicio de sesión de Teams. |
| task/fetch                      | `handleTeamsTaskModuleFetch`        | Captura del módulo de tareas de Teams. |
| task/submit                     | `handleTeamsTaskModuleSubmit`       | Envío del módulo de tareas de Teams. |

Las actividades de invocación enumeradas anteriormente son para los bots de conversación de Teams. Bot Framework SDK también admite invocaciones específicas de las extensiones de mensajería. Para más información, consulte [¿Qué son las extensiones de mensajería?](https://aka.ms/azure-bot-what-are-messaging-extensions)

---

## <a name="next-steps"></a>Pasos siguientes
Para crear bots para Teams, consulte la [documentación](https://aka.ms/teams-docs) para desarrolladores de Microsoft Teams.
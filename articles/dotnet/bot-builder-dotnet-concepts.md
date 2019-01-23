---
title: Conceptos clave de Bot Framework SDK para .NET | Microsoft Docs
description: Descripción de los conceptos clave y las herramientas para la creación e implementación de bots de conversación disponibles en Bot Framework SDK para .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e01473a06a0cdbef635de33e5734b02351e36cea
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54223810"
---
# <a name="key-concepts-in-the-bot-framework-sdk-for-net"></a>Conceptos clave de Bot Framework SDK para .NET

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-concepts.md)
> - [Node.js](../nodejs/bot-builder-nodejs-concepts.md)

En este artículo se presentan los conceptos clave de Bot Framework SDK para .NET.

## <a name="connector"></a>Conector

El [conector de Bot Framework](bot-builder-dotnet-connector.md) proporciona una única API REST que permite que un bot se comunique a través de varios canales, como Skype, correo electrónico, Slack y mucho más. Facilita la comunicación entre el bot y el usuario mediante la retransmisión de mensajes del bot al canal y viceversa. 

En Bot Framework SDK para .NET, la biblioteca [Connector][connectorLibrary] habilita el acceso al conector. 

## <a name="activity"></a>Actividad

[!INCLUDE [Activity concept overview](../includes/snippet-dotnet-concept-activity.md)]

Para más información acerca de las actividades en Bot Framework SDK para .NET, consulte [Introducción a las actividades](bot-builder-dotnet-activities.md).

## <a name="dialog"></a>Diálogo

Al crear un bot con Bot Framework SDK para .NET, puede usar [diálogos](bot-builder-dotnet-dialogs.md) para modelar una conversación y administrar el [flujo de conversación](../bot-service-design-conversation-flow.md#dialog-stack). Un diálogo puede estar formado por otros diálogos para maximizar la reutilización, y un contexto de diálogo mantiene la [pila de diálogos](../bot-service-design-conversation-flow.md) que están activos en la conversación en cualquier momento dado. Una conversación que consta de diálogos se puede transferir entre varios equipos, lo que permite escalar la implementación del bot. 

En Bot Framework SDK para .NET, la biblioteca [Builder][builderLibrary] le permite administrar los diálogos.

## <a name="formflow"></a>FormFlow

Puede usar [FormFlow](bot-builder-dotnet-formflow.md) en Bot Framework SDK para .NET para simplificar la creación de un bot de que recopila información del usuario. Por ejemplo, un bot que recibe pedidos de sándwich debe recopilar cierta información del usuario, como tipo de pan, elección de ingredientes, tamaño, etc. Dadas las instrucciones básicas, FormFlow puede generar automáticamente los diálogos necesarios para administrar una conversación guiada como esta.

## <a name="state"></a>Estado

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

Para más información acerca de cómo administrar el estado con Bot Framework SDK para .NET, consulte [Administración de datos de estado](bot-builder-dotnet-state.md).

## <a name="naming-conventions"></a>Convenciones de nomenclatura

La biblioteca de Bot Framework SDK para .NET usa convenciones de nomenclatura fuertemente tipadas con PascalCase. Sin embargo, los mensajes JSON que se transportan de un lado para otro a través del cable utilizan convenciones de nomenclatura con CamelCase. Por ejemplo, la propiedad **ReplyToId** de C# se serializa como **replyToId** en el mensaje JSON que se transporta a través del cable.

## <a name="security"></a>Seguridad

Debe asegurarse de que solo el servicio del conector de Bot Framework pueda llamar al punto de conexión del bot. Para obtener más información sobre este tema, consulte [Secure your bot](bot-builder-dotnet-security.md) (Protección del bot).

## <a name="next-steps"></a>Pasos siguientes

Ya conoce los conceptos sobre cada bot. Puede [crear un bot con Visual Studio](bot-builder-dotnet-quickstart.md) rápidamente con una plantilla. A continuación, estudie cada concepto clave con más detalle, empezando por los diálogos.

> [!div class="nextstepaction"]
> [Diálogos en Bot Framework SDK para .NET](bot-builder-dotnet-dialogs.md)

[connectorLibrary]: /dotnet/api/microsoft.bot.connector

[builderLibrary]: /dotnet/api/microsoft.bot.builder.dialogs
